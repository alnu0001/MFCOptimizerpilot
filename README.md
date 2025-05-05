<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Smart MFC Optimizer Pilot (AI-Enhanced)</title>
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
  <style>
    body { font-family: 'Segoe UI', sans-serif; margin: 0; padding: 0; background-color: #f9f9f9; color: #333; }
    header { background-color: #1a202c; color: white; padding: 15px; text-align: center; font-size: 24px; }
    .tabs { display: flex; justify-content: center; margin-top: 10px; }
    .tab { margin: 0 10px; padding: 10px 20px; cursor: pointer; background: #e2e8f0; border-radius: 5px; }
    .tab.active { background: #4a5568; color: white; }
    .container { padding: 20px; max-width: 1200px; margin: auto; }
    .panel { display: none; background: white; padding: 20px; border-radius: 8px; box-shadow: 0 0 10px rgba(0,0,0,0.1); }
    .panel.active { display: block; }
    label, select, input, button, textarea { display: block; margin-top: 10px; width: 100%; }
    canvas { margin-top: 20px; max-width: 100%; }
    .input-section { margin-bottom: 20px; }
    .tooltip { font-size: 12px; color: gray; }
    .results-box { margin-top: 15px; background: #edf2f7; padding: 15px; border-radius: 5px; }
  </style>
</head>
<body>
  <header>Smart MFC Optimizer Pilot (AI-Enhanced)</header>
  <div class="tabs">
    <div class="tab active" onclick="switchTab('inputTab')">Inputs</div>
    <div class="tab" onclick="switchTab('resultsTab')">Results</div>
    <div class="tab" onclick="switchTab('graphsTab')">Graphs</div>
  </div>
  <div class="container">
    <div id="inputTab" class="panel active">
      <div class="input-section">
        <label for="naturalLang">Describe Your Scenario</label>
        <textarea id="naturalLang" rows="2" placeholder="e.g. pH increased, heat increased, humidity decreased"></textarea>

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

        <button onclick="parseNaturalLanguage(); simulateMFC();">Describe Scenario</button>
        <button onclick="simulateMFC()">Simulate</button>
        <button onclick="loadPrevious()">Load Previous</button>
        <button onclick="resetInputs()">Reset</button>
      </div>
    </div>

    <div id="resultsTab" class="panel">
      <h2>Simulation Results</h2>
      <div id="numericOutput" class="results-box"></div>
      <button onclick="exportReport('pdf')">Download as PDF</button>
      <button onclick="exportReport('docx')">Download as Word</button>
    </div>

    <div id="graphsTab" class="panel">
      <h2>Graphs</h2>
      <canvas id="chartPower"></canvas>
      <canvas id="chartVoltage"></canvas>
      <canvas id="chartResistance"></canvas>
    </div>
  </div>

  <script>
    let chartPower, chartVoltage, chartResistance;

    function switchTab(id) {
      document.querySelectorAll('.panel').forEach(p => p.classList.remove('active'));
      document.querySelectorAll('.tab').forEach(t => t.classList.remove('active'));
      document.getElementById(id).classList.add('active');
      document.querySelector(`.tab[onclick*='${id}']`).classList.add('active');
    }

    function bestMatch(type, value) {
      if (value.toLowerCase().includes('best')) {
        return type === 'microbe' ? 'Shewanella' : type === 'substrate' ? 'Acetate' : 'mtrC';
      }
      return value;
    }

    function parseNaturalLanguage() {
      const input = document.getElementById('naturalLang').value.toLowerCase();
      if (input.includes('ph increase')) {
        document.getElementById('inputCOD').value = Math.max(0, parseInt(document.getElementById('inputCOD').value) - 5);
      }
      if (input.includes('heat increase')) {
        document.getElementById('inputCE').value = Math.max(0, parseInt(document.getElementById('inputCE').value) - 5);
      }
      if (input.includes('humidity decrease')) {
        document.getElementById('inputCE').value = Math.max(0, parseInt(document.getElementById('inputCE').value) - 3);
        document.getElementById('inputCOD').value = Math.max(0, parseInt(document.getElementById('inputCOD').value) - 3);
      }
    }

    function simulateMFC() {
      const CE = parseFloat(document.getElementById('inputCE').value);
      const COD = parseFloat(document.getElementById('inputCOD').value);
      const E = parseFloat(document.getElementById('inputVoltage').value);
      const hours = parseInt(document.getElementById('inputTimeScale').value);
      const microbe = bestMatch('microbe', document.getElementById('inputMicrobe').value);
      const substrate = bestMatch('substrate', document.getElementById('inputSubstrate').value);
      const enzyme = document.getElementById('inputEnzyme').value;

      const Faraday = 96485, e_per_g_COD = 0.00834, COD_input = 20;
      const COD_removed = COD_input * (COD / 100);
      const mol_e = COD_removed * e_per_g_COD;
      const total_charge = mol_e * Faraday * (CE / 100);
      const power = (total_charge * E / (3600 * hours)).toFixed(3);
      const voltage_drop = (E * (1 - CE / 100)).toFixed(3);
      const internal_resistance = ((E - voltage_drop) / (total_charge / (3600 * hours))).toFixed(2);

      document.getElementById("numericOutput").innerHTML = `
        <strong>Power Output:</strong> ${power} W/m²<br>
        <strong>Voltage Drop:</strong> ${voltage_drop} V<br>
        <strong>Internal Resistance:</strong> ${internal_resistance} Ω<br>
        <strong>Microbe:</strong> ${microbe}, <strong>Substrate:</strong> ${substrate}, <strong>Enzyme:</strong> ${enzyme}`;

      localStorage.setItem('lastSim', JSON.stringify({ CE, COD, E, hours, microbe, substrate, enzyme, power, voltage_drop, internal_resistance }));
      renderCharts(hours, parseFloat(power), parseFloat(voltage_drop), parseFloat(internal_resistance));
      switchTab('graphsTab');
    }

    function renderCharts(hours, power, voltage_drop, resistance) {
      const labels = Array.from({ length: hours }, (_, i) => i + 1);
      const powers = labels.map(i => power + Math.sin(i / 10) * 0.002);
      const voltages = labels.map(i => voltage_drop + Math.cos(i / 20) * 0.001);
      const resistances = labels.map(i => resistance + Math.sin(i / 15) * 0.005);

      if (chartPower) chartPower.destroy();
      if (chartVoltage) chartVoltage.destroy();
      if (chartResistance) chartResistance.destroy();

      chartPower = new Chart(document.getElementById('chartPower'), {
        type: 'line', data: { labels, datasets: [{ label: "Power Output (W/m²)", data: powers, borderColor: 'green', fill: false }] }, options: { responsive: true }
      });
      chartVoltage = new Chart(document.getElementById('chartVoltage'), {
        type: 'line', data: { labels, datasets: [{ label: "Voltage Drop (V)", data: voltages, borderColor: 'red', fill: false }] }, options: { responsive: true }
      });
      chartResistance = new Chart(document.getElementById('chartResistance'), {
        type: 'line', data: { labels, datasets: [{ label: "Internal Resistance (Ω)", data: resistances, borderColor: 'blue', fill: false }] }, options: { responsive: true }
      });
    }

    function exportReport(format) {
      const results = document.getElementById("numericOutput").innerText;
      const canvasPower = document.getElementById('chartPower');
      const canvasVoltage = document.getElementById('chartVoltage');
      const canvasResistance = document.getElementById('chartResistance');
      const powerImg = canvasPower.toDataURL("image/png");
      const voltageImg = canvasVoltage.toDataURL("image/png");
      const resistanceImg = canvasResistance.toDataURL("image/png");

      if (format === 'pdf') {
        const { jsPDF } = window.jspdf;
        const doc = new jsPDF();
        doc.setFontSize(14);
        doc.text("MFC Optimizer Report", 10, 10);
        doc.setFontSize(10);
        doc.text(results, 10, 20);
        doc.addImage(powerImg, 'PNG', 10, 40, 180, 40);
        doc.addImage(voltageImg, 'PNG', 10, 90, 180, 40);
        doc.addImage(resistanceImg, 'PNG', 10, 140, 180, 40);
        doc.save("mfc_report.pdf");
      } else {
        alert("Word export not yet implemented. Use PDF.");
      }
    }
  </script>
</body>
</html>
