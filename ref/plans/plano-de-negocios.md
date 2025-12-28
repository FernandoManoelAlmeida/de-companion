# Plano de Negócios - Detetive Existencial Companion App

**Data:** 26 de Dezembro de 2025  
**Versão:** 1.0  
**Tipo:** Progressive Web App (PWA)

---

## 1. SUMÁRIO EXECUTIVO

### 1.1. Visão Geral do Produto

O **Detetive Existencial Companion App** é um Progressive Web App (PWA) desenvolvido para suportar jogadores e narradores do sistema de RPG de mesa "Detetive Existencial", baseado em Disco Elysium. O aplicativo oferece ferramentas digitais completas para criação de personagens, gerenciamento de campanhas, rolagem de dados, e armazenamento de conteúdo.

### 1.2. Proposta de Valor

- **Para Jogadores:** Ficha digital interativa, calculadora automática de testes, histórico de progressão, anotações organizadas
- **Para Narradores:** Gerenciamento de campanhas, NPCs, sessões, eventos aleatórios, e ferramentas narrativas
- **Diferencial:** Sistema offline-first com exportação JSON/MD, sem necessidade de conta, instalável como app nativo

### 1.3. Modelo de Negócio

**Produto Gratuito e Open Source**

- Licença: MIT ou GPL v3
- Monetização: Não aplicável (projeto fan-made)
- Sustentabilidade: Comunidade e contribuições voluntárias

---

## 2. ANÁLISE DE MERCADO

### 2.1. Mercado-Alvo

#### Público Primário

- **Jogadores de RPG de Mesa:** Entusiastas de sistemas narrativos e investigativos
- **Fãs de Disco Elysium:** Comunidade estabelecida que busca experiências similares
- **Idade:** 18-45 anos
- **Perfil:** Gamers, leitores, interessados em narrativas complexas e filosofia

#### Público Secundário

- **Criadores de Conteúdo:** Streamers e podcasters de RPG
- **Game Designers:** Interessados em sistemas narrativos inovadores
- **Comunidade Indie:** Desenvolvedores e artistas independentes

### 2.2. Análise Competitiva

| Concorrente            | Pontos Fortes                      | Pontos Fracos                    | Diferencial do DE Companion               |
| ---------------------- | ---------------------------------- | -------------------------------- | ----------------------------------------- |
| **D&D Beyond**         | Integração oficial, vasto conteúdo | Requer conta, pago, online-only  | Offline-first, gratuito, exportação livre |
| **Roll20**             | Multiplayer, mapas virtuais        | Complexo, requer conta           | Foco em narrativa, simplicidade           |
| **Notion/Google Docs** | Flexível, colaborativo             | Não especializado para RPG       | Ferramentas específicas do sistema        |
| **Planilhas Físicas**  | Tradicional, tangível              | Difícil de organizar, sem backup | Digital, backup automático, calculadora   |

### 2.3. Oportunidades de Mercado

- **Crescimento do RPG de Mesa:** Mercado em expansão pós-pandemia
- **Comunidade Disco Elysium:** Base de fãs engajada e criativa
- **Lacuna de Ferramentas:** Poucos companions para sistemas narrativos indie
- **Tendência PWA:** Crescente adoção de web apps instaláveis

---

## 3. ESTRATÉGIA DE PRODUTO

### 3.1. Funcionalidades Core (MVP)

#### 3.1.1. Criação de Personagem

- Distribuição de 8 pontos em atributos (INT, PSY, FYS, MOT)
- Distribuição de 12 pontos em 24 perícias
- Seleção de Reflexão inicial
- Cálculo automático de Moral e Saúde
- Validação de limites (Sobrecarga Temporal)

#### 3.1.2. Ficha de Personagem Interativa

- Visualização de atributos, perícias e recursos
- Rolagem de dados integrada (2d6 + modificadores)
- Gerenciamento de Moral e Saúde
- Gabinete de Reflexões (slots, processamento, soluções)
- Inventário e dinheiro (R$)
- Histórico de XP e progressão

#### 3.1.3. Sistema de Rolagem

- Rolagem padrão: 2d6 + modificadores
- Rolagem genérica: notação xdY (1d20, 2d6, 3d8+5, etc.)
- Testes Passivos (automáticos)
- Testes Assistidos (3d6, descarta menor)
- Indicação de CD (6-16)
- Histórico de rolagens (padrão e genéricas)

#### 3.1.4. Repositório de Conteúdo

