# Plano de Implementação: Gerador de Personagens Aleatórios

**Data:** 26 de Dezembro de 2025  
**Objetivo:** Criar personagens completos aleatoriamente com base em arquétipos do sistema

---

## 1. VISÃO GERAL

### 1.1. Funcionalidade
Gerar personagens completos e balanceados aleatoriamente, seguindo as regras do sistema:
- 8 pontos em atributos (1-5 cada)
- 12 pontos em perícias (respeitando limites)
- 1 Reflexão inicial (Nível 1-2)
- Nome brasileiro aleatório
- Recursos calculados automaticamente

### 1.2. Casos de Uso
- Jogadores que querem começar rápido
- Narradores criando NPCs
- Testes e demonstrações
- Inspiração para criação manual

---

## 2. ARQUÉTIPOS PRÉ-DEFINIDOS

### 2.1. Builds Recomendadas (do sistema)

```typescript
const ARCHETYPES = {
  detective: {
    name: 'Detetive Clássico',
    attributes: { intellect: 4, psyche: 2, physique: 2, motorics: 2 },
    skillWeights: {
      // INTELECTO (prioridade alta)
      logic: 1.5,
      encyclopedia: 1.5,
      rhetoric: 1.2,
      visualCalculus: 1.5,
      // PSIQUE
      volition: 1.2,
      empathy: 1.3,
      // MOTRICIDADE
      perception: 1.4,
      reactionSpeed: 1.2
    },
    thoughtPool: ['Advogado de Si Mesmo', 'Lente de Homem Morto']
  },
  
  emotional: {
    name: 'Trem-Bala Emocional',
    attributes: { intellect: 1, psyche: 5, physique: 2, motorics: 2 },
    skillWeights: {
      // PSIQUE (prioridade alta)
      volition: 1.8,
      inlandEmpire: 1.8,
      empathy: 1.8,
      suggestion: 1.5,
      // FÍSICO
      electrochemistry: 1.3,
      // MOTRICIDADE
      reactionSpeed: 1.4
    },
    thoughtPool: ['Camarada de Copas', 'O Apodrecimento']
  },
  
  brute: {
    name: 'Brutamontes com Sentimentos',
    attributes: { intellect: 1, psyche: 2, physique: 5, motorics: 2 },
    skillWeights: {
      // FÍSICO (prioridade alta)
      endurance: 1.8,
      painThreshold: 1.8,
      physicalInstrument: 1.8,
      electrochemistry: 1.5,
      shivers: 1.8,
      halfLight: 1.4,
      // PSIQUE
      empathy: 1.3
    },
    thoughtPool: ['Coxinha do Bar', 'Foda-se o Mundo']
  },
  
  fast: {
    name: 'Veloz e Perigoso',
    attributes: { intellect: 2, psyche: 1, physique: 3, motorics: 4 },
    skillWeights: {
      // MOTRICIDADE (prioridade alta)
      handEyeCoordination: 1.8,
      reactionSpeed: 1.8,
      savoirFaire: 1.8,
      interfacing: 1.8,
      // FÍSICO
      endurance: 1.4,
      physicalInstrument: 1.4,
      halfLight: 1.4
    },
    thoughtPool: ['Gato Preto', 'Lutador de Rua']
  }
};
```

### 2.2. Arquétipo Aleatório Puro

```typescript
const RANDOM_ARCHETYPE = {
  name: 'Aleatório',
  attributes: null, // Distribuir 8 pontos aleatoriamente
  skillWeights: null, // Sem pesos, distribuição uniforme
  thoughtPool: null // Qualquer reflexão Nível 1-2
};
```

---

## 3. ALGORITMOS DE GERAÇÃO

### 3.1. Distribuição de Atributos

#### Opção 1: Baseada em Arquétipo
```typescript
function generateAttributesFromArchetype(archetype: Archetype): Attributes {
  return archetype.attributes;
}
```

