<script setup>
import { ref, onMounted, onBeforeUnmount } from "vue";

const canvas = ref(null);
const tool = ref("pencil");
const color = ref("#7c3aed");
const size = ref(6);
const palette = ["#000000", "#ffffff", "#ef4444", "#f59e0b", "#10b981", "#3b82f6", "#7c3aed", "#ec4899", "#14b8a6", "#6366f1"];

let ctx = null;
let drawing = false;
let lastX = 0;
let lastY = 0;
let snapshot = null;
let shapeStart = null;
const undoStack = [];
const redoStack = [];
const MAX_UNDO = 30;
let pointerId = null;

function setupCanvas() {
  const c = canvas.value;
  const dpr = window.devicePixelRatio || 1;
  const rect = c.getBoundingClientRect();
  c.width = Math.floor(rect.width * dpr);
  c.height = Math.floor(rect.height * dpr);
  ctx = c.getContext("2d");
  ctx.setTransform(dpr, 0, 0, dpr, 0, 0);
  ctx.lineCap = "round";
  ctx.lineJoin = "round";
  loadSaved();
}

function getPos(e) {
  const rect = canvas.value.getBoundingClientRect();
  return { x: e.clientX - rect.left, y: e.clientY - rect.top };
}

function pushUndo() {
  const c = canvas.value;
  undoStack.push(ctx.getImageData(0, 0, c.width, c.height));
  if (undoStack.length > MAX_UNDO) undoStack.shift();
  redoStack.length = 0;
}

function onPointerDown(e) {
  drawing = true;
  pointerId = e.pointerId;
  canvas.value.setPointerCapture(e.pointerId);
  const { x, y } = getPos(e);
  lastX = x;
  lastY = y;
  pushUndo();
  if (tool.value === "pencil" || tool.value === "eraser") {
    ctx.beginPath();
    ctx.moveTo(x, y);
    ctx.lineTo(x, y);
    applyStroke();
  } else {
    shapeStart = { x, y };
    snapshot = ctx.getImageData(0, 0, canvas.value.width, canvas.value.height);
  }
}

function applyStroke() {
  ctx.lineWidth = size.value;
  ctx.strokeStyle = color.value;
  ctx.globalCompositeOperation = tool.value === "eraser" ? "destination-out" : "source-over";
  ctx.stroke();
}

function onPointerMove(e) {
  if (!drawing || e.pointerId !== pointerId) return;
  const { x, y } = getPos(e);
  if (tool.value === "pencil" || tool.value === "eraser") {
    const midX = (lastX + x) / 2;
    const midY = (lastY + y) / 2;
    ctx.beginPath();
    ctx.moveTo(lastX, lastY);
    ctx.quadraticCurveTo(lastX, lastY, midX, midY);
    applyStroke();
    lastX = x;
    lastY = y;
  } else {
    ctx.putImageData(snapshot, 0, 0);
    ctx.globalCompositeOperation = "source-over";
    ctx.strokeStyle = color.value;
    ctx.lineWidth = size.value;
    ctx.beginPath();
    if (tool.value === "line") {
      ctx.moveTo(shapeStart.x, shapeStart.y);
      ctx.lineTo(x, y);
    } else if (tool.value === "rect") {
      ctx.rect(shapeStart.x, shapeStart.y, x - shapeStart.x, y - shapeStart.y);
    } else if (tool.value === "circle") {
      const r = Math.hypot(x - shapeStart.x, y - shapeStart.y);
      ctx.arc(shapeStart.x, shapeStart.y, r, 0, Math.PI * 2);
    }
    ctx.stroke();
  }
}

function onPointerUp(e) {
  if (!drawing) return;
  drawing = false;
  shapeStart = null;
  snapshot = null;
  save();
}

function undo() {
  if (undoStack.length === 0) return;
  const c = canvas.value;
  redoStack.push(ctx.getImageData(0, 0, c.width, c.height));
  const img = undoStack.pop();
  ctx.putImageData(img, 0, 0);
  save();
}

function redo() {
  if (redoStack.length === 0) return;
  const c = canvas.value;
  undoStack.push(ctx.getImageData(0, 0, c.width, c.height));
  const img = redoStack.pop();
  ctx.putImageData(img, 0, 0);
  save();
}

function clearCanvas() {
  pushUndo();
  ctx.clearRect(0, 0, canvas.value.width, canvas.value.height);
  save();
}

function newCanvas() {
  ctx.clearRect(0, 0, canvas.value.width, canvas.value.height);
  undoStack.length = 0;
  redoStack.length = 0;
  localStorage.removeItem("canvas-drawing");
}

function save() {
  localStorage.setItem("canvas-drawing", canvas.value.toDataURL("image/png"));
}

function loadSaved() {
  const data = localStorage.getItem("canvas-drawing");
  if (!data) return;
  const img = new Image();
  img.onload = () => {
    const rect = canvas.value.getBoundingClientRect();
    ctx.drawImage(img, 0, 0, rect.width, rect.height);
  };
  img.src = data;
}

function onResize() {
  const data = canvas.value.toDataURL("image/png");
  setupCanvas();
  const img = new Image();
  img.onload = () => {
    const rect = canvas.value.getBoundingClientRect();
    ctx.drawImage(img, 0, 0, rect.width, rect.height);
  };
  img.src = data;
}

onMounted(() => {
  setupCanvas();
  window.addEventListener("resize", onResize);
});

onBeforeUnmount(() => {
  window.removeEventListener("resize", onResize);
});

const tools = [
  { id: "pencil", label: "✏️ Lápis" },
  { id: "line", label: "／ Linha" },
  { id: "rect", label: "▭ Retângulo" },
  { id: "circle", label: "◯ Círculo" },
  { id: "eraser", label: "🧽 Borracha" },
];
</script>

<template>
  <main class="app">
    <h1>Canvas Drawing</h1>
    <div class="toolbar">
      <div class="tools">
        <button
          v-for="t in tools"
          :key="t.id"
          :class="{ active: tool === t.id }"
          @click="tool = t.id"
        >{{ t.label }}</button>
      </div>
      <div class="palette">
        <button
          v-for="c in palette"
          :key="c"
          :style="{ background: c }"
          :class="{ active: color === c }"
          :aria-label="'Cor ' + c"
          @click="color = c"
        ></button>
        <input type="color" v-model="color" class="picker" aria-label="Cor personalizada" />
      </div>
      <div class="size">
        <label for="brush-size">Espessura: {{ size }}px</label>
        <input id="brush-size" type="range" min="1" max="30" v-model.number="size" />
      </div>
      <div class="actions">
        <button @click="undo" title="Desfazer">↶</button>
        <button @click="redo" title="Refazer">↷</button>
        <button @click="clearCanvas" title="Limpar">🚫</button>
        <button @click="newCanvas" title="Novo">📄</button>
      </div>
    </div>
    <canvas
      ref="canvas"
      class="canvas"
      @pointerdown="onPointerDown"
      @pointermove="onPointerMove"
      @pointerup="onPointerUp"
      @pointercancel="onPointerUp"
    ></canvas>
  </main>
</template>
