# Plano de Viabilidade Técnica - Detetive Existencial Companion App

**Data:** 26 de Dezembro de 2025  
**Versão:** 1.0  
**Objetivo:** Avaliar viabilidade técnica e recomendar stack tecnológico

---

## 1. ANÁLISE DE REQUISITOS TÉCNICOS

### 1.1. Requisitos Funcionais Críticos

#### RF1: Funcionamento Offline Completo

- **Prioridade:** CRÍTICA
- **Descrição:** App deve funcionar 100% sem conexão à internet
- **Implicações:**
  - Armazenamento local robusto (IndexedDB)
  - Service Workers para cache de assets
  - Sincronização futura (opcional)

#### RF2: Instalação como App Nativo

- **Prioridade:** ALTA
- **Descrição:** Usuário deve poder "instalar" via navegador
- **Implicações:**
  - PWA manifest configurado
  - Service Worker registrado
  - Ícones e splash screens

#### RF3: Responsividade Mobile/Desktop

- **Prioridade:** CRÍTICA
- **Descrição:** Layout adaptável para telas 320px-4K
- **Implicações:**
  - Design mobile-first
  - Breakpoints bem definidos
  - Touch-friendly UI

#### RF4: Exportação/Importação de Dados

- **Prioridade:** ALTA
- **Descrição:** Exportar fichas e campanhas em JSON e Markdown
- **Implicações:**
  - Serialização de dados complexos
  - Parser de Markdown
  - Validação de schemas

#### RF5: Performance

- **Prioridade:** ALTA
- **Descrição:** Carregamento rápido, interações fluidas
- **Implicações:**
  - Code splitting
  - Lazy loading
  - Otimização de bundle

### 1.2. Requisitos Não-Funcionais

| Requisito                         | Meta                                | Medição                  |
| --------------------------------- | ----------------------------------- | ------------------------ |
| **Tempo de Carregamento Inicial** | < 3s (3G)                           | Lighthouse Performance   |
| **Tempo de Interação**            | < 100ms                             | First Input Delay        |
| **Tamanho do Bundle**             | < 500KB (gzipped)                   | Webpack Bundle Analyzer  |
| **Acessibilidade**                | WCAG 2.1 AA                         | Lighthouse Accessibility |
| **Compatibilidade**               | Chrome 90+, Safari 14+, Firefox 88+ | BrowserStack             |
| **Armazenamento Local**           | Suporte a 50+ personagens           | IndexedDB quota          |

---

## 2. COMPARAÇÃO DE TECNOLOGIAS

### 2.1. Opção 1: Next.js 14+ (App Router)

#### Prós

✅ **PWA Nativo:** Plugin `next-pwa` com suporte completo  
✅ **Performance:** React Server Components, streaming SSR  
✅ **SEO:** Renderização server-side para landing page  
✅ **Developer Experience:** TypeScript, Hot Reload, ESLint integrado  
✅ **Ecossistema:** Vasto, com bibliotecas maduras  
✅ **Deployment:** Vercel (zero-config), Netlify, self-hosted  
✅ **Offline-First:** Service Workers com estratégias de cache avançadas  
✅ **Code Splitting:** Automático por rota

#### Contras

❌ **Curva de Aprendizado:** App Router é novo (2023)  
❌ **Overhead:** Mais pesado que Vite para apps simples  
❌ **Complexidade:** Conceitos de Server/Client Components

#### Viabilidade: ⭐⭐⭐⭐⭐ (5/5)

**Recomendação:** **ALTAMENTE RECOMENDADO**

---

### 2.2. Opção 2: React + Vite

#### Prós

✅ **Leveza:** Bundle menor, build mais rápido  
✅ **Simplicidade:** Menos conceitos abstratos  
✅ **Performance de Dev:** HMR extremamente rápido  
✅ **Flexibilidade:** Controle total sobre configuração

#### Contras

❌ **PWA Manual:** Requer configuração manual de Service Workers  
❌ **Sem SSR:** Apenas client-side rendering  
❌ **Menos Baterias:** Precisa configurar roteamento, meta tags, etc.  
❌ **SEO Limitado:** Sem renderização server-side

#### Viabilidade: ⭐⭐⭐⭐ (4/5)

**Recomendação:** Viável, mas requer mais trabalho manual

---

### 2.3. Opção 3: React Native (Expo)

