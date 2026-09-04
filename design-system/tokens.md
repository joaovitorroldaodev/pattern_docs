# Tokens

Tokens são as variáveis de baixo nível que sustentam todos os componentes do design system. Eles são organizados em duas camadas:

- **Primitivos** — valores brutos (escalas de cor neutra, espaçamento, tamanhos). Raramente usados diretamente nos componentes.
- **Semânticos** — aliases com significado (`bg`, `border`, `text`, `bg-danger`...). É isso que os componentes consomem. Cada projeto pode sobrescrever os valores semânticos sem tocar nos componentes.

> Regra geral: **componentes nunca referenciam cor primitiva diretamente.** Sempre um token semântico. Isso é o que torna a paleta maleável entre projetos — você redefine os tokens semânticos por projeto/tema, os componentes não mudam.

---

## Cor

### Primitivos (escala neutra de referência)

| Token         | Valor     |
| ------------- | --------- |
| `neutral-0`   | `#ffffff` |
| `neutral-50`  | `#fafafa` |
| `neutral-100` | `#f2f2f2` |
| `neutral-200` | `#e4e4e4` |
| `neutral-300` | `#cccccc` |
| `neutral-500` | `#8a8a8a` |
| `neutral-700` | `#4a4a4a` |
| `neutral-900` | `#1a1a1a` |
| `neutral-950` | `#0d0d0d` |

Estes existem só como base default. Um projeto pode ignorar essa escala inteira e alimentar os tokens semânticos abaixo com sua própria paleta (inclusive não-neutra).

### Semânticos — superfície e texto

| Token            | Default (mapeia para)                                      | Uso                                                                          |
| ---------------- | ---------------------------------------------------------- | ---------------------------------------------------------------------------- |
| `surface-1`      | `neutral-0`                                                | fundo base da aplicação                                                      |
| `surface-2`      | `neutral-950`                                              | fundo invertido (ex: texto de botão primary)                                 |
| `overlay-scrim`  | `rgba(13, 13, 13, 0.5)` (`neutral-950` a 50% de opacidade) | fundo escurecido atrás de elementos de overlay (modal, e futuramente drawer) |
| `text-primary`   | `neutral-950`                                              | texto/ação de maior ênfase                                                   |
| `text-secondary` | `neutral-700`                                              | texto de apoio, botão secondary                                              |
| `text-muted`     | `neutral-500`                                              | texto desabilitado, placeholder                                              |
| `border-strong`  | `neutral-300`                                              | bordas visíveis (botão secondary, input)                                     |
| `border-subtle`  | `neutral-200`                                              | divisores, bordas discretas                                                  |

### Semânticos — ação e feedback

| Token           | Default       | Uso                              |
| --------------- | ------------- | -------------------------------- |
| `bg-accent`     | `#2563eb`     | ações de destaque                |
| `text-accent`   | `#ffffff`     | texto sobre `bg-accent`          |
| `bg-danger`     | `#dc2626`     | ações destrutivas                |
| `text-danger`   | `#ffffff`     | texto sobre `bg-danger`          |
| `bg-success`    | `#16a34a`     | confirmação/sucesso              |
| `text-success`  | `#ffffff`     | texto sobre `bg-success`         |
| `fill-disabled` | `neutral-100` | fundo de elementos desabilitados |

**Como um componente deve declarar cor** (exemplo, botão primary):

```css
background: var(--bg, var(--text-primary));
color: var(--fg, var(--surface-2));
```

O componente expõe `--bg` e `--fg` como pontos de override locais, com fallback pros tokens semânticos globais. Assim dá pra recolorir uma instância única sem criar uma variante nova.

---

## Espaçamento

Escala em base 4px, suficiente para cobrir padding, gap e margin de todos os componentes.

| Token     | Valor  |
| --------- | ------ |
| `space-1` | `4px`  |
| `space-2` | `8px`  |
| `space-3` | `12px` |
| `space-4` | `16px` |
| `space-5` | `20px` |
| `space-6` | `24px` |
| `space-8` | `32px` |

---

## Tamanho de componente (densidade confortável)

Alturas padrão para elementos interativos (botão, input, select). Densidade **confortável** definida como padrão do sistema.

| Token     | Altura | Padding horizontal | Font-size |
| --------- | ------ | ------------------ | --------- |
| `size-sm` | `32px` | `space-3` (12px)   | `13px`    |
| `size-md` | `40px` | `space-4` (16px)   | `14px`    |
| `size-lg` | `48px` | `space-6` (24px)   | `15px`    |

