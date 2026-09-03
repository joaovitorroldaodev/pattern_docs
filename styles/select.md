# Padrão de Componente — Select (Variante Modernista)

Este documento orienta como desenvolver o componente de **select** na linguagem visual **modernista**, seguindo a mesma arquitetura de código do `input-modern` (JS puro, SCSS, props via JSDoc, anatomia `label → field → hint`) e estendendo-a para as particularidades de um campo de seleção: lista de opções, filtro por digitação, posicionamento flutuante e scroll interno.

`createModernSelect` (`select-modern.js` / `select-modern.scss`) é a implementação de referência.

> **Nota de proveniência:** assim como o `input-modern`, este componente foi desenvolvido sem acesso a um `DESIGN-SYSTEM.md`/`select-classic.js` de referência. As decisões marcadas como **[inferido]** foram extrapoladas do padrão do input e devem ser revisadas contra uma implementação clássica real, se/quando ela existir.

---

## 1. Quando usar este padrão

- Sempre que o produto/tela já usa a variante modernista para os demais campos — **um mesmo formulário não deve misturar select clássico e modernista**.
- Use `select-modern` quando a escolha for entre um conjunto **fechado** de opções (uma de várias). Para "digite e crie uma opção nova" ou seleção múltipla, este componente precisa ser estendido — ver seção 15.

---

## 2. Anatomia do componente

O select modernista tem uma parte a mais que o input: a **listbox flutuante**. Ela não fica dentro do `__field` — fica posicionada sobre ele, ancorada por um wrapper específico:

```
.select-modern
├── .select-modern__label        (opcional)
├── .select-modern__wrap         (âncora de posicionamento — position: relative)
│   ├── .select-modern__field    (moldura visual: borda/fundo/radius vivem aqui)
│   │   ├── .select-modern__icon--start   (opcional)
│   │   ├── .select-modern__control       (input nativo — digitável, sem borda própria)
│   │   └── .select-modern__icon--end     (chevron, sempre presente)
│   ├── .select-modern__listbox  (painel flutuante, fechado por padrão)
│   │   └── .select-modern__option  × N
│   └── select (nativo, escondido — ver seção 8)
└── .select-modern__hint         (opcional: helper text OU mensagem de erro)
```

**Por que a listbox é HTML nosso, e não `<option>` nativo [inferido]:** a primeira versão deste componente usou um `<select>` nativo dentro do `__field`. O problema: o navegador controla 100% da aparência da lista de opções aberta — não há CSS que sobreponha radius, cor de hover ou ausência de sombra de forma consistente entre browsers. O resultado destoava visualmente do resto do design system, parecendo um componente emprestado de outro lugar. Renderizar a lista como `<ul>`/`<li>` comuns resolve isso: ela recebe exatamente as mesmas regras visuais do `__field`.

---

## 3. Estilo visual — regras herdadas do input, com duas exceções conscientes

| Característica | Como aplicar |
|---|---|
| Bordas do `__field` | Ausentes por padrão. Só `outline` (sempre) e `filled`/`error` (só no erro) têm borda, fina (1.5px) |
| Cantos | Bem arredondados (10–16px conforme o tamanho) — nunca retos, exceto `underline` |
| Profundidade do `__field` | Nenhuma, em nenhum estado — sem sombra |
| Hover do `__field` | Só `filter: brightness()` — nunca fundo/borda/sombra |
| Tipografia | Fonte geométrica (Inter ou equivalente), peso 500, leve `letter-spacing` |
| Transições | `background-color`, `color`, `border-color`, `filter` — sempre com `filter` incluído. `cubic-bezier(.4, 0, .2, 1)`, ~0.22s |

A listbox flutuante segue as mesmas regras de radius/cor/ausência-de-sombra do `__field`, com **duas exceções conscientes**, porque ela é uma superfície flutuante sobre a página — não um campo "encaixado" no layout:

1. **A listbox sempre tem borda** (1.5px, cor neutra), mesmo nas variantes `filled`, `underline` e `ghost`, que não têm borda no `__field`. Sem sombra permitida em lugar nenhum, um painel branco flutuando sobre a página precisa de algum contorno para se separar do fundo — a borda aqui cumpre o papel que a sombra cumpriria em outro design system.
2. **Os itens da lista mudam de fundo no hover/seleção** (`background-color: $selm-neutral-bg`), diferente da regra "hover só com `filter`" do `__field`. Essa regra existe para o `__field` porque ele é uma superfície única (um campo, um botão); dentro de uma lista de opções, destacar o item sob o cursor é o comportamento esperado de qualquer menu — a mesma lógica do `ghost` do botão **clássico** ("fundo aparece no hover"), só que aplicada a item de lista, não a um componente inteiro.

