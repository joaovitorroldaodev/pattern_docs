# badge-modern — Documentação de Construção

Terceiro componente da variante modernista, na mesma linha de `TABLE-MODERN.md`: registra como as
regras de `style-guide.md` (escritas para um botão) foram traduzidas para a anatomia de um badge,
e onde essa tradução exigiu uma decisão nova.

---

## 1. Mapeamento variante-a-variante

| Regra do botão | Tradução em `badge-modern` |
|---|---|
| `solid` = fundo cheio | Igual, sem adaptação — um badge `solid` é estruturalmente idêntico a um botão `solid` pequeno |
| `outline` = borda fina sempre visível | Igual — borda de 1.5px, sem fundo |
| `ghost` = sem fundo em nenhum estado | Igual — só texto na cor de acento; funciona como etiqueta mínima |
| `link` = sem sublinhado | **Tradução semântica, não visual**: `link` é visualmente idêntico a `ghost`; a diferença é que `link` exige `onClick` e é renderizado como elemento acionável. Ver §2 |
| Cantos por tamanho (10–16px) | Mesma escala absoluta, sem virar um caso especial de "sempre pill" — ver §3 |
| Hover = `filter: brightness()` | Restrito a badges **interativos** — ver §4, é a decisão mais consequente deste componente |
| Pressed = `scale(.97)` | Reaproveitado sem alteração — um badge tem porte próximo ao de um botão pequeno, diferente da linha de tabela que motivou o desvio em `table-modern` |

---

## 2. `link` como diferença semântica, não visual

Nos componentes anteriores, cada variante do botão sempre correspondia a uma diferença *visual*
mapeável (mesmo quando a tradução exigia reinterpretação, como o "sem linhas divisórias" da tabela).
Badge é o primeiro caso em que isso não fecha: um badge `ghost` e um badge `link` são visualmente
idênticos — a única diferença é que `link` faz sentido *apenas* quando o badge é acionável (ex.:
chip de filtro clicável), e nesse caso deve ser um `<button>` internamente, não um `<span>` decorativo.

Por isso `badge-modern.js` valida em `mergeProps`: `variant: 'link'` sem `onClick` lança erro. A
regra geral daqui pra frente: **se uma variante não tem correspondência visual clara num novo
componente, ela pode virar uma diferença semântica/estrutural — desde que seja validada em tempo de
criação, não deixada para o consumidor descobrir em produção.**

---

## 3. Raio dos cantos: por que não virou um caso "sempre pill"

Badges, por convenção de mercado, costumam ser sempre totalmente arredondados (pill), independente
de tamanho — uma regra à parte de "raio proporcional ao tamanho". Isso **não** foi adotado aqui.

Em vez disso, `badge-modern` usa exatamente a mesma escala absoluta de `table-modern`/`button-modern`
(10/12/14/16px conforme `sm`/`md`/`lg`/`xl`). Como a altura de um badge nesses tamanhos (20/24/28/32px)
já é próxima do dobro do raio correspondente, o resultado visual **já é** um pill — sem precisar de
uma regra à parte. A vantagem de não criar a exceção: se um dia o componente ganhar um tamanho maior
(`2xl`, por exemplo) que não deva mais ser um pill perfeito, o comportamento já é o correto por
construção, não por uma condicional extra a lembrar de manter.

**Regra geral**: antes de introduzir uma convenção de mercado como regra à parte, checar se a escala
já existente do sistema produz o mesmo resultado nas dimensões reais do componente. Só documentar
como exceção quando ela genuinamente não fechar.

---

## 4. Hover restrito a badges interativos — a decisão mais consequente

`style-guide.md` define hover como parte inegociável da linguagem visual (`filter: brightness()`
em todo elemento, sempre). Isso nunca precisou de exceção em botão (sempre acionável) nem em tabela
(a linha inteira é a unidade de hover, sempre presente). Badge quebra essa premissa: a esmagadora
maioria dos badges em uso real são **decorativos** — uma etiqueta de status, uma contagem, uma tag —
sem nenhuma ação associada.

Aplicar hover a um elemento não-clicável cria uma affordance falsa: o usuário aprende a esperar que
"brilha no hover" significa "clicável", e um badge decorativo que reage visualmente ensina o
contrário do que é verdade. Por isso `badge-modern` calcula `interactive = onClick || removable` e
só aplica a classe `--interactive` (hover + pressed) quando essa condição é verdadeira. Um badge
puramente informativo não reage a nada — nem visualmente, nem ao teclado (sem `tabindex`).

