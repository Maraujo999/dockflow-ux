# requests-owner — Figma-ready prototype

Fonte: `specs/screens/requests-owner.yaml`  
Referências: `design/figma/tokens.json`, `design/figma/01-components.md`

## 1) Wireframe textual MOBILE (390x844)

- **AppShell**
  - Topbar: "Requests"
- **Tabela/lista de requests (adaptada para cards no mobile)**
  - Campos por item: `requestId`, `startAt`, `carrier`, `vehicle`, `fitStatus`, `reason`
  - Badge de expiração quando `hold_expires_at` expirado: **expired**
- **Ações por item**
  - `ApproveBtn`
  - `RejectBtn`
  - `ProposeBtn`
- **Modais**
  - `RejectModal` com `reasonOptional`
  - `ProposeNewTimeModal` com seleção de `slotId` (+ noteOptional)
- **Estados**
  - loading: skeleton-table
  - empty: "No requests"
  - error: retry-inline
- **Feedback**
  - toasts: `approved`, `rejected`, `proposed`, `error_default`

## 2) Delta DESKTOP (1440x900)

- Tabela completa com colunas explícitas e ações na última coluna.
- Paginação por cursor no rodapé (next/prev).
- Modais centralizados com foco em decisão rápida.
- Requests expirados destacados visualmente e com Approve desabilitado.

## 3) Spec Figma

- **Pages**
  - `Manager / Requests Owner`
- **Frames**
  - `Requests / Mobile`
  - `Requests / Desktop`
  - `Reject Modal`
  - `Propose Modal`
  - variações loading/empty/error/success
- **Grid**
  - Mobile: 4 col, gutter 16, margin 16
  - Desktop: 12 col, gutter 24, margin 32
- **Auto-layout**
  - Mobile: lista vertical de cards (gap 12)
  - Desktop: table + toolbar/pagination
- **Componentes + variantes**
  - AppShell
  - Table/List + row actions
  - Badge status (`fitStatus`, `expired`)
  - Button (primary/secondary/destructive/ghost + loading/disabled)
  - Modal `RejectModal`
  - Modal `ProposeNewTimeModal` (com slots disponíveis)
  - Toast success/error
- **Estados obrigatórios**
  - loading: skeleton-table
  - empty: mensagem sem requests
  - error: retry-inline
  - success: toast e atualização da linha

## 4) Mapa de interações

- Abrir tela → query `requests` com cursor/limit
- Tap `Approve`:
  - se `expired`: bloqueia ação + tooltip/mensagem
  - se válido: mutation `approve` → toast `approved` + refresh cursor atual
- Tap `Reject` → abre `RejectModal` → submit `reject` → toast `rejected`
- Tap `Propose` → abre modal, carrega `freeSlotsForProposal` → submit `proposeNewTime` → toast `proposed`
- Falha de rede/mutation → toast `error_default` + retry inline

## 5) Copy final

### PT
- Empty: **No requests**
- Badge: **expired**
- Ações: **Approve**, **Reject**, **Propose new time**
- Modal Reject: **Reason (optional)**
- Modal Propose: **Select a free slot**
- Toasts: **Request approved**, **Request rejected**, **New time proposed**, **Something went wrong**

### ES (interno opcional)
- Empty: **No hay solicitudes**
- Badge: **expirada**
- Ações: **Aprobar**, **Rechazar**, **Proponer nuevo horario**
- Modal Reject: **Motivo (opcional)**
- Modal Propose: **Selecciona un slot libre**
- Toasts: **Solicitud aprobada**, **Solicitud rechazada**, **Nuevo horario propuesto**, **Algo salió mal**

## 6) Dados mínimos por tela (anti-overfetch)

- `requests` (cursor-based):
  - `id`, `requestId`, `startAt`, `carrier`, `vehicle`, `fitStatus`, `reason`, `hold_expires_at`
  - `nextCursor`, `hasMore`
- `freeSlotsForProposal` (sob demanda no modal):
  - `slotId`, `startAt`, `endAt`, `resourceId`
- Não carregar slots livres globalmente para todas as linhas; apenas quando modal abrir.

## 7) Critérios de aceite (UX/perf/segurança)

- Paginação por cursor implementada (sem carregar tudo).
- Request expirada mostra badge `expired` e bloqueia `Approve`.
- Modais com validação Zod (`RejectSchema`, `ProposeSchema`).
- CTAs com loading/disabled durante mutation (anti double-submit).
- Estados loading/empty/error/success presentes.
- E2E alvo: `approve_request`, `propose_new_time`.
