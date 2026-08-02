# Angular Enterprise Modulith - Architectural Directives

This document defines the strict architectural constraints, patterns, and code standards for development in **Angular 21+**, utilizing a **Vertical Modulith** architecture enforced by **Sheriff**, state orchestration via **mediatr-ts**, styling with **Tailwind CSS**, and **Bun** as the high-performance package manager and runtime.

---

## 🛠️ 1. General Settings & Engine Config
*   **Surgery Mode:** Enabled (localized, incremental, and highly surgical edits to existing files).
*   **Token Economy (No Prose):** Enabled. Claude must generate code directly without introductory or explanatory preambles. Output code blocks or brief technical tables first.
*   **Default CLI & Runtime:** **Bun** (`bun install`, `bun x ng`, `bun test`). Never use `npm`, `pnpm`, or `yarn`.

---

## 📂 2. Modulith Project Structure
We follow Manfred Steyer's Strategic Design Vertical Slicing. The application is divided into self-contained business **domains** (verticals) and technical layers.

```
src/app/
├── domains/
│   ├── [domain-name]/                <-- e.g., assets, booking
│   │   ├── feature-[slice]/          <-- Smart components & Use Cases
│   │   │   ├── internal/             <-- Private components/services
│   │   │   └── public-api.ts         <-- Explicit exports (if not barrel-less)
│   │   ├── ui-[slice]/               <-- Reusable dumb/presentational components
│   │   ├── data/                     <-- Domain models, API services, MediatR state
│   │   └── util/                     <-- Domain-specific helper utilities
│   └── shared/                       <-- Cross-cutting technical domain
│       ├── ui/                       <-- Global Design System presentational elements
│       └── util/                     <-- Global helpers (JWT, logging, auth guards)
```

### Architectural Layer Rules:
1.  **feature:** Contains Smart Components that handle orchestrations, communicate with MediatR/Backend services, and are non-reusable.
2.  **ui:** Contains Dumb (Presentational) Components. They have no domain knowledge, do not inject services or MediatR, and communicate solely via `@Input()` and `@Output()` (or modern Angular Signals inputs and outputs).
3.  **data:** Houses domain interfaces/types, state management schemas, API clients, and MediatR handlers.
4.  **util:** Houses domain-specific pure functions, custom pipes, or local configuration files.

---

## 🛡️ 3. Architectural Enforcement (Sheriff)
Acoplamentos e dependências proibidas são impedidos em tempo de linting usando o **Sheriff** com a opção `enableBarrelLess` ativa.

Crie ou mantenha o arquivo `sheriff.config.ts` na raiz do projeto configurado da seguinte forma:

```typescript
import { sameTag, SheriffConfig } from '@softarc/sheriff-core';

export const config: SheriffConfig = {
  enableBarrelLess: true, // Habilita ocultação de código na pasta "internal/" sem exigir arquivos index.ts
  
  modules: {
    'src/app/domains/<domain>': {
      'feature-<name>': ['domain:<domain>', 'type:feature'],
      'ui-<name>': ['domain:<domain>', 'type:ui'],
      'data': ['domain:<domain>', 'type:data'],
      'util-<name>': ['domain:<domain>', 'type:util'],
      
      // Fallbacks para estruturas de pastas simplificadas
      data: ['domain:<domain>', 'type:data'],
      ui: ['domain:<domain>', 'type:ui'],
      util: ['domain:<domain>', 'type:util'],
    },
    'src/app/domains/shared': ['domain:shared'],
    'src/app/testing': ['testing']
  },
  
  depRules: {
    root: '*',
    
    // Regra 1: Isolamento de Domínio (Domínios não se acoplam entre si, apenas ao shared)
    'domain:<domain>': [sameTag, 'domain:shared'],
    
    // Regra 2: Fluxo de Camadas Técnicas Inward (Feature -> UI -> Data -> Util)
    'type:feature': ['type:ui', 'type:data', 'type:util'],
    'type:ui': ['type:data', 'type:util'],
    'type:data': ['type:util'],
    'type:util': [],
    
    'testing': '*'
  }
};
```

