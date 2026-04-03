<!DOCTYPE html>.
<html lang="vi">,;;;;;;;;;;;;;;

<head>,
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Neural Network Visualizer</title>
<style>
  @import url('https://fonts.googleapis.com/css2?family=Space+Mono:wght@400;700&family=Syne:wght@400;700;800&display=swap');

  :root {
    --bg: #050a0f;
    --surface: #0d1821;
    --border: #1a2a3a;
    --accent: #00e5ff;
    --accent2: #ff6b35;
    --accent3: #7fff6b;
    --text: #cce8ff;
    --muted: #4a6a8a;
    --positive: #00ff9f;
    --negative: #ff4466;
  }

  * { margin: 0; padding: 0; box-sizing: border-box; }

  body {
    background: var(--bg);
    color: var(--text);
    font-family: 'Space Mono', monospace;
    min-height: 100vh;
    overflow-x: hidden;
  }

  /* Noise texture overlay */
  body::before {
    content: '';
    position: fixed;
    inset: 0;
    background-image: url("data:image/svg+xml,%3Csvg viewBox='0 0 256 256' xmlns='http://www.w3.org/2000/svg'%3E%3Cfilter id='noise'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='0.9' numOctaves='4' stitchTiles='stitch'/%3E%3C/filter%3E%3Crect width='100%25' height='100%25' filter='url(%23noise)' opacity='0.04'/%3E%3C/svg%3E");
    pointer-events: none;
    z-index: 9999;
    opacity: 0.4;
  }

  header {
    padding: 2rem 3rem;
    border-bottom: 1px solid var(--border);
    display: flex;
    align-items: center;
    justify-content: space-between;
    background: rgba(13, 24, 33, 0.8);
    backdrop-filter: blur(12px);
    position: sticky;
    top: 0;
    z-index: 100;
  }

  .logo {
    font-family: 'Syne', sans-serif;
    font-size: 1.4rem;
    font-weight: 800;
    letter-spacing: -0.02em;
    color: var(--accent);
    text-shadow: 0 0 20px rgba(0, 229, 255, 0.4);
  }

  .logo span { color: var(--text); }

  .status-bar {
    display: flex;
    gap: 2rem;
    font-size: 0.7rem;
    color: var(--muted);
  }

  .status-item { display: flex; align-items: center; gap: 0.5rem; }
  .dot {
    width: 6px; height: 6px;
    border-radius: 50%;
    background: var(--positive);
    box-shadow: 0 0 6px var(--positive);
    animation: pulse 2s infinite;
  }

  @keyframes pulse {
    0%, 100% { opacity: 1; }
    50% { opacity: 0.3; }
  }

  .main-layout {
    display: grid;
    grid-template-columns: 280px 1fr 280px;
    gap: 0;
    height: calc(100vh - 73px);
  }

  .sidebar {
    border-right: 1px solid var(--border);
    padding: 1.5rem;
    overflow-y: auto;
    background: var(--surface);
  }

  .sidebar-right {
    border-left: 1px solid var(--border);
    border-right: none;
  }

  .section-title {
    font-family: 'Syne', sans-serif;
    font-size: 0.65rem;
    font-weight: 700;
    letter-spacing: 0.15em;
    text-transform: uppercase;
    color: var(--muted);
    margin-bottom: 1rem;
    padding-bottom: 0.5rem;
    border-bottom: 1px solid var(--border);
  }

  .control-group { margin-bottom: 1.5rem; }

  label {
    display: block;
    font-size: 0.7rem;
    color: var(--muted);
    margin-bottom: 0.4rem;
  }

  input[type="range"] {
    -webkit-appearance: none;
    width: 100%;
    height: 2px;
    background: var(--border);
    outline: none;
    border-radius: 2px;
  }

  input[type="range"]::-webkit-slider-thumb {
    -webkit-appearance: none;
    width: 12px; height: 12px;
    border-radius: 50%;
    background: var(--accent);
    cursor: pointer;
    box-shadow: 0 0 8px var(--accent);
  }

  .value-display {
    font-size: 0.75rem;
    color: var(--accent);
    text-align: right;
    margin-top: 0.3rem;
  }

  .btn {
    display: block;
    width: 100%;
    padding: 0.6rem 1rem;
    border: 1px solid var(--border);
    background: transparent;
    color: var(--text);
    font-family: 'Space Mono', monospace;
    font-size: 0.7rem;
    cursor: pointer;
    transition: all 0.2s;
    text-align: left;
    letter-spacing: 0.05em;
    margin-bottom: 0.5rem;
  }

  .btn:hover {
    border-color: var(--accent);
    color: var(--accent);
    background: rgba(0, 229, 255, 0.05);
    box-shadow: 0 0 12px rgba(0, 229, 255, 0.1);
  }

  .btn-primary {
    border-color: var(--accent);
    color: var(--accent);
    background: rgba(0, 229, 255, 0.08);
  }

  .btn-primary:hover {
    background: rgba(0, 229, 255, 0.15);
    box-shadow: 0 0 20px rgba(0, 229, 255, 0.2);
  }

  .btn-danger {
    border-color: var(--negative);
    color: var(--negative);
  }

  .btn-danger:hover {
    background: rgba(255, 68, 102, 0.08);
    box-shadow: 0 0 12px rgba(255, 68, 102, 0.1);
  }

  /* Canvas area */
  .canvas-area {
    position: relative;
    overflow: hidden;
    background: radial-gradient(ellipse at center, #0a1520 0%, var(--bg) 70%);
  }

  canvas {
    position: absolute;
    top: 0; left: 0;
    width: 100%; height: 100%;
  }

  .canvas-overlay {
    position: absolute;
    bottom: 1.5rem;
    left: 50%;
    transform: translateX(-50%);
    display: flex;
    gap: 0.75rem;
    z-index: 10;
  }

  .pill {
    padding: 0.4rem 1rem;
    border: 1px solid var(--border);
    background: rgba(13, 24, 33, 0.85);
    backdrop-filter: blur(8px);
    font-size: 0.65rem;
    color: var(--muted);
    border-radius: 20px;
    cursor: pointer;
    transition: all 0.2s;
    letter-spacing: 0.05em;
  }

  .pill:hover, .pill.active {
    border-color: var(--accent);
    color: var(--accent);
  }

  /* Info panels */
  .metric-card {
    background: var(--bg);
    border: 1px solid var(--border);
    padding: 1rem;
    margin-bottom: 0.75rem;
  }

  .metric-label {
    font-size: 0.62rem;
    color: var(--muted);
    text-transform: uppercase;
    letter-spacing: 0.1em;
    margin-bottom: 0.3rem;
  }

  .metric-value {
    font-family: 'Syne', sans-serif;
    font-size: 1.5rem;
    font-weight: 700;
    color: var(--accent);
    line-height: 1;
  }

  .metric-sub {
    font-size: 0.62rem;
    color: var(--muted);
    margin-top: 0.2rem;
  }

  .weight-grid {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(28px, 1fr));
    gap: 3px;
    margin-top: 0.5rem;
  }

  .weight-cell {
    height: 28px;
    border-radius: 2px;
    transition: all 0.3s;
    cursor: default;
    position: relative;
  }

  .activation-bar {
    height: 4px;
    border-radius: 2px;
    background: var(--border);
    overflow: hidden;
    margin-bottom: 0.3rem;
  }

  .activation-fill {
    height: 100%;
    border-radius: 2px;
    transition: width 0.4s ease;
  }

  .node-label {
    font-size: 0.6rem;
    color: var(--muted);
    margin-bottom: 0.2rem;
    display: flex;
    justify-content: space-between;
  }

  .log-area {
    background: var(--bg);
    border: 1px solid var(--border);
    padding: 0.75rem;
    font-size: 0.62rem;
    height: 160px;
    overflow-y: auto;
    font-family: 'Space Mono', monospace;
  }

  .log-line {
    padding: 0.15rem 0;
    border-bottom: 1px solid rgba(26, 42, 58, 0.5);
    color: var(--muted);
  }

  .log-line.info { color: var(--accent); }
  .log-line.success { color: var(--positive); }
  .log-line.warn { color: var(--accent2); }

  select {
    width: 100%;
    padding: 0.5rem;
    background: var(--bg);
    border: 1px solid var(--border);
    color: var(--text);
    font-family: 'Space Mono', monospace;
    font-size: 0.7rem;
    outline: none;
    cursor: pointer;
  }

  select:focus { border-color: var(--accent); }

  .layer-tags {
    display: flex;
    flex-wrap: wrap;
    gap: 0.4rem;
    margin-top: 0.5rem;
  }

  .layer-tag {
    padding: 0.2rem 0.6rem;
    border: 1px solid var(--border);
    font-size: 0.6rem;
    border-radius: 2px;
    color: var(--muted);
    cursor: pointer;
    transition: all 0.2s;
  }

  .layer-tag.active {
    border-color: var(--accent2);
    color: var(--accent2);
    background: rgba(255, 107, 53, 0.08);
  }

  scrollbar-width: thin;
  scrollbar-color: var(--border) transparent;

  ::-webkit-scrollbar { width: 4px; }
  ::-webkit-scrollbar-track { background: transparent; }
  ::-webkit-scrollbar-thumb { background: var(--border); border-radius: 2px; }

  .epoch-counter {
    font-family: 'Syne', sans-serif;
    font-size: 3rem;
    font-weight: 800;
    color: var(--accent);
    text-shadow: 0 0 30px rgba(0, 229, 255, 0.3);
    line-height: 1;
  }

  .loss-mini-chart {
    height: 60px;
    position: relative;
    overflow: hidden;
  }

  .loss-mini-chart canvas {
    position: relative;
    width: 100%;
    height: 100%;
  }
