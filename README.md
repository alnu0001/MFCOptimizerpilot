<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>MFC Optimizer Pilot</title>
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
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
      box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
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
  <header>MFC Optimizer Pilot (LLM-Enhanced)</header>
  <div class="tabs">
    <div class="tab active" onclick="switchTab('inputTab', this)">Inputs</div>
    <div class="tab" onclick="switchTab('resultsTab', this)">Results</div>
  </div>
  <div class="container">
    <div id="inputTab" class="panel active">
      <div class="input-section">
        <label for="inputCE">Coulombic Efficiency (%)</label>
        <input id="inputCE" type="number" value="70" min="0" max="100" />

        <label for="inputCOD">COD Removal (%)</label>
        <input id="inputCOD" type="number" value="70" min="0" max="100" />

        <button onclick="simulateMFC()">Simulate</button>
        <button onclick="loadPrevious()">Load Previous</button>
        <button onclick="clearHistory()">Clear History</button>
        <button onclick="compareWithPrevious()">Compare with Previous</button>
      </div>
    </div>
    <div id="resultsTab" class="panel">
      <h2>Simulation Results</h2>
      <div id="numericOutput"></div>
    </div>
  </div>

  <script>
    function switchTab(tabId, element) {
      document.querySelectorAll('.tab').forEach(tab => tab.classList.remove('active'));
      document.querySelectorAll('.panel').forEach(panel => panel.classList.remove('active'));
      element.classList.add('active');
      document.getElementById(tabId).classList.add('active');
    }

    function simulateMFC() {
      const CE = parseFloat(document.getElementById('inputCE').value);
      const COD = parseFloat(document.getElementById('inputCOD').value);
      const E = 0.4;
      const COD_input = 20;
      const e_per_g_COD = 0.00834;
      const Faraday = 96485;

      const COD_removed = COD_input * (COD / 100);
      const mol_e = COD_removed * e_per_g_COD;
      const total_charge = mol_e * Faraday * (CE / 100);
      const power = (total_charge * E / 3600).toFixed(3);

      const output = `<strong>Power Output:</strong> ${power} W/m²`;
      document.getElementById("numericOutput").innerHTML = output;

      const entry = {
        CE, COD, power: parseFloat(power), timestamp: new Date().toISOString()
      };
      const history = JSON.parse(localStorage.getItem('mfcHistory') || '[]');
      history.push(entry);
      localStorage.setItem('mfcHistory', JSON.stringify(history));
    }

    function loadPrevious() {
      const history = JSON.parse(localStorage.getItem('mfcHistory') || '[]');
      if (history.length === 0) return alert('No history found.');
      const prev = history[history.length - 1];
      document.getElementById('inputCE').value = prev.CE;
      document.getElementById('inputCOD').value = prev.COD;
      alert('Previous simulation loaded.');
    }

    function clearHistory() {
      localStorage.removeItem('mfcHistory');
      alert('History cleared.');
      document.getElementById("numericOutput").innerHTML = '';
    }

    function compareWithPrevious() {
      const history = JSON.parse(localStorage.getItem('mfcHistory') || '[]');
      if (history.length < 2) return alert('Need at least two entries to compare.');
      const last = history[history.length - 1];
      const secondLast = history[history.length - 2];
      const delta = (last.power - secondLast.power).toFixed(3);
      alert(`Latest Power: ${last.power} W/m²\nPrevious: ${secondLast.power} W/m²\nChange: ${delta} W/m²`);
    }
  </script>
</body>
</html>