#### Opção 2: Aleatória Balanceada
```typescript
function generateRandomAttributes(): Attributes {
  const attributes = {
    intellect: 1,
    psyche: 1,
    physique: 1,
    motorics: 1
  };
  
  let pointsRemaining = 4; // 8 total - 4 já distribuídos
  
  while (pointsRemaining > 0) {
    // Escolher atributo aleatório
    const attr = randomChoice(['intellect', 'psyche', 'physique', 'motorics']);
    
    // Verificar se não excede máximo (5)
    if (attributes[attr] < 5) {
      attributes[attr]++;
      pointsRemaining--;
    }
  }
  
  return attributes;
}
```

### 3.2. Distribuição de Perícias

```typescript
function generateSkills(
  attributes: Attributes,
  archetype: Archetype | null
): Skills {
  const skills = initializeSkillsFromAttributes(attributes);
  let pointsRemaining = 12;
  
  // Criar pool de perícias disponíveis
  const skillPool = Object.keys(skills).map(skill => ({
    name: skill,
    weight: archetype?.skillWeights?.[skill] || 1.0,
    limit: getSkillLimit(skill, attributes)
  }));
  
  while (pointsRemaining > 0) {
    // Escolher perícia com base em pesos
    const skill = weightedRandomChoice(skillPool);
    
    // Verificar se pode aumentar
    if (skills[skill.name] < skill.limit) {
      skills[skill.name]++;
      pointsRemaining--;
    } else {
      // Remover do pool se atingiu limite
      skillPool = skillPool.filter(s => s.name !== skill.name);
    }
    
    // Evitar loop infinito
    if (skillPool.length === 0) break;
  }
  
  return skills;
}

// Inicializar perícias com nível do atributo pai
function initializeSkillsFromAttributes(attributes: Attributes): Skills {
  return {
    // INTELECTO
    logic: attributes.intellect,
    encyclopedia: attributes.intellect,
    rhetoric: attributes.intellect,
    conceptualization: attributes.intellect,
    visualCalculus: attributes.intellect,
    drama: attributes.intellect,
    
    // PSIQUE
    volition: attributes.psyche,
    inlandEmpire: attributes.psyche,
    empathy: attributes.psyche,
    authority: attributes.psyche,
    espritDeCorps: attributes.psyche,
    suggestion: attributes.psyche,
    
    // FÍSICO
    endurance: attributes.physique,
    painThreshold: attributes.physique,
    physicalInstrument: attributes.physique,
    electrochemistry: attributes.physique,
    shivers: attributes.physique,
    halfLight: attributes.physique,
    
    // MOTRICIDADE
    handEyeCoordination: attributes.motorics,
    perception: attributes.motorics,
    reactionSpeed: attributes.motorics,
    savoirFaire: attributes.motorics,
    interfacing: attributes.motorics,
    composure: attributes.motorics
  };
}

// Escolha ponderada
function weightedRandomChoice<T extends { weight: number }>(
  items: T[]
): T {
  const totalWeight = items.reduce((sum, item) => sum + item.weight, 0);
  let random = Math.random() * totalWeight;
  
  for (const item of items) {
    random -= item.weight;
    if (random <= 0) return item;
  }
  
  return items[items.length - 1];
}
```

### 3.3. Seleção de Reflexão Inicial

```typescript
const LEVEL_1_THOUGHTS = [
  'Advogado de Si Mesmo',
  'Camarada de Copas',
  'Dia 1 Servo',
  'Gato Preto',
  'Lente de Homem Morto'
];

const LEVEL_2_THOUGHTS = [
  'Ama-se Como um Predador',
  'Apologia do Homem-Bomba',
  'Arte Suja',
  'Coxinha do Bar',
  'Disco Elysium',
  'Dose Dupla',
  'Foco Laser',
  'Forte Inabalável',
  'Lutador de Rua',
  'Wompty-Dompty Dom Centre',
  'Iluminação Falsa'
];

function generateInitialThought(archetype: Archetype | null): string {
  if (archetype?.thoughtPool) {
    // Escolher do pool do arquétipo
    return randomChoice(archetype.thoughtPool);
  }
  
  // 70% chance Nível 1, 30% chance Nível 2
  const pool = Math.random() < 0.7 ? LEVEL_1_THOUGHTS : LEVEL_2_THOUGHTS;
  return randomChoice(pool);
}
```

