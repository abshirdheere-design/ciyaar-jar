[index.html](https://github.com/user-attachments/files/25851170/index.html)
<!DOCTYPE html>
<html lang="so">
<head>
<meta charset="UTF-8">
<title>Ciyaarta Godadka 5x5</title>
<style>
body{font-family:Arial;text-align:center;margin-top:20px;}
#board{display:grid;grid-template-columns:repeat(5,80px);gap:10px;justify-content:center;}
.cell{width:80px;height:80px;border:2px solid black;border-radius:50%;cursor:pointer;}
.red{background:red;}
.blue{background:blue;}
.selected{outline:4px solid yellow;}
.move{background:#90ee90;}
#winnerScreen{position:fixed;top:0;left:0;width:100%;height:100%;background:white;display:none;align-items:center;justify-content:center;flex-direction:column;font-size:50px;font-weight:bold;}
#berberText{position:fixed;top:40%;left:50%;transform:translate(-50%,-50%);font-size:60px;font-weight:bold;color:orange;display:none;}
#playerNameScreen{position:fixed;top:0;left:0;width:100%;height:100%;background:white;display:flex;align-items:center;justify-content:center;flex-direction:column;font-size:25px;}
button{font-size:20px;padding:10px 20px;margin:10px;}
input{font-size:20px;padding:5px;margin:5px;text-align:center;}
</style>
</head>
<body>

<div id="playerNameScreen">
<h2>Magacaaga geli:</h2>
<input type="text" id="playerName" placeholder="Guest">
<button onclick="startGame()">Bilaab</button>
</div>

<h2>Ciyaarta Godadka 5x5</h2>
<p id="status"></p>
<div id="board"></div>
<div id="berberText">BERBER DHAC!</div>

<div id="winnerScreen">
<div id="winnerText"></div>
<div style="font-size:25px;margin-top:20px;">Dib ma u bilaabaysaa?</div>
<div>
<button onclick="restartGame()">Haa</button>
<button onclick="stopGame()">Maya</button>
</div>
</div>

<script>
let playerName="";
function startGame(){
  const nameInput=document.getElementById("playerName");
  playerName=nameInput.value || "Guest";
  document.getElementById("playerNameScreen").style.display="none";
  draw();
}

// Ciyaarta variables
const SIZE=25, WIDTH=5;
let board=new Array(SIZE).fill(null);
let phase="placing", currentPlayer="red";
let redPlaced=0, bluePlaced=0, placeCount=0;
let selected=null, moves=[];

// DOM
const boardDiv=document.getElementById("board");

// Draw board
function draw(){
  boardDiv.innerHTML="";
  for(let i=0;i<SIZE;i++){
    let cell=document.createElement("div");
    cell.className="cell";
    if(board[i]=="red") cell.classList.add("red");
    if(board[i]=="blue") cell.classList.add("blue");
    if(i===selected) cell.classList.add("selected");
    if(moves.includes(i)) cell.classList.add("move");
    cell.onclick=()=>clickCell(i);
    boardDiv.appendChild(cell);
  }

  let text=phase=="placing" ? `Dhigista: ${currentPlayer} (${placeCount}/2)` : `Turn: ${currentPlayer}`;
  document.getElementById("status").innerText=text;
}

function clickCell(i){
  if(phase=="placing") placePiece(i);
  else movePiece(i);
  draw();
}

// Dhigista
function placePiece(i){
  if(board[i]!=null) return;
  board[i]=currentPlayer;
  placeCount++;
  if(currentPlayer=="red") redPlaced++; else bluePlaced++;
  if(placeCount==2){ placeCount=0; switchPlayer(); }
  if(redPlaced>=12 && bluePlaced>=12){ phase="move"; currentPlayer="red"; checkNoMoveLose(); }
}

// Dhaqdhaqaaqa
function movePiece(i){
  if(board[i]===currentPlayer){ selected=i; moves=getMoves(i); return; }
  if(selected!=null && moves.includes(i)){
    let old=selected;
    board[i]=currentPlayer; board[old]=null;

    // Check capture / fusad
    let captured=capture(i);

    // Haddii uu waab galo (hal god uu dhaqaaqi karo)
    let nextMoves=getMoves(i);
    if(!captured && nextMoves.length===1){
      if(confirm("WAAB AYAAD GASHAY! BERBER DHAC! Ma rabtaa inaad ka baxdo waabka?")){
        // Haa, sii soco
        selected=i; moves=nextMoves; return;
      } else {
        // Maya, dib u bilaab ciyaarta
        restartGame(); return;
      }
    }

    selected=null; moves=[];
    checkWinner();
    switchPlayer();
    checkNoMoveLose();
  }
}

// Hel moves
function getMoves(pos){
  let list=[];
  let row=Math.floor(pos/WIDTH), col=pos%WIDTH;
  let dirs=[[1,0],[-1,0],[0,1],[0,-1]];
  for(let [dr,dc] of dirs){
    let r=row+dr, c=col+dc;
    if(r>=0 && r<WIDTH && c>=0 && c<WIDTH){
      let idx=r*WIDTH+c;
      if(board[idx]==null) list.push(idx);
    }
  }
  return list;
}

function capture(pos){
  let didCapture=false;
  let row=Math.floor(pos/WIDTH), col=pos%WIDTH;
  let dirs=[[1,0],[-1,0],[0,1],[0,-1]];
  for(let [dr,dc] of dirs){
    let r1=row+dr, c1=col+dc, r2=row+2*dr, c2=col+2*dc;
    if(r2>=0 && r2<WIDTH && c2>=0 && c2<WIDTH){
      let mid=r1*WIDTH+c1, end=r2*WIDTH+c2;
      if(board[mid] && board[mid]!=currentPlayer && board[end]===currentPlayer){
        board[mid]=null; didCapture=true;
      }
    }
  }
  if(didCapture) showBerber();
  return didCapture;
}

function showBerber(){
  let text=document.getElementById("berberText");
  text.style.display="block";
  setTimeout(()=>{text.style.display="none";},1500);
}

function countPieces(player){
  return board.filter(p=>p===player).length;
}

function checkWinner(){
  if(countPieces("red")<=1) showWinner("Blue");
  if(countPieces("blue")<=1) showWinner("Red");
}

function hasAnyMove(player){
  return board.some((cell,i)=>cell===player && getMoves(i).length>0);
}

function checkNoMoveLose(){
  if(!hasAnyMove(currentPlayer)){
    let winner=currentPlayer==="red"?"Blue":"Red";
    showWinner(winner);
  }
}

function showWinner(player){
  document.getElementById("winnerText").innerText="🏆 "+player.toUpperCase()+" AYAA GUULEYSTAY";
  document.getElementById("winnerScreen").style.display="flex";
}

function restartGame(){ location.reload(); }
function stopGame(){ document.getElementById("winnerText").innerText="Ciyaarta waa la joojiyay"; }
function switchPlayer(){ currentPlayer=currentPlayer==="red"?"blue":"red"; }

draw();
</script>
</body>
</html>
