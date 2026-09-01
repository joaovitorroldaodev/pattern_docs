# Como criar o Botão seguindo o padrão

Guia prático, passo a passo, para reconstruir (ou entender) o componente `Button` do zero seguindo as regras definidas em `DESIGN-SYSTEM.md` (clássico) e `DESIGN-SYSTEM-MODERNISTA.md` (modernista). Use como referência ao criar qualquer componente novo — não só botão.

---

## Passo 0 — Decida a linguagem visual

Antes de escrever qualquer código, defina se o componente segue o padrão **clássico** ou **modernista** (ver item 1 de `DESIGN-SYSTEM-MODERNISTA.md`). Isso muda:

- O prefixo das classes CSS (`.btn` vs `.btn-modern`)
- O nome da função JS (`createButton` vs `createModernButton`)
- Os tokens visuais (bordas, sombra, cantos, curva de transição)

Este guia mostra os dois lados sempre que houver diferença.

---

## Passo 1 — Defina a lista de props (contrato antes do código)

Antes de escrever a função, escreva o `@typedef` com todas as props, tipo e valor padrão. Isso obriga a pensar na API antes da implementação:

```js
/**
 * @typedef {Object} ButtonProps
 * @property {string} [text='']
 * @property {string} [color]
 * @property {string} [fg]
 * @property {'solid'|'outline'|'ghost'|'link'} [variant='solid']
 * @property {'sm'|'md'|'lg'|'xl'} [size='md']
 * @property {string} [iconStart]
 * @property {string} [iconEnd]
 * @property {boolean} [block=false]
 * @property {boolean} [disabled=false]
 * @property {'button'|'submit'|'reset'} [type='button']
 * @property {string} [id]
 * @property {string} [className]
 * @property {Object.<string,string>} [attrs]
 * @property {(event: MouseEvent) => void} [onClick]
 */
```

Toda prop nova em componentes futuros deve seguir esse mesmo nível de detalhe: tipo, se é opcional (`[ ]`) e valor padrão quando fizer sentido.

---

## Passo 2 — Escreva a função-fábrica (`create{Componente}`)

Estrutura fixa que todo componente segue:

```js
function createButton(props = {}) {
  // 2.1 — destructuring com valor padrão (é a "lista de props" na prática)
  const {
    text = '',
    color,
    fg,
    variant = 'solid',
    size = 'md',
    iconStart,
    iconEnd,
    block = false,
    disabled = false,
    type = 'button',
    id,
    className = '',
    attrs = {},
    onClick,
  } = props;

  // 2.2 — validação de enums, com fallback seguro
  const VALID_VARIANTS = ['solid', 'outline', 'ghost', 'link'];
  const VALID_SIZES = ['sm', 'md', 'lg', 'xl'];
  const safeVariant = VALID_VARIANTS.includes(variant) ? variant : 'solid';
  const safeSize = VALID_SIZES.includes(size) ? size : 'md';

  // 2.3 — cria o elemento real (nunca uma string HTML)
  const btn = document.createElement('button');
  btn.type = type;

  // 2.4 — monta as classes
  const classes = ['btn', `btn--${safeVariant}`, `btn--${safeSize}`];
  if (block) classes.push('btn--block');
  if (disabled) classes.push('btn--disabled');
  if (className) classes.push(className);
  btn.className = classes.join(' ');

  // 2.5 — atributos simples
  if (id) btn.id = id;
  if (disabled) btn.disabled = true;

  // 2.6 — cor via custom property, nunca via classe fixa
  if (color) btn.style.setProperty('--btn-color', color);
  if (fg) btn.style.setProperty('--btn-fg', fg);

  // 2.7 — ícones (string HTML/SVG) em spans dedicados
  if (iconStart) {
    const span = document.createElement('span');
    span.className = 'btn__icon btn__icon--start';
    span.innerHTML = iconStart;
    btn.appendChild(span);
  }

  // 2.8 — texto em span próprio (facilita updateButton depois)
  if (text) {
    const label = document.createElement('span');
    label.className = 'btn__label';
    label.textContent = text;
    btn.appendChild(label);
  }

  if (iconEnd) {
    const span = document.createElement('span');
    span.className = 'btn__icon btn__icon--end';
    span.innerHTML = iconEnd;
    btn.appendChild(span);
  }

  // 2.9 — atributos extras arbitrários (data-*, aria-*, etc.)
  Object.entries(attrs).forEach(([key, value]) => btn.setAttribute(key, value));

  // 2.10 — evento
  if (typeof onClick === 'function') btn.addEventListener('click', onClick);

  return btn;
}
```

