# table-modern — Documentação de Construção

Este documento registra **como e por que** `table-modern.js`/`.scss` foi construído, no papel de
terceiro documento do design system: `DESIGN-SYSTEM.md` (padrão clássico, arquitetura-base) →
`style-guide.md` (variante modernista, definida em cima do botão) → **este arquivo** (variante
modernista aplicada a um segundo componente, tabela). Ele existe para que o próximo componente
modernista (`select-modern`, `card-modern`, o que vier) tenha um segundo exemplo de referência
além do botão, e para que decisões que exigiram interpretação — porque o guia original descreve
regras pensadas para um botão, não para uma tabela — fiquem explícitas em vez de implícitas no
código.

---

## 1. Ponto de partida: o que já estava decidido, o que precisou ser decidido agora

O item 1 do `DESIGN-SYSTEM.md` e o corpo do `style-guide.md` já fixavam, antes de qualquer linha
de `table-modern` ser escrita:

- JS puro, sem TypeScript, sem framework/lib de runtime
- SCSS como fonte, `.css` gerado
- Props documentadas via JSDoc
- Par de funções `create{Componente}` / `update{Componente}`
- Exportação via `module.exports`/`window`
- Nomenclatura BEM, um bloco por componente
- Cor aplicada via custom property (`--{prefixo}-color`), nunca classe fixa
- Hover = só `filter: brightness()`, nunca troca de fundo/borda/sombra
- Pressed = só `transform: scale()`, nunca sombra interna
- Cantos arredondados por tamanho (10/12/14/16px conforme `sm`/`md`/`lg`/`xl`)
- Zero sombra, em qualquer estado, em qualquer variante
- `cubic-bezier(.4, 0, .2, 1)` nas transições de interação, nunca `ease` padrão
- `filter` sempre presente na lista de `transition` do bloco base
- Alta compatibilidade: nada de CSS recente (isso incluiu, na prática, banir `color-mix()`)

Nada disso foi renegociado. O trabalho de `table-modern` foi **traduzir** essas regras — escritas
pensando em um elemento pequeno e discreto — para um componente estrutural, denso em conteúdo e
com estados que um botão não tem (ordenação, seleção, carregamento, vazio). As seções 2 a 6
documentam essa tradução; a seção 7 documenta os dois pontos em que a tradução exigiu um desvio
deliberado da regra literal.

---

## 2. Mapeamento variante-a-variante: do botão para a tabela

| Regra do botão (`style-guide.md`) | Leitura literal | Tradução aplicada em `table-modern` |
|---|---|---|
| `solid` = fundo cheio | Área inteira preenchida | Só o **cabeçalho** preenchido com `--tablem-color` — preencher a tabela toda prejudicaria a leitura de uma grade densa de dados |
| `outline` = borda fina sempre visível | Borda ao redor do elemento | Borda de 1.5px ao redor do **container inteiro**, mais espessa no divisor cabeçalho/corpo |
| `ghost` = sem fundo em nenhum estado | — | Igual: sem fundo, hierarquia só por peso tipográfico |
| `link` = sem sublinhado | Remove o adorno padrão da variante | Traduzido para "sem nenhuma linha divisória entre linhas" — o adorno padrão de uma tabela é o divisor de linha, não um sublinhado |
| Hover = só `filter: brightness()` | Aplicado ao elemento clicado | Aplicado à **linha** inteira sob o cursor (`tr:hover`), não à tabela toda |
| Pressed = `transform: scale(.97)` | Compressão de ~3% | Recalibrado para `.995` em linhas clicáveis — ver §7.1 |
| Radius por tamanho (10–16px) | Aplicado ao próprio elemento | Aplicado ao **container**, com `overflow: hidden` na tabela interna para clipar os cantos do cabeçalho/última linha |

Em nenhum desses casos a *regra* mudou — o que mudou foi qual elemento do componente ela passou
a governar, porque uma tabela tem uma anatomia interna (cabeçalho, linha, célula, container) que
um botão não tem.

---

## 3. Arquitetura de props: o que é novo em relação ao botão

O contrato do botão (`text`, `color`, `fg`, `variant`, `size`, `iconStart`, `iconEnd`, `block`,
`disabled`, `type`, `id`, `className`, `attrs`, `onClick`) é o de um componente sem estado interno
de dados. Tabela tem estado de dados por natureza, então a API precisou de um bloco de props que
o botão nunca teve:

```js
columns, data, caption
sortable, sortKey, sortDir, onSort
selectable, selectedRowIds, rowIdKey, onSelectionChange
loading, emptyMessage
onRowClick
```

Duas decisões de design por trás desse bloco:

- **Componente controlado, não autônomo.** `table-modern` não ordena nem seleciona nada sozinho —
  `onSort`/`onSelectionChange` avisam a intenção, e quem consome decide o novo estado e chama
  `updateModernTable` de volta. Isso segue o mesmo espírito de `onClick` no botão (o componente
  notifica, não decide), só que agora há estado (`sortKey`, `sortDir`, `selectedRowIds`) a manter
  em sincronia — responsabilidade de quem usa, não do componente.
- **`columns` como fonte de verdade da estrutura**, com `render` custom por coluna. Evita que o
  componente precise conhecer o formato de cada dado de negócio; ele só sabe iterar `columns` e
  aplicar `render(value, row, rowIndex)` quando fornecido.

---

## 4. Decisões de acessibilidade (fora do escopo do `style-guide.md`, adicionadas aqui)

O guia do botão não trata de acessibilidade porque um `<button>` já carrega a semântica certa por
padrão. Uma `<table>` de dados exige decisões explícitas, tomadas com base em documentação MDN e
WAI-ARIA:

| Decisão | Onde no código | Por quê |
|---|---|---|
| `scope="col"` em todo `<th>` | `buildHeaderCell` | Associa cada cabeçalho à coluna inteira para leitores de tela — recomendação central do MDN para tabelas de dados |
| `<caption>` opcional como nome acessível | `render`, prop `caption` | É o mecanismo padrão de dar nome/descrição à tabela, equivalente ao `aria-label` de um `<button>` |
| `aria-sort` só no `<th>` da coluna ordenada no momento | `buildHeaderCell` | A spec ARIA exige que o atributo exista em no máximo um cabeçalho por vez — nunca na `<table>` |
| Cabeçalho ordenável é um `<button>` dentro do `<th>`, não o `<th>` inteiro clicável | `buildHeaderCell` | `<th>` não é nativamente focável/acionável por teclado; um `<button>` interno resolve isso sem ARIA extra |
| Linha clicável ganha `tabindex="0"` + handler de `Enter`/`Espaço` | `buildRow` | Mouse não é a única forma de acionar `onRowClick` |
| `prefers-reduced-motion: reduce` desativa `scale()` de pressed e troca o shimmer por pulso de opacidade | `table-modern.scss`, fim do arquivo | Movimento de posição (`background-position` animado, `transform: scale`) é o tipo de efeito que essa preferência do usuário existe para evitar; um pulso de opacidade ainda comunica "carregando" sem o movimento |

---

## 5. Estados que o botão não tem: `loading` e vazio

Botão não tem estado de "carregando dados" nem de "sem conteúdo" — só `disabled`. Tabela tem os
dois, e cada um recebeu uma solução própria, ambas seguindo a estética modernista já fixada:

- **`loading`**: linhas substituídas por barras de skeleton com gradiente animado (shimmer),
  `1.4s ease-in-out infinite`, larguras variadas em ciclo (`90% / 55% / 75% / 40% / 65%`) e atraso
  escalonado por linha — evita o efeito de "bloco piscando uniforme". Curva `ease-in-out`, e não a
  `$tablem-ease` (`cubic-bezier(.4,0,.2,1)`) usada nas interações: aquela é uma curva de entrada/
  saída pensada para uma transição que acontece uma vez; um loop contínuo pede simetria de
  ida-e-volta.
- **Vazio**: célula única, centralizada, com `emptyMessage` customizável — sem ilustração nem
  componente extra, mantendo a mesma economia visual do resto do sistema.

---

## 6. Cor sem `color-mix()`: o padrão a seguir daqui pra frente

