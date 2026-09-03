# Padrão de Componentes de UI — Variante Modernista

Este documento é o complemento de `DESIGN-SYSTEM.md` e define as regras da linguagem visual **modernista**, usada como alternativa ao padrão clássico. Use este guia quando o produto/tela pedir uma estética mais flat, contemporânea e com microinterações — mantendo a mesma arquitetura de código (JS puro, SCSS, props via JSDoc) descrita no documento clássico.

`createModernButton` (`button-modern.js` / `button-modern.scss`) é a implementação de referência.

---

## 1. Quando usar este padrão em vez do clássico

- O produto/marca pede uma comunicação mais "startup/SaaS atual" em vez de tradicional/corporativa.
- A tela já usa bastante espaço em branco e hierarquia por peso tipográfico, não por bordas.
- Não é uma mistura: **um mesmo fluxo/tela não deve misturar botões clássicos e modernistas**. A escolha é por produto/área, não por componente isolado.

Os princípios gerais do item 1 do `DESIGN-SYSTEM.md` (JS puro, SCSS, sem TS/libs, props via JSDoc, `update{Componente}`, exportação via `module.exports`/`window`) continuam valendo integralmente aqui — só o item 2 em diante (estilo visual) muda.

---

## 2. Estilo visual — "modernista"

| Característica | Como aplicar |
|---|---|
| Bordas | Ausentes por padrão (`border: none`). Só a variante `outline` tem borda, fina (1.5px) |
| Cantos | Bem arredondados (10–16px conforme o tamanho) — nunca retos |
| Profundidade | Nenhuma, nem em repouso nem no hover — sem sombra em nenhum estado |
| Hover | Não altera fundo, borda nem sombra — só `filter: brightness()` no elemento inteiro (mesma técnica do clássico) |
| Estado pressionado | `transform: scale(.97)` — microinteração de compressão, sem sombra interna |
| Tipografia | Fonte mais geométrica (Inter ou equivalente), peso 500, leve `letter-spacing` |
| Espaçamento | Padding mais generoso que a versão clássica — "respiro" faz parte da estética |
| Transições | `filter`, `background-color`, `color`, `border-color` e `transform` — sempre com `filter` incluído (senão o `brightness()` do hover troca sem transição nenhuma). Curva `cubic-bezier(.4, 0, .2, 1)` (ease-out suave), não o `ease` genérico. Hover em ~0.22s, clique (`transform`) mais rápido, ~0.15s |

Evitar: qualquer sombra em repouso, cantos retos, bordas grossas, aparência "de botão físico", transições no `ease` padrão do navegador (a "freada" no fim é perceptível e foge do tom suave da estética). A sensação deve ser de superfície plana que reage ao toque, não de objeto com relevo.

---

## 3. Cor

Mesma lógica do padrão clássico — cor via **custom property**, não via classe fixa:

```js
btn.style.setProperty('--btnm-color', '#635bff');
```

```scss
.btn-modern {
  --btnm-color: #635bff; // fallback
}
.btn-modern--solid {
  background-color: var(--btnm-color);
}
```

O hover usa a mesma técnica do clássico — `filter: brightness()` no elemento inteiro, sem tocar em `background-color` nem `box-shadow`:

```scss
$btnm-hover-brightness: .92;
$btnm-ease: cubic-bezier(.4, 0, .2, 1);

// no bloco base do componente:
transition: background-color .22s $btnm-ease,
            color .22s $btnm-ease,
            border-color .22s $btnm-ease,
            filter .22s $btnm-ease,
            transform .15s $btnm-ease;

// no hover de cada variante:
&:hover {
  filter: brightness(#{$btnm-hover-brightness});
}
```

**Atenção:** `filter` precisa obrigatoriamente estar listado no `transition` do bloco base. É um erro comum aplicar o `brightness()` no hover e esquecer de incluir `filter` na lista de propriedades — o resultado é uma troca instantânea, sem suavização nenhuma, mesmo com todo o resto do CSS certo.

