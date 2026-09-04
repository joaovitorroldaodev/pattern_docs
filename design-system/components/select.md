# Select

Campo de escolha entre opções predefinidas. Usa o mesmo esqueleto visual do `Input` (mesma altura, borda, radius), trocando o conteúdo por um valor selecionado + indicador de dropdown.

---

## Anatomia

```
[ ícone opcional | Valor selecionado / placeholder | chevron ]
   └─ (aberto) ─▶ [ lista de opções flutuante ]
        [ ✓ ] Opção selecionada
        [   ] Opção
        [   ] Opção (disabled)
```

- **Trigger** — mesmo container visual do `Input` (borda, radius, altura), com um `chevron` fixo à direita
- **Painel de opções** — lista flutuante, usa `shadow-overlay` do tokens.md, abre abaixo (ou acima, se não houver espaço)
- **Opção** — label + indicador de seleção (`✓`) quando ativa
- **Grupo** (opcional) — cabeçalho não-clicável agrupando opções relacionadas

---

## Props

| Prop          | Tipo                                                     | Default     | Descrição                                   |
| ------------- | -------------------------------------------------------- | ----------- | ------------------------------------------- |
| `size`        | `"sm" \| "md" \| "lg"`                                   | `"md"`      | Mesma escala do Input/Button                |
| `state`       | `"default" \| "error" \| "success"`                      | `"default"` | Mesmo padrão do Input                       |
| `disabled`    | `boolean`                                                | `false`     | Desabilita interação                        |
| `multiple`    | `boolean`                                                | `false`     | Permite seleção múltipla (chips no trigger) |
| `searchable`  | `boolean`                                                | `false`     | Adiciona campo de busca no topo do painel   |
| `placeholder` | `string`                                                 | —           | Texto quando nada selecionado               |
| `options`     | `{ value: string; label: string; disabled?: boolean }[]` | —           | Lista de opções                             |
| `value`       | `string \| string[]`                                     | —           | Valor(es) selecionado(s)                    |
| `onChange`    | `(value: string \| string[]) => void`                    | —           | Handler de mudança                          |

---

## Tamanhos

Idênticos ao `Input` — o trigger do Select é visualmente indistinguível de um Input até você interagir.

| Size | Altura | Padding horizontal | Font-size | Radius                          |
| ---- | ------ | ------------------ | --------- | ------------------------------- |
| `sm` | 32px   | 12px               | 13px      | `radius-sm` (6px)               |
| `md` | 40px   | 16px               | 14px      | `radius-md` (8px) — **default** |
| `lg` | 48px   | 24px               | 15px      | `radius-md` (8px)               |

O painel de opções não escala com o `size` do trigger — usa sempre `text-base` (14px) e altura de linha confortável (40px por opção), independente do tamanho do trigger, para manter a lista legível mesmo quando o trigger é `sm`.

---

## Estados

Herda o mesmo conjunto do `Input` (`default` / `error` / `success` / `disabled`), com um estado adicional:

| Estado            | Comportamento                                                                             |
| ----------------- | ----------------------------------------------------------------------------------------- |
| Default           | Igual ao Input                                                                            |
| Focus / Aberto    | Borda `bg-accent` (2px) no trigger + painel visível com `shadow-overlay`                  |
| `state="error"`   | Igual ao Input                                                                            |
| `state="success"` | Igual ao Input                                                                            |
| Disabled          | Igual ao Input, painel não abre                                                           |
| Opção `disabled`  | `color: text-muted`, `cursor: not-allowed`, não clicável, pulada na navegação por teclado |

---

## Movimento

O painel de opções é um dropdown — se enquadra no Pilar 9 (`principles.md`): movimento existe só como resposta direta a abrir/fechar, nunca como decoração.

