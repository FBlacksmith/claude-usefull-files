---
name: angular-modulith-mediatr
description: Scaffolds a production-ready Angular Modulith workspace using Bun, Tailwind CSS, mediatr-ts, and Sheriff architectural enforcement.
argument-hint: [AppName]
disable-model-invocation: true
user-invocable: true
---

### Angular Modulith + mediatr-ts Scaffolding Blueprint

Use this skill to scaffold an enterprise-ready Angular application structured as a **Modulith**, leveraging **Bun** as the package manager/runtime, **Tailwind CSS** for styles, **mediatr-ts** for decoupled CQRS in the frontend, and **Sheriff** for compile-time boundary enforcement.

The argument provided (`$0`) will be used as the Angular application name and workspace directory.

---

#### 1. Workspace Setup & CLI Rules (Bun-First)
*   **Package Manager:** Never use `npm` or `pnpm` commands. Always use **Bun** (`bun install`, `bun x`, `bun test`) to ensure lightning-fast installs and execution.
*   **LTS Angular CLI:** Always target the latest stable Angular release using Bun to run the Angular CLI.
*   **Clean Scaffolding:** Generate a clean standalone application without boilerplate components.

Run the following commands using the CLI to initialize the workspace inside the current directory:

```bash
# Install Angular CLI globally/locally with Bun and create a clean standalone app
bun x @angular/cli new $0 --directory . --style css --routing true --standalone true --package-manager bun

# Install Core Architecture Libraries
bun install mediatr-ts reflect-metadata
bun install -D @softarc/sheriff-core @softarc/eslint-plugin-sheriff tailwindcss postcss autoprefixer eslint
```

---

#### 2. Tailwind CSS Setup
To ensure utility-first styling without cluttering component-specific style files, initialize Tailwind CSS in the workspace:

```bash
bun x tailwindcss init -p
```

Configure `tailwind.config.js` to look for Angular templates and classes:
```javascript
/** @type {import('tailwindcss').Config} */
module.exports = {
  content: [
    "./src/**/*.{html,ts}",
  ],
  theme: {
    extend: {},
  },
  plugins: [],
}
```

Add the Tailwind directives to your global style sheet (`src/styles.css`):
```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

---

#### 3. Architectural Matrix & Folder Structure
Following Strategic Design and Moduliths, the codebase must be organized into **business-focused domains** (vertical slices) and **technical categories** (layers):

```
src/
└── app/
    ├── domains/
    │   ├── [domain-name]/                 # E.g., ticketing, booking, checkin
    │   │   ├── feature-[name]/            # Smart components, use case orchestration (interacts with MediatR)
    │   │   │   ├── internal/              # Private feature implementations
    │   │   │   └── public-api.ts          # Public exports (index.ts)
    │   │   ├── ui-[name]/                 # Dumb/presentational reusable components
    │   │   ├── data/                      # mediatr-ts commands, queries, handlers, state and domain models
    │   │   └── util/                      # Domain-specific helpers and utilities
    │   └── shared/                        # Shared domain-agnostic layer (technical logging, auth, etc.)
    │       ├── ui-common/
    │       ├── util-auth/
    │       └── data-shared/
    └── testing/                           # Shared testing helpers and mocks
```

---

#### 4. Information Hiding (Barrel-less / Internal Pattern)
To keep the codebase predictable while preserving tree-shaking and lazy loading, do not abuse massive global barrel files.
*   **`internal/` folders:** Private, non-exported implementations (e.g., helpers, specific forms) must be placed inside an `internal/` subdirectory inside each module.
*   **`public-api.ts` (or `index.ts`):** Only export the minimum necessary interface required by other modules (such as the main route config or the main entry component). Bypassing this API to access files in `internal/` is strictly prohibited.

---

#### 5. Compile-time Boundary Enforcement (Sheriff)
To enforce domain isolation and layer dependencies, we configure **Sheriff**. Under this architecture:
1.  **Domain Isolation:** A domain module can only depend on itself or on the `shared` domain (using `sameTag` rules).
2.  **Layer Hierarchy:**
    *   `feature` can depend on `ui`, `data`, and `util`.
    *   `ui` can depend on `data` and `util`.
    *   `data` can depend on `util`.
    *   `util` has NO dependencies on higher layers.
    *   *No higher layer can be imported into a lower layer.*

Create `sheriff.config.ts` in the root folder:

```typescript
import { sameTag, SheriffConfig } from '@softarc/sheriff-core';

