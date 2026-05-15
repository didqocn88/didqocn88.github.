# didqocn88.github.
<!DOCTYPE html>
<html lang="ko">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>치매 예방 미로 훈련</title>
<link href="https://fonts.googleapis.com/css2?family=Noto+Sans+KR:wght@400;700;900&display=swap" rel="stylesheet">
<style>
*{box-sizing:border-box;margin:0;padding:0}
body{background:#f7f7f5;font-family:'Noto Sans KR',sans-serif;color:#1a1a1a}
#app{display:flex;flex-direction:column;align-items:center;padding:20px 16px 60px;min-height:100vh}
.sbtn{cursor:pointer;border-radius:12px;font-family:'Noto Sans KR',sans-serif;font-weight:700;font-size:18px;padding:11px 20px;border:2.5px solid #ccc;background:#fff;color:#555;transition:all .15s;min-width:82px}
.sbtn.on{color:#fff;border-color:transparent;box-shadow:0 2px 12px rgba(0,0,0,.18)}
.sbtn:hover{transform:scale(1.04)}
.dbtn{cursor:pointer;width:78px;height:78px;font-size:34px;border-radius:16px;border:3px solid #d0d0d0;background:#fff;display:flex;align-items:center;justify-content:center;box-shadow:0 3px 10px rgba(0,0,0,.10);transition:all .12s;user-select:none;-webkit-tap-highlight-color:transparent;font-family:inherit}
.dbtn:active{transform:scale(.9);background:#e8e8e8}
.abtn{cursor:pointer;border:none;border-radius:14px;font-family:'Noto Sans KR',sans-serif;font-weight:700;transition:all .13s}
.abtn:hover{opacity:.88;transform:scale(1.03)}
.fsbtn{cursor:pointer;border-radius:10px;font-family:'Noto Sans KR',sans-serif;font-weight:700;font-size:15px;padding:7px 14px;border:2px solid #ccc;background:#fff;color:#555;transition:all .15s}
.fsbtn.on{background:#455a64;color:#fff;border-color:#455a64}
#maze-svg{display:block;background:#fff;border-radius:12px;box-shadow:0 4px 28px rgba(0,0,0,.09)}
#win-overlay{position:absolute;inset:0;display:none;flex-direction:column;align-items:center;justify-content:center;background:rgba(255,255,255,.93);border-radius:12px;gap:16px}
#win-overlay.show{display:flex}
.dpad-row{display:flex;gap:8px}
.home-btn{cursor:pointer;border:2px solid #ccc;border-radius:12px;background:#fff;font-family:'Noto Sans KR',sans-serif;font-weight:700;font-size:16px;padding:10px 22px;color:#555;transition:all .15s;margin-bottom:4px}
.home-btn:hover{background:#f0f0f0}
</style>
</head>
<body>
<div id="app">

  <!-- 홈 버튼 + 글씨 크기 -->
  <div style="width:100%;max-width:560px;display:flex;justify-content:space-between;align-items:center;margin-bottom:10px;flex-wrap:wrap;gap:8px">
    <button class="home-btn" onclick="history.back()">← 홈으로</button>
    <div style="display:flex;gap:6px;align-items:center">
      <span id="fs-label" style="font-size:14px;color:#888;font-weight:700;margin-right:2px">글씨:</span>
      <button class="fsbtn" data-fs="0" onclick="setFs(0)">작게</button>
      <button class="fsbtn on" data-fs="1" onclick="setFs(1)">보통</button>
      <button class="fsbtn" data-fs="2" onclick="setFs(2)">크게</button>
    </div>
  </div>

  <div id="title" style="font-size:30px;font-weight:900;margin-bottom:4px;text-align:center">치매 예방 미로 훈련</div>
  <div id="subtitle" style="font-size:18px;color:#666;margin-bottom:18px;text-align:center">단계를 골라서 목적지까지 이동하세요</div>

  <!-- 단계 탭 -->
  <div id="stage-tabs" style="display:flex;gap:10px;flex-wrap:wrap;justify-content:center;margin-bottom:16px"></div>

  <!-- 정보 카드 -->
  <div id="info-card" style="width:100%;max-width:520px;background:#fff;border-radius:18px;padding:16px 20px;margin-bottom:14px;box-shadow:0 2px 14px rgba(0,0,0,.07)">
    <div id="stage-label" style="font-size:24px;font-weight:900;margin-bottom:5px"></div>
    <div id="stage-desc" style="font-size:17px;color:#333;font-weight:700;line-height:1.45"></div>
    <div id="order-panel" style="margin-top:12px;display:none"></div>
    <div id="countdown-banner" style="display:none;margin-top:12px;padding:10px 14px;border-radius:12px;background:#fff8e1;border:2.5px solid #f9a825;font-size:18px;font-weight:900;color:#e65100"></div>
    <div id="hidden-banner" style="display:none;margin-top:12px;padding:10px 14px;border-radius:12px;background:#fce4ec;border:2.5px solid #c62828;font-size:18px;font-weight:900;color:#b71c1c">기억을 떠올리며 찾아가세요!</div>
    <div style="margin-top:10px;font-size:17px;color:#777;font-weight:700">이동 횟수: <span id="move-count" style="font-size:22px;font-weight:900">0</span>번</div>
  </div>

  <!-- 액션 버튼 -->
  <div style="display:flex;gap:10px;margin-bottom:12px;flex-wrap:wrap;justify-content:center">
    <button id="hint-btn" onclick="showHint()" style="cursor:pointer;border:none;border-radius:14px;font-family:'Noto Sans KR',sans-serif;font-weight:700;font-size:18px;padding:11px 22px;background:#ff8f00;color:#fff;transition:all .13s">💡 길 힌트 (1회)</button>
    <button onclick="initStage()" style="cursor:pointer;border:none;border-radius:14px;font-family:'Noto Sans KR',sans-serif;font-weight:700;font-size:18px;padding:11px 22px;background:#546e7a;color:#fff;transition:all .13s">🗺️ 맵 새로고침</button>
  </div>

  <!-- 미로 -->
  <div style="position:relative" id="maze-wrap">
    <svg id="maze-svg"></svg>
    <div id="win-overlay">
      <div style="font-size:70px">🎉</div>
      <div style="font-size:30px;font-weight:900;color:#2e7d32;text-align:center">정말 잘 하셨어요!</div>
      <div id="win-moves" style="font-size:20px;color:#555;font-weight:700"></div>
      <div style="display:flex;gap:12px;flex-wrap:wrap;justify-content:center">
        <button class="abtn" onclick="initStage()" style="background:#eee;color:#333;padding:13px 28px;font-size:20px">🔄 다시하기</button>
        <button id="next-btn" class="abtn" onclick="nextStage()" style="padding:13px 28px;font-size:20px;color:#fff">다음 단계 →</button>
      </div>
    </div>
  </div>

  <!-- D패드 -->
  <div style="margin-top:22px;display:flex;flex-direction:column;align-items:center;gap:8px">
    <button class="dbtn" ontouchstart="move('N')" onclick="move('N')">⬆️</button>
    <div class="dpad-row">
      <button class="dbtn" ontouchstart="move('W')" onclick="move('W')">⬅️</button>
      <div style="width:78px;height:78px;display:flex;align-items:center;justify-content:center;background:#f0f0f0;border-radius:16px"><div style="width:22px;height:22px;border-radius:50%;background:#ccc"></div></div>
      <button class="dbtn" ontouchstart="move('E')" onclick="move('E')">➡️</button>
    </div>
    <button class="dbtn" ontouchstart="move('S')" onclick="move('S')">⬇️</button>
  </div>

  <div style="margin-top:14px;font-size:16px;color:#aaa;text-align:center">키보드 방향키로도 이동할 수 있어요</div>
</div>

<script>
const STAGES = [
  {id:1,cols:5,rows:5,goals:1,color:"#2e7d32",label:"1단계",desc:"길이 보여요 · 천천히 이동해 보세요",goalHide:false,orderHide:false,hideDelay:null},
  {id:2,cols:7,rows:7,goals:1,color:"#1565c0",label:"2단계",desc:"갈림길이 생겼어요 · 방향을 잘 살펴보세요",goalHide:false,orderHide:false,hideDelay:null},
  {id:3,cols:7,rows:7,goals:1,color:"#e65100",label:"3단계",desc:"목적지 위치를 3초 후 숨겨요 · 위치를 기억하세요!",goalHide:true,orderHide:false,hideDelay:3000},
  {id:4,cols:9,rows:9,goals:3,color:"#6a1e9a",label:"4단계",desc:"목적지 순서를 3초 후 숨겨요 · 순서를 기억하세요!",goalHide:false,orderHide:true,hideDelay:3000},
  {id:5,cols:9,rows:9,goals:3,color:"#b71c1c",label:"5단계",desc:"위치와 순서를 모두 3초 후 숨겨요!",goalHide:true,orderHide:true,hideDelay:3000},
];
const GOAL_LABELS=["약국","병원","집"];
const GOAL_ICONS=["💊","🏥","🏠"];
const GOAL_COLORS=["#1565c0","#c62828","#2e7d32"];
const FONT_SCALES=[0.85,1.0,1.2];
const CS_BASE=[54,46,46,40,40];

let stageIdx=0, maze=null, player={r:0,c:0}, playerDir="E";
let goals=[], currentGoal=0, trail=new Set(), status="playing", moves=0;
let memHidden=false, memCountdown=null;
let hintPath=null, hintUsed=false, hintCountdown=null;
let fsIdx=1;
let hideTimer=null, hideTick=null, hintTimer=null, hintTick=null;

function fs(){return FONT_SCALES[fsIdx];}
function CS(){return CS_BASE[stageIdx];}

function generateMaze(cols,rows){
  const cells=Array.from({length:rows},(_,r)=>Array.from({length:cols},(_,c)=>({r,c,walls:{N:true,S:true,E:true,W:true},visited:false})));
  const stack=[];const start=cells[0][0];start.visited=true;stack.push(start);
  const dirs=[{d:"N",dr:-1,dc:0,opp:"S"},{d:"S",dr:1,dc:0,opp:"N"},{d:"E",dr:0,dc:1,opp:"W"},{d:"W",dr:0,dc:-1,opp:"E"}];
  while(stack.length){
    const cur=stack[stack.length-1];
    const nb=dirs.map(({d,dr,dc,opp})=>{const nr=cur.r+dr,nc=cur.c+dc;if(nr>=0&&nr<rows&&nc>=0&&nc<cols&&!cells[nr][nc].visited)return{cell:cells[nr][nc],d,opp};return null;}).filter(Boolean);
    if(!nb.length){stack.pop();continue;}
    const{cell,d,opp}=nb[Math.floor(Math.random()*nb.length)];
    cur.walls[d]=false;cell.walls[opp]=false;cell.visited=true;stack.push(cell);
  }
  return cells;
}

function findPath(from,to){
  if(!maze||!to)return[];
  const rows=maze.length,cols=maze[0].length;
  const queue=[[from]];const visited=new Set([`${from.r},${from.c}`]);
  const dirMap={N:[-1,0],S:[1,0],E:[0,1],W:[0,-1]};
  while(queue.length){
    const path=queue.shift();const cur=path[path.length-1];
    if(cur.r===to.r&&cur.c===to.c)return path;
    for(const[dir,[dr,dc]]of Object.entries(dirMap)){
      if(maze[cur.r][cur.c].walls[dir])continue;
      const nr=cur.r+dr,nc=cur.c+dc,key=`${nr},${nc}`;
      if(!visited.has(key)&&nr>=0&&nr<rows&&nc>=0&&nc<cols){visited.add(key);queue.push([...path,{r:nr,c:nc}]);}
    }
  }
  return[];
}

function placeGoals(rows,cols,count){
  if(count===1){
    const cands=[];
    for(let r=0;r<rows;r++)for(let c=0;c<cols;c++)if(r+c>2)cands.push({r,c});
    cands.sort((a,b)=>(b.r+b.c)-(a.r+a.c));
    const topN=cands.slice(0,Math.max(1,Math.floor(cands.length*0.3)));
    return[topN[Math.floor(Math.random()*topN.length)]];
  }
  const zones=[(r,c)=>r<Math.floor(rows*0.4)&&c>Math.floor(cols*0.55),(r,c)=>r>Math.floor(rows*0.55)&&c<Math.floor(cols*0.4),(r,c)=>r>Math.floor(rows*0.6)&&c>Math.floor(cols*0.6)];
  const used=new Set(["0,0"]);const results=[];
  for(let i=0;i<count;i++){
    let cands=[];
    for(let r=0;r<rows;r++)for(let c=0;c<cols;c++)if(!used.has(`${r},${c}`)&&zones[i](r,c))cands.push({r,c});
    if(!cands.length)for(let r=0;r<rows;r++)for(let c=0;c<cols;c++)if(!used.has(`${r},${c}`))cands.push({r,c});
    const pick=cands[Math.floor(Math.random()*Math.min(4,cands.length))];
    results.push(pick);used.add(`${pick.r},${pick.c}`);
  }
  return results;
}

function initStage(){
  const s=STAGES[stageIdx];
  clearTimeout(hideTimer);clearInterval(hideTick);clearTimeout(hintTimer);clearInterval(hintTick);
  maze=generateMaze(s.cols,s.rows);
  player={r:0,c:0};playerDir="E";trail=new Set(["0,0"]);moves=0;status="playing";currentGoal=0;
  goals=placeGoals(s.rows,s.cols,s.goals);
  memHidden=false;memCountdown=s.hideDelay?Math.floor(s.hideDelay/1000):null;
  hintPath=null;hintUsed=false;hintCountdown=null;
  if(s.hideDelay){
    hideTick=setInterval(()=>{if(memCountdown>1)memCountdown--;else{clearInterval(hideTick);}renderInfo();},1000);
    hideTimer=setTimeout(()=>{memHidden=true;memCountdown=null;clearInterval(hideTick);renderInfo();renderMaze();},s.hideDelay);
  }
  renderInfo();renderMaze();updateHintBtn();
  document.getElementById('win-overlay').classList.remove('show');
}

function setFs(i){
  fsIdx=i;
  document.querySelectorAll('.fsbtn').forEach(b=>b.classList.toggle('on',+b.dataset.fs===i));
  const f=FONT_SCALES[i];
  document.getElementById('title').style.fontSize=Math.round(30*f)+'px';
  document.getElementById('subtitle').style.fontSize=Math.round(18*f)+'px';
  renderInfo();renderMaze();
}

function renderTabs(){
  const wrap=document.getElementById('stage-tabs');wrap.innerHTML='';
  STAGES.forEach((s,i)=>{
    const b=document.createElement('button');
    b.className='sbtn'+(i===stageIdx?' on':'');
    b.textContent=s.label;
    if(i===stageIdx)b.style.background=s.color;
    b.onclick=()=>{stageIdx=i;renderTabs();initStage();};
    wrap.appendChild(b);
  });
}

function renderInfo(){
  const s=STAGES[stageIdx],f=fs();
  const card=document.getElementById('info-card');
  card.style.border=`3px solid ${s.color}`;
  const lbl=document.getElementById('stage-label');
  lbl.textContent=s.label;lbl.style.color=s.color;lbl.style.fontSize=Math.round(24*f)+'px';
  const desc=document.getElementById('stage-desc');
  desc.textContent=s.desc;desc.style.fontSize=Math.round(17*f)+'px';
  document.getElementById('move-count').style.color=s.color;
  document.getElementById('move-count').style.fontSize=Math.round(22*f)+'px';
  document.getElementById('move-count').textContent=moves;

  const op=document.getElementById('order-panel');
  if(s.goals>1){
    op.style.display='block';
    if(s.orderHide&&memHidden){
      op.innerHTML=`<div style="font-size:${Math.round(19*f)}px;color:#b71c1c;font-weight:900">순서를 기억하며 찾아가세요!</div>`;
    } else {
      let html=`<div style="font-size:${Math.round(15*f)}px;color:#555;font-weight:700;margin-bottom:6px">이동 순서:</div><div style="display:flex;gap:8px;flex-wrap:wrap;align-items:center">`;
      GOAL_LABELS.slice(0,s.goals).forEach((label,i)=>{
        const done=i<currentGoal,active=i===currentGoal;
        html+=`<div style="display:flex;align-items:center;gap:5px;padding:${Math.round(6*f)}px ${Math.round(14*f)}px;border-radius:30px;background:${done?'#e8f5e9':active?GOAL_COLORS[i]:'#f0f0f0'};color:${done?'#388e3c':active?'#fff':'#aaa'};font-weight:900;font-size:${Math.round(17*f)}px;border:2.5px solid ${active?GOAL_COLORS[i]:'transparent'};transition:all .3s">${done?'✅':GOAL_ICONS[i]} ${label}</div>`;
      });
      html+='</div>';op.innerHTML=html;
    }
  } else {op.style.display='none';}

  const cb=document.getElementById('countdown-banner');
  if(memCountdown!==null){cb.style.display='block';cb.style.fontSize=Math.round(18*f)+'px';cb.textContent=`👀 ${memCountdown}초 후 숨겨져요! 잘 기억하세요!`;}
  else cb.style.display='none';

  const hb=document.getElementById('hidden-banner');
  hb.style.display=memHidden?'block':'none';
  hb.style.fontSize=Math.round(18*f)+'px';

  const nb=document.getElementById('next-btn');
  nb.style.background=s.color;
  nb.style.display=stageIdx<STAGES.length-1?'block':'none';
}

function renderMaze(){
  const s=STAGES[stageIdx],f=fs(),cs=CS();
  const rows=maze.length,cols=maze[0].length;
  const W=cols*cs+2,H=rows*cs+2;
  const svg=document.getElementById('maze-svg');
  svg.setAttribute('width',W);svg.setAttribute('height',H);
  svg.style.border=`3px solid ${s.color}`;
  let html='';
  for(let r=0;r<rows;r++){
    for(let c=0;c<cols;c++){
      const cell=maze[r][c];
      const x=c*cs+1,y=r*cs+1;
      const isPlayer=r===player.r&&c===player.c;
      const isTrail=trail.has(`${r},${c}`)&&!isPlayer;
      const isHint=hintPath&&hintPath.has(`${r},${c}`)&&!isPlayer;
      const goalIdx=goals.findIndex(g=>g.r===r&&g.c===c);
      const isPast=goalIdx>=0&&goalIdx<currentGoal;
      const isGoalCell=goalIdx>=0&&!isPast;
      const isCurrent=goalIdx===currentGoal;
      const fill=isHint?"#fff9c4":isTrail?"#ddeeff":"#fff";
      html+=`<rect x="${x}" y="${y}" width="${cs}" height="${cs}" fill="${fill}"/>`;
      if(isTrail&&!isHint)html+=`<circle cx="${x+cs/2}" cy="${y+cs/2}" r="${cs*0.09}" fill="#90caf9"/>`;
      if(isHint)html+=`<circle cx="${x+cs/2}" cy="${y+cs/2}" r="${cs*0.18}" fill="#ffca28" opacity="0.7"/>`;
      let showIcon=false,showQ=false;
      if(isGoalCell){if(!memHidden)showIcon=true;else if(!s.goalHide&&s.orderHide)showQ=true;}
      if(showIcon)html+=`<text x="${x+cs/2}" y="${y+cs/2+cs*0.19}" text-anchor="middle" font-size="${cs*0.52}">${GOAL_ICONS[goalIdx]??'⭐'}</text>`;
      if(showQ){
        html+=`<rect x="${x+cs*0.15}" y="${y+cs*0.15}" width="${cs*0.7}" height="${cs*0.7}" rx="6" fill="${isCurrent?(GOAL_COLORS[goalIdx]+'22'):'#f5f5f5'}" stroke="${isCurrent?GOAL_COLORS[goalIdx]:'#ccc'}" stroke-width="2"/>`;
        html+=`<text x="${x+cs/2}" y="${y+cs/2+cs*0.22}" text-anchor="middle" font-size="${cs*0.46}" font-weight="900" fill="${isCurrent?GOAL_COLORS[goalIdx]:'#bbb'}">?</text>`;
      }
      if(cell.walls.N)html+=`<line x1="${x}" y1="${y}" x2="${x+cs}" y2="${y}" stroke="#2a2a2a" stroke-width="2.2" stroke-linecap="square"/>`;
      if(cell.walls.S)html+=`<line x1="${x}" y1="${y+cs}" x2="${x+cs}" y2="${y+cs}" stroke="#2a2a2a" stroke-width="2.2" stroke-linecap="square"/>`;
      if(cell.walls.W)html+=`<line x1="${x}" y1="${y}" x2="${x}" y2="${y+cs}" stroke="#2a2a2a" stroke-width="2.2" stroke-linecap="square"/>`;
      if(cell.walls.E)html+=`<line x1="${x+cs}" y1="${y}" x2="${x+cs}" y2="${y+cs}" stroke="#2a2a2a" stroke-width="2.2" stroke-linecap="square"/>`;
    }
  }
  const cx=player.c*cs+1+cs/2,cy=player.r*cs+1+cs/2;
  const rad=cs*0.36,angle={N:-90,S:90,E:0,W:180}[playerDir]??0;
  const tip=rad,base=rad*0.52,hw=rad*0.44;
  html+=`<g transform="translate(${cx},${cy}) rotate(${angle})">
    <circle r="${rad*1.1}" fill="#1976d2" opacity="0.15"/>
    <polygon points="${tip},0 ${-base},${hw} ${-base*0.45},0 ${-base},${-hw}" fill="#1565c0"/>
    <circle r="${rad*0.25}" fill="#fff" cx="${-base*0.1}" cy="0"/>
  </g>`;
  svg.innerHTML=html;
  // scale dpad buttons
  document.querySelectorAll('.dbtn').forEach(b=>{b.style.width=Math.round(78*f)+'px';b.style.height=Math.round(78*f)+'px';b.style.fontSize=Math.round(34*f)+'px';});
}

function move(dir){
  if(!maze||status!=='playing')return;
  playerDir=dir;
  const cell=maze[player.r][player.c];
  if(cell.walls[dir])return;
  const[dr,dc]={N:[-1,0],S:[1,0],E:[0,1],W:[0,-1]}[dir];
  player={r:player.r+dr,c:player.c+dc};moves++;
  trail.add(`${player.r},${player.c}`);
  if(hintPath){hintPath=null;clearTimeout(hintTimer);clearInterval(hintTick);updateHintBtn();}
  const goal=goals[currentGoal];
  if(goal&&player.r===goal.r&&player.c===goal.c){
    if(currentGoal+1>=goals.length){
      status="won";
      document.getElementById('win-overlay').classList.add('show');
      document.getElementById('win-moves').textContent=`이동 횟수: ${moves}번`;
    } else {currentGoal++;}
  }
  renderInfo();renderMaze();
}

function showHint(){
  if(hintUsed||!maze||status!=='playing')return;
  const goal=goals[currentGoal];if(!goal)return;
  const path=findPath(player,goal);
  hintPath=new Set(path.map(p=>`${p.r},${p.c}`));
  hintUsed=true;hintCountdown=3;
  updateHintBtn();renderMaze();
  hintTick=setInterval(()=>{
    hintCountdown--;updateHintBtn();
    if(hintCountdown<=0){clearInterval(hintTick);clearTimeout(hintTimer);hintPath=null;renderMaze();}
  },1000);
  hintTimer=setTimeout(()=>{hintPath=null;clearInterval(hintTick);renderMaze();},3000);
}

function updateHintBtn(){
  const btn=document.getElementById('hint-btn');
  if(hintUsed){
    btn.style.background='#e0e0e0';btn.style.color='#aaa';btn.style.cursor='default';
    btn.textContent=hintCountdown?`💡 힌트 표시 중 (${hintCountdown}초)`:'💡 힌트 사용됨';
  } else {
    btn.style.background='#ff8f00';btn.style.color='#fff';btn.style.cursor='pointer';
    btn.textContent='💡 길 힌트 (1회)';
  }
}

function nextStage(){if(stageIdx<STAGES.length-1){stageIdx++;renderTabs();initStage();}}

document.addEventListener('keydown',e=>{
  if(status!=='playing')return;
  const map={ArrowUp:'N',ArrowDown:'S',ArrowLeft:'W',ArrowRight:'E',w:'N',s:'S',a:'W',d:'E'};
  const dir=map[e.key];if(!dir)return;
  e.preventDefault();move(dir);
});

renderTabs();
initStage();
</script>
</body>
</html>
