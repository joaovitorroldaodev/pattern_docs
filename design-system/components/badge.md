# Badge

Rótulo compacto e não-interativo (na maioria dos casos) para indicar status, categoria ou contagem. Usado dentro de listas, tabelas, cards e headers — nunca como ação principal da tela.

---

## Anatomia

```
[ dot opcional ] Label [ botão remover opcional ]
```

- **Container** — fundo, radius, padding compacto
- **Dot** (opcional) — indicador circular de status, à esquerda do label (ex: bolinha verde para "ativo")
- **Label** — texto curto, idealmente uma ou duas palavras
- **Botão remover** (opcional) — "x" à direita, só em badges removíveis (ex: tags de filtro)

---

## Props

| Prop        | Tipo                                             | Default     | Descrição                                         |
| ----------- | ------------------------------------------------ | ----------- | ------------------------------------------------- |
| `variant`   | `"neutral" \| "accent" \| "danger" \| "success"` | `"neutral"` | Estilo visual / significado semântico             |
| `size`      | `"sm" \| "md"`                                   | `"md"`      | Badge não tem `lg` — é sempre um elemento pequeno |
| `dot`       | `boolean`                                        | `false`     | Mostra indicador circular antes do label          |
| `removable` | `boolean`                                        | `false`     | Mostra botão de remover, torna o badge interativo |
| `onRemove`  | `() => void`                                     | —           | Handler do botão remover                          |

---

## Variantes

| Variante  | Uso                                         | Fundo                       | Texto                       |
| --------- | ------------------------------------------- | --------------------------- | --------------------------- |
| `neutral` | Categoria genérica, tag sem carga semântica | `border-subtle`             | `text-secondary`            |
| `accent`  | Destaque, novidade, categoria em foco       | `bg-accent`, 12% opacidade  | `bg-accent` (texto sólido)  |
| `danger`  | Erro, status crítico, vencido               | `bg-danger`, 12% opacidade  | `bg-danger` (texto sólido)  |
| `success` | Ativo, aprovado, concluído                  | `bg-success`, 12% opacidade | `bg-success` (texto sólido) |

> Diferente do Button, os badges de `accent`/`danger`/`success` usam a cor **como texto sobre um fundo suave** (mesma cor em baixa opacidade), não cor sólida com texto branco — um badge é informativo, não uma ação, então não deve competir visualmente com botões reais na mesma tela.

---

## Tamanhos

| Size | Altura | Padding horizontal | Font-size        | Radius                      |
| ---- | ------ | ------------------ | ---------------- | --------------------------- |
| `sm` | 20px   | `space-2` (8px)    | `text-xs` (12px) | `radius-full`               |
| `md` | 24px   | `space-3` (12px)   | `text-xs` (12px) | `radius-full` — **default** |

Badge sempre usa `radius-full` (pill), independente do `radius-md` que o resto do sistema usa como padrão — é a exceção intencional da escala, porque o formato de pílula reforça visualmente "isso é uma etiqueta, não um container de conteúdo".

---

## Estados

Badge é majoritariamente estático. Os únicos estados aplicáveis:

| Estado                               | Comportamento                                                                      |
| ------------------------------------ | ---------------------------------------------------------------------------------- |
| Default                              | Conforme tabela de variantes                                                       |
| `removable` + hover no botão remover | `opacity: 0.7` só no ícone "x", não no badge inteiro                               |
| `removable` + focus                  | Anel de foco (`bg-accent`, 2px) só no botão remover, badge em si nunca recebe foco |

---

## Tokens usados

| Token                                    | Uso                                                                     |
| ---------------------------------------- | ----------------------------------------------------------------------- |
| `border-subtle`                          | fundo do `variant="neutral"`                                            |
| `text-secondary`                         | texto do `variant="neutral"`                                            |
| `bg-accent` / `bg-danger` / `bg-success` | cor-base do fundo (12% de opacidade) e do texto nas variantes coloridas |
| `text-xs`                                | tamanho de fonte, único em todos os `size`                              |
| `space-2/3`                              | padding horizontal: `space-2` no `sm`, `space-3` no `md`                |
| `radius-full`                            | radius fixo (pílula), exceção intencional à escala padrão do sistema    |

