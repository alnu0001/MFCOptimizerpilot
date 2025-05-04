<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>MFC Optimizer Pilot (ML-Enhanced)</title>
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/@tensorflow/tfjs"></script>
  <style>
    :root {
      --bg-color: #121212;
      --panel-bg: #1e1e1e;
      --text-color: #e0e0e0;
      --accent: #2d3748;
      --highlight: #4fd1c5;
      --input-bg: #2a2a2a;
      --border-color: #333;
    }

    body.light-mode {
      --bg-color: #ffffff;
      --panel-bg: #f1f1f1;
      --text-color: #222222;
      --accent: #e2e8f0;
      --highlight: #3182ce;
      --input-bg: #ffffff;
      --border-color: #ccc;
    }

    body {
      font-family: 'Segoe UI', sans-serif;
      margin: 0;
      padding: 0;
      background-color: var(--bg-color);
      color: var(--text-color);
      transition: all 0.3s ease;
    }
    header {
      background-color: var(--accent);
      color: white;
      padding: 15px;
      text-align: center;
      font-size: 24px;
      position: relative;
    }
    .theme-toggle {
      position: absolute;
      right: 20px;
      top: 15px;
      background: var(--highlight);
      color: black;
      border: none;
      padding: 5px 10px;
      border-radius: 5px;
      cursor: pointer;
      font-weight: bold;
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
      background: #2d2d2d;
      color: #aaa;
      border-radius: 5px;
      border: 1px solid var(--border-color);
    }
    .tab.active {
      background: var(--highlight);
      color: black;
    }
    .container {
      padding: 20px;
      max-width: 1200px;
      margin: auto;
    }
    .panel {
      display: none;
      background: var(--panel-bg);
      padding: 20px;
      border-radius: 8px;
      box-shadow: 0 0 10px rgba(0,0,0,0.5);
    }
    .panel.active {
      display: block;
    }
    label, select, input, button {
      display: block;
      margin-top: 10px;
      width: 100%;
    }
    input, select {
      background-color: var(--input-bg);
      border: 1px solid var(--border-color);
      color: var(--text-color);
      padding: 8px;
      border-radius: 4px;
    }
    button {
      background-color: var(--highlight);
      color: black;
      border: none;
      padding: 10px;
      border-radius: 5px;
      font-weight: bold;
      cursor: pointer;
    }
    button:hover {
      opacity: 0.9;
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
  <header>
    MFC Optimizer Pilot (ML-Enhanced)
    <button class="theme-toggle" onclick="toggleTheme()">Toggle Night Mode</button>
  </header>

  <!-- Existing HTML stays unchanged below -->
  <!-- JS BELOW -->
  <script>
    function toggleTheme() {
      document.body.classList.toggle('light-mode');
    }
  </script>
</body>
</html>
