# Plano de Ação - Detetive Existencial Companion App
## AI-Guided Development Roadmap

**Data:** 26 de Dezembro de 2025  
**Versão:** 1.0  
**Objetivo:** Guia passo-a-passo para IA executar desenvolvimento completo

---

## FASE 1: SETUP INICIAL (Semana 1)

### 1.1. Inicializar Projeto
```bash
npx create-next-app@latest de-companion --typescript --tailwind --app --src-dir
cd de-companion
npm install dexie zustand react-hook-form zod @radix-ui/react-accordion @radix-ui/react-dialog lucide-react
npm install -D @types/node
```

### 1.2. Configurar PWA
```bash
npm install next-pwa
```

Criar `next.config.js`, `public/manifest.json`, gerar ícones (72x72 até 512x512).

### 1.3. Estrutura de Pastas
```
src/
├── app/
│   ├── layout.tsx
│   ├── page.tsx
│   ├── characters/
│   ├── campaigns/
│   └── repository/
├── components/
│   ├── ui/
│   ├── character/
│   └── dice/
├── lib/
│   ├── db.ts (Dexie)
│   ├── schemas.ts (Zod)
│   └── utils.ts
└── types/
    └── index.ts
```

**Critério de Aceitação:** `npm run dev` funciona, PWA manifest válido.

---

## FASE 2: DATA LAYER (Semana 2)

### 2.1. Schemas TypeScript
Criar em `src/types/index.ts`:
- `Character` (independente, não vinculado a campanha)
- `Campaign` (com `narratorId`, anotações privadas)
- `CampaignCharacterLink` (N:N relationship)
- `Session`, `SessionEvent` (com `isRevealed`)
- `NarratorNote` (sempre privada até revelada)
- `ViewMode` (narrator | player)

### 2.2. IndexedDB com Dexie
```typescript
// src/lib/db.ts
import Dexie, { Table } from 'dexie';

class DEDatabase extends Dexie {
  characters!: Table<Character>;
  campaigns!: Table<Campaign>;
  characterLinks!: Table<CampaignCharacterLink>;
  
  constructor() {
    super('DetetiveExistencialDB');
    this.version(1).stores({
      characters: 'id, name, createdAt',
      campaigns: 'id, name, createdAt, narratorId',
      characterLinks: 'id, characterId, campaignId',
      sessions: 'id, campaignId, date',
      narratorNotes: 'id, campaignId, isRevealed'
    });
  }
}

export const db = new DEDatabase();
```

### 2.3. CRUD Operations
Criar hooks: `useCharacters()`, `useCampaigns()`.

**Critério de Aceitação:** Criar/ler/atualizar/deletar personagem no IndexedDB.

---

## FASE 3: DESIGN SYSTEM (Semana 3)

### 3.1. Configurar Tailwind
Adicionar cores customizadas em `tailwind.config.ts`.

### 3.2. Componentes Base
Criar em `src/components/ui/`:
- `Button.tsx` (variantes: primary, secondary, danger, ghost)
- `Input.tsx` (com label, error, helperText)
- `Card.tsx` (Header, Content, Footer)
- `Badge.tsx` (cores por atributo)

**Critério de Aceitação:** Storybook ou página de teste mostrando todos os componentes.

---

## FASE 4: CRIAÇÃO DE PERSONAGEM (Semanas 4-5)

### 4.1. Wizard Multi-Step
```
/characters/new
├── Step 1: Nome
├── Step 2: Atributos (8 pontos)
├── Step 3: Perícias (12 pontos)
├── Step 4: Reflexão Inicial
└── Step 5: Revisão
```

### 4.2. Validação
- Atributos: soma = 8, cada 1-5
- Perícias: respeitam limite (Atributo +1)
- Reflexão: escolher 1 de nível 1-2

### 4.3. Componentes
- `AttributeSelector.tsx`
- `SkillSelector.tsx`
- `ThoughtSelector.tsx`

**Critério de Aceitação:** Criar personagem completo e salvar no IndexedDB.

---

## FASE 5: FICHA INTERATIVA (Semanas 6-7)

