# Princípios

Este documento define a filosofia por trás do design system — o "porquê" que orienta decisões futuras quando uma situação nova aparecer e não houver regra explícita ainda escrita.

---

## Objetivo

Um sistema de componentes React **refinado, moderno e estável entre versões**, pensado para ser reutilizado entre múltiplos projetos. Refinado e moderno aqui significam contenção, não decoração: menos elementos visuais competindo por atenção, hierarquia clara por contraste e espaçamento — não por efeitos.

---

## Pilares

### 1. Cor é injetada, não hardcoded

Nenhum componente define cor fixa. Todo componente consome **tokens semânticos** (`text-primary`, `bg-danger`, `border-strong`...) definidos em `tokens.md`, e aceita overrides pontuais via props (`bg`/`fg`) quando fizer sentido. Isso é o que torna o sistema **maleável entre projetos**: a paleta muda por fora, os componentes nunca precisam ser tocados.

### 2. Densidade confortável como padrão

Componentes interativos (Button, Input, Select) usam altura base de 40px (`size-md`), com escala de `sm` (32px) a `lg` (48px). A prioridade é legibilidade e área de toque confortável — não empacotar o máximo de informação possível na tela.

### 3. Bordas finas, sem sombra decorativa

Borda padrão de 0.5px (`border-width`). Profundidade visual vem de contraste e borda, não de `box-shadow`. Sombra (`shadow-overlay`) é reservada só para elementos que realmente flutuam sobre o conteúdo — dropdown, popover, modal — nunca como decoração de card ou botão.

### 4. Poucos pesos tipográficos

Apenas `font-weight-regular` (400) e `font-weight-medium` (500) em todo o sistema. Hierarquia de texto é construída com tamanho e cor (`text-primary` vs `text-secondary` vs `text-muted`), não com peso de fonte. Isso evita que a interface fique "gritando" com múltiplos graus de negrito.

### 5. Um radius por escala, aplicado com consistência

`radius-sm` (6px) para elementos pequenos, `radius-md` (8px) como padrão da maioria dos componentes, `radius-lg` (12px) para containers maiores. Nunca introduzir um valor de radius fora dessa escala.

### 6. Hierarquia de ênfase é explícita

Em qualquer conjunto de ações (botões, opções), deve haver no máximo **uma ação de maior ênfase visível por vez** (`primary` ou `accent`). Se duas ações parecem igualmente importantes na tela, isso é um sinal de que a hierarquia da tela — não do componente — precisa ser repensada.

### 7. API consistente entre componentes de formulário

`size` (`sm`/`md`/`lg`) e `state` (`default`/`error`/`success`) são o vocabulário compartilhado de todo componente de entrada de dados (Input, Select, e futuros como Textarea, Checkbox). Um desenvolvedor que aprendeu a API do Input já sabe usar o Select.

### 8. Acessibilidade não é opcional

Todo componente interativo deve: usar elemento semântico correto, manter `focus-visible` visível (nunca `outline: none` sem substituto), ter contraste mínimo AA, e funcionar via teclado sem depender de mouse. Isso está documentado por componente, não só aqui — mas é um princípio, não um adendo.

### 9. Movimento é funcional, não decorativo

Transição existe só como resposta direta a uma ação do usuário (abrir, fechar, expandir) — nunca como animação ambiente, hover chamativo ou decoração de entrada em cascata. Reservado a componentes de overlay (modal, popover, dropdown, toast); componentes estáticos não precisam de tokens de movimento além de `focus-visible`/`hover` instantâneos.

Convenção fixa, não escolhida caso a caso: entrada desacelera (`easing-out`), saída acelera (`easing-in`) — ver categoria "Movimento" em `tokens.md`. `prefers-reduced-motion: reduce` deve sempre reduzir a duração a praticamente zero; isso decorre diretamente do pilar 8 e não é opcional.

---

## Regras de estabilidade entre versões

Válidas para o sistema como um todo — cada `component.md` pode adicionar regras específicas, mas nunca contradizer estas:

1. **Nomes de tokens e de props/variantes são o contrato público.** Não renomear — depreciar e manter funcional por pelo menos uma versão major antes de remover.
2. **Valores podem evoluir, nomes não.** Ajustar o valor de `radius-md` de 8px para 10px é uma mudança de versão menor; renomear `radius-md` para `radius-default` é uma mudança que quebra contrato.
3. **Novas variantes/props podem ser adicionadas livremente.** Isso nunca é uma mudança que quebra compatibilidade.
4. **Toda mudança de valor visual em um token deve ser documentada em changelog**, mesmo quando não quebra nada — porque afeta a aparência de todos os projetos consumidores simultaneamente.
5. **Componentes de formulário compartilham vocabulário de `size`/`state`.** Se um componente novo precisar de um estado que os outros não têm, isso deve ser discutido antes de virar exceção — geralmente é sinal de que o estado deveria ser promovido pra todos.

---

## Estrutura de documentação

```
design-system/
├── principles.md        # este arquivo
├── tokens.md             # tokens semânticos e primitivos
├── components/
│   ├── button.md
│   ├── input.md
│   ├── select.md
│   ├── card.md
│   ├── modal.md
│   └── ...
```

Cada arquivo em `components/` segue o mesmo template fixo: Anatomia → Props → Variantes/Tamanhos → Estados → **Movimento** (opcional — só componentes de overlay: modal, popover, dropdown, toast) → Tokens usados → Acessibilidade → Exemplo de código → Estabilidade entre versões. O template existe para que os arquivos sirvam tanto como referência de leitura quanto como input estruturado para geração de código por IA.