O rascunho inicial deste componente usou `color-mix(in srgb, currentColor 8%, transparent)` para
derivar tons translúcidos de divisores e realces a partir da cor de acento — e isso **violava**
a regra de alta compatibilidade herdada do `DESIGN-SYSTEM.md` (`style-guide.md`, §3: *"sem
`color-mix()` ou qualquer recurso CSS recente"*). Foi corrigido, e o padrão resultante deve ser
reaproveitado em qualquer componente futuro que precise de tons derivados:

```scss
// Custom properties fixas em rgba(), independentes de --{prefixo}-color,
// sobrescrevíveis pelo consumidor quando necessário:
--tablem-divider: rgba(0, 0, 0, .08);
--tablem-divider-strong: rgba(0, 0, 0, .12);
--tablem-stripe: rgba(0, 0, 0, .04);
--tablem-selected-bg: rgba(99, 91, 255, .1);
```

Sem `color-mix()`, não há forma de alta compatibilidade de derivar "a cor de acento em 10% de
opacidade" dinamicamente a partir de `--{prefixo}-color`. A solução adotada — uma custom property
**separada e explícita** para cada tom derivado, com um valor fixo razoável de fallback — é o
padrão a repetir: mais verboso que `color-mix()`, mas dentro da regra de compatibilidade e ainda
assim sobrescrevível pelo consumidor tema a tema.

---

## 7. Desvios deliberados da regra literal (e por quê)

Dois pontos em que aplicar a regra do botão *ao pé da letra* teria produzido um resultado pior do
que a intenção original da regra:

### 7.1 — `transform: scale(.995)` em vez de `scale(.97)` no pressed

A regra diz `scale(.97)` para o feedback de clique. Esse valor foi calibrado para um botão —
elemento pequeno, tipicamente 32–48px de largura, onde 3% de compressão é um deslocamento sutil
de 1–2px. Uma linha de tabela ocupa a largura inteira do container; os mesmos 3% deslocam dezenas
de pixels nas bordas, o que deixa de ser "microinteração" e passa a ser um salto visível.
`table-modern` usa `.995` em linhas clicáveis: mesma linguagem de "compressão ao toque", calibrada
para a geometria do elemento. **Regra geral para componentes futuros**: `.97` é o valor de
referência para elementos de escala parecida com um botão; elementos que ocupam a largura do
container devem recalibrar a favor de uma compressão perceptualmente equivalente, não do mesmo
percentual numérico.

### 7.2 — Curva de easing diferente para o shimmer de loading

A regra fixa `cubic-bezier(.4, 0, .2, 1)` para transições de interação (hover, clique). O shimmer
de loading não é uma transição de interação — é uma animação em loop contínuo sem começo/fim
definido pelo usuário. Uma curva de ease-out (rápida no início, lenta no fim) pensada para "o
usuário fez algo, o elemento reage" não faz sentido para algo que se repete sozinho; `ease-in-out`
foi usado em seu lugar. **Regra geral**: `$tablem-ease`/`cubic-bezier(.4,0,.2,1)` vale para toda
transição disparada por uma ação do usuário; animações de loop autônomo (loading, skeleton, spinner
futuro) podem usar uma curva simétrica própria, desde que documentada como exceção — não uma
substituição silenciosa da regra.

---

## 8. Organização de arquivos

Segue a seção 9 do `style-guide.md`, com o sufixo `-modern` do próprio componente:

```
table-modern.js               ← lógica, função-fábrica, JSDoc de props
table-modern.scss             ← estilos-fonte
table-modern.css              ← gerado a partir do .scss (sass real, não escrito à mão)
demo-modern.html              ← demo com dependência externa (table-modern.css/.js por caminho relativo)
demo-modern-standalone.html   ← mesmo demo, com CSS/JS embutidos inline — usar esta versão
                                 sempre que o arquivo for aberto isoladamente (preview, e-mail,
                                 qualquer contexto sem os arquivos irmãos disponíveis)
TABLE-MODERN.md               ← este documento
```

Nota sobre `demo-comparativo.html` (item 9 do `style-guide.md`): o guia pede uma comparação lado a
lado com a versão clássica do componente. Não existe ainda um `table.js` clássico no design
system, então esse comparativo não pôde ser feito — fica pendente para quando o padrão clássico de
tabela existir.

---

## 9. Checklist — componente modernista construído sobre um segundo elemento estrutural

Complementa a checklist do item 10 do `style-guide.md` (pensada para elementos simples/atômicos
como um botão) para componentes com estado de dados próprio:

- [ ] Todas as regras visuais do §2 deste documento foram mapeadas explicitamente para a anatomia
      do novo componente (não só copiadas do botão sem adaptação)?
- [ ] Cor via custom property, e qualquer tom derivado dela é uma custom property própria em
      `rgba()` fixo — nunca `color-mix()` (§6)?
- [ ] Hover aplicado à unidade de interação correta do componente (linha, item de lista, célula —
      não ao componente inteiro de uma vez)?
- [ ] Se houver `transform: scale()` de pressed, o valor foi recalibrado para a geometria do
      elemento, não copiado de `.97` sem checar a largura típica (§7.1)?
- [ ] Toda animação de loop autônomo (loading, skeleton) usa curva própria documentada como
      exceção à `$tablem-ease`, em vez de reusar a curva de interação (§7.2)?
- [ ] `prefers-reduced-motion: reduce` cobre toda animação de movimento do componente (não só as
      transições de hover/clique)?
- [ ] Estados que o botão não tinha (loading, vazio, seleção, etc.) foram resolvidos com a mesma
      economia visual do resto do sistema — sem introduzir sombra, borda grossa ou cor fora da
      paleta definida por `--{prefixo}-color`?
- [ ] Semântica HTML nativa (`<table>`, `<th scope>`, `<caption>`, etc.) foi preservada, e ARIA só
      foi adicionado onde a semântica nativa não bastava?
- [ ] Componente controlado ou autônomo? Se controlado (como `table-modern`), os callbacks
      (`onX`) notificam intenção e não alteram estado sozinhos — decisão documentada, não implícita?
- [ ] Novas custom properties de cor/tom seguem o prefixo do componente (`--tablem-*`) e estão
      listadas com seus valores de fallback num só lugar no topo do SCSS?