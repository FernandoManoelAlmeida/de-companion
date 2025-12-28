# Plano de Implementação: Sistema de Internacionalização (i18n)

**Data:** 26 de Dezembro de 2025  
**Objetivo:** Suporte a múltiplos idiomas com PT-BR base, EN nativo, e tradução AI para outros idiomas

---

## 1. ESTRATÉGIA DE TRADUÇÃO

### 1.1. Níveis de Suporte

#### Tier 1: Idiomas Nativos (Tradução Manual)

- **Português (pt-BR)** - Idioma base, 100% completo
- **Inglês (en)** - Tradução nativa manual, 100% completo

#### Tier 2: Idiomas com Tradução AI (On-Demand)

- **Espanhol (es)**
- **Francês (fr)**
- **Alemão (de)**
- **Italiano (it)**
- **Japonês (ja)**
- **Chinês Simplificado (zh-CN)**
- Outros sob demanda

### 1.2. Sistema Híbrido (Inspirado no Reddit)

**Como o Reddit Funciona:**

- Conteúdo base em inglês (ou idioma original)
- Botão "Translate" aparece para usuários de outros idiomas
- Tradução via API (Google Translate ou similar)
- Cache de traduções para performance
- Fallback para original se tradução falhar

**Nossa Implementação:**

1. **Interface (UI)**: Tradução manual (pt-BR + en)
2. **Conteúdo Estático**: Tradução manual
3. **Conteúdo Dinâmico** (nomes de personagens, anotações):
   - Opção de traduzir via AI
   - Botão "Traduzir" individual
   - Cache local de traduções

---

## 2. STACK TECNOLÓGICO

### 2.1. Biblioteca i18n: next-intl

**Por que next-intl?**

- ✅ Melhor integração com Next.js App Router (2024)
- ✅ Type-safe (TypeScript)
- ✅ Server Components support
- ✅ Automatic locale detection
- ✅ Performance otimizada

```bash
npm install next-intl
```

### 2.2. API de Tradução AI

**Opção 1: DeepL API (Recomendado)**

- ✅ Qualidade superior (melhor que Google)
- ✅ 500,000 caracteres/mês GRÁTIS
- ✅ Suporta 31 idiomas
- ✅ Contexto e nuances preservados

```bash
npm install deepl-node
```

**Opção 2: LibreTranslate (Open Source)**

- ✅ Completamente gratuito
- ✅ Self-hosted ou API pública
- ✅ Sem limites de caracteres
- ❌ Qualidade inferior ao DeepL

```bash
npm install @libretranslate/client
```

**Opção 3: Google Gemini API**

- ✅ Gratuito com limites generosos
- ✅ Qualidade excelente
- ✅ Contexto preservado
- ❌ Requer Google Cloud account

**Decisão:** Usar **DeepL** como primário, **LibreTranslate** como fallback.

---

## 3. ESTRUTURA DE ARQUIVOS

### 3.1. Organização de Traduções

```
messages/
├── pt-BR.json          # Português (base)
├── en.json             # Inglês (nativo)
├── es.json             # Espanhol (AI-generated)
├── fr.json             # Francês (AI-generated)
└── common/
    ├── skills.json     # Perícias (compartilhado)
    ├── thoughts.json   # Reflexões (compartilhado)
    └── items.json      # Itens (compartilhado)
```

### 3.2. Exemplo de Arquivo de Tradução

**messages/pt-BR.json**

```json
{
  "common": {
    "appName": "Detetive Existencial Companion",
    "loading": "Carregando...",
    "save": "Salvar",
    "cancel": "Cancelar",
    "delete": "Deletar",
    "export": "Exportar",
    "import": "Importar"
  },
  "character": {
    "create": "Criar Personagem",
    "name": "Nome",
    "attributes": "Atributos",
    "skills": "Perícias",
    "morale": "Moral",
    "health": "Saúde",
    "xp": "Experiência"
  },
  "skills": {
    "logic": {
      "name": "Lógica",
      "description": "O pedante que analisa tecnicamente cada argumento..."
    },
    "encyclopedia": {
      "name": "Enciclopédia",
      "description": "O nerd que vomita curiosidades aleatórias..."
    }
  },
  "dice": {
    "roll": "Rolar Dados",
    "result": "Resultado",
    "success": "Sucesso",
    "failure": "Falha",
    "genericRoll": "Rolagem Genérica",
    "notation": "Notação (ex: 1d20, 2d6+5)"
  },
  "narrator": {
    "mode": "Modo Narrador",
    "playerMode": "Modo Jogador",
    "privateNotes": "Anotações Privadas",
    "reveal": "Revelar",
    "revealAll": "Revelar Todas",
    "hidden": "Oculto"
  }
}
```

