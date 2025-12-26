# Plano de Implementação: Sistema de Progressão (XP)

**Data:** 26 de Dezembro de 2025  
**Objetivo:** Sistema completo de ganho e gasto de XP com cálculos automáticos

---

## 1. REGRAS DE PROGRESSÃO (Seção 9 do Sistema)

### 1.1. Ganho de XP
**Fontes de XP:**
- **Testes Bem-Sucedidos:** 1 XP por teste importante
- **Reflexões Internalizadas:** 2-5 XP (depende do nível)
- **Marcos Narrativos:** 3-10 XP (decisão do narrador)
- **Fim de Sessão:** 2-5 XP por jogador

### 1.2. Gasto de XP
**Custos (Seção 9.2):**
```typescript
const XP_COSTS = {
  skill: {
    // Custo para aumentar perícia
    formula: (currentLevel: number) => currentLevel * 2,
    // Exemplo: Nível 3 → 4 custa 6 XP (3 * 2)
  },
  
  attribute: {
    // Custo para aumentar atributo
    formula: (currentLevel: number) => currentLevel * 10,
    // Exemplo: INT 3 → 4 custa 30 XP (3 * 10)
  },
  
  thoughtSlot: {
    // Custo para novo slot no Gabinete
    formula: (currentSlots: number) => currentSlots * 5,
    // Exemplo: 3 → 4 slots custa 15 XP (3 * 5)
  }
};
```

### 1.3. Limites e Validações
- **Perícia:** Não pode exceder Atributo Pai + 1
- **Atributo:** Máximo 6 (sem bônus especiais)
- **Slots de Reflexão:** Máximo 12

---

## 2. IMPLEMENTAÇÃO TÉCNICA

### 2.1. Estrutura de Dados

```typescript
interface Character {
  // ... outros campos
  
  resources: {
    xp: number; // XP atual disponível
    xpTotal: number; // XP total ganho na vida
  };
  
  progression: {
    xpHistory: XPTransaction[];
    skillUpgrades: SkillUpgrade[];
    attributeUpgrades: AttributeUpgrade[];
  };
}

interface XPTransaction {
  id: string;
  type: 'gain' | 'spend';
  amount: number;
  source: string; // "Teste de Percepção", "Reflexão: Lógica Fuzzy", etc.
  timestamp: Date;
  sessionId?: string;
}

interface SkillUpgrade {
  id: string;
  skillName: string;
  fromLevel: number;
  toLevel: number;
  xpCost: number;
  timestamp: Date;
}

interface AttributeUpgrade {
  id: string;
  attributeName: 'intellect' | 'psyche' | 'physique' | 'motorics';
  fromLevel: number;
  toLevel: number;
  xpCost: number;
  timestamp: Date;
  // Quando atributo sobe, perícias relacionadas também sobem
  affectedSkills: string[];
}
```

### 2.2. Funções de Cálculo

