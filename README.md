<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>MFC Optimizer Pilot (LLM-Enhanced)</title>
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
  <header>MFC Optimizer Pilot (LLM-Enhanced)</header>

  <div class="tabs">
    <div class="tab active" onclick="switchTab('inputTab')">Inputs</div>
    <div class="tab" onclick="switchTab('resultsTab')">Results</div>
    <div class="tab" onclick="switchTab('graphsTab')">Graphs</div>
    <div class="tab" onclick="switchTab('aiTab')">AI Assistant</div>
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

    <!-- AI Assistant Tab -->
    <div id="aiTab" class="panel">
      <h2>AI Optimization Assistant</h2>
      <div class="chat-container" id="chatBox"></div>
      <textarea id="aiInput" placeholder="Ask the optimizer anything..." rows="3"></textarea>
      <button onclick="sendAIQuery()">Send</button>
    </div>
  </div>

  <script>
    function switchTab(tabId) {
      document.querySelectorAll('.tab').forEach(tab => tab.classList.remove('active'));
      document.querySelectorAll('.panel').forEach(panel => panel.classList.remove('active'));
      document.querySelector(`.tab[onclick*="${tabId}"]`).classList.add('active');
      document.getElementById(tabId).classList.add('active');
    }

    function simulateMFC() {
      const CE = parseFloat(document.getElementById('inputCE').value);
      const COD = parseFloat(document.getElementById('inputCOD').value);
      const E = parseFloat(document.getElementById('inputVoltage').value);
      const hours = parseInt(document.getElementById('inputTimeScale').value);
      const microbe = document.getElementById('inputMicrobe').value;
      const substrate = document.getElementById('inputSubstrate').value;
      const enzyme = document.getElementById('inputEnzyme').value;

      const COD_input = 20;
      const e_per_g_COD = 0.00834;
      const Faraday = 96485;
      const COD_removed = COD_input * (COD / 100);
      const mol_e = COD_removed * e_per_g_COD;
      const total_charge = mol_e * Faraday * (CE / 100);
      const power = (total_charge * E / (3600 * hours)).toFixed(3);
      const voltage_drop = (E * (1 - CE / 100)).toFixed(3);
      const internal_resistance = ((E - voltage_drop) / (total_charge / (3600 * hours))).toFixed(2);

      document.getElementById("numericOutput").innerHTML =
        `<strong>Power Output:</strong> ${power} W/m²<br>
         <strong>Voltage Drop:</strong> ${voltage_drop} V<br>
         <strong>Internal Resistance:</strong> ${internal_resistance} Ω<br>
         <strong>Microbe:</strong> ${microbe}, <strong>Substrate:</strong> ${substrate}, <strong>Enzyme:</strong> ${enzyme}`;

      localStorage.setItem('lastSim', JSON.stringify({ CE, COD, E, hours, microbe, substrate, enzyme, power, voltage_drop, internal_resistance }));
      renderCharts(hours, power, voltage_drop, internal_resistance);
      switchTab('resultsTab');
    }

    function loadPrevious() {
      const data = JSON.parse(localStorage.getItem('lastSim'));
      if (!data) return alert("No previous data.");
      document.getElementById('inputCE').value = data.CE;
      document.getElementById('inputCOD').value = data.COD;
      document.getElementById('inputVoltage').value = data.E;
      document.getElementById('inputTimeScale').value = data.hours;
      document.getElementById('inputMicrobe').value = data.microbe;
      document.getElementById('inputSubstrate').value = data.substrate;
      document.getElementById('inputEnzyme').value = data.enzyme;
      simulateMFC();
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
      chartPower?.destroy();
      chartVoltage?.destroy();
      chartResistance?.destroy();
    }

    let chartPower, chartVoltage, chartResistance;
    function renderCharts(hours, power, voltage_drop, resistance) {
      const labels = Array.from({ length: hours }, (_, i) => i + 1);
      const powers = labels.map(() => parseFloat(power));
      const voltages = labels.map(() => parseFloat(voltage_drop));
      const resistances = labels.map(() => parseFloat(resistance));

      chartPower?.destroy();
      chartVoltage?.destroy();
      chartResistance?.destroy();

      const config = (label, data, color) => ({
        type: 'line',
        data: {
          labels,
          datasets: [{ label, data, borderColor: color, fill: false }]
        },
        options: {
          responsive: true,
          scales: {
            x: { title: { display: true, text: 'Time (h)' } },
            y: { beginAtZero: true }
          }
        }
      });

      chartPower = new Chart(document.getElementById('chartPower'), config("Power Output (W/m²)", powers, 'green'));
      chartVoltage = new Chart(document.getElementById('chartVoltage'), config("Voltage Drop (V)", voltages, 'red'));
      chartResistance = new Chart(document.getElementById('chartResistance'), config("Internal Resistance (Ω)", resistances, 'blue'));
    }

    const chatBox = document.getElementById('chatBox');
    async function sendAIQuery() {
      const input = document.getElementById('aiInput').value;
      chatBox.innerHTML += `<div class='chat-msg'><strong>You:</strong> ${input}</div>`;
      const res = await fetch('http://localhost:11434/api/generate', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ model: "noesis", prompt: input })
      });
      const result = await res.json();
      chatBox.innerHTML += `<div class='chat-msg'><strong>AI:</strong> ${result.response}</div>`;
      document.getElementById('aiInput').value = '';
      chatBox.scrollTop = chatBox.scrollHeight;
    }
  </script>
</body>
</html>
