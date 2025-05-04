<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>MFC Optimizer Pilot (ML-Enhanced)</title>
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/@tensorflow/tfjs"></script>
  <style>
    body {
      font-family: 'Segoe UI', sans-serif;
      margin: 0;
      padding: 0;
      background-color: #f9f9f9;
      color: #333;
    }
    header {
      background-color: #1a202c;
      color: white;
      padding: 15px;
      text-align: center;
      font-size: 24px;
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
      background: #e2e8f0;
      border-radius: 5px;
    }
    .tab.active {
      background: #4a5568;
      color: white;
    }
    .container {
      padding: 20px;
      max-width: 1200px;
      margin: auto;
    }
    .panel {
      display: none;
      background: white;
      padding: 20px;
      border-radius: 8px;
      box-shadow: 0 0 10px rgba(0,0,0,0.1);
    }
    .panel.active {
      display: block;
    }
    label, select, input, button {
      display: block;
      margin-top: 10px;
      width: 100%;
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
<body>
  <header>MFC Optimizer Pilot (ML-Enhanced)</header>

  <div class="tabs">
    <div class="tab active" onclick="switchTab('inputTab')">Inputs</div>
    <div class="tab" onclick="switchTab('resultsTab')">Results</div>
    <div class="tab" onclick="switchTab('graphsTab')">Graphs</div>
  </div>

  <div class="container">
    <!-- Input Tab -->
    <div id="inputTab" class="panel active">
      <div class="input-section">
        <label for="inputCE">Coulombic Efficiency (%)</label>
        <input id="inputCE" type="number" value="70" min="0" max="100" />

        <label for="inputCOD">COD Removal (%)</label>
        <input id="inputCOD" type="number" value="70" min="0" max="100" />

        <label for="inputMicrobe">Microbe Type <span class="tooltip">(e.g. Geobacter)</span></label>
        <input list="microbes" id="inputMicrobe" />
        <datalist id="microbes">
          <option value="Geobacter" />
          <option value="Shewanella" />
          <option value="Pseudomonas aeruginosa" />
          <option value="Yeast" />
          <option value="Best option" />
        </datalist>

        <label for="inputSubstrate">Substrate Composition</label>
        <input list="substrates" id="inputSubstrate" />
        <datalist id="substrates">
          <option value="Starch" />
          <option value="Molasses" />
          <option value="Acetate" />
          <option value="Best option" />
        </datalist>

        <label for="inputEnzyme">Enzyme Type</label>
        <input id="inputEnzyme" type="text" value="mtrC" />

        <label for="inputVoltage">Cell Voltage (V)</label>
        <input id="inputVoltage" type="number" value="0.4" step="0.01" min="0" />

        <label for="inputTimeScale">Simulation Duration</label>
        <select id="inputTimeScale">
          <option value="24">24 Hours</option>
          <option value="168">7 Days</option>
          <option value="720">30 Days</option>
        </select>

        <button onclick="simulateMFC()">Simulate</button>
        <button onclick="loadPrevious()">Load Previous</button>
        <button onclick="resetInputs()">Reset</button>
      </div>
    </div>

    <!-- Results Tab -->
    <div id="resultsTab" class="panel">
      <h2>Simulation Results</h2>
      <div id="numericOutput"></div>
    </div>

    <!-- Graphs Tab -->
    <div id="graphsTab" class="panel">
      <h2>Graphs</h2>
      <canvas id="chartPower"></canvas>
      <canvas id="chartVoltage"></canvas>
      <canvas id="chartResistance"></canvas>
    </div>
  </div>

  <script>
    let model;
    tf.loadLayersModel('model/model.json').then(m => model = m);

    const chartIDs = ['chartPower', 'chartVoltage', 'chartResistance'];
    let charts = [];

    const encode = (val, list) => list.findIndex(opt => opt.toLowerCase() === val.toLowerCase()) / list.length || 0;
    const microbes = ['Geobacter', 'Shewanella', 'Pseudomonas aeruginosa', 'Yeast'];
    const substrates = ['Starch', 'Molasses', 'Acetate'];

    function switchTab(id) {
      document.querySelectorAll('.panel').forEach(p => p.classList.remove('active'));
      document.querySelectorAll('.tab').forEach(t => t.classList.remove('active'));
      document.getElementById(id).classList.add('active');
      document.querySelector(`.tab[onclick*='${id}']`).classList.add('active');
    }

    async function simulateMFC() {
      const CE = parseFloat(document.getElementById('inputCE').value);
      const COD = parseFloat(document.getElementById('inputCOD').value);
      const E = parseFloat(document.getElementById('inputVoltage').value);
      const hours = parseInt(document.getElementById('inputTimeScale').value);
      const microbe = encode(document.getElementById('inputMicrobe').value || 'Geobacter', microbes);
      const substrate = encode(document.getElementById('inputSubstrate').value || 'Starch', substrates);
      const enzyme = document.getElementById('inputEnzyme').value.length / 10;

      const input = tf.tensor2d([[CE / 100, COD / 100, E, microbe, substrate, enzyme]]);
      const output = await model.predict(input).array();
      const [power, voltage_drop, internal_resistance] = output[0];

      document.getElementById("numericOutput").innerHTML =
        `<strong>Power Output:</strong> ${power.toFixed(3)} W/m²<br>
         <strong>Voltage Drop:</strong> ${voltage_drop.toFixed(3)} V<br>
         <strong>Internal Resistance:</strong> ${internal_resistance.toFixed(2)} Ω`;

      localStorage.setItem('lastSim', JSON.stringify({ CE, COD, E, hours, microbe, substrate, enzyme, power, voltage_drop, internal_resistance }));
      renderCharts(hours, power, voltage_drop, internal_resistance);
      switchTab('graphsTab');
    }

    function loadPrevious() {
      const data = JSON.parse(localStorage.getItem('lastSim'));
      if (!data) return alert("No previous data.");
      document.getElementById('inputCE').value = data.CE;
      document.getElementById('inputCOD').value = data.COD;
      document.getElementById('inputVoltage').value = data.E;
      document.getElementById('inputTimeScale').value = data.hours;
      document.getElementById("numericOutput").innerHTML =
        `<strong>Power Output:</strong> ${data.power.toFixed(3)} W/m²<br>
         <strong>Voltage Drop:</strong> ${data.voltage_drop.toFixed(3)} V<br>
         <strong>Internal Resistance:</strong> ${data.internal_resistance.toFixed(2)} Ω`;
      renderCharts(data.hours, data.power, data.voltage_drop, data.internal_resistance);
      switchTab('graphsTab');
    }

    function resetInputs() {
      document.getElementById('inputCE').value = 70;
      document.getElementById('inputCOD').value = 70;
      document.getElementById('inputMicrobe').value = '';
      document.getElementById('inputSubstrate').value = '';
      document.getElementById('inputEnzyme').value = 'mtrC';
      document.getElementById('inputVoltage').value = 0.4;
      document.getElementById('inputTimeScale').value = 24;
      document.getElementById('numericOutput').innerHTML = '';
      charts.forEach(c => c.destroy());
      charts = [];
    }

    function renderCharts(hours, power, voltage, resistance) {
      const labels = Array.from({ length: hours }, (_, i) => i + 1);
      const powers = labels.map(i => power + Math.sin(i / 10) * 0.01);
      const voltages = labels.map(i => voltage + Math.cos(i / 20) * 0.01);
      const resistances = labels.map(i => resistance + Math.sin(i / 15) * 0.01);

      const ctxs = chartIDs.map(id => document.getElementById(id).getContext('2d'));
      const dataSets = [
        { label: "Power Output (W/m²)", data: powers, color: "green" },
        { label: "Voltage Drop (V)", data: voltages, color: "red" },
        { label: "Internal Resistance (Ω)", data: resistances, color: "blue" }
      ];
      charts.forEach(c => c.destroy());
      charts = dataSets.map((d, i) => new Chart(ctxs[i], {
        type: 'line',
        data: { labels, datasets: [{ label: d.label, data: d.data, borderColor: d.color, fill: false }] },
        options: { responsive: true, scales: { x: { title: { display: true, text: 'Time (h)' } } } }
      }));
    }
  </script>
</body>
</html>