---

## Acessibilidade

- Badge puramente informativo (sem `removable`) não deve ser focável nem ter `role` interativo — é texto, tratado como `<span>`.
- Quando usado só com `dot` (sem label visível) para indicar status, obrigatório incluir texto para leitor de tela via `sr-only` ou `aria-label` — cor sozinha nunca comunica o status.
- Botão remover precisa de `aria-label` (ex: `aria-label="Remover filtro: Ativo"`, incluindo o valor do badge no texto).
- Contraste do texto colorido sobre fundo em baixa opacidade deve ser validado — em fundos claros, cores muito claras (`bg-success` claro demais) podem não atingir AA; testar com o tema real do projeto, já que a paleta é maleável.

---

## Exemplo de código (React)

```tsx
type BadgeProps = {
  variant?: "neutral" | "accent" | "danger" | "success";
  size?: "sm" | "md";
  dot?: boolean;
  removable?: boolean;
  onRemove?: () => void;
  children: React.ReactNode;
};

export function Badge({
  variant = "neutral",
  size = "md",
  dot,
  removable,
  onRemove,
  children,
}: BadgeProps) {
  return (
    <span className={`badge badge--${variant} badge--${size}`}>
      {dot && <span className="badge__dot" aria-hidden="true" />}
      {children}
      {removable && (
        <button
          type="button"
          className="badge__remove"
          aria-label={`Remover: ${children}`}
          onClick={onRemove}
        >
          ×
        </button>
      )}
    </span>
  );
}
```

```css
.badge {
  display: inline-flex;
  align-items: center;
  gap: var(--space-1);
  font-family: var(--font-family);
  font-weight: var(--font-weight-medium);
  font-size: var(--text-xs);
  border-radius: var(--radius-full);
  white-space: nowrap;
}

.badge--neutral {
  background: var(--border-subtle);
  color: var(--text-secondary);
}
.badge--accent {
  background: color-mix(in srgb, var(--bg-accent) 12%, transparent);
  color: var(--bg-accent);
}
.badge--danger {
  background: color-mix(in srgb, var(--bg-danger) 12%, transparent);
  color: var(--bg-danger);
}
.badge--success {
  background: color-mix(in srgb, var(--bg-success) 12%, transparent);
  color: var(--bg-success);
}

.badge--sm {
  height: 20px;
  padding: 0 var(--space-2);
}
.badge--md {
  height: 24px;
  padding: 0 var(--space-3);
}

.badge__dot {
  width: 6px;
  height: 6px;
  border-radius: var(--radius-full);
  background: currentColor;
}

.badge__remove {
  background: none;
  border: none;
  cursor: pointer;
  color: inherit;
  padding: 0;
  line-height: 1;
}

.badge__remove:hover {
  opacity: 0.7;
}
.badge__remove:focus-visible {
  outline: 2px solid var(--bg-accent);
  outline-offset: 2px;
}
```

---

## Estabilidade entre versões

- `radius-full` é fixo para Badge — não deve seguir eventuais mudanças no `radius-md` global, é uma exceção documentada e estável por si só.
- Conjunto de `variant` (`neutral`/`accent`/`danger`/`success`) reflete diretamente os tokens `bg-*` do `tokens.md` — se um novo token semântico de cor for adicionado lá (ex: `bg-warning`), a variante correspondente deve ser adicionada aqui também, nunca criada com cor solta.
- `size` tem apenas `sm`/`md` (sem `lg`) intencionalmente — se surgir necessidade de um badge maior, provavelmente o componente certo é outro (ex: `Tag` ou `Chip` interativo), não uma extensão do Badge.
