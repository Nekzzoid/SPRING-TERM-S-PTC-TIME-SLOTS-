<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>PTC Booking System</title>

<style>

/* ===== GENERAL ===== */
body{
margin:0;
font-family:Arial, sans-serif;
background:#0f172a;
color:white;
text-align:center;
}

.container{
padding:40px 20px;
}

/* ===== GOLD COUNTDOWN ===== */
.countdown-box{
font-size:40px;
font-weight:bold;
color:#ffd700;
letter-spacing:3px;
animation:goldGlow 2s infinite alternate;
}

@keyframes goldGlow{
from{ text-shadow:0 0 10px #ffd700; }
to{ text-shadow:0 0 30px #ffd700, 0 0 50px #facc15; }
}

/* ===== OVERLAY ===== */
#closedOverlay{
position:fixed;
top:0;
left:0;
width:100%;
height:100%;
background:rgba(0,0,0,0.95);
display:none;
align-items:center;
justify-content:center;
flex-direction:column;
z-index:5000;
}

#closedOverlay h1{
font-size:50px;
color:#ffd700;
animation:pulse 2s infinite;
}

@keyframes pulse{
0%{transform:scale(1);}
50%{transform:scale(1.1);}
100%{transform:scale(1);}
}

/* ===== FIREWORK CANVAS ===== */
canvas{
position:fixed;
top:0;
left:0;
pointer-events:none;
z-index:4000;
}

</style>
</head>

<body>

<div class="container">
<h2>Parent Teacher Conference Booking</h2>
<h3>Countdown to Closing</h3>
<div id="countdown" class="countdown-box"></div>
</div>

<div id="closedOverlay">
<h1>BOOKING CLOSED</h1>
<p>The booking window has officially closed.</p>
<p>Please contact the school administration.</p>
</div>

<canvas id="fireworks"></canvas>

<audio id="bellSound">
<source src="https://actions.google.com/sounds/v1/alarms/church_bells.ogg" type="audio/ogg">
</audio>

<script>

/* ===== EVENT DATE (SET YOUR SATURDAY HERE) ===== */
const eventDate = new Date("February 28, 2026 15:00:00 GMT+0100").getTime();

let closedTriggered = false;

const countdown = document.getElementById("countdown");
const overlay = document.getElementById("closedOverlay");
const bell = document.getElementById("bellSound");

setInterval(function(){

let now = new Date().getTime();
let distance = eventDate - now;

if(distance <= 0 && !closedTriggered){

closedTriggered = true;
countdown.innerHTML = "00d 00h 00m 00s";
overlay.style.display = "flex";

/* PLAY BELL */
bell.play();

/* START FIREWORKS */
startFireworks();

return;
}

if(distance > 0){

let days = Math.floor(distance / (1000*60*60*24));
let hours = Math.floor((distance % (1000*60*60*24)) / (1000*60*60));
let minutes = Math.floor((distance % (1000*60*60)) / (1000*60));
let seconds = Math.floor((distance % (1000*60)) / 1000);

countdown.innerHTML =
`${String(days).padStart(2,'0')}d 
${String(hours).padStart(2,'0')}h 
${String(minutes).padStart(2,'0')}m 
${String(seconds).padStart(2,'0')}s`;

}

},1000);


/* ===== FIREWORKS SYSTEM ===== */

const canvas = document.getElementById("fireworks");
const ctx = canvas.getContext("2d");

canvas.width = window.innerWidth;
canvas.height = window.innerHeight;

let particles = [];

function random(min,max){
return Math.random()*(max-min)+min;
}

function createFirework(){

let x = random(100,canvas.width-100);
let y = random(100,canvas.height/2);

for(let i=0;i<80;i++){
particles.push({
x:x,
y:y,
radius:2,
color:`hsl(${random(0,360)},100%,50%)`,
speedX:random(-5,5),
speedY:random(-5,5),
alpha:1
});
}

}

function startFireworks(){

setInterval(createFirework,800);

animate();

}

function animate(){

requestAnimationFrame(animate);

ctx.fillStyle="rgba(0,0,0,0.2)";
ctx.fillRect(0,0,canvas.width,canvas.height);

particles.forEach((p,index)=>{

p.x+=p.speedX;
p.y+=p.speedY;
p.alpha-=0.01;

ctx.beginPath();
ctx.arc(p.x,p.y,p.radius,0,Math.PI*2);
ctx.fillStyle=`rgba(${hexToRgb(p.color)},${p.alpha})`;
ctx.fill();

if(p.alpha<=0){
particles.splice(index,1);
}

});

}

/* Convert HSL to RGB helper */
function hexToRgb(color){
let c = document.createElement("canvas").getContext("2d");
c.fillStyle = color;
let rgb = c.fillStyle.match(/\d+/g);
return rgb[0]+","+rgb[1]+","+rgb[2];
}

</script>

</body>
</html>