#### Prós

✅ **Apps Nativos:** Publicação em App Store e Google Play  
✅ **Performance Nativa:** Acesso a APIs nativas  
✅ **Expo:** Simplifica desenvolvimento e build

#### Contras

❌ **Não é Web:** Requer publicação em stores (custo, aprovação)  
❌ **Complexidade:** Duas bases de código (iOS + Android)  
❌ **Overhead:** Mais pesado que PWA  
❌ **Distribuição:** Usuário precisa baixar da store  
❌ **Atualizações:** Processo de review para updates

#### Viabilidade: ⭐⭐ (2/5)

**Recomendação:** **NÃO RECOMENDADO** para este caso de uso

---

### 2.4. Decisão Final

| Critério                 | Next.js | Vite    | React Native | Peso |
| ------------------------ | ------- | ------- | ------------ | ---- |
| **PWA Support**          | 5/5     | 3/5     | 1/5          | 30%  |
| **Offline-First**        | 5/5     | 4/5     | 5/5          | 25%  |
| **Developer Experience** | 5/5     | 5/5     | 3/5          | 15%  |
| **Performance**          | 5/5     | 5/5     | 4/5          | 15%  |
| **Deployment**           | 5/5     | 4/5     | 2/5          | 10%  |
| **Manutenibilidade**     | 5/5     | 4/5     | 3/5          | 5%   |
| **TOTAL**                | **5.0** | **4.0** | **2.8**      | 100% |

### 🏆 Vencedor: **Next.js 14+ (App Router)**

---

## 3. ARQUITETURA TÉCNICA DETALHADA

### 3.1. Stack Tecnológico Completo

```
Frontend Framework
├── Next.js 14+ (App Router)
├── React 18+
└── TypeScript 5+

Styling
├── Tailwind CSS 3+
├── CSS Modules (para componentes específicos)
└── Radix UI (componentes acessíveis)

State Management
├── Zustand (global state)
├── React Hook Form (forms)
└── Zod (validação de schemas)

Internationalization
├── next-intl (i18n framework)
├── deepl-node (AI translation)
└── @libretranslate/client (fallback)

Multiplayer & Realtime
├── firebase (Realtime Database)
├── @firebase/auth (autenticação)
└── @firebase/database (sync tempo real)

Data Layer
├── Dexie.js (IndexedDB wrapper)
├── Immer (immutable updates)
└── date-fns (manipulação de datas)

PWA
├── next-pwa (Service Workers)
├── workbox (cache strategies)
└── web-vitals (performance monitoring)

Testing
├── Vitest (unit tests)
├── Testing Library (component tests)
└── Playwright (E2E tests)

Development Tools
├── ESLint + Prettier
├── Husky (git hooks)
├── Commitlint (conventional commits)
└── TypeScript strict mode
```

### 3.2. Arquitetura de Dados

#### 3.2.1. Modelo de Dados (TypeScript Schemas)

