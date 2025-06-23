# Project-5
Water charity game
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Water Quest</title>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/p5.js/1.4.2/p5.min.js"></script>
  <style>
    body { margin: 0; font-family: Arial, sans-serif; background: #f0f0f0; }
    #game-container { display: flex; justify-content: center; align-items: center; height: 100vh; }
    #ui { position: absolute; top: 10px; left: 10px; color: #000; }
    #menu { position: absolute; top: 10px; right: 10px; background: #FFC107; padding: 10px; border-radius: 5px; }
    #menu button { margin: 5px; padding: 5px 10px; background: #2196F3; color: white; border: none; cursor: pointer; }
    #menu button:hover { background: #1976D2; }
    #event { position: absolute; bottom: 10px; left: 50%; transform: translateX(-50%); background: #fff; padding: 10px; border: 1px solid #000; display: none; }
  </style>
</head>
<body>
  <div id="game-container">
    <div id="ui">
      <img src="https://cdn.charitywater.org/img/logo.png" alt="charity: water" width="100">
      <p>Score: <span id="score">0</span> | Funds: <span id="funds">1000</span> | Water Quality: <span id="quality">50%</span></p>
    </div>
    <div id="menu">
      <button onclick="selectBuilding('well')">Well (200)</button>
      <button onclick="selectBuilding('pipe')">Pipe (100)</button>
      <button onclick="selectBuilding('purifier')">Purifier (300)</button>
    </div>
    <div id="event">
      <p id="event-text"></p>
      <button onclick="applyAISuggestion()">Apply AI Suggestion</button>
      <button onclick="closeEvent()">Ignore</button>
    </div>
  </div>
  <script>
    let houses = [];
    let wells = [];
    let pipes = [];
    let purifiers = [];
    let funds = 1000;
    let score = 0;
    let waterQuality = 50;
    let selectedBuilding = null;
    let peopleServed = 0;
    let eventTimer = 0;
    let eventActive = false;

    function setup() {
      createCanvas(600, 400).parent('game-container');
      // Initialize 20 houses
      for (let i = 0; i < 20; i++) {
        houses.push({ x: random(50, 550), y: random(50, 350), served: false });
      }
    }

    function draw() {
      background(245);
      // Draw houses
      for (let house of houses) {
        fill(house.served ? '#4CAF50' : '#757575');
        rect(house.x, house.y, 20, 20);
      }
      // Draw wells
      for (let well of wells) {
        fill('#2196F3');
        ellipse(well.x, well.y, 15, 15);
      }
      // Draw pipes
      for (let pipe of pipes) {
        stroke('#2196F3');
        line(pipe.x1, pipe.y1, pipe.x2, pipe.y2);
        noStroke();
      }
      // Draw purifiers
      for (let purifier of purifiers) {
        fill('#FFC107');
        rect(purifier.x, purifier.y, 15, 15);
      }
      // Update water quality
      waterQuality += purifiers.length * 0.05;
      if (waterQuality > 100) waterQuality = 100;
      // Update event timer
      eventTimer += deltaTime / 1000;
      if (eventTimer > 10 && !eventActive) {
        triggerEvent();
        eventTimer = 0;
      }
      // Update UI
      document.getElementById('score').innerText = score;
      document.getElementById('funds').innerText = funds;
      document.getElementById('quality').innerText = Math.round(waterQuality) + '%';
      // Check win/lose conditions
      if (peopleServed >= 1000 && waterQuality >= 80) {
        alert('You win! All 1000 people have clean water!');
        noLoop();
      } else if (funds <= 0 && waterQuality < 50) {
        alert('Game over! Funds depleted with poor water quality.');
        noLoop();
      }
    }

    function mousePressed() {
      if (selectedBuilding && funds >= getBuildingCost(selectedBuilding)) {
        if (selectedBuilding === 'well') {
          wells.push({ x: mouseX, y: mouseY });
          connectHouses(mouseX, mouseY, 50);
        } else if (selectedBuilding === 'pipe') {
          pipes.push({ x1: mouseX, y1: mouseY, x2: mouseX + 20, y2: mouseY });
          connectHouses(mouseX, mouseY, 30);
        } else if (selectedBuilding === 'purifier') {
          purifiers.push({ x: mouseX, y: mouseY });
        }
        funds -= getBuildingCost(selectedBuilding);
        score = peopleServed * 100 + Math.round(funds / 100) * 10;
        selectedBuilding = null;
      }
    }

    function getBuildingCost(building) {
      if (building === 'well') return 200;
      if (building === 'pipe') return 100;
      if (building === 'purifier') return 300;
      return 0;
    }

    function connectHouses(x, y, range) {
      for (let house of houses) {
        if (!house.served && dist(x, y, house.x, house.y) < range) {
          house.served = true;
          peopleServed += 50;
        }
      }
    }

    function triggerEvent() {
      eventActive = true;
      let events = [
        { text: 'Drought hits! Water quality drops.', effect: () => waterQuality -= 20, suggestion: 'Build a rainwater catchment (purifier).', apply: () => selectBuilding('purifier') },
        { text: 'Contamination detected! Purifiers needed.', effect: () => waterQuality -= 15, suggestion: 'Build a purifier.', apply: () => selectBuilding('purifier') },
        { text: 'Community grows! More water needed.', effect: () => houses.push({ x: random(50, 550), y: random(50, 350), served: false }), suggestion: 'Build a well.', apply: () => selectBuilding('well') }
      ];
      let event = random(events);
      document.getElementById('event-text').innerText = event.text + ' AI Suggestion: ' + event.suggestion;
      document.getElementById('event').style.display = 'block';
      event.effect();
      window.currentEvent = event;
    }

    function selectBuilding(type) {
      selectedBuilding = type;
    }

    function applyAISuggestion() {
      if (window.currentEvent) {
        window.currentEvent.apply();
        closeEvent();
      }
    }

    function closeEvent() {
      document.getElementById('event').style.display = 'none';
      eventActive = false;
    }
  </script>
</body>
</html>
