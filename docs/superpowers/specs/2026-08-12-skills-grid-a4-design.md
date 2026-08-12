# Design — Grade de Competências na sidebar e ajuste do currículo para 1 página A4

Data: 2026-08-12
Arquivos afetados: `style.css` (principal), `index.html` (apenas se necessário)

## Problema

### 1. Rag visual nas Competências

`.skills-grid` (style.css:244) usa `display: flex; flex-wrap: wrap`. Cada `.skill-tag` tem a
largura do próprio texto, então as 12 competências se distribuem de forma irregular: rótulos
longos como "Governança de TI e ITIL" ocupam a linha inteira sozinhos, enquanto os curtos se
agrupam. O resultado é visualmente desorganizado.

### 2. O currículo não cabe em um A4 — e o excedente é cortado em silêncio

Medições feitas com Chrome headless na largura de impressão A4 (794px), com o bloco
`@media print` aplicado:

| Bloco | Altura medida |
|---|---|
| `.skills-grid` (Competências) | 362px |
| `.section-block` Contato | 210px |
| `.section-block` Formação | 274px |
| `.section-block` Idiomas | 121px |
| `.profileText` | 194px |
| **Sidebar (soma intrínseca)** | **1232px** |
| `.timeline` | 987px |
| **Coluna direita (intrínseca)** | **1338px** |
| **Altura útil de um A4 (`margin: 0`)** | **1123px** |

A coluna direita excede o A4 em ~215px. Hoje isso não aparece porque `.container` combina
`overflow: hidden` (style.css:40) com `height: 100vh !important` (style.css:461): o conteúdo
excedente é **recortado**, não paginado. Confirmado empiricamente — ao remover essas duas
travas numa cópia de teste, o PDF gerado passa de 1 para 2 páginas. Ou seja, o botão
"Baixar PDF" atualmente descarta o final do currículo sem avisar.

As Competências, com 362px, são o maior bloco da sidebar e portanto o melhor alvo de
economia — o que faz as duas partes deste design convergirem.

## Solução

### Parte 1 — Competências em grade 2 colunas

`.skills-grid` passa de flex-wrap para:

```
display: grid;
grid-template-columns: 1fr 1fr;
gap: 6px;
```

Cada `.skill-tag` preenche 100% da célula, então as 12 competências formam uma grade 2×6 com
colunas e linhas alinhadas. O rag desaparece porque nenhuma tag tem mais largura que a célula.

Detalhes:

- `.skill-tag` ganha `text-align: center` e `overflow-wrap: break-word` (proteção contra
  overflow, embora nenhuma palavra isolada exceda a largura da célula nas medições).
- Rótulos longos quebram em 2 linhas dentro da célula. Como as células de uma mesma linha da
  grade esticam juntas (`align-items: stretch`, padrão), as linhas permanecem alinhadas — a
  quebra não reintroduz irregularidade.
- Fundo, borda, `border-radius` e o estado `:hover` atuais são preservados.
- **A lista das 12 competências foi substituída pela lista fornecida pelo usuário** (decisão de
  conteúdo dele, não consequência do trabalho de layout). O layout não altera texto por conta
  própria. Os novos rótulos são em média mais longos — "Resolução Analítica de Problemas" e
  "Foco na Experiência do Usuário" ocupam 3 linhas na célula — e o ganho medido já considera isso.

No `@media print`, `.skill-tag` recebe `font-size` e `line-height` próprios, sobrepondo a regra
genérica `span { font-size: 11px !important }` da linha 475 que hoje se aplica às tags.

Redução estimada: **362px → ~230px** (~130px economizados).

### Parte 2 — Fazer o A4 caber de verdade

A Parte 1 sozinha não basta: ela economiza na sidebar (1232px → ~1100px), mas a coluna direita
segue com 1338px. Alterações restritas ao bloco `@media print` — **a visualização em tela fica
inalterada**:

1. Trocar `height: 100vh !important` por `height: auto !important` em `.container`, para que o
   conteúdo deixe de ser recortado e o overflow real fique visível durante o ajuste.
2. Apertar espaçamentos primeiro (prioridade definida pelo usuário): `gap` e `padding` de
   `.right_Side`, `padding-bottom` de `.timeline-item`, `margin-bottom` de `.section-title` e
   `padding` dos `.section-block`.
3. Só se os espaçamentos não bastarem, reduzir `line-height` do corpo de texto de 1.5 para ~1.4.
4. Fonte só como último recurso, e nunca abaixo de 9.5px.

O texto do currículo não é encurtado.

## Critério de aceite

Verificado com o mesmo harness de medição usado no diagnóstico
(Chrome headless `--print-to-pdf` + contagem de páginas no PDF):

1. O PDF gerado a partir de `index.html` tem **exatamente 1 página**.
2. Com `overflow: hidden` removido numa cópia de teste, o PDF **continua com 1 página** — isto
   é, cabe de verdade, não está sendo recortado.
3. A última experiência (Stefanini Group / Grupo FIAT, 2013) aparece completa no PDF.
4. Nenhum corpo de texto fica abaixo de 9.5px.
5. As Competências formam uma grade 2×6 alinhada, com as 12 competências e seus textos originais.
6. A renderização em tela (acima de 900px e no responsivo) permanece equivalente à atual, exceto
   pela grade de Competências.

## Resultado medido

| Etapa | Coluna direita | `.skills-grid` |
|---|---|---|
| Antes | 1338px | 362px |
| Grade 2 colunas | 1338px | 233px |
| + espaçamentos apertados | 1228px | 233px |
| + `line-height` 1.5 → 1.4 | 1171px | 233px |
| + fonte 11px → 10.5px | **1044px** | 233px |

Limite do A4: 1123px. Folga final: 79px. Sidebar final: 1039px.

PDFs gerados com Chrome headless: `index.html` → 1 página; cópia sem `overflow: hidden` →
1 página (prova que cabe, em vez de estar recortado). Antes da mudança, essa mesma cópia sem
recorte dava 2 páginas.

## Defeito pré-existente encontrado (fora de escopo, não corrigido)

A regra `p, li, span { font-size: ... !important }` do bloco `@media print` também atinge o
`<span>Gonçalves</span>` dentro de `.name-block h2` (index.html:30). No PDF, o sobrenome sai em
10.5px enquanto o nome sai em 1.6em. Na tela o nome aparece correto. O defeito já existia antes
desta mudança (a regra usava 11px). Corrigir custaria ~25px dos 79px de folga.

## Fora de escopo

- Alterar o texto de qualquer seção do currículo.
- Redesenhar outras seções da sidebar ou a timeline.
- Mudar a paleta, as fontes ou o layout de duas colunas.