```typescript
// Character Schema
interface Character {
  id: string; // UUID
  name: string;
  createdAt: Date;
  updatedAt: Date;

  // Atributos (8 pontos totais)
  attributes: {
    intellect: number; // 1-5
    psyche: number; // 1-5
    physique: number; // 1-5
    motorics: number; // 1-5
  };

  // Perícias (24 total, 6 por atributo)
  skills: {
    // INTELECTO
    logic: number;
    encyclopedia: number;
    rhetoric: number;
    conceptualization: number;
    visualCalculus: number;
    drama: number;

    // PSIQUE
    volition: number;
    inlandEmpire: number;
    empathy: number;
    authority: number;
    espritDeCorps: number;
    suggestion: number;

    // FÍSICO
    endurance: number;
    painThreshold: number;
    physicalInstrument: number;
    electrochemistry: number;
    shivers: number;
    halfLight: number;

    // MOTRICIDADE
    handEyeCoordination: number;
    perception: number;
    reactionSpeed: number;
    savoirFaire: number;
    interfacing: number;
    composure: number;
  };

  // Recursos
  resources: {
    morale: number;
    moraleMax: number;
    health: number;
    healthMax: number;
    money: number; // R$
    xp: number;
  };

  // Gabinete de Reflexões
  thoughtCabinet: {
    slots: number; // 3-12
    thoughts: Thought[];
  };

  // Inventário
  inventory: Item[];

  // Condições ativas
  conditions: Condition[];

  // Histórico
  history: {
    rollHistory: Roll[];
    xpHistory: XPGain[];
    progressionHistory: Progression[];
  };
}

// Thought (Reflexão)
interface Thought {
  id: string;
  name: string;
  level: 1 | 2 | 3 | 4;
  status: 'processing' | 'internalized';
  acquiredAt: Date;
  internalizedAt?: Date;
  problem: string; // Descrição da penalidade
  solution: string; // Descrição do bônus
}

// Campaign Schema
interface Campaign {
  id: string;
  name: string;
  description: string;
  createdAt: Date;
  updatedAt: Date;
  narratorId: string; // Identificador do narrador (pode ser device ID)

  // Personagens vinculados (N:N relationship)
  characterLinks: CampaignCharacterLink[];

  // NPCs
  npcs: NPC[];

  // Sessões
  sessions: Session[];

  // Saúde da Cidade
  cityHealth: {
    morale: number; // 0-10
    health: number; // 0-10
  };

  // Anotações do Narrador (privadas)
  narratorNotes: NarratorNote[];
}

// Character-Campaign Link (permite N personagens em N campanhas)
interface CampaignCharacterLink {
  characterId: string;
  campaignId: string;
  joinedAt: Date;
  isActive: boolean; // Personagem ainda ativo na campanha?
}

// Session Schema
interface Session {
  id: string;
  campaignId: string;
  sessionNumber: number;
  date: Date;
  summary: string; // Visível para todos
  narratorSummary: string; // Apenas narrador
  events: SessionEvent[];
  xpAwarded: number;
}

// Session Event (com controle de visibilidade)
interface SessionEvent {
  id: string;
  description: string;
  timestamp: Date;
  isRevealed: boolean; // Narrador pode revelar/ocultar
  type: 'roll' | 'note' | 'combat' | 'discovery' | 'other';
}

// Narrator Note (sempre privada até revelada)
interface NarratorNote {
  id: string;
  content: string;
  createdAt: Date;
  isRevealed: boolean;
  revealedAt?: Date;
  tags: string[];
}

// Export/Import Schema
interface ExportData {
  version: string; // Schema version
  exportedAt: Date;
  type: 'character' | 'campaign' | 'full';
  mode: 'narrator' | 'player'; // Determina se exporta dados privados
  data:
    | Character
    | Campaign
    | {
        characters: Character[];
        campaigns: Campaign[];
        characterLinks: CampaignCharacterLink[];
      };
}

// View Mode (controla visibilidade)
interface ViewMode {
  mode: 'narrator' | 'player';
  campaignId?: string; // Se em modo jogador, qual campanha
  characterId?: string; // Se em modo jogador, qual personagem
}
```

#### 3.2.2. IndexedDB Structure (Dexie.js)

```typescript
import Dexie, { Table } from 'dexie';

class DEDatabase extends Dexie {
  characters!: Table<Character>;
  campaigns!: Table<Campaign>;
  sessions!: Table<Session>;
  npcs!: Table<NPC>;
  notes!: Table<Note>;
  settings!: Table<Setting>;

  constructor() {
    super('DetetiveExistencialDB');
    this.version(1).stores({
      characters: 'id, name, createdAt, updatedAt',
      campaigns: 'id, name, createdAt, updatedAt',
      sessions: 'id, campaignId, date',
      npcs: 'id, campaignId, name',
      notes: 'id, campaignId, createdAt',
      settings: 'key',
    });
  }
}

export const db = new DEDatabase();
```

### 3.3. Arquitetura de Componentes