### 🔒 Information Hiding Pattern
*   Toda lógica de implementação que for estritamente privada de uma camada ou componente deve ser colocada dentro de uma subpasta chamada `internal/`.
*   O Sheriff garante que arquivos fora do módulo que tentem importar de subpastas `internal/` disparem um erro de compilação/linting.
*   **Exemplo:** `src/app/domains/checkin/data/internal/validation.ts` só pode ser importado por arquivos sob `src/app/domains/checkin/data/`.

---

## 🧠 4. State Management & CQRS with MediatR-TS
Não acople componentes diretamente a serviços de API ou stores globais de outros domínios. Toda intenção ou consulta do frontend é despachada como um Command ou Query utilizando `mediatr-ts` com Inversify para Dependency Injection.

### Exemplo de Implementação de Fluxo MediatR:

#### 1. Command Definition (`data/commands/create-asset.command.ts`)
```typescript
import { IRequest } from 'mediatr-ts';

export class CreateAssetCommand implements IRequest<string> {
  constructor(
    public readonly name: string,
    public readonly ticker: string,
    public readonly targetAllocation: number
  ) {}
}
```

#### 2. Handler Implementation (`data/handlers/create-asset.handler.ts`)
```typescript
import { IRequestHandler, requestHandler } from 'mediatr-ts';
import { CreateAssetCommand } from '../commands/create-asset.command';
import { inject, injectable } from 'inversify';
import { AssetApiService } from '../services/asset-api.service';

@requestHandler(CreateAssetCommand)
@injectable()
export class CreateAssetHandler implements IRequestHandler<CreateAssetCommand, string> {
  constructor(
    @inject(AssetApiService) private apiService: AssetApiService
  ) {}

  async handle(value: CreateAssetCommand): Promise<string> {
    const result = await this.apiService.post({
      name: value.name,
      ticker: value.ticker,
      target: value.targetAllocation
    });
    return result.id;
  }
}
```

#### 3. Component Dispatching (`feature-manage/manage.component.ts`)
```typescript
import { Component, inject } from '@angular/core';
import { Mediator } from 'mediatr-ts';
import { CreateAssetCommand } from '../../data/commands/create-asset.command';

@Component({
  selector: 'app-manage-assets',
  template: `
    <button (click)="onRegister()" class="bg-blue-600 text-white px-4 py-2 rounded-lg">
      Cadastrar Ativo
    </button>
  `
})
export class ManageAssetsComponent {
  private mediator = inject(Mediator);

  async onRegister(): Promise<void> {
    const command = new CreateAssetCommand('Investimento A', 'INVA11', 15.0);
    const assetId = await this.mediator.send<string>(command);
    console.log('Ativo criado:', assetId);
  }
}
```

---

## 🎨 5. UI & Styling Rules (Tailwind CSS)
*   **Utility-First:** Proibido escrever arquivos `.css` ou `.scss` customizados adicionais para estilização. Todo o layout e espaçamento deve ser feito de forma utilitária diretamente no HTML utilizando as classes de design utilitárias do Tailwind CSS.
*   **Sinalização Reativa:** Sempre priorize o uso de modern **Angular Signals** (`signal()`, `computed()`) para gerenciar as mudanças de estado da tela nos componentes em vez de RxJS/BehaviorSubjects para estados locais e simples de visualização.

---

## 🚀 6. Verificações de Qualidade Pré-Commit
Antes de finalizar qualquer implementação em Angular, o agente deve rodar os seguintes comandos locais para garantir a conformidade arquitetural:
1.  `bun run lint` (Para garantir que as restrições do Sheriff e do ESLint foram plenamente atendidas).
2.  `bun test` (Para rodar o suite de testes unitários).