Isso mantém a regra de alta compatibilidade do documento principal (sem `color-mix()` ou qualquer recurso CSS recente) e garante que qualquer variante — mesmo `outline`/`ghost`/`link`, que não têm fundo — reage ao hover da mesma forma.

---

## 4. Variantes

As mesmas 4 variantes do clássico, mesmo nome, comportamento equivalente na função, mas execução visual diferente:

| Variante | Clássico | Modernista |
|---|---|---|
| `solid` | Fundo cheio + relevo | Fundo cheio, chapado, sem sombra em nenhum estado |
| `outline` | Borda sólida sempre visível | Borda fina (1.5px), sem fundo em nenhum estado |
| `ghost` | Fundo aparece no hover (cinza neutro) | Sem fundo em nenhum estado |
| `link` | Sublinhado | Sem sublinhado |

Em todas as variantes, o hover é só `filter: brightness()` — nenhuma delas ganha fundo, borda ou sombra ao passar o mouse.

Classe: `.btn-modern--{variante}`.

---

## 5. Tamanhos

Mesma escala `sm`/`md`/`lg`/`xl`, mas com `border-radius` proporcionalmente maior e paddings mais folgados que o clássico:

```
sm  →  radius 10px
md  →  radius 12px  (padrão)
lg  →  radius 14px
xl  →  radius 16px
```

Classe: `.btn-modern--{tamanho}`.

---

## 6. Ícones

Mesma convenção do clássico: `iconStart` / `iconEnd` recebendo string de HTML/SVG, sem biblioteca de ícones. Classes internas usam o prefixo `btn-modern__icon` em vez de `btn__icon`.

---

## 7. Nomenclatura de classes

Mesmo padrão BEM do documento clássico, trocando apenas o nome do bloco:

```
.btn-modern
.btn-modern--{variante|tamanho|estado}
.btn-modern__{elemento}
```

Componentes futuros nesta linguagem visual devem seguir `.{componente}-modern` como bloco, para deixar claro visualmente no CSS/HTML qual variante de estilo está em uso.

---

## 8. API JavaScript

Idêntica em forma à do clássico (mesmo contrato de props, mesma lista: `text`, `color`, `fg`, `variant`, `size`, `iconStart`, `iconEnd`, `block`, `disabled`, `type`, `id`, `className`, `attrs`, `onClick`), trocando apenas o nome da função:

```js
createModernButton(props)
updateModernButton(el, novasProps)
```

Isso é proposital: **a troca entre estilo clássico e modernista deve ser só na função chamada** (`createButton` → `createModernButton`), sem exigir que quem consome o componente reaprenda a API.

---

## 9. Organização de arquivos

Segue a mesma estrutura do clássico, com sufixo `-modern`:

```
button-modern.js       ← lógica, função-fábrica, JSDoc de props
button-modern.scss     ← estilos-fonte
button-modern.css      ← gerado a partir do .scss
demo-comparativo.html  ← comparação lado a lado com a versão clássica
```

---

## 10. Checklist para criar um novo componente modernista

- [ ] Segue a estética modernista (sem borda/relevo em repouso, cantos generosos, sombra só no hover)?
- [ ] Cor aplicada via custom property (`--{componente}m-color`)?
- [ ] Hover usa só `filter: brightness()`, sem alterar fundo/borda/sombra?
- [ ] `filter` está incluído na lista de `transition` do bloco base (senão o hover fica instantâneo)?
- [ ] Transições usam `cubic-bezier(.4, 0, .2, 1)`, não o `ease` padrão do navegador?
- [ ] Feedback de clique via `transform: scale()`, não sombra interna?
- [ ] Suporta as mesmas 4 variantes e 4 tamanhos da versão clássica, com a mesma API de props?
- [ ] Nome da função e das classes usa o sufixo/prefixo `-modern`?
- [ ] `demo` mostrando a variante modernista, idealmente comparada com a clássica?