**messages/en.json**

```json
{
  "common": {
    "appName": "Existential Detective Companion",
    "loading": "Loading...",
    "save": "Save",
    "cancel": "Cancel",
    "delete": "Delete",
    "export": "Export",
    "import": "Import"
  },
  "character": {
    "create": "Create Character",
    "name": "Name",
    "attributes": "Attributes",
    "skills": "Skills",
    "morale": "Morale",
    "health": "Health",
    "xp": "Experience"
  },
  "dice": {
    "roll": "Roll Dice",
    "result": "Result",
    "success": "Success",
    "failure": "Failure",
    "genericRoll": "Generic Roll",
    "notation": "Notation (e.g., 1d20, 2d6+5)"
  }
}
```

---

## 4. IMPLEMENTAÇÃO TÉCNICA

### 4.1. Configuração next-intl

**i18n.ts**

```typescript
import { getRequestConfig } from 'next-intl/server';
import { notFound } from 'next/navigation';

// Idiomas suportados
export const locales = ['pt-BR', 'en', 'es', 'fr', 'de'] as const;
export type Locale = (typeof locales)[number];

export default getRequestConfig(async ({ locale }) => {
  // Validar locale
  if (!locales.includes(locale as Locale)) notFound();

  return {
    messages: (await import(`./messages/${locale}.json`)).default,
  };
});
```

**middleware.ts**

```typescript
import createMiddleware from 'next-intl/middleware';
import { locales } from './i18n';

export default createMiddleware({
  locales,
  defaultLocale: 'pt-BR',
  localeDetection: true, // Auto-detecta via Accept-Language
  localePrefix: 'as-needed', // /en/characters, mas / para pt-BR
});

export const config = {
  matcher: ['/', '/(pt-BR|en|es|fr|de)/:path*'],
};
```

**app/[locale]/layout.tsx**

```typescript
import { NextIntlClientProvider } from 'next-intl';
import { getMessages } from 'next-intl/server';

export default async function LocaleLayout({
  children,
  params: { locale }
}: {
  children: React.ReactNode;
  params: { locale: string };
}) {
  const messages = await getMessages();

  return (
    <html lang={locale}>
      <body>
        <NextIntlClientProvider messages={messages}>
          {children}
        </NextIntlClientProvider>
      </body>
    </html>
  );
}
```

### 4.2. Uso em Componentes

**Client Component**

```typescript
'use client';
import { useTranslations } from 'next-intl';

export function CharacterForm() {
  const t = useTranslations('character');

  return (
    <div>
      <h1>{t('create')}</h1>
      <label>{t('name')}</label>
      <input placeholder={t('name')} />
      <button>{t.rich('common.save')}</button>
    </div>
  );
}
```

**Server Component**

```typescript
import { useTranslations } from 'next-intl';

export default function CharactersPage() {
  const t = useTranslations('character');

  return <h1>{t('create')}</h1>;
}
```

### 4.3. Language Switcher

**components/LanguageSwitcher.tsx**

```typescript
'use client';
import { useLocale } from 'next-intl';
import { useRouter, usePathname } from 'next/navigation';
import { locales } from '@/i18n';

const languageNames: Record<string, string> = {
  'pt-BR': '🇧🇷 Português',
  'en': '🇺🇸 English',
  'es': '🇪🇸 Español',
  'fr': '🇫🇷 Français',
  'de': '🇩🇪 Deutsch'
};

export function LanguageSwitcher() {
  const locale = useLocale();
  const router = useRouter();
  const pathname = usePathname();

  const switchLocale = (newLocale: string) => {
    // Remove locale atual do path
    const pathWithoutLocale = pathname.replace(`/${locale}`, '');
    // Adiciona novo locale
    const newPath = `/${newLocale}${pathWithoutLocale}`;
    router.push(newPath);
  };

  return (
    <select
      value={locale}
      onChange={(e) => switchLocale(e.target.value)}
      className="language-switcher"
    >
      {locales.map((loc) => (
        <option key={loc} value={loc}>
          {languageNames[loc]}
        </option>
      ))}
    </select>
  );
}
```

