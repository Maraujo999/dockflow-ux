✅ Teste 1 — tokens (breakpoints, spacing, grid)

Breakpoint mobile: 390×844

Breakpoint desktop: 1440×900

Spacing scale (px): 0, 4, 8, 12, 16, 20, 24, 32, 40

Grid mobile: 4 colunas • gutter 16 • margem 16

Grid desktop: 12 colunas • gutter 24 • margem 32


✅ Teste 2 — components (01-components.md)

Componentes definidos:

AppShell (Topbar + Content + no desktop: Sidebar)

Button (primary/secondary/destructive/ghost + loading + disabled)

Input (text, numeric/PIN, textarea)

Select (site, resource, vehicle profile)

DatePicker (data + range)

Badge (status de slot e appointment)

Toast (success/error + action opcional)

Modal (confirm / form)

Table/List (row + actions + paginação/cursor)

Stepper (3–5 passos)

CalendarGrid (time slots + SlotCard)

PinInput (6–8 dígitos + submit rápido)

QRScanner (bottom sheet)

PhotoUploader (multi upload + progress + retry)

SignatureCanvas (canvas + clear + save)


✅ Teste 3 — protótipo Figma-ready (gate-checkin.yaml) com estados + interações

Assumptions (para fechar gaps do YAML sem travar):

PinInput usa 6 dígitos por padrão (ajustável para 6–8).

eventKey controla a intenção: LOOKUP (validar/buscar) e ARRIVED (confirmar chegada).

checkin retorna o resultPanel.state (ok | alreadyArrived | invalidOrExpired) + dados mínimos do appointment para o resumo.

reprintBtn abre um modal de “Comprovante” (ou aciona impressão se houver suporte no device).

1) Decisão UX (trade-offs)

Meta portaria = 1 gesto + 1 toque: digitar PIN (auto-busca ao completar) ou scan QR (abre bottom sheet) → ver resumo → Confirmar Chegada.

Evitar erro operacional: sempre mostrar Resumo antes de marcar chegada; reduz check-in no caminhão errado.

Capacidade cheia visível: banner persistente (não bloqueia o fluxo), mas muda o feedback pós-chegada para “WAITING_OUTSIDE” quando aplicável.

2) Fluxo do usuário (mobile-first)

Guarda abre /app/gate/checkin

Digita PIN (auto-submit ao completar) ou toca Scan QR

App mostra Result Panel:

ok: mostra resumo + botão Marcar Chegada

alreadyArrived: mostra horário + Reimprimir

invalidOrExpired: mensagem + dica do que fazer

Ações secundárias:

Registrar Saída (markExited) disponível como ação secundária (condicional: só quando o backend indicar que faz sentido)

3) Wireframe textual
MOBILE — Frame 390×844 (grid 4 col / margem 16 / gutter 16)

AppShell (gate-shell)

Topbar (sticky)

Título: “Portaria — Check-in”

Ação ghost (opcional): “Ajuda”

Banner (condicional)

capacityFull = true: Banner warning “Capacidade do site cheia — priorize agendados / aguarde liberação”

Bloco Entrada

PinInput

Label: “PIN ou token”

Helper (caption): “Digite o PIN do motorista (6 dígitos)”

Auto-submit ao completar

Estado: disabled/loading quando rate-limit ativo

Botão secondary (full width): “Scan QR”

Abre QRScanner em bottom sheet (lazy)

Result Panel (card)

Estado loading: inline-spinner + texto “Validando…”

Estado ok:

appointmentSummary (Card)

Linha 1: Carrier + placa (body)

Linha 2: Janela/horário (caption)

Linha 3: Badge do AppointmentStatus

Button primary (full width): “Marcar Chegada”

Button ghost (full width / ou link): “Registrar Saída” (secondaryAction) (pode ficar abaixo, menor)

Estado alreadyArrived:

Texto: “Já chegou”

Campo: timestamp (ex.: “Chegou às 10:42”)

Button secondary: “Reimprimir”

Button ghost: “Registrar Saída” (se permitido)

Estado invalidOrExpired:

Texto forte: “PIN inválido ou expirado”

auditHint (caption): “Confirme com o motorista ou use Scan QR”

Toast region (global):

arrived_ok, exited_ok, error_default

DESKTOP — Delta (Frame 1440×900, grid 12 col / margem 32 / gutter 24)

Manter conteúdo centralizado (max-width ~560–640)

Opcional: 2 colunas

Esquerda: PinInput + Scan QR

Direita: Result Panel (fixo, evita “jump”)

4) Especificação Figma (Figma-ready)

Frames

Gate / Check-in / Mobile (390x844)

Gate / Check-in / Desktop (1440x900)