</style>
</head>
<body>

<header>
  <div class="logo">NEURAL<span>VIZ</span></div>
  <div class="status-bar">
    <div class="status-item"><div class="dot"></div> LIVE</div>
    <div class="status-item" id="param-count">PARAMS: —</div>
    <div class="status-item" id="fps-counter">FPS: —</div>
  </div>
</header>

<div class="main-layout">

  <!-- LEFT SIDEBAR -->
  <div class="sidebar">
    <div class="control-group">
      <div class="section-title">Kiến trúc mạng</div>

      <label>Activation Function</label>
      <select id="activation-select">
        <option value="relu">ReLU</option>
        <option value="sigmoid">Sigmoid</option>
        <option value="tanh">Tanh</option>
        <option value="leaky_relu">Leaky ReLU</option>
        <option value="elu">ELU</option>
      </select>
    </div>

    <div class="control-group">
      <label>Số lượng lớp ẩn: <span id="layers-val">3</span></label>
      <input type="range" id="layers-slider" min="1" max="6" value="3">
    </div>

    <div class="control-group">
      <label>Neurons mỗi lớp: <span id="neurons-val">6</span></label>
      <input type="range" id="neurons-slider" min="2" max="12" value="6">
    </div>

    <div class="control-group">
      <label>Learning Rate: <span id="lr-val">0.010</span></label>
      <input type="range" id="lr-slider" min="1" max="100" value="10">
    </div>

    <div class="control-group">
      <div class="section-title">Điều khiển</div>
      <button class="btn btn-primary" id="train-btn">▶ BẮT ĐẦU TRAINING</button>
      <button class="btn" id="step-btn">⏭ FORWARD PASS</button>
      <button class="btn btn-danger" id="reset-btn">↺ RESET WEIGHTS</button>
    </div>

    <div class="control-group">
      <div class="section-title">Tốc độ animation</div>
      <label>Speed: <span id="speed-val">5</span></label>
      <input type="range" id="speed-slider" min="1" max="10" value="5">
    </div>

    <div class="control-group">
      <div class="section-title">Layer Inspector</div>
      <div class="layer-tags" id="layer-tags"></div>
    </div>
  </div>

  <!-- CENTER CANVAS -->
  <div class="canvas-area">
    <canvas id="network-canvas"></canvas>
    <div class="canvas-overlay">
      <div class="pill active" data-view="network">NETWORK</div>
      <div class="pill" data-view="weights">WEIGHT MAP</div>
      <div class="pill" data-view="gradient">GRADIENT FLOW</div>
    </div>
  </div>

  <!-- RIGHT SIDEBAR -->
  <div class="sidebar sidebar-right">
    <div class="section-title">Training Stats</div>

    <div class="metric-card">
      <div class="metric-label">Epoch</div>
      <div class="epoch-counter" id="epoch-display">0</div>
    </div>

    <div class="metric-card">
      <div class="metric-label">Loss</div>
      <div class="metric-value" id="loss-display">—</div>
      <div class="metric-sub" id="loss-delta">Δ —</div>
      <div class="loss-mini-chart">
        <canvas id="loss-chart"></canvas>
      </div>
    </div>

    <div class="metric-card">
      <div class="metric-label">Accuracy</div>
      <div class="metric-value" id="acc-display">—</div>
    </div>

    <div class="section-title" style="margin-top:1rem">Activations</div>
    <div id="activation-display"></div>

    <div class="section-title" style="margin-top:1rem">Console Log</div>
    <div class="log-area" id="log-area"></div>
  </div>