```typescript
// lib/progression.ts

/**
 * Calcular custo de XP para aumentar perícia
 */
export function calculateSkillUpgradeCost(currentLevel: number): number {
  return currentLevel * 2;
}

/**
 * Calcular custo de XP para aumentar atributo
 */
export function calculateAttributeUpgradeCost(currentLevel: number): number {
  return currentLevel * 10;
}

/**
 * Calcular custo de XP para novo slot de reflexão
 */
export function calculateThoughtSlotCost(currentSlots: number): number {
  return currentSlots * 5;
}

/**
 * Validar se pode aumentar perícia
 */
export function canUpgradeSkill(
  character: Character,
  skillName: string
): { valid: boolean; reason?: string } {
  const skill = character.skills[skillName];
  const parentAttribute = getParentAttribute(skillName);
  const attributeLevel = character.attributes[parentAttribute];
  
  // Verificar limite (Atributo + 1)
  if (skill >= attributeLevel + 1) {
    return {
      valid: false,
      reason: `Perícia não pode exceder ${parentAttribute.toUpperCase()} + 1 (máximo ${attributeLevel + 1})`
    };
  }
  
  // Verificar XP disponível
  const cost = calculateSkillUpgradeCost(skill);
  if (character.resources.xp < cost) {
    return {
      valid: false,
      reason: `XP insuficiente. Necessário: ${cost}, Disponível: ${character.resources.xp}`
    };
  }
  
  return { valid: true };
}

/**
 * Validar se pode aumentar atributo
 */
export function canUpgradeAttribute(
  character: Character,
  attributeName: string
): { valid: boolean; reason?: string } {
  const currentLevel = character.attributes[attributeName];
  
  // Verificar limite máximo
  if (currentLevel >= 6) {
    return {
      valid: false,
      reason: 'Atributo já está no máximo (6)'
    };
  }
  
  // Verificar XP disponível
  const cost = calculateAttributeUpgradeCost(currentLevel);
  if (character.resources.xp < cost) {
    return {
      valid: false,
      reason: `XP insuficiente. Necessário: ${cost}, Disponível: ${character.resources.xp}`
    };
  }
  
  return { valid: true };
}

/**
 * Aumentar perícia (com XP)
 */
export async function upgradeSkill(
  characterId: string,
  skillName: string
): Promise<void> {
  const character = await db.characters.get(characterId);
  if (!character) throw new Error('Personagem não encontrado');
  
  // Validar
  const validation = canUpgradeSkill(character, skillName);
  if (!validation.valid) {
    throw new Error(validation.reason);
  }
  
  const currentLevel = character.skills[skillName];
  const cost = calculateSkillUpgradeCost(currentLevel);
  
  // Atualizar personagem
  await db.characters.update(characterId, {
    skills: {
      ...character.skills,
      [skillName]: currentLevel + 1
    },
    resources: {
      ...character.resources,
      xp: character.resources.xp - cost
    },
    progression: {
      ...character.progression,
      xpHistory: [
        ...character.progression.xpHistory,
        {
          id: generateUUID(),
          type: 'spend',
          amount: cost,
          source: `Upgrade: ${skillName} ${currentLevel} → ${currentLevel + 1}`,
          timestamp: new Date()
        }
      ],
      skillUpgrades: [
        ...character.progression.skillUpgrades,
        {
          id: generateUUID(),
          skillName,
          fromLevel: currentLevel,
          toLevel: currentLevel + 1,
          xpCost: cost,
          timestamp: new Date()
        }
      ]
    }
  });
  
  // Verificar se perícia estava com Sobrecarga Temporal
  if (currentLevel === character.attributes[getParentAttribute(skillName)] + 1) {
    // Remover penalidade de sobrecarga (-4)
    console.log(`Sobrecarga Temporal removida de ${skillName}`);
  }
}

/**
 * Aumentar atributo (com XP)
 */
export async function upgradeAttribute(
  characterId: string,
  attributeName: string
): Promise<void> {
  const character = await db.characters.get(characterId);
  if (!character) throw new Error('Personagem não encontrado');
  
  // Validar
  const validation = canUpgradeAttribute(character, attributeName);
  if (!validation.valid) {
    throw new Error(validation.reason);
  }
  
  const currentLevel = character.attributes[attributeName];
  const cost = calculateAttributeUpgradeCost(currentLevel);
  
  // Perícias afetadas (todas do mesmo atributo)
  const affectedSkills = getSkillsByAttribute(attributeName);
  
  // Atualizar personagem
  const newSkills = { ...character.skills };
  affectedSkills.forEach(skill => {
    // Perícias sobem automaticamente com o atributo
    newSkills[skill] = character.skills[skill] + 1;
  });
  
  await db.characters.update(characterId, {
    attributes: {
      ...character.attributes,
      [attributeName]: currentLevel + 1
    },
    skills: newSkills,
    resources: {
      ...character.resources,
      xp: character.resources.xp - cost,
      // Moral e Saúde podem aumentar se Vontade/Endurance subiram
      morale: attributeName === 'psyche' 
        ? 1 + newSkills.volition 
        : character.resources.morale,
      health: attributeName === 'physique'
        ? 1 + newSkills.endurance
        : character.resources.health
    },
    progression: {
      ...character.progression,
      xpHistory: [
        ...character.progression.xpHistory,
        {
          id: generateUUID(),
          type: 'spend',
          amount: cost,
          source: `Upgrade: ${attributeName.toUpperCase()} ${currentLevel} → ${currentLevel + 1}`,
          timestamp: new Date()
        }
      ],
      attributeUpgrades: [
        ...character.progression.attributeUpgrades,
        {
          id: generateUUID(),
          attributeName,
          fromLevel: currentLevel,
          toLevel: currentLevel + 1,
          xpCost: cost,
          timestamp: new Date(),
          affectedSkills
        }
      ]
    }
  });
}

/**
 * Ganhar XP
 */
export async function gainXP(
  characterId: string,
  amount: number,
  source: string,
  sessionId?: string
): Promise<void> {
  const character = await db.characters.get(characterId);
  if (!character) throw new Error('Personagem não encontrado');
  
  await db.characters.update(characterId, {
    resources: {
      ...character.resources,
      xp: character.resources.xp + amount,
      xpTotal: character.resources.xpTotal + amount
    },
    progression: {
      ...character.progression,
      xpHistory: [
        ...character.progression.xpHistory,
        {
          id: generateUUID(),
          type: 'gain',
          amount,
          source,
          timestamp: new Date(),
          sessionId
        }
      ]
    }
  });
}

/**
 * Mapear perícia → atributo pai
 */
function getParentAttribute(skillName: string): keyof Attributes {
  const skillMap: Record<string, keyof Attributes> = {
    // INTELECTO
    logic: 'intellect',
    encyclopedia: 'intellect',
    rhetoric: 'intellect',
    conceptualization: 'intellect',
    visualCalculus: 'intellect',
    drama: 'intellect',
    // PSIQUE
    volition: 'psyche',
    inlandEmpire: 'psyche',
    empathy: 'psyche',
    authority: 'psyche',
    espritDeCorps: 'psyche',
    suggestion: 'psyche',
    // FÍSICO
    endurance: 'physique',
    painThreshold: 'physique',
    physicalInstrument: 'physique',
    electrochemistry: 'physique',
    shivers: 'physique',
    halfLight: 'physique',
    // MOTRICIDADE
    handEyeCoordination: 'motorics',
    perception: 'motorics',
    reactionSpeed: 'motorics',
    savoirFaire: 'motorics',
    interfacing: 'motorics',
    composure: 'motorics'
  };
  
  return skillMap[skillName];
}

/**
 * Obter perícias de um atributo
 */
function getSkillsByAttribute(attributeName: string): string[] {
  const attributeSkills: Record<string, string[]> = {
    intellect: ['logic', 'encyclopedia', 'rhetoric', 'conceptualization', 'visualCalculus', 'drama'],
    psyche: ['volition', 'inlandEmpire', 'empathy', 'authority', 'espritDeCorps', 'suggestion'],
    physique: ['endurance', 'painThreshold', 'physicalInstrument', 'electrochemistry', 'shivers', 'halfLight'],
    motorics: ['handEyeCoordination', 'perception', 'reactionSpeed', 'savoirFaire', 'interfacing', 'composure']
  };
  
  return attributeSkills[attributeName] || [];
}
```

