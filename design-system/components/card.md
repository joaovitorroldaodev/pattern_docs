# Card

Container de conteúdo agrupado — a unidade estrutural usada para exibir um item, um resumo ou um bloco de conteúdo relacionado dentro de listas, grids ou dashboards.

---

## Anatomia

```
Card
├── Card.Header      (opcional — título + descrição)
│   ├── title
│   └── description
├── Card.Body        (conteúdo livre — obrigatório)
└── Card.Footer       (opcional — ações, geralmente Button)
```

`Card` é o container. `Card.Header`, `Card.Body` e `Card.Footer` são subcomponentes de composição — nenhum é obrigatório exceto `Card.Body`, e a ordem é sempre Header → Body → Footer quando presentes.

---

## Props

| Prop          | Tipo                      | Default     | Descrição                                                                                                       |
| ------------- | ------------------------- | ----------- | --------------------------------------------------------------------------------------------------------------- |
| `as`          | `ElementType`             | `'div'`     | Elemento semântico renderizado. Usar `'article'` para itens de listagem, `'a'`/`'button'` quando `interactive`. |
| `size`        | `'sm' \| 'md' \| 'lg'`    | `'md'`      | Densidade do padding interno.                                                                                   |
| `variant`     | `'default' \| 'outlined'` | `'default'` | `outlined` usa `border-strong` em vez de `border-subtle` — mais ênfase estrutural, sem introduzir sombra.       |
| `interactive` | `boolean`                 | `false`     | Habilita estado `hover`/`focus-visible` e comportamento de clique. Exige `as='button'` ou `as='a'`.             |
| `disabled`    | `boolean`                 | `false`     | Só válido com `interactive`. Bloqueia interação e usa `fill-disabled`.                                          |
| `bg`          | `string`                  | —           | Override pontual de `--bg` (fundo). Ver `tokens.md`.                                                            |
| `fg`          | `string`                  | —           | Override pontual de `--fg` (texto).                                                                             |
| `className`   | `string`                  | —           | Classes adicionais.                                                                                             |
| `children`    | `ReactNode`               | —           | Conteúdo — tipicamente `Card.Header`, `Card.Body`, `Card.Footer`.                                               |

Não existe prop `radius`: Card usa `radius-md` fixo, conforme a escala única de radius do sistema (pilar 5 de `principles.md`).

---

## Variantes / Tamanhos

**`size`** controla o padding interno (não a altura — Card não é um componente de altura fixa como Input/Button):

| `size` | Padding                    |
| ------ | -------------------------- |
| `sm`   | `space-4` (16px)           |
| `md`   | `space-5` (20px) — default |
| `lg`   | `space-6` (24px)           |

**`variant`**:

- `default` — `border-subtle`, uso geral em grids e listas densas.
- `outlined` — `border-strong`, para quando o Card precisa se destacar sozinho na tela (ex: card único de destaque, formulário isolado).

Nunca combinar `outlined` com sombra para reforçar destaque — isso viola o pilar de "bordas finas, sem sombra decorativa". Destaque adicional vem de `bg-accent` pontual via prop `bg`, não de elevação.

---

## Estados

| Estado          | Aplica-se a                       | Comportamento                                                                                 |
| --------------- | --------------------------------- | --------------------------------------------------------------------------------------------- |
| `default`       | todos                             | `surface-1` + borda conforme `variant`.                                                       |
| `hover`         | `interactive={true}`              | Borda muda para `border-strong` (independente do `variant`). Sem sombra, sem scale/transform. |
| `focus-visible` | `interactive={true}`              | Anel de foco visível (`outline`), nunca suprimido.                                            |
| `disabled`      | `interactive={true}` + `disabled` | `fill-disabled` como fundo, `text-muted` no conteúdo, `cursor: not-allowed`, sem hover/focus. |

Card não-interativo não tem estado de `hover`/`focus` — ele é um container passivo, e adicionar feedback de interação a um elemento não-clicável quebraria a expectativa do usuário.