### 5.1. Visualização
`/characters/[id]` mostra:
- Atributos (badges coloridos)
- Perícias (lista por atributo, accordion mobile)
- Recursos (Moral, Saúde, Dinheiro, XP)
- Reflexões (Gabinete)

### 5.2. Sistema de Rolagem

#### Rolagem Padrão (2d6 + modificadores)
```typescript
function rollDice(skill: number, attribute: number): {
  dice: [number, number];
  total: number;
} {
  const d1 = Math.floor(Math.random() * 6) + 1;
  const d2 = Math.floor(Math.random() * 6) + 1;
  return {
    dice: [d1, d2],
    total: d1 + d2 + skill + attribute
  };
}
```

#### Rolagem Genérica (xdY notation)
```typescript
// Parser de notação xdY
function parseRollNotation(notation: string): {
  count: number;
  sides: number;
  modifier: number;
} | null {
  // Regex: 1d20, 2d6+5, 3d8-2
  const regex = /^(\d+)d(\d+)([+-]\d+)?$/i;
  const match = notation.trim().match(regex);
  
  if (!match) return null;
  
  return {
    count: parseInt(match[1]),
    sides: parseInt(match[2]),
    modifier: match[3] ? parseInt(match[3]) : 0
  };
}

// Executar rolagem genérica
function rollGeneric(notation: string): {
  rolls: number[];
  total: number;
  notation: string;
} | null {
  const parsed = parseRollNotation(notation);
  if (!parsed) return null;
  
  const { count, sides, modifier } = parsed;
  
  // Validação
  if (count < 1 || count > 100) return null; // Max 100 dados
  if (sides < 2 || sides > 100) return null; // Max d100
  
  // Rolar dados
  const rolls: number[] = [];
  for (let i = 0; i < count; i++) {
    rolls.push(Math.floor(Math.random() * sides) + 1);
  }
  
  const sum = rolls.reduce((a, b) => a + b, 0);
  const total = sum + modifier;
  
  return { rolls, total, notation };
}
```

Componentes:
- `DiceRoller.tsx` (sistema padrão 2d6)
- `GenericDiceRoller.tsx` (notação xdY)

### 5.3. Edição Inline
Clicar em recursos permite editar (Moral, Saúde, Dinheiro).

**Critério de Aceitação:** Rolar dados, ver resultado, histórico de rolagens.

---

## FASE 6: GABINETE DE REFLEXÕES (Semana 8)

### 6.1. Componente ThoughtCabinet
- Mostrar slots (3-12)
- Reflexões processando vs internalizadas
- Adicionar/remover reflexões

### 6.2. Dados de Reflexões
Criar `src/data/thoughts.ts` com todas as reflexões (Níveis 1-4).

### 6.3. Processamento
- Status: `processing` | `internalized`
- Mostrar Problema durante processamento
- Mostrar Solução após internalizar

**Critério de Aceitação:** Adicionar reflexão, processar, internalizar, ver bônus.

---

## FASE 7: REPOSITÓRIO DE CONTEÚDO (Semana 9)

### 7.1. Páginas
- `/repository/skills` - 24 perícias com descrições
- `/repository/thoughts` - Todas reflexões
- `/repository/reference` - Tabelas (CDs, itens, armas)

### 7.2. Dados Estáticos
Criar `src/data/`:
- `skills.ts` (24 vozes)
- `thoughts.ts` (reflexões)
- `items.ts` (equipamentos)
- `conditions.ts` (condições)

### 7.3. Busca e Filtros
Filtrar reflexões por nível, perícias por atributo.

**Critério de Aceitação:** Visualizar todo conteúdo do sistema, buscar funciona.

---

## FASE 8: FERRAMENTAS DO NARRADOR (Semanas 10-11)

### 8.1. Sistema de Vinculação Personagem-Campanha
- Criar campanha (independente de personagens)
- Vincular personagens existentes à campanha
- Desvincular/marcar como inativo
- Um personagem pode estar em múltiplas campanhas

