# CLAUDE.md — Corrida Sem Fim A&CIA Móveis

## Visão Geral do Projeto

Jogo de corrida infinita (endless runner) no estilo pixel art desenvolvido para a empresa **A&CIA Móveis**.
Tecnologia: HTML5 + JavaScript puro (Canvas API), sem frameworks ou dependências externas.
Funciona 100% no navegador, desktop e mobile.

---

## Estrutura de Arquivos

```
acia_runner/
├── index.html          ← jogo completo (código JS inline)
└── assets/             ← todos os PNGs do jogo
    ├── personagem_01.png   (frame 1 de 8 — animação de corrida)
    ├── personagem_02.png   (frame 2)
    ├── personagem_03.png   (frame 3)
    ├── personagem_04.png   (frame 4)
    ├── personagem_05.png   (frame 5)
    ├── personagem_06.png   (frame 6)
    ├── personagem_07.png   (frame 7)
    ├── personagem_08.png   (frame 8)
    ├── cidade.png          (background do jogo em loop — 4832×953px)
    ├── inicio.png          (background da tela inicial — fachada da loja — 2442×954px)
    ├── titulo.png          (logo "Corrida Sem Fim A&CIA Móveis" — 2150×741px)
    ├── coracao.png         (ícone de vida — pixel art coração vermelho)
    ├── quadro_coracao.png  (caixa/frame para os 3 corações no HUD)
    ├── escudo.png          (item coletável — escudo ciano)
    ├── efeito_escudo.png   (efeito visual circular ao redor do personagem)
    ├── buster_velocidade.png (item coletável — raio dourado de velocidade)
    ├── balao_de_fala.png   (balão "Bota pra torar!!!" — aparece ao pegar buster)
    ├── moeda.png           (item coletável — moeda dourada)
    ├── cone.png            (obstáculo — cone de trânsito)
    ├── buraco.png          (obstáculo — buraco na rua)
    └── pedra.png           (obstáculo — pedra)
```

---

## Mecânicas do Jogo

### Controles
- **PC:** `ESPAÇO` ou `↑` = Pular | `P` / `ESC` = Pausar | `R` = Ranking
- **Mobile:** Toque na tela = Pular | Arrastar para baixo = Agachar

### Sistema de Vidas
- 3 vidas (corações) exibidas no HUD dentro do `quadro_coracao.png`
- Ao colidir com obstáculo: perde 1 vida + invencibilidade de ~110 frames (~1.8s)
- Com 1 vida restante: coração pulsa para alertar o jogador
- Game Over quando perder as 3 vidas

### Itens Coletáveis
| Item | Efeito |
|------|--------|
| Moeda | +10 pontos, contador no HUD |
| Escudo | Bloqueia 1 colisão, barra ciano no HUD por 5 segundos |
| Buster (raio) | +3.5 velocidade por 4 segundos, trilha de fantasmas, balão "Bota pra torar!!!" |

### Obstáculos
- **Cone** — pular por cima
- **Pedra** — pular por cima
- **Buraco** — pular por cima

### Progressão de Dificuldade
- Velocidade inicial: `5.5`
- A cada 500m: `+0.8` de velocidade
- Velocidade máxima: `18`
- Intervalo de spawn diminui conforme avança

### Personagem
- Corre automaticamente no meio da rua (`GROUND_Y = 269` no canvas 800×380)
- 8 frames de animação (`personagem_01.png` a `personagem_08.png`) em loop
- FPS da animação aumenta com a velocidade do jogo
- Pisca durante invencibilidade
- Trilha de fantasmas ao pegar o buster

---

## Arquitetura do Código (index.html)

### Canvas
- Tamanho lógico: **800×380px**
- Responsivo via `SCALE` — se adapta ao tamanho da janela (máx 2×)
- `ctx.imageSmoothingEnabled = false` — mantém pixel art nítido

### Constantes Principais
```javascript
const GROUND_Y  = 269;   // posição Y dos pés do personagem (meio da rua)
const CHAR_X    = 150;   // posição X fixa do personagem
const GRAVITY   = 0.62;
const JUMP_VY   = -13.2;
const CHAR_W    = 90, CHAR_H = 106;  // tamanho de renderização do personagem
const COL_W     = 52, COL_H  = 88;   // caixa de colisão
```

### Estados do Jogo
```
title → playing → paused → playing
                → dead → (nameOverlay) → gameover → title
```