</div>

<script>
// ============================================================
//   NEURAL NETWORK ENGINE
// ============================================================

class ActivationFunctions {
  static relu(x) { return Math.max(0, x); }
  static reluDeriv(x) { return x > 0 ? 1 : 0; }

  static sigmoid(x) { return 1 / (1 + Math.exp(-x)); }
  static sigmoidDeriv(x) { const s = ActivationFunctions.sigmoid(x); return s * (1 - s); }

  static tanh(x) { return Math.tanh(x); }
  static tanhDeriv(x) { return 1 - Math.pow(Math.tanh(x), 2); }

  static leaky_relu(x) { return x > 0 ? x : 0.01 * x; }
  static leaky_reluDeriv(x) { return x > 0 ? 1 : 0.01; }

  static elu(x) { return x >= 0 ? x : (Math.exp(x) - 1); }
  static eluDeriv(x) { return x >= 0 ? 1 : Math.exp(x); }

  static apply(name, x) { return ActivationFunctions[name](x); }
  static deriv(name, x) { return ActivationFunctions[`${name}Deriv`](x); }
}

class Matrix {
  constructor(rows, cols, init = 'random') {
    this.rows = rows;
    this.cols = cols;
    this.data = Array.from({length: rows}, () =>
      Array.from({length: cols}, () => {
        if (init === 'random') return (Math.random() * 2 - 1) * Math.sqrt(2 / cols);
        if (init === 'zeros') return 0;
        return 0;
      })
    );
  }

  static multiply(a, b) {
    if (a.cols !== b.rows) throw new Error('Matrix dimension mismatch');
    const result = new Matrix(a.rows, b.cols, 'zeros');
    for (let i = 0; i < a.rows; i++)
      for (let j = 0; j < b.cols; j++)
        for (let k = 0; k < a.cols; k++)
          result.data[i][j] += a.data[i][k] * b.data[k][j];
    return result;
  }

  static add(a, b) {
    const result = new Matrix(a.rows, a.cols, 'zeros');
    for (let i = 0; i < a.rows; i++)
      for (let j = 0; j < a.cols; j++)
        result.data[i][j] = a.data[i][j] + b.data[i][j];
    return result;
  }

  static subtract(a, b) {
    const result = new Matrix(a.rows, a.cols, 'zeros');
    for (let i = 0; i < a.rows; i++)
      for (let j = 0; j < a.cols; j++)
        result.data[i][j] = a.data[i][j] - b.data[i][j];
    return result;
  }

