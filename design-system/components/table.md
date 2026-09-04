# Table

Exibição de dados tabulares — linhas e colunas com cabeçalho fixo, seleção opcional e ordenação por coluna. Usado para listagens densas (registros, transações, itens administráveis), diferente do `Card`, que é para blocos de conteúdo agrupado.

---

## Anatomia

```
Table                          (wrapper com borda + radius, scroll horizontal se necessário)
├── Table.Header
│   └── Table.HeaderRow
│       ├── Table.HeaderCell     (label, sortable?, width?)
│       └── ...
├── Table.Body
│   ├── Table.Row                 (selected?, disabled?, onClick?)
│   │   ├── Table.Cell
│   │   └── ...
│   └── Table.EmptyState          (renderizado no lugar das linhas quando não há dados)
└── Table.Footer                  (opcional — paginação, resumo, totais)
```

Assim como o `Modal`, o `Table` renderiza semânticamente um `<table>` HTML real por baixo da composição — `Table.Header`/`Table.Body` viram `<thead>`/`<tbody>`, `Table.HeaderRow`/`Table.Row` viram `<tr>`, `Table.HeaderCell`/`Table.Cell` viram `<th>`/`<td>`. A API de composição existe para facilitar o uso, não para abandonar a semântica nativa da tabela.

---

## Props

**`Table`**

| Prop                | Tipo                      | Default     | Descrição                                                                    |
| ------------------- | ------------------------- | ----------- | ---------------------------------------------------------------------------- |
| `size`              | `"sm" \| "md" \| "lg"`    | `"md"`      | Altura da linha — mapeia direto para `size-sm/md/lg` do `tokens.md`          |
| `variant`           | `"default" \| "bordered"` | `"default"` | `bordered` adiciona divisórias verticais entre células, além das horizontais |
| `stickyHeader`      | `boolean`                 | `false`     | Cabeçalho fixo ao rolar verticalmente dentro do wrapper                      |
| `selectable`        | `boolean`                 | `false`     | Adiciona coluna de checkbox em `Table.HeaderRow`/`Table.Row`                 |
| `loading`           | `boolean`                 | `false`     | Substitui as linhas por placeholders estáticos (ver Estados)                 |
| `selectedRows`      | `string[]`                | `[]`        | IDs das linhas selecionadas (controlado)                                     |
| `onSelectionChange` | `(ids: string[]) => void` | —           | Handler de mudança de seleção                                                |

**`Table.HeaderCell`**

| Prop            | Tipo                      | Default | Descrição                                       |
| --------------- | ------------------------- | ------- | ----------------------------------------------- |
| `sortable`      | `boolean`                 | `false` | Habilita clique/teclado para alternar ordenação |
| `sortDirection` | `"asc" \| "desc" \| null` | `null`  | Direção atual (controlado pelo consumidor)      |
| `onSort`        | `() => void`              | —       | Handler de clique/Enter no header ordenável     |
| `width`         | `string`                  | —       | Largura fixa da coluna (ex: `"120px"`)          |

**`Table.Row`**

| Prop       | Tipo         | Default | Descrição                                                 |
| ---------- | ------------ | ------- | --------------------------------------------------------- |
| `disabled` | `boolean`    | `false` | Linha não selecionável/clicável, usa `text-muted`         |
| `onClick`  | `() => void` | —       | Torna a linha inteira clicável (ex: navegar para detalhe) |

Não existe prop `radius` em `Table` — assim como `Card`, usa `radius-md` fixo do wrapper (pilar 5 de `principles.md`).

---

## Variantes / Tamanhos

**`size`** controla a altura da linha e o padding vertical da célula — usa exatamente a mesma escala de `Button`/`Input`/`Select`:

| `size` | Altura da linha    | Padding horizontal da célula | Font-size          |
| ------ | ------------------ | ---------------------------- | ------------------ |
| `sm`   | 32px               | `space-3` (12px)             | `text-sm` (13px)   |
| `md`   | 40px — **default** | `space-4` (16px)             | `text-base` (14px) |
| `lg`   | 48px               | `space-6` (24px)             | `text-md` (15px)   |

**`variant`**:

- `default` — só divisórias horizontais (`border-subtle` entre linhas), uso geral.
- `bordered` — divisórias horizontais e verticais, útil em tabelas muito densas onde alinhar colunas visualmente ajuda a leitura (ex: dados financeiros, planilhas).

