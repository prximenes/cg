# Computação Gráfica 2D — Material de Aula

Material didático interativo cobrindo os fundamentos de Computação Gráfica 2D.
Cada arquivo HTML é independente, autocontido e funcional com Live Server.

---

## Estrutura do Projeto

```
cg/
 ├── README.md                       ← este arquivo
 ├── rasterizacao/
 │     ├── dda.html                  ← Algoritmo DDA
 │     ├── bresenham_reta.html       ← Bresenham para retas
 │     └── bresenham_circulo.html    ← Bresenham para círculos
 │
 ├── curvas/
 │     └── bezier.html               ← Curvas de Bézier interativas
 │
 ├── transformacoes2d/
 │     ├── translacao.html           ← Translação (coordenadas homogêneas)
 │     ├── rotacao.html              ← Rotação 2D
 │     ├── escala.html               ← Escala e reflexão
 │     └── composicao.html           ← Composição com THREE.Group (sistema solar)
 │
 └── viewing/
       └── janela_viewport.html      ← Transformação Janela → Viewport
```

**Dependências:** nenhuma instalação necessária.
- `rasterizacao/` e `curvas/`: Canvas 2D puro (sem bibliotecas).
- `transformacoes2d/` e `viewing/`: [Three.js r128](https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js) via CDN.

**Como abrir:** qualquer arquivo HTML diretamente no navegador ou com Live Server (VS Code).

---

## Parte 1 — Rasterização

> **Conceito central:** converter geometria matemática contínua em pixels discretos.

A tela de um computador é uma grade de pixels. Quando pedimos para desenhar uma reta de
`(0,0)` a `(800,600)`, a GPU (ou, nestes exercícios, nosso código) precisa decidir quais
dos milhões de pixels da tela devem ser acesos. Esse processo é a **rasterização**.

---

### 1.1 — DDA (Digital Differential Analyzer)

**Arquivo:** `rasterizacao/dda.html`

#### Conceito

A reta matemática entre P1=(x1,y1) e P2=(x2,y2) pode ser descrita parametricamente.
O DDA avança passo a passo ao longo do **eixo dominante** (aquele com maior variação),
calculando a posição no eixo secundário como número de ponto flutuante e arredondando
para o pixel mais próximo.

#### Matemática

```
dx    = x2 - x1           (variação total em x)
dy    = y2 - y1           (variação total em y)
steps = max(|dx|, |dy|)   (número de pixels a plotar = eixo dominante)

xInc  = dx / steps        (incremento em x por passo — pode ser fração)
yInc  = dy / steps        (incremento em y por passo — pode ser fração)

para i de 0 até steps:
    pixel( round(x), round(y) )
    x ← x + xInc
    y ← y + yInc
```

A divisão por `steps` garante que o eixo dominante avance exatamente **1 pixel por passo**,
enquanto o eixo secundário avança proporcionalmente (entre 0 e 1).

#### Implementação — `dda.html` (linhas 238–290)

```javascript
function algoritmoDDA(x1, y1, x2, y2) {
  const dx    = x2 - x1;
  const dy    = y2 - y1;
  const steps = Math.max(Math.abs(dx), Math.abs(dy));

  // Incrementos fracionários: o eixo dominante avança 1/passo
  const xInc = dx / steps;
  const yInc = dy / steps;

  let x = x1;
  let y = y1;

  for (let i = 0; i <= steps; i++) {
    // Arredonda para o pixel inteiro mais próximo
    ctx.fillRect(Math.round(x), Math.round(y), 1, 1);

    x += xInc;   // avança fração em x
    y += yInc;   // avança fração em y
  }
}
```

#### Prós e contras

| Aspecto | DDA |
|---|---|
| Complexidade | Simples de entender |
| Aritmética | **Ponto flutuante** — divisão e arredondamento |
| Precisão | Boa, mas acumula erros de arredondamento |
| Performance | Mais lento (operações float) |
| Uso atual | Didático / referência |

---

### 1.2 — Bresenham para Retas

**Arquivo:** `rasterizacao/bresenham_reta.html`

#### Por que Bresenham?

