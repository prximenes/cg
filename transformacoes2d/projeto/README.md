# Projeto — Transformações Geométricas 2D

## Objetivo

Criar uma **cena ou aplicação 2D interativa** usando HTML, JavaScript e Canvas 2D que demonstre domínio das transformações geométricas aprendidas em aula.

O tema é **livre** — o grupo escolhe o que quer criar.

## Informações Gerais

- **Grupos:** até 3 alunos
- **Prazo:** 17 dias corridos a partir da data da aula prática
- **Tecnologia:** JavaScript + HTML + Canvas 2D
- **Alternativa:** quem preferir pode usar OpenGL (C/C++ ou WebGL)

## Requisitos Obrigatórios

O projeto **deve** utilizar:

1. **Translação** (`ctx.translate`) — mover objetos na cena
2. **Rotação** (`ctx.rotate`) — girar pelo menos um objeto
3. **Escala** (`ctx.scale`) — redimensionar objetos
4. **Composição de transformações** — combinar pelo menos duas operações em sequência (ex: rotação + translação)
5. **Rotação ou escala com ponto fixo** — pelo menos um caso usando o padrão T → Op → T
6. **Animação** — usar `requestAnimationFrame` com reset de matriz a cada frame
7. **save/restore ou setTransform** — gerenciamento correto da pilha de estados

## Requisitos Bônus (pontos extras)

- Interatividade com teclado ou mouse
- Uso de reflexão ou cisalhamento
- Hierarquia de transformações (ex: braço de robô, sistema solar com luas)

## Sugestões de Tema

- **Sistema Solar 2D** — Sol no centro, planetas orbitando, luas orbitando planetas
- **Relógio analógico** — ponteiros girando com rotação em ponto fixo
- **Aquário** — peixes nadando com translate + rotate na direção do movimento
- **Cena de cidade** — casas, carros movendo, nuvens, objetos compostos
- **Robô articulado** — braço com segmentos que rotacionam hierarquicamente
- **Jogo simples** — tipo Asteroids: nave com rotação, tiros, asteroides
- **Padrão geométrico animado** — mosaico, caleidoscópio, fractais
- **Engrenagens** — rodas dentadas interligadas rotacionando

Ou invente o seu!

## Critérios de Avaliação

| Critério | Peso |
|----------|------|
| Uso correto das transformações (T, R, S) | 30% |
| Composição e ordem das transformações | 20% |
| Rotação/escala com ponto fixo | 15% |
| Animação com reset de matriz | 15% |
| Organização do código (save/restore, funções) | 10% |
| Criatividade e apresentação visual | 10% |
| **Bônus:** interatividade, reflexão, hierarquia | +10% |

## Entrega

- Arquivos: `index.html` + qualquer JS/CSS auxiliar
- Deve funcionar abrindo o `index.html` direto no navegador (sem servidor)
- Incluir um `README.md` no projeto explicando:
  - Tema escolhido
  - Nomes dos integrantes do grupo
  - Quais transformações foram usadas e onde no código
- Entregar via repositório Git ou arquivo `.zip`

## Template

Um arquivo `template.html` está disponível nesta pasta como ponto de partida. Ele já contém:

- Canvas configurado
- Loop de animação com `requestAnimationFrame`
- Reset da matriz a cada frame
- Função de exemplo para desenhar formas
- Comentários indicando onde colocar suas transformações

### Como usar o template

1. Copie `template.html` e renomeie para `index.html`
2. Crie funções para desenhar seus objetos
3. Adicione variáveis de estado (ângulo, posição, escala)
4. Aplique as transformações no loop de animação
5. Incremente: interatividade, mais objetos, composições

## Dicas

- **Comece simples!** Desenhe formas básicas (`fillRect`, `arc`, `lineTo`) e aplique transformações
- Use `save()/restore()` para isolar transformações de cada objeto
- Lembre-se: no Canvas 2D, a leitura das transformações é **de baixo para cima**
- Sempre resete a matriz no início de cada frame
- Ângulos são em **radianos**: `rad = graus * Math.PI / 180`
