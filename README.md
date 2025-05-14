<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>MFC Optimizer Pilot</title>
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/@tensorflow/tfjs"></script>
  <script src="/ollama-api/noesis.js"></script>
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
    label, select, input, button, textarea {
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
    .chat-container {
      max-height: 300px;
      overflow-y: auto;
      border: 1px solid #ddd;
      padding: 10px;
      background: #f1f1f1;
    }
    .chat-msg {
      margin-bottom: 10px;
    }
  </style>
</head>
<body>
  <header>MFC Optimizer Pilot</header>
  <div class="tabs">
    <div class="tab active" onclick="switchTab('inputTab')">Inputs</div>
    <div class="tab" onclick="switchTab('resultsTab')">Results</div>
    <div class="tab" onclick="switchTab('graphsTab')">Graphs</div>
    <div class="tab" onclick="switchTab('aiTab')">AI Assistant</div>
  </div>
  <div class="container">
    <div id="inputTab" class="panel active">
      <div class="input-section">
        <label for="inputCE">Coulombic Efficiency(%)</label>
        <input id="inputCE" type="number" value="70" min="0" max="100" />
        <label for="inputCOD">COD Removal(%)</label>
        <input id="inputCOD" type="number" value="70" min="0" max="100" />
        <label for="inputMicrobe">Microbe Type <span class="tooltip">(e.g.Geobacter)</span></label>
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
        <label for="inputVoltage">Cell Voltage(V)</label>
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
    <div id="resultsTab" class="panel">
      <h2>Simulation Results</h2>
      <div id="numericOutput"></div>
    </div>
    <div id="graphsTab" class="panel">
      <h2>Graphs</h2>
      <canvas id="chartPower"></canvas>
      <canvas id="chartVoltage"></canvas>
      <canvas id="chartResistance"></canvas>
      <canvas id="chartCurrent"></canvas>
      <canvas id="chartGain"></canvas>
      <button onclick="exportMFCData()">Export Simulation Data (CSV)</button>
    </div>
    <div id="aiTab" class="panel">
      <h2>AI Optimization Assistant</h2>
      <div class="chat-container" id="chatBox"></div>
      <textarea id="aiInput" placeholder="Ask the optimizer anything..." rows="3"></textarea>
      <button onclick="sendAIQuery()">Send</button>
    </div>
  </div>
<script>
function renderCharts(hours, basePower, baseVoltage, baseResistance) {
  const labels = Array.from({ length: hours }, (_, i) => i + 1);
  const superCapVoltage = labels.map(t => parseFloat((9.3 * (1 - Math.exp(-t / 35))).toFixed(2)));
  const mfcVoltage = labels.map(t => parseFloat((0.6 + 0.02 * Math.sin(t / 15)).toFixed(3)));
  const currentMFC = labels.map(t => parseFloat((14 - 10 * Math.exp(-t / 25)).toFixed(2)));
  const lvbmOutputCurrent = labels.map(t => parseFloat((2.1 + 0.3 * Math.sin(t / 10)).toFixed(2)));
  const boostedVoltage = labels.map(t => (t < 30 ? 17 : t < 60 ? 35 : 99));
  const boostedGain = boostedVoltage.map((v, i) => parseFloat((v / mfcVoltage[i]).toFixed(1)));
  const power = boostedVoltage.map((v, i) => parseFloat(((v * lvbmOutputCurrent[i]) / 1000).toFixed(3)));

  chartPower?.destroy();
  chartVoltage?.destroy();
  chartResistance?.destroy();
  chartCurrent?.destroy();
  chartGain?.destroy();

  const createChart = (id, label, data, color, yLabel) => new Chart(document.getElementById(id), {
    type: 'line',
    data: {
      labels,
      datasets: [{ label, data, borderColor: color, fill: false, borderWidth: 2 }]
    },
    options: {
      responsive: true,
      scales: {
        x: { title: { display: true, text: 'Time (h)' } },
        y: { beginAtZero: true, title: { display: true, text: yLabel } }
      },
      plugins: {
        legend: { position: 'top' },
        tooltip: { mode: 'index', intersect: false }
      }
    }
  });

  chartPower = createChart('chartPower', 'Power Output (W)', power, 'green', 'Power (W)');
  chartVoltage = createChart('chartVoltage', 'MFC Voltage (V)', mfcVoltage, 'red', 'Voltage (V)');
  chartResistance = createChart('chartResistance', 'Voltage Gain (V/V)', boostedGain, 'blue', 'Gain (V/V)');
  chartCurrent = createChart('chartCurrent', 'Output Current (mA)', lvbmOutputCurrent, 'orange', 'Current (mA)');
  chartGain = createChart('chartGain', 'Supercapacitor Voltage (V)', superCapVoltage, 'purple', 'Voltage (V)');

  const exportData = labels.map((t, i) => ({
    hour: t,
    mfcVoltage: mfcVoltage[i],
    lvbmOutputCurrent: lvbmOutputCurrent[i],
    superCapVoltage: superCapVoltage[i],
    boostedVoltage: boostedVoltage[i],
    power: power[i]
  }));

  console.log('---Microcontroller UART Log---');
  exportData.slice(-5).forEach(row => {
    console.log(`Time ${row.hour}h: V=${row.mfcVoltage}V, I=${row.lvbmOutputCurrent}mA, Cap=${row.superCapVoltage}V`);
  });

  if (boostedVoltage[boostedVoltage.length - 1] > 90) {
    console.log('Actuator: High-voltage relay ENGAGED');
  }
  if (lvbmOutputCurrent[lvbmOutputCurrent.length - 1] < 2) {
    console.log('Actuator: Throttle output to preserve power');
  }

  window.exportMFCData = () => {
    let csv = 'Time(h),MFC Voltage (V),LVBM Current (mA),Supercap Voltage (V),Boosted Voltage (V),Power (W)\n';
    exportData.forEach(row => {
      csv += `${row.hour},${row.mfcVoltage},${row.lvbmOutputCurrent},${row.superCapVoltage},${row.boostedVoltage},${row.power}\n`;
    });
    const blob = new Blob([csv], { type: 'text/csv' });
    const link = document.createElement('a');
    link.href = URL.createObjectURL(blob);
    link.download = 'mfc_simulation_data.csv';
    link.click();
  };
}
</script>
</body>
</html>
