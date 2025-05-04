<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>MFC Simulator</title>
  <link href="https://fonts.googleapis.com/css2?family=Press+Start+2P&display=swap" rel="stylesheet">
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/chartjs-chart-matrix"></script>
  <style>
    body {
      font-family: 'Press Start 2P', cursive;
      background-color: #f4f4f4;
      color: #111;
      margin: 0;
      padding: 20px;
    }
    h1, h2 {
      text-align: center;
    }
    .container {
      display: flex;
      flex-wrap: wrap;
      justify-content: center;
      gap: 20px;
    }
    .panel {
      background: white;
      padding: 20px;
      border-radius: 8px;
      box-shadow: 0 0 10px rgba(0,0,0,0.1);
      width: 360px;
    }
    label {
      display: block;
      margin-top: 10px;
    }
    input, select, button {
      width: 100%;
      padding: 8px;
      margin-top: 5px;
      border: 1px solid #ccc;
      border-radius: 4px;
      font-family: monospace;
    }
    canvas {
      max-width: 100%;
      margin-top: 15px;
    }
    button {
      background-color: #4CAF50;
      color: white;
      font-weight: bold;
      cursor: pointer;
    }
  </style>
</head>
<body>
  <h1>MFC Simulator</h1>
  <div class="container">
    <div class="panel">
      <h2>Input Parameters</h2>
      <label>Coulombic Efficiency (%)</label>
      <input type="number" id="ce" value="70" min="0" max="100">
      <label>COD Removal (%)</label>
      <input type="number" id="cod" value="70" min="0" max="100">
      <label>Microbe</label>
      <select id="microbe">
        <option>Shewanella oneidensis</option>
        <option>Pseudomonas aeruginosa</option>
        <option>Bacillus piscis</option>
        <option>Halophilic consortia</option>
        <option>Saccharomyces cerevisiae</option>
        <option>Acetivibrio thermocellus</option>
        <option>Geobacter sulfurreducens</option>
      </select>
      <label>Substrate</label>
      <select id="substrate">
        <option>acetate</option>
        <option>molasses wastewater</option>
        <option>slaughterhouse waste</option>
        <option>brewery effluent</option>
        <option>lignocellulose</option>
      </select>
      <label>Electrode</label>
      <select id="electrode">
        <option>carbon cloth</option>
        <option>PANI nanofiber</option>
        <option>3D-printed gyroid</option>
        <option>Fe3O4-PANI composite</option>
        <option>Ni-silicide nanowires</option>
      </select>
      <label>Enzyme</label>
      <input type="text" id="enzyme" placeholder="e.g. mtrC">
      <label>Cell Voltage (V)</label>
      <input type="number" step="0.01" id="voltage" value="0.4">
      <label>Duration (hours)</label>
      <input type="number" id="duration" value="24" min="1" max="168">
      <button onclick="simulate()">Run Simulation</button>
      <button onclick="saveInputs()">💾 Save</button>
      <button onclick="loadInputs()">📂 Load</button>
    </div>
    <div class="panel">
      <h2>Results</h2>
      <canvas id="powerChart"></canvas>
      <canvas id="voltageChart"></canvas>
      <canvas id="currentChart"></canvas>
      <canvas id="resistanceChart"></canvas>
      <canvas id="pHChart"></canvas>
    </div>
    <div class="panel">
      <h2>CE × COD Heatmap</h2>
      <canvas id="heatmapChart" width="350" height="350"></canvas>
    </div>
  </div>

<script>
const FARADAY = 96485;
const COD_input = 20;
const e_per_g_COD = 0.00834;
const I0 = {
  "Shewanella oneidensis": 3.0,
  "Pseudomonas aeruginosa": 0.2,
  "Bacillus piscis": 0.12,
  "Halophilic consortia": 0.56,
  "Saccharomyces cerevisiae": 0.10,
  "Acetivibrio thermocellus": 0.16,
  "Geobacter sulfurreducens": 1.0
};

