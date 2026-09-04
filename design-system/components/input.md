# Input

Campo de entrada de texto de uma linha. Base para formulários — usado sozinho ou dentro de um `Field` (label + input + mensagem de erro/ajuda).

---

## Anatomia

```
[ Label ]
[ ícone opcional | Texto digitado / placeholder | ícone opcional / clear ]
[ Mensagem de ajuda ou erro ]
```

- **Container** — borda, radius, altura, fundo
- **Ícone leading** — opcional (ex: lupa em campo de busca)
- **Texto/placeholder** — a própria `<input>`
- **Ícone trailing / botão clear** — opcional (ex: "x" para limpar, olho para senha)
- **Label** — acima do campo, não é parte do componente `Input` em si (fica no wrapper `Field`, mas documentado aqui pra referência de espaçamento)
- **Mensagem de ajuda/erro** — abaixo do campo, também parte do `Field`

---

## Props

| Prop          | Tipo                                | Default     | Descrição                                   |
| ------------- | ----------------------------------- | ----------- | ------------------------------------------- |
| `size`        | `"sm" \| "md" \| "lg"`              | `"md"`      | Mapeia para `size-sm/md/lg` do tokens.md    |
| `state`       | `"default" \| "error" \| "success"` | `"default"` | Estado de validação                         |
| `disabled`    | `boolean`                           | `false`     | Desabilita interação                        |
| `readOnly`    | `boolean`                           | `false`     | Não editável, mas com valor selecionável    |
| `iconLeft`    | `ReactNode`                         | `undefined` | Ícone antes do texto                        |
| `iconRight`   | `ReactNode`                         | `undefined` | Ícone depois do texto (ex: toggle de senha) |
| `clearable`   | `boolean`                           | `false`     | Mostra botão "x" quando há valor            |
| `placeholder` | `string`                            | —           | Texto de placeholder                        |
| `value`       | `string`                            | —           | Valor controlado                            |
| `onChange`    | `(value: string) => void`           | —           | Handler de mudança                          |

---

## Tamanhos

Mesma escala do `Button`, garantindo alinhamento visual quando usados lado a lado (ex: input + botão de submit).

| Size | Altura | Padding horizontal | Font-size | Radius                          |
| ---- | ------ | ------------------ | --------- | ------------------------------- |
| `sm` | 32px   | 12px               | 13px      | `radius-sm` (6px)               |
| `md` | 40px   | 16px               | 14px      | `radius-md` (8px) — **default** |
| `lg` | 48px   | 24px               | 15px      | `radius-md` (8px)               |

---

## Estados visuais

| Estado             | Borda                                        | Fundo           | Observação                                        |
| ------------------ | -------------------------------------------- | --------------- | ------------------------------------------------- |
| Default            | `border-strong`                              | `surface-1`     | —                                                 |
| Focus              | `bg-accent` (2px, substitui a borda default) | `surface-1`     | Anel de foco visível — mesmo tratamento do Button |
| Hover (não focado) | `border-strong`, `opacity: 0.9` no container | `surface-1`     | Sutil, só sinaliza interatividade                 |
| `state="error"`    | `bg-danger`                                  | `surface-1`     | Mensagem de ajuda abaixo troca para `text-danger` |
| `state="success"`  | `bg-success`                                 | `surface-1`     | —                                                 |
| Disabled           | `border-subtle`                              | `fill-disabled` | `color: text-muted`, `cursor: not-allowed`        |
| Read-only          | `border-subtle`                              | `surface-1`     | Sem anel de foco ao clicar                        |

> Diferente do Button, o `hover` do Input **não** usa opacidade no texto/borda inteira — só um leve reforço de borda, porque aqui hover não indica uma ação imediata, só "campo interativo".

---

## Tokens usados

| Token                       | Uso                                                       |
| --------------------------- | --------------------------------------------------------- |
| `border-strong`             | borda no estado `default`                                 |
| `border-subtle`             | borda quando `disabled`/`readOnly`                        |
| `surface-1`                 | fundo do campo em todos os estados exceto `disabled`      |
| `bg-accent`                 | borda de foco (2px)                                       |
| `bg-danger` / `text-danger` | borda em `state="error"`, cor da mensagem de erro (Field) |
| `bg-success`                | borda em `state="success"`                                |
| `fill-disabled`             | fundo quando `disabled`                                   |
| `text-muted`                | texto/placeholder quando `disabled`                       |
| `text-primary`              | cor do texto digitado                                     |
| `size-sm/md/lg`             | altura, padding horizontal e font-size por `size`         |
| `radius-sm/md`              | radius: `sm` usa `radius-sm`, `md`/`lg` usam `radius-md`  |
| `border-width`              | espessura da borda padrão (0.5px, fixo do sistema)        |
| `space-2`                   | gap entre ícone e texto                                   |
| `space-3`                   | padding horizontal do `size="sm"`                         |

