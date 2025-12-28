# Plano de Implementação: Modo Multiplayer

**Data:** 26 de Dezembro de 2025  
**Objetivo:** Permitir que múltiplos usuários compartilhem campanhas em tempo real

---

## 1. VISÃO GERAL

### 1.1. Funcionalidade

Transformar campanhas **locais** (apenas um dispositivo) em campanhas **compartilhadas** (múltiplos usuários conectados em tempo real).

### 1.2. Casos de Uso

- **Sessão Presencial:** Todos na mesma mesa, cada um com seu dispositivo
- **Sessão Online:** Jogadores remotos via Discord/Zoom
- **Jogo Assíncrono:** Jogadores fazem ações em momentos diferentes

### 1.3. Benefícios

- ✅ Sincronização automática de dados
- ✅ Narrador revela informações em tempo real
- ✅ Todos veem rolagens simultaneamente
- ✅ Histórico compartilhado de eventos
- ✅ Facilita jogo remoto

---

## 2. ARQUITETURA TÉCNICA

### 2.1. Escolha de Backend: Firebase Realtime Database

**Por que Firebase?**

- ✅ Sincronização em tempo real nativa
- ✅ Offline support (cache local)
- ✅ Fácil integração com Next.js
- ✅ **Gratuito:** 1GB armazenamento, 10GB/mês transferência
- ✅ Autenticação integrada (Firebase Auth)
- ✅ Security Rules para controle de acesso

**Alternativas Consideradas:**

- Supabase: Mais complexo, PostgreSQL
- WebRTC P2P: Sem servidor, mas requer todos online
- Socket.io: Requer servidor Node.js próprio

### 2.2. Stack Tecnológico

```bash
# Dependências
npm install firebase
npm install @firebase/auth
npm install @firebase/database
```

### 2.3. Estrutura de Dados no Firebase

```typescript
// Firebase Realtime Database
{
  "campaigns": {
    "{campaignId}": {
      "metadata": {
        "name": "Mistério de Revachol",
        "description": "...",
        "createdAt": 1703606400000,
        "updatedAt": 1703606400000,
        "mode": "multiplayer", // "local" | "multiplayer"
        "inviteCode": "ABC-123-XYZ",
        "narratorId": "user_abc123"
      },

      "permissions": {
        "narrator": "user_abc123",
        "players": {
          "user_def456": true,
          "user_ghi789": true
        },
        "spectators": {
          "user_jkl012": true
        }
      },

      "characters": {
        "{characterId}": {
          "ownerId": "user_def456",
          "name": "João Silva",
          "attributes": { /* ... */ },
          "skills": { /* ... */ },
          "resources": { /* ... */ },
          "updatedAt": 1703606400000
        }
      },

      "sessions": {
        "{sessionId}": {
          "sessionNumber": 12,
          "date": 1703606400000,
          "summary": "Confronto no cais",
          "narratorSummary": "Pistas ocultas...",
          "isActive": true,

          "events": {
            "{eventId}": {
              "type": "roll", // "roll" | "note" | "combat" | "revelation"
              "userId": "user_def456",
              "characterId": "{characterId}",
              "timestamp": 1703606400000,
              "isRevealed": true,
              "data": {
                "skill": "perception",
                "result": 15,
                "dice": [5, 4],
                "success": true
              }
            }
          }
        }
      },

      "narratorNotes": {
        "{noteId}": {
          "content": "Suspeito esconde arma no porão",
          "createdAt": 1703606400000,
          "isRevealed": false,
          "revealedAt": null,
          "tags": ["pista", "arma"]
        }
      },

      "presence": {
        "user_abc123": {
          "online": true,
          "lastSeen": 1703606400000,
          "role": "narrator"
        },
        "user_def456": {
          "online": true,
          "lastSeen": 1703606400000,
          "role": "player",
          "characterId": "{characterId}"
        }
      },

      "chat": {
        "{messageId}": {
          "userId": "user_def456",
          "userName": "João",
          "message": "Vou investigar o porão",
          "timestamp": 1703606400000,
          "type": "text" // "text" | "roll" | "system"
        }
      }
    }
  },

  "users": {
    "{userId}": {
      "displayName": "Fernando",
      "email": "fernando@example.com",
      "photoURL": null,
      "createdAt": 1703606400000
    }
  },

  "inviteCodes": {
    "ABC-123-XYZ": {
      "campaignId": "{campaignId}",
      "createdBy": "user_abc123",
      "expiresAt": 1704211200000, // 7 dias
      "maxUses": 10,
      "usedCount": 2
    }
  }
}
```