### 3.4. Gerador de Nomes Brasileiros

```typescript
const FIRST_NAMES_MALE = [
  'João', 'Pedro', 'Lucas', 'Gabriel', 'Rafael', 'Matheus',
  'Carlos', 'Fernando', 'Ricardo', 'André', 'Bruno', 'Diego',
  'Felipe', 'Gustavo', 'Henrique', 'Igor', 'Júlio', 'Leonardo',
  'Marcelo', 'Nicolas', 'Otávio', 'Paulo', 'Rodrigo', 'Thiago',
  'Vinicius', 'William', 'Alexandre', 'Daniel', 'Eduardo', 'Fábio'
];

const FIRST_NAMES_FEMALE = [
  'Ana', 'Maria', 'Julia', 'Beatriz', 'Carolina', 'Fernanda',
  'Gabriela', 'Helena', 'Isabela', 'Juliana', 'Larissa', 'Mariana',
  'Natália', 'Paula', 'Rafaela', 'Sofia', 'Tatiana', 'Vanessa',
  'Amanda', 'Bianca', 'Camila', 'Daniela', 'Eduarda', 'Fabiana',
  'Giovana', 'Letícia', 'Melissa', 'Patrícia', 'Renata', 'Vitória'
];

const LAST_NAMES = [
  'Silva', 'Santos', 'Oliveira', 'Souza', 'Rodrigues', 'Ferreira',
  'Alves', 'Pereira', 'Lima', 'Gomes', 'Costa', 'Ribeiro',
  'Martins', 'Carvalho', 'Rocha', 'Almeida', 'Nascimento', 'Araújo',
  'Melo', 'Barbosa', 'Cardoso', 'Correia', 'Dias', 'Fernandes',
  'Freitas', 'Garcia', 'Gonçalves', 'Lopes', 'Machado', 'Marques',
  'Mendes', 'Miranda', 'Monteiro', 'Moreira', 'Nunes', 'Pinto',
  'Ramos', 'Reis', 'Rezende', 'Ribeiro', 'Rocha', 'Santana',
  'Teixeira', 'Vieira', 'Castro', 'Campos', 'Moura', 'Pires'
];

function generateBrazilianName(gender?: 'male' | 'female'): string {
  const genderChoice = gender || (Math.random() < 0.5 ? 'male' : 'female');
  
  const firstName = genderChoice === 'male'
    ? randomChoice(FIRST_NAMES_MALE)
    : randomChoice(FIRST_NAMES_FEMALE);
  
  const lastName = randomChoice(LAST_NAMES);
  
  // 30% chance de nome composto
  if (Math.random() < 0.3) {
    const middleName = randomChoice(LAST_NAMES);
    return `${firstName} ${middleName} ${lastName}`;
  }
  
  return `${firstName} ${lastName}`;
}
```

---

## 4. IMPLEMENTAÇÃO COMPLETA

### 4.1. Função Principal

```typescript
interface GenerateCharacterOptions {
  archetype?: 'detective' | 'emotional' | 'brute' | 'fast' | 'random';
  gender?: 'male' | 'female';
  name?: string;
}

function generateRandomCharacter(
  options: GenerateCharacterOptions = {}
): Character {
  // 1. Selecionar arquétipo
  const archetypeKey = options.archetype || 'random';
  const archetype = archetypeKey === 'random' 
    ? null 
    : ARCHETYPES[archetypeKey];
  
  // 2. Gerar atributos
  const attributes = archetype
    ? generateAttributesFromArchetype(archetype)
    : generateRandomAttributes();
  
  // 3. Gerar perícias
  const skills = generateSkills(attributes, archetype);
  
  // 4. Gerar reflexão inicial
  const initialThought = generateInitialThought(archetype);
  
  // 5. Gerar nome
  const name = options.name || generateBrazilianName(options.gender);
  
  // 6. Calcular recursos
  const morale = 1 + skills.volition;
  const health = 1 + skills.endurance;
  
  // 7. Criar personagem
  return {
    id: generateUUID(),
    name,
    createdAt: new Date(),
    updatedAt: new Date(),
    attributes,
    skills,
    resources: {
      morale,
      moraleMax: morale,
      health,
      healthMax: health,
      money: 10, // R$ 10 inicial
      xp: 0
    },
    thoughtCabinet: {
      slots: 3,
      thoughts: [
        {
          id: generateUUID(),
          name: initialThought,
          level: LEVEL_1_THOUGHTS.includes(initialThought) ? 1 : 2,
          status: 'internalized',
          acquiredAt: new Date(),
          internalizedAt: new Date(),
          problem: getThoughtProblem(initialThought),
          solution: getThoughtSolution(initialThought)
        }
      ]
    },
    inventory: [],
    conditions: [],
    history: {
      rollHistory: [],
      xpHistory: [],
      progressionHistory: []
    }
  };
}
```

