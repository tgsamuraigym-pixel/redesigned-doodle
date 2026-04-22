# redesigned-doodle
<!DOCTYPE html>
<html lang="uk">
<head>
<meta charset="UTF-8">
<title>BTC Tap ULTRA</title>

<style>
body {
  margin: 0;
  background: black;
  color: white;
  text-align: center;
  font-family: Arial;
}

#score {
  font-size: 40px;
}

#level {
  margin-top: 10px;
  color: gold;
}

#coin {
  width: 220px;
  height: 220px;
  border-radius: 50%;
  margin: 40px auto;
  background: radial-gradient(circle, #FFD700, #FF8C00);
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 90px;
  cursor: pointer;
  transition: 0.1s;
  position: relative;
}

#coin:active {
  transform: scale(0.85);
}

.plus {
  position: absolute;
  color: lime;
  font-weight: bold;
  animation: float 0.7s forwards;
}

@keyframes float {
  0% {opacity:1; transform:translateY(0);}
  100% {opacity:0; transform:translateY(-40px);}
}

button {
  padding: 10px 20px;
  margin: 5px;
  border: none;
  border-radius: 10px;
  cursor: pointer;
}

#warning {
  color: red;
  height: 20px;
}
</style>
</head>

<body>

<h1>💰 BTC Tap ULTRA</h1>
<div id="score">0</div>
<div id="level">Рівень: 1</div>

<div id="coin">₿</div>

<div id="warning"></div>

<div>
  <button onclick="buyClick()">+1 клік (50)</button>
  <button onclick="buyAuto()">Авто (100)</button>
</div>

<script>
let score = 0;
let perClick = 1;
let auto = 0;
let level = 1;

// анти-бот
let clicks = [];
let intervals = [];
let blocked = false;

const MAX_CPS = 13;

const coin = document.getElementById("coin");
const scoreText = document.getElementById("score");
const levelText = document.getElementById("level");
const warning = document.getElementById("warning");

// звук
const audio = new Audio("https://www.soundjay.com/buttons/sounds/button-16.mp3");

// load
if(localStorage.getItem("save")){
  let s = JSON.parse(localStorage.getItem("save"));
  score = s.score;
  perClick = s.perClick;
  auto = s.auto;
  level = s.level;
}

// save
setInterval(()=>{
  localStorage.setItem("save", JSON.stringify({score, perClick, auto, level}));
},1000);

// авто дохід
setInterval(()=>{
  score += auto;
  update();
},1000);

coin.onclick = () => {
  if(blocked) return;

  let now = Date.now();

  if(clicks.length){
    intervals.push(now - clicks[clicks.length-1]);
  }

  clicks.push(now);
  clicks = clicks.filter(t => now - t < 1000);

  // анти CPS
  if(clicks.length > MAX_CPS){
    block("🚫 Швидкість!");
    return;
  }

  // анти-бот (однакові інтервали)
  if(intervals.length > 5){
    let avg = intervals.reduce((a,b)=>a+b)/intervals.length;
    let suspicious = intervals.every(i => Math.abs(i - avg) < 10);

    if(suspicious){
      block("🤖 Бот detected!");
      return;
    }
  }

  score += perClick;
  showPlus(perClick);
  audio.currentTime = 0;
  audio.play();

  level = Math.floor(score / 100) + 1;

  update();
};

function block(msg){
  blocked = true;
  warning.innerText = msg;
  setTimeout(()=>{
    blocked = false;
    warning.innerText = "";
    clicks = [];
    intervals = [];
  },2000);
}

function showPlus(val){
  let el = document.createElement("div");
  el.className = "plus";
  el.innerText = "+" + val;
  el.style.left = "50%";
  el.style.top = "50%";
  coin.appendChild(el);

  setTimeout(()=>el.remove(),700);
}

function buyClick(){
  if(score >= 50){
    score -= 50;
    perClick++;
    update();
  }
}

function buyAuto(){
  if(score >= 100){
    score -= 100;
    auto++;
    update();
  }
}

function update(){
  scoreText.innerText = Math.floor(score);
  levelText.innerText = "Рівень: " + level;
}

update();
</script>

</body>
</html>