---

## 3. SISTEMA DE AUTENTICAÇÃO

### 3.1. Firebase Auth (Anônimo + Email)

```typescript
// lib/firebase.ts
import { initializeApp } from 'firebase/app';
import { getAuth, signInAnonymously, signInWithEmailAndPassword } from 'firebase/auth';
import { getDatabase } from 'firebase/database';

const firebaseConfig = {
  apiKey: process.env.NEXT_PUBLIC_FIREBASE_API_KEY,
  authDomain: process.env.NEXT_PUBLIC_FIREBASE_AUTH_DOMAIN,
  databaseURL: process.env.NEXT_PUBLIC_FIREBASE_DATABASE_URL,
  projectId: process.env.NEXT_PUBLIC_FIREBASE_PROJECT_ID,
};

const app = initializeApp(firebaseConfig);
export const auth = getAuth(app);
export const database = getDatabase(app);

// Login anônimo (padrão)
export async function signInAnonymous() {
  const result = await signInAnonymously(auth);
  return result.user;
}

// Login com email (opcional)
export async function signInWithEmail(email: string, password: string) {
  const result = await signInWithEmailAndPassword(auth, email, password);
  return result.user;
}
```

### 3.2. Fluxo de Autenticação

```typescript
// hooks/useAuth.ts
import { useEffect, useState } from 'react';
import { auth, signInAnonymous } from '@/lib/firebase';
import { onAuthStateChanged, User } from 'firebase/auth';

export function useAuth() {
  const [user, setUser] = useState<User | null>(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    const unsubscribe = onAuthStateChanged(auth, async (user) => {
      if (!user) {
        // Auto-login anônimo
        user = await signInAnonymous();
      }
      setUser(user);
      setLoading(false);
    });

    return unsubscribe;
  }, []);

  return { user, loading };
}
```

---

## 4. SISTEMA DE CONVITES

### 4.1. Geração de Código de Convite

```typescript
// lib/invite-codes.ts
import { ref, set, get } from 'firebase/database';
import { database } from './firebase';

function generateInviteCode(): string {
  const chars = 'ABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789';
  const parts = [
    Array.from({ length: 3 }, () => chars[Math.floor(Math.random() * chars.length)]).join(''),
    Array.from({ length: 3 }, () => chars[Math.floor(Math.random() * chars.length)]).join(''),
    Array.from({ length: 3 }, () => chars[Math.floor(Math.random() * chars.length)]).join(''),
  ];
  return parts.join('-'); // "ABC-123-XYZ"
}

export async function createInviteCode(
  campaignId: string,
  userId: string,
  maxUses: number = 10
): Promise<string> {
  const code = generateInviteCode();
  const inviteRef = ref(database, `inviteCodes/${code}`);

  await set(inviteRef, {
    campaignId,
    createdBy: userId,
    createdAt: Date.now(),
    expiresAt: Date.now() + 7 * 24 * 60 * 60 * 1000, // 7 dias
    maxUses,
    usedCount: 0,
  });

  return code;
}

export async function validateInviteCode(code: string): Promise<{
  valid: boolean;
  campaignId?: string;
  error?: string;
}> {
  const inviteRef = ref(database, `inviteCodes/${code}`);
  const snapshot = await get(inviteRef);

  if (!snapshot.exists()) {
    return { valid: false, error: 'Código inválido' };
  }

  const invite = snapshot.val();

  if (Date.now() > invite.expiresAt) {
    return { valid: false, error: 'Código expirado' };
  }

  if (invite.usedCount >= invite.maxUses) {
    return { valid: false, error: 'Código atingiu limite de usos' };
  }

  return { valid: true, campaignId: invite.campaignId };
}
```

### 4.2. Entrar na Campanha

```typescript
// lib/campaign-join.ts
import { ref, update, increment } from 'firebase/database';
import { database } from './firebase';

export async function joinCampaign(
  inviteCode: string,
  userId: string,
  characterId?: string
): Promise<void> {
  // 1. Validar código
  const validation = await validateInviteCode(inviteCode);
  if (!validation.valid) {
    throw new Error(validation.error);
  }

  const campaignId = validation.campaignId!;

  // 2. Adicionar usuário às permissões
  const permissionsRef = ref(database, `campaigns/${campaignId}/permissions/players/${userId}`);
  await set(permissionsRef, true);

  // 3. Vincular personagem (se fornecido)
  if (characterId) {
    const charRef = ref(database, `campaigns/${campaignId}/characters/${characterId}`);
    await update(charRef, {
      ownerId: userId,
      linkedAt: Date.now(),
    });
  }

  // 4. Incrementar contador de usos
  const inviteRef = ref(database, `inviteCodes/${inviteCode}/usedCount`);
  await update(inviteRef, increment(1));

  // 5. Adicionar presença
  const presenceRef = ref(database, `campaigns/${campaignId}/presence/${userId}`);
  await set(presenceRef, {
    online: true,
    lastSeen: Date.now(),
    role: 'player',
    characterId: characterId || null,
  });
}
```

