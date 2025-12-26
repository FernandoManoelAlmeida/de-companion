# Plano de Execução - Detetive Existencial Companion App
## Adaptado para Execução Autônoma por IA

**Data:** 26 de Dezembro de 2025  
**Objetivo:** Desenvolvimento incremental e executável do projeto completo

---

## 🎯 ESTRATÉGIA DE EXECUÇÃO

### Princípios
1. **Desenvolvimento Incremental:** Uma feature por vez, totalmente funcional
2. **Validação Contínua:** Testar após cada fase
3. **Commits Frequentes:** Salvar progresso regularmente
4. **Priorização:** MVP primeiro, features avançadas depois

### Estrutura
- **Checkpoints:** Pontos de validação obrigatórios
- **Rollback:** Possibilidade de reverter se algo falhar
- **Documentação:** Código auto-documentado

---

## 📦 FASE 0: PREPARAÇÃO (AGORA)

### 0.1. Verificar Ambiente
```bash
# Verificar Node.js
node --version  # Deve ser >= 18

# Verificar npm
npm --version

# Verificar diretório
pwd  # Deve estar em /home/progfernando/Projetos/pessoal/de-companion
```

### 0.2. Limpar Diretório (se necessário)
```bash
# Verificar se já existe projeto
ls -la

# Se existir node_modules ou package.json, perguntar ao usuário
```

**CHECKPOINT 0:** Ambiente verificado e pronto

---

## 📦 FASE 1: SETUP INICIAL (30 min)

### 1.1. Criar Projeto Next.js
```bash
cd /home/progfernando/Projetos/pessoal/de-companion
npx create-next-app@latest . --typescript --tailwind --app --src-dir --no-git
```