  map(fn) {
    const result = new Matrix(this.rows, this.cols, 'zeros');
    for (let i = 0; i < this.rows; i++)
      for (let j = 0; j < this.cols; j++)
        result.data[i][j] = fn(this.data[i][j], i, j);
    return result;
  }

  transpose() {
    const result = new Matrix(this.cols, this.rows, 'zeros');
    for (let i = 0; i < this.rows; i++)
      for (let j = 0; j < this.cols; j++)
        result.data[j][i] = this.data[i][j];
    return result;
  }

  static fromArray(arr) {
    const m = new Matrix(arr.length, 1, 'zeros');
    for (let i = 0; i < arr.length; i++) m.data[i][0] = arr[i];
    return m;
  }

  toArray() {
    return this.data.map(row => row[0]);
  }

  scale(s) {
    return this.map(x => x * s);
  }

  hadamard(other) {
    const result = new Matrix(this.rows, this.cols, 'zeros');
    for (let i = 0; i < this.rows; i++)
      for (let j = 0; j < this.cols; j++)
        result.data[i][j] = this.data[i][j] * other.data[i][j];
    return result;
  }
}

class NeuralNetwork {
  constructor(layerSizes, activationName = 'relu', learningRate = 0.01) {
    this.layerSizes = layerSizes;
    this.activationName = activationName;
    this.lr = learningRate;
    this.weights = [];
    this.biases = [];
    this.preActivations = [];
    this.activations = [];
    this.gradients = [];
    this.initWeights();
  }

  initWeights() {
    this.weights = [];
    this.biases = [];
    for (let i = 0; i < this.layerSizes.length - 1; i++) {
      this.weights.push(new Matrix(this.layerSizes[i+1], this.layerSizes[i]));
      this.biases.push(new Matrix(this.layerSizes[i+1], 1, 'zeros'));
    }
    this.gradients = this.weights.map(w => new Matrix(w.rows, w.cols, 'zeros'));
  }

  forward(input) {
    this.activations = [Matrix.fromArray(input)];
    this.preActivations = [];

    for (let i = 0; i < this.weights.length; i++) {
      const z = Matrix.add(
        Matrix.multiply(this.weights[i], this.activations[i]),
        this.biases[i]
      );
      this.preActivations.push(z);
      const isLast = i === this.weights.length - 1;
      const a = isLast
        ? z.map(x => ActivationFunctions.sigmoid(x))
        : z.map(x => ActivationFunctions.apply(this.activationName, x));
      this.activations.push(a);
    }

    return this.activations[this.activations.length - 1];
  }

  backward(target) {
    const targetM = Matrix.fromArray(Array.isArray(target) ? target : [target]);
    const output = this.activations[this.activations.length - 1];
    let delta = Matrix.subtract(output, targetM);

    const newGrads = [];

    for (let i = this.weights.length - 1; i >= 0; i--) {
      const isLast = i === this.weights.length - 1;
      const deriv = isLast
        ? this.preActivations[i].map(x => ActivationFunctions.sigmoidDeriv(x))
        : this.preActivations[i].map(x => ActivationFunctions.deriv(this.activationName, x));

      delta = delta.hadamard(deriv);
      const weightGrad = Matrix.multiply(delta, this.activations[i].transpose());
      newGrads[i] = weightGrad;
      this.weights[i] = Matrix.subtract(this.weights[i], weightGrad.scale(this.lr));
      this.biases[i] = Matrix.subtract(this.biases[i], delta.scale(this.lr));

      if (i > 0) {
        delta = Matrix.multiply(this.weights[i].transpose(), delta);
      }
    }

    this.gradients = newGrads.map((g, i) => g || this.gradients[i]);
    return newGrads;
  }

  loss(output, target) {
    const o = output.toArray();
    const t = Array.isArray(target) ? target : [target];
    return t.reduce((sum, tv, i) => sum + 0.5 * Math.pow((o[i] || 0) - tv, 2), 0);
  }

  get paramCount() {
    return this.weights.reduce((s, w) => s + w.rows * w.cols, 0)
         + this.biases.reduce((s, b) => s + b.rows, 0);
  }
}

// ============================================================
//   VISUALIZATION ENGINE
// ============================================================

class NetworkRenderer {
  constructor(canvas, nn) {
    this.canvas = canvas;
    this.ctx = canvas.getContext('2d');
    this.nn = nn;
    this.animProgress = [];
    this.signalParticles = [];
    this.viewMode = 'network';
    this.selectedLayer = -1;
    this.resize();
  }

  resize() {
    this.canvas.width = this.canvas.offsetWidth * window.devicePixelRatio;
    this.canvas.height = this.canvas.offsetHeight * window.devicePixelRatio;
    this.ctx.scale(window.devicePixelRatio, window.devicePixelRatio);
    this.W = this.canvas.offsetWidth;
    this.H = this.canvas.offsetHeight;
  }

  getNodePositions() {
    const sizes = this.nn.layerSizes;
    const marginX = 80;
    const xStep = (this.W - marginX * 2) / (sizes.length - 1);
    const positions = [];

    for (let l = 0; l < sizes.length; l++) {
      const count = sizes[l];
      const yStep = Math.min((this.H - 80) / Math.max(count - 1, 1), 60);
      const totalH = (count - 1) * yStep;
      const startY = (this.H - totalH) / 2;
      const x = marginX + l * xStep;
      const layer = [];
      for (let n = 0; n < count; n++) {
        layer.push({ x, y: startY + n * yStep });
      }
      positions.push(layer);
    }
    return positions;
  }

