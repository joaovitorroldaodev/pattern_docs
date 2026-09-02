# Padrão de Componente — Input (Variante Modernista)

Este documento orienta como desenvolver o componente de **input** na linguagem visual **modernista**, seguindo a mesma arquitetura de código do `button-modern` (JS puro, SCSS, props via JSDoc) e estendendo os princípios do `style-guide.md` para as particularidades de um campo de texto: label, validação, texto de apoio e ícones internos.

`createModernInput` (`input-modern.js` / `input-modern.scss`) é a implementação de referência.

> **Nota de proveniência:** o `DESIGN-SYSTEM.md` (documento clássico que define a arquitetura base do design system) não estava disponível na criação deste guia. As decisões marcadas como **[inferido]** abaixo foram extrapoladas do padrão observado em `button-modern` e devem ser revisadas contra um `input-classic.js` real, se/quando ele existir.

---

## 1. Quando usar este padrão

- Sempre que o produto/tela já usa a variante modernista para outros componentes (botão, etc.) — **um mesmo fluxo não deve misturar input clássico e modernista**.
- A escolha é por produto/área, nunca por campo isolado dentro da mesma tela.
- Os princípios gerais de arquitetura (JS puro, SCSS, sem TS/libs, props via JSDoc, `update{Componente}`, exportação via `module.exports`/`window`) valem integralmente aqui.

---

## 2. Anatomia do componente

Diferente do botão, o input não é um elemento único — ele é composto por até quatro partes, todas dentro do elemento raiz:

```
.input-modern
├── .input-modern__label       (opcional)
├── .input-modern__field       (wrapper visual: borda/fundo/radius vivem aqui, não no <input>)
│   ├── .input-modern__icon--start   (opcional)
│   ├── .input-modern__control       (o <input> nativo, sem estilo próprio de borda/fundo)
│   └── .input-modern__icon--end     (opcional)
└── .input-modern__hint        (opcional: helper text OU mensagem de erro)
```

**Por que separar `__field` do `__control`:** o `<input>` nativo não pode receber ícones dentro dele. Colocar borda, fundo e radius no wrapper (`__field`) — e deixar o `<input>` transparente e sem borda — é o que permite ícones "dentro" do campo sem hacks de posicionamento.

---

## 3. Estilo visual — regras herdadas do `style-guide.md`

| Característica | Como aplicar |
|---|---|
| Bordas | Ausentes por padrão. Só `outline` (sempre) e `filled`/`error` (só no estado de erro) têm borda, fina (1.5px) |
| Cantos | Bem arredondados (10–16px conforme o tamanho) — nunca retos, exceto `underline` |
| Profundidade | Nenhuma, em nenhum estado — sem sombra, nem em repouso, nem no hover, nem no foco |
| Hover | Não altera fundo, borda nem sombra — só `filter: brightness()` no `__field` inteiro |
| Tipografia | Fonte geométrica (Inter ou equivalente), peso 500, leve `letter-spacing` |
| Espaçamento | Padding generoso — mesma lógica de "respiro" do botão |
| Transições | `background-color`, `color`, `border-color`, `filter` — sempre com `filter` incluído. Curva `cubic-bezier(.4, 0, .2, 1)`, ~0.22s |

Evitar: sombra em qualquer estado (inclusive foco — ver seção 5), cantos retos fora da variante `underline`, `ease` padrão do navegador.

---

## 4. Cor

Mesma lógica do botão — cor via **custom property**, não classe fixa:

```js
inputEl.style.setProperty('--inpm-color', '#635bff');
```

```scss
.input-modern {
  --inpm-color: #635bff; // fallback
}
```

`--inpm-color` **não** vira background (diferente do botão `solid`) — um input com fundo na cor de destaque seria confundido com estado de erro/alerta. Ela é usada como cor de **foco** (borda) e pode ser referenciada por variantes futuras para o label/ícone quando o campo está ativo.

---

## 5. Estado de foco **[inferido]**

O `style-guide.md` original foi escrito pensando em botão, que não tem foco visualmente distinto do hover. Um input **precisa** de indicador de foco acessível (WCAG 2.4.7), então este guia estabelece a regra que falta:

- Foco é indicado por `border-color: var(--inpm-color)` (via `:focus-within` no `__field`), **nunca** por `box-shadow`/anel — mantém a regra "zero profundidade" do modernista.
- Na variante `filled`, que não tem borda em nenhum estado de repouso, o foco usa um `brightness` sutil no lugar da borda, para não introduzir uma borda que a variante propositalmente não tem.
- `filter: brightness()` de foco é opcional e mais discreto que o de hover — o indicador principal é sempre a cor da borda, não o brilho.

---

## 6. Variantes

| Variante | Borda | Fundo | Quando usar |
|---|---|---|---|
| `outline` (padrão) | Fina (1.5px), sempre visível | Nenhum | Uso geral, formulários densos |
| `filled` | Nenhuma (exceto erro) | Neutro chapado | Formulários com poucos campos, hierarquia por bloco de cor |
| `underline` | Só linha inferior | Nenhum | Interfaces minimalistas, uma linha por campo |
| `ghost` | Nenhuma | Nenhum | Campos de busca/inline dentro de outro componente (ex.: dentro de um card) |

Classe: `.input-modern--{variante}`.

Em todas, o hover é só `filter: brightness()` — nenhuma ganha fundo, borda ou sombra ao passar o mouse.

---

## 7. Tamanhos

Mesma escala `sm`/`md`/`lg`/`xl` do botão, radius e padding no wrapper (`__field`), nunca no `<input>`:

```
sm  →  radius 10px
md  →  radius 12px  (padrão)
lg  →  radius 14px
xl  →  radius 16px
```

Classe: `.input-modern--{tamanho}`.

---

## 8. Estados

| Estado | Classe | Comportamento |
|---|---|---|
| Obrigatório | `required: true` na prop | Asterisco (`__required`) ao lado do label |
| Erro | `error: 'mensagem'` | Borda vira vermelha (mesmo em `filled`/`underline`), `__hint` mostra a mensagem em vermelho, `aria-invalid="true"` |
| Helper text | `helperText: 'texto'` | `__hint` neutro abaixo do campo — some automaticamente se `error` estiver presente |
| Desabilitado | `disabled: true` | Fundo neutro fixo, texto apagado, hover desativado (`filter: none`) |

**Regra importante:** `error` e `helperText` nunca aparecem juntos — erro sempre tem prioridade, pois é a informação mais urgente para quem preenche o campo.

---

## 9. Ícones

Mesma convenção do botão: `iconStart` / `iconEnd` recebendo string de HTML/SVG, sem biblioteca de ícones. Ficam dentro do `__field`, ladeando o `<input>`. Classes internas usam o prefixo `input-modern__icon`.

---

## 10. Nomenclatura de classes

Mesmo padrão BEM do botão, trocando apenas o nome do bloco:

```
.input-modern
.input-modern--{variante|tamanho|estado}
.input-modern__{elemento}
```

Componentes futuros de campo (`textarea`, `select`) devem seguir o mesmo esqueleto `label` → `field` (wrapper visual) → `hint`, trocando `.input-modern` pelo bloco correspondente (`.textarea-modern`, `.select-modern`).

---

## 11. API JavaScript

```js
createModernInput(props)
updateModernInput(el, novasProps)
```

Contrato de props (ver JSDoc completo em `input-modern.js`):

| Prop | Tipo | Descrição |
|---|---|---|
| `label` | string | Texto acima do campo |
| `placeholder` | string | Placeholder nativo |
| `value` | string | Valor controlado inicial |
| `type` | string | Tipo nativo (`text`, `email`, `password`, `number`...) |
| `color` | string | Cor de destaque (`--inpm-color`) |
| `variant` | `'outline'\|'filled'\|'underline'\|'ghost'` | Variante visual |
| `size` | `'sm'\|'md'\|'lg'\|'xl'` | Tamanho |
| `iconStart` / `iconEnd` | string (HTML/SVG) | Ícones internos |
| `block` | boolean | Ocupa 100% da largura |
| `disabled` | boolean | Estado desabilitado |
| `required` | boolean | Marca obrigatório |
| `error` | string | Mensagem de erro (ativa estado de erro) |
| `helperText` | string | Texto de apoio |
| `id` / `name` | string | Atributos nativos |
| `className` | string | Classes extras no elemento raiz |
| `attrs` | object | Atributos HTML adicionais no `<input>` |
| `onInput` | function(event, value) | Disparado a cada digitação |
| `onFocus` / `onBlur` | function(event) | Eventos de foco |