### 4.2. Funções Auxiliares

```typescript
function randomChoice<T>(array: T[]): T {
  return array[Math.floor(Math.random() * array.length)];
}

function generateUUID(): string {
  return crypto.randomUUID();
}

function getSkillLimit(skillName: string, attributes: Attributes): number {
  const attributeMap = {
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
  
  const parentAttribute = attributeMap[skillName];
  return attributes[parentAttribute] + 1; // Limite = Atributo + 1
}
```

---

## 5. INTERFACE DO USUÁRIO

### 5.1. Componente RandomCharacterGenerator

```tsx
'use client';
import { useState } from 'react';
import { generateRandomCharacter } from '@/lib/character-generator';

export function RandomCharacterGenerator() {
  const [archetype, setArchetype] = useState<string>('random');
  const [gender, setGender] = useState<string>('random');
  const [generatedCharacter, setGeneratedCharacter] = useState<Character | null>(null);

  const handleGenerate = () => {
    const options = {
      archetype: archetype === 'random' ? undefined : archetype,
      gender: gender === 'random' ? undefined : gender
    };
    
    const character = generateRandomCharacter(options);
    setGeneratedCharacter(character);
  };

  const handleSave = async () => {
    if (!generatedCharacter) return;
    await db.characters.add(generatedCharacter);
    router.push(`/characters/${generatedCharacter.id}`);
  };

  return (
    <div className="random-generator">
      <h2>Gerador de Personagens Aleatórios</h2>
      
      <div className="options">
        <label>
          Arquétipo:
          <select value={archetype} onChange={(e) => setArchetype(e.target.value)}>
            <option value="random">🎲 Aleatório</option>
            <option value="detective">🔍 Detetive Clássico</option>
            <option value="emotional">💔 Trem-Bala Emocional</option>
            <option value="brute">💪 Brutamontes</option>
            <option value="fast">⚡ Veloz e Perigoso</option>
          </select>
        </label>
        
        <label>
          Gênero:
          <select value={gender} onChange={(e) => setGender(e.target.value)}>
            <option value="random">🎲 Aleatório</option>
            <option value="male">♂️ Masculino</option>
            <option value="female">♀️ Feminino</option>
          </select>
        </label>
      </div>
      
      <button onClick={handleGenerate} className="btn-primary">
        🎲 Gerar Personagem
      </button>
      
      {generatedCharacter && (
        <div className="preview">
          <h3>{generatedCharacter.name}</h3>
          
          <div className="attributes">
            <h4>Atributos</h4>
            <ul>
              <li>INT: {generatedCharacter.attributes.intellect}</li>
              <li>PSY: {generatedCharacter.attributes.psyche}</li>
              <li>FYS: {generatedCharacter.attributes.physique}</li>
              <li>MOT: {generatedCharacter.attributes.motorics}</li>
            </ul>
          </div>
          
          <div className="resources">
            <h4>Recursos</h4>
            <ul>
              <li>Moral: {generatedCharacter.resources.morale}</li>
              <li>Saúde: {generatedCharacter.resources.health}</li>
            </ul>
          </div>
          
          <div className="thought">
            <h4>Reflexão Inicial</h4>
            <p>{generatedCharacter.thoughtCabinet.thoughts[0].name}</p>
          </div>
          
          <div className="actions">
            <button onClick={handleSave} className="btn-success">
              ✓ Salvar Personagem
            </button>
            <button onClick={handleGenerate} className="btn-secondary">
              🔄 Gerar Outro
            </button>
          </div>
        </div>
      )}
    </div>
  );
}
```