O DDA usa `dx/steps` — uma divisão com resultado fracionário. Jack Bresenham (1962) percebeu
que, multiplicando tudo por `2·dx`, é possível tomar a mesma decisão usando **apenas
inteiros**: somas e subtrações, sem divisão, sem ponto flutuante. Isso foi revolucionário
para hardware gráfico da época e ainda é relevante hoje.

#### A variável de decisão P

A cada passo no eixo x, precisamos decidir: o próximo pixel é `(x+1, y)` ou `(x+1, y+1)`?

A distância da reta real ao pixel atual pode ser representada por um **parâmetro acumulado P**:

```
Parâmetro inicial (1º octante):
    p₀ = 2·dy − dx

A cada passo em x:
    se p ≥ 0:   y++              (a reta passou acima do centro do pixel)
                p ← p + 2·(dy − dx)

    se p < 0:   y não muda       (a reta ainda está abaixo)
                p ← p + 2·dy
```

A multiplicação por `2·dx` (implícita na fórmula acima) elimina toda divisão.
Somente **soma e subtração de inteiros**.

#### Generalização para 8 octantes

O algoritmo base só trata o 1º octante (inclinação entre 0 e 1, esquerda→direita).
Para cobrir qualquer reta, verificamos o **eixo dominante** e os **sinais** de dx e dy:

```
dx = |x2 - x1|,   sx = sinal(x2 - x1)   (+1 ou -1)
dy = |y2 - y1|,   sy = sinal(y2 - y1)   (+1 ou -1)

se |dx| ≥ |dy|:   eixo dominante = X
    p₀ = 2·dy − dx
    loop de 0 até dx:
        pinta (x, y)
        se p ≥ 0:  y += sy;  p += 2·(dy − dx)
        senão:               p += 2·dy
        x += sx              ← eixo dominante sempre avança

se |dy| > |dx|:   eixo dominante = Y  (troca de eixo)
    p₀ = 2·dx − dy
    loop de 0 até dy:
        pinta (x, y)
        se p ≥ 0:  x += sx;  p += 2·(dx − dy)
        senão:               p += 2·dx
        y += sy
```

#### Implementação — `bresenham_reta.html` (linhas 269–344)

```javascript
function bresenhamReta(x1, y1, x2, y2) {
  let dx = Math.abs(x2 - x1);
  let dy = Math.abs(y2 - y1);
  let sx = (x2 >= x1) ? 1 : -1;  // sinal de avanço em x
  let sy = (y2 >= y1) ? 1 : -1;  // sinal de avanço em y

  let x = x1, y = y1;

  if (dx >= dy) {
    // Eixo dominante = X
    let p = 2 * dy - dx;           // parâmetro de decisão inicial

    for (let i = 0; i <= dx; i++) {
      ctx.fillRect(x, y, 1, 1);    // pinta pixel

      if (p >= 0) {
        y += sy;                   // y sobe (ou desce)
        p += 2 * (dy - dx);        // atualização com mudança de y
      } else {
        p += 2 * dy;               // atualização sem mudança de y
      }
      x += sx;                     // eixo dominante sempre avança
    }
  } else {
    // Eixo dominante = Y (papéis de x e y trocados)
    let p = 2 * dx - dy;

    for (let i = 0; i <= dy; i++) {
      ctx.fillRect(x, y, 1, 1);

      if (p >= 0) {
        x += sx;
        p += 2 * (dx - dy);
      } else {
        p += 2 * dx;
      }
      y += sy;
    }
  }
}
```

#### Comparativo DDA × Bresenham

| | DDA | Bresenham |
|---|---|---|
| Aritmética | Float (divisão) | **Inteiros puros** |
| Operações/passo | 2 somas float + round | 1 soma int + comparação |
| Precisão | Boa | Ótima (sem acúmulo float) |
| Hardware | Lento | **Ideal para FPGA/GPU** |

---

### 1.3 — Bresenham para Círculos

**Arquivo:** `rasterizacao/bresenham_circulo.html`

#### A simetria de 8 octantes

Um círculo possui **simetria octafold**: o arco de 0° a 45° (1º octante)
é suficiente para reconstruir os 360°. Para cada ponto `(x, y)` calculado,
geramos 8 pixels de graça por reflexão:

