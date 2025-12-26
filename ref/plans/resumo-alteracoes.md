# Resumo das Alterações - Detetive Existencial Companion App

**Data:** 26 de Dezembro de 2025  
**Versão:** 1.1 (Atualizada)

---

## Alterações Implementadas

### 1. Separação Personagem/Campanha

#### Antes
- Personagens eram criados dentro de campanhas
- Relacionamento 1:N (uma campanha, vários personagens)

#### Depois ✅
- **Personagens são entidades independentes**
- **Relacionamento N:N** via `CampaignCharacterLink`
- Um personagem pode participar de múltiplas campanhas
- Campanhas podem vincular personagens existentes
- Status de ativo/inativo por campanha

#### Impacto nos Documentos
- **Plano de Viabilidade**: Novos schemas (`CampaignCharacterLink`, `ViewMode`)
- **Plano de Design**: UI para vincular personagens existentes
- **Plano de Ação**: Fase 8 expandida com sistema de vinculação

---

### 2. Visibilidade Diferenciada (Narrador vs Jogador)

#### Funcionalidades Adicionadas

**Modo Narrador** 🎭
- Vê todas as anotações privadas
- Vê rolagens ocultas de dados
- Vê eventos não revelados
- Controla o que jogadores podem ver

**Modo Jogador** 👤
- Vê apenas informações reveladas
- Vê apenas eventos públicos
- Não vê anotações privadas do narrador
- Acesso à própria ficha de personagem

#### Novos Tipos de Dados
```typescript
interface NarratorNote {
  id: string;
  content: string;
  isRevealed: boolean; // Controle de visibilidade
  revealedAt?: Date;
  tags: string[];
}

interface SessionEvent {
  id: string;
  description: string;
  isRevealed: boolean; // Narrador controla
  type: 'roll' | 'note' | 'combat' | 'discovery';
}

interface ViewMode {
  mode: 'narrator' | 'player';
  campaignId?: string;
  characterId?: string;
}
```

---

### 3. Sistema de Revelação Granular

#### Controles Implementados

**Revelação Individual**
- Botão 👁 **Revelar** em cada anotação privada
- Botão 👁 **Revelar** em cada rolagem de dados
- Botão 👁 **Revelar** em cada evento de sessão

**Revelação em Massa**
- Botão **Revelar Todas** para anotações
- Botão **Revelar Todos Eventos** para sessão

#### Interface (Modo Narrador)
```
📝 ANOTAÇÕES PRIVADAS (5)
• "Suspeito esconde arma no porão" 🔒
  [👁 Revelar]
• "Pista: sangue tipo O+" ✓ Revelado
  
[Revelar Todas] [Nova Anotação]
```

#### Interface (Modo Jogador)
```
EVENTOS REVELADOS
• 14:45 - Combate iniciado
• 14:50 - Pista descoberta: sangue O+

(Anotações privadas do narrador
 aparecerão aqui quando reveladas)
```

---

## Arquivos Atualizados

### ✅ plano-de-viabilidade.md
- Schemas atualizados com `CampaignCharacterLink`
- Adicionado `NarratorNote` com `isRevealed`
- Adicionado `SessionEvent` com controle de visibilidade
- Adicionado `ViewMode` para alternar entre modos
- IndexedDB atualizado com novas tabelas

### ✅ plano-de-design.md
- Wireframes para Modo Narrador
- Wireframes para Modo Jogador
- UI de vinculação de personagens
- Controles de revelação (botões individuais e em massa)
- Toggle de modo (🎭 Narrador ↔️ 👤 Jogador)

### ✅ plano-de-acao.md
- Fase 8 expandida com sistema de vinculação
- Implementação de `ViewModeToggle.tsx`
- Implementação de `NarratorNotes.tsx`
- Sistema de revelação granular
- Critérios de aceitação atualizados

---

## Fluxo de Uso Completo

### Criação de Personagem (Independente)
1. Jogador acessa `/characters/new`
2. Cria personagem completo
3. Personagem salvo no IndexedDB (sem vínculo a campanha)

### Criação de Campanha (Narrador)
1. Narrador acessa `/campaigns/new`
2. Cria campanha com nome e descrição
3. Campanha salva (sem personagens ainda)

### Vinculação de Personagens
1. Narrador acessa `/campaigns/[id]`
2. Clica **[+ Vincular Personagem Existente]**
3. Seleciona personagens da lista global
4. Personagens vinculados via `CampaignCharacterLink`

### Durante a Sessão (Modo Narrador)
1. Narrador faz anotação privada: "Suspeito é o mordomo"
2. Anotação salva com `isRevealed: false`
3. Jogadores não veem
4. Narrador clica **[👁 Revelar]** quando apropriado
5. `isRevealed` vira `true`, `revealedAt` registrado
6. Jogadores veem a informação

### Visualização (Modo Jogador)
1. Jogador acessa `/campaigns/[id]`
2. Vê apenas eventos com `isRevealed: true`
3. Vê apenas anotações reveladas
4. Pode acessar ficha do próprio personagem

---

### 4. Sistema de Rolagem Genérica (xdY)

#### Funcionalidade Adicionada

**Rolagem com Notação xdY**
- Parser de notação: `1d20`, `2d6`, `3d8+5`, `4d10-3`
- Validação de entrada (1-100 dados, d2-d100)
- Suporte a modificadores positivos e negativos
- Histórico separado de rolagens genéricas

#### Exemplos de Uso
```
1d20      → Rola 1 dado de 20 faces
2d6       → Rola 2 dados de 6 faces
3d8+5     → Rola 3 dados de 8 faces e soma 5
4d10-3    → Rola 4 dados de 10 faces e subtrai 3
```

#### Componente
```typescript
<GenericDiceRoller onRoll={(result) => {
  console.log(result.rolls);    // [15]
  console.log(result.total);    // 15
  console.log(result.notation); // "1d20"
}} />
```

#### Casos de Uso
- Rolagens de dano em combate
- Eventos aleatórios (d20)
- Testes customizados do narrador
- Qualquer situação que não use o sistema padrão 2d6

---

## Próximos Passos

1. ✅ Documentos atualizados
2. ✅ Sistema de rolagem genérica especificado
3. → Iniciar implementação (se aprovado)
4. → Criar mockups em Figma (opcional)
5. → Setup do projeto Next.js

---

**Todas as alterações solicitadas foram implementadas nos documentos estratégicos.**  
**O sistema agora suporta:**
- ✅ Personagens independentes de campanhas
- ✅ Vinculação N:N personagem-campanha
- ✅ Modo Narrador vs Modo Jogador
- ✅ Revelação granular de informações