---

## 3. INTERFACE DO USUÁRIO

### 3.1. Painel de Progressão

```tsx
// components/ProgressionPanel.tsx
'use client';
import { useState } from 'react';
import { useCharacter } from '@/hooks/useCharacter';
import { 
  calculateSkillUpgradeCost, 
  calculateAttributeUpgradeCost,
  canUpgradeSkill,
  canUpgradeAttribute,
  upgradeSkill,
  upgradeAttribute
} from '@/lib/progression';

export function ProgressionPanel({ characterId }: { characterId: string }) {
  const character = useCharacter(characterId);
  const [selectedTab, setSelectedTab] = useState<'skills' | 'attributes'>('skills');

  if (!character) return null;

  return (
    <div className="progression-panel">
      <div className="xp-display">
        <h3>Experiência</h3>
        <div className="xp-current">
          <span className="xp-amount">{character.resources.xp}</span>
          <span className="xp-label">XP Disponível</span>
        </div>
        <div className="xp-total">
          Total ganho: {character.resources.xpTotal} XP
        </div>
      </div>

      <div className="tabs">
        <button 
          className={selectedTab === 'skills' ? 'active' : ''}
          onClick={() => setSelectedTab('skills')}
        >
          Perícias
        </button>
        <button 
          className={selectedTab === 'attributes' ? 'active' : ''}
          onClick={() => setSelectedTab('attributes')}
        >
          Atributos
        </button>
      </div>

      {selectedTab === 'skills' && (
        <SkillUpgradeList character={character} />
      )}

      {selectedTab === 'attributes' && (
        <AttributeUpgradeList character={character} />
      )}
    </div>
  );
}

function SkillUpgradeList({ character }: { character: Character }) {
  const handleUpgrade = async (skillName: string) => {
    try {
      await upgradeSkill(character.id, skillName);
    } catch (error) {
      alert(error.message);
    }
  };

  return (
    <div className="skill-upgrade-list">
      {Object.entries(character.skills).map(([skillName, level]) => {
        const cost = calculateSkillUpgradeCost(level);
        const validation = canUpgradeSkill(character, skillName);
        const parentAttr = getParentAttribute(skillName);
        const limit = character.attributes[parentAttr] + 1;

        return (
          <div key={skillName} className="skill-upgrade-item">
            <div className="skill-info">
              <span className="skill-name">{skillName}</span>
              <span className="skill-level">Nível {level}</span>
              <span className="skill-limit">Limite: {limit}</span>
            </div>

            <div className="upgrade-action">
              <span className="xp-cost">{cost} XP</span>
              <button
                onClick={() => handleUpgrade(skillName)}
                disabled={!validation.valid}
                title={validation.reason}
              >
                ↑ Aumentar
              </button>
            </div>

            {!validation.valid && (
              <div className="validation-error">{validation.reason}</div>
            )}
          </div>
        );
      })}
    </div>
  );
}

function AttributeUpgradeList({ character }: { character: Character }) {
  const handleUpgrade = async (attrName: string) => {
    try {
      await upgradeAttribute(character.id, attrName);
    } catch (error) {
      alert(error.message);
    }
  };

  const attributes = [
    { name: 'intellect', label: 'Intelecto', icon: '🧠' },
    { name: 'psyche', label: 'Psique', icon: '💭' },
    { name: 'physique', label: 'Físico', icon: '💪' },
    { name: 'motorics', label: 'Motricidade', icon: '⚡' }
  ];

  return (
    <div className="attribute-upgrade-list">
      {attributes.map(({ name, label, icon }) => {
        const level = character.attributes[name];
        const cost = calculateAttributeUpgradeCost(level);
        const validation = canUpgradeAttribute(character, name);
        const affectedSkills = getSkillsByAttribute(name);

        return (
          <div key={name} className="attribute-upgrade-item">
            <div className="attribute-info">
              <span className="attribute-icon">{icon}</span>
              <span className="attribute-name">{label}</span>
              <span className="attribute-level">Nível {level}</span>
            </div>

            <div className="upgrade-action">
              <span className="xp-cost">{cost} XP</span>
              <button
                onClick={() => handleUpgrade(name)}
                disabled={!validation.valid}
                title={validation.reason}
              >
                ↑ Aumentar
              </button>
            </div>

            <div className="affected-skills">
              <small>
                Aumenta: {affectedSkills.join(', ')}
              </small>
            </div>

            {!validation.valid && (
              <div className="validation-error">{validation.reason}</div>
            )}
          </div>
        );
      })}
    </div>
  );
}
```