function simulate() {
  const CE = parseFloat(document.getElementById("ce").value) / 100;
  const COD = parseFloat(document.getElementById("cod").value) / 100;
  const microbe = document.getElementById("microbe").value;
  const voltage = parseFloat(document.getElementById("voltage").value);
  const enzyme = document.getElementById("enzyme").value.toLowerCase();
  const hours = parseInt(document.getElementById("duration").value);

  let I_base = I0[microbe] || 0.1;
  if (enzyme === 'mtrc') I_base *= 1.2;

  const time = [], power = [], current = [], voltage_arr = [], resistance = [], pH = [];
  for (let t = 0; t <= hours; t++) {
    const frac = 1 - (1 - COD) * (t / hours);
    const I = I_base * frac * CE;
    const V = voltage * frac;
    const P = I * V;
    const R = I > 0 ? (V / I) : null;
    const pH_val = 7 - 2 * (1 - frac);

    time.push(t);
    power.push(P.toFixed(3));
    current.push(I.toFixed(3));
    voltage_arr.push(V.toFixed(3));
    resistance.push(R ? R.toFixed(2) : null);
    pH.push(pH_val.toFixed(2));
  }

  chartLine("powerChart", "Power (W/m²)", time, power);
  chartLine("voltageChart", "Voltage (V)", time, voltage_arr);
  chartLine("currentChart", "Current (A)", time, current);
  chartLine("resistanceChart", "Internal Resistance (Ω)", time, resistance);
  chartLine("pHChart", "Anode pH", time, pH);
  heatmap();
}

function chartLine(id, label, x, y) {
  new Chart(document.getElementById(id), {
    type: 'line',
    data: {
      labels: x,
      datasets: [{
        label,
        data: y,
        borderColor: 'blue',
        fill: false
      }]
    },
    options: {
      responsive: true,
      scales: {
        x: { title: { display: true, text: 'Time (h)' } },
        y: { title: { display: true, text: label }, beginAtZero: true }
      }
    }
  });
}

function heatmap() {
  const ctx = document.getElementById('heatmapChart').getContext('2d');
  const data = [];
  for (let ce = 10; ce <= 100; ce += 10) {
    for (let cod = 10; cod <= 100; cod += 10) {
      const COD_removed = COD_input * (cod / 100);
      const CE_fraction = ce / 100;
      const P = COD_removed * e_per_g_COD * FARADAY * CE_fraction * (0.4 / 86400);
      data.push({x: ce, y: cod, v: parseFloat(P.toFixed(2))});
    }
  }

  new Chart(ctx, {
    type: 'matrix',
    data: {
      datasets: [{
        label: "Power (W/m²)",
        data,
        backgroundColor(ctx) {
          const value = ctx.dataset.data[ctx.dataIndex].v;
          const alpha = Math.min(1, value / 0.1);
          return `rgba(0, 128, 255, ${alpha})`;
        },
        width: () => 30,
        height: () => 30
      }]
    },
    options: {
      scales: {
        x: { title: { display: true, text: 'CE (%)' }, min: 0, max: 110 },
        y: { title: { display: true, text: 'COD (%)' }, min: 0, max: 110 }
      },
      plugins: {
        tooltip: {
          callbacks: {
            label: ctx => `Power: ${ctx.raw.v} W/m²`
          }
        },
        title: {
          display: true,
          text: 'Power Output Heatmap'
        }
      }
    }
  });
}

function saveInputs() {
  const inputs = {
    ce: document.getElementById("ce").value,
    cod: document.getElementById("cod").value,
    microbe: document.getElementById("microbe").value,
    substrate: document.getElementById("substrate").value,
    electrode: document.getElementById("electrode").value,
    enzyme: document.getElementById("enzyme").value,
    voltage: document.getElementById("voltage").value,
    duration: document.getElementById("duration").value
  };
  localStorage.setItem("mfc_inputs", JSON.stringify(inputs));
  alert("Saved!");
}

function loadInputs() {
  const inputs = JSON.parse(localStorage.getItem("mfc_inputs"));
  if (!inputs) return alert("No saved inputs.");
  for (const key in inputs) document.getElementById(key).value = inputs[key];
  simulate();
}
</script>
</body>
</html>