Grid

Mobile: 4 col / margin 16 / gutter 16

Desktop: 12 col / margin 32 / gutter 24

Tipografia (tokens)

H1: 20/700/28 (Topbar title)

H2: 18/600/26 (títulos de card/estado)

Body: 14/400/20 (conteúdo)

Caption: 12/400/16 (helper/audit)

Raios

Card: radius md (12)

Botões: radius md (12)

Inputs: radius md (12)

Componentes (com variantes/estados)

PinInput

Variants: default | loading | disabled | error

Interaction: onComplete -> submit(LOOKUP) + lock (rate-limit)

Button

primary/secondary/ghost + loading/disabled

QRScanner (Bottom Sheet)

States: idle | scanning | permissionDenied | scanSuccess | scanError

Interaction: scanSuccess -> close sheet -> submit(LOOKUP)

ResultPanel (Card)

Variants: loading | ok | alreadyArrived | invalidOrExpired | networkError

Toast

Variants: success | error + action Retry quando aplicável

Banner

Variant: warning (capacityFull)

Auto-layout (regras)

Página: coluna (gap 16/20/24 conforme seção)

ResultPanel: coluna, gap 12

Botões: full-width no mobile; no desktop podem virar “inline” (primary + ghost)

5) Plano de implementação (Next.js + TS + shadcn/ui)

Rota

app/app/gate/checkin/page.tsx (ou app/(app)/gate/checkin/page.tsx)

Componentes

GateShell (layout)

PinInputField (React Hook Form)

QrScannerSheet (lazy import)

ResultPanel

CapacityBanner

useRateLimitLock(ms) (hook simples)

React Hook Form + Zod

PinSchema com campos:

pinOrQrToken: string (min 6, max 8, numeric quando digitado)

eventKey: 'LOOKUP' | 'ARRIVED'

Validação: onSubmit e onBlur (mensagens curtas)

TanStack Query (mutations)

useMutation(checkin) → POST /api/gate/checkin

useMutation(markExited) → POST /api/gate/exited

Estratégia:

Ao completar PIN ou scan: dispara checkin({ token, eventKey:'LOOKUP' })

Ao tocar “Marcar Chegada”: dispara checkin({ token, eventKey:'ARRIVED' })

Ao tocar “Registrar Saída”: dispara markExited({ token }) (ou com appointmentId se retornar)

Performance

QrScannerSheet via dynamic(() => import(...), { ssr:false })

Evitar overfetch: não buscar listas; só mutation + retorno mínimo

6) Estados e feedback (obrigatórios)

Loading

Inline spinner no ResultPanel

Botões disabled + loading

Empty

Antes de qualquer busca: ResultPanel mostra “Digite o PIN ou use Scan QR”

Error (network)

Toast error_default + action “Tentar novamente”

ResultPanel variant networkError com botão Retry

Success

Toast arrived_ok / exited_ok

Limpar PIN/token do input após sucesso + foco volta ao PinInput

7) Requisitos mínimos de dados (anti-overfetch)

Resposta de POST /api/gate/checkin (mínimo sugerido):

state: 'ok' | 'alreadyArrived' | 'invalidOrExpired'

capacityFull: boolean

appointment?: { id, carrierName, plate, windowStart, windowEnd, appointmentStatus, arrivedAt? }

canMarkExited?: boolean

message? (para invalid/expired)

Por quê: evita baixar appointment completo/documentos na portaria.

8) Testes + critérios de aceite

Aceite (do YAML + UX)

Botão/submit com rate-limit lock após submit

Nunca logar PIN/QR no console

Estados obrigatórios (loading/empty/error/success) presentes

E2E (Playwright)

checkin_ok: PIN válido → resumo → marcar chegada → toast sucesso

checkin_invalid: PIN inválido → mensagem + auditHint

mark_exited: buscar ok → registrar saída → toast sucesso

Unit/UI

Zod: rejeita tamanho inválido

ResultPanel: renderiza corretamente cada variant

9) Riscos UX e mitigação

Fraude/abuso de PIN: lock client-side + backend rate-limit; limpar campo após erro repetido; não revelar detalhes (“inválido/expirado” genérico).

Vazamento de token: nunca em querystring; não persistir em localStorage; limpar ao navegar.

Race visual (double submit): lock + desabilitar ao isPending.

Capacidade cheia: banner + mensagem clara pós-chegada (“Aguardar fora”) quando aplicável.

Se você colar aqui o conteúdo do /mnt/data/gate-checkin.yaml em uma versão mais detalhada (ex.: campos do appointmentSummary), eu adapto o ResultPanel para exatamente os atributos que você já definiu — sem inventar nada além do MVP.