**Respostas automáticas:**
- TypeScript: Yes
- ESLint: Yes
- Tailwind CSS: Yes
- `src/` directory: Yes
- App Router: Yes
- Import alias: Yes (@/*)

### 1.2. Instalar Dependências Core
```bash
npm install dexie zustand react-hook-form zod
npm install @radix-ui/react-accordion @radix-ui/react-dialog @radix-ui/react-select
npm install lucide-react
npm install -D @types/node
```

### 1.3. Configurar PWA
```bash
npm install next-pwa
```

Criar `next.config.js`:
```javascript
const withPWA = require('next-pwa')({
  dest: 'public',
  disable: process.env.NODE_ENV === 'development'
});

module.exports = withPWA({
  reactStrictMode: true,
});
```

### 1.4. Criar Estrutura de Pastas
```bash
mkdir -p src/components/ui
mkdir -p src/components/character
mkdir -p src/components/dice
mkdir -p src/components/campaign
mkdir -p src/lib
mkdir -p src/types
mkdir -p src/data
mkdir -p src/hooks
mkdir -p public/icons
```

### 1.5. Criar Arquivos Base
- `src/lib/db.ts` (Dexie setup)
- `src/types/index.ts` (TypeScript types)
- `src/lib/utils.ts` (Utility functions)
- `public/manifest.json` (PWA manifest)

**CHECKPOINT 1:** Projeto criado, dependências instaladas, estrutura pronta

---

## 📦 FASE 2: DESIGN SYSTEM (2 horas)

### 2.1. Configurar Tailwind
Atualizar `tailwind.config.ts` com cores customizadas:
```typescript
export default {
  theme: {
    extend: {
      colors: {
        noir: {
          bg: '#0a0a0a',
          accent: '#d4af37',
          // ... outras cores
        }
      }
    }
  }
}
```

### 2.2. Criar Componentes Base
Criar em ordem:
1. `src/components/ui/Button.tsx`
2. `src/components/ui/Input.tsx`
3. `src/components/ui/Card.tsx`
4. `src/components/ui/Badge.tsx`

### 2.3. Testar Design System
Criar `src/app/design-system/page.tsx` para visualizar componentes

**CHECKPOINT 2:** Design system funcionando, componentes testados

---

## 📦 FASE 3: DATA LAYER (3 horas)

### 3.1. Definir Types
Criar todos os tipos em `src/types/index.ts`:
- `Character`
- `Campaign`
- `Skill`
- `Attribute`
- `Thought`
- etc.

### 3.2. Setup Dexie (IndexedDB)
```typescript
// src/lib/db.ts
import Dexie, { Table } from 'dexie';

class DEDatabase extends Dexie {
  characters!: Table<Character>;
  campaigns!: Table<Campaign>;
  
  constructor() {
    super('DetetiveExistencialDB');
    this.version(1).stores({
      characters: 'id, name, createdAt',
      campaigns: 'id, name, createdAt'
    });
  }
}

export const db = new DEDatabase();
```

### 3.3. Criar Hooks
- `src/hooks/useCharacters.ts`
- `src/hooks/useCampaigns.ts`

### 3.4. Testar CRUD
Criar página de teste para verificar:
- Criar personagem
- Ler personagem
- Atualizar personagem
- Deletar personagem

**CHECKPOINT 3:** IndexedDB funcionando, CRUD testado

---

## 📦 FASE 4: CRIAÇÃO DE PERSONAGEM (5 horas)

### 4.1. Wizard Multi-Step
Criar componentes:
1. `src/components/character/CharacterWizard.tsx`
2. `src/components/character/StepName.tsx`
3. `src/components/character/StepAttributes.tsx`
4. `src/components/character/StepSkills.tsx`
5. `src/components/character/StepThought.tsx`
6. `src/components/character/StepReview.tsx`

### 4.2. Validação
Implementar validação com Zod:
- Atributos: soma = 8, cada 1-5
- Perícias: respeitam limite (Atributo + 1)

### 4.3. Página de Criação
Criar `src/app/characters/new/page.tsx`

### 4.4. Testar Fluxo Completo
- Criar personagem do início ao fim
- Verificar salvamento no IndexedDB
- Verificar validações

**CHECKPOINT 4:** Criação de personagem completa e funcional

---

## 📦 FASE 5: FICHA INTERATIVA (4 horas)

### 5.1. Componentes da Ficha
1. `src/components/character/CharacterSheet.tsx`
2. `src/components/character/AttributeDisplay.tsx`
3. `src/components/character/SkillList.tsx`
4. `src/components/character/ResourceBar.tsx`
5. `src/components/character/ThoughtCabinet.tsx`

### 5.2. Sistema de Rolagem
1. `src/components/dice/DiceRoller.tsx` (2d6 padrão)
2. `src/components/dice/GenericDiceRoller.tsx` (xdY)
3. `src/lib/dice.ts` (lógica de rolagem)

### 5.3. Página da Ficha
Criar `src/app/characters/[id]/page.tsx`

### 5.4. Testar
- Visualizar ficha
- Rolar dados (padrão e genérico)
- Editar recursos (Moral, Saúde)

**CHECKPOINT 5:** Ficha interativa funcional

---

## 📦 FASE 6: REPOSITÓRIO DE CONTEÚDO (3 horas)

### 6.1. Dados Estáticos
Criar arquivos em `src/data/`:
1. `skills.ts` (24 perícias)
2. `thoughts.ts` (reflexões)
3. `items.ts` (equipamentos)
4. `conditions.ts` (condições)

### 6.2. Páginas de Repositório
1. `src/app/repository/skills/page.tsx`
2. `src/app/repository/thoughts/page.tsx`
3. `src/app/repository/reference/page.tsx`

### 6.3. Busca e Filtros
Implementar busca simples

**CHECKPOINT 6:** Repositório completo e navegável

---

## 📦 FASE 7: EXPORTAÇÃO (2 horas)

### 7.1. Exportar JSON
```typescript
// src/lib/export.ts
export async function exportCharacterJSON(id: string) {
  const char = await db.characters.get(id);
  const json = JSON.stringify(char, null, 2);
  downloadFile(`${char.name}.json`, json);
}
```

### 7.2. Exportar Markdown
Template de ficha em MD

### 7.3. Importar
Validar e importar JSON

**CHECKPOINT 7:** Export/Import funcionando

---

## 📦 FASE 8: GERADOR ALEATÓRIO (3 horas)

### 8.1. Algoritmos
Implementar em `src/lib/character-generator.ts`:
- `generateRandomAttributes()`
- `generateSkills()`
- `generateBrazilianName()`

### 8.2. Componente UI
`src/components/character/RandomCharacterGenerator.tsx`

### 8.3. Testar
Gerar 10 personagens aleatórios

**CHECKPOINT 8:** Gerador funcionando

---

## 📦 FASE 9: PROGRESSÃO XP (4 horas)

### 9.1. Funções de Progressão
Implementar em `src/lib/progression.ts`:
- `calculateSkillUpgradeCost()`
- `upgradeSkill()`
- `upgradeAttribute()`
- `gainXP()`

### 9.2. UI de Progressão
`src/components/character/ProgressionPanel.tsx`

### 9.3. Testar
- Ganhar XP
- Gastar XP em perícia
- Gastar XP em atributo

**CHECKPOINT 9:** Progressão XP completa

---

## 📦 FASE 10: PWA (1 hora)

### 10.1. Manifest
Criar `public/manifest.json` completo

### 10.2. Ícones
Gerar ícones 72x72 até 512x512

### 10.3. Service Worker
Configurar cache strategies

### 10.4. Testar
- Instalar PWA
- Funcionar offline

**CHECKPOINT 10:** PWA instalável e offline

---

## 🎯 MVP COMPLETO

Após Fase 10, teremos um **MVP funcional** com:
- ✅ Criação de personagem
- ✅ Ficha interativa
- ✅ Rolagem de dados
- ✅ Repositório de conteúdo
- ✅ Export/Import
- ✅ Gerador aleatório
- ✅ Progressão XP
- ✅ PWA offline

---

## 📦 FASES AVANÇADAS (Pós-MVP)

### FASE 11: Ferramentas do Narrador (6 horas)
- Campanhas
- Modo Narrador vs Jogador
- Anotações privadas
- Revelação granular

### FASE 12: Multiplayer (1 semana)
- Firebase setup
- Convites
- Sincronização tempo real
- Chat

### FASE 13: Internacionalização (1 semana)
- next-intl setup
- Traduções pt-BR + en
- Language switcher

### FASE 14: Calculadora de Combate (1 semana)
- Modo automático
- Modo manual
- Rastreamento de iniciativa

---

## 🚀 EXECUÇÃO

### Como Executar Este Plano

**Passo 1:** Começar pela Fase 0 (Preparação)
**Passo 2:** Executar cada fase sequencialmente
**Passo 3:** Validar checkpoint antes de prosseguir
**Passo 4:** Commit após cada checkpoint
**Passo 5:** Testar continuamente

### Estimativa de Tempo (MVP)
- **Total:** ~30 horas de desenvolvimento
- **Distribuição:** 10 fases × 3h média
- **Prazo:** 1-2 semanas (desenvolvimento focado)

---

**Este plano está pronto para execução imediata.**  
**Posso começar pela Fase 0 agora mesmo?**
