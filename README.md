# sc-totem-web (Frontend) - Ecossistema Hubinity - Planned

> Parte integrante do ecossistema distribuído Hubinity.
> ⚠️ **Status atual: Planned** — código de implementação ainda não foi escrito. Este README descreve o papel arquitetural pretendido conforme PRD seção 4 e roadmap em `docs/phases/`.

---

## 💻 Visão Geral

- **O que faz:** Aplicação de auto-atendimento touch-screen para o totem da cafeteria Star Coffee. Implementa o fluxo PWA `IDLE → CATEGORIES → PRODUCTS → REVIEW → IDENTIFY → PAYMENT_METHOD → PAYMENT (PIX QR ou dinheiro) → PRINTING → IDLE`, com retorno ao estado ocioso após cada pedido.
- **Problema que resolve:** elimina a fila do balcão da cafeteria deixando o próprio cliente operar — escolher produtos, identificar-se, pagar via PIX ou em dinheiro, receber a comanda impressa.
- **Posicionamento no Ecossistema:** consumidor único do `sc-order-service` (que por sua vez espelha o catálogo). É a UI do kiosk físico que roda em modo standalone, sem barra de navegação, em hardware dedicado.

## 🏗️ Papel na Arquitetura

- **Tipo de Componente:** Progressive Web App Angular 22 (`display: standalone`, service worker), otimizada para touch grande.
- **Responsabilidades Principais (planejadas):**
  - State machine completa do fluxo do totem (XState).
  - Tela ociosa de boas-vindas com retorno automático ao fim de cada pedido (TTL de inatividade).
  - Telas de catálogo (categorias → produtos) com cache local e exibição offline em "modo limitado" se o backend cair.
  - Carrinho local com revisão/edição.
  - Identificação do cliente (teclado on-screen) e fluxo de pagamento PIX (QR + polling) ou dinheiro (instrução de pagar no balcão).
  - Acionamento de impressão da comanda após confirmação de pagamento.
- **Limites e Fronteiras (Boundaries):** não é dono de catálogo nem de pedido (apenas chama API do `sc-order-service`); não persiste nada localmente além de cache offline.

## 🔗 Dependências e Comunicação (Planejadas)

### Serviços Internos da Hubinity

- **`sc-order-service`** — REST (`/api/v1/catalog/categories`, `/api/v1/catalog/products`, `/api/v1/orders`, `/api/v1/orders/{id}/payments`, `/api/v1/orders/{id}/cancel`, `/api/v1/orders/{id}/print`).
- **`platform-iam` (Keycloak)** — realm `star-coffee`, client de **device** com refresh token de longa duração (~30 dias); redirect URI `http://localhost:4203/*` em local. Cada totem físico é cadastrado individualmente como device.

### Infraestrutura e Serviços Externos

- **Netlify** — hosting (plano Free) com PWA + service worker.
- **Hardware totem** — touch-screen + impressora térmica USB/rede; browser em kiosk-mode com auto-update controlado.

## 🛠️ Tecnologias e Ferramentas (Stack Prevista)

| Camada | Tecnologia | Versão |
| :--- | :--- | :--- |
| Framework | Angular | 22 (PWA) |
| Service Worker | Angular Service Worker (`@angular/service-worker`) | compatível com 22 |
| Estilo | Tailwind CSS | 4 |
| Preset de tokens | `@hubinity/tailwind-preset` (variante Star Coffee — marrom/creme) | última publicada |
| Primitives headless | `@spartan/ui` ou `ng-primitives` | última estável |
| Ícones | `lucide-angular` | última estável |
| Fonte | Manrope (grande, `text-2xl+`) via `@fontsource` | — |
| State machine | XState | última estável |
| Auth | adapter Keycloak (device flow / refresh longa duração) | — |
| E2E | Playwright | última estável |
| Hosting | Netlify | Free |

## 📐 Padrões de Projeto e Arquitetura do Código (Previstos)

- **Estilo Arquitetural:** SPA kiosk-mode + PWA com service worker para cache offline e instalação como app nativo no totem.
- **Padrões Relevantes:**
  - **State Machine explícita (XState)** para o fluxo `IDLE → ... → IDLE` — estados, guardas e timeouts (TTL de `DRAFT` no front em 5min; reset por inatividade).
  - **Polling** com backoff para confirmação PIX (a cada 3s) com cancel manual disponível.
  - **Touch UX** — targets ≥ 44×44px, fontes grandes (`text-2xl+`), alto contraste, animações reduzidas em `prefers-reduced-motion`.
  - **Offline-first** — service worker cacheia catálogo recente; aplicação entra em "modo limitado" se o backend ficar mudo > N minutos.
  - Estilização **exclusivamente via Tailwind CSS 4** consumindo a variante Star Coffee do preset.

## 🗺️ Roadmap & Posição no Board

- **Fase do PRD:** Fase 3 — Totem (PRD seção 9).
- **Tasks no board:**
  - `3.9` — Bootstrap PWA (Angular 22 + Tailwind 4 + tema Star Coffee + kiosk-mode + XState + targets ≥44px).
  - `3.10` — Telas IDLE → CATEGORIES → PRODUCTS.
  - `3.11` — Telas REVIEW → IDENTIFY → PAYMENT_METHOD.
  - `3.12` — Tela PIX (QR Code + polling).
  - `3.13` — Tela DINHEIRO + acionamento de impressão da comanda.
  - `3.15` — E2E fluxo completo do totem (com mock de pagamento).
  - `3.16` — Deploy Netlify (PWA) + setup do totem físico (kiosk-mode + auto-update).
- **Dependências bloqueadoras:** `sc-order-service` com endpoints de pedido (Fase 3.4–3.6) e integração PIX sandbox (Fase 3.5) funcionando.

## ⚙️ Variáveis de Ambiente (Previstas)

```bash
NG_APP_ORDER_API_BASE_URL=https://sc-order-service.up.railway.app
NG_APP_KEYCLOAK_URL=https://iam.hubinity.app
NG_APP_KEYCLOAK_REALM=star-coffee
NG_APP_KEYCLOAK_CLIENT_ID=sc-totem-device
NG_APP_TOTEM_ID=totem-01            # identificador único do hardware
NG_APP_IDLE_TIMEOUT_SEC=120         # retorno automático ao IDLE
NG_APP_PIX_POLL_INTERVAL_MS=3000
```

## 🚀 Como Será Executado (Quando Implementado)

### Pré-requisitos

- Node.js 22 LTS
- Acesso ao GitHub Packages para `@hubinity/tailwind-preset` (variante Star Coffee)
- `sc-order-service` acessível (local ou staging)
- Para o totem físico: browser Chromium em kiosk-mode + impressora térmica conectada

### Execução (Será disponível após bootstrap da Fase 3.9)

```bash
npm ci
npm start         # dev server em http://localhost:4203
npm run build     # build prod (PWA com service worker)
npx playwright test    # E2E com mock de pagamento

# Modo kiosk para hardware (Chromium)
chromium --kiosk --app=https://sc-totem-web.netlify.app
```