### Carregamento de Imagens
```javascript
const IMG = { f01:"assets/personagem_01.png", ... };
// Imagens em REMOVE_BLACK recebem remoção de fundo preto via Canvas API
const REMOVE_BLACK = []; // transparência já pré-processada nos PNGs
```

### Sistema de Ranking
- Salvo em `localStorage` com chave `csf_rank`
- Até 20 entradas, ordenado por distância (metros)
- Campos: `{ name, dist, coins }`
- Exibido via overlay HTML (`#rankOverlay`) com tabela Top 15

### Overlays HTML (fora do canvas)
- `#rankOverlay` — tela de ranking (z-index 20)
- `#nameOverlay` — input de nome após game over (z-index 21)
- `#pauseBtn` — botão ⏸ (canto superior direito)

---

## HUD (Interface durante o jogo)

```
[❤❤❤] quadro_coracao          [📦 XXXm]          [🪙 XX]
                               [VEL Xx ⚡]         [PT XXXX]
[🛡 ████████░░] barra escudo
[⚡ ████░░░░░░] barra buster
```

---

## Background / Cenário

### Tela Inicial
- `inicio.png` (fachada da A&CIA Móveis + cidade) — renderizado estático cobrindo 100% do canvas
- Logo `titulo.png` centralizado no topo
- Painel translúcido na parte inferior com botão de start e recorde

### Durante o Jogo
- `cidade.png` (4832×953) — rolagem infinita em parallax (`bgX -= speed * 0.3`)
- Imagem escalada para cobrir exatamente a altura do canvas (380px)
- Personagem corre na faixa do meio da rua (GROUND_Y=269 = 71% da altura da imagem)

---

## Deploy / Como Colocar Online

### Opção 1 — Servidor estático simples
```bash
# Qualquer hosting estático funciona (Vercel, Netlify, GitHub Pages, etc.)
# Basta fazer upload da pasta acia_runner/ mantendo a estrutura:
acia_runner/
├── index.html
└── assets/  (todos os PNGs)
```

### Opção 2 — Vercel (recomendado para facilidade)
```bash
cd acia_runner
npx vercel --yes
```

### Opção 3 — Netlify drag & drop
1. Acesse app.netlify.com
2. Arraste a pasta `acia_runner/` para o deploy zone

### Opção 4 — GitHub Pages
```bash
git init
git add .
git commit -m "Corrida Sem Fim A&CIA"
git branch -M main
git remote add origin https://github.com/SEU_USER/acia-runner.git
git push -u origin main
# Ativar GitHub Pages nas configurações do repo → branch main → / (root)
```

### Opção 5 — Servidor próprio (Apache/Nginx)
```bash
# Copiar para o webroot
cp -r acia_runner/ /var/www/html/corrida/
# Acessar: https://seusite.com.br/corrida/
```

> ⚠️ **Importante:** `index.html` e pasta `assets/` devem estar sempre juntos no mesmo diretório.
> O jogo carrega as imagens via caminhos relativos `assets/nome.png`.

---

## Pontos para Evolução Futura

- [ ] Ranking online (substituir localStorage por API/backend)
- [ ] Efeitos sonoros e música de fundo
- [ ] Novos obstáculos: cachorro, moto, piso molhado
- [ ] Novos cenários: noite, chuva, trânsito
- [ ] Skins do personagem desbloqueáveis por moedas
- [ ] Sistema de missões diárias
- [ ] Compartilhamento de score nas redes sociais
- [ ] PWA (Progressive Web App) para instalação no celular

---

## Informações da Empresa

- **Empresa:** A&CIA Móveis
- **Slogan:** "A Loja que Vai até Você!"
- **Jogo:** Corrida Sem Fim A&CIA
- **Paleta de cores da marca:** Azul `#1a44cc`, Vermelho `#cc1a1a`, Amarelo `#FFD700`, Branco

---

## Observações Técnicas

- Todos os PNGs de sprites já têm fundo transparente (pré-processado com PIL/Python)
- `cidade.png` e `inicio.png` são cenas completas — NÃO aplicar remoção de fundo
- O jogo usa `localStorage` para persistir ranking — funciona mesmo offline
- `roundRect` é usado via Canvas API (disponível em todos browsers modernos)
- Testado em: Chrome, Firefox, Safari, Edge, mobile iOS e Android