| Ação          | Duração                 | Easing                               |
| ------------- | ----------------------- | ------------------------------------ |
| Abrir painel  | `duration-fast` (150ms) | `easing-out` — desacelera na chegada |
| Fechar painel | `duration-fast` (150ms) | `easing-in` — acelera na saída       |

Transição aplicada em `opacity` + leve `translateY` (4px) no painel — não anima o trigger nem o restante da página. Não usa `overlay-scrim` (esse token é reservado para overlays que bloqueiam a tela inteira, como Modal; o painel do Select não bloqueia interação com o resto da UI).

Respeita `prefers-reduced-motion: reduce` — duração reduzida a praticamente zero, o painel aparece/some sem transição.

---

## Tokens usados

Herda o mesmo trigger visual do `Input`, então compartilha os tokens dele:

| Token           | Uso                                                                 |
| --------------- | ------------------------------------------------------------------- |
| `border-strong` | borda do trigger no estado `default`                                |
| `border-subtle` | borda do trigger quando `disabled`                                  |
| `surface-1`     | fundo do trigger e do painel de opções                              |
| `bg-accent`     | borda de foco do trigger (2px) quando aberto                        |
| `bg-danger`     | borda do trigger em `state="error"`                                 |
| `bg-success`    | borda do trigger em `state="success"`                               |
| `fill-disabled` | fundo do trigger quando `disabled`                                  |
| `text-muted`    | texto do trigger quando `disabled`, texto de opção `disabled`       |
| `text-primary`  | valor selecionado exibido no trigger                                |
| `size-sm/md/lg` | altura, padding horizontal e font-size do trigger por `size`        |
| `radius-sm/md`  | radius do trigger: `sm` usa `radius-sm`, `md`/`lg` usam `radius-md` |
| `border-width`  | espessura da borda do trigger (0.5px, fixo do sistema)              |
| `space-2/3`     | gap interno e padding do trigger em `size="sm"`                     |

Exclusivos do painel de opções (dropdown):

| Token            | Uso                                                             |
| ---------------- | --------------------------------------------------------------- |
| `shadow-overlay` | elevação do painel flutuante sobre o conteúdo                   |
| `radius-lg`      | cantos do painel — ligeiramente maior que o `radius` do trigger |
| `duration-fast`  | duração da transição de abrir/fechar o painel                   |
| `easing-out`     | curva de entrada do painel (desacelera)                         |
| `easing-in`      | curva de saída do painel (acelera)                              |

---

## Acessibilidade

- Implementar com padrão `combobox` (`role="combobox"`, `aria-expanded`, `aria-controls` apontando pro `listbox`).
- Painel de opções usa `role="listbox"`, cada opção `role="option"` com `aria-selected`.
- Navegação por teclado obrigatória: `↑/↓` move entre opções, `Enter` seleciona, `Esc` fecha o painel, `Home/End` vão pro início/fim da lista.
- Em `searchable`, o foco permanece no campo de busca — a navegação por seta ainda deve mover a seleção na lista, sem tirar o foco do input de busca.
- `multiple`: anunciar via `aria-multiselectable="true"` no listbox.

---

## Exemplo de código (React)

