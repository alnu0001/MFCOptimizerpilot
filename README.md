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
      transition: background-color 0.3s, color 0.3s;
    }
    body.dark {
      background-color: #1a202c;
      color: #f1f1f1;
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
      flex-wrap: wrap;
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
      color: #000;
      padding: 20px;
      border-radius: 8px;
      box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
    }
    body.dark .panel {
      background: #2d3748;
      color: #f1f1f1;
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
  </style>
</head>
<body>
  <header>MFC Optimizer Pilot (LLM-Enhanced)</header>
  <div class="tabs">
    <div class="tab active" onclick="switchTab('inputTab', this)">Inputs</div>
    <div class="tab" onclick="switchTab('resultsTab', this)">Results</div>
    <div class="tab" onclick="switchTab('graphsTab', this)">Graphs</div>
    <div class="tab" onclick="toggleDarkMode()">Dark Mode</div>
  </div>
  <div class="container">
    <div id="inputTab" class="panel active">
      <div class="input-section">
        <label for="inputTime">Time (s)</label>
        <input id="inputTime" type="number" value="3600" min="0" />

        <label for="inputSubstrate">Substrate Concentration (mg/L)</label>
        <input id="inputSubstrate" type="number" value="500" min="0" />

        <label for="inputRint">Internal Resistance (Ω)</label>
        <input id="inputRint" type="number" value="50" min="0" />

        <label for="inputRext">External Resistance (Ω)</label>
        <input id="inputRext" type="number" value="100" min="0" />

        <label for="inputTemp">Temperature (°C)</label>
        <input id="inputTemp" type="number" value="30" min="0" />

        <label for="inputPH">pH Level</label>
        <input id="inputPH" type="number" step="0.1" value="7.0" min="0" max="14" />

        <label for="inputArea">Electrode Surface Area (cm²)</label>
        <input id="inputArea" type="number" value="25" min="0" />

        <label for="inputDistance">Distance Between Electrodes (cm)</label>
        <input id="inputDistance" type="number" value="2" min="0" />

        <label for="inputMicrobe">Microbe Type</label>
        <input list="microbes" id="inputMicrobe" />
        <datalist id="microbes">
          <option value="Geobacter" />
          <option value="Shewanella" />
          <option value="Pseudomonas aeruginosa" />
          <option value="Desulfuromonas" />
        </datalist>

        <label for="inputEnzyme">Enzyme Type</label>
        <input id="inputEnzyme" type="text" value="OmcZ" />

        <label for="inputVoltage">Voltage (V)</label>
        <input id="inputVoltage" type="number" value="0.4" step="0.01" min="0" />

        <label for="inputCurrent">Current (A)</label>
        <input id="inputCurrent" type="number" value="0.01" step="0.001" min="0" />

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
    <div id="graphsTab" class="panel">
      <h2>Simulation Graphs</h2>
      <canvas id="chartPower"></canvas>
      <canvas id="chartVoltage"></canvas>
      <canvas id="chartResistance"></canvas>
      <canvas id="chartTotalResistance"></canvas>
      <canvas id="chartVoltageCurrent"></canvas>
    </div>
  </div>

  <script>
    function switchTab(tabId, element) {
      document.querySelectorAll('.tab').forEach(tab => tab.classList.remove('active'));
      document.querySelectorAll('.panel').forEach(panel => panel.classList.remove('active'));
      element.classList.add('active');
      document.getElementById(tabId).classList.add('active');
    }

    function toggleDarkMode() {
      document.body.classList.toggle('dark');
    }
  </script>
</body>
</html>
