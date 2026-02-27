# pod-readonly — Figma-ready prototype

Fonte: `specs/screens/pod-readonly.yaml`  
Referências: `design/figma/tokens.json`, `design/figma/01-components.md`

## 1) Wireframe textual MOBILE (390x844)

- **Public shell** (sem ações de edição)
- **Header**
  - appointmentSummary
  - timestamps relevantes
- **Sections (somente leitura)**
  - checklist
  - photos (thumbnails primeiro, galeria lazy)
  - signature
  - documents
  - incidents
- **Ações**
  - `downloadPdfBtn`
- **Estados**
  - loading: skeleton
  - error: retry
- **Segurança visual**
  - Tag "Read-only"
  - Sem campos editáveis/CTAs de mutação

## 2) Delta DESKTOP (1440x900)

- Conteúdo centralizado com largura confortável para leitura.
- Layout em 2 colunas:
  - Coluna principal: checklist, incidents, documents
  - Coluna lateral: photos + signature + download PDF
- Header fixo com resumo e timestamps para contexto constante.

## 3) Spec Figma

- **Pages**
  - `Public / POD Readonly`
- **Frames**
  - `POD Public / Mobile`
  - `POD Public / Desktop`
  - variações loading/error
- **Grid**
  - Mobile: 4 col, gutter 16, margin 16
  - Desktop: 12 col, gutter 24, margin 32
- **Auto-layout**
  - Stack vertical por seção, gap 16
  - Cards com radius md (12)
- **Componentes + variantes**
  - PublicShell/AppShell minimal
  - Badge (`readOnly`)
  - List de checklist
  - PhotoGallery (thumbnail/loading/error)
  - Signature card
  - Document links
  - Button secondary `Download PDF`
  - Toast/error inline para falhas de carregamento/download
- **Estados obrigatórios**
  - loading: skeleton por seção
  - error: bloco retry global
  - success (download): feedback discreto/toast
  - empty parcial por seção (ex.: sem incidentes)

## 4) Mapa de interações

- Acesso `GET /public/pods/{publicId}?token=` → render read-only
- Scroll pelas seções não dispara overfetch desnecessário
- Tap thumbnail → abre visualização ampliada (lightbox)
- Tap `Download PDF` → inicia download
- Erro de token/rede → estado error + retry

## 5) Copy final

### PT
- Badge: **Read-only**
- Botão: **Download PDF**
- Error: **Não foi possível carregar o POD**
- Retry: **Tentar novamente**
- Seção vazia: **Sem incidentes**

### ES (público)
- Badge: **Solo lectura**
- Botão: **Descargar PDF**
- Error: **No se pudo cargar el POD**
- Retry: **Reintentar**
- Seção vazia: **Sin incidentes**

## 6) Dados mínimos por tela (anti-overfetch)

- `podPublic`:
  - summary: appointment id/ref, carrier, vehicle, route/site
  - timestamps essenciais
  - checklist (itens + status)
  - photos (thumbUrl, fullUrl)
  - signature (imageUrl)
  - documents (name, url, mime)
  - incidents (type, notes, photos refs)
- Estratégia de mídia:
  - thumbnails primeiro
  - full-res sob demanda ao abrir imagem
- Não persistir token; não logar token em cliente/servidor.

## 7) Critérios de aceite (UX/perf/segurança)

- Tela é estritamente read-only (nenhum endpoint de mutation exposto).
- Fotos carregam lazy com prioridade para thumbnails.
- Token obrigatório e não persistido em storage/logs.
- Estados loading/error implementados com retry.
- Download PDF funcional sem expor token em mensagens de erro.
- E2E alvo: `pod_public_view`.
