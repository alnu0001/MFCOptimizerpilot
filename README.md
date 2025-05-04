<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>MFC Optimizer Pilot (ML-Enhanced)</title>
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/@tensorflow/tfjs"></script>
  <style>
    :root {
      --bg-color: #ffffff;
      --panel-bg: #f1f1f1;
      --text-color: #222222;
      --accent: #e2e8f0;
      --highlight: #3182ce;
      --input-bg: #ffffff;
      --border-color: #ccc;
    }

    body.dark-mode {
      --bg-color: #121212;
      --panel-bg: #1e1e1e;
      --text-color: #e0e0e0;
      --accent: #2d3748;
      --highlight: #4fd1c5;
      --input-bg: #2a2a2a;
      --border-color: #333;
    }

    body {
      font-family: 'Segoe UI', sans-serif;
      margin: 0;
      padding: 0;
      background-color: var(--bg-color);
      color: var(--text-color);
      transition: all 0.3s ease;
    }
    header {
      background-color: var(--accent);
      color: black;
      padding: 15px;
      text-align: center;
      font-size: 24px;
      position: relative;
    }
    .theme-toggle {
      position: absolute;
      right: 20px;
      top: 15px;
      background: var(--highlight);
      color: black;
      border: none;
      padding: 5px 10px;
      border-radius: 5px;
      cursor: pointer;
      font-weight: bold;
    }
    .tabs {
      display: flex;
      justify-content: center;
      margin-top: 10px;
    }
    .tab {
      margin: 0 10px;
      padding: 10px 20px;
      cursor: pointer;
      background: #2d2d2d;
      color: #aaa;
      border-radius: 5px;
      border: 1px solid var(--border-color);
    }
    .tab.active {
      background: var(--highlight);
      color: black;
    }
    .container {
      padding: 20px;
      max-width: 1200px;
      margin: auto;
    }
    .panel {
      display: none;
      background: var(--panel-bg);
      padding: 20px;
      border-radius: 8px;
      box-shadow: 0 0 10px rgba(0,0,0,0.5);
    }
    .panel.active {
      display: block;
    }
    label, select, input, button {
      display: block;
      margin-top: 10px;
      width: 100%;
    }
    input, select {
      background-color: var(--input-bg);
      border: 1px solid var(--border-color);
      color: var(--text-color);
      padding: 8px;
      border-radius: 4px;
    }
    button {
      background-color: var(--highlight);
      color: black;
      border: none;
      padding: 10px;
      border-radius: 5px;
      font-weight: bold;
      cursor: pointer;
    }
    button:hover {
      opacity: 0.9;
    }
    canvas {
      margin-top: 20px;
      max-width: 100%;
    }
    .input-section {
      margin-bottom: 20px;
    }
    .tooltip {
      font-size: 12px;
      color: gray;
    }
  </style>
</head>
<body class="light-mode">
  <header>
    MFC Optimizer Pilot (ML-Enhanced)
    <button class="theme-toggle" onclick="toggleTheme()">Night Mode</button>
  </header>

  <div class="container">
    <div class="input-section">
      <label for="ce">Coulombic Efficiency (%)</label>
      <input id="ce" type="number" value="75" />

      <label for="cod">COD Removal (%)</label>
      <input id="cod" type="number" value="80" />

      <label for="microbe">Microbe (encoded 0–1)</label>
      <input id="microbe" type="number" value="0.5" step="0.01" />

      <label for="substrate">Substrate (encoded 0–1)</label>
      <input id="substrate" type="number" value="0.5" step="0.01" />

      <label for="enzyme">Enzyme (encoded 0–1)</label>
      <input id="enzyme" type="number" value="0.5" step="0.01" />

      <button onclick="runPrediction()">Simulate</button>
      <button onclick="loadPrevious()">Load Previous</button>
      <button onclick="resetInputs()">Reset</button>
    </div>
    <div id="results"></div>
  </div>

  <script>
    function applyTheme() {
      const savedTheme = localStorage.getItem('theme') || 'light';
      document.body.classList.toggle('dark-mode', savedTheme === 'dark');
    }

    function toggleTheme() {
      const isDark = document.body.classList.toggle('dark-mode');
      localStorage.setItem('theme', isDark ? 'dark' : 'light');
    }

    window.addEventListener('DOMContentLoaded', applyTheme);

    let model;
    async function loadModel() {
      model = await tf.loadLayersModel('model/model.json');
      console.log("ML model loaded.");
    }

    async function runPrediction() {
      if (!model) {
        alert("Model not loaded yet.");
        return;
      }
      const ce = parseFloat(document.getElementById('ce').value) / 100;
      const cod = parseFloat(document.getElementById('cod').value) / 100;
      const microbe = parseFloat(document.getElementById('microbe').value);
      const substrate = parseFloat(document.getElementById('substrate').value);
      const enzyme = parseFloat(document.getElementById('enzyme').value);

      const input = tf.tensor2d([[ce, cod, microbe, substrate, enzyme]]);
      const prediction = model.predict(input);
      const result = await prediction.array();
      const [voltage, power, resistance] = result[0];

      document.getElementById('results').innerHTML =
        `<strong>Predicted Voltage:</strong> ${voltage.toFixed(3)} V<br>
         <strong>Predicted Power:</strong> ${power.toFixed(3)} W/m²<br>
         <strong>Predicted Resistance:</strong> ${resistance.toFixed(2)} Ω`;
    }

    function loadPrevious() {
      alert("This feature will be updated in next release.");
    }

    function resetInputs() {
      document.getElementById('ce').value = 75;
      document.getElementById('cod').value = 80;
      document.getElementById('microbe').value = 0.5;
      document.getElementById('substrate').value = 0.5;
      document.getElementById('enzyme').value = 0.5;
      document.getElementById('results').innerHTML = '';
    }

    loadModel();
  </script>
</body>
</html>
