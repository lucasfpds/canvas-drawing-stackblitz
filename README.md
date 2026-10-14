# Canvas Drawing App

> Ferramentas de desenho livre, formas, borracha e paleta de cores, com undo/redo e salvamento local.

## Tecnologias

- **Vue 3** com `<script setup>` (JavaScript)
- **Vite** como bundler
- **Canvas 2D API** nativa para desenho
- **CSS puro** — sem pré-processador
- **devicePixelRatio** para traços nítidos em telas retina
- **localStorage** para persistência do desenho

## Requisitos

- Node.js 18+

## Como executar

```bash
npm install
npm run dev
```

Abra http://localhost:5173 no navegador.

## Scripts

| Script | Descrição |
| --- | --- |
| `npm run dev` | Servidor de desenvolvimento (Vite) |
| `npm run build` | Build de produção |
| `npm run preview` | Serve o build de produção |

## Estrutura

```
src/
  main.js              entry: cria o app e importa o CSS
  App.vue              toolbar, canvas, ferramentas, undo/redo, salvar
  styles.css           layout, toolbar, paleta, canvas
```

### Funcionalidades

- 5 ferramentas: lápis, linha, retângulo, círculo, borracha
- Paleta com 10 cores + `input type="color"` para cor personalizada
- Slider de espessura (1–30px)
- Lápis/borracha com interpolação entre eventos (traço contínuo via `quadraticCurveTo`)
- Borracha via `globalCompositeOperation = 'destination-out'`
- Formas com preview em tempo real (snapshot + redraw)
- Undo/redo com pilha de `ImageData` (limite 30 níveis)
- Salvamento em `localStorage` — o desenho persiste ao recarregar
- Botão "novo" limpa o canvas e remove o item salvo


## Rodando no StackBlitz

Basta importar o repositório e rodar normalmente:

```
https://stackblitz.com/github/lucasfpds/canvas-drawing-stackblitz
```

O projeto não depende de binários nativos — Vite é JavaScript puro, então
sobe sem bindings nativos no WebContainer do StackBlitz.