  getNodeActivation(layer, node) {
    if (!this.nn.activations || !this.nn.activations[layer]) return 0;
    const a = this.nn.activations[layer];
    return Math.tanh(a.data[node] ? a.data[node][0] : 0);
  }

  getWeightColor(w) {
    const abs = Math.min(Math.abs(w), 3) / 3;
    if (w > 0) {
      const g = Math.floor(abs * 229);
      return `rgba(0, ${g}, 255, ${0.1 + abs * 0.6})`;
    } else {
      const r = Math.floor(abs * 255);
      return `rgba(${r}, 68, 102, ${0.1 + abs * 0.6})`;
    }
  }

  getActivationColor(val) {
    const v = (val + 1) / 2;
    const r = Math.floor((1 - v) * 255);
    const g = Math.floor(v * 229);
    return `rgb(${r}, ${g}, 180)`;
  }

  spawnSignal(fromLayer, fromNode) {
    const pos = this.getNodePositions();
    if (fromLayer >= pos.length - 1) return;
    const src = pos[fromLayer][fromNode];
    const nextCount = this.nn.layerSizes[fromLayer + 1];

    for (let t = 0; t < nextCount; t++) {
      const dst = pos[fromLayer + 1][t];
      const w = this.nn.weights[fromLayer].data[t][fromNode];
      const strength = Math.min(Math.abs(w), 2) / 2;
      if (strength < 0.1) continue;
      this.signalParticles.push({
        sx: src.x, sy: src.y,
        tx: dst.x, ty: dst.y,
        t: 0, speed: 0.02 + Math.random() * 0.02,
        strength,
        positive: w > 0,
        layer: fromLayer
      });
    }
  }

  drawWeightMap() {
    const ctx = this.ctx;
    ctx.clearRect(0, 0, this.W, this.H);

    if (this.nn.weights.length === 0) return;

    const L = this.selectedLayer >= 0
      ? Math.min(this.selectedLayer, this.nn.weights.length - 1)
      : 0;

    const W = this.nn.weights[L];
    const cellW = Math.min((this.W - 80) / W.cols, 40);
    const cellH = Math.min((this.H - 80) / W.rows, 40);
    const totalW = cellW * W.cols;
    const totalH = cellH * W.rows;
    const startX = (this.W - totalW) / 2;
    const startY = (this.H - totalH) / 2;

    ctx.fillStyle = 'rgba(255,255,255,0.03)';
    ctx.font = '10px Space Mono';
    ctx.fillText(`LAYER ${L} WEIGHTS  [${W.rows} × ${W.cols}]`, startX, startY - 20);

    const maxW = W.data.flat().reduce((m, v) => Math.max(m, Math.abs(v)), 0.0001);

    for (let r = 0; r < W.rows; r++) {
      for (let c = 0; c < W.cols; c++) {
        const val = W.data[r][c];
        const norm = val / maxW;
        const cx = startX + c * cellW;
        const cy = startY + r * cellH;

        const alpha = 0.2 + Math.abs(norm) * 0.8;
        if (val > 0) {
          ctx.fillStyle = `rgba(0, 200, 255, ${alpha})`;
        } else {
          ctx.fillStyle = `rgba(255, 68, 102, ${alpha})`;
        }
        ctx.fillRect(cx + 1, cy + 1, cellW - 2, cellH - 2);
      }
    }
  }

  drawGradientFlow() {
    const ctx = this.ctx;
    ctx.clearRect(0, 0, this.W, this.H);
    const pos = this.getNodePositions();

    for (let l = 0; l < this.nn.gradients.length; l++) {
      const g = this.nn.gradients[l];
      if (!g) continue;
      const maxG = g.data.flat().reduce((m, v) => Math.max(m, Math.abs(v)), 0.0001);

      for (let r = 0; r < Math.min(g.rows, pos[l+1]?.length || 0); r++) {
        for (let c = 0; c < Math.min(g.cols, pos[l]?.length || 0); c++) {
          const gval = g.data[r][c];
          const norm = Math.abs(gval) / maxG;
          const src = pos[l][c];
          const dst = pos[l+1][r];
          const alpha = norm * 0.7;
          const color = gval > 0
            ? `rgba(0, 255, 159, ${alpha})`
            : `rgba(255, 107, 53, ${alpha})`;

          ctx.beginPath();
          ctx.moveTo(src.x, src.y);
          ctx.lineTo(dst.x, dst.y);
          ctx.strokeStyle = color;
          ctx.lineWidth = norm * 3;
          ctx.stroke();
        }
      }
    }

    // Labels
    for (let l = 0; l < pos.length; l++) {
      for (let n = 0; n < pos[l].length; n++) {
        const {x, y} = pos[l][n];
        ctx.beginPath();
        ctx.arc(x, y, 10, 0, Math.PI * 2);
        ctx.fillStyle = 'rgba(13,24,33,0.9)';
        ctx.fill();
        ctx.strokeStyle = 'rgba(74,106,138,0.6)';
        ctx.lineWidth = 1;
        ctx.stroke();
      }
    }
  }

