# Canvas Drawing App

> Ferramentas de desenho livre, formas, borracha e paleta de cores, com undo/redo e salvamento local.

## Stack

- Vite + Vue 3 (`<script setup>`, JavaScript) — sem TypeScript, sem lint/test, zero libs extras
- CSS puro em `src/styles.css`
- Arquivos: `index.html`, `package.json` (deps: `vue`), `vite.config.js`, `.stackblitzrc`, `src/main.js`, `src/App.vue`, `src/styles.css`
- App.vue único — todo o estado da ferramenta atual (ferramenta, cor, espessura) no `setup`

## Implementação

### 1. Estrutura

- `<canvas>` ocupando a área principal; toolbar no topo:
  - ferramentas: lápis, linha, retângulo, círculo, borracha
  - paleta: ~10 swatches + `input type="color"`
  - espessura: range 1–30
  - ações: undo, redo, limpar, salvar
- Canvas dimensionado com `devicePixelRatio` (traço nítido); coordenadas do ponteiro convertidas via `getBoundingClientRect()` + escala

### 2. Ferramentas

- Lápis/borracha: pontos em `pointermove` com interpolação entre eventos (traço contínuo), `lineCap`/`lineJoin: round`
- Borracha: mesmo traço com `globalCompositeOperation = 'destination-out'`
- Formas (linha/retângulo/círculo): guardar o ponto inicial; a cada `pointermove`, restaurar o snapshot do início do gesto (`putImageData`) e redesenhar a forma como preview; no `pointerup` a forma vira definitiva

### 3. Undo/redo (snapshots)

- Ao fim de cada ação (traço, forma, limpar): `undoStack.push(canvas.getImageData())`, limite 30 (descarta a mais antiga); `redoStack` zera a cada ação nova
- Undo: `redoStack.push(atual)` → `putImageData(undoStack.pop())`; redo é o inverso

### 4. Salvamento local

- Salvar: `localStorage.setItem('canvas-drawing', canvas.toDataURL('image/png'))`
- Ao carregar a página: se houver item salvo, desenhar a imagem (`Image` + `drawImage`) no canvas
- Botão "novo": limpa o canvas e remove o item salvo

## Checklist — 100% da descrição

- [ ] Desenho livre
- [ ] Formas (linha, retângulo, círculo)
- [ ] Borracha
- [ ] Paleta de cores
- [ ] Undo/redo
- [ ] Salvamento local (persiste ao recarregar)