---

## 5. SINCRONIZAÇÃO EM TEMPO REAL

### 5.1. Hook de Sincronização

```typescript
// hooks/useCampaignSync.ts
import { useEffect, useState } from 'react';
import { ref, onValue, off } from 'firebase/database';
import { database } from '@/lib/firebase';
import { db } from '@/lib/db'; // IndexedDB local

export function useCampaignSync(campaignId: string) {
  const [syncing, setSyncing] = useState(false);
  const [lastSync, setLastSync] = useState<Date | null>(null);

  useEffect(() => {
    if (!campaignId) return;

    const campaignRef = ref(database, `campaigns/${campaignId}`);

    // Escutar mudanças
    const unsubscribe = onValue(campaignRef, async (snapshot) => {
      setSyncing(true);
      const data = snapshot.val();

      if (data) {
        // Sincronizar com IndexedDB local
        await syncToLocal(campaignId, data);
        setLastSync(new Date());
      }

      setSyncing(false);
    });

    return () => off(campaignRef, 'value', unsubscribe);
  }, [campaignId]);

  return { syncing, lastSync };
}

async function syncToLocal(campaignId: string, data: any) {
  // Atualizar campanha local
  await db.campaigns.put({
    id: campaignId,
    ...data.metadata,
    mode: 'multiplayer',
    syncedAt: Date.now(),
  });

  // Atualizar personagens
  if (data.characters) {
    for (const [charId, charData] of Object.entries(data.characters)) {
      await db.characters.put({
        id: charId,
        ...(charData as any),
      });
    }
  }

  // Atualizar sessões
  if (data.sessions) {
    for (const [sessionId, sessionData] of Object.entries(data.sessions)) {
      await db.sessions.put({
        id: sessionId,
        campaignId,
        ...(sessionData as any),
      });
    }
  }
}
```

### 5.2. Revelar Anotação (Tempo Real)

```typescript
// lib/narrator-actions.ts
import { ref, update } from 'firebase/database';
import { database } from './firebase';

export async function revealNote(campaignId: string, noteId: string): Promise<void> {
  const noteRef = ref(database, `campaigns/${campaignId}/narratorNotes/${noteId}`);

  await update(noteRef, {
    isRevealed: true,
    revealedAt: Date.now(),
  });

  // Todos os jogadores conectados recebem atualização automaticamente
}

export async function revealAllNotes(campaignId: string): Promise<void> {
  const notesRef = ref(database, `campaigns/${campaignId}/narratorNotes`);
  const snapshot = await get(notesRef);

  if (snapshot.exists()) {
    const updates: any = {};
    snapshot.forEach((child) => {
      updates[`${child.key}/isRevealed`] = true;
      updates[`${child.key}/revealedAt`] = Date.now();
    });

    await update(notesRef, updates);
  }
}
```

---

## 6. SISTEMA DE PRESENÇA

### 6.1. Detectar Online/Offline

```typescript
// hooks/usePresence.ts
import { useEffect } from 'react';
import { ref, onDisconnect, set, serverTimestamp } from 'firebase/database';
import { database } from '@/lib/firebase';

export function usePresence(campaignId: string, userId: string, role: string) {
  useEffect(() => {
    if (!campaignId || !userId) return;

    const presenceRef = ref(database, `campaigns/${campaignId}/presence/${userId}`);

    // Marcar como online
    set(presenceRef, {
      online: true,
      lastSeen: serverTimestamp(),
      role,
    });

    // Configurar desconexão
    onDisconnect(presenceRef).set({
      online: false,
      lastSeen: serverTimestamp(),
      role,
    });

    // Cleanup
    return () => {
      set(presenceRef, {
        online: false,
        lastSeen: Date.now(),
        role,
      });
    };
  }, [campaignId, userId, role]);
}
```

### 6.2. Componente de Presença

