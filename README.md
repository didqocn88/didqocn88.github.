<!DOCTYPE html>
<html lang="ko">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>치매 예방 미로 훈련</title>

<link href="https://fonts.googleapis.com/css2?family=Noto+Sans+KR:wght@400;700;900&display=swap" rel="stylesheet">

<style>
*{box-sizing:border-box;margin:0;padding:0}
body{
  background:#f7f7f5;
  font-family:'Noto Sans KR',sans-serif;
  color:#1a1a1a
}
#app{
  display:flex;
  flex-direction:column;
  align-items:center;
  padding:20px 16px 60px;
  min-height:100vh
}
button{font-family:inherit}

.sbtn{
  cursor:pointer;
  border-radius:12px;
  font-weight:700;
  font-size:18px;
  padding:11px 20px;
  border:2px solid #ccc;
  background:#fff;
}

.sbtn.on{
  color:#fff;
}

.dbtn{
  cursor:pointer;
  width:78px;
  height:78px;
  font-size:34px;
  border-radius:16px;
  border:3px solid #d0d0d0;
  background:#fff;
}

#maze-svg{
  background:#fff;
  border-radius:12px;
}
</style>
</head>

<body>

<div id="app">

<h1 style="font-size:32px;font-weight:900;margin-bottom:8px">
치매 예방 미로 훈련
</h1>

<div style="margin-bottom:16px;color:#666">
방향키 또는 버튼으로 이동하세요
</div>

<div id="stage-tabs"
style="display:flex;gap:10px;margin-bottom:16px"></div>

<div style="margin-bottom:12px">
이동 횟수:
<span id="move-count" style="font-weight:900">0</span>
</div>

<div style="margin-bottom:14px">
<button onclick="initStage()">🗺️ 새로고침</button>
</div>

<div style="position:relative">
<svg id="maze-svg"></svg>
</div>

<div style="margin-top:20px;display:flex;flex-direction:column;align-items:center;gap:8px">
  <button class="dbtn" onclick="move('N')">⬆️</button>

  <div style="display:flex;gap:8px">
    <button class="dbtn" onclick="move('W')">⬅️</button>
    <button class="dbtn" onclick="move('E')">➡️</button>
  </div>

  <button class="dbtn" onclick="move('S')">⬇️</button>
</div>

</div>

<script>

const STAGES = [
  {id:1,cols:5,rows:5,color:"#2e7d32",label:"1단계"},
  {id:2,cols:7,rows:7,color:"#1565c0",label:"2단계"},
  {id:3,cols:9,rows:9,color:"#e65100",label:"3단계"},
];

let stageIdx = 0;
let maze = null;
let player = {r:0,c:0};
let moves = 0;

function generateMaze(cols,rows){

  const cells = Array.from({length:rows},(_,r)=>
    Array.from({length:cols},(_,c)=>({
      r,c,
      walls:{N:true,S:true,E:true,W:true},
      visited:false
    }))
  );

  const stack = [];
  const start = cells[0][0];

  start.visited = true;
  stack.push(start);

  const dirs = [
    {d:"N",dr:-1,dc:0,opp:"S"},
    {d:"S",dr:1,dc:0,opp:"N"},
    {d:"E",dr:0,dc:1,opp:"W"},
    {d:"W",dr:0,dc:-1,opp:"E"},
  ];

  while(stack.length){

    const cur = stack[stack.length-1];

    const neighbors = dirs.map(({d,dr,dc,opp})=>{
      const nr = cur.r + dr;
      const nc = cur.c + dc;

      if(
        nr>=0 &&
        nr<rows &&
        nc>=0 &&
        nc<cols &&
        !cells[nr][nc].visited
      ){
        return {cell:cells[nr][nc],d,opp};
      }

      return null;

    }).filter(Boolean);

    if(!neighbors.length){
      stack.pop();
      continue;
    }

    const {cell,d,opp} =
      neighbors[Math.floor(Math.random()*neighbors.length)];

    cur.walls[d] = false;
    cell.walls[opp] = false;

    cell.visited = true;
    stack.push(cell);
  }

  return cells;
}

function renderTabs(){

  const wrap = document.getElementById("stage-tabs");
  wrap.innerHTML = "";

  STAGES.forEach((s,i)=>{

    const b = document.createElement("button");

    b.className = "sbtn" + (i===stageIdx ? " on" : "");
    b.textContent = s.label;

    if(i===stageIdx){
      b.style.background = s.color;
    }

    b.onclick = ()=>{
      stageIdx = i;
      renderTabs();
      initStage();
    };

    wrap.appendChild(b);

  });

}

function initStage(){

  const s = STAGES[stageIdx];

  maze = generateMaze(s.cols,s.rows);

  player = {r:0,c:0};

  moves = 0;

  renderMaze();

}

function renderMaze(){

  const s = STAGES[stageIdx];

  const cs = 50;

  const rows = maze.length;
  const cols = maze[0].length;

  const W = cols * cs + 2;
  const H = rows * cs + 2;

  const svg = document.getElementById("maze-svg");

  svg.setAttribute("width",W);
  svg.setAttribute("height",H);

  svg.style.border = `3px solid ${s.color}`;

  let html = "";

  for(let r=0;r<rows;r++){

    for(let c=0;c<cols;c++){

      const cell = maze[r][c];

      const x = c*cs+1;
      const y = r*cs+1;

      html += `
      <rect
        x="${x}"
        y="${y}"
        width="${cs}"
        height="${cs}"
        fill="#fff"
      />
      `;

      if(cell.walls.N)
      html += `<line x1="${x}" y1="${y}" x2="${x+cs}" y2="${y}" stroke="#222"/>`;

      if(cell.walls.S)
      html += `<line x1="${x}" y1="${y+cs}" x2="${x+cs}" y2="${y+cs}" stroke="#222"/>`;

      if(cell.walls.W)
      html += `<line x1="${x}" y1="${y}" x2="${x}" y2="${y+cs}" stroke="#222"/>`;

      if(cell.walls.E)
      html += `<line x1="${x+cs}" y1="${y}" x2="${x+cs}" y2="${y+cs}" stroke="#222"/>`;

    }

  }

  const px = player.c*cs+1+cs/2;
  const py = player.r*cs+1+cs/2;

  html += `
  <circle
    cx="${px}"
    cy="${py}"
    r="${cs*0.25}"
    fill="#1976d2"
  />
  `;

  svg.innerHTML = html;

  document.getElementById("move-count").textContent = moves;

}

function move(dir){

  const cell = maze[player.r][player.c];

  if(cell.walls[dir]) return;

  const map = {
    N:[-1,0],
    S:[1,0],
    E:[0,1],
    W:[0,-1]
  };

  const [dr,dc] = map[dir];

  player.r += dr;
  player.c += dc;

  moves++;

  renderMaze();

}

document.addEventListener("keydown",e=>{

  const map = {
    ArrowUp:"N",
    ArrowDown:"S",
    ArrowLeft:"W",
    ArrowRight:"E"
  };

  const dir = map[e.key];

  if(!dir) return;

  e.preventDefault();

  move(dir);

});

renderTabs();
initStage();

</script>

</body>
</html>