Nenhuma variante introduz sombra ou zebra de cor saturada — se listras alternadas forem necessárias por legibilidade, o tom usado é `fill-disabled` (o mesmo tom neutro de "desabilitado", só que aqui reaproveitado como fundo de linha par), nunca uma cor nova fora da paleta semântica.

---

## Estados

| Estado                     | Aplica-se a                                | Comportamento                                                                                                                                                                                                                                                   |
| -------------------------- | ------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `default`                  | linha                                      | `surface-1`, divisória inferior `border-subtle`.                                                                                                                                                                                                                |
| `hover`                    | linha com `onClick`                        | Divisória inferior muda para `border-strong` — mesmo tratamento de "linha interativa" do `Card`, sem fundo colorido nem sombra.                                                                                                                                 |
| `selected`                 | linha (`selectable`)                       | Fundo `bg-accent` a 8% de opacidade (mais discreto que o Badge, pois cobre uma área bem maior da tela).                                                                                                                                                         |
| `focus-visible`            | checkbox, header ordenável, linha clicável | Anel de foco visível, nunca suprimido.                                                                                                                                                                                                                          |
| `disabled`                 | linha                                      | `text-muted` no conteúdo, checkbox desabilitado, sem hover/click.                                                                                                                                                                                               |
| `loading`                  | tabela inteira                             | Linhas reais substituídas por blocos estáticos em `fill-disabled` (mesma contagem/altura de linha configurada). **Sem animação de pulso/shimmer** — o sistema não usa movimento ambiente contínuo (pilar 9), só transições disparadas por ação real do usuário. |
| vazio (`Table.EmptyState`) | tabela sem dados                           | Texto centralizado em `text-secondary`, substitui o corpo da tabela.                                                                                                                                                                                            |

---

## Tokens usados

| Token            | Uso                                                                         |
| ---------------- | --------------------------------------------------------------------------- |
| `surface-1`      | fundo do wrapper e das linhas                                               |
| `border-subtle`  | divisória horizontal entre linhas (e vertical, no `variant="bordered"`)     |
| `border-strong`  | borda externa do wrapper, divisória de linha em `hover`                     |
| `border-width`   | espessura de todas as divisórias (0.5px, fixo do sistema)                   |
| `bg-accent`      | fundo de linha `selected` (8% opacidade) e anel de foco                     |
| `fill-disabled`  | blocos do estado `loading`, fundo de linha par (se listrado)                |
| `text-primary`   | conteúdo padrão das células, cabeçalhos de coluna                           |
| `text-secondary` | mensagem do `Table.EmptyState`                                              |
| `text-muted`     | conteúdo de linha `disabled`                                                |
| `size-sm/md/lg`  | altura da linha, padding horizontal e font-size por `size`                  |
| `radius-md`      | radius do wrapper (cantos externos apenas — células não têm radius próprio) |
| `space-2`        | gap entre conteúdo e ícone de ordenação no header                           |
| `space-3/4/6`    | padding horizontal da célula conforme `size`                                |

`Table` **não** usa `shadow-overlay` nem tokens de Movimento — não é um componente de overlay (pilar 3 e pilar 9 de `principles.md`).

---

## Acessibilidade

- `Table.HeaderCell` ordenável usa `role="columnheader"` com `aria-sort` (`"ascending"` / `"descending"` / `"none"`), atualizado conforme `sortDirection`. Deve ser navegável e ativável por teclado (`Tab` + `Enter`/`Espaço`), não só por clique do mouse.
- Coluna de seleção: cada checkbox precisa de `aria-label` descrevendo a linha (ex: `aria-label="Selecionar linha: Pedido #4821"`); o checkbox do header (selecionar tudo) usa `aria-label="Selecionar todas as linhas"` e reflete estado indeterminado (`aria-checked="mixed"`) quando a seleção é parcial.
- Linha com `onClick` (navegação/detalhe) deve também ser acionável via teclado — na prática, envolver o conteúdo clicável em elemento focável (`role="button"` na célula relevante, ou tornar a primeira célula um link), nunca depender só do `onClick` do `<tr>`, que não recebe foco nativamente.
- Quando a tabela tem scroll horizontal (mais colunas do que cabe na viewport), o wrapper com scroll deve ser focável (`tabIndex="0"`) e ter `aria-label` descrevendo o conteúdo, para que teclado/leitor de tela consigam navegar o scroll.
- Estado `loading`: o wrapper recebe `aria-busy="true"`; os blocos placeholder são `aria-hidden="true"` (não são conteúdo real, não devem ser lidos).
- `Table.EmptyState` deve ser um texto real dentro da estrutura da tabela (uma linha com `colSpan` cobrindo todas as colunas), não um elemento solto fora do `<table>` — mantém a leitura coerente por assistive tech.