---

## 5. TRADUÇÃO AI PARA CONTEÚDO DINÂMICO

### 5.1. Serviço de Tradução

**lib/translation.ts**

```typescript
import * as deepl from 'deepl-node';

// Inicializar DeepL (free tier)
const translator = new deepl.Translator(process.env.DEEPL_API_KEY || '');

// Fallback para LibreTranslate
async function translateWithLibre(text: string, targetLang: string): Promise<string> {
  const response = await fetch('https://libretranslate.com/translate', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      q: text,
      source: 'pt',
      target: targetLang,
      format: 'text',
    }),
  });
  const data = await response.json();
  return data.translatedText;
}

// Traduzir texto com fallback
export async function translateText(
  text: string,
  targetLang: string,
  sourceLang: string = 'pt-BR'
): Promise<string> {
  try {
    // Tentar DeepL primeiro
    if (process.env.DEEPL_API_KEY) {
      const result = await translator.translateText(
        text,
        sourceLang as deepl.SourceLanguageCode,
        targetLang as deepl.TargetLanguageCode
      );
      return result.text;
    }

    // Fallback para LibreTranslate
    return await translateWithLibre(text, targetLang);
  } catch (error) {
    console.error('Translation failed:', error);
    return text; // Retorna original se falhar
  }
}

// Cache de traduções (IndexedDB)
interface TranslationCache {
  original: string;
  translated: string;
  targetLang: string;
  timestamp: Date;
}

export async function getCachedTranslation(
  text: string,
  targetLang: string
): Promise<string | null> {
  const db = await openDB('translations');
  const key = `${text}_${targetLang}`;
  const cached = await db.get('cache', key);

  if (cached && Date.now() - cached.timestamp < 7 * 24 * 60 * 60 * 1000) {
    return cached.translated; // Cache válido por 7 dias
  }

  return null;
}

export async function cacheTranslation(
  text: string,
  translated: string,
  targetLang: string
): Promise<void> {
  const db = await openDB('translations');
  const key = `${text}_${targetLang}`;
  await db.put(
    'cache',
    {
      original: text,
      translated,
      targetLang,
      timestamp: Date.now(),
    },
    key
  );
}
```

### 5.2. Componente de Tradução Dinâmica

**components/TranslatableText.tsx**

```typescript
'use client';
import { useState } from 'react';
import { useLocale } from 'next-intl';
import { translateText, getCachedTranslation, cacheTranslation } from '@/lib/translation';

interface Props {
  text: string;
  originalLang?: string;
}

export function TranslatableText({ text, originalLang = 'pt-BR' }: Props) {
  const locale = useLocale();
  const [translated, setTranslated] = useState<string | null>(null);
  const [loading, setLoading] = useState(false);

  // Se já está no idioma original, não mostrar botão
  if (locale === originalLang) {
    return <span>{text}</span>;
  }

  const handleTranslate = async () => {
    setLoading(true);

    // Verificar cache primeiro
    const cached = await getCachedTranslation(text, locale);
    if (cached) {
      setTranslated(cached);
      setLoading(false);
      return;
    }

    // Traduzir via API
    const result = await translateText(text, locale, originalLang);
    setTranslated(result);

    // Cachear resultado
    await cacheTranslation(text, result, locale);
    setLoading(false);
  };

  return (
    <div className="translatable-text">
      <span>{translated || text}</span>
      {!translated && (
        <button
          onClick={handleTranslate}
          disabled={loading}
          className="translate-btn"
        >
          {loading ? '⏳' : '🌐 Traduzir'}
        </button>
      )}
    </div>
  );
}
```

---

## 6. GERAÇÃO DE TRADUÇÕES AI (Build Time)

### 6.1. Script de Tradução Automática

**scripts/generate-translations.ts**