### 3.2. Histórico de Progressão

```tsx
// components/ProgressionHistory.tsx
export function ProgressionHistory({ character }: { character: Character }) {
  const history = character.progression.xpHistory.sort(
    (a, b) => b.timestamp.getTime() - a.timestamp.getTime()
  );

  return (
    <div className="progression-history">
      <h3>Histórico de XP</h3>
      
      <div className="history-list">
        {history.map((transaction) => (
          <div 
            key={transaction.id} 
            className={`history-item ${transaction.type}`}
          >
            <div className="transaction-type">
              {transaction.type === 'gain' ? '+ ' : '- '}
              {transaction.amount} XP
            </div>
            <div className="transaction-source">
              {transaction.source}
            </div>
            <div className="transaction-date">
              {formatDate(transaction.timestamp)}
            </div>
          </div>
        ))}
      </div>
    </div>
  );
}
```

### 3.3. Ganhar XP (Narrador)

```tsx
// components/AwardXP.tsx (Modo Narrador)
export function AwardXP({ campaignId }: { campaignId: string }) {
  const [amount, setAmount] = useState(1);
  const [source, setSource] = useState('');
  const [selectedCharacters, setSelectedCharacters] = useState<string[]>([]);

  const handleAward = async () => {
    for (const charId of selectedCharacters) {
      await gainXP(charId, amount, source);
    }
    
    // Notificar jogadores (se multiplayer)
    if (isMultiplayer) {
      notifyPlayers(`Você ganhou ${amount} XP: ${source}`);
    }
  };

  return (
    <div className="award-xp">
      <h3>Conceder XP</h3>
      
      <label>
        Quantidade:
        <input 
          type="number" 
          value={amount} 
          onChange={(e) => setAmount(parseInt(e.target.value))}
          min={1}
        />
      </label>
      
      <label>
        Motivo:
        <input 
          value={source} 
          onChange={(e) => setSource(e.target.value)}
          placeholder="Ex: Teste de Percepção bem-sucedido"
        />
      </label>
      
      <div className="character-selection">
        {/* Lista de personagens para selecionar */}
      </div>
      
      <button onClick={handleAward}>
        Conceder XP
      </button>
    </div>
  );
}
```

