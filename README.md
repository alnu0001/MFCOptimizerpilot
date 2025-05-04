<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>MFC Simulator Pro</title>
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
  <style>
    body {
      font-family: 'Segoe UI', sans-serif;
      background-color: #f4f4f4;
      margin: 0;
      padding: 20px;
    }
    header {
      background-color: #2e3b4e;
      color: white;
      padding: 15px;
      text-align: center;
      font-size: 24px;
    }
    .container {
      display: grid;
      grid-template-columns: 1fr 2fr;
      gap: 20px;
      margin-top: 20px;
    }
    .panel {
      background: white;
      padding: 20px;
      border-radius: 8px;
      box-shadow: 0 0 10px rgba(0,0,0,0.1);
    }
    label, select, input, button {
      display: block;
      width: 100%;
      margin-top: 10px;
    }
    canvas {
      margin-top: 20px;
      max-width: 100%;
    }
  </style>
</head>
<body>
  <header>Microbial Fuel Cell Simulator Pro</header>
  <div class="container">
    <div class="panel">
      <h2>Input Parameters</h2>
      <label>Coulombic Efficiency (%)</label>
      <input id="inputCE" type="number" value="70" />

      <label>COD Removal (%)</label>
      <input id="inputCOD" type="number" value="70" />

      <label>Microbe Type</label>
      <input list="microbes" id="inputMicrobe" />
      <datalist id="microbes">
        <option value="Geobacter" />
        <option value="Shewanella" />
        <option value="Pseudomonas aeruginosa" />
        <option value="Yeast" />
        <option value="Best option" />
      </datalist>

      <label>Substrate Composition</label>
      <input list="substrates" id="inputSubstrate" />
      <datalist id="substrates">
        <option value="Starch" />
        <option value="Molasses" />
        <option value="Acetate" />
        <option value="Best option" />
      </datalist>

      <label>Enzyme Type</label>
      <input id="inputEnzyme" type="text" value="mtrC" />

      <label>Cell Voltage (V)</label>
      <input id="inputVoltage" type="number" value="0.4" step="0.01" />

      <label>Simulation Duration</label>
      <select id="inputTimeScale">
        <option value="24">24 Hours</option>
        <option value="168">7 Days</option>
        <option value="720">30 Days</option>
      </select>

      <button onclick="simulateMFC()">Simulate</button>
      <button onclick="loadPrevious()">Load Previous</button>

      <div id="numericOutput"></div>
    </div>

    <div class="panel">
      <h2>Simulation Graphs</h2>
      <canvas id="chartPower"></canvas>
      <canvas id="chartVoltage"></canvas>
      <canvas id="chartResistance"></canvas>
    </div>
  </div>

  <script>
    const Faraday = 96485, e_per_g_COD = 0.00834, COD_input = 20;

    function bestMatch(type, value) {
      if (value.toLowerCase().includes('best')) {
        return type === 'microbe' ? 'Shewanella' :
               type === 'substrate' ? 'Acetate' :
               'mtrC';
      }
      return value;
    }

    function simulateMFC() {
      const CE = parseFloat(document.getElementById('inputCE').value);
      const COD = parseFloat(document.getElementById('inputCOD').value);
      const E = parseFloat(document.getElementById('inputVoltage').value);
      const hours = parseInt(document.getElementById('inputTimeScale').value);
      const microbe = bestMatch('microbe', document.getElementById('inputMicrobe').value);
      const substrate = bestMatch('substrate', document.getElementById('inputSubstrate').value);
      const enzyme = document.getElementById('inputEnzyme').value;

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
      document.getElementById("numericOutput").innerHTML =
        `<strong>Power Output:</strong> ${data.power} W/m²<br>
         <strong>Voltage Drop:</strong> ${data.voltage_drop} V<br>
         <strong>Internal Resistance:</strong> ${data.internal_resistance} Ω<br>
         <strong>Microbe:</strong> ${data.microbe}, <strong>Substrate:</strong> ${data.substrate}, <strong>Enzyme:</strong> ${data.enzyme}`;
      renderCharts(data.hours, data.power, data.voltage_drop, data.internal_resistance);
    }

    function renderCharts(hours, power, voltage_drop, resistance) {
      const labels = Array.from({ length: hours }, (_, i) => i + 1);
      const powers = labels.map(() => parseFloat(power));
      const voltages = labels.map(() => parseFloat(voltage_drop));
      const resistances = labels.map(() => parseFloat(resistance));

      const config = (label, data, color) => ({
        type: 'line',
        data: {
          labels,
          datasets: [{
            label,
            data,
            borderColor: color,
            fill: false
          }]
        },
        options: {
          responsive: true,
          scales: { x: { title: { display: true, text: 'Time (h)' } } }
        }
      });

      new Chart(document.getElementById('chartPower'), config("Power Output (W/m²)", powers, 'green'));
      new Chart(document.getElementById('chartVoltage'), config("Voltage Drop (V)", voltages, 'red'));
      new Chart(document.getElementById('chartResistance'), config("Internal Resistance (Ω)", resistances, 'blue'));
    }
  </script>
</body>
</html>
