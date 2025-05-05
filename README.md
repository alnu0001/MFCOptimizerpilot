<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Smart MFC Simulator</title>
  <style>
    body {
      font-family: sans-serif;
      margin: 20px;
    }
    label, select, input {
      display: block;
      margin: 10px 0;
    }
    button {
      margin-top: 10px;
      padding: 10px 15px;
      font-size: 1rem;
    }
    .output {
      margin-top: 20px;
      background: #f9f9f9;
      padding: 10px;
      border-left: 4px solid #0077cc;
    }
    .dropdown {
      position: relative;
      display: inline-block;
    }
    .dropdown-content {
      display: none;
      position: absolute;
      background-color: white;
      box-shadow: 0px 8px 16px rgba(0,0,0,0.2);
      z-index: 1;
      min-width: 160px;
    }
    .dropdown-content button {
      width: 100%;
      text-align: left;
    }
    .dropdown:hover .dropdown-content {
      display: block;
    }
  </style>
</head>
<body>

<h1>Microbial Fuel Cell Optimizer</h1>

<form id="mfcForm">
  <label>Coulombic Efficiency (%):
    <input type="number" id="ce" min="0" max="100" value="70">
  </label>
  <label>COD Removal (%):
    <input type="number" id="cod" min="0" max="100" value="70">
  </label>
  <label>Microbe Type:
    <input type="text" id="microbe" placeholder="e.g. Shewanella">
  </label>
  <label>Substrate Composition:
    <input type="text" id="substrate" placeholder="e.g. Starch">
  </label>
  <label>Enzyme Type:
    <input type="text" id="enzyme" placeholder="e.g. mtrC">
  </label>
  <label>Cell Voltage (V):
    <input type="number" id="voltage" step="0.01" value="0.4">
  </label>

  <button type="button" onclick="simulate()">Run Simulation</button>
  <button type="button" onclick="resetForm()">Reset</button>
</form>

<div class="output" id="result" style="display:none;"></div>

<div class="dropdown">
  <button>Download ▼</button>
  <div class="dropdown-content">
    <button onclick="downloadDoc('pdf')">Download as PDF</button>
    <button onclick="downloadDoc('word')">Download as Word</button>
  </div>
</div>

<script>
  function simulate() {
    const ce = parseFloat(document.getElementById('ce').value);
    const cod = parseFloat(document.getElementById('cod').value);
    const microbe = document.getElementById('microbe').value;
    const substrate = document.getElementById('substrate').value;
    const enzyme = document.getElementById('enzyme').value;
    const voltage = parseFloat(document.getElementById('voltage').value);

    const power = ((ce/100) * (cod/100) * voltage * 0.25).toFixed(4); // Simplified calc

    const output = `
      <h3>Simulation Results</h3>
      <p><strong>Microbe:</strong> ${microbe}</p>
      <p><strong>Substrate:</strong> ${substrate}</p>
      <p><strong>Enzyme:</strong> ${enzyme}</p>
      <p><strong>Cell Voltage:</strong> ${voltage} V</p>
      <p><strong>Estimated Power Output:</strong> ${power} W/m²</p>
    `;
    document.getElementById('result').innerHTML = output;
    document.getElementById('result').style.display = 'block';
  }

  function resetForm() {
    document.getElementById('mfcForm').reset();
    document.getElementById('result').style.display = 'none';
  }

  function downloadDoc(type) {
    const result = document.getElementById('result');
    if (result.style.display === 'none') {
      alert("Run simulation first!");
      return;
    }

    const content = result.innerText;
    const blob = new Blob([content], { type: type === 'pdf' ? 'application/pdf' : 'application/msword' });
    const link = document.createElement("a");
    link.href = URL.createObjectURL(blob);
    link.download = `MFC_Simulation.${type === 'pdf' ? 'pdf' : 'doc'}`;
    link.click();
  }
</script>

</body>
</html>
