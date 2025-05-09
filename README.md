<!DOCTYPE html>
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
      <!-- (omitted unchanged input form content) -->
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

    async function sendAIQuery() {
      const input = document.getElementById('aiInput').value;
      if (!input.trim()) return;
      const chatBox = document.getElementById('chatBox');
      chatBox.innerHTML += `<div class='chat-msg'><strong>You:</strong> ${input}</div>`;

      try {
        const response = await fetch('http://localhost:11434/api/generate', {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify({
            model: 'noesis',
            prompt: `You are a smart assistant specialized in MFC (Microbial Fuel Cell) optimization. Respond helpfully, concisely, and contextually. User input: "${input}"`
          })
        });
        const result = await response.json();
        chatBox.innerHTML += `<div class='chat-msg'><strong>AI:</strong> ${result.response.trim()}</div>`;
      } catch (err) {
        chatBox.innerHTML += `<div class='chat-msg'><strong>AI:</strong> I'm having trouble connecting to the optimizer backend right now.</div>`;
      }

      document.getElementById('aiInput').value = '';
      chatBox.scrollTop = chatBox.scrollHeight;
    }
  </script>
</body>
</html>