```
app/
├── (marketing)/
│   ├── page.tsx                 # Landing page (SSR)
│   └── about/page.tsx
├── (app)/
│   ├── layout.tsx               # App layout (PWA shell)
│   ├── characters/
│   │   ├── page.tsx             # Lista de personagens
│   │   ├── new/page.tsx         # Criação de personagem
│   │   └── [id]/
│   │       ├── page.tsx         # Ficha do personagem
│   │       ├── edit/page.tsx
│   │       └── sheet/page.tsx   # Ficha interativa
│   ├── campaigns/
│   │   ├── page.tsx
│   │   ├── new/page.tsx
│   │   └── [id]/
│   │       ├── page.tsx
│   │       ├── sessions/page.tsx
│   │       └── npcs/page.tsx
│   ├── repository/
│   │   ├── page.tsx
│   │   ├── skills/page.tsx
│   │   ├── thoughts/page.tsx
│   │   └── reference/page.tsx
│   └── tools/
│       ├── dice-roller/page.tsx
│       └── random-events/page.tsx
├── api/                         # API routes (se necessário)
└── components/
    ├── character/
    │   ├── AttributeSelector.tsx
    │   ├── SkillSelector.tsx
    │   ├── ThoughtCabinet.tsx
    │   └── CharacterSheet.tsx
    ├── dice/
    │   ├── DiceRoller.tsx
    │   └── RollHistory.tsx
    ├── campaign/
    │   ├── CampaignManager.tsx
    │   └── SessionPlanner.tsx
    └── ui/
        ├── Button.tsx
        ├── Card.tsx
        ├── Input.tsx
        └── Modal.tsx
```

### 3.4. PWA Configuration

#### 3.4.1. next.config.js

```javascript
const withPWA = require('next-pwa')({
  dest: 'public',
  register: true,
  skipWaiting: true,
  disable: process.env.NODE_ENV === 'development',
  runtimeCaching: [
    {
      urlPattern: /^https:\/\/fonts\.(?:googleapis|gstatic)\.com\/.*/i,
      handler: 'CacheFirst',
      options: {
        cacheName: 'google-fonts',
        expiration: {
          maxEntries: 4,
          maxAgeSeconds: 365 * 24 * 60 * 60, // 1 year
        },
      },
    },
    {
      urlPattern: /\.(?:eot|otf|ttc|ttf|woff|woff2|font.css)$/i,
      handler: 'StaleWhileRevalidate',
      options: {
        cacheName: 'static-font-assets',
        expiration: {
          maxEntries: 4,
          maxAgeSeconds: 7 * 24 * 60 * 60, // 1 week
        },
      },
    },
    {
      urlPattern: /\.(?:jpg|jpeg|gif|png|svg|ico|webp)$/i,
      handler: 'StaleWhileRevalidate',
      options: {
        cacheName: 'static-image-assets',
        expiration: {
          maxEntries: 64,
          maxAgeSeconds: 24 * 60 * 60, // 24 hours
        },
      },
    },
    {
      urlPattern: /\.(?:js)$/i,
      handler: 'StaleWhileRevalidate',
      options: {
        cacheName: 'static-js-assets',
        expiration: {
          maxEntries: 32,
          maxAgeSeconds: 24 * 60 * 60, // 24 hours
        },
      },
    },
    {
      urlPattern: /\.(?:css|less)$/i,
      handler: 'StaleWhileRevalidate',
      options: {
        cacheName: 'static-style-assets',
        expiration: {
          maxEntries: 32,
          maxAgeSeconds: 24 * 60 * 60, // 24 hours
        },
      },
    },
    {
      urlPattern: /\/api\/.*/i,
      handler: 'NetworkFirst',
      options: {
        cacheName: 'apis',
        expiration: {
          maxEntries: 16,
          maxAgeSeconds: 24 * 60 * 60, // 24 hours
        },
        networkTimeoutSeconds: 10,
      },
    },
    {
      urlPattern: /.*/i,
      handler: 'NetworkFirst',
      options: {
        cacheName: 'others',
        expiration: {
          maxEntries: 32,
          maxAgeSeconds: 24 * 60 * 60, // 24 hours
        },
        networkTimeoutSeconds: 10,
      },
    },
  ],
});

module.exports = withPWA({
  reactStrictMode: true,
  typescript: {
    ignoreBuildErrors: false,
  },
});
```

#### 3.4.2. manifest.json