export const config: SheriffConfig = {
  enableBarrelLess: true,
  modules: {
    'src/app/domains/<domain>': {
      'feature-<name>': ['domain:<domain>', 'type:feature'],
      'ui-<name>': ['domain:<domain>', 'type:ui'],
      'data': ['domain:<domain>', 'type:data'],
      'util-<name>': ['domain:<domain>', 'type:util'],
      
      // Fallbacks
      'data': ['domain:<domain>', 'type:data'],
      'ui': ['domain:<domain>', 'type:ui'],
      'util': ['domain:<domain>', 'type:util'],
    },
    'src/app/testing': ['testing'],
  },
  depRules: {
    root: '*',
    'domain:*': [sameTag, 'domain:shared'],
    
    'type:feature': ['type:ui', 'type:data', 'type:util'],
    'type:ui': ['type:data', 'type:util'],
    'type:data': ['type:util'],
    'type:util': [],
    
    testing: '*',
    '*': ['testing']
  }
};
```

Register Sheriff's ESLint bridge in `eslint.config.js`:
```javascript
import sheriff from "@softarc/eslint-plugin-sheriff";

export default [
  ...
  {
    plugins: {
      sheriff,
    },
    rules: {
      "sheriff/enforce-boundaries": "error",
    },
  },
];
```

---

#### 6. Decoupled Communication via mediatr-ts
Instead of components importing services from other domains directly (tight coupling), utilize **mediatr-ts** to dispatch and handle operations. This perfectly mirrors backend CQRS/Mediator architecture in the frontend.

##### Setup mediatr-ts Resolver
Import `reflect-metadata` once at the very top of your `main.ts` or `app.config.ts` before bootstrap:
```typescript
import 'reflect-metadata';
```

##### Command / Query Definition (`src/app/domains/ticketing/data/`)
```typescript
import { IRequest } from 'mediatr-ts';

// Query to get user tickets
export class GetTicketsQuery implements IRequest<Ticket[]> {
  constructor(public readonly userId: string) {}
}

// Command to book a flight
export class BookFlightCommand implements IRequest<boolean> {
  constructor(public readonly flightId: string, public readonly seat: string) {}
}
```

##### Handler Implementation (`src/app/domains/ticketing/data/internal/`)
```typescript
import { IRequestHandler, requestHandler } from 'mediatr-ts';
import { GetTicketsQuery } from '../get-tickets.query';
import { Ticket } from '../models/ticket.model';

@requestHandler(GetTicketsQuery)
export class GetTicketsHandler implements IRequestHandler<GetTicketsQuery, Ticket[]> {
  async handle(value: GetTicketsQuery): Promise<Ticket[]> {
    // Process business logic or call backend API
    return [
      { id: '1', route: 'New York - London', price: 300 }
    ];
  }
}
```

##### Injecting and Dispatching in a Smart Component (`src/app/domains/ticketing/feature-booking/`)
```typescript
import { Component, OnInit, inject, signal } from '@angular/core';
import { Mediator } from 'mediatr-ts';
import { GetTicketsQuery } from '../data/get-tickets.query';
import { Ticket } from '../data/models/ticket.model';

@Component({
  selector: 'ticketing-booking',
  standalone: true,
  template: `
    <div class="p-6 bg-gray-100 rounded-lg">
      <h2 class="text-2xl font-bold text-gray-900 mb-4">My Bookings</h2>
      @for (ticket of tickets(); track ticket.id) {
        <div class="p-4 bg-white shadow mb-2 rounded border-l-4 border-indigo-500">
          <p class="font-medium">{{ ticket.route }}</p>
          <span class="text-sm text-gray-500">$ {{ ticket.price }}</span>
        </div>
      }
    </div>
  `
})
export class BookingComponent implements OnInit {
  private mediator = inject(Mediator);
  tickets = signal<Ticket[]>([]);

  async ngOnInit() {
    // Clean, loosely-coupled command/query orchestration
    const result = await this.mediator.send<Ticket[]>(new GetTicketsQuery('user-123'));
    this.tickets.set(result);
  }
}
```

---

#### 7. TypeScript Path Mappings (`tsconfig.json`)
To avoid fragile, unreadable relative imports (e.g., `../../../../data`), configure lightweight path mappings in `tsconfig.json`:

```json
{
  "compilerOptions": {
    "baseUrl": "./",
    "paths": {
      "@$0/*": ["src/app/domains/*"]
    }
  }
}
```

With this, you can always import from the architecture matrix cleanly:
```typescript
import { GetTicketsQuery } from '@$0/ticketing/data';
```
