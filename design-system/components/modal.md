# Modal

Diálogo modal sobre um overlay que bloqueia interação com o resto da tela — usado para confirmações, formulários curtos e fluxos que exigem decisão antes de continuar.

> `overlay-scrim` entra em `tokens.md`, seção "Semânticos — superfície e texto", default `rgba(13, 13, 13, 0.5)` (derivado de `neutral-950` a 50% de opacidade). É uma adição, não quebra o contrato de estabilidade (regra 3 de `tokens.md`: novos tokens podem ser adicionados livremente). Reservado para outros componentes de overlay futuros (ex: Drawer).

---

## Anatomia

```
Modal
├── Overlay              (scrim — cobre a viewport, fecha ao clicar se closeOnOverlayClick)
└── Modal.Container       (role="dialog", radius-lg, shadow-overlay)
    ├── Modal.Header       (título + botão de fechar)
    ├── Modal.Body         (conteúdo livre)
    └── Modal.Footer       (ações — geralmente Button secondary + primary)
```

Diferente do Card, o Modal **não é composto livremente na árvore** — ele é montado via portal na raiz do `document.body`, para não herdar `overflow`/`z-index`/`transform` de containers pais.

---

## Props

| Prop                  | Tipo                   | Default | Descrição                                                                                                                                    |
| --------------------- | ---------------------- | ------- | -------------------------------------------------------------------------------------------------------------------------------------------- |
| `open`                | `boolean`              | —       | **Obrigatório.** Visibilidade controlada externamente — Modal não guarda estado próprio de aberto/fechado.                                   |
| `onClose`             | `() => void`           | —       | **Obrigatório.** Chamado ao clicar no overlay, pressionar Esc, ou clicar no botão de fechar.                                                 |
| `size`                | `'sm' \| 'md' \| 'lg'` | `'md'`  | Largura máxima do container. Padding interno **não varia por size** (diferente do Card) — ver Tokens usados.                                 |
| `closeOnOverlayClick` | `boolean`              | `true`  | Se `false`, só fecha via botão de fechar ou `onClose` programático — usar para fluxos onde perder o progresso é caro (ex: formulário longo). |
| `closeOnEsc`          | `boolean`              | `true`  | Se `false`, tecla Esc não fecha o modal.                                                                                                     |
| `initialFocusRef`     | `RefObject`            | —       | Elemento a receber foco na abertura. Se omitido, foca o primeiro elemento focável dentro do `Modal.Container`.                               |
| `children`            | `ReactNode`            | —       | Tipicamente `Modal.Header`, `Modal.Body`, `Modal.Footer`.                                                                                    |

Não existem props `bg`/`fg` de override pontual aqui — diferente do Card, o Modal não é um componente decorativo reposicionável em temas distintos dentro da mesma tela; ele sempre usa `surface-1` para manter previsibilidade em qualquer contexto de uso.

Não existe prop `variant`: um único tratamento visual de modal evita a ambiguidade de "qual modal é mais importante" — se duas variantes de ênfase fossem necessárias, isso seria sinal de que uma delas deveria ser um Drawer ou Popover, não uma variante do Modal.

---

## Variantes / Tamanhos

| `size` | `max-width`       |
| ------ | ----------------- |
| `sm`   | `400px`           |
| `md`   | `560px` — default |
| `lg`   | `760px`           |

Altura sempre automática (`max-height: 90vh` com scroll interno no `Modal.Body`) — Modal nunca deve cortar conteúdo sem indicar overflow.

---

## Estados

| Estado                   | Comportamento                                                                                                                                                                                                                                |
| ------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `open=true` → `entered`  | Overlay + Container renderizados via portal, `aria-hidden` aplicado ao restante da árvore, animação de entrada dispara no mount.                                                                                                             |
| `open=false` → `exiting` | Modal permanece montado durante `duration-base` para tocar a animação de saída — só então é desmontado. Continua sem `display: none`: o elemento existe no DOM só pelo tempo da transição, nunca fica invisível-mas-focável indefinidamente. |
| `exiting` → `closed`     | Nada renderizado no DOM (unmount real).                                                                                                                                                                                                      |
| `focus-visible`          | No botão de fechar e em qualquer elemento focável dentro do Container — nunca suprimido.                                                                                                                                                     |

Modal não adota o vocabulário `state` (`default`/`error`/`success`) do princípio 7 — esse vocabulário é específico de componentes de entrada de dados (Input, Select); Modal é um componente estrutural, não um campo de formulário.

---

## Movimento

> Movimento passa a ser categoria própria em `tokens.md`, e a seção "Movimento" abaixo passa a ser opcional no template fixo (entre Estados e Tokens usados), reservada a componentes de overlay.

Transição dispara em resposta direta à ação do usuário (abrir/fechar), não é decoração ambiente — por isso é apropriada mesmo num sistema que evita movimento gratuito.

| Elemento  | Entrada                                                                                      | Saída                                       |
| --------- | -------------------------------------------------------------------------------------------- | ------------------------------------------- |
| Overlay   | `opacity 0→1`, `duration-fast` (150ms), `easing-out`                                         | `opacity 1→0`, `duration-fast`, `easing-in` |
| Container | `opacity 0→1` + `translateY(8px)→0` + `scale(0.98)→1`, `duration-base` (200ms), `easing-out` | inverso, `duration-base`, `easing-in`       |

Regras:

- Overlay e Container animam com curvas diferentes por convenção de movimento: **entrada desacelera** (`easing-out` — chega "suave"), **saída acelera** (`easing-in` — sai "rápido"), o padrão consagrado para diálogos.
- Deslocamento vertical limitado a 8px — o suficiente para dar direção ao movimento sem virar "slide-up" chamativo.
- `@media (prefers-reduced-motion: reduce)`: duração cai para praticamente zero. Não é opcional — decorre do pilar 8 (acessibilidade) de `principles.md`.
- Nenhuma animação de `hover` ou ambiente contínuo — só transição de entrada/saída, disparada por interação real.

---

## Tokens usados

| Token            | Uso                                                                                              |
| ---------------- | ------------------------------------------------------------------------------------------------ |
| `overlay-scrim`  | fundo do overlay (confirmado — ver nota no topo do documento)                                    |
| `surface-1`      | fundo do `Modal.Container`                                                                       |
| `shadow-overlay` | elevação do Container sobre o scrim — uso legítimo do token, reservado exatamente para este caso |
| `radius-lg`      | radius do Container — modal é "container maior" por definição em `tokens.md`                     |
| `border-subtle`  | linha divisória entre Header/Body e Body/Footer (não a borda externa do Container)               |
| `text-primary`   | título do `Modal.Header`                                                                         |
| `text-secondary` | corpo do `Modal.Body`                                                                            |
| `space-6`        | padding interno do Container, fixo em todos os `size`                                            |
| `space-4`        | gap entre Header/Body/Footer                                                                     |
| `duration-fast`  | transição do Overlay (confirmado — categoria "Movimento" em `tokens.md`)                         |
| `duration-base`  | transição do Container (confirmado)                                                              |
| `easing-out`     | curva de entrada — desacelera (confirmado)                                                       |
| `easing-in`      | curva de saída — acelera (confirmado)                                                            |

Container **não** tem `border` externa — a combinação `shadow-overlay` + contraste contra o `overlay-scrim` já é suficiente para separar o modal do restante da tela; adicionar borda seria redundante com o princípio de conter decoração ao mínimo necessário.

---

## Acessibilidade

- `Modal.Container` recebe `role="dialog"` e `aria-modal="true"`.
- `aria-labelledby` aponta para o `id` gerado internamente pelo título em `Modal.Header`. Se não houver `Modal.Header`, o consumidor deve passar `aria-label` manualmente — Modal sem nome acessível é um erro de uso, não algo que o componente deve mascarar silenciosamente.
- **Focus trap**: enquanto `open=true`, Tab/Shift+Tab ciclam somente entre os elementos focáveis dentro do Container. Foco nunca escapa para o restante da página.
- **Foco inicial**: ao abrir, foco vai para `initialFocusRef` ou o primeiro elemento focável do Container.
- **Foco de retorno**: ao fechar, foco retorna ao elemento que tinha foco antes da abertura (tipicamente o botão/trigger que abriu o modal).
- Esc fecha o modal quando `closeOnEsc` (default `true`) — isso não é opcional de acessibilidade, é convenção de teclado esperada para diálogos.
- O restante da árvore (`document.body` fora do portal) recebe `aria-hidden="true"` enquanto o modal está aberto, para leitores de tela não navegarem para conteúdo inacessível por trás do overlay.

---

## Exemplo de código

```jsx
import { useRef, useState } from "react";
import { Modal } from "./Modal";
import { Button } from "./Button";

function DeleteConfirmation() {
  const [open, setOpen] = useState(false);
  const triggerRef = useRef(null);

  return (
    <>
      <Button ref={triggerRef} variant="danger" onClick={() => setOpen(true)}>
        Excluir conta
      </Button>

      <Modal size="sm" open={open} onClose={() => setOpen(false)}>
        <Modal.Header title="Excluir conta" />
        <Modal.Body>
          <p>Essa ação não pode ser desfeita. Todos os dados serão perdidos.</p>
        </Modal.Body>
        <Modal.Footer>
          <Button variant="secondary" onClick={() => setOpen(false)}>
            Cancelar
          </Button>
          <Button variant="danger" onClick={handleDelete}>
            Excluir
          </Button>
        </Modal.Footer>
      </Modal>
    </>
  );
}
```

---

## Estabilidade entre versões

Segue as regras gerais de `principles.md`. Específico deste componente:

1. `size` é o único vocabulário público de variação visual — não renomear `sm`/`md`/`lg`, e novo valor (ex: `full` para modal fullscreen mobile) só deve ser adicionado se justificado por um caso de uso real, não especulativamente.
2. `open`/`onClose` são o contrato de controle — Modal nunca deve ganhar estado interno de visibilidade em versão futura; isso quebraria todo consumidor que depende do padrão controlado.
3. A ausência de `variant` é uma decisão deliberada (ver Variantes/Tamanhos) — antes de adicionar uma, revisar se o caso não é na verdade um componente novo (Drawer, AlertDialog).
4. `overlay-scrim` — entra no changelog do design system como um todo, não só na doc do Modal: é um token compartilhável por outros overlays.
5. `duration-fast`/`duration-base`/`easing-out`/`easing-in` — entram em `tokens.md` como categoria própria ("Movimento"), compartilhável por qualquer overlay futuro (Popover, Dropdown, Toast), não exclusivos do Modal.
6. Seção "Movimento" do template — passa a existir como seção opcional em `principles.md`, entre Estados e Tokens usados, só para componentes de overlay. Card, Button e Input não a usam.