```json
{
  "name": "Detetive Existencial Companion",
  "short_name": "DE Companion",
  "description": "Companion app para o RPG de mesa Detetive Existencial",
  "start_url": "/",
  "display": "standalone",
  "background_color": "#0a0a0a",
  "theme_color": "#1a1a1a",
  "orientation": "portrait-primary",
  "icons": [
    {
      "src": "/icons/icon-72x72.png",
      "sizes": "72x72",
      "type": "image/png"
    },
    {
      "src": "/icons/icon-96x96.png",
      "sizes": "96x96",
      "type": "image/png"
    },
    {
      "src": "/icons/icon-128x128.png",
      "sizes": "128x128",
      "type": "image/png"
    },
    {
      "src": "/icons/icon-144x144.png",
      "sizes": "144x144",
      "type": "image/png"
    },
    {
      "src": "/icons/icon-152x152.png",
      "sizes": "152x152",
      "type": "image/png"
    },
    {
      "src": "/icons/icon-192x192.png",
      "sizes": "192x192",
      "type": "image/png"
    },
    {
      "src": "/icons/icon-384x384.png",
      "sizes": "384x384",
      "type": "image/png"
    },
    {
      "src": "/icons/icon-512x512.png",
      "sizes": "512x512",
      "type": "image/png"
    }
  ],
  "categories": ["games", "utilities"],
  "screenshots": [
    {
      "src": "/screenshots/desktop-1.png",
      "sizes": "1280x720",
      "type": "image/png",
      "form_factor": "wide"
    },
    {
      "src": "/screenshots/mobile-1.png",
      "sizes": "750x1334",
      "type": "image/png",
      "form_factor": "narrow"
    }
  ]
}
```

---

## 4. ANÁLISE DE RISCOS TÉCNICOS

### 4.1. Riscos Identificados

| Risco                                | Probabilidade | Impacto | Mitigação                                             |
| ------------------------------------ | ------------- | ------- | ----------------------------------------------------- |
| **Quota de IndexedDB**               | Baixa         | Alto    | Implementar limpeza de dados antigos, alertar usuário |
| **Incompatibilidade de Navegadores** | Média         | Médio   | Polyfills, feature detection, fallbacks               |
| **Performance em Mobile Antigo**     | Média         | Médio   | Code splitting agressivo, lazy loading                |
| **Bugs no Service Worker**           | Média         | Alto    | Testes extensivos, estratégia de fallback             |
| **Complexidade de Exportação**       | Baixa         | Baixo   | Schemas bem definidos, validação robusta              |
| **Perda de Dados**                   | Baixa         | Crítico | Backup automático, exportação fácil                   |

### 4.2. Plano de Contingência

#### Quota de IndexedDB Excedida

```typescript
async function checkStorageQuota() {
  if ('storage' in navigator && 'estimate' in navigator.storage) {
    const { usage, quota } = await navigator.storage.estimate();
    const percentUsed = (usage! / quota!) * 100;

    if (percentUsed > 80) {
      // Alertar usuário para exportar dados
      showWarning('Armazenamento quase cheio. Exporte seus dados.');
    }
  }
}
```

#### Fallback para Navegadores Antigos

```typescript
const supportsIndexedDB = 'indexedDB' in window;
const supportsServiceWorker = 'serviceWorker' in navigator;

if (!supportsIndexedDB) {
  // Fallback para localStorage (limitado)
  console.warn('IndexedDB não suportado, usando localStorage');
}

if (!supportsServiceWorker) {
  // App funciona, mas sem cache offline
  console.warn('Service Workers não suportados, sem modo offline');
}
```

---

## 5. ESTIMATIVA DE ESFORÇO

### 5.1. Breakdown de Desenvolvimento

| Fase                        | Tarefas                                | Horas    | Complexidade |
| --------------------------- | -------------------------------------- | -------- | ------------ |
| **Setup Inicial**           | Next.js, TypeScript, Tailwind, Dexie   | 16h      | Baixa        |
| **Design System**           | Componentes UI, temas, responsividade  | 40h      | Média        |
| **Data Layer**              | Schemas, IndexedDB, CRUD operations    | 32h      | Média        |
| **Criação de Personagem**   | Formulário, validação, cálculos        | 48h      | Alta         |
| **Ficha Interativa**        | Visualização, edição, rolagem de dados | 64h      | Alta         |
| **Gabinete de Reflexões**   | Processamento, internalização, UI      | 32h      | Média        |
| **Repositório de Conteúdo** | Perícias, reflexões, tabelas           | 24h      | Baixa        |
| **Ferramentas do Narrador** | Campanhas, NPCs, sessões               | 56h      | Alta         |
| **Exportação/Importação**   | JSON, Markdown, validação              | 24h      | Média        |
| **PWA Configuration**       | Service Workers, manifest, ícones      | 16h      | Média        |
| **Testes**                  | Unit, integration, E2E                 | 48h      | Média        |
| **Polish & Bugs**           | Refinamentos, correções                | 40h      | Variável     |
| **TOTAL**                   |                                        | **440h** |              |

