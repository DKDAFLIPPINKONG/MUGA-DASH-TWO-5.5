<!DOCTYPE html>
<html>
<head>
<title>MUGA DASH 5.5 TWO</title>
<style>
body{margin:0;overflow:hidden;font-family:Arial;background:#111;}
canvas{display:none;background:#111;}
#menu, #deathMenu, #leaderboardMenu, #highscoreInputMenu{
  position:absolute;
  width:100%;
  height:100%;
  display:flex;
  flex-direction:column;
  justify-content:center;
  align-items:center;
  background:#0b1f3a;
  z-index:10;
}
#title{color:white;font-size:50px;font-weight:bold;margin-bottom:40px;text-align:center;}
button{
  font-size:30px;padding:15px 40px;cursor:pointer;border:none;border-radius:8px;
  background:#4cafef;color:white;margin:10px;
}
#buttonRow{display:flex;flex-direction:row;}
ul{list-style:none;color:white;font-size:22px;padding:0;text-align:left;}
input[type=text]{font-size:25px;padding:10px;width:300px;margin:15px;}
</style>
<body>

<div id="menu">
  <div id="title">MUGA DASH 5.5 TWO</div>
  <div id="buttonRow">
    <button id="easyBtn">Easy</button>
    <button id="mediumBtn">Medium</button>
    <button id="hardBtn">Hard</button>
  </div>
</div>

<div id="deathMenu" style="display:none;">
  <div id="title">You Died!</div>
  <button id="restartBtn">Restart</button>
  <button id="backMenuBtn">Back to Menu</button>
  <button id="leaderboardBtn">Leaderboard</button>
</div>

<div id="leaderboardMenu" style="display:none;">
  <div id="title">Leaderboard</div>
  <input type="text" id="nameInput" placeholder="Type your name and press Enter" />
  <ul id="leaderboardList"></ul>
  <button id="closeInputBtn">Close</button>
</div>

<canvas id="game"></canvas>

<script>
const canvas=document.getElementById("game");
const ctx=canvas.getContext("2d");

const menu=document.getElementById("menu");
const easyBtn=document.getElementById("easyBtn");
const mediumBtn=document.getElementById("mediumBtn");
const hardBtn=document.getElementById("hardBtn");

const deathMenu=document.getElementById("deathMenu");
const restartBtn=document.getElementById("restartBtn");
const backMenuBtn=document.getElementById("backMenuBtn");
const leaderboardBtn=document.getElementById("leaderboardBtn");

const leaderboardMenu=document.getElementById("leaderboardMenu");
const leaderboardList=document.getElementById("leaderboardList");
const nameInput=document.getElementById("nameInput");
const closeInputBtn=document.getElementById("closeInputBtn");

canvas.width=window.innerWidth;
canvas.height=window.innerHeight;

let gravity=0.6,speed=6,spikeChance=0.3;
let platforms=[],player,lastPlatform,score=0,gameActive=false,platformCount=0;
let currentDifficulty="easy";
let nameSubmitted=false;
let canSubmitName=false;

let leaderboard={easy:[],medium:[],hard:[]};
let badWords=["fuck","shit","bitch","damn","ass","cunt","nigger","nigga","slut","dick"];

function saveLeaderboard(){localStorage.setItem("mugaLeaderboard",JSON.stringify(leaderboard));}

function spawnPlatform(){
    let maxGap=80,maxHeightChange=35;
    let newX=lastPlatform.x+lastPlatform.width+(Math.random()*maxGap+40);
    let newY=lastPlatform.y+(Math.random()*maxHeightChange*2-maxHeightChange);
    let minH=canvas.height/2,maxH=canvas.height-80;
    newY=Math.max(minH,Math.min(maxH,newY));
    let p={x:newX,y:newY,width:220,height:22,spike:null};
    platformCount++;
    if(platformCount>3&&Math.random()<spikeChance){p.spike={x:Math.random()*(p.width-20),y:-20,width:20,height:20};}
    platforms.push(p);
    lastPlatform=p;
}

function initGame(difficulty){
    gameActive=true;
    deathMenu.style.display="none";
    leaderboardMenu.style.display="none";
    menu.style.display="none";
    canvas.style.display="block";
    currentDifficulty=difficulty;
    nameSubmitted=false;
    canSubmitName=false;
    nameInput.value="";
    if(difficulty==="easy"){speed=6;spikeChance=0.3;}
    else if(difficulty==="medium"){speed=8;spikeChance=0.4;}
    else if(difficulty==="hard"){speed=10;spikeChance=0.5;}
    platforms=[];score=0;platformCount=0;
    let startPlatform={x:150,y:canvas.height-150,width:260,height:22,spike:null};
    platforms.push(startPlatform);
    player={x:180,y:startPlatform.y-25,width:25,height:25,vy:0,grounded:true};
    lastPlatform=startPlatform;
    for(let i=0;i<12;i++) spawnPlatform();
}

function jump(){if(player&&player.grounded){player.vy=-11;player.grounded=false;}}
document.addEventListener("keydown",e=>{if(e.code==="Space") jump();});
document.addEventListener("mousedown",jump);

function backToMenu(){gameActive=false;canvas.style.display="none";deathMenu.style.display="none";menu.style.display="flex";}

function checkHighscore(){
    let lb=leaderboard[currentDifficulty];
    if(lb.length<10||score>lb[lb.length-1].score){
        canSubmitName=true;
        leaderboardMenu.style.display="flex";
        updateLeaderboardDisplay();
    }else{
        showDeathMenu();
    }
}

function submitName(){
    if(!canSubmitName||nameSubmitted) return;
    let name=nameInput.value.trim();
    if(!name) name="Femboy";
    name=name.toLowerCase();
    for(let word of badWords){if(name.includes(word)){name="Femboy";break;}}
    name=name.charAt(0).toUpperCase()+name.slice(1);
    leaderboard[currentDifficulty].push({name:name,score:score});
    leaderboard[currentDifficulty].sort((a,b)=>b.score-a.score);
    if(leaderboard[currentDifficulty].length>10) leaderboard[currentDifficulty].pop();
    saveLeaderboard();
    nameSubmitted=true;
    updateLeaderboardDisplay();
}

function updateLeaderboardDisplay(){
    leaderboardList.innerHTML="";
    let lb=leaderboard[currentDifficulty];
    if(lb.length===0) leaderboardList.innerHTML="<li>No scores yet!</li>";
    else{lb.forEach((entry,i)=>{leaderboardList.innerHTML+=`<li>${i+1}. ${entry.name} - ${entry.score}</li>`;});}
}

closeInputBtn.onclick=()=>{
    leaderboardMenu.style.display="none";
    deathMenu.style.display="flex";
    leaderboardBtn.style.display="none";
    canSubmitName=false;
};
nameInput.addEventListener("keypress",e=>{if(e.key==="Enter") submitName();});

function showDeathMenu(){
    deathMenu.style.display="flex";
    leaderboardBtn.style.display="inline-block";
}

function update(){
    if(!gameActive||!player) return;
    player.vy+=gravity;
    player.y+=player.vy;
    player.grounded=false;
    platforms.forEach(p=>{
        p.x-=speed;
        if(p.spike) p.spikeScreenX=p.x+p.spike.x;
        if(player.x<p.x+p.width&&player.x+player.width>p.x&&player.y+player.height<=p.y+10&&player.y+player.height+player.vy>=p.y){
            player.y=p.y-player.height;player.vy=0;player.grounded=true;
        }
        if(p.spike){
            let sx=p.spikeScreenX,sy=p.y-p.spike.height;
            if(player.x<sx+p.spike.width&&player.x+player.width>sx&&player.y<sy+p.spike.height&&player.y+player.height>sy){
                if(!nameSubmitted) checkHighscore(); else showDeathMenu();
            }
        }
    });
    if(platforms[platforms.length-1].x<canvas.width+200) spawnPlatform();
    platforms=platforms.filter(p=>p.x+p.width>0);
    if(player.y>canvas.height){if(!nameSubmitted) checkHighscore(); else showDeathMenu();}
}

function draw(){
    if(!player||!gameActive) return;
    ctx.clearRect(0,0,canvas.width,canvas.height);
    ctx.fillStyle="white";
    ctx.fillRect(player.x,player.y,player.width,player.height);
    platforms.forEach(p=>{
        ctx.fillStyle="cyan";
        ctx.fillRect(p.x,p.y,p.width,p.height);
        if(p.spike){
            let sx=p.spikeScreenX,sy=p.y-p.spike.height;
            ctx.fillStyle="red";
            ctx.beginPath();
            ctx.moveTo(sx,sy+p.spike.height);
            ctx.lineTo(sx+p.spike.width/2,sy);
            ctx.lineTo(sx+p.spike.width,sy+p.spike.height);
            ctx.closePath();ctx.fill();
        }
    });
    ctx.fillStyle="yellow";
    ctx.font="30px Arial";
    ctx.fillText(`Score (${currentDifficulty}): ${score}`,20,40);
}

function loop(){update();draw();requestAnimationFrame(loop);}
loop();
setInterval(()=>{if(gameActive) score+=1;},1000);

// Buttons
 easyBtn.onclick=()=>{initGame("easy");};
 mediumBtn.onclick=()=>{initGame("medium");};
 hardBtn.onclick=()=>{initGame("hard");};
 restartBtn.onclick=()=>{initGame(currentDifficulty);};
 backMenuBtn.onclick=backToMenu;
 leaderboardBtn.onclick=()=>{
    deathMenu.style.display="none";
    leaderboardMenu.style.display="flex";
    nameInput.style.display="none";
    submitNameBtn.style.display="none";
    updateLeaderboardDisplay();
};
</script>

</body>
</html>

