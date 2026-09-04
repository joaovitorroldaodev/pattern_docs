# Button

Ação clicável primária da interface. Usado para disparar ações — enviar formulário, confirmar, navegar, abrir modal.

---

## Anatomia

```
[ ícone opcional ]  Label  [ ícone opcional / spinner ]
```

Um botão é composto por:

- **Container** — define fundo, borda, radius, altura, padding
- **Label** — texto, obrigatório (exceto na variante `icon-only`)
- **Ícone (leading/trailing)** — opcional, alinhado ao texto com `space-2` (8px) de gap
- **Spinner** — substitui o ícone leading quando `loading = true`

---

## Props

| Prop        | Tipo                                                          | Default     | Descrição                                                  |
| ----------- | ------------------------------------------------------------- | ----------- | ---------------------------------------------------------- |
| `variant`   | `"primary" \| "secondary" \| "ghost" \| "danger" \| "accent"` | `"primary"` | Estilo visual / hierarquia da ação                         |
| `size`      | `"sm" \| "md" \| "lg"`                                        | `"md"`      | Mapeia direto para `size-sm/md/lg` do tokens.md            |
| `disabled`  | `boolean`                                                     | `false`     | Desabilita interação e aplica estilo disabled              |
| `loading`   | `boolean`                                                     | `false`     | Mostra spinner, desabilita interação                       |
| `iconLeft`  | `ReactNode`                                                   | `undefined` | Ícone antes do label                                       |
| `iconRight` | `ReactNode`                                                   | `undefined` | Ícone depois do label                                      |
| `iconOnly`  | `boolean`                                                     | `false`     | Botão quadrado só com ícone (requer `aria-label`)          |
| `bg` / `fg` | `string` (CSS color)                                          | `undefined` | Override pontual de cor pra essa instância (ver tokens.md) |
| `onClick`   | `() => void`                                                  | —           | Handler de clique                                          |

---

## Variantes

| Variante    | Uso                                                         | Fundo          | Texto            | Borda           |
| ----------- | ----------------------------------------------------------- | -------------- | ---------------- | --------------- |
| `primary`   | Ação principal da tela — no máximo uma por contexto/seção   | `text-primary` | `surface-2`      | nenhuma         |
| `secondary` | Ação alternativa, mesmo peso visual de contexto que primary | transparente   | `text-primary`   | `border-strong` |
| `ghost`     | Ação de baixa ênfase, terciária                             | transparente   | `text-secondary` | nenhuma         |
| `danger`    | Ação destrutiva (excluir, remover, cancelar assinatura)     | `bg-danger`    | `text-danger`    | nenhuma         |
| `accent`    | Ação de destaque fora da hierarquia neutra (upsell, CTA)    | `bg-accent`    | `text-accent`    | nenhuma         |

> Regra: no máximo **um botão `primary` visível por vez** no mesmo contexto (formulário, card, seção). Se parecer que você precisa de dois, um dos dois provavelmente é `secondary` ou `ghost`.

---

## Tamanhos

Consomem diretamente `size-sm` / `size-md` / `size-lg` do `tokens.md`.

| Size | Altura | Padding horizontal | Font-size | Radius                          |
| ---- | ------ | ------------------ | --------- | ------------------------------- |
| `sm` | 32px   | 12px               | 13px      | `radius-sm` (6px)               |
| `md` | 40px   | 16px               | 14px      | `radius-md` (8px) — **default** |
| `lg` | 48px   | 24px               | 15px      | `radius-md` (8px)               |

`iconOnly` usa largura = altura (botão quadrado), mesmo radius do tamanho correspondente.

---

## Estados

| Estado           | Comportamento visual                                                                                                                  |
| ---------------- | ------------------------------------------------------------------------------------------------------------------------------------- |
| Default          | Conforme tabela de variantes                                                                                                          |
| Hover            | `opacity: 0.85` sobre o fundo/borda da variante (não trocar de cor, só opacidade)                                                     |
| Active (pressed) | `transform: scale(0.97)` — feedback tátil sutil, sem mudança de cor                                                                   |
| Focus-visible    | Anel de foco: `outline: 2px solid var(--bg-accent); outline-offset: 2px` — **nunca remover o focus ring**, só quando `:focus-visible` |
| Disabled         | `background: fill-disabled`, `color: text-muted`, `cursor: not-allowed`, sem hover/active                                             |
| Loading          | Ícone leading vira spinner, label permanece visível, `disabled` implícito (não clicável)                                              |

---

## Tokens usados