**Regra geral, a mais importante deste documento para componentes futuros**: a linguagem de hover/
pressed do sistema modernista descreve *como* um elemento acionável reage, não uma obrigação de todo
componente ter algum estado de hover. Antes de aplicar `filter: brightness()`/`transform: scale()`
por padrão a um novo componente, perguntar se ele é inerentemente acionável (botão, linha clicável)
ou se a acionabilidade é opcional/situacional (badge, possivelmente ícone standalone, etc.) — no
segundo caso, o hover deve ser condicional à presença real de uma ação.

---

## 5. Modo `dot` e a regra de acessibilidade que ele introduz

`dot` é uma adição sem equivalente no botão: um indicador circular puro, sem texto, tipicamente
usado para status ("online"/"ausente"/"offline") ou notificação. Como comunica algo *só* pela cor,
ele esbarra diretamente no Critério de Sucesso 1.4.1 da WCAG ("Use of Color"): informação não pode
depender só de cor para quem não a percebe (daltonismo, leitor de tela, tela em escala de cinza).

Duas decisões seguem direto disso, ambas impostas em `mergeProps`, não deixadas como responsabilidade
do consumidor:

- `dot=true` **exige** `text` — vira `aria-label`/`role="img"` no elemento, nunca é omitido.
- Não há variante "dot sem `text`" possível pela API — a validação lança erro antes de renderizar.

`pulse` (anel animado) segue a mesma convenção de "loop autônomo com curva própria" registrada em
`TABLE-MODERN.md` §7.2 (`ease-out` no `@keyframes badgem-pulse-ring`, não a `$badgem-ease` de
interação), e some por completo em `prefers-reduced-motion: reduce` — mas o anel, ao sumir, é
substituído por uma opacidade estática levemente mais alta no próprio dot, para não perder de vez o
único sinal de "isto está ativo" para quem desativou animações.

---

## 6. Estrutura HTML: por que nunca há `<button>` dentro de `<button>`

Um badge pode ser simultaneamente clicável (`onClick`) e removível (`removable`) — ver o exemplo
"Filtrar por urgente" no demo. HTML não permite `<button>` aninhado. A resolução:

```
<span class="badge-modern">          ← wrapper, sempre <span>, nunca <button>
  [ícone opcional]
  <button class="__label-btn">texto</button>   ← só existe se onClick
  [ícone opcional]
  <button class="__remove">×</button>          ← só existe se removable
</span>
```

Os dois `<button>` são irmãos dentro do wrapper, nunca um dentro do outro. `evt.stopPropagation()`
no clique do botão de remover evita que remover dispare também o `onClick` do badge (que, sendo um
elemento diferente, tecnicamente não borbulharia para o `__label-btn`, mas o `stopPropagation` é
mantido por precaução caso um consumidor futuro coloque um listener no wrapper em vez de usar `onClick`
via prop).

---

## 7. Checklist adicional — componente com identidade puramente decorativa

Complementa a checklist de `TABLE-MODERN.md` §9 para o caso de um componente que, ao contrário de
botão/tabela, é decorativo por padrão e só ocasionalmente acionável:

- [ ] Hover/pressed foram condicionados a uma verificação real de "isto é acionável", não aplicados
      incondicionalmente (§4)?
- [ ] Toda variante do conjunto padrão (`solid`/`outline`/`ghost`/`link`) tem uma correspondência —
      visual OU semântica — justificada por escrito, não só "porque o botão tem essas quatro"?
- [ ] Se alguma informação for comunicada só por cor (status, categoria), existe um nome acessível
      (`aria-label`, texto visível, ou ambos) documentado como obrigatório na validação de props, não
      como recomendação no comentário (§5)?
- [ ] Antes de adotar uma convenção de mercado do tipo de componente (badges = sempre pill, spinners
      = sempre giram, etc.) como regra à parte, foi checado se a escala já existente do sistema produz
      o mesmo resultado nas dimensões reais do componente (§3)?
- [ ] Se o componente pode acumular mais de um controle interativo simultâneo (ex.: acionável +
      removível), a estrutura HTML foi desenhada para nunca aninhar elementos interativos incompatíveis
      (`<button>` em `<button>`, `<a>` em `<a>`)?