Evitar: sombra em qualquer parte (inclusive a listbox — a borda substitui a sombra, não some com a regra), recursos CSS recentes como `:has()` ou `color-mix()` (o guia pede alta compatibilidade — ver nota na seção 9 sobre como o foco foi resolvido sem eles).

---

## 4. Cor

Mesma lógica dos demais componentes — cor via **custom property**:

```js
selectEl.style.setProperty('--selm-color', '#635bff');
```

Usada como cor de foco (borda ou texto da opção selecionada) e para destacar o item selecionado na listbox (`font-weight` + `color: var(--selm-color)`). Não vira background do `__field` nem da listbox, pelo mesmo motivo do input: confundiria com estado de erro.

---

## 5. Foco **[inferido]**

O `input-modern` resolve foco com `border-color` via `:focus-within`. O select **não pode** usar `:focus-within`/`:has()` da forma direta, porque:

- O guia pede alta compatibilidade, evitando recursos CSS recentes — `:has()` é recente demais para essa régua.
- O foco "de verdade" está no `<input>` (`__control`), mas o indicador visual precisa aparecer no `__field` (o wrapper com a borda), não no input isolado.

**Solução adotada:** o JS alterna uma classe (`select-modern__field--focused`) no `focus`/`blur` do input, e o CSS reage a essa classe — não a um seletor de foco do CSS. O mesmo padrão vale para `--open` (lista aberta): também é uma classe controlada por JS, não um pseudo-seletor.

---

## 6. Variantes

Mesmas 4 variantes do input, mesma lógica de borda/fundo no `__field`. A listbox **não muda entre variantes** — sempre com borda neutra, independente de qual variante o `__field` usa (ver seção 3).

| Variante | Borda do `__field` | Fundo do `__field` |
|---|---|---|
| `outline` (padrão) | Fina (1.5px), sempre visível | Nenhum |
| `filled` | Nenhuma (exceto erro) | Neutro chapado |
| `underline` | Só linha inferior | Nenhum |
| `ghost` | Nenhuma | Nenhum |

Classe: `.select-modern--{variante}`.

---

## 7. Tamanhos

Mesma escala `sm`/`md`/`lg`/`xl`, radius e padding no `__field` (mesmos valores do input). A diferença: o tamanho também calibra a **altura máxima da listbox**, para caber ~5 opções antes de precisar rolar (ver seção 10).

```
sm  →  radius 10px  →  listbox até ~5 × 32px + padding
md  →  radius 12px  →  listbox até ~5 × 36px + padding  (padrão)
lg  →  radius 14px  →  listbox até ~5 × 40px + padding
xl  →  radius 16px  →  listbox até ~5 × 44px + padding
```

---

## 8. Select nativo escondido

Por baixo do combobox visível existe um `<select>` nativo, sincronizado com o valor atual, mas com `aria-hidden="true"` e `tabIndex="-1"`. Ele não participa da navegação nem é lido por leitor de tela — sua única função é manter o valor disponível em `FormData`/submit nativo, para quem depende disso fora do próprio componente.

**Não usar `display: none`** nesse select — isso o excluiria de `FormData` em alguns navegadores. A técnica usada é posicioná-lo fora da área visível (`width: 1px; height: 1px; opacity: 0`), mantendo-o funcional.

---

## 9. Filtro por digitação **[inferido]**

O `__control` não é mais somente-leitura — é um `<input type="text">` que aceita digitação, seguindo o padrão de **combobox editável** do ARIA Authoring Practices Guide.

Regras de comportamento:

- **Abrir sem digitar** (clique, foco, seta ↑/↓) mostra todas as opções.
- **Digitar filtra** por substring (case-insensitive) nas labels das opções — a lista some as que não contêm o texto.
- **Só uma opção da lista confirma o valor.** Se o campo perder o foco (ou Esc for pressionado) com texto digitado que não veio de uma seleção, o texto reverte para a opção selecionada atual (ou fica vazio, se nenhuma). Isso evita que texto livre digitado vire um "valor fantasma" que não existe nas opções — importante porque o componente representa uma escolha de um conjunto fechado, não um campo de texto livre.
- **Seleção acontece no `mousedown` da opção, com `preventDefault`** — não no `click`. É o truque padrão do padrão combobox: se a seleção esperasse o `click`, o `blur` do input (que fecha a lista) dispararia antes e a opção sumiria da tela antes do clique ser processado.