```tsx
// components/PresenceIndicator.tsx
'use client';
import { useEffect, useState } from 'react';
import { ref, onValue } from 'firebase/database';
import { database } from '@/lib/firebase';

interface User {
  online: boolean;
  lastSeen: number;
  role: 'narrator' | 'player' | 'spectator';
  characterId?: string;
}

export function PresenceIndicator({ campaignId }: { campaignId: string }) {
  const [users, setUsers] = useState<Record<string, User>>({});

  useEffect(() => {
    const presenceRef = ref(database, `campaigns/${campaignId}/presence`);

    const unsubscribe = onValue(presenceRef, (snapshot) => {
      setUsers(snapshot.val() || {});
    });

    return unsubscribe;
  }, [campaignId]);

  const online = Object.entries(users).filter(([_, u]) => u.online);
  const offline = Object.entries(users).filter(([_, u]) => !u.online);

  return (
    <div className="presence-indicator">
      <h4>👥 Online ({online.length})</h4>
      <ul>
        {online.map(([userId, user]) => (
          <li key={userId}>
            {user.role === 'narrator' ? '🎭' : '👤'} {userId}
            <span className="online-dot">●</span>
          </li>
        ))}
      </ul>

      {offline.length > 0 && (
        <>
          <h4>💤 Offline ({offline.length})</h4>
          <ul>
            {offline.map(([userId, user]) => (
              <li key={userId} className="offline">
                {user.role === 'narrator' ? '🎭' : '👤'} {userId}
              </li>
            ))}
          </ul>
        </>
      )}
    </div>
  );
}
```

---

## 7. SECURITY RULES (Firebase)

```json
{
  "rules": {
    "campaigns": {
      "$campaignId": {
        ".read": "auth != null && (
          data.child('permissions/narrator').val() == auth.uid ||
          data.child('permissions/players/' + auth.uid).exists() ||
          data.child('permissions/spectators/' + auth.uid).exists()
        )",

        "metadata": {
          ".write": "auth != null && data.parent().child('permissions/narrator').val() == auth.uid"
        },

        "narratorNotes": {
          ".read": "auth != null && data.parent().child('permissions/narrator').val() == auth.uid",
          ".write": "auth != null && data.parent().child('permissions/narrator').val() == auth.uid"
        },

        "characters": {
          "$characterId": {
            ".write": "auth != null && (
              data.child('ownerId').val() == auth.uid ||
              data.parent().parent().child('permissions/narrator').val() == auth.uid
            )"
          }
        },

        "sessions": {
          ".write": "auth != null && data.parent().child('permissions/narrator').val() == auth.uid"
        },

        "presence": {
          "$userId": {
            ".write": "auth != null && $userId == auth.uid"
          }
        },

        "chat": {
          ".write": "auth != null"
        }
      }
    },

    "inviteCodes": {
      "$code": {
        ".read": "auth != null",
        ".write": "auth != null"
      }
    }
  }
}
```

---

## 8. INTERFACE DO USUÁRIO

### 8.1. Criar Campanha Multiplayer

```tsx
// components/CreateCampaignModal.tsx
'use client';
import { useState } from 'react';
import { useAuth } from '@/hooks/useAuth';
import { createInviteCode } from '@/lib/invite-codes';

export function CreateCampaignModal() {
  const { user } = useAuth();
  const [mode, setMode] = useState<'local' | 'multiplayer'>('local');
  const [inviteCode, setInviteCode] = useState<string | null>(null);

  const handleCreate = async () => {
    // Criar campanha localmente
    const campaign = await db.campaigns.add({
      name,
      description,
      mode,
      narratorId: user!.uid,
      createdAt: new Date(),
    });

    if (mode === 'multiplayer') {
      // Criar no Firebase
      const campaignRef = ref(database, `campaigns/${campaign.id}`);
      await set(campaignRef, {
        metadata: { name, description, narratorId: user!.uid },
        permissions: { narrator: user!.uid, players: {}, spectators: {} },
      });

      // Gerar código de convite
      const code = await createInviteCode(campaign.id, user!.uid);
      setInviteCode(code);
    }
  };

  return (
    <div className="modal">
      <h2>Nova Campanha</h2>

      <input placeholder="Nome da campanha" />

      <label>
        Modo:
        <select value={mode} onChange={(e) => setMode(e.target.value as any)}>
          <option value="local">Local (apenas este dispositivo)</option>
          <option value="multiplayer">Multiplayer (compartilhado)</option>
        </select>
      </label>

      <button onClick={handleCreate}>Criar Campanha</button>

      {inviteCode && (
        <div className="invite-code">
          <h3>Código de Convite:</h3>
          <code>{inviteCode}</code>
          <button onClick={() => navigator.clipboard.writeText(inviteCode)}>Copiar</button>
        </div>
      )}
    </div>
  );
}
```

