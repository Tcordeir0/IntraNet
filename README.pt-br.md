<p align="right">
  <a href="./README.md">🇺🇸 English Version</a>
</p>

<h1 align="center">IntraNet</h1>
<p align="center"><em>Uma plataforma corporativa profissional para gerenciamento de operações logísticas em larga escala.</em></p>

<p align="center">
  <img src="https://img.shields.io/badge/Status-Pronto%20para%20Produção-brightgreen?style=for-the-badge" />
  <img src="https://img.shields.io/badge/Versão-1.0.7-blue?style=for-the-badge" />
  <img src="https://img.shields.io/badge/Licença-Proprietária-red?style=for-the-badge" />
  <img src="https://img.shields.io/badge/Showcase-Portfólio-orange?style=for-the-badge" />
</p>

<p align="center">
  <img src="https://img.shields.io/badge/React-18-61DAFB?style=flat-square&logo=react&logoColor=white" />
  <img src="https://img.shields.io/badge/TypeScript-5.x-3178C6?style=flat-square&logo=typescript&logoColor=white" />
  <img src="https://img.shields.io/badge/Supabase-BaaS-3ECF8E?style=flat-square&logo=supabase&logoColor=white" />
  <img src="https://img.shields.io/badge/Vite-5.x-646CFF?style=flat-square&logo=vite&logoColor=white" />
  <img src="https://img.shields.io/badge/TailwindCSS-3.x-38BDF8?style=flat-square&logo=tailwindcss&logoColor=white" />
  <img src="https://img.shields.io/badge/PWA-Pronto-5A0FC8?style=flat-square&logo=pwa&logoColor=white" />
  <img src="https://img.shields.io/badge/Mobile-Responsivo-green?style=flat-square&logo=android&logoColor=white" />
</p>

---

## 📌 Visão Geral

**IntraNet** é uma plataforma corporativa profissional projetada para gerenciamento de operações em larga escala, centralizando fluxos de trabalho integrados para:

- 🎫 **Sistema de Chamados** — Gerenciamento completo de tickets com fluxos Kanban visuais e rastreamento de SLA por prioridade.
- 💻 **Monitoramento de Ativos de TI** — Monitoramento em tempo real de hardware e dispositivos com telemetria via agentes WebSocket.
- 👥 **Gestão de Usuários e Departamentos** — Gerenciamento centralizado de colaboradores, permissões e hierarquia de setores.
- 📦 **Controle de Estoque** — Rastreamento em tempo real de materiais, equipamentos e recursos.
- 📊 **Analytics Executivo** — Dashboards de desempenho e métricas operacionais (MTTR, SLA, Tendências de Volume).
- 🔔 **Notificações Integradas** — Notificações push nativas e webhooks externos para alertas entre sistemas.
- 💬 **Chat Interno** — Mensagens em tempo real integradas diretamente ao fluxo de chamados.

### 🎯 Objetivo Principal

Servir como um **hub unificado** para comunicação e gerenciamento operacional, consolidando múltiplos fluxos de trabalho em uma interface intuitiva e responsiva — eliminando a necessidade de diversas ferramentas desconectadas.

---

## 📱 Suporte Mobile & PWA

A plataforma foi construída com mentalidade **mobile-first** e é totalmente instalável como Progressive Web App:

| Recurso | Detalhes |
|---|---|
| **PWA Instalável** | Funciona como app nativo no Android e iOS via prompt do navegador |
| **Suporte Offline** | Service Worker com estratégia network-first + fallback por cache |
| **Gestos Touch** | Swipe da borda esquerda para abrir/fechar sidebar |
| **Safe Area iOS** | Suporte a notch e Dynamic Island via `env(safe-area-inset-*)` |
| **Layout Responsivo** | Sidebar adaptativa (drawer no mobile, fixa no desktop) |
| **Kanban por Swipe** | Navegação entre colunas via swipe com scroll-snap |
| **Toasts Mobile** | Notificações com swipe-to-dismiss via Sonner |
| **Cards Adaptativos** | Lista de tickets em cards no mobile vs tabela no desktop |