### 5.2. Wireframe

```
┌────────────────────────────────────────┐
│ Gerador de Personagens Aleatórios     │
├────────────────────────────────────────┤
│                                        │
│ Arquétipo: [🎲 Aleatório      ▼]      │
│ Gênero:    [🎲 Aleatório      ▼]      │
│                                        │
│ [🎲 Gerar Personagem]                  │
│                                        │
├────────────────────────────────────────┤
│ PREVIEW (após gerar)                   │
│                                        │
│ Nome: João Silva                       │
│                                        │
│ Atributos:                             │
│ • INT: 4  • PSY: 2                     │
│ • FYS: 2  • MOT: 2                     │
│                                        │
│ Recursos:                              │
│ • Moral: 4  • Saúde: 3                 │
│                                        │
│ Reflexão Inicial:                      │
│ • Advogado de Si Mesmo                 │
│                                        │
│ [✓ Salvar Personagem] [🔄 Gerar Outro] │
└────────────────────────────────────────┘
```

---

## 6. TESTES

### 6.1. Testes Unitários

```typescript
describe('Random Character Generator', () => {
  it('should generate valid attributes (sum = 8)', () => {
    const attrs = generateRandomAttributes();
    const sum = Object.values(attrs).reduce((a, b) => a + b, 0);
    expect(sum).toBe(8);
  });

  it('should respect attribute limits (1-5)', () => {
    const attrs = generateRandomAttributes();
    Object.values(attrs).forEach(value => {
      expect(value).toBeGreaterThanOrEqual(1);
      expect(value).toBeLessThanOrEqual(5);
    });
  });

  it('should distribute 12 skill points', () => {
    const attrs = { intellect: 4, psyche: 2, physique: 2, motorics: 2 };
    const skills = generateSkills(attrs, null);
    
    const basePoints = Object.values(attrs).reduce((a, b) => a + b, 0) * 6; // 48
    const totalPoints = Object.values(skills).reduce((a, b) => a + b, 0);
    
    expect(totalPoints).toBe(basePoints + 12);
  });

  it('should respect skill limits', () => {
    const attrs = { intellect: 3, psyche: 2, physique: 2, motorics: 2 };
    const skills = generateSkills(attrs, null);
    
    // Perícias de INT não podem exceder 4 (3 + 1)
    expect(skills.logic).toBeLessThanOrEqual(4);
    expect(skills.encyclopedia).toBeLessThanOrEqual(4);
  });

  it('should generate Brazilian names', () => {
    const name = generateBrazilianName();
    expect(name).toMatch(/^[A-Za-zÀ-ÿ]+ [A-Za-zÀ-ÿ]+( [A-Za-zÀ-ÿ]+)?$/);
  });
});
```

---

## 7. INTEGRAÇÃO COM PLANOS EXISTENTES

### 7.1. Adicionar ao Plano de Ação

**Fase 12: Gerador de Personagens (Semana 18)**
- Implementar algoritmos de geração
- Criar componente RandomCharacterGenerator
- Testes unitários
- Integrar com criação de personagem

### 7.2. Adicionar ao Plano de Design

**Novo Componente:**
- RandomCharacterGenerator (organism)
- Botão "🎲 Gerar Aleatório" na tela de personagens

---

## 8. CRITÉRIOS DE ACEITAÇÃO

- [ ] Gerar personagem com atributos válidos (soma = 8)
- [ ] Gerar personagem com perícias válidas (12 pontos extras)
- [ ] Respeitar limites de perícias (Atributo + 1)
- [ ] Selecionar reflexão inicial (Nível 1-2)
- [ ] Gerar nome brasileiro aleatório
- [ ] Calcular recursos automaticamente
- [ ] Permitir escolha de arquétipo
- [ ] Permitir escolha de gênero
- [ ] Preview antes de salvar
- [ ] Salvar no IndexedDB

---

**Este plano está pronto para implementação.**  
**Estimativa:** 1 semana de desenvolvimento