**Diferença deliberada em relação ao botão:** o input não recebe `onClick`, e sim `onInput`/`onFocus`/`onBlur` — o contrato de eventos muda porque a interação do componente muda, mas o **formato** do contrato (objeto de props único, callbacks nomeados por evento) é o mesmo.

---

## 12. Organização de arquivos

```
input-modern.js        ← lógica, função-fábrica, JSDoc de props
input-modern.scss       ← estilos-fonte
input-modern.css        ← gerado a partir do .scss
demo-input-modern.html  ← demo standalone (CSS/JS embutidos, sem dependências externas)
```

**Atenção ao montar o demo:** se o HTML for aberto como arquivo isolado (fora de um servidor/pasta), `<link>` e `<script src>` apontando para arquivos externos podem não resolver dependendo de como o arquivo é aberto. Prefira embutir CSS/JS diretamente no HTML de demo (`<style>`/`<script>` inline) para garantir que funcione sozinho.

---

## 13. Passo a passo para desenvolver um novo campo modernista

1. **Defina a anatomia.** Liste as partes visuais necessárias (label? ícones? texto de apoio?) e monte a estrutura `label → field → hint`, mesmo que alguma parte não exista neste componente específico.
2. **Separe estilo do wrapper vs. do controle nativo.** Borda, fundo, radius e padding vivem no wrapper (`__field`); o elemento nativo (`<input>`, `<textarea>`, etc.) fica transparente e sem borda própria.
3. **Aplique a cor via custom property**, nunca via classe fixa ou valor hardcoded no JS.
4. **Implemente as 4 variantes** (`outline`, `filled`, `underline`, `ghost`) com a mesma lógica de bordas/fundo descrita na seção 6.
5. **Implemente os 4 tamanhos** com radius e padding crescentes (seção 7).
6. **Trate hover com `filter: brightness()`** apenas — nunca alterando fundo/borda/sombra.
7. **Trate foco separadamente do hover**, usando `border-color` (ou `brightness` na variante sem borda) — nunca `box-shadow`.
8. **Modele o estado de erro** como uma sobreposição visual (borda vermelha + hint vermelho), nunca como uma variante nova — erro deve funcionar em cima de qualquer variante.
9. **Escreva `create{Componente}` e `update{Componente}`**, com o segundo reaproveitando a função de render interna do primeiro (evita duplicar lógica de montagem do DOM).
10. **Exporte via `module.exports`/`window`**, no mesmo formato dos demais componentes.
11. **Rode o checklist da seção 14** antes de considerar o componente pronto.

---

## 14. Checklist para criar um novo campo modernista

- [ ] Segue a estrutura `label → field (wrapper visual) → hint`?
- [ ] Borda/fundo/radius aplicados no wrapper, não no elemento nativo?
- [ ] Cor aplicada via custom property (`--{componente}m-color`)?
- [ ] Hover usa só `filter: brightness()`, sem alterar fundo/borda/sombra?
- [ ] Foco usa `border-color` (ou `brightness` se a variante não tem borda), nunca `box-shadow`?
- [ ] `filter` está incluído na lista de `transition` do bloco base?
- [ ] Transições usam `cubic-bezier(.4, 0, .2, 1)`, não o `ease` padrão?
- [ ] Suporta as 4 variantes e 4 tamanhos com a mesma API de props do input de referência?
- [ ] Estado de erro sobrepõe qualquer variante, sem virar uma variante própria?
- [ ] `error` e `helperText` nunca renderizam ao mesmo tempo?
- [ ] Nome da função e das classes usa o sufixo/prefixo `-modern`?
- [ ] Demo é standalone (CSS/JS embutidos), sem depender de arquivos externos para abrir?