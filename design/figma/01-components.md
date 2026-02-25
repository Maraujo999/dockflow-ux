# DockFlow UI Library (MVP)

## Princípios
- Mobile-first (390x844). Desktop é adaptação.
- 1 CTA principal por tela.
- Estados obrigatórios: loading (skeleton), empty, error (retry), success (toast).
- Portaria/Operação: 1–2 cliques para ações críticas.

## Componentes base (shadcn/ui mapping)
- AppShell: Topbar + Content + (Desktop: Sidebar)
- Button: primary | secondary | destructive | ghost + loading + disabled
- Input: text, numeric (PIN), textarea
- Select: site, resource, vehicle profile
- DatePicker: data + range (para calendário)
- Badge: status (FREE/BLOCKED/BOOKED/REQUESTED) e appointment status
- Toast: success/error + action opcional
- Modal: confirm / form (Create Block, Reject, Propose time)
- Table/List: row + actions + pagination/cursor
- Stepper: 3–5 passos (Carrier portal)
- CalendarGrid: time slots (15/30m) + SlotCard
- PinInput: 6–8 dígitos + submit rápido
- QRScanner: bottom sheet
- PhotoUploader: multi upload + progress + retry
- SignatureCanvas: canvas + clear + save