---

## Exemplo de código (React)

```jsx
import { Table } from "./Table";

<Table
  size="md"
  selectable
  selectedRows={selected}
  onSelectionChange={setSelected}
>
  <Table.Header>
    <Table.HeaderRow>
      <Table.HeaderCell sortable sortDirection={sort} onSort={toggleSort}>
        Cliente
      </Table.HeaderCell>
      <Table.HeaderCell width="140px">Status</Table.HeaderCell>
      <Table.HeaderCell width="120px">Valor</Table.HeaderCell>
    </Table.HeaderRow>
  </Table.Header>

  <Table.Body>
    {orders.length === 0 ? (
      <Table.EmptyState>Nenhum pedido encontrado</Table.EmptyState>
    ) : (
      orders.map((order) => (
        <Table.Row key={order.id} onClick={() => openOrder(order.id)}>
          <Table.Cell>{order.customerName}</Table.Cell>
          <Table.Cell>
            <Badge variant={order.status === "paid" ? "success" : "neutral"}>
              {order.statusLabel}
            </Badge>
          </Table.Cell>
          <Table.Cell>{order.formattedTotal}</Table.Cell>
        </Table.Row>
      ))
    )}
  </Table.Body>
</Table>;
```

```css
.table-wrapper {
  background: var(--surface-1);
  border: var(--border-width) solid var(--border-strong);
  border-radius: var(--radius-md);
  overflow-x: auto;
}

.table {
  width: 100%;
  border-collapse: collapse;
}

.table th,
.table td {
  text-align: left;
  border-bottom: var(--border-width) solid var(--border-subtle);
  font-family: var(--font-family);
  color: var(--text-primary);
}

.table--bordered th,
.table--bordered td {
  border-right: var(--border-width) solid var(--border-subtle);
}
.table--bordered th:last-child,
.table--bordered td:last-child {
  border-right: none;
}

.table--sm th,
.table--sm td {
  height: 32px;
  padding: 0 var(--space-3);
  font-size: var(--text-sm);
}
.table--md th,
.table--md td {
  height: 40px;
  padding: 0 var(--space-4);
  font-size: var(--text-base);
}
.table--lg th,
.table--lg td {
  height: 48px;
  padding: 0 var(--space-6);
  font-size: var(--text-md);
}

.table__row--hover:hover {
  border-bottom-color: var(--border-strong);
  cursor: pointer;
}
.table__row--selected {
  background: color-mix(in srgb, var(--bg-accent) 8%, transparent);
}
.table__row--disabled {
  color: var(--text-muted);
}

.table__header--sticky {
  position: sticky;
  top: 0;
  background: var(--surface-1);
  z-index: 1;
}

.table__loading-block {
  background: var(--fill-disabled);
  border-radius: var(--radius-sm);
  height: 14px;
}
```

---

## Estabilidade entre versões

Segue as regras gerais de `principles.md`. Específico deste componente:

1. `size` segue exatamente a mesma escala de `Button`/`Input`/`Select` — não introduzir um `size` exclusivo do Table, para manter alinhamento visual quando esses componentes aparecem juntos (ex: filtros acima de uma tabela).
2. `variant` (`default`/`bordered`) é vocabulário público — não renomear. Novas variantes futuras (ex: `"compact"` como atalho pra `size="sm"` + `variant="bordered"`) só devem ser adicionadas se justificadas por uso real repetido, não especulativamente.
3. A composição (`Table.Header`, `Table.Body`, `Table.Row`, `Table.Cell`...) é o contrato de API — a ordem e a obrigatoriedade de cada subcomponente não deve mudar sem depreciação, pelo mesmo motivo do `Card`.
4. `loading` renderiza placeholders estáticos, sem animação — se uma futura versão quiser adicionar indicação de carregamento mais rica, isso deve ser revisado contra o pilar 9 antes de introduzir qualquer movimento contínuo, não assumido como aceitável por analogia com outras bibliotecas.
5. Ausência de tokens de Movimento é deliberada — `Table` não é overlay. Se `stickyHeader` algum dia precisar de uma transição de sombra/borda ao rolar, isso deve ser avaliado como possível exceção pontual, documentada aqui, não herdada silenciosamente de `Modal`/`Select`.