---

## 4. WIREFRAMES

### Painel de Progressão
```
┌────────────────────────────────────────┐
│ PROGRESSÃO                             │
├────────────────────────────────────────┤
│ Experiência                            │
│ ┌────────────────────────────────────┐ │
│ │        45 XP                       │ │
│ │    XP Disponível                   │ │
│ │ Total ganho: 127 XP                │ │
│ └────────────────────────────────────┘ │
├────────────────────────────────────────┤
│ [Perícias] [Atributos]                 │
├────────────────────────────────────────┤
│ Lógica                    Nível 4      │
│ Limite: 5 (INT 4 + 1)                  │
│ 8 XP [↑ Aumentar]                      │
│                                        │
│ Percepção                 Nível 5      │
│ Limite: 5 (MOT 4 + 1)                  │
│ ⚠ Limite atingido                      │
│                                        │
│ Vontade                   Nível 3      │
│ Limite: 3 (PSY 2 + 1)                  │
│ 6 XP [↑ Aumentar]                      │
└────────────────────────────────────────┘
```

---

## 5. NOTIFICAÇÕES E FEEDBACK

### 5.1. Notificação de XP Ganho
```tsx
// Quando jogador ganha XP
showNotification({
  type: 'success',
  title: 'XP Ganho!',
  message: `+${amount} XP: ${source}`,
  duration: 3000
});
```

### 5.2. Animação de Level Up
```tsx
// Quando perícia/atributo sobe
showAnimation({
  type: 'levelUp',
  target: skillName,
  from: oldLevel,
  to: newLevel
});
```

---

## 6. INTEGRAÇÃO COM OUTRAS FEATURES

### 6.1. Testes de Perícia
Após teste bem-sucedido importante:
```typescript
// Após rolagem bem-sucedida
if (isImportantTest && success) {
  await gainXP(characterId, 1, `Teste de ${skillName} bem-sucedido`);
}
```

### 6.2. Reflexões
Ao internalizar reflexão:
```typescript
const xpReward = thought.level * 2; // Nível 1 = 2 XP, Nível 2 = 4 XP, etc.
await gainXP(characterId, xpReward, `Reflexão internalizada: ${thought.name}`);
```

### 6.3. Fim de Sessão
```typescript
// Narrador concede XP no fim da sessão
await gainXP(characterId, 5, 'Fim de sessão #12');
```

---

## 7. CRITÉRIOS DE ACEITAÇÃO

- [ ] Calcular custo de XP corretamente (perícia, atributo)
- [ ] Validar limites (perícia ≤ atributo + 1, atributo ≤ 6)
- [ ] Validar XP disponível antes de gastar
- [ ] Aumentar perícia com XP
- [ ] Aumentar atributo com XP (perícias sobem automaticamente)
- [ ] Atualizar Moral/Saúde se Vontade/Endurance subiram
- [ ] Ganhar XP de múltiplas fontes
- [ ] Histórico completo de transações
- [ ] Interface clara e intuitiva
- [ ] Notificações de ganho/gasto de XP

---

**Este plano está pronto para implementação.**  
**Estimativa:** 2 semanas de desenvolvimento