---

## Acessibilidade

- Sempre associar `<label>` via `htmlFor`/`id` — nunca depender só de `placeholder` como label.
- `state="error"` deve setar `aria-invalid="true"` e associar a mensagem de erro via `aria-describedby`.
- Placeholder não substitui label — contraste de placeholder pode ficar abaixo do mínimo AA, então nunca é a única fonte de instrução do campo.
- Ícone de "clear" e toggle de senha precisam de `aria-label` (ex: `aria-label="Limpar campo"`).
- Campo `disabled` deve ser pulado na ordem de tab automaticamente (comportamento nativo do `<input disabled>` — não precisa de `tabIndex` manual).

---

## Exemplo de código (React)

```tsx
type InputProps = {
  size?: "sm" | "md" | "lg";
  state?: "default" | "error" | "success";
  disabled?: boolean;
  readOnly?: boolean;
  iconLeft?: React.ReactNode;
  iconRight?: React.ReactNode;
  clearable?: boolean;
  placeholder?: string;
  value?: string;
  onChange?: (value: string) => void;
  id?: string;
  "aria-describedby"?: string;
};

export function Input({
  size = "md",
  state = "default",
  disabled,
  readOnly,
  iconLeft,
  iconRight,
  clearable,
  value,
  onChange,
  ...rest
}: InputProps) {
  return (
    <div
      className={`input input--${size} input--${state}`}
      data-disabled={disabled || undefined}
    >
      {iconLeft && (
        <span className="input__icon input__icon--left">{iconLeft}</span>
      )}
      <input
        className="input__field"
        value={value}
        disabled={disabled}
        readOnly={readOnly}
        aria-invalid={state === "error" || undefined}
        onChange={(e) => onChange?.(e.target.value)}
        {...rest}
      />
      {clearable && value && (
        <button
          type="button"
          className="input__clear"
          aria-label="Limpar campo"
          onClick={() => onChange?.("")}
        >
          ×
        </button>
      )}
      {iconRight && !clearable && (
        <span className="input__icon input__icon--right">{iconRight}</span>
      )}
    </div>
  );
}
```

```css
.input {
  display: inline-flex;
  align-items: center;
  gap: var(--space-2);
  background: var(--surface-1);
  border: var(--border-width) solid var(--border-strong);
  transition: border-color 0.12s ease;
}

.input:focus-within {
  border: 2px solid var(--bg-accent);
}

.input--error {
  border-color: var(--bg-danger);
}
.input--success {
  border-color: var(--bg-success);
}

.input[data-disabled] {
  background: var(--fill-disabled);
  border-color: var(--border-subtle);
  color: var(--text-muted);
  cursor: not-allowed;
}

.input__field {
  flex: 1;
  border: none;
  outline: none;
  background: transparent;
  font-family: var(--font-family);
  color: var(--text-primary);
}

.input--sm {
  height: 32px;
  padding: 0 var(--space-3);
  font-size: var(--text-sm);
  border-radius: var(--radius-sm);
}
.input--md {
  height: 40px;
  padding: 0 var(--space-4);
  font-size: var(--text-base);
  border-radius: var(--radius-md);
}
.input--lg {
  height: 48px;
  padding: 0 var(--space-6);
  font-size: var(--text-md);
  border-radius: var(--radius-md);
}

.input__clear {
  background: none;
  border: none;
  cursor: pointer;
  color: var(--text-muted);
}
```

---

## Estabilidade entre versões

- `state` usa os mesmos três valores (`default`/`error`/`success`) que devem se repetir em qualquer componente de formulário futuro (`Select`, `Textarea`) para manter API consistente entre eles.
- `size` segue exatamente a mesma escala do `Button` — não introduzir tamanhos exclusivos do Input sem justificativa forte, para manter alinhamento visual entre os dois.