```
Dado o ponto (x, y) relativo ao centro (cx, cy):

  Octante 1: (cx+x, cy−y)      Octante 5: (cx−x, cy+y)
  Octante 2: (cx+y, cy−x)      Octante 6: (cx−y, cy+x)
  Octante 3: (cx+y, cy+x)      Octante 7: (cx−y, cy−x)
  Octante 4: (cx+x, cy+y)      Octante 8: (cx−x, cy−y)
```

#### A variável de decisão do círculo

Começamos em `(x=0, y=r)` — topo do círculo — e percorremos até `x = y` (45°).
Em cada passo, decidimos se y decrementa ou mantém:

```
Parâmetro inicial:
    p₀ = 1 − r

A cada passo (x = 0, 1, 2, ...  enquanto x < y):
    x++

    se p < 0:   y não muda
                p ← p + 2·x + 1

    se p ≥ 0:   y−−
                p ← p + 2·(x − y) + 1

    pinta os 8 octantes com (cx±x, cy±y) e (cx±y, cy±x)
```

#### Derivação intuitiva da fórmula

O parâmetro `p` representa a diferença entre o quadrado do raio ideal
e a distância ao quadrado do pixel candidato:

```
p = x² + y² − r²

Quando p < 0: o ponto está dentro do círculo → y pode ficar
Quando p ≥ 0: o ponto saiu do círculo → y deve decrementar

A atualização incremental evita recalcular x²+y² do zero:
    Δp (y não muda) = 2·x_novo + 1   (diferença de x² entre passos)
    Δp (y decresce) = 2·(x_novo − y_novo) + 1
```

#### Implementação — `bresenham_circulo.html` (linhas 149–215)

```javascript
function bresenhamCirculo(cx, cy, r, cor) {
  // Pinta os 8 pixels simétricos de um só passo
  function pintaOctantes(x, y) {
    const pts = [
      [cx + x, cy - y],  [cx + y, cy - x],
      [cx + y, cy + x],  [cx + x, cy + y],
      [cx - x, cy + y],  [cx - y, cy + x],
      [cx - y, cy - x],  [cx - x, cy - y],
    ];
    pts.forEach(([px, py]) => ctx.fillRect(px, py, 1, 1));
  }

  let x = 0;
  let y = r;
  let p = 1 - r;          // parâmetro de decisão inicial

  pintaOctantes(x, y);    // pinta ponto inicial (topo)

  while (x < y) {         // percorre até 45°
    x++;

    if (p < 0) {
      p = p + 2 * x + 1;           // y não muda
    } else {
      y--;
      p = p + 2 * (x - y) + 1;    // y decrementou
    }

    pintaOctantes(x, y);
  }
}
```

---

## Parte 2 — Curvas de Bézier

**Arquivo:** `curvas/bezier.html`

### Conceito

Uma curva de Bézier é definida por **pontos de controle** que atraem
a curva sem necessariamente tocá-la. A curva passa obrigatoriamente
pelos pontos extremos (P₀ e Pₙ) e é contida no **fecho convexo** dos pontos de controle.

### Fórmula Geral — Polinômios de Bernstein

Para `n+1` pontos de controle P₀, P₁, …, Pₙ e parâmetro `t ∈ [0, 1]`:

```
         n
B(t) =  Σ  C(n,i) · tⁱ · (1−t)^(n−i) · Pᵢ
        i=0

onde C(n,i) = n! / (i! · (n−i)!)   (coeficiente binomial)
```

### Casos particulares

```
Grau 1 — linear (2 pontos):
    B(t) = (1−t)·P₀ + t·P₁

Grau 2 — quadrática (3 pontos):
    B(t) = (1−t)²·P₀ + 2·(1−t)·t·P₁ + t²·P₂

Grau 3 — cúbica (4 pontos):
    B(t) = (1−t)³·P₀ + 3·(1−t)²·t·P₁ + 3·(1−t)·t²·P₂ + t³·P₃
```

### Implementação — `bezier.html`