---

## Tokens usados

| Token            | Uso                                          |
| ---------------- | -------------------------------------------- |
| `surface-1`      | fundo padrão do card                         |
| `fill-disabled`  | fundo quando `interactive` + `disabled`      |
| `border-subtle`  | borda no `variant='default'`                 |
| `border-strong`  | borda no `variant='outlined'` e no `hover`   |
| `border-width`   | espessura da borda (0.5px, fixo do sistema)  |
| `radius-md`      | radius fixo do componente                    |
| `text-primary`   | título (`Card.Header` title)                 |
| `text-secondary` | descrição (`Card.Header` description), corpo |
| `text-muted`     | conteúdo quando `disabled`                   |
| `space-4/5/6`    | padding interno conforme `size`              |
| `space-2`        | gap entre Header/Body/Footer                 |

Card **não** usa `shadow-overlay` em nenhum estado — reservado a dropdown/popover/modal (pilar 3 de `principles.md`).

---

## Acessibilidade

- Card não-interativo: `as='div'` (default) ou `as='article'` quando representa um item semanticamente independente (ex: item de feed, post). Nunca usar `role="button"` num `div` — se precisa de interação, use `as='button'`/`as='a'`.
- Card interativo (`interactive={true}`): obrigatoriamente `as='button'` (ação) ou `as='a'` (navegação) — nunca `onClick` num `div` puro. Isso garante foco por teclado e leitura correta por screen reader sem trabalho extra de ARIA.
- `focus-visible` nunca é suprimido (`outline: none` sem substituto é proibido, conforme princípio geral de acessibilidade do sistema).
- Contraste mínimo AA entre `text-primary`/`text-secondary` e `surface-1` é responsabilidade do tema (tokens semânticos) — o componente não faz verificação própria, mas não deve ser usado com combinações de tema que quebrem AA.
- Quando `disabled`, o elemento interativo deve receber o atributo nativo `disabled` (se `as='button'`) ou `aria-disabled="true"` + remoção de `tabIndex` (se `as='a'`, que não tem `disabled` nativo).

---

## Exemplo de código

```jsx
import { Card } from './Card';

// Card estático, uso geral em grid
<Card>
  <Card.Header
    title="Relatório mensal"
    description="Gerado automaticamente em 1º de cada mês"
  />
  <Card.Body>
    <p>Conteúdo do card...</p>
  </Card.Body>
  <Card.Footer>
    <Button size="sm">Ver detalhes</Button>
  </Card.Footer>
</Card>

// Card interativo, selecionável, com destaque via variant
<Card as="button" interactive variant="outlined" onClick={handleSelect}>
  <Card.Header title="Plano Pro" description="Para times em crescimento" />
  <Card.Body>
    <p>R$ 49/mês</p>
  </Card.Body>
</Card>

// Card desabilitado
<Card as="button" interactive disabled>
  <Card.Body>Indisponível no seu plano atual</Card.Body>
</Card>
```

---

## Estabilidade entre versões

Segue as regras gerais de `principles.md`. Específico deste componente:

1. `size` e `variant` são o vocabulário público — não renomear valores (`sm`/`md`/`lg`, `default`/`outlined`) nem a prop em si.
2. Novas variantes de `variant` (ex: futura `'ghost'`) podem ser adicionadas livremente, desde que não introduzam sombra decorativa nem quebrem o pilar de "no máximo uma ênfase visível por vez" quando usadas em conjunto (ex: `outlined` + `bg-accent` simultâneos devem ser evitados na documentação de uso, não bloqueados pelo componente).
3. `interactive` não pode virar `true` por default em versão futura — isso mudaria semântica de elemento (`div` → `button`/`a`) de forma que quebra todo consumidor existente sem migração explícita.
4. Se um novo componente de formulário precisar do mesmo padrão de composição `Header/Body/Footer`, isso deve ser promovido a um padrão de composição documentado à parte, não duplicado ad-hoc.