### 5.2. Cronograma Realista

**Assumindo 1 desenvolvedor full-time (40h/semana):**

- **Duração:** 11 semanas (~3 meses)

**Assumindo 1 desenvolvedor part-time (20h/semana):**

- **Duração:** 22 semanas (~5-6 meses)

**Assumindo equipe de 2 desenvolvedores (40h/semana cada):**

- **Duração:** 6 semanas (~1.5 meses)

---

## 6. REQUISITOS DE INFRAESTRUTURA

### 6.1. Desenvolvimento

- **Hardware:** Computador moderno (8GB+ RAM)
- **Software:** Node.js 18+, Git, VS Code
- **Serviços:** GitHub (repositório), Vercel (preview deployments)

### 6.2. Produção

- **Hospedagem:** Vercel (gratuito para projetos open source)
  - Bandwidth: Ilimitado
  - Build time: 100h/mês (suficiente)
  - Deployments: Ilimitados
- **Domínio:** Opcional (~R$ 40/ano)
- **CDN:** Cloudflare (gratuito)
- **Monitoramento:** Vercel Analytics (gratuito)

### 6.3. Custos Operacionais

| Item                | Custo Mensal | Custo Anual |
| ------------------- | ------------ | ----------- |
| Hospedagem (Vercel) | R$ 0         | R$ 0        |
| Domínio             | R$ 3         | R$ 40       |
| CDN (Cloudflare)    | R$ 0         | R$ 0        |
| **TOTAL**           | **R$ 3**     | **R$ 40**   |

---

## 7. MÉTRICAS DE SUCESSO TÉCNICO

### 7.1. Performance Targets

| Métrica                     | Target  | Ferramenta              |
| --------------------------- | ------- | ----------------------- |
| **Lighthouse Performance**  | > 90    | Chrome DevTools         |
| **First Contentful Paint**  | < 1.5s  | Web Vitals              |
| **Time to Interactive**     | < 3.0s  | Web Vitals              |
| **Cumulative Layout Shift** | < 0.1   | Web Vitals              |
| **Bundle Size (Initial)**   | < 300KB | Next.js Bundle Analyzer |
| **Bundle Size (Total)**     | < 500KB | Next.js Bundle Analyzer |

### 7.2. Quality Targets

| Métrica               | Target      | Ferramenta           |
| --------------------- | ----------- | -------------------- |
| **Test Coverage**     | > 80%       | Vitest               |
| **TypeScript Strict** | 100%        | tsc --noEmit         |
| **Accessibility**     | WCAG 2.1 AA | Lighthouse, axe-core |
| **Browser Support**   | 95%+ users  | BrowserStack         |
| **PWA Score**         | 100         | Lighthouse PWA       |

---

## 8. CONCLUSÃO E RECOMENDAÇÕES

### 8.1. Viabilidade Geral: ✅ ALTAMENTE VIÁVEL

O projeto é **tecnicamente viável** e **recomendado** com a seguinte stack:

```
✅ Next.js 14+ (App Router)
✅ TypeScript 5+
✅ Tailwind CSS 3+
✅ Dexie.js (IndexedDB)
✅ Zustand (State Management)
✅ next-pwa (PWA Support)
```

### 8.2. Justificativa da Recomendação

1. **PWA Support:** Next.js + next-pwa oferece a melhor experiência PWA com mínimo esforço
2. **Offline-First:** IndexedDB via Dexie.js é robusto e bem suportado
3. **Performance:** Next.js otimiza automaticamente bundle, imagens, e carregamento
4. **Developer Experience:** TypeScript + Next.js é produtivo e manutenível
5. **Deployment:** Vercel oferece hosting gratuito e zero-config
6. **Comunidade:** Vasto ecossistema de bibliotecas e suporte

### 8.3. Próximos Passos

1. ✅ **Aprovação do Plano de Viabilidade**
2. → **Criar Plano de Design (UI/UX)**
3. → **Criar Plano de Ação (Implementation Roadmap)**
4. → **Iniciar Desenvolvimento do MVP**

---

**Documento preparado para:** Desenvolvimento AI-Guided  
**Aprovação necessária:** Stakeholder (Fernando)  
**Revisão técnica:** Recomendada antes do início do desenvolvimento