Para a versão modernista, é o mesmo código trocando `btn` → `btn-modern`, `--btn-color` → `--btnm-color`, e o nome da função para `createModernButton`.

---

## Passo 3 — Escreva a função `update{Componente}`

Todo componente precisa de uma forma de mudar props sem recriar o elemento:

```js
function updateButton(btn, newProps = {}) {
  if (!btn) return;

  if (newProps.text !== undefined) {
    let label = btn.querySelector('.btn__label');
    if (!label) {
      label = document.createElement('span');
      label.className = 'btn__label';
      btn.appendChild(label);
    }
    label.textContent = newProps.text;
  }

  if (newProps.color !== undefined) btn.style.setProperty('--btn-color', newProps.color);
  if (newProps.fg !== undefined) btn.style.setProperty('--btn-fg', newProps.fg);

  if (newProps.variant !== undefined) {
    btn.classList.remove('btn--solid', 'btn--outline', 'btn--ghost', 'btn--link');
    btn.classList.add(`btn--${newProps.variant}`);
  }

  if (newProps.size !== undefined) {
    btn.classList.remove('btn--sm', 'btn--md', 'btn--lg', 'btn--xl');
    btn.classList.add(`btn--${newProps.size}`);
  }

  if (newProps.disabled !== undefined) {
    btn.disabled = !!newProps.disabled;
    btn.classList.toggle('btn--disabled', !!newProps.disabled);
  }
}
```

Regra: só atualize o que veio em `newProps` (checando `!== undefined`), nunca reaplique todas as props — senão qualquer chamada parcial reseta o resto do botão.

---

## Passo 4 — Exporte para os dois formatos de uso

```js
if (typeof module !== 'undefined' && module.exports) {
  module.exports = { createButton, updateButton };
}
if (typeof window !== 'undefined') {
  window.createButton = createButton;
  window.updateButton = updateButton;
}
```

Isso garante que o componente funcione tanto com `<script src="button.js">` direto quanto com `require`/`import` em um bundler, sem precisar reescrever nada.

---

## Passo 5 — Escreva o SCSS: tokens primeiro

Antes das classes, defina as variáveis do componente no topo do arquivo:

```scss
@use "sass:map";

$btn-font-family: -apple-system, "Segoe UI", Tahoma, Arial, sans-serif;
$btn-font-weight: 600;
$btn-radius: 4px;
$btn-transition: background-color .15s ease-in-out,
                  border-color .15s ease-in-out,
                  box-shadow .15s ease-in-out,
                  transform .05s ease-in-out;
$btn-default-color: #2c5aa0;

$btn-sizes: (
  sm: (py: .32em, px: .8em,  fs: .8125rem, gap: .4em, radius: 3px),
  md: (py: .5em,  px: 1.1em, fs: .9375rem, gap: .5em, radius: 4px),
  lg: (py: .65em, px: 1.4em, fs: 1.0625rem, gap: .6em, radius: 5px),
  xl: (py: .8em,  px: 1.75em, fs: 1.1875rem, gap: .7em, radius: 6px),
);
```

> **Atenção com `transition`:** liste **todas** as propriedades que algum estado (hover/active/disabled) vai alterar — inclusive `filter`, se usar `filter: brightness()` em algum lugar. Esquecer uma propriedade aqui é o erro mais comum: o CSS visual fica certo, mas a troca de estado acontece sem suavização nenhuma.