- Biblioteca completa das 24 Vozes (perícias)
- Catálogo de Reflexões (Níveis 1-4)
- Tabelas de referência (CDs, itens, armas)
- Condições e efeitos
- Glossário de termos

#### 3.1.5. Ferramentas do Narrador

- Gerenciamento de campanhas
- Criação e organização de NPCs
- Planejamento de sessões
- Gerador de eventos aleatórios urbanos (d20)
- Rastreamento de Moral/Saúde da Cidade
- Anotações e diário de campanha

#### 3.1.6. Exportação/Importação

- Exportar fichas em JSON e Markdown
- Exportar campanhas completas
- Importar dados salvos
- Backup local automático (LocalStorage/IndexedDB)

#### 3.1.7. Gerador de Personagens Aleatórios

- 5 arquétipos pré-definidos (Detetive, Emocional, Brutamontes, Veloz, Aleatório)
- Distribuição automática de atributos e perícias
- Seleção de Reflexão inicial
- Gerador de nomes brasileiros
- Preview antes de salvar

#### 3.1.8. Modo Multiplayer (Tempo Real)

- **Campanhas compartilhadas** entre múltiplos usuários
- **Sistema de convites** com códigos (ABC-123-XYZ)
- **Sincronização em tempo real** via Firebase
- **Permissões por role** (Narrador, Jogador, Espectador)
- **Sistema de presença** (online/offline)
- **Chat integrado** para comunicação
- **Offline support** com cache local
- **Gratuito** (Firebase Spark plan)

### 3.2. Funcionalidades Futuras (Pós-MVP)

- **Internacionalização (i18n)**:
  - Português (pt-BR) - idioma base
  - Inglês (en) - tradução nativa
  - Espanhol, Francês, Alemão - tradução AI
  - Botão "Traduzir" para conteúdo dinâmico
- Calculadora de combate avançada
- Temas visuais customizáveis
- Integração com Discord/Foundry VTT
- Compartilhamento de imagens na campanha
- Geração de relatórios de sessão

---

## 4. ESTRATÉGIA TECNOLÓGICA

### 4.1. Stack Recomendado

**Next.js 14+ (App Router) com TypeScript**

#### Justificativa

- ✅ **PWA Nativo:** Suporte completo a Service Workers e instalação
- ✅ **SEO:** Renderização server-side para landing page
- ✅ **Performance:** Otimizações automáticas, code splitting
- ✅ **Developer Experience:** Hot reload, TypeScript, React Server Components
- ✅ **Offline-First:** Estratégias de cache robustas
- ✅ **Mobile-Friendly:** Responsivo por padrão

#### Alternativas Descartadas

- **React (Vite):** Menos recursos PWA nativos, sem SSR
- **React Native:** Requer publicação em stores, mais complexo para web
- **Vanilla JS:** Desenvolvimento mais lento, menos manutenível

### 4.2. Arquitetura de Dados

```
IndexedDB (Dexie.js)
├── characters (fichas de personagem)
├── campaigns (campanhas do narrador)
├── sessions (sessões de jogo)
├── npcs (personagens não-jogáveis)
├── notes (anotações gerais)
└── settings (configurações do app)
```

### 4.3. Padrões de Design

- **Atomic Design:** Componentes reutilizáveis (atoms, molecules, organisms)
- **State Management:** Zustand ou Context API
- **Styling:** Tailwind CSS + CSS Modules
- **Forms:** React Hook Form + Zod (validação)

---

## 5. ROADMAP DE DESENVOLVIMENTO

### Fase 1: Fundação (4-6 semanas)

- [ ] Setup do projeto Next.js + TypeScript
- [ ] Configuração PWA (manifest, service worker)
- [ ] Design system e componentes base
- [ ] Estrutura de dados (schemas TypeScript)
- [ ] Implementação IndexedDB (Dexie.js)

### Fase 2: Core Features (6-8 semanas)

- [ ] Criação de personagem completa
- [ ] Ficha interativa com todos os recursos
- [ ] Sistema de rolagem de dados
- [ ] Gabinete de Reflexões
- [ ] Repositório de conteúdo (perícias, reflexões)

### Fase 3: Ferramentas do Narrador (4-6 semanas)

- [ ] Gerenciamento de campanhas
- [ ] Sistema de NPCs
- [ ] Planejamento de sessões
- [ ] Eventos aleatórios
- [ ] Rastreamento da Cidade

### Fase 4: Exportação e Polish (2-4 semanas)

