# dock-operator-pod — Figma-ready prototype

Fonte: `specs/screens/dock-operator-pod.yaml`  
Referências: `design/figma/tokens.json`, `design/figma/01-components.md`

## 1) Wireframe textual MOBILE (390x844)

- **AppShell**
  - Topbar: "Operations"
- **Lista de appointments do dia**
  - Cards com: `status`, `carrier`, `docsCount`
  - Ação por card: `openDetail`
- **Detail panel (na mesma tela via navegação/painel)**
  - CTA: `startDockingBtn` (quando status ARRIVED)
  - CTA: `completeBtn` (quando status DOCKED)
- **Seção POD (form)**
  - Checklist (checkboxList)
  - Photos (`PhotoUploader` lazy, multi upload + progress + retry)
  - Signature (`SignatureCanvas` lazy, clear + save)
  - Notes (`textarea`)
  - Incident toggle
    - Quando ativo: campos `type`, `notesOptional`, `photos`
- **Estados**
  - loading: skeleton
  - empty: "No appointments"
  - error: retry
- **Feedback**
  - Toasts: `docked`, `completed`, `pod_saved`, `pod_finalized`, `error_default`

## 2) Delta DESKTOP (1440x900)

- Split-view de alta produtividade:
  - Esquerda (4 cols): lista de appointments
  - Direita (8 cols): detalhe + formulário POD
- CTAs principais fixos no topo do painel de detalhe.
- Uploader e assinatura com área maior para operação de mouse.

## 3) Spec Figma

- **Pages**
  - `Operator / Operations + POD`
- **Frames**
  - `Operations / Mobile / List`
  - `Operations / Mobile / Detail+POD`
  - `Operations / Desktop / Split`
  - variações de loading/empty/error/success
- **Grid**
  - Mobile: 4 col, gutter 16, margin 16
  - Desktop: 12 col, gutter 24, margin 32
- **Auto-layout**
  - Lista: stack vertical gap 12
  - Detail/POD: seções em cards (gap 16)
  - CTAs: full-width mobile, inline desktop
- **Componentes + variantes**
  - AppShell
  - List/Card row
  - Button (primary/secondary/destructive/ghost + loading/disabled)
  - Input textarea
  - PhotoUploader (idle/uploading/error/success)
  - SignatureCanvas (empty/drawn/saved)
  - Modal de confirmação para `complete`/`finalize`
  - Toast success/error
- **Estados obrigatórios**
  - loading: skeleton em lista e detalhe
  - empty: mensagem sem appointments
  - error: bloco retry
  - success: toast e atualização incremental da tela

## 4) Mapa de interações

- Tap em card da lista → carrega `appointmentDetail`
- Tap `Start docking` → mutation `startDocking` → toast `docked` + refresh item
- Edição de POD → `podSave` (draft incremental)
- Tap `Finalize POD` (quando disponível) → `podFinalize` → toast `pod_finalized`
- Tap `Complete` → `complete` → toast `completed`
- Upload de foto → URL pré-assinada + progresso + retry por item
- Erro em qualquer ação → toast `error_default` + manter dados locais

## 5) Copy final

### PT
- Empty: **No appointments**
- CTAs: **Start docking**, **Complete**, **Save draft**, **Finalize POD**
- Incidente: **Report incident**
- Toasts: **Docking started**, **Operation completed**, **POD saved**, **POD finalized**, **Something went wrong**

### ES (interno opcional)
- Empty: **No hay citas**
- CTAs: **Iniciar atraque**, **Completar**, **Guardar borrador**, **Finalizar POD**
- Incidente: **Reportar incidente**
- Toasts: **Atraque iniciado**, **Operación completada**, **POD guardado**, **POD finalizado**, **Algo salió mal**

## 6) Dados mínimos por tela (anti-overfetch)

- Lista `todaysAppointments`: `id`, `status`, `carrier`, `docsCount`, `updatedAt`, `cursor`
- Detalhe `appointmentDetail`:
  - identificação mínima da operação
  - estado de docking/completion
  - estrutura POD atual (checklist flags, photos refs, signature ref, notes, incident)
- Upload: usar fluxo direto com URL pré-assinada (sem proxy de arquivo completo no app server).
- Salvar incremental (draft) por blocos alterados para evitar payload grande.

## 7) Critérios de aceite (UX/perf/segurança)

- Upload de imagens via URL pré-assinada com retry por arquivo.
- POD draft não perde dados ao navegar/atualizar detalhe.
- CTAs bloqueados durante mutation ativa (anti double-submit).
- Lazy load de `SignatureCanvas` e `PhotoUploader` funcionando.
- Estados loading/empty/error/success implementados.
- Logs não devem incluir conteúdo sensível de evidências.
- E2E alvo: `start_docking`, `fill_pod_and_finalize`.
