# Exemplos de Aula — Transformações 2D no Canvas

Exemplos mínimos em Canvas 2D para acompanhar os slides **transf2d-p3**.
Todos são HTML standalone — abrir direto no navegador.

## Mapa: Exemplos → Slides

| # | Arquivo | Slide |
|---|---------|-------|
| 1 | `01-translate.html` | **ctx.translate — Translação** — Desenha um retângulo, aplica translate, desenha de novo. |
| 2 | `02-rotate-scale.html` | **ctx.rotate e ctx.scale** — Rotação (radianos) e escala ao redor da origem. |
| 3 | `03-save-restore.html` | **ctx.setTransform e ctx.save/restore** — save → transforma → desenha → restore → desenha. |
| 4 | `04-reset-acumulacao.html` | **A Importância do Reset da Matriz** — Dois canvas: sem reset (acumula) vs com reset. |
| 5 | `05-escala-ponto-fixo.html` | **Ordem das Transformações** — Escala com ponto fixo: translate→scale→translate. |
| 6 | `06-rotacao-ponto-fixo.html` | **Rotação com Ponto Arbitrário + Translação** — Botões para comparar ordem correta vs errada. |
| 7 | `07-exemplo-completo.html` | **Composição com Matrizes** — Composição T · R · S numa casa, mostra equivalência com WebGL. |

## Ordem durante a aula

1. **`01-translate`** no slide "ctx.translate" — mostra o retângulo pulando de posição
2. **`02-rotate-scale`** no slide "ctx.rotate e ctx.scale" — rotação e escala ao redor da origem
3. **`03-save-restore`** no slide "ctx.save/restore" — restore desfaz as transformações
4. **`04-reset-acumulacao`** no slide "Reset da Matriz" — clicar Iniciar e ver a escala explodir no canvas esquerdo
5. **`05-escala-ponto-fixo`** no slide "De Baixo para Cima" — mostra o padrão translate→scale→translate
6. **`06-rotacao-ponto-fixo`** no slide "Rotação + Translação" — alternar entre ordem correta e errada
7. **`07-exemplo-completo`** no slide "Composição com Matrizes no WebGL" — M = T · R · S aplicado numa casa, com código WebGL equivalente no `<pre>`