- [ ] Exportação JSON/MD
- [ ] Importação de dados
- [ ] Backup automático
- [ ] Testes de usabilidade
- [ ] Otimizações de performance

### Fase 5: Lançamento (2 semanas)

- [ ] Testes finais
- [ ] Documentação de usuário
- [ ] Deploy (Vercel/Netlify)
- [ ] Divulgação na comunidade

**Total Estimado:** 18-26 semanas (4-6 meses)

---

## 6. ESTRATÉGIA DE DISTRIBUIÇÃO

### 6.1. Hospedagem

- **Plataforma:** Vercel ou Netlify (gratuito para projetos open source)
- **Domínio:** `detetive-existencial.app` ou similar
- **CDN:** Cloudflare para performance global

### 6.2. Instalação

- **Web:** Acesso direto via navegador
- **PWA:** Botão "Instalar App" em Chrome, Edge, Firefox, Brave
- **Mobile:** Instalação via navegador mobile (iOS Safari, Android Chrome)

### 6.3. Divulgação

- **GitHub:** Repositório público com documentação
- **Reddit:** r/rpg_brasil, r/rpg, r/DiscoElysium
- **Discord:** Servidores de RPG de mesa e Disco Elysium
- **YouTube/Twitch:** Parcerias com criadores de conteúdo
- **Itch.io:** Página do projeto com link para o app

---

## 7. MÉTRICAS DE SUCESSO

### 7.1. KPIs Técnicos

- **Performance:** Lighthouse Score > 90
- **Acessibilidade:** WCAG 2.1 AA compliance
- **Offline:** 100% funcional sem internet
- **Instalações PWA:** Meta de 500+ no primeiro ano

### 7.2. KPIs de Engajamento

- **Usuários Ativos:** 1000+ no primeiro ano
- **Retenção:** 40%+ após 30 dias
- **Exportações:** 50%+ dos usuários exportam dados
- **Contribuições:** 10+ contributors no GitHub

### 7.3. KPIs de Comunidade

- **Stars no GitHub:** 100+ no primeiro ano
- **Feedback Positivo:** 80%+ de reviews positivas
- **Conteúdo Gerado:** 5+ streams/vídeos usando o app

---

## 8. RISCOS E MITIGAÇÕES

| Risco                     | Probabilidade | Impacto | Mitigação                                   |
| ------------------------- | ------------- | ------- | ------------------------------------------- |
| **Baixa adoção**          | Média         | Alto    | Marketing focado, parcerias com influencers |
| **Complexidade técnica**  | Baixa         | Médio   | Stack moderno, documentação robusta         |
| **Bugs críticos**         | Média         | Alto    | Testes automatizados, beta testing          |
| **Abandono do projeto**   | Baixa         | Alto    | Documentação clara para novos maintainers   |
| **Questões de copyright** | Baixa         | Médio   | Licença fan-made clara, sem monetização     |

---

## 9. SUSTENTABILIDADE

### 9.1. Modelo Open Source

- **Licença:** MIT (permissiva) ou GPL v3 (copyleft)
- **Contribuições:** Guia de contribuição, code of conduct
- **Governança:** Maintainer principal + core contributors

### 9.2. Custos Operacionais

- **Hospedagem:** R$ 0 (Vercel/Netlify free tier)
- **Domínio:** R$ 40/ano (opcional)
- **Desenvolvimento:** Voluntário (open source)

### 9.3. Crescimento da Comunidade

- **Documentação:** Wiki completo, tutoriais em vídeo
- **Suporte:** GitHub Issues, Discord server
- **Roadmap Público:** Transparência nas decisões

---

## 10. CONCLUSÃO

O **Detetive Existencial Companion App** preenche uma lacuna importante no ecossistema de RPG de mesa narrativo, oferecendo ferramentas digitais especializadas para um sistema único e inovador. Com uma estratégia de desenvolvimento focada, tecnologia moderna (Next.js PWA), e modelo open source sustentável, o projeto tem potencial para se tornar referência na comunidade de RPG indie.

### Próximos Passos Imediatos

1. Validar plano de viabilidade técnica
2. Criar design mockups (UI/UX)
3. Iniciar desenvolvimento do MVP
4. Estabelecer presença na comunidade (GitHub, Discord)

---

**Documento preparado para:** Desenvolvimento AI-Guided  
**Aprovação necessária:** Stakeholder (Fernando)  
**Revisão:** Trimestral