Este comportamento é uma extensão razoável do "aceita digitação" pedido, mas é uma decisão de produto, não só de estilo — se o produto precisar que qualquer texto digitado seja aceito como valor livre (não vinculado às opções), este componente não é o ponto de partida certo; seria um campo de texto com sugestões (`datalist`-like), não um select.

---

## 10. Posicionamento e scroll da listbox **[inferido]**

Dois comportamentos que não existem no input, porque o input não abre um painel flutuante:

**Flip vertical:** por padrão a listbox abre para baixo do `__field`. Antes de cada abertura (e a cada filtragem, já que o número de opções — e portanto a altura da lista — muda ao digitar), o JS mede o espaço disponível acima e abaixo do campo na viewport. Se não houver espaço suficiente abaixo **e** houver mais espaço acima, uma classe (`select-modern__wrap--drop-up`) inverte a ancoragem — mesmo painel, mesma aparência, só o lado que muda.

Essa decisão é recalculada a cada abertura, não fica reagindo continuamente a scroll/resize com a lista já aberta — se o produto precisar disso (ex.: listas dentro de modais que rolam), é uma extensão futura, não o comportamento padrão.

**Scroll interno após 5 itens:** a listbox tem `max-height` calibrado por tamanho (seção 7) para caber aproximadamente 5 opções. Com 5 ou menos, o conteúdo é mais curto que o `max-height` e não aparece barra de rolagem. Com mais de 5, `overflow-y: auto` entra em ação e a lista rola **dentro do próprio painel** — a página ao redor não se move. A opção ativa (navegação por teclado) sempre chama `scrollIntoView({ block: 'nearest' })` para acompanhar o scroll.

---

## 11. Estados

Mesma tabela do input (seção 8 do `GUIA-input-modern.md`), com uma adição: quando a listbox está filtrada e nenhuma opção corresponde ao texto digitado, ela mostra um item não-interativo ("Nenhum resultado encontrado") em vez de ficar vazia — feedback de que o filtro rodou e não achou nada, diferente de "a lista ainda não carregou".

| Estado | Como ativar | Comportamento |
|---|---|---|
| Obrigatório | `required: true` | Asterisco no label |
| Erro | `error: 'mensagem'` | Borda vermelha no `__field` (mesmo em `filled`/`underline`), hint vermelho |
| Helper text | `helperText: 'texto'` | Hint neutro — some se `error` estiver presente |
| Desabilitado | `disabled: true` | Campo não reage a clique/hover/teclado, fundo neutro fixo |
| Sem resultados | (automático) | Item não-interativo na listbox durante filtro sem match |

---

## 12. Ícones

Mesma convenção: `iconStart` recebe HTML/SVG cru, fica à esquerda dentro do `__field`. O chevron (`__icon--end`) é sempre gerado internamente pelo componente (não é uma prop) — ele sinaliza "isto abre uma lista" e gira 180° quando a lista está aberta (`select-modern__field--open`), mesma família de técnica que o `transform: scale()` de clique do botão.

---

## 13. Nomenclatura de classes

```
.select-modern
.select-modern--{variante|tamanho|estado}
.select-modern__{elemento}
.select-modern__option--{estado}
```

O `__wrap` (âncora de posicionamento) é específico deste tipo de componente — qualquer campo futuro com painel flutuante (autocomplete, date picker) deve seguir o mesmo padrão: `wrap` (position: relative) contendo `field` + o painel flutuante, em vez de tentar posicionar o painel relativo ao `root`.

---

## 14. API JavaScript

```js
createModernSelect(props)
updateModernSelect(el, novasProps)
```

| Prop | Tipo | Descrição |
|---|---|---|
| `label` | string | Texto acima do campo |
| `options` | `{value, label, disabled?}[]` | Lista de opções |
| `placeholder` | string | Texto exibido via `placeholder` nativo quando nada foi digitado/selecionado |
| `value` | string | Valor selecionado inicial |
| `color` | string | Cor de destaque (`--selm-color`) |
| `variant` | `'outline'\|'filled'\|'underline'\|'ghost'` | Variante visual |
| `size` | `'sm'\|'md'\|'lg'\|'xl'` | Tamanho (também calibra o `max-height` da listbox) |
| `iconStart` | string (HTML/SVG) | Ícone à esquerda |
| `block` | boolean | Ocupa 100% da largura |
| `disabled` | boolean | Estado desabilitado |
| `required` | boolean | Marca obrigatório |
| `error` | string | Mensagem de erro |
| `helperText` | string | Texto de apoio |
| `id` / `name` | string | Atributos nativos (`name` vai no `<select>` escondido) |
| `className` | string | Classes extras no elemento raiz |
| `attrs` | object | Atributos HTML adicionais no `<select>` nativo escondido |
| `onChange` | function(event, value) | Disparado quando uma opção é confirmada (não a cada tecla) |
| `onFocus` / `onBlur` | function(event) | Eventos de foco do campo |

