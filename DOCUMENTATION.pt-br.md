<p align="right">
  <a href="./DOCUMENTATION.md">🇺🇸 English Version</a>
</p>

# Documentação Técnica — Logistics Management Hub

> **Status do Projeto:** Pronto para Produção (Versão Showcase)  
> **Arquitetura:** Serverless + Híbrido em Tempo Real  
> **Versão:** 1.0.7

---

## 1. Arquitetura de Fluxo de Dados

O sistema foi projetado para **baixa latência** e **alta consistência**:

1. **Frontend (React 18):** Comunica via REST para CRUD padrão e WebSockets para telemetria.
2. **Supabase Realtime:** Sincroniza mudanças no banco de dados (tickets, mensagens) em todos os clientes em <1s usando `LISTEN/NOTIFY` do PostgreSQL.
3. **Camada WebSocket:** Processa dados de telemetria de alta frequência vindos de agentes de hardware remotos.
4. **Edge Functions:** Processam lógica pesada (geocodificação, notificações em massa, dispatch de webhooks) fora da thread principal.

### Diagrama de Fluxo
```
Ação do Usuário
    │
    ├──► REST (PostgREST) ──► PostgreSQL ──► Validação RLS
    │                                │
    │                                └──► Realtime ──► Todos os clientes conectados
    │
    └──► Edge Function ──► APIs Externas / Webhooks
```

---

## 2. Schema do Banco de Dados (Conceitual)

### Entidades Principais

| Entidade | Descrição |
|---|---|
| **Profiles** | Metadados estendidos do usuário com atributos por função (Admin, Operador, Dev, Usuário) |
| **Tickets (Chamados)** | Fluxo de trabalho baseado em máquina de estados com transições de status e rastreamento de SLA |
| **Ativos** | Rastreamento em tempo real de especificações de hardware, métricas de saúde e telemetria |
| **Setores** | Organização hierárquica de unidades de negócio e associações departamentais |
| **Grupos** | Grupos de e-mail para roteamento de notificações |
| **Estoque** | Materiais, equipamentos e recursos com controle de quantidade |

### Implementação em Tempo Real
O sistema utiliza `LISTEN/NOTIFY` do PostgreSQL através do Supabase Realtime para enviar atualizações diretamente à interface sem polling — garantindo sincronização instantânea para quadros Kanban, mensagens de chat e mudanças de status de tickets.

### Exemplos de Row-Level Security

```sql
-- Usuários só podem ver seus próprios tickets
CREATE POLICY "Usuários visualizam seus tickets"
ON tickets FOR SELECT
USING (auth.uid() = requester_id);

-- Operadores podem ver tickets dos setores atribuídos
CREATE POLICY "Operadores visualizam tickets do setor"
ON tickets FOR SELECT
USING (
  EXISTS (
    SELECT 1 FROM user_sectors
    WHERE user_id = auth.uid() AND sector_id = tickets.sector_id
  )
);

-- Admins têm acesso total
CREATE POLICY "Admins têm acesso completo"
ON tickets FOR ALL
USING (
  EXISTS (
    SELECT 1 FROM profiles
    WHERE id = auth.uid() AND role = 'admin'
  )
);
```

---

## 3. Edge Functions

As Edge Functions são implantadas como funções serverless em Deno via Supabase:

| Função | Finalidade |
|---|---|
| `send-notification` | Dispara notificações push + chamadas de webhook externo |
| `admin-create-user` | Cria usuários com atribuição de perfil RBAC (ignora restrições de anon) |
| `telemetry-processor` | Agrega e armazena dados de telemetria de hardware |

---

## 4. Módulos Avançados

### 4.1 Motor Kanban

Interface de arrastar e soltar sofisticada construída com `@hello-pangea/dnd` que gerencia:
- **Atualizações otimistas de UI** — Feedback visual imediato antes da confirmação do servidor.
- **Atribuição automática** na transição do ticket para "Em Andamento".
- **Modais de justificativa obrigatória** para estados terminais (Cancelado / Finalizado).
- **Colunas controladas por acesso** — Restrições por função em quais transições de status são permitidas.

### 4.2 Sistema de Telemetria de Ativos

Implementação WebSocket customizada recebendo dados de alta frequência de agentes de hardware:
- Tendências de uso de **CPU / RAM / Disco** com gráficos históricos.
- **Heartbeats de status** Online/Offline com limiares de timeout configuráveis.
- **Mapeamento geográfico** baseado em metadados de rede (localização por IP).
- **Limiares de alerta** — Notificações automáticas quando métricas excedem limites definidos.

### 4.3 Construtor de Formulários Dinâmicos

Um motor de formulários orientado a JSON que permite que administradores criem templates de tickets complexos com lógica condicional **sem escrever nenhum código novo**. Suporta:
- Campos de texto, dropdowns, checkboxes, anexos de arquivos.
- Regras de visibilidade condicional (mostrar campo X somente se campo Y for igual a Z).
- Validação de campos obrigatórios com mensagens de erro personalizadas.

### 4.4 Chat Interno (Tempo Real)

Mensagens em tempo real integradas ao contexto de cada ticket:
- Mensagens armazenadas no PostgreSQL com assinaturas Realtime.
- Suporta anexos de arquivos e confirmações de leitura.
- Escopado por ticket — cada conversa é isolada ao seu contexto de chamado.

---

## 5. Indicadores-Chave de Desempenho (KPIs)

O dashboard de analytics fornece insights em tempo real:

| KPI | Definição |
|---|---|
| **MTTR** | Mean Time to Repair — tempo médio de resolução por ticket |
| **Tendências de Volume** | Gráficos de distribuição semanal e mensal de tickets |
| **Conformidade com SLA** | Alertas visuais para tickets que ultrapassam os limiares de SLA por prioridade |
| **Pontuação de Saúde de Ativos** | Métrica agregada de saúde em todo o hardware monitorado |
| **Abertos / Em Andamento / Fechados** | Distribuição de status de tickets em tempo real |

---

## 6. Segurança e Conformidade

### Fluxo de Autenticação
1. Usuário envia credenciais → Supabase Auth valida e emite JWT.
2. JWT incluído em todas as requisições subsequentes da API.
3. Políticas de RLS do PostgreSQL avaliam `auth.uid()` em cada query — **zero bypass possível** pelo cliente.

### Matriz de Permissões

| Ação | Usuário | Operador | Dev | Admin |
|---|---|---|---|---|
| Ver próprios tickets | ✅ | ✅ | ✅ | ✅ |
| Ver tickets do setor | ❌ | ✅ | ✅ | ✅ |
| Criar tickets | ✅ | ✅ | ✅ | ✅ |
| Gerenciar usuários | ❌ | ❌ | ✅ | ✅ |
| Acessar analytics | ❌ | ❌ | ✅ | ✅ |
| Configuração do sistema | ❌ | ❌ | ❌ | ✅ |

---

*Este documento faz parte de um portfólio técnico. Para consultas sobre a implementação completa, entre em contato com o desenvolvedor via [GitHub](https://github.com/Tcordeir0).*