```tsx
type Option = { value: string; label: string; disabled?: boolean };

type SelectProps = {
  size?: "sm" | "md" | "lg";
  state?: "default" | "error" | "success";
  disabled?: boolean;
  multiple?: boolean;
  searchable?: boolean;
  placeholder?: string;
  options: Option[];
  value?: string | string[];
  onChange?: (value: string | string[]) => void;
};

export function Select({
  size = "md",
  state = "default",
  disabled,
  multiple,
  options,
  value,
  onChange,
  placeholder,
}: SelectProps) {
  const [open, setOpen] = React.useState(false);
  const selectedLabel = Array.isArray(value)
    ? options
        .filter((o) => value.includes(o.value))
        .map((o) => o.label)
        .join(", ")
    : options.find((o) => o.value === value)?.label;

  return (
    <div
      className={`select select--${size} select--${state}`}
      data-disabled={disabled || undefined}
    >
      <button
        type="button"
        role="combobox"
        aria-expanded={open}
        aria-controls="select-listbox"
        className="select__trigger"
        disabled={disabled}
        onClick={() => setOpen((o) => !o)}
      >
        <span className="select__value">{selectedLabel || placeholder}</span>
        <ChevronIcon className="select__chevron" data-open={open} />
      </button>

      {open && (
        <ul
          id="select-listbox"
          role="listbox"
          aria-multiselectable={multiple}
          className="select__panel"
        >
          {options.map((opt) => (
            <li
              key={opt.value}
              role="option"
              aria-selected={
                Array.isArray(value)
                  ? value.includes(opt.value)
                  : value === opt.value
              }
              aria-disabled={opt.disabled}
              className="select__option"
              onClick={() => !opt.disabled && onChange?.(opt.value)}
            >
              {opt.label}
            </li>
          ))}
        </ul>
      )}
    </div>
  );
}
```

```css
.select__trigger {
  display: inline-flex;
  align-items: center;
  justify-content: space-between;
  width: 100%;
  gap: var(--space-2);
  background: var(--surface-1);
  border: var(--border-width) solid var(--border-strong);
  font-family: var(--font-family);
  color: var(--text-primary);
  cursor: pointer;
}

.select--sm .select__trigger {
  height: 32px;
  padding: 0 var(--space-3);
  font-size: var(--text-sm);
  border-radius: var(--radius-sm);
}
.select--md .select__trigger {
  height: 40px;
  padding: 0 var(--space-4);
  font-size: var(--text-base);
  border-radius: var(--radius-md);
}
.select--lg .select__trigger {
  height: 48px;
  padding: 0 var(--space-6);
  font-size: var(--text-md);
  border-radius: var(--radius-md);
}

.select--error .select__trigger {
  border-color: var(--bg-danger);
}
.select--success .select__trigger {
  border-color: var(--bg-success);
}

.select[data-disabled] .select__trigger {
  background: var(--fill-disabled);
  color: var(--text-muted);
  cursor: not-allowed;
}

.select__chevron[data-open="true"] {
  transform: rotate(180deg);
}

.select__panel {
  position: absolute;
  margin-top: var(--space-1);
  background: var(--surface-1);
  border-radius: var(--radius-lg);
  box-shadow: var(--shadow-overlay);
  list-style: none;
  padding: var(--space-1);
  max-height: 280px;
  overflow-y: auto;

  opacity: 0;
  transform: translateY(-4px);
  transition:
    opacity var(--duration-fast) var(--easing-in),
    transform var(--duration-fast) var(--easing-in);
}

.select__panel[data-open="true"] {
  opacity: 1;
  transform: translateY(0);
  transition-timing-function: var(--easing-out);
}

@media (prefers-reduced-motion: reduce) {
  .select__panel {
    transition-duration: 0.01ms;
  }
}

.select__option {
  height: 40px;
  display: flex;
  align-items: center;
  padding: 0 var(--space-3);
  border-radius: var(--radius-sm);
  font-size: var(--text-base);
  cursor: pointer;
}

.select__option[aria-selected="true"] {
  background: var(--fill-disabled);
}
.select__option[aria-disabled="true"] {
  color: var(--text-muted);
  cursor: not-allowed;
}
```

---

## Estabilidade entre versões

- Trigger deve permanecer visualmente idêntico ao `Input` em todos os tamanhos/estados — qualquer mudança de estilo no `Input` (borda, radius, altura) deve ser espelhada aqui.
- `state` com os mesmos 3 valores do `Input`/`Button` — não adicionar valores exclusivos do Select.
- Padrão `combobox`/`listbox` de acessibilidade é o contrato de interação — não trocar para outro padrão ARIA sem revisão explícita, pois quebra expectativa de navegação por teclado entre todos os componentes de seleção do sistema.