**Diferença deliberada em relação ao input:** não existe `onInput` — digitar só filtra a lista, não é um valor em si. `onChange` só dispara quando uma opção é de fato selecionada, para não confundir "usuário está digitando/filtrando" com "usuário escolheu um valor".

---

## 15. Fora do escopo desta versão

Registrado aqui para quem for estender o componente, não esquecer/reconstruir do zero:

- **Seleção múltipla.** Exigiria repensar `__value` (hoje um `<input>` de texto único) para algo como chips dentro do `__field`, e o `<select>` escondido passaria a ser `multiple`.
- **Criar opção nova a partir do texto digitado** ("tag input"). O comportamento atual reverte texto não confirmado — um modo "criar opção" precisaria de uma prop explícita (`allowCreate`) e um item especial no fim da listbox filtrada.
- **Reposicionamento contínuo** durante scroll/resize com a lista aberta (hoje só recalcula ao abrir/filtrar).
- **Virtualização** para listas muito longas (centenas de opções) — hoje toda a lista é renderizada, só a exibição é que rola.

---

## 16. Passo a passo para desenvolver um campo com painel flutuante

Este roteiro generaliza o que foi feito aqui para qualquer campo futuro que precise abrir um painel sobre a página (autocomplete, date picker, color picker):

1. **Separe `wrap` de `field`.** O `wrap` é a âncora de posicionamento (`position: relative`); o painel flutuante é posicionado `absolute` dentro dele, nunca relativo ao `root` inteiro (que também contém label/hint, deslocando o cálculo).
2. **Decida o que do painel flutuante quebra as regras do field** — e documente por quê. Aqui foi a borda (substituindo a sombra proibida) e o hover dos itens (comportamento de lista, não de superfície única). Não copie essas exceções cegamente para outro componente sem reavaliar se fazem sentido nele.
3. **Resolva foco/estado aberto via classe controlada por JS**, não via pseudo-seletor CSS recente (`:has()`, `:focus-within` quando ele não cobre o caso). Mantém a régua de compatibilidade do guia.
4. **Implemente o flip vertical medindo a viewport antes de abrir**, não fixando uma direção. Recalcule sempre que o conteúdo do painel mudar de tamanho (ex.: filtro).
5. **Limite a altura do painel a um número fixo de itens visíveis** (aqui, 5) via `max-height` + `overflow-y: auto`, calibrado por tamanho — não deixe o painel crescer sem limite.
6. **Se o campo aceitar digitação, decida explicitamente** o que acontece com texto que não corresponde a nenhuma opção — reverter (como aqui) ou permitir criar uma opção nova são dois produtos diferentes; não deixe isso indefinido.
7. **Selecione no `mousedown` com `preventDefault`, não no `click`**, sempre que a seleção precisar sobreviver ao campo perdendo o foco.
8. **Mantenha um elemento nativo equivalente escondido** (`aria-hidden`, fora do fluxo de tab, sem `display:none`) quando o componente substituir um elemento de formulário nativo — garante compatibilidade com submit/FormData sem exigir JS extra de quem consome o componente.

---

## 17. Checklist para criar um novo campo com painel flutuante

- [ ] `wrap` separado do `field`, com `position: relative` só no `wrap`?
- [ ] Painel flutuante com a mesma linguagem visual do field (radius por tamanho, cor via custom property)?
- [ ] Exceções às regras gerais (borda no painel, hover com fundo nos itens) documentadas com o motivo?
- [ ] Foco/estado aberto via classe JS, sem depender de `:has()` ou seletores CSS recentes?
- [ ] Flip vertical calculado antes de abrir (e recalculado se o conteúdo mudar de tamanho)?
- [ ] Altura do painel limitada a um número fixo de itens, com scroll interno após isso?
- [ ] Elemento nativo equivalente mantido escondido, sem `display: none`, para compatibilidade de formulário?
- [ ] Seleção de item via `mousedown` + `preventDefault` (não `click`)?
- [ ] Comportamento de texto não confirmado (se o campo aceitar digitação) decidido e documentado?
- [ ] Demo standalone (CSS/JS embutidos), com casos de teste para flip e para scroll da lista?