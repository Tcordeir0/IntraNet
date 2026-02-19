# Documentação Técnica

<p align="right">
  <a href="./DOCUMENTATION.md">🇺🇸 English Version</a>
</p>

> **Desenvolvido por:** Talys Matheus Cordeiro Silva (Tcordeiro) — Showcase de Portfólio Profissional

---

## Índice

1. [Visão Geral da Arquitetura](#1-visão-geral-da-arquitetura)
2. [Fluxo de Dados](#2-fluxo-de-dados)
3. [Modelo de Segurança (RBAC + RLS)](#3-modelo-de-segurança-rbac--rls)
4. [Módulos Principais](#4-módulos-principais)
5. [Implementação Mobile & PWA](#5-implementação-mobile--pwa)
6. [Tempo Real & Performance](#6-tempo-real--performance)
7. [Edge Functions](#7-edge-functions)
8. [Matriz de Permissões](#8-matriz-de-permissões)

---

## 1. Visão Geral da Arquitetura

O sistema segue uma arquitetura **serverless-first** utilizando o Supabase como camada principal de Backend-as-a-Service (BaaS):

```
┌─────────────────────────────────────────────────────────────┐
│          Camada Cliente (React 18 + TypeScript)              │
│         Vite · Tailwind · shadcn/ui · React Query            │
└──────────────────────────┬──────────────────────────────────┘
                           │
            ┌──────────────┼──────────────┐
            │              │              │
     ┌──────▼──────┐ ┌─────▼──────┐ ┌────▼───────┐
     │   API REST  │ │  Realtime  │ │  WebSocket  │
     │ (PostgREST) │ │  (LISTEN/  │ │ (Telemetria)│
     └──────┬──────┘ │  NOTIFY)   │ └────┬───────┘
            │        └─────┬──────┘      │
     ┌──────▼──────────────▼─────────────▼──────┐
     │              Plataforma Supabase           │
     │   PostgreSQL · Auth · Storage · Edge Fn   │
     └───────────────────────────────────────────┘
```

### Resumo do Stack Técnico

| Camada | Tecnologia | Propósito |
|---|---|---|
| Frontend | React 18 + TypeScript | UI tipada e reativa |
| Build | Vite 5 | HMR rápido, saída ESM |
| Estilização | Tailwind CSS + shadcn/ui | Sistema de design |
| Estado | React Query (TanStack) | Estado server-side + cache |
| Banco de Dados | PostgreSQL (Supabase) | Armazenamento principal |
| Auth | Supabase Auth (JWT) | Gerenciamento de sessão |
| Tempo Real | Supabase Realtime | Subscriptions WebSocket |
| Edge | Deno (Edge Functions) | Computação serverless |
| Telemetria | Agente WebSocket em Node.js | Monitoramento de hardware |
| Gráficos | Recharts | Visualização de dados |
| DnD | @hello-pangea/dnd | Drag-and-drop do Kanban |
| Mobile | Service Worker + Manifest | Suporte PWA |

---

## 2. Fluxo de Dados

### Fluxo do Ciclo de Vida de um Chamado

```
Usuário → Formulário Novo Chamado → Setor/Tópico → JSONB Estruturado
                                                           ↓
                                                    PostgreSQL RLS
                                                           ↓
                                       +------------------+------------------+
                                       |                                     |
                                Notify Realtime                       Edge Function
                               (Atualização UI Push)              (send-notification)
                                       |                                     |
                                Kanban Board                     Push + Dispatch Webhook
                               atualiza automático               (email do grupo + externo)
```

### Extração Inteligente de Campos

Os dados de formulário dinâmico são persistidos na coluna `dados_formulario` (JSONB), possibilitando:

```typescript
// Prioridade 1: JSONB estruturado do formulário dinâmico
const placa = dados_formulario?.placa
           || dados_formulario?.placaVeiculo

// Prioridade 2: Regex inteligente em bullets de descrição legada
|| extractFromBullets(descricao, 'Placa')

// Normalizado para busca (remove acentos, hífens, espaços)
|| normalized(rawText)
```

---

## 3. Modelo de Segurança (RBAC + RLS)

### Hierarquia de Papéis

| Papel | Código | Nível de Acesso |
|---|---|---|
| Administrador | `admin` | Total — todos os dados, todos os usuários |
| Desenvolvedor | `dev` | Total + painéis técnicos |
| Operador | `operador` | Chamados próprios + atribuídos |
| Usuário | `usuario` | Apenas envios próprios |
| Visualizador | `visualizar_kanban` | Acesso leitura ao Kanban |

### Conceito de Row-Level Security (RLS)

Todas as tabelas aplicam políticas RLS que avaliam o JWT autenticado para determinar a visibilidade das linhas:

- **Usuários** veem apenas seus próprios chamados, a menos que sejam atribuídos como operador.
- **Operadores** veem chamados do próprio setor ou atribuídos diretamente a eles.
- **Admins** bypassam todas as restrições com acesso total de leitura/escrita.
- **Visualizadores** acessam o Kanban somente leitura — chat e drag-and-drop são bloqueados na UI.

### Resolução Segura de Perfis

Para resolver nomes de exibição de usuários sem expor dados privados (email/telefone), uma **Edge Function** bypassa o RLS usando a chave de serviço, retornando apenas `{ id, nome }` — nunca dados completos do perfil.

---

## 4. Módulos Principais

### 4.1 Chamados & Kanban

O sistema de chamados suporta dois quadros Kanban independentes com controles de acesso separados:

- **Kanban Padrão** — para operadores de suporte geral
- **Kanban Operacional** — para operadores logísticos/frete (`kanban_acesso = 'OPERACIONAL'`)

Os cards exibem campos extraídos dinamicamente:

| Campo | Origem |
|---|---|
| Placa | `dados_formulario.placa` ou extração de bullets |
| CT-e / Nota de Frete | `dados_formulario.cte` ou extração de bullets |
| Cliente | `dados_formulario.cliente` ou extração de bullets |
| Status SLA | Calculado de `created_at` vs limiar de 3h |
| Responsável | Nome do operador atribuído com avatar |

### 4.2 Sistema SLA

Substituiu o campo manual legado de "Urgência" por uma métrica automática baseada em tempo:

```typescript
export const calcularStatusPrazo = (
  created_at: string,
  finalizado_at: string | null
): 'No Prazo' | 'Em Atraso' => {
  const prazoLimite = new Date(
    new Date(created_at).getTime() + 3 * 60 * 60 * 1000 // 3 horas
  );
  if (finalizado_at) return 'No Prazo';
  return new Date() > prazoLimite ? 'Em Atraso' : 'No Prazo';
};
```

Exibido como badge colorido (verde/vermelho) em todos os cards do Kanban e no dashboard de relatórios.

### 4.3 Monitoramento de Ativos

Telemetria de hardware em tempo real via abordagem híbrida:

- **PostgreSQL**: Metadados persistentes de ativos (specs, localização, propriedade)
- **WebSocket Server**: CPU/RAM/Disco em tempo real via agentes Node.js locais
- **UI**: Visualização em árvore organizada por pastas com contadores online/offline por setor

### 4.4 Analytics & Relatórios

Métricas KPI computadas via queries server-side no PostgreSQL e exibidas via Recharts:

- Gráfico de evolução diária (chamados criados vs finalizados)
- Donut chart de conformidade SLA (distribuição No Prazo / Em Atraso)
- Gráficos de barras de demanda por setor/filial
- Usuários mais ativos por filial
- Cálculo de MTTR (Tempo Médio de Resolução)

### 4.5 Estoque

Rastreamento de materiais e ativos de TI com:

- Organização por sala (salas de servidores, almoxarifados, escritórios)
- Rastreamento de quantidade em tempo real
- Exportação Excel com nome de arquivo carimbado pelo usuário (`EstoqueSala_AAAA-MM-DD_<usuario>.xlsx`)

---

## 5. Implementação Mobile & PWA

### Estratégia do Service Worker

```
Requisição → Rede disponível?
              Sim → Servir + atualizar Cache
              Não → Servir do Cache
                     Sem Cache → Página Fallback
```

### Web App Manifest

```json
{
  "name": "IntraNet",
  "short_name": "IntraNet",
  "display": "standalone",
  "orientation": "portrait",
  "theme_color": "#0f172a",
  "background_color": "#0f172a",
  "icons": [
    { "src": "/icons/icon-192.png", "sizes": "192x192", "type": "image/png" },
    { "src": "/icons/icon-512.png", "sizes": "512x512", "type": "image/png" }
  ]
}
```

### Suporte Safe Area iOS

```css
/* iPhone notch / Dynamic Island */
padding-top: env(safe-area-inset-top);
padding-bottom: env(safe-area-inset-bottom);
padding-left: env(safe-area-inset-left);
padding-right: env(safe-area-inset-right);
```

> Sem uso de `100vw` ou `w-screen` — usa `100%` com `overflow: hidden` para evitar sangramento do bounce scroll do iOS.

### Implementação dos Gestos Swipe

```typescript
// Hook customizado: useSwipeGesture
// - Touch start: registra posição X se dentro de 40px da borda esquerda
// - Touch move: calcula delta, atualiza CSS transform para feedback visual
// - Touch end: se delta > 80px, abre sidebar; caso contrário, retorna ao lugar
// Listeners passivos para desempenho de rolagem em 60fps
```

### Kanban Responsivo

- Container com scroll horizontal e colunas `min-width: 800px`
- `scroll-snap-type: x mandatory` para navegação fluida entre colunas no touch
- Drag-and-drop funcional no touch (@hello-pangea/dnd com suporte a touch)

---

## 6. Tempo Real & Performance

### Estabilidade de Subscriptions

Subscriptions do chat são estabilizadas com `useRef` para evitar loops causados por mudanças de referência de objeto:

```typescript
// Problemático: referência instável faz useEffect re-executar
useEffect(() => { subscribe(user) }, [user])

// Estável: extrai primitivo, usa ref para não-primitivo
const userIdRef = useRef(user?.id)
useEffect(() => { subscribe(userIdRef.current) }, [user?.id])
```

### Debounce no Kanban

Atualizações Realtime com debounce de 500ms para evitar tempestades de requisições de eventos DB consecutivos rápidos:

```typescript
const debouncedRefetch = useCallback(
  debounce(() => refetchChamados(), 500), []
);
channel.on('postgres_changes', { event: '*', table: 'chamados' }, debouncedRefetch);
```

### Timeouts de Segurança

Operações críticas (envio de mensagem, upload de arquivo, transferência de chamado) incluem timeouts de segurança de 15 segundos para desbloquear a UI caso o servidor falhe em responder.

---

## 7. Edge Functions

Implantadas no runtime Deno, invocadas via `supabase.functions.invoke()`:

| Função | Propósito |
|---|---|
| `send-notification` | Despacha push notifications + webhook externo com email do grupo |
| `admin-create-user` | Cria usuários com service role — suporta todos os tipos de perfil incluindo `dev` |
| `get-public-profile` | Resolve nome de exibição do usuário com segurança sem expor PII |

Todas as funções exigem JWT válido no header `Authorization` (verify_jwt = true).

---

## 8. Matriz de Permissões

| Funcionalidade | Admin | Dev | Operador | Usuário | Visualizador |
|---|---|---|---|---|---|
| Ver todos os chamados | ✅ | ✅ | Próprio setor | Apenas próprios | ✅ (leitura) |
| Criar chamados | ✅ | ✅ | ✅ | ✅ | ❌ |
| Atribuir/transferir chamados | ✅ | ✅ | ✅ | ❌ | ❌ |
| Finalizar chamados | ✅ | ✅ | ✅ | ❌ | ❌ |
| Arrastar cards Kanban | ✅ | ✅ | ✅ | ❌ | ❌ |
| Acessar painéis Admin | ✅ | ✅ | ❌ | ❌ | ❌ |
| Ver analytics | ✅ | ✅ | ✅ | ❌ | ❌ |
| Exportar Excel | ✅ | ✅ | ✅ | ❌ | ❌ |
| Gerenciar usuários | ✅ | ✅ | ❌ | ❌ | ❌ |
| Monitoramento de ativos | ✅ | ✅ | ✅ | ❌ | ❌ |

---

*Documentação mantida por **Talys Matheus Cordeiro Silva (Tcordeiro)***