```javascript
// Coeficiente binomial C(n,k) — versão iterativa sem fatorial
function binomial(n, k) {
  if (k < 0 || k > n) return 0;
  if (k === 0 || k === n) return 1;
  let resultado = 1;
  for (let i = 0; i < k; i++) {
    resultado = resultado * (n - i) / (i + 1);
  }
  return Math.round(resultado);
}

// Ponto na curva de Bézier dado t ∈ [0,1]
function bezier(t, pts) {
  const n = pts.length - 1;   // grau = nº de pontos - 1
  let bx = 0, by = 0;

  for (let i = 0; i <= n; i++) {
    // Peso de Bernstein para o i-ésimo ponto de controle
    const bernstein = binomial(n, i)
                    * Math.pow(t, i)
                    * Math.pow(1 - t, n - i);
    bx += bernstein * pts[i].x;
    by += bernstein * pts[i].y;
  }

  return { x: bx, y: by };
}

// Renderiza a curva avaliando 'resolucao' pontos
for (let i = 0; i <= resolucao; i++) {
  const t = i / resolucao;    // t varia de 0 a 1
  const p = bezier(t, pontos);
  ctx.lineTo(p.x, p.y);
}
```

### Algoritmo de De Casteljau

De Casteljau é uma forma **geométrica** de calcular o mesmo ponto B(t).
Em vez de avaliar a fórmula diretamente, interpola linearmente pares
adjacentes de pontos de forma recursiva, até restar um único ponto:

```
Nível 0 (original):   P₀   P₁   P₂   P₃
                        ↓ lerp(t) entre pares
Nível 1:          Q₀=(1-t)P₀+tP₁,  Q₁=(1-t)P₁+tP₂,  Q₂=(1-t)P₂+tP₃
                        ↓ lerp(t) entre pares
Nível 2:          R₀=(1-t)Q₀+tQ₁,  R₁=(1-t)Q₁+tQ₂
                        ↓ lerp(t)
Nível 3:          B(t) = (1-t)R₀ + t·R₁   ← ponto na curva
```

```javascript
function deCasteljau(t, pts) {
  let atual = pts.slice();

  while (atual.length > 1) {
    const proximo = [];
    for (let i = 0; i < atual.length - 1; i++) {
      // Interpolação linear entre pontos adjacentes
      proximo.push({
        x: (1 - t) * atual[i].x + t * atual[i + 1].x,
        y: (1 - t) * atual[i].y + t * atual[i + 1].y,
      });
    }
    atual = proximo;   // substitui pelo nível interpolado
  }

  return atual[0];   // ponto final na curva
}
```

### Interatividade em `bezier.html`

| Ação | Resultado |
|---|---|
| Clique em área vazia | Adiciona ponto de controle |
| Arrastar ponto | Move ponto de controle (re-renderiza em tempo real) |
| Duplo clique em ponto | Remove o ponto |
| Botão "Animar t" | Desenha a construção De Casteljau animada |
| Slider "Resolução" | Controla quantos pontos são avaliados na curva |

---

## Parte 3 — Transformações 2D

> **Tecnologia:** Three.js com OrthographicCamera (sem perspectiva).

### Por que coordenadas homogêneas?

Em 2D, um ponto é `[x, y]`. Com matrizes 2×2 conseguimos rotação e escala,
mas **não translação** — somar um vetor não é multiplicação de matriz.

A solução é adicionar uma coordenada extra `w=1`, formando o vetor `[x, y, 1]`
(coordenadas homogêneas). Com isso, todas as transformações afins (T, R, S e
composições) são **multiplicações de matriz 3×3**.

```
Ponto 2D normal:       [x, y]
Ponto homogêneo:       [x, y, 1]   (w = 1 para pontos, w = 0 para vetores)

Transformação:   P' = M · P
```

---

### 3.1 — Translação

**Arquivo:** `transformacoes2d/translacao.html`

```
Matriz de Translação T(tx, ty):

    | 1  0  tx |   | x |   | x + tx |
    | 0  1  ty | × | y | = | y + ty |
    | 0  0   1 |   | 1 |   |   1    |
```

A translação **exige** a terceira linha/coluna homogênea.
Sem ela, não há como somar `tx` e `ty` por multiplicação de matriz.

**No Three.js:**
```javascript
// Three.js representa internamente como matriz 4x4
// Para 2D usamos apenas position.x e position.y:
mesh.position.x += tx;
mesh.position.y += ty;

// Ou equivalente:
mesh.translateX(tx);
mesh.translateY(ty);
```

**Controles:** setas `← → ↑ ↓` ou botões na tela. O campo "Passo" define
quantas unidades por acionamento.

---

### 3.2 — Rotação