### 8.2. Entrar na Campanha

```tsx
// components/JoinCampaignModal.tsx
'use client';
import { useState } from 'react';
import { useAuth } from '@/hooks/useAuth';
import { joinCampaign, validateInviteCode } from '@/lib/campaign-join';

export function JoinCampaignModal() {
  const { user } = useAuth();
  const [code, setCode] = useState('');
  const [error, setError] = useState<string | null>(null);

  const handleJoin = async () => {
    setError(null);

    const validation = await validateInviteCode(code);
    if (!validation.valid) {
      setError(validation.error!);
      return;
    }

    await joinCampaign(code, user!.uid);
    // Redirecionar para campanha
  };

  return (
    <div className="modal">
      <h2>Entrar em Campanha</h2>

      <input
        placeholder="Código de Convite (ABC-123-XYZ)"
        value={code}
        onChange={(e) => setCode(e.target.value.toUpperCase())}
      />

      {error && <p className="error">{error}</p>}

      <button onClick={handleJoin}>Entrar</button>
    </div>
  );
}
```

---

## 9. CHAT INTEGRADO

```typescript
// lib/chat.ts
import { ref, push, query, orderByChild, limitToLast } from 'firebase/database';
import { database } from './firebase';

export async function sendMessage(
  campaignId: string,
  userId: string,
  userName: string,
  message: string
): Promise<void> {
  const chatRef = ref(database, `campaigns/${campaignId}/chat`);

  await push(chatRef, {
    userId,
    userName,
    message,
    timestamp: Date.now(),
    type: 'text',
  });
}

export function useChatMessages(campaignId: string, limit: number = 50) {
  const [messages, setMessages] = useState<any[]>([]);

  useEffect(() => {
    const chatRef = ref(database, `campaigns/${campaignId}/chat`);
    const chatQuery = query(chatRef, orderByChild('timestamp'), limitToLast(limit));

    const unsubscribe = onValue(chatQuery, (snapshot) => {
      const msgs: any[] = [];
      snapshot.forEach((child) => {
        msgs.push({ id: child.key, ...child.val() });
      });
      setMessages(msgs);
    });

    return unsubscribe;
  }, [campaignId, limit]);

  return messages;
}
```

---

## 10. CUSTOS E LIMITES

### Firebase Spark (Gratuito)

- **Armazenamento:** 1 GB
- **Transferência:** 10 GB/mês
- **Conexões simultâneas:** 100

### Estimativa de Uso

- 1 campanha ativa ≈ 5 MB
- 1 sessão de 4 horas ≈ 2 MB de eventos
- **Capacidade:** ~200 campanhas simultâneas

---

## 11. ROADMAP DE IMPLEMENTAÇÃO

### Fase 1: Setup (1 semana)

- [ ] Configurar Firebase projeto
- [ ] Implementar autenticação anônima
- [ ] Security Rules básicas

### Fase 2: Convites (1 semana)

- [ ] Sistema de códigos de convite
- [ ] UI de criar/entrar campanha
- [ ] Validação e expiração

### Fase 3: Sincronização (2 semanas)

- [ ] Sync em tempo real (Firebase ↔ IndexedDB)
- [ ] Resolução de conflitos
- [ ] Offline support

### Fase 4: Presença e Chat (1 semana)

- [ ] Sistema de presença
- [ ] Indicadores online/offline
- [ ] Chat integrado

### Fase 5: Testes e Polish (1 semana)

- [ ] Testes com múltiplos usuários
- [ ] Performance optimization
- [ ] Bug fixes

**Total:** 6 semanas

---

## 12. CRITÉRIOS DE ACEITAÇÃO

- [ ] Narrador cria campanha multiplayer
- [ ] Gera código de convite
- [ ] Jogador entra com código
- [ ] Vincula personagem à campanha
- [ ] Sincronização em tempo real funciona
- [ ] Narrador revela anotação, jogadores veem
- [ ] Presença online/offline atualiza
- [ ] Chat funciona
- [ ] Funciona offline (cache local)
- [ ] Security rules impedem acesso não autorizado

---

**Este plano está pronto para implementação.**  
**Custo:** R$ 0 (Firebase gratuito)  
**Estimativa:** 6 semanas de desenvolvimento