Para o modernista, troque a curva `ease-in-out` por `cubic-bezier(.4, 0, .2, 1)` e use durações um pouco maiores no hover (~0.22s) — ver item 2 de `DESIGN-SYSTEM-MODERNISTA.md`.

---

## Passo 6 — Bloco base (`.btn`)

```scss
.btn {
  --btn-color: #{$btn-default-color}; // fallback
  --btn-fg: #fff;

  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: .5em;

  font-family: $btn-font-family;
  font-weight: $btn-font-weight;
  cursor: pointer;
  border-radius: $btn-radius;
  border: 1px solid transparent;
  transition: $btn-transition;

  &:focus-visible {
    outline: 2px solid rgba(0, 0, 0, .35);
    outline-offset: 2px;
  }

  &.btn--disabled,
  &:disabled {
    cursor: not-allowed;
    opacity: .55;
  }

  &.btn--block {
    display: flex;
    width: 100%;
  }

  &__icon {
    display: inline-flex;
    line-height: 0;
    svg, img { width: 1em; height: 1em; }
  }
}
```

Sempre declare as custom properties de cor (`--btn-color`, `--btn-fg`) dentro do próprio bloco base, com fallback — isso evita que o componente quebre visualmente se alguém usar sem passar a prop `color`.

---

## Passo 7 — Tamanhos (via loop do mapa Sass)

```scss
@each $name, $s in $btn-sizes {
  .btn--#{$name} {
    padding: map.get($s, py) map.get($s, px);
    font-size: map.get($s, fs);
    gap: map.get($s, gap);
    border-radius: map.get($s, radius);
  }
}
```

Gerar os tamanhos a partir do mapa (em vez de escrever 4 blocos manuais) garante que todo tamanho novo (`xxl`, por exemplo) só precise de uma linha no `$btn-sizes`.

---

## Passo 8 — Variantes

Escreva as 4 na ordem `solid → outline → ghost → link`, sempre reaproveitando `var(--btn-color)`:

```scss
.btn--solid {
  background-color: var(--btn-color);
  border-color: var(--btn-color);
  color: var(--btn-fg);

  &:hover:not(.btn--disabled):not(:disabled) {
    filter: brightness(0.92);
  }
}

.btn--outline {
  background-color: transparent;
  border-color: var(--btn-color);
  color: var(--btn-color);

  &:hover:not(.btn--disabled):not(:disabled) {
    filter: brightness(0.92);
  }
}

// ghost e link seguem o mesmo padrão de hover
```

Regra fixa: o seletor de hover **sempre** exclui os estados desabilitados (`:not(.btn--disabled):not(:disabled)`), senão um botão desabilitado ainda reage visualmente ao mouse.

---

## Passo 9 — Compile o SCSS

```bash
npx sass button.scss button.css --no-source-map
```

Rode sempre que editar o `.scss`. O `.css` gerado nunca deve ser editado manualmente — qualquer ajuste visual entra pelo `.scss`.

---

## Passo 10 — Escreva o `demo.html`

Cubra, no mínimo:

- As 4 variantes com a cor padrão
- 2–3 cores customizadas (pra provar que a custom property funciona)
- Os 4 tamanhos
- Ícone no início, no fim, e nos dois ao mesmo tempo
- Estado `disabled`
- `block: true`

```html
<link rel="stylesheet" href="button.css">
<script src="button.js"></script>
<script>
  document.body.appendChild(createButton({ text: 'Salvar', color: '#2e7d32' }));
</script>
```

Se for compartilhar o demo fora do projeto (Slack, e-mail, preview isolado), gere também uma versão standalone com CSS e JS embutidos via `<style>`/`<script>` inline, para não depender de arquivos irmãos.

---

## Passo 11 — Rode o checklist final

Antes de considerar o componente pronto, confira contra o checklist do item 10 de `DESIGN-SYSTEM.md` (clássico) ou item 10 de `DESIGN-SYSTEM-MODERNISTA.md` (modernista) — ambos cobrem os pontos que mais geram inconsistência entre componentes (transição incompleta, cor fixa em vez de custom property, variante faltando, etc.).