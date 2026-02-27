# calendar-resource — Figma-ready prototype

Fonte: `specs/screens/calendar-resource.yaml`  
Referências: `design/figma/tokens.json`, `design/figma/01-components.md`

## 1) Wireframe textual MOBILE (390x844)

- **AppShell** com Topbar fixa
  - Título: **Calendar**
  - Filtros em stack (gap 8/12):
    - `Select` site
    - `Select` resource
    - `DatePicker` (range)
    - botão/menu de filtros extras
- **Barra de ações**
  - CTA primário: `Button primary` **Create block**
  - CTA secundário: `Button secondary` **Create internal appointment**
- **CalendarGrid** (lazy)
  - Granularidade selecionável: 15/30 min
  - SlotCard variantes:
    - `BOOKED`: carrierName, operation, vehiclePlate, appointmentStatus (`Badge`)
    - `BLOCKED`: reason, ownerName (`Badge`)
  - Tap no slot/card abre **Appointment Drawer**
- **Estados obrigatórios**
  - loading: skeleton-grid
  - empty: "No slots in this range"
  - error: retry-inline
- **Feedback**
  - Toasts: `block_created`, `appointment_created`, `error_default`

## 2) Delta DESKTOP (1440x900)

- Layout AppShell com **Sidebar** (se disponível) + conteúdo em 2 áreas:
  - Coluna esquerda (3-4 cols): filtros + ações
  - Coluna direita (8-9 cols): CalendarGrid
- Drawer pode virar painel lateral persistente no desktop.
- Ações permanecem visíveis (sticky) para reduzir rolagem.

## 3) Spec Figma

- **Pages**
  - `Manager / Calendar`
- **Frames**
  - `Calendar / Mobile / Default` (390x844)
  - `Calendar / Desktop / Default` (1440x900)
  - Frames extras por estado: loading, empty, error, success
- **Grid**
  - Mobile: 4 col, gutter 16, margin 16
  - Desktop: 12 col, gutter 24, margin 32
- **Auto-layout**
  - Tela: vertical, padding 16 (mobile) / 32 (desktop), gap 16
  - Bloco de filtros: vertical (mobile) / horizontal com wrap (desktop)
  - SlotCard: vertical, gap 8, radius md (12)
- **Componentes + variantes**
  - AppShell
  - Select (site/resource)
  - DatePicker (range)
  - Button (primary/secondary/ghost, loading, disabled)
  - CalendarGrid + SlotCard (BOOKED/BLOCKED/FREE)
  - Modal `CreateBlockModal`
  - Modal `CreateAppointmentInternalModal`
  - Drawer `OpenAppointmentDrawer`
  - Badge (status)
  - Toast (success/error)
- **Estados obrigatórios**
  - loading: skeleton-grid
  - empty: mensagem + ação para ajustar filtros
  - error: bloco inline com Retry
  - success: toast confirmando mutation

## 4) Mapa de interações

- `siteSelect` change → refetch `resources` + `slots` (range atual)
- `resourceSelect` change → refetch `slots` (range atual)
- `datePicker` change → refetch `slots` (novo from/to)
- Tap em SlotCard BOOKED/BLOCKED → abre Appointment Drawer
- Tap `Create block` → abre modal → submit → toast `block_created` + refresh slots
- Tap `Create internal appointment` → abre modal → submit → toast `appointment_created` + refresh slots
- Erro em query/mutation → toast `error_default` + ação retry inline

## 5) Copy final

### PT
- Título: **Calendar**
- Empty: **No slots in this range**
- Ações: **Create block**, **Create internal appointment**, **Retry**
- Toasts: **Block created successfully**, **Appointment created successfully**, **Something went wrong**

### ES (público interno opcional)
- Título: **Calendario**
- Empty: **No hay slots en este rango**
- Ações: **Crear bloqueo**, **Crear cita interna**, **Reintentar**
- Toasts: **Bloqueo creado con éxito**, **Cita creada con éxito**, **Algo salió mal**

## 6) Dados mínimos por tela (anti-overfetch)

- `sites`: apenas `id`, `name`
- `resources`: `id`, `name`, `siteId`
- `slots`:
  - comum: `id`, `startAt`, `endAt`, `status`
  - BOOKED: `carrierName`, `operation`, `vehiclePlate`, `appointmentStatus`
  - BLOCKED: `reason`, `ownerName`
- Paginação/janela temporal: buscar somente `from/to` visível no calendário.
- Sem carregar detalhes completos do appointment até abrir drawer.

## 7) Critérios de aceite (UX/perf/segurança)

- Alterar filtros refaz query **somente** do range atual.
- CTA de modal fica disabled/loading durante mutation (anti double-submit).
- CalendarGrid lazy-loaded para reduzir custo inicial.
- Estados loading/empty/error/success implementados e consistentes.
- Erros não expõem payload sensível em UI/log.
- E2E alvo: `calendar_filtering`, `create_block_flow`.
