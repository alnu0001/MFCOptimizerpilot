<!--
Microbial Fuel Cells (MFCs) are bio-electrochemical devices that harness the metabolic activity of microorganisms to convert organic matter into electrical energy. They have potential applications in renewable energy and wastewater treatment. The MFC Optimizer is an interactive client-side tool that lets users input design parameters (microbe type, substrate, enzymes, coulombic efficiency, COD removal, etc.) and uses a pre-trained machine learning model to predict the MFC output (e.g., voltage or power) given those inputs. It then simulates how this output evolves over time under the specified conditions. The tool uses Chart.js for visualization and stores user settings in localStorage so inputs persist across sessions. It runs fully in the browser (client-side only), making it easy to deploy (for example, via GitHub Pages).
-->
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>MFC Optimizer Tool</title>
  <!-- Load TensorFlow.js and Chart.js from CDN -->
  <script src="https://cdn.jsdelivr.net/npm/@tensorflow/tfjs"></script>
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
  <!-- Basic styling -->
  <style>
    body { font-family: Arial, sans-serif; margin: 20px; }
    h1 { color: #2c3e50; }
    form { margin-bottom: 20px; }
    label { display: block; margin: 5px 0; }
    input[type='number'], select { padding: 5px; margin: 2px 0; width: 220px; }
    button { padding: 10px 15px; margin-top: 10px; background: #3498db; color: white; border: none; cursor: pointer; }
    button:hover { background: #2980b9; }
    #predictionResult { font-weight: bold; }
    canvas { max-width: 100%; }
  </style>
</head>
<body>
  <h1>MFC (Microbial Fuel Cell) Optimizer</h1>
  <p>Enter parameters below and press "Predict & Simulate" to see the MFC output prediction and its behavior over time.</p>
  <!-- Input form for MFC parameters -->
  <form id="mfcForm">
    <label>Microbe Type:
      <select id="microbe">
        <option value="ExoelectrogenicBacteria">Exoelectrogenic Bacteria</option>
        <option value="PhotosyntheticBacteria">Photosynthetic Bacteria</option>
        <option value="Yeast">Yeast</option>
        <option value="Algae">Algae</option>
      </select>
    </label>
    <label>Substrate:
      <select id="substrate">
        <option value="Glucose">Glucose</option>
        <option value="Acetate">Acetate</option>
        <option value="Cellulose">Cellulose</option>
        <option value="Wastewater">Wastewater</option>
      </select>
    </label>
    <label>Enzyme:
      <select id="enzyme">
        <option value="Hydrogenase">Hydrogenase</option>
        <option value="Cytochrome">Cytochrome</option>
        <option value="Laccase">Laccase</option>
        <option value="Rhodopseudomonas Enzymes">Rhodopseudomonas Enzymes</option>
      </select>
    </label>
    <label>Coulombic Efficiency (%):
      <input type="number" id="coulEff" min="0" max="100" step="1" value="50">
    </label>
    <label>COD Removal (%):
      <input type="number" id="codRemoval" min="0" max="100" step="1" value="50">
    </label>
    <button type="button" onclick="runPrediction()">Predict & Simulate</button>
  </form>
  <p>Predicted MFC Output: <span id="predictionResult">--</span> (arbitrary units)</p>
  <!-- Chart container -->
  <canvas id="mfcChart" width="600" height="400"></canvas>
  <!-- Scripts for model loading, prediction, simulation, and localStorage -->
  <script>
    let model;
    // Load the pre-trained TensorFlow.js model (placeholder path)
    async function loadModel() {
      model = await tf.loadLayersModel('path/to/model.json');
      console.log('Model loaded.');
    }
    // Encoding functions for select inputs
    function encodeMicrobe(val) {
      switch(val) {
        case 'ExoelectrogenicBacteria': return 0;
        case 'PhotosyntheticBacteria': return 1;
        case 'Yeast': return 2;
        case 'Algae': return 3;
      }
    }
    function encodeSubstrate(val) {
      switch(val) {
        case 'Glucose': return 0;
        case 'Acetate': return 1;
        case 'Cellulose': return 2;
        case 'Wastewater': return 3;
      }
    }
    function encodeEnzyme(val) {
      switch(val) {
        case 'Hydrogenase': return 0;
        case 'Cytochrome': return 1;
        case 'Laccase': return 2;
        case 'Rhodopseudomonas Enzymes': return 3;
      }
    }
    // Initialize Chart.js line chart
    const ctx = document.getElementById('mfcChart').getContext('2d');
    const mfcChart = new Chart(ctx, {
      type: 'line',
      data: {
        labels: [],
        datasets: [{
          label: 'Predicted Output Over Time',
          data: [],
          borderColor: 'rgba(52, 152, 219, 1)',
          backgroundColor: 'rgba(52, 152, 219, 0.2)',
          fill: true
        }]
      },
      options: {
        responsive: true,
        scales: {
          x: { title: { display: true, text: 'Time (hours)' } },
          y: { title: { display: true, text: 'MFC Output' } }
        }
      }
    });
    // Run prediction using model and simulate chart
    async function runPrediction() {
      if (!model) {
        alert('Model not loaded yet. Please wait.');
        return;
      }
      // Read input values
      const microbe = encodeMicrobe(document.getElementById('microbe').value);
      const substrate = encodeSubstrate(document.getElementById('substrate').value);
      const enzyme = encodeEnzyme(document.getElementById('enzyme').value);
      const coulEff = parseFloat(document.getElementById('coulEff').value) / 100.0;
      const codRemoval = parseFloat(document.getElementById('codRemoval').value) / 100.0;
      // Prepare tensor input for prediction
      const inputTensor = tf.tensor2d([[microbe, substrate, enzyme, coulEff, codRemoval]]);
      // Run model prediction
      const outputTensor = model.predict(inputTensor);
      const outputData = await outputTensor.data();
      const predicted = outputData[0];
      // Display predicted output (rounded)
      document.getElementById('predictionResult').textContent = predicted.toFixed(2);
      // Simulate output over time (example: exponential increase)
      const hours = Array.from({length: 25}, (_, i) => i);
      const simData = hours.map(h => parseFloat((predicted * Math.exp(0.1 * h)).toFixed(2)));
      mfcChart.data.labels = hours.map(h => h + 'h');
      mfcChart.data.datasets[0].data = simData;
      mfcChart.update();
    }
    // Save user settings to localStorage
    function saveSettings() {
      localStorage.setItem('microbe', document.getElementById('microbe').value);
      localStorage.setItem('substrate', document.getElementById('substrate').value);
      localStorage.setItem('enzyme', document.getElementById('enzyme').value);
      localStorage.setItem('coulEff', document.getElementById('coulEff').value);
      localStorage.setItem('codRemoval', document.getElementById('codRemoval').value);
    }
    // Load saved settings from localStorage
    function loadSettings() {
      if (localStorage.getItem('microbe')) {
        document.getElementById('microbe').value = localStorage.getItem('microbe');
      }
      if (localStorage.getItem('substrate')) {
        document.getElementById('substrate').value = localStorage.getItem('substrate');
      }
      if (localStorage.getItem('enzyme')) {
        document.getElementById('enzyme').value = localStorage.getItem('enzyme');
      }
      if (localStorage.getItem('coulEff')) {
        document.getElementById('coulEff').value = localStorage.getItem('coulEff');
      }
      if (localStorage.getItem('codRemoval')) {
        document.getElementById('codRemoval').value = localStorage.getItem('codRemoval');
      }
    }
    // Event listeners to save settings on change
    document.getElementById('microbe').addEventListener('change', saveSettings);
    document.getElementById('substrate').addEventListener('change', saveSettings);
    document.getElementById('enzyme').addEventListener('change', saveSettings);
    document.getElementById('coulEff').addEventListener('input', saveSettings);
    document.getElementById('codRemoval').addEventListener('input', saveSettings);
    // On page load: load settings and model
    window.addEventListener('load', async () => {
      loadSettings();
      await loadModel();
    });
  </script>
</body>
</html>