### 8.2. Modo Narrador vs Modo Jogador
Componente `ViewModeToggle.tsx`:
- Toggle entre modos
- Modo Narrador: vê tudo (anotações privadas, rolagens ocultas)
- Modo Jogador: vê apenas informações reveladas

### 8.3. Sistema de Revelação
`/campaigns/[id]` (Modo Narrador):
- Anotações privadas com botão "Revelar" individual
- Botão "Revelar Todas" para batch
- Rolagens de dados com controle de visibilidade
- Eventos de sessão com flag `isRevealed`

### 8.4. Anotações do Narrador
Componente `NarratorNotes.tsx`:
- CRUD de anotações privadas
- Tags para organização
- Revelar/ocultar individualmente
- Histórico de revelações

### 8.5. NPCs e Eventos
- CRUD de NPCs vinculados à campanha
- Gerador de eventos aleatórios (d20)

**Critério de Aceitação:** 
- Criar campanha, vincular 2+ personagens
- Alternar entre modo narrador/jogador
- Criar anotação privada, revelar para jogadores
- Rolar dados como narrador, controlar visibilidade

---

## FASE 9: EXPORTAÇÃO/IMPORTAÇÃO (Semana 12)

### 9.1. Exportar JSON
```typescript
async function exportCharacter(id: string) {
  const char = await db.characters.get(id);
  const json = JSON.stringify({
    version: '1.0',
    exportedAt: new Date(),
    type: 'character',
    data: char
  }, null, 2);
  downloadFile(`${char.name}.json`, json);
}
```

### 9.2. Exportar Markdown
Template de ficha em Markdown.

### 9.3. Importar
Validar schema com Zod, importar para IndexedDB.

**Critério de Aceitação:** Exportar personagem, importar em nova instalação.

---

## FASE 10: POLISH E TESTES (Semanas 13-14)

### 10.1. Testes Unitários
```bash
npm install -D vitest @testing-library/react
```
Testar: rolagem de dados, cálculos, validações.

### 10.2. Acessibilidade
- Lighthouse Accessibility > 90
- Navegação por teclado
- ARIA labels

### 10.3. Performance
- Lighthouse Performance > 90
- Bundle < 500KB
- PWA Score 100

### 10.4. Responsividade
Testar em: 320px, 768px, 1024px, 1440px.

**Critério de Aceitação:** Todos os testes passam, Lighthouse > 90.

---

## VERIFICAÇÃO FINAL

### Checklist de Funcionalidades
- [ ] Criar personagem (wizard completo)
- [ ] Visualizar ficha interativa
- [ ] Rolar dados (2d6 + modificadores)
- [ ] Gerenciar Gabinete de Reflexões
- [ ] Visualizar repositório de conteúdo
- [ ] Criar e gerenciar campanhas
- [ ] Adicionar NPCs e sessões
- [ ] Gerar eventos aleatórios
- [ ] Exportar/importar JSON e MD
- [ ] Funciona 100% offline
- [ ] Instalável como PWA
- [ ] Responsivo (mobile + desktop)

### Métricas de Qualidade
- [ ] Lighthouse Performance > 90
- [ ] Lighthouse Accessibility > 90
- [ ] Lighthouse PWA = 100
- [ ] Test Coverage > 80%
- [ ] TypeScript strict mode
- [ ] Zero erros de console

---

## DEPLOYMENT

### Vercel
```bash
npm install -g vercel
vercel login
vercel --prod
```

### Variáveis de Ambiente
Nenhuma necessária (app offline-first).

### Domínio Customizado (Opcional)
Configurar em Vercel dashboard.

---

## PRÓXIMOS PASSOS PÓS-MVP

1. Modo multiplayer (compartilhamento de campanha)
2. Gerador de personagens aleatórios
3. Calculadora de combate avançada
4. Temas customizáveis
5. Suporte a múltiplos idiomas
6. Integração com Discord

---

**Este plano está pronto para execução autônoma por IA.**  
**Cada fase tem critérios claros de aceitação.**  
**Estimativa total: 14 semanas (3.5 meses) para 1 desenvolvedor.**