| Token                       | Uso                                                                             |
| --------------------------- | ------------------------------------------------------------------------------- |
| `text-primary`              | fundo do `primary`, texto do `secondary`/`ghost`                                |
| `surface-2`                 | texto sobre o `primary` (fundo invertido)                                       |
| `text-secondary`            | texto do `ghost`                                                                |
| `border-strong`             | borda do `secondary`                                                            |
| `bg-danger` / `text-danger` | fundo e texto do `danger`                                                       |
| `bg-accent` / `text-accent` | fundo e texto do `accent`, e o anel de foco (`bg-accent`) em todas as variantes |
| `fill-disabled`             | fundo quando `disabled`                                                         |
| `text-muted`                | texto quando `disabled`                                                         |
| `size-sm/md/lg`             | altura, padding horizontal e font-size por `size`                               |
| `radius-sm/md`              | radius: `sm` usa `radius-sm`, `md`/`lg` usam `radius-md`                        |
| `border-width`              | espessura da borda do `secondary` (0.5px, fixo do sistema)                      |
| `font-weight-medium`        | peso do label em todas as variantes                                             |
| `space-2`                   | gap entre ícone e label                                                         |

---

## Acessibilidade

- Elemento semântico `<button>` — nunca `<div onClick>`.
- `iconOnly` **exige** `aria-label` descritivo (ex: `aria-label="Configurações"`).
- Estado `loading` deve anunciar mudança via `aria-busy="true"` no container.
- Contraste mínimo AA (4.5:1) entre texto e fundo em todas as variantes — validar especialmente em overrides de `bg`/`fg` customizados.
- Focus ring nunca deve ser suprimido com `outline: none` sem substituto equivalente.
- Área de toque mínima de 40px mesmo no `size sm` em contextos touch (usar padding invisível se necessário).

---

## Exemplo de código (React)

```tsx
type ButtonProps = {
  variant?: "primary" | "secondary" | "ghost" | "danger" | "accent";
  size?: "sm" | "md" | "lg";
  disabled?: boolean;
  loading?: boolean;
  iconLeft?: React.ReactNode;
  iconRight?: React.ReactNode;
  iconOnly?: boolean;
  bg?: string;
  fg?: string;
  children?: React.ReactNode;
  onClick?: () => void;
  "aria-label"?: string;
};

export function Button({
  variant = "primary",
  size = "md",
  disabled,
  loading,
  iconLeft,
  iconRight,
  iconOnly,
  bg,
  fg,
  children,
  onClick,
  ...rest
}: ButtonProps) {
  return (
    <button
      className={`btn btn--${variant} btn--${size} ${iconOnly ? "btn--icon-only" : ""}`}
      style={{ "--bg": bg, "--fg": fg } as React.CSSProperties}
      disabled={disabled || loading}
      aria-busy={loading || undefined}
      onClick={onClick}
      {...rest}
    >
      {loading ? <Spinner /> : iconLeft}
      {!iconOnly && children}
      {!loading && iconRight}
    </button>
  );
}
```

```css
.btn {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: var(--space-2);
  font-family: var(--font-family);
  font-weight: var(--font-weight-medium);
  border: none;
  cursor: pointer;
  transition:
    opacity 0.12s ease,
    transform 0.08s ease;
}

.btn:hover:not(:disabled) {
  opacity: 0.85;
}
.btn:active:not(:disabled) {
  transform: scale(0.97);
}
.btn:focus-visible {
  outline: 2px solid var(--bg-accent);
  outline-offset: 2px;
}
.btn:disabled {
  background: var(--fill-disabled);
  color: var(--text-muted);
  cursor: not-allowed;
}

.btn--primary {
  background: var(--bg, var(--text-primary));
  color: var(--fg, var(--surface-2));
}
.btn--secondary {
  background: transparent;
  color: var(--fg, var(--text-primary));
  border: var(--border-width) solid var(--border-strong);
}
.btn--ghost {
  background: transparent;
  color: var(--fg, var(--text-secondary));
}
.btn--danger {
  background: var(--bg, var(--bg-danger));
  color: var(--fg, var(--text-danger));
}
.btn--accent {
  background: var(--bg, var(--bg-accent));
  color: var(--fg, var(--text-accent));
}

.btn--sm {
  height: 32px;
  padding: 0 var(--space-3);
  font-size: var(--text-sm);
  border-radius: var(--radius-sm);
}
.btn--md {
  height: 40px;
  padding: 0 var(--space-4);
  font-size: var(--text-base);
  border-radius: var(--radius-md);
}
.btn--lg {
  height: 48px;
  padding: 0 var(--space-6);
  font-size: var(--text-md);
  border-radius: var(--radius-md);
}

.btn--icon-only {
  padding: 0;
  aspect-ratio: 1 / 1;
}
```

---

## Estabilidade entre versões

- Nomes de `variant` e `size` são o contrato público — não renomear (ex: `secondary` → `outline`) sem depreciar antes.
- Novos variantes podem ser adicionados livremente; nenhuma variante existente deve ser removida sem depreciação.
- Mudança de valores visuais (altura, padding, radius) é aceitável e deve refletir o `tokens.md` — o componente não deve hardcodar nenhum desses valores fora dos tokens.