`size-md` é a altura base — usar como padrão quando nenhum tamanho for especificado.

---

## Radius

| Token         | Valor   | Uso                                     |
| ------------- | ------- | --------------------------------------- |
| `radius-sm`   | `6px`   | elementos pequenos (badge, chip)        |
| `radius-md`   | `8px`   | padrão (botão, input, card)             |
| `radius-lg`   | `12px`  | containers maiores (modal, card grande) |
| `radius-full` | `999px` | elementos circulares (avatar, dot)      |

`radius-md` é o valor base do sistema.

---

## Borda

| Token          | Valor   |
| -------------- | ------- |
| `border-width` | `0.5px` |

Bordas finas por padrão — reforça o visual refinado/discreto em vez de contornos pesados.

---

## Tipografia

| Token                 | Valor                                            |
| --------------------- | ------------------------------------------------ |
| `font-family`         | `-apple-system, "Inter", "Segoe UI", sans-serif` |
| `font-weight-regular` | `400`                                            |
| `font-weight-medium`  | `500`                                            |

Apenas dois pesos no sistema inteiro. Isso é intencional — evita inconsistência visual e mantém a hierarquia baseada em tamanho/cor, não em peso de fonte.

| Token       | Valor  | Uso                         |
| ----------- | ------ | --------------------------- |
| `text-xs`   | `12px` | badges, labels auxiliares   |
| `text-sm`   | `13px` | botão small, texto de apoio |
| `text-base` | `14px` | corpo padrão, botão medium  |
| `text-md`   | `15px` | botão large                 |
| `text-lg`   | `18px` | títulos de seção            |
| `text-xl`   | `24px` | títulos de página           |

---

## Sombra

Sistema intencionalmente **sem sombra decorativa** por padrão — profundidade é comunicada por borda e contraste, não por drop-shadow. Reservada apenas para elementos flutuantes reais:

| Token            | Valor                         | Uso                      |
| ---------------- | ----------------------------- | ------------------------ |
| `shadow-overlay` | `0 4px 16px rgba(0,0,0,0.12)` | dropdown, popover, modal |

---

## Movimento

Transições existem só como resposta direta a uma ação do usuário (abrir, fechar, expandir) — nunca como animação ambiente ou decoração contínua. Reservado a componentes de overlay (modal, e futuramente popover, dropdown, toast); componentes estáticos (Card, Button, Input) não têm tokens de movimento próprios além de `focus-visible`/`hover` instantâneos.

| Token           | Valor                        | Uso                                            |
| --------------- | ---------------------------- | ---------------------------------------------- |
| `duration-fast` | `150ms`                      | transições de elementos finos (overlay/scrim)  |
| `duration-base` | `200ms`                      | transição padrão de container (modal, popover) |
| `easing-out`    | `cubic-bezier(0, 0, 0.2, 1)` | curva de **entrada** — desacelera              |
| `easing-in`     | `cubic-bezier(0.4, 0, 1, 1)` | curva de **saída** — acelera                   |

Convenção fixa: entrada sempre usa `easing-out` (chega "suave"), saída sempre usa `easing-in` (sai "rápido") — é o padrão consagrado para diálogos e não deve ser invertido componente a componente.

Todo componente que usa estes tokens deve respeitar `prefers-reduced-motion: reduce` reduzindo a duração a praticamente zero — não é opcional, decorre do princípio de acessibilidade (ver `principles.md`).

---

## Estabilidade entre versões

Regras para manter esse arquivo estável conforme o sistema evolui:

1. **Nunca renomear um token existente** — só adicionar novos ou depreciar (marcar como `@deprecated` no comentário, manter funcional por pelo menos uma versão major).
2. **Nunca remover um token sem deprecar primeiro.**
3. Mudança de _valor_ default (ex: ajustar `radius-md` de `8px` para `10px`) é aceitável, mas deve ser documentada em changelog — o nome do token é o contrato, não o valor.

---

## Changelog

| Data       | Mudança                                                                                                                                                               |
| ---------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 2026-09-04 | Adicionado `overlay-scrim` (seção Semânticos — superfície e texto), a pedido do componente Modal.                                                                     |
| 2026-09-04 | Adicionada categoria **Movimento** (`duration-fast`, `duration-base`, `easing-out`, `easing-in`), a pedido do componente Modal — compartilhável por futuros overlays. |