```typescript
import * as fs from 'fs';
import * as path from 'path';
import * as deepl from 'deepl-node';

const translator = new deepl.Translator(process.env.DEEPL_API_KEY!);

async function translateFile(sourcePath: string, targetLang: string): Promise<void> {
  // Ler arquivo fonte (pt-BR)
  const source = JSON.parse(fs.readFileSync(sourcePath, 'utf-8'));
  const translated: any = {};

  // Traduzir recursivamente
  async function translateObject(obj: any, target: any) {
    for (const [key, value] of Object.entries(obj)) {
      if (typeof value === 'string') {
        const result = await translator.translateText(
          value,
          'pt',
          targetLang as deepl.TargetLanguageCode
        );
        target[key] = result.text;
      } else if (typeof value === 'object') {
        target[key] = {};
        await translateObject(value, target[key]);
      }
    }
  }

  await translateObject(source, translated);

  // Salvar arquivo traduzido
  const targetPath = path.join(path.dirname(sourcePath), `${targetLang}.json`);
  fs.writeFileSync(targetPath, JSON.stringify(translated, null, 2));
  console.log(`✓ Generated ${targetLang}.json`);
}

// Gerar traduções para todos os idiomas
async function main() {
  const languages = ['en', 'es', 'fr', 'de'];
  const sourcePath = './messages/pt-BR.json';

  for (const lang of languages) {
    await translateFile(sourcePath, lang);
  }
}

main();
```

**package.json**

```json
{
  "scripts": {
    "translate": "tsx scripts/generate-translations.ts"
  }
}
```

---

## 7. VARIÁVEIS DE AMBIENTE

**.env.local**

```bash
# DeepL API (Free tier: 500k chars/month)
DEEPL_API_KEY=your_deepl_api_key_here

# Ou LibreTranslate (self-hosted)
LIBRETRANSLATE_URL=https://libretranslate.com
```

---

## 8. VERIFICAÇÃO

### 8.1. Testes Manuais

1. **Trocar Idioma**
   - Abrir app em `/`
   - Selecionar idioma no switcher
   - Verificar se URL muda para `/en/`, `/es/`, etc.
   - Verificar se textos mudam

2. **Tradução Dinâmica**
   - Criar anotação em português
   - Trocar para inglês
   - Clicar botão "Translate"
   - Verificar se tradução aparece

3. **Cache**
   - Traduzir texto
   - Recarregar página
   - Verificar se tradução persiste (cache)

### 8.2. Testes Automatizados

```typescript
// __tests__/i18n.test.ts
import { translateText } from '@/lib/translation';

describe('Translation Service', () => {
  it('should translate Portuguese to English', async () => {
    const result = await translateText('Olá', 'en', 'pt-BR');
    expect(result).toBe('Hello');
  });

  it('should cache translations', async () => {
    // Implementar teste de cache
  });
});
```

---

## 9. ROADMAP DE IMPLEMENTAÇÃO

### Fase 1: Setup Básico (1 semana)

- [ ] Instalar next-intl
- [ ] Configurar middleware e i18n.ts
- [ ] Criar estrutura de pastas messages/
- [ ] Traduzir manualmente pt-BR.json → en.json

### Fase 2: Language Switcher (3 dias)

- [ ] Criar componente LanguageSwitcher
- [ ] Integrar em layout principal
- [ ] Testar troca de idiomas

### Fase 3: Tradução AI (1 semana)

- [ ] Configurar DeepL API
- [ ] Implementar serviço de tradução
- [ ] Criar cache de traduções (IndexedDB)
- [ ] Componente TranslatableText

### Fase 4: Geração Automática (3 dias)

- [ ] Script generate-translations.ts
- [ ] Gerar es.json, fr.json, de.json
- [ ] Revisar traduções geradas

### Fase 5: Testes e Polish (3 dias)

- [ ] Testes manuais
- [ ] Testes automatizados
- [ ] Ajustes de UX

**Total:** ~3 semanas

---

## 10. CUSTOS E LIMITES

### DeepL Free Tier

- **500,000 caracteres/mês** grátis
- Estimativa: ~100 páginas de texto
- Suficiente para MVP

### LibreTranslate (Fallback)

- **Gratuito e ilimitado**
- Self-hosted ou API pública
- Qualidade inferior, mas funcional

---

**Este plano está pronto para execução.**  
**Idiomas Tier 1 (manuais): pt-BR, en**  
**Idiomas Tier 2 (AI): es, fr, de, it, ja, zh-CN**
