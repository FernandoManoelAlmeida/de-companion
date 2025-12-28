# Plano de Implementação: Calculadora de Combate Avançada

**Data:** 26 de Dezembro de 2025  
**Objetivo:** Ferramenta interativa para gerenciar combates com opção de modo manual

---

## 1. MODOS DE OPERAÇÃO

### 1.1. Modo Automático (Padrão)

- ✅ Cálculos automáticos de ataque/defesa
- ✅ Aplicação automática de dano
- ✅ Condições aplicadas automaticamente
- ✅ Validação de regras
- ✅ Histórico completo

### 1.2. Modo Manual (Opcional)

- ✅ Narrador controla todos os cálculos
- ✅ Sistema apenas registra ações
- ✅ Sem validação automática
- ✅ Flexibilidade total para house rules
- ✅ Histórico narrativo

### 1.3. Toggle de Modo

```tsx
// components/CombatCalculator.tsx
export function CombatCalculator({ campaignId }: { campaignId: string }) {
  const [autoMode, setAutoMode] = useState(true);

  return (
    <div className="combat-calculator">
      <div className="mode-toggle">
        <label>
          <input
            type="checkbox"
            checked={autoMode}
            onChange={(e) => setAutoMode(e.target.checked)}
          />
          Cálculos Automáticos
        </label>
        <span className="mode-description">
          {autoMode
            ? 'Sistema calcula dano e aplica condições automaticamente'
            : 'Controle manual de todos os cálculos'}
        </span>
      </div>

      {autoMode ? <AutomaticCombatInterface /> : <ManualCombatInterface />}
    </div>
  );
}
```

---

## 2. INTERFACE MODO AUTOMÁTICO

### 2.1. Ataque Automático

```tsx
function AutomaticAttack({ attacker, defender }: Props) {
  const [weapon, setWeapon] = useState<Weapon | null>(null);
  const [cover, setCover] = useState<CoverType | null>(null);

  const handleAttack = async () => {
    // 1. Rolar ataque
    const attackRoll = rollDice(attacker.skills.physicalInstrument, attacker.attributes.physique);
    const attackBonus = weapon?.bonus || 0;
    const attackTotal = attackRoll.total + attackBonus;

    // 2. Rolar defesa
    const defenseRoll = rollDice(defender.skills.halfLight, defender.attributes.physique);
    const coverBonus = cover ? rollDice(0, 0).dice[0] : 0; // +1d6
    const defenseTotal = defenseRoll.total + coverBonus;

    // 3. Calcular resultado
    const hit = attackTotal > defenseTotal;
    const damage = hit ? 1 : 0;

    // 4. Aplicar dano
    if (hit) {
      await applyDamage(defender.id, damage);

      // 5. Aplicar condições automáticas
      await applyCondition(defender.id, 'Sangramento Leve');

      // 6. Verificar Instante de Morte
      if (defender.health <= 3 && defender.health - damage <= 0) {
        await checkInstantDeath(defender.id);
      }
    }

    // 7. Registrar no histórico
    await logCombatAction({
      attacker: attacker.id,
      defender: defender.id,
      attackRoll,
      defenseRoll,
      hit,
      damage,
    });
  };

  return (
    <div className="automatic-attack">
      {/* Interface de ataque */}
      <button onClick={handleAttack}>Executar Ataque</button>
    </div>
  );
}
```

---

## 3. INTERFACE MODO MANUAL

### 3.1. Registro Manual

```tsx
function ManualCombatInterface() {
  const [action, setAction] = useState('');
  const [result, setResult] = useState('');

  const handleLogAction = async () => {
    // Apenas registrar no histórico, sem cálculos
    await logNarrativeAction({
      description: action,
      result,
      timestamp: new Date(),
    });

    setAction('');
    setResult('');
  };

  return (
    <div className="manual-combat">
      <h3>Modo Manual</h3>
      <p>Registre ações livremente sem cálculos automáticos</p>

      <label>
        Ação:
        <textarea
          value={action}
          onChange={(e) => setAction(e.target.value)}
          placeholder="Ex: João ataca o suspeito com faca"
        />
      </label>

      <label>
        Resultado:
        <textarea
          value={result}
          onChange={(e) => setResult(e.target.value)}
          placeholder="Ex: Acerto! Suspeito recebe 1 de dano"
        />
      </label>

      <button onClick={handleLogAction}>Registrar Ação</button>

      <div className="manual-tools">
        <h4>Ferramentas Auxiliares</h4>
        <button onClick={() => rollDice(0, 0)}>🎲 Rolar 2d6</button>
        <button onClick={() => applyDamageManual()}>❤️ Aplicar Dano Manual</button>
        <button onClick={() => addConditionManual()}>⚠️ Adicionar Condição</button>
      </div>
    </div>
  );
}
```

---

## 4. WIREFRAME COMPARATIVO

### Modo Automático

```
┌────────────────────────────────────────┐
│ COMBATE                                │
│ ☑ Cálculos Automáticos                 │
├────────────────────────────────────────┤
│ ATAQUE: João → Suspeito                │
│                                        │
│ Atacante: João Silva                   │
│ Perícia: Instrumento Físico (6)        │
│ Arma: [Faca de bolso (+1) ▼]          │
│                                        │
│ Defensor: Suspeito                     │
│ Perícia: Meia-Luz (3)                  │
│ Cobertura: [Nenhuma ▼]                 │
│                                        │
│ [Executar Ataque Automático]           │
│                                        │
│ Resultado:                             │
│ Ataque: 2d6 (4,3) + 6 + 1 = 14        │
│ Defesa: 2d6 (2,5) + 3 = 10            │
│ ✓ ACERTO! Dano: 1                      │
│ Suspeito: 4→3 Saúde                    │
│ + Condição: Sangramento Leve           │
└────────────────────────────────────────┘
```

### Modo Manual

```
┌────────────────────────────────────────┐
│ COMBATE                                │
│ ☐ Cálculos Automáticos                 │
├────────────────────────────────────────┤
│ MODO MANUAL                            │
│ Registre ações livremente              │
│                                        │
│ Ação:                                  │
│ [João ataca suspeito com faca______]   │
│                                        │
│ Resultado:                             │
│ [Acerto! Suspeito recebe 1 de dano_]   │
│                                        │
│ [Registrar Ação]                       │
│                                        │
│ Ferramentas:                           │
│ [🎲 Rolar 2d6] [❤️ Dano] [⚠️ Condição] │
│                                        │
│ Histórico:                             │
│ • João ataca suspeito (Acerto, 1 dano) │
│ • Suspeito contra-ataca (Errou)        │
└────────────────────────────────────────┘
```

---

## 5. CONFIGURAÇÕES PERSISTENTES

```typescript
// Salvar preferência do usuário
interface CombatSettings {
  autoMode: boolean;
  showAnimations: boolean;
  confirmActions: boolean;
}

// Salvar em localStorage
localStorage.setItem(
  'combatSettings',
  JSON.stringify({
    autoMode: true,
    showAnimations: true,
    confirmActions: false,
  })
);
```

---

## 6. CRITÉRIOS DE ACEITAÇÃO

- [ ] Toggle de modo automático/manual funciona
- [ ] Modo automático calcula tudo corretamente
- [ ] Modo manual permite registro livre
- [ ] Preferência é salva (localStorage)
- [ ] Histórico funciona em ambos os modos
- [ ] Ferramentas auxiliares no modo manual
- [ ] Interface clara sobre qual modo está ativo

---

**Este plano complementa o sistema de combate com flexibilidade total.**  
**Estimativa:** 1 semana adicional (total 4 semanas para combate completo)