  drawNetwork() {
    const ctx = this.ctx;
    ctx.clearRect(0, 0, this.W, this.H);
    const pos = this.getNodePositions();

    // Grid
    ctx.strokeStyle = 'rgba(26,42,58,0.3)';
    ctx.lineWidth = 0.5;
    const gridStep = 40;
    for (let x = 0; x < this.W; x += gridStep) {
      ctx.beginPath(); ctx.moveTo(x, 0); ctx.lineTo(x, this.H); ctx.stroke();
    }
    for (let y = 0; y < this.H; y += gridStep) {
      ctx.beginPath(); ctx.moveTo(0, y); ctx.lineTo(this.W, y); ctx.stroke();
    }

    // Draw connections
    for (let l = 0; l < pos.length - 1; l++) {
      for (let n = 0; n < pos[l].length; n++) {
        for (let t = 0; t < pos[l+1].length; t++) {
          if (this.nn.weights[l]) {
            const w = this.nn.weights[l].data[t]?.[n] || 0;
            ctx.beginPath();
            ctx.moveTo(pos[l][n].x, pos[l][n].y);
            ctx.lineTo(pos[l+1][t].x, pos[l+1][t].y);
            ctx.strokeStyle = this.getWeightColor(w);
            ctx.lineWidth = Math.min(Math.abs(w) * 2, 2);
            ctx.stroke();
          }
        }
      }
    }

    // Draw signal particles
    this.signalParticles = this.signalParticles.filter(p => {
      p.t += p.speed;
      if (p.t >= 1) return false;

      const x = p.sx + (p.tx - p.sx) * p.t;
      const y = p.sy + (p.ty - p.sy) * p.t;
      const size = 3 + p.strength * 4;
      const alpha = Math.sin(p.t * Math.PI);

      ctx.beginPath();
      ctx.arc(x, y, size, 0, Math.PI * 2);
      const color = p.positive ? `rgba(0, 229, 255, ${alpha})` : `rgba(255, 68, 102, ${alpha})`;
      ctx.fillStyle = color;
      ctx.fill();

      // glow
      const grd = ctx.createRadialGradient(x, y, 0, x, y, size * 2);
      grd.addColorStop(0, p.positive ? `rgba(0,229,255,${alpha*0.3})` : `rgba(255,68,102,${alpha*0.3})`);
      grd.addColorStop(1, 'transparent');
      ctx.fillStyle = grd;
      ctx.beginPath();
      ctx.arc(x, y, size * 2, 0, Math.PI * 2);
      ctx.fill();

      return true;
    });

    // Draw nodes
    for (let l = 0; l < pos.length; l++) {
      const isInput = l === 0;
      const isOutput = l === pos.length - 1;

      for (let n = 0; n < pos[l].length; n++) {
        const {x, y} = pos[l][n];
        const act = this.getNodeActivation(l, n);

        // Shadow/glow
        const glowColor = isOutput ? 'rgba(127, 255, 107, 0.3)'
          : isInput ? 'rgba(255, 107, 53, 0.3)'
          : 'rgba(0, 229, 255, 0.3)';
        ctx.shadowBlur = 12;
        ctx.shadowColor = glowColor;

        // Node
        ctx.beginPath();
        ctx.arc(x, y, 14, 0, Math.PI * 2);
        const grad = ctx.createRadialGradient(x - 3, y - 3, 2, x, y, 14);

        const baseColor = isOutput ? '#7fff6b' : isInput ? '#ff6b35' : '#00e5ff';
        const fillAlpha = 0.2 + Math.abs(act) * 0.5;
        grad.addColorStop(0, baseColor + '44');
        grad.addColorStop(1, 'rgba(13,24,33,0.9)');
        ctx.fillStyle = grad;
        ctx.fill();

        ctx.shadowBlur = 0;
        ctx.strokeStyle = baseColor;
        ctx.lineWidth = 1.5;
        ctx.stroke();

        // Activation value text
        ctx.fillStyle = 'rgba(255,255,255,0.5)';
        ctx.font = `7px Space Mono`;
        ctx.textAlign = 'center';
        ctx.fillText(act.toFixed(1), x, y + 3);
      }
    }

    // Layer labels
    const labels = ['INPUT', ...this.nn.layerSizes.slice(1, -1).map((_, i) => `H${i+1}`), 'OUTPUT'];
    ctx.fillStyle = 'rgba(74,106,138,0.7)';
    ctx.font = '9px Space Mono';
    ctx.textAlign = 'center';
    for (let l = 0; l < pos.length; l++) {
      const x = pos[l][0].x;
      ctx.fillText(labels[l] || `L${l}`, x, 20);
      ctx.fillText(`×${this.nn.layerSizes[l]}`, x, 32);
    }
  }

  render() {
    if (this.viewMode === 'weights') this.drawWeightMap();
    else if (this.viewMode === 'gradient') this.drawGradientFlow();
    else this.drawNetwork();
  }
}

// ============================================================
//   APP STATE & CONTROLLER
// ============================================================

let nn, renderer;
let isTraining = false;
let trainInterval = null;
let epoch = 0;
let lossHistory = [];
let lastLoss = null;
let currentView = 'network';
let fps = 0, frameCount = 0, lastFpsTime = performance.now();

