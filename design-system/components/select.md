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

## Tokens usados

Mesmos do `Input` (`border-strong` · `border-subtle` · `surface-1` · `bg-accent` · `bg-danger` · `bg-success` · `fill-disabled` · `text-muted` · `text-primary` · `size-sm/md/lg` · `radius-sm/md` · `border-width` · `space-2/3`), mais:

`shadow-overlay` (painel flutuante) · `radius-lg` (cantos do painel, ligeiramente maior que o trigger)

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