**Arquivo:** `transformacoes2d/rotacao.html`

#### Rotação ao redor da origem

```
Matriz de Rotação R(θ):

    | cos θ  −sin θ  0 |   | x |   | x·cos θ − y·sin θ |
    | sin θ   cos θ  0 | × | y | = | x·sin θ + y·cos θ |
    |   0       0    1 |   | 1 |   |         1          |
```

Propriedades:
- `R(θ)` é **ortogonal**: `R⁻¹(θ) = Rᵀ(θ) = R(−θ)`
- Preserva distâncias (isometria)
- Ângulo positivo = sentido anti-horário (convenção matemática)

**No Three.js (rotação ao redor da origem):**
```javascript
mesh.rotation.z += theta;   // radianos, Z = rotação no plano XY
```

#### Rotação ao redor de ponto arbitrário (px, py)

Para rotacionar em torno de um ponto que não é a origem:

```
M = T(px, py) · R(θ) · T(−px, −py)

Sequência:
  1. Translada (px,py) para a origem
  2. Rotaciona
  3. Translada de volta
```

**Implementação em `rotacao.html`:**
```javascript
function girar(sentido) {
  const delta = sentido * passo_em_radianos;
  const px = pivX, py = pivY;

  // Posição atual em relação ao pivô
  const dx = mesh.position.x - px;
  const dy = mesh.position.y - py;

  const cosA = Math.cos(delta);
  const sinA = Math.sin(delta);

  // Aplica R(θ) à posição relativa ao pivô
  mesh.position.x = px + dx * cosA - dy * sinA;
  mesh.position.y = py + dx * sinA + dy * cosA;

  // Rotaciona o próprio objeto
  mesh.rotation.z += delta;
}
```

**Controles:** teclas `Q` (−θ) e `E` (+θ), ou botões. Campo "Piv. X/Y" define o pivô.

---

### 3.3 — Escala

**Arquivo:** `transformacoes2d/escala.html`

```
Matriz de Escala S(sx, sy):

    | sx   0   0 |   | x |   | sx · x |
    |  0  sy   0 | × | y | = | sy · y |
    |  0   0   1 |   | 1 |   |    1   |
```

Casos especiais:

```
sx = sy          → escala uniforme (preserva forma)
sx ≠ sy          → escala não-uniforme (distorce)
0 < sx < 1       → reduz horizontalmente
sx > 1           → aumenta horizontalmente
sx = −1          → reflexão em relação ao eixo Y
sy = −1          → reflexão em relação ao eixo X
sx = sy = −1     → rotação de 180° (equivalente)
```

**No Three.js:**
```javascript
mesh.scale.set(sx, sy, 1);   // sz=1 → sem escala em Z para 2D
```

#### Escala ao redor de ponto arbitrário

```
M = T(px, py) · S(sx, sy) · T(−px, −py)
```

**Controles:** sliders independentes para `sx` e `sy` (−3 a +3), botões de reflexão,
teclado `+/-` para escala uniforme, `X`/`Y` para reflexões.

---

### 3.4 — Composição de Transformações

**Arquivo:** `transformacoes2d/composicao.html`

#### A ordem das matrizes importa!

```
Aplicar S, depois R, depois T:
    M = T · R · S

A multiplicação lê-se da direita para esquerda:
primeiro S é aplicado, depois R, depois T.

T·R ≠ R·T   (matrices não comutam em geral)
```

#### THREE.Group — hierarquia de transformações

O Three.js implementa composição naturalmente via hierarquia de objetos.
A transformação no **espaço mundo** de um nó filho é o produto das
matrizes de todos os seus ancestrais:

```
M_mundo_filho = M_pai · M_avô · M_bisavô · ... · M_local_filho
```

**Sistema Solar em `composicao.html`:**

```javascript
// Hierarquia de grupos:
scene
└── grupoSol                     ← rotação própria do sol
    ├── solMesh
    └── grupOrbTerra              ← orbita da Terra ao redor do Sol
        └── grupoTerra            ← rotação própria da Terra
            ├── terraMesh
            └── grupOrbLua        ← órbita da Lua ao redor da Terra
                └── luaMesh

// Loop de animação:
solMesh.rotation.z    += velSol;       // só o sol gira
grupOrbTerra.rotation.z += velTerra;  // toda a Terra (+ Lua) orbita o Sol
grupoTerra.rotation.z   += velPropria; // Terra gira em torno de si
grupOrbLua.rotation.z   += velLua;    // Lua orbita a Terra
```

