# Plano de Execução Mestre: Detetive Existencial Companion App

**Versão:** 1.1 (Consolidada)  
**Data:** 26 de Dezembro de 2025  
**Objetivo:** Fonte única de verdade para implementação completa e autônoma por IA.

---

## 🛠️ 1. FUNDAÇÃO TÉCNICA

### 1.1. Stack de Tecnologia
- **Framework:** Next.js 14+ (App Router)
- **Linguagem:** TypeScript 5+
- **Estilização:** Tailwind CSS + Radix UI
- **Banco Local:** IndexedDB via Dexie.js
- **Estado:** Zustand + React Hook Form + Zod
- **Multiplayer:** Firebase (Realtime DB + Auth)
- **i18n:** next-intl + DeepL API
- **PWA:** next-pwa (Workbox)

### 1.2. Padrões de Código
- **Arquitetura:** Atomic Design (atoms, molecules, organisms, templates)
- **Tipagem:** Interfaces TypeScript obrigatórias para todo dado persistido
- **Offline-First:** Sincronização Firebase <-> IndexedDB com prioridade local

---

## 💾 2. ARQUITETURA DE DADOS (SCHEMAS)

### 2.1. Interfaces TypeScript (`src/types/index.ts`)
```typescript
// Personagem Independente
export interface Character {
  id: string; // UUID
  name: string;
  archetype?: string;
  attributes: Attributes;
  skills: Record<string, number>;
  resources: {
    morale: number;
    moraleMax: number;
    health: number;
    healthMax: number;
    money: number;
    xp: number;
    xpTotal: number;
  };
  thoughtCabinet: {
    slots: number;
    thoughts: Thought[];
  };
  inventory: Item[];
  conditions: string[];
  progression: {
    xpHistory: XPTransaction[];
    upgrades: UpgradeLog[];
  };
  createdAt: Date;
  updatedAt: Date;
}

// Campanha (Multiplayer/Narrador)
export interface Campaign {
  id: string;
  name: string;
  description: string;
  narratorId: string;
  mode: 'local' | 'multiplayer';
  inviteCode?: string;
  cityHealth: { morale: number; health: number };
  sessions: Session[];
  narratorNotes: Note[];
  npcs: NPC[];
}

// Vínculo N:N entre Personagem e Campanha
export interface CampaignCharacterLink {
  id: string;
  characterId: string;
  campaignId: string;
  role: 'player' | 'narrator';
  isActive: boolean;
}
```

### 2.2. Banco de Dados (`src/lib/db.ts`)
```typescript
class DEDatabase extends Dexie {
  characters!: Table<Character>;
  campaigns!: Table<Campaign>;
  characterLinks!: Table<CampaignCharacterLink>;
  translations!: Table<{ key: string; text: string; lang: string }>;

  constructor() {
    super('DetetiveExistencialDB');
    this.version(1).stores({
      characters: 'id, name, createdAt',
      campaigns: 'id, name, narratorId, mode',
      characterLinks: 'id, characterId, campaignId, role',
      translations: 'key, lang'
    });
  }
}
```

---

## 🧮 3. ALGORITMOS CORE

### 3.1. Sistema de Dados (xdY Parser)
```typescript
function parseRoll(notation: string) {
  const regex = /^(\d+)d(\d+)([+-]\d+)?$/i;
  const match = notation.match(regex);
  if (!match) return null;
  const [_, count, sides, mod] = match;
  return { count: +count, sides: +sides, mod: mod ? +mod : 0 };
}
```

### 3.2. Progressão XP (Fórmulas)
- **Custo Perícia:** `currentLevel * 2` XP.
- **Custo Atributo:** `currentLevel * 10` XP.
- **Custo Slot Reflexão:** `currentSlots * 5` XP.
- **Limite Perícia:** `ParentAttribute + 1`.

### 3.3. Gerador Aleatório (Arquétipos)
- **Detetive:** INT 4, PSY 2, FYS 2, MOT 2.
- **Emocional:** INT 1, PSY 5, FYS 2, MOT 2.
- **Brutamontes:** INT 1, PSY 2, FYS 5, MOT 2.
- **Veloz:** INT 2, PSY 1, FYS 3, MOT 4.

---

## 🎨 4. DESIGN SYSTEM (NOIR EXISTENCIAL)

### 4.1. Tokens de Cores
- **Fundo:** `#0a0a0a` (Noir Base)
- **Acento:** `#d4af37` (Âmbar Existencial)
- **Perigo:** `#c44536` (Vermelho Físico)
- **Psique:** `#9b59b6` (Roxo Moral)
- **Texto:** `#e8e8e8` (Branco Sujo)

### 4.2. Tipografia
- **Títulos:** `Playfair Display` (Serifada)
- **Corpo:** `Inter` (Sans-Serif)

---

## 🚀 5. ROADMAP DE EXECUÇÃO (15 FASES)

### FASE 1: Setup & PWA
- Inicializar Next.js 14+, configurar `next-pwa` e manifest.
- **Bash:** `npx create-next-app@14 . --typescript --tailwind --app --src-dir`

### FASE 2: Database & Schemas
- Implementar `src/lib/db.ts` com Dexie e as interfaces TypeScript completas.

### FASE 3: Design System & UI Base
- Configurar `tailwind.config.ts`. Criar `Button`, `Input`, `Card` e `Badge`.

### FASE 4: Criação de Personagem (Wizard)
- Formulário multi-step com validação Zod. Distribuição de 8 pts em Atributos e 12 pts em Perícias.

### FASE 5: Ficha Interativa & Dados
- Tela `/characters/[id]`. Implementar `DiceRoller` (2d6 e xdY).

### FASE 6: Gabinete de Reflexões
- Gerenciamento de slots (3-12). Lógica de processamento e bônus/penalidades.

### FASE 7: Repositório de Conteúdo
- Biblioteca estática: 24 Vozes (Perícias), Itens, Reflexões Nível 1-4.

### FASE 8: Sistema de Progressão XP
- Fórmulas de custo, histórico de upgrades e coupling Atributo-Perícia.

### FASE 9: Gerador Aleatório
- Algoritmo de nomes brasileiros e builds baseadas em arquétipos.

### FASE 10: Ferramentas do Narrador
- Cadastro de Campanhas, NPCs e Controle de Revelação de Informação.

### FASE 11: Modo Multiplayer (Firebase)
- Integração Realtime DB, convites (códigos), presença e chat de mesa.

### FASE 12: Internacionalização (i18n)
- `next-intl` (pt-BR e en nativos) + Tradução dinâmica via DeepL/Gemini.

### FASE 13: Calculadora de Combate
- Modo Automático (cálculos de regra) e Modo Manual (Toggle Narrador).

### FASE 14: Polish & PWA Offline
- Cache de assets pesados, Service Worker robusto e animações Framer Motion.

### FASE 15: Exportação & Lançamento
- Exportar JSON/Markdown. Deploy final na Vercel.

---

## ✅ CHECKPOINT FINAL DE VALIDAÇÃO
- [ ] Personagem criado com 8/12 pts?
- [ ] Rolar "2d6+5" funciona?
- [ ] Upgrade de Atributo aumenta 6 Perícias?
- [ ] Narrador revelou anotação privada?
- [ ] Offline funciona (Service Worker)?

---
**Execução:** Siga cada fase sequencialmente, realizando commits após cada checkpoint.