// XOR dataset (4 samples, can extend)
const dataset = [
  { input: [0, 0, 0, 1], output: [0] },
  { input: [0, 0, 1, 0], output: [0] },
  { input: [0, 1, 0, 0], output: [1] },
  { input: [0, 1, 1, 0], output: [1] },
  { input: [1, 0, 0, 1], output: [0] },
  { input: [1, 0, 1, 0], output: [1] },
  { input: [1, 1, 0, 0], output: [1] },
  { input: [1, 1, 1, 1], output: [0] },
];

function buildLayerSizes() {
  const hidden = parseInt(document.getElementById('layers-slider').value);
  const neurons = parseInt(document.getElementById('neurons-slider').value);
  return [4, ...Array(hidden).fill(neurons), 1];
}

function getLearningRate() {
  return parseInt(document.getElementById('lr-slider').value) / 1000;
}

function getActivation() {
  return document.getElementById('activation-select').value;
}

function initNetwork() {
  const sizes = buildLayerSizes();
  const lr = getLearningRate();
  const act = getActivation();
  nn = new NeuralNetwork(sizes, act, lr);
  epoch = 0;
  lossHistory = [];
  lastLoss = null;
  document.getElementById('epoch-display').textContent = '0';
  document.getElementById('loss-display').textContent = '—';
  document.getElementById('acc-display').textContent = '—';
  document.getElementById('loss-delta').textContent = 'Δ —';
  document.getElementById('param-count').textContent = `PARAMS: ${nn.paramCount.toLocaleString()}`;
  updateLayerTags();
  updateActivationDisplay();
  log('Network reset. Layers: ' + sizes.join(' → '), 'info');
}

function updateLayerTags() {
  const container = document.getElementById('layer-tags');
  container.innerHTML = '';
  nn.layerSizes.forEach((_, i) => {
    const label = i === 0 ? 'INPUT' : i === nn.layerSizes.length - 1 ? 'OUTPUT' : `H${i}`;
    const tag = document.createElement('div');
    tag.className = 'layer-tag';
    tag.textContent = label;
    tag.dataset.layer = i;
    tag.addEventListener('click', () => {
      document.querySelectorAll('.layer-tag').forEach(t => t.classList.remove('active'));
      tag.classList.add('active');
      if (renderer) {
        renderer.selectedLayer = parseInt(tag.dataset.layer);
      }
    });
    container.appendChild(tag);
  });
}

function updateActivationDisplay() {
  const container = document.getElementById('activation-display');
  container.innerHTML = '';
  if (!nn.activations.length) return;

  nn.activations.forEach((layer, li) => {
    for (let n = 0; n < Math.min(layer.rows, 8); n++) {
      const val = layer.data[n]?.[0] || 0;
      const norm = (Math.tanh(val) + 1) / 2;
      const div = document.createElement('div');
      div.innerHTML = `
        <div class="node-label">
          <span>L${li}N${n}</span>
          <span style="color:var(--accent)">${val.toFixed(3)}</span>
        </div>
        <div class="activation-bar">
          <div class="activation-fill" style="width:${norm*100}%; background: ${norm > 0.5 ? 'var(--accent)' : 'var(--accent2)'}"></div>
        </div>
      `;
      container.appendChild(div);
    }
  });
}

function log(msg, type = '') {
  const area = document.getElementById('log-area');
  const line = document.createElement('div');
  line.className = `log-line ${type}`;
  const ts = new Date().toLocaleTimeString('vi-VN', {hour12: false});
  line.textContent = `[${ts}] ${msg}`;
  area.appendChild(line);
  area.scrollTop = area.scrollHeight;
  if (area.children.length > 50) area.removeChild(area.firstChild);
}

function trainStep() {
  const lr = getLearningRate();
  nn.lr = lr;

  let totalLoss = 0;
  let correct = 0;

  const stepsPerEpoch = parseInt(document.getElementById('speed-slider').value) * 2;

  for (let s = 0; s < stepsPerEpoch; s++) {
    const sample = dataset[Math.floor(Math.random() * dataset.length)];
    const output = nn.forward(sample.input);
    totalLoss += nn.loss(output, sample.output);
    const pred = output.data[0][0] > 0.5 ? 1 : 0;
    if (pred === sample.output[0]) correct++;
    nn.backward(sample.output);
  }

  epoch += stepsPerEpoch;
  const avgLoss = totalLoss / stepsPerEpoch;
  const acc = (correct / stepsPerEpoch * 100).toFixed(1);

  lossHistory.push(avgLoss);
  if (lossHistory.length > 200) lossHistory.shift();

  const delta = lastLoss !== null ? (avgLoss - lastLoss) : 0;
  lastLoss = avgLoss;

  document.getElementById('epoch-display').textContent = epoch.toLocaleString();
  document.getElementById('loss-display').textContent = avgLoss.toFixed(4);
  document.getElementById('acc-display').textContent = acc + '%';
  document.getElementById('loss-delta').textContent =
    `Δ ${delta >= 0 ? '+' : ''}${delta.toFixed(4)}`;

  if (epoch % 100 < stepsPerEpoch) {
    log(`Epoch ${epoch}: loss=${avgLoss.toFixed(4)} acc=${acc}%`,
      avgLoss < 0.05 ? 'success' : avgLoss < 0.2 ? 'info' : '');
  }

  drawLossChart();
  updateActivationDisplay();

  // Spawn signals
  if (nn.activations.length > 0) {
    const layer = Math.floor(Math.random() * (nn.layerSizes.length - 1));
    const node = Math.floor(Math.random() * nn.layerSizes[layer]);
    renderer.spawnSignal(layer, node);
  }
}