> O sistema roda perfeitamente em **Android, iOS (iPhone/iPad) e todas as telas de 320px a 1920px+**.

---

## 🖼️ Preview Visual

A plataforma possui uma interface moderna com suporte completo ao **modo claro/escuro** e **responsividade mobile**.

### 🖥️ Desktop

<p align="center">
  <img src="screenshots/preview-1.jpg" width="400" alt="Visão Geral do Dashboard" />
  <img src="screenshots/preview-2.jpg" width="400" alt="Histórico e SLA" />
</p>
<p align="center">
  <img src="screenshots/preview-3.jpg" width="400" alt="Relatórios e Analytics" />
  <img src="screenshots/preview-4.jpg" width="400" alt="Monitoramento de Ativos" />
</p>
<p align="center">
  <img src="screenshots/preview-5.jpg" width="400" alt="Formulário de Novo Chamado" />
  <img src="screenshots/preview-6.jpg" width="400" alt="Gestão de Usuários" />
</p>
<p align="center">
  <img src="screenshots/preview-7.jpg" width="400" alt="Modal de Histórico de Versões" />
  <img src="screenshots/preview-8.jpg" width="400" alt="Perfil do Usuário" />
</p>
<p align="center">
  <img src="screenshots/preview-9.jpg" width="400" alt="Inventário da Sala de TI" />
</p>

### 📱 Mobile (iPhone — PWA)

> Todas as telas capturadas rodando como PWA instalado no iPhone, demonstrando o modo escuro, layouts responsivos e navegação otimizada para toque.

<p align="center">
  <img src="screenshots/mobile-1.jpg" width="180" alt="Tela de Login — Dark Mode" />
  <img src="screenshots/mobile-2.jpg" width="180" alt="Sidebar Drawer — Navegação por Swipe" />
  <img src="screenshots/mobile-3.jpg" width="180" alt="Dashboard — Cards de KPI" />
</p>
<p align="center">
  <img src="screenshots/mobile-4.jpg" width="180" alt="Dashboard — Tendência Semanal & Donut SLA" />
  <img src="screenshots/mobile-5.jpg" width="180" alt="Novo Chamado — Mobile" />
  <img src="screenshots/mobile-6.jpg" width="180" alt="Monitoramento de Ativos — Mobile" />
</p>

---

## 🏗️ Arquitetura do Sistema

A plataforma segue uma abordagem moderna **serverless-first** combinada com capacidades em tempo real:

```
┌─────────────────────────────────────────────────────┐
│                  React 18 + TypeScript               │
│         (Vite + Tailwind + shadcn/ui + PWA)          │
└────────────────────────┬────────────────────────────┘
                         │
           ┌─────────────┼─────────────┐
           │             │             │
    ┌──────▼──────┐ ┌────▼────┐ ┌─────▼──────┐
    │   API REST  │ │Realtime │ │  WebSocket  │
    │  (PostgREST)│ │(LISTEN/ │ │  (Telemetria│
    └──────┬──────┘ │NOTIFY)  │ └─────┬──────┘
           │        └────┬────┘       │
    ┌──────▼──────────────▼───────────▼──────┐
    │         Supabase (PostgreSQL + Auth)    │
    │              + Edge Functions           │
    └────────────────────────────────────────┘
```

| Camada | Tecnologia |
|---|---|
| Frontend | React 18, TypeScript, Vite |
| Estilização | Tailwind CSS, shadcn/ui, Material-UI |
| Estado | React Query (TanStack) |
| Backend (BaaS) | Supabase (PostgreSQL, Auth, Storage, Realtime) |
| Computação | Edge Functions (Deno Runtime) |
| Telemetria | WebSocket Server customizado em Node.js |
| Gráficos | Recharts |
| Drag-and-Drop | @hello-pangea/dnd |
| Mobile / PWA | Service Worker, Web App Manifest |

---

## 💻 Stack Técnico