A Lua herda automaticamente a posição da Terra, que herda a posição
do Sol — sem cálculos manuais de posição absoluta.

**Para obter a posição no espaço mundo:**
```javascript
const posicaoMundo = new THREE.Vector3();
luaMesh.getWorldPosition(posicaoMundo);
// retorna as coordenadas absolutas, após multiplicar todas as matrizes
```

---

## Parte 4 — Viewing 2D (Janela → Viewport)

**Arquivo:** `viewing/janela_viewport.html`

### O pipeline de Viewing 2D

O processo de exibição 2D envolve dois espaços:

```
Mundo (World Space)              Tela (Screen Space)
┌─────────────────┐              ┌──────────────────┐
│                 │              │                  │
│   Janela        │  ─────────→  │   Viewport       │
│  (wx, wy)       │  mapeamento  │   (vx, vy) px    │
│  coords lógicas │              │   pixels de tela │
└─────────────────┘              └──────────────────┘
   wx ∈ [wxMin, wxMax]              vx ∈ [vxMin, vxMax]
   wy ∈ [wyMin, wyMax]              vy ∈ [vyMin, vyMax]
```

### Fórmula de Mapeamento

```
Fatores de escala:
    sx = (vxMax − vxMin) / (wxMax − wxMin)
    sy = (vyMax − vyMin) / (wyMax − wyMin)

Mapeamento de ponto (xw, yw) → (xv, yv):
    xv = vxMin + (xw − wxMin) · sx
    yv = vyMin + (yw − wyMin) · sy
```

Esta é uma **interpolação linear** (lerp) que normaliza o ponto dentro
da janela e re-escala para o viewport.

#### Inversão do eixo Y

Em coordenadas matemáticas, Y cresce para cima.
Em canvas/tela, Y cresce para baixo. A correção:

```
yv = vyMin + (wyMax − yw) · sy    ← usa (wyMax − yw) em vez de (yw − wyMin)
```

#### Implementação — `janela_viewport.html`

```javascript
function janelaParaViewport(xw, yw, janela, viewport) {
  const sx = (viewport.xMax - viewport.xMin)
           / (janela.xMax   - janela.xMin);
  const sy = (viewport.yMax - viewport.yMin)
           / (janela.yMax   - janela.yMin);

  const xv = viewport.xMin + (xw - janela.xMin) * sx;
  // Inverte Y (canvas cresce para baixo, mundo cresce para cima)
  const yv = viewport.yMin + (janela.yMax - yw)  * sy;

  return { x: xv, y: yv };
}
```

### Distorção por proporções diferentes

Se a janela tem proporção 16:9 e o viewport é quadrado (1:1),
os objetos serão **distorcidos**:

```
Janela:    largura 20 × altura 10  →  razão 2:1
Viewport:  400px   × 400px        →  razão 1:1

sx = 400/20 = 20    (1 unidade = 20 pixels em X)
sy = 400/10 = 40    (1 unidade = 40 pixels em Y)
sx ≠ sy → distorção!
```

Para preservar proporções: `sx = sy = min(sx_calc, sy_calc)` e ajustar
o viewport. Em `janela_viewport.html`, a opção **"Manter proporção"**
faz isso automaticamente.

### Equivalência com OrthographicCamera (Three.js)

```javascript
// A câmera define exatamente a janela:
const camera = new THREE.OrthographicCamera(
  wxMin,   // left
  wxMax,   // right
  wyMax,   // top
  wyMin,   // bottom
  near, far
);

// O renderer define o viewport:
renderer.setSize(vxMax - vxMin, vyMax - vyMin);
renderer.setViewport(vxMin, vyMin, vxMax - vxMin, vyMax - vyMin);
// Three.js calcula a fórmula acima internamente.
```

---

## Referência Rápida — Matrizes de Transformação

Todas as transformações 2D em coordenadas homogêneas (matriz 3×3):