function drawLossChart() {
  const canvas = document.getElementById('loss-chart');
  const ctx = canvas.getContext('2d');
  canvas.width = canvas.offsetWidth * window.devicePixelRatio;
  canvas.height = canvas.offsetHeight * window.devicePixelRatio;
  ctx.scale(window.devicePixelRatio, window.devicePixelRatio);
  const w = canvas.offsetWidth, h = canvas.offsetHeight;
  ctx.clearRect(0, 0, w, h);

  if (lossHistory.length < 2) return;

  const maxL = Math.max(...lossHistory);
  const minL = Math.min(...lossHistory);
  const range = maxL - minL || 0.001;

  ctx.beginPath();
  const step = w / (lossHistory.length - 1);
  lossHistory.forEach((l, i) => {
    const x = i * step;
    const y = h - ((l - minL) / range) * h * 0.85 - 5;
    i === 0 ? ctx.moveTo(x, y) : ctx.lineTo(x, y);
  });
  ctx.strokeStyle = 'rgba(0, 229, 255, 0.8)';
  ctx.lineWidth = 1.5;
  ctx.stroke();

  // Fill under
  ctx.lineTo(w, h); ctx.lineTo(0, h);
  ctx.fillStyle = 'rgba(0, 229, 255, 0.05)';
  ctx.fill();
}

function animate() {
  requestAnimationFrame(animate);
  if (renderer) renderer.render();

  frameCount++;
  const now = performance.now();
  if (now - lastFpsTime >= 1000) {
    fps = frameCount;
    frameCount = 0;
    lastFpsTime = now;
    document.getElementById('fps-counter').textContent = `FPS: ${fps}`;
  }
}

// ============================================================
//   EVENT LISTENERS
// ============================================================

document.getElementById('train-btn').addEventListener('click', () => {
  isTraining = !isTraining;
  const btn = document.getElementById('train-btn');
  if (isTraining) {
    btn.textContent = '⏸ DỪNG TRAINING';
    btn.style.borderColor = 'var(--accent2)';
    btn.style.color = 'var(--accent2)';
    trainInterval = setInterval(trainStep, 16);
    log('Training started...', 'info');
  } else {
    clearInterval(trainInterval);
    btn.textContent = '▶ BẮT ĐẦU TRAINING';
    btn.style.borderColor = '';
    btn.style.color = '';
    log('Training paused.', 'warn');
  }
});

document.getElementById('step-btn').addEventListener('click', () => {
  const sample = dataset[Math.floor(Math.random() * dataset.length)];
  const output = nn.forward(sample.input);
  const lv = nn.loss(output, sample.output);
  updateActivationDisplay();
  for (let l = 0; l < nn.layerSizes.length - 1; l++) {
    for (let n = 0; n < nn.layerSizes[l]; n++) {
      setTimeout(() => renderer.spawnSignal(l, n), l * 100);
    }
  }
  log(`Forward pass: input=[${sample.input}] output=${output.data[0][0].toFixed(3)} loss=${lv.toFixed(4)}`, 'success');
});

document.getElementById('reset-btn').addEventListener('click', () => {
  if (isTraining) {
    isTraining = false;
    clearInterval(trainInterval);
    const btn = document.getElementById('train-btn');
    btn.textContent = '▶ BẮT ĐẦU TRAINING';
    btn.style.borderColor = '';
    btn.style.color = '';
  }
  initNetwork();
});

['layers-slider', 'neurons-slider'].forEach(id => {
  document.getElementById(id).addEventListener('input', function() {
    const valId = id.replace('-slider', '-val');
    document.getElementById(valId).textContent = this.value;
    if (isTraining) { clearInterval(trainInterval); isTraining = false; }
    initNetwork();
    renderer.nn = nn;
  });
});

document.getElementById('lr-slider').addEventListener('input', function() {
  const lr = parseInt(this.value) / 1000;
  document.getElementById('lr-val').textContent = lr.toFixed(3);
  if (nn) nn.lr = lr;
});

document.getElementById('speed-slider').addEventListener('input', function() {
  document.getElementById('speed-val').textContent = this.value;
});

document.getElementById('activation-select').addEventListener('change', function() {
  if (nn) nn.activationName = this.value;
  log(`Activation changed to ${this.value}`, 'info');
});

document.querySelectorAll('.pill').forEach(pill => {
  pill.addEventListener('click', function() {
    document.querySelectorAll('.pill').forEach(p => p.classList.remove('active'));
    this.classList.add('active');
    currentView = this.dataset.view;
    if (renderer) renderer.viewMode = currentView;
    log(`View: ${currentView}`, '');
  });
});

window.addEventListener('resize', () => {
  if (renderer) renderer.resize();
});

// ============================================================
//   INIT
// ============================================================

document.addEventListener('DOMContentLoaded', () => {
  initNetwork();
  const canvas = document.getElementById('network-canvas');
  renderer = new NetworkRenderer(canvas, nn);
  renderer.viewMode = 'network';
  log('NeuralViz initialized. Press ▶ to start.', 'success');
  log('Dataset: 8 XOR samples, 4→?→1 topology', 'info');
  animate();
});
</script>
</body>
</html>