### Frontend
- **React 18 & TypeScript:** Desenvolvimento de UI robusto e tipado.
- **React Query:** Gerenciamento avançado de estado server-side com cache, refetching em background e atualizações otimistas.
- **Tailwind CSS & shadcn/ui:** Biblioteca de componentes consistente, acessível e de alta performance.
- **Recharts:** Visualização de dados complexos para dashboards operacionais e KPIs.
- **@hello-pangea/dnd:** Experiência fluida de arrastar e soltar para o quadro Kanban.
- **Sonner:** Notificações toast com swipe-to-dismiss nativo no mobile.

### Backend & Infraestrutura
- **Supabase Cloud:** Banco de dados, autenticação, storage e engine de tempo real integrados.
- **PostgreSQL RLS:** Row-Level Security impondo isolamento de dados multi-tenant no nível do banco.
- **Engine em Tempo Real:** Supabase Realtime com subscriptions com debounce para atualizações instantâneas da UI.
- **Edge Computing:** Funções serverless em Deno para lógica de negócio crítica (notificações, geocodificação, resolução segura de perfis).
- **WebSocket Server:** Agente Node.js customizado para dados de telemetria de hardware em alta frequência.
- **Service Worker:** Implementação manual com estratégia network-first + fallback por cache para suporte offline.

---

## 🔐 Segurança

| Recurso | Implementação |
|---|---|
| **RBAC** | Permissões granulares para Admin, Operador, Dev e Usuário |
| **Autenticação JWT** | Sessões seguras gerenciadas pelo Supabase Auth |
| **Row-Level Security** | Políticas aplicadas diretamente no nível do PostgreSQL |
| **Criptografia de Dados** | Tratamento seguro de telemetria e dados sensíveis |
| **Validação de Entrada** | Validação de schemas Zod em todos os formulários |
| **Modo Somente Visualização** | Chat e drag-and-drop bloqueados para usuários em modo leitura do Kanban |

---

## 📋 Módulos Principais

| Módulo | Descrição | Status |
|---|---|---|
| 🎫 Chamados / Kanban | Gerenciamento completo do ciclo de vida com SLA | ✅ Produção |
| 💻 Telemetria de Ativos | Monitoramento de hardware em tempo real via WebSocket | ✅ Produção |
| 👥 Gestão de Usuários | RBAC + hierarquia departamental | ✅ Produção |
| 📊 Dashboard Analytics | KPIs, MTTR, conformidade com SLA | ✅ Produção |
| 🔔 Notificações | Push + integração com Webhook | ✅ Produção |
| 💬 Chat Interno | Mensagens em tempo real por chamado | ✅ Produção |
| 📦 Estoque | Rastreamento de materiais + exportação Excel | ✅ Produção |
| 📱 PWA | App instalável com suporte offline | ✅ Produção |

---

## 🖼️ Nota de Portfólio

> **Nota:** Este repositório é um **showcase** de design arquitetural e proficiência técnica. O código-fonte da lógica de negócio principal, algoritmos proprietários e implementações de servidor privado estão em um **repositório privado** para proteger a propriedade intelectual.

**O que está incluído aqui:**
- ✅ Documentação técnica detalhada
- ✅ Diagramas de arquitetura do sistema
- ✅ Especificações do stack tecnológico
- ✅ Previews visuais e screenshots
- ✅ Arquivos de configuração demonstrando expertise em setup de ambiente

**O que está no repositório privado:**
- 🔒 Código-fonte completo da aplicação
- 🔒 Implementações das Edge Functions
- 🔒 Scripts de migração do banco de dados
- 🔒 Implementação do servidor WebSocket

---

## 📞 Contato

> Interessado na implementação completa ou tem alguma dúvida?

**Talys Matheus Cordeiro Silva (Tcordeiro)**  
Sinta-se à vontade para entrar em contato pelo [GitHub](https://github.com/Tcordeir0) ou conectar-se no [LinkedIn](https://www.linkedin.com/in/thalescordeiro/).

---

## 📜 Licença

Copyright © 2026 Talys Matheus Cordeiro Silva — Todos os direitos reservados.  
Este projeto é para **fins de demonstração apenas**. Cópia, modificação, distribuição ou uso não autorizados deste projeto, em parte ou no todo, são estritamente proibidos.

Veja [LICENSE](./LICENSE) para mais detalhes.