```
Translação T(tx, ty):          Escala S(sx, sy):
| 1   0   tx |                 | sx   0   0 |
| 0   1   ty |                 |  0  sy   0 |
| 0   0    1 |                 |  0   0   1 |

Rotação R(θ):                  Reflexão eixo X:
| cosθ  −sinθ  0 |             |  1   0   0 |
| sinθ   cosθ  0 |             |  0  −1   0 |
|  0      0    1 |             |  0   0   1 |

Identidade I:                  Cisalhamento Sh(shx, shy):
| 1  0  0 |                    |  1  shx  0 |
| 0  1  0 |                    | shy   1  0 |
| 0  0  1 |                    |  0    0  1 |
```

### Composição — ordem de aplicação

```javascript
// Aplicar S primeiro, depois R, depois T:
// (lê-se da direita para esquerda na multiplicação)
M = T * R * S

// No Three.js, a ordem de propriedades equivale a:
// scale → aplicado no espaço local
// rotation → aplicado após scale
// position → aplicado por último (translação no espaço pai)
mesh.scale.set(sx, sy, 1);
mesh.rotation.z = theta;
mesh.position.set(tx, ty, 0);
```

---

## Referência de Controles

| Arquivo | Teclado | Mouse |
|---|---|---|
| `dda.html` | — | Clique 2× para P1→P2 |
| `bresenham_reta.html` | — | Clique 2× para P1→P2 |
| `bresenham_circulo.html` | — | Clique 1× centro, 2× raio |
| `bezier.html` | — | Clique = add ponto, drag = mover, dblclick = remover |
| `translacao.html` | `← → ↑ ↓` mover, `R` reset | — |
| `rotacao.html` | `Q` ← , `E` →, `R` reset | — |
| `escala.html` | `+/-` uniforme, `X/Y` reflexão, `R` reset | Sliders |
| `composicao.html` | — | Sliders velocidade/raio |
| `janela_viewport.html` | — | Inputs de limites |

---

## Conceitos-Chave Resumidos

| Conceito | Definição curta |
|---|---|
| **Rasterização** | Converter geometria contínua em pixels discretos |
| **Eixo dominante** | O eixo com maior variação; determina o número de passos |
| **Variável de decisão P** | Inteiro acumulado que decide qual pixel é mais próximo da curva |
| **Coord. homogêneas** | Adição de coordenada w para representar translação como matriz |
| **Composição de matrizes** | Produto de matrizes de transformação; ordem importa |
| **Fecho convexo** | Menor polígono convexo que contém todos os pontos de controle |
| **Janela (Window)** | Região do espaço-mundo que está sendo visualizada |
| **Viewport** | Região de pixels na tela onde a janela é projetada |

---

## Progressão Sugerida de Estudo

```
1. rasterizacao/dda.html
       ↓  (mesmo problema, solução melhor)
2. rasterizacao/bresenham_reta.html
       ↓  (mesmo princípio, geometria circular)
3. rasterizacao/bresenham_circulo.html
       ↓  (saindo do pixel, entrando em geometria paramétrica)
4. curvas/bezier.html
       ↓  (transformações no plano)
5. transformacoes2d/translacao.html
6. transformacoes2d/rotacao.html
7. transformacoes2d/escala.html
       ↓  (tudo junto)
8. transformacoes2d/composicao.html
       ↓  (como a câmera enxerga o mundo)
9. viewing/janela_viewport.html
```

Cada etapa constrói sobre a anterior. Os comentários dentro do código
de cada arquivo detalham a matemática específica daquele algoritmo.

---

## Créditos

**Autor:** Pedro
**Ano:** 2026
**Disciplina:** Computação Gráfica 2D

### Licença

Este material está licenciado sob a
[Creative Commons Atribuição 4.0 Internacional (CC BY 4.0)](https://creativecommons.org/licenses/by/4.0/).

```
© 2026 Pedro Ximenes — CC BY 4.0

Você é livre para:
  ✓ Compartilhar — copiar e redistribuir em qualquer formato
  ✓ Adaptar      — remixar, transformar e criar a partir do material
  ✓ Uso comercial e não comercial

Sob as seguintes condições:
  ✎ Atribuição  — Você deve dar crédito ao autor original (Pedro),
                   indicar se alterações foram feitas e incluir
                   um link para a licença.
```

> Material desenvolvido para fins didáticos.
> Algoritmos implementados em HTML + JavaScript puro e Three.js.
