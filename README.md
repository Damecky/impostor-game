<!DOCTYPE html>
<html lang="pl">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
<title>🕵️‍♂️ Impostor Game</title>
<style>
body {
  font-family: 'Segoe UI', sans-serif;
  background: #0d1117;
  color: #fff;
  text-align: center;
  margin: 0;
  padding: 0;
}
h1 { color: #00d26a; margin-top: 30px; font-size: 28px; }
button {
  background: #00d26a;
  color: black;
  border: none;
  border-radius: 12px;
  padding: 20px 40px;
  font-size: 20px;
  margin: 15px 0;
  cursor: pointer;
  width: 80%;
  max-width: 300px;
}
button:hover { background: #00b85d; }
input {
  font-size: 20px;
  padding: 15px;
  border-radius: 10px;
  border: 1px solid #ccc;
  width: 80%;
  max-width: 300px;
  text-align: center;
  margin-bottom: 15px;
}
.hidden { display: none; }
.card {
  margin-top: 50px;
  padding: 30px;
  background: #161b22;
  border-radius: 20px;
  display: inline-block;
  font-size: 24px;
  max-width: 90%;
  word-wrap: break-word;
}
</style>
</head>
<body>
<h1>🕵️‍♂️ Impostor Game</h1>

<div id="menu">
  <input id="playerName" placeholder="Twoje imię" />
  <button id="btnCreate">Stwórz grę</button>
  <p style="margin:10px 0;">lub</p>
  <input id="joinCode" placeholder="Kod pokoju" />
  <button id="btnJoin">Dołącz</button>
</div>

<div id="hostPanel" class="hidden">
  <h2>Kod pokoju: <span id="roomCode"></span></h2>
  <p id="players">Gracze: 0/4</p>
  <button id="btnStart">Rozdaj role</button>
  <button id="btnNewWord" class="hidden">Nowa runda</button>
</div>

<div id="waiting" class="hidden">
  <h2>Oczekiwanie na rozpoczęcie...</h2>
</div>

<div id="result" class="hidden card"></div>

<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-database-compat.js"></script>

<script>
const firebaseConfig = {
  apiKey: "AIzaSyAy1mE_Q3fJo9W2Aa9EQUqp0L0Bn53XPHc",
  authDomain: "impostor-game-ebc12.firebaseapp.com",
  databaseURL: "https://impostor-game-ebc12-default-rtdb.europe-west1.firebasedatabase.app/",
  projectId: "impostor-game-ebc12",
  storageBucket: "impostor-game-ebc12.appspot.com",
  messagingSenderId: "561452405867",
  appId: "1:561452405867:web:24fb4cdf0320c2c3488d2e",
  measurementId: "G-DKBLTBFHJ5"
};
firebase.initializeApp(firebaseConfig);
const db = firebase.database();

const words = [
  "TikTok","mem","Netflix","lody","rower","żaba","cebula","pączek",
  "pizza","drama","uwu","sigma","snap","Discord","YouTube","szkoła",
  "tramwaj","wakacje","park","kot","pies","telefon","komputer","fryzura",
  "emoji","Chad","Sigma face","ban","flex","kanapka","długopis","kaktus",
  "drip","domówka","herbata","miłość","gra","keyboard","film","sport",
  "pralka","sushi","team","keyboard","muzyka","Spotify","pociąg"
];

let playerId = Math.random().toString(36).substring(2,9);
let roomRef = null;
let isHost = false;
let playerName = "";

function randomCode() {
  return Math.floor(1000 + Math.random() * 9000).toString();
}

// Tworzenie pokoju
window.createRoom = async function() {
  playerName = document.getElementById("playerName").value.trim();
  if (!playerName) return alert("Podaj swoje imię!");
  
  isHost = true;
  const code = randomCode();
  roomRef = db.ref("rooms/" + code);
  await roomRef.set({ players: {}, started: false });

  document.getElementById("menu").classList.add("hidden");
  document.getElementById("hostPanel").classList.remove("hidden");
  document.getElementById("roomCode").innerText = code;

  const playerRef = db.ref(`rooms/${code}/players/${playerId}`);
  await playerRef.set({ name: playerName, ready: true });

  roomRef.on("value", snapshot => {
    const data = snapshot.val();
    const playersList = data.players ? Object.values(data.players).map(p => p.name) : [];
    document.getElementById("players").innerText = `Gracze: ${playersList.length}/4 (${playersList.join(", ")})`;
    document.getElementById("btnStart").disabled = playersList.length !== 4;
  });
}

// Dołączanie do pokoju
window.joinRoom = async function() {
  playerName = document.getElementById("playerName").value.trim();
  if (!playerName) return alert("Podaj swoje imię!");
  
  const code = document.getElementById("joinCode").value.trim();
  if (!code) return alert("Podaj kod pokoju!");
  roomRef = db.ref("rooms/" + code);

  const snapshot = await roomRef.get();
  if (!snapshot.exists()) return alert("Pokój nie istnieje!");

  const playerRef = db.ref(`rooms/${code}/players/${playerId}`);
  await playerRef.set({ name: playerName, ready: true });

  document.getElementById("menu").classList.add("hidden");
  document.getElementById("waiting").classList.remove("hidden");

  roomRef.on("value", snap => {
    const data = snap.val();
    if (data.started && data.roles && data.word) {
      const role = data.roles[playerId];
      showResult(role === "impostor" ? "🕵️‍♂️ JESTEŚ IMPOSTOREM!" : `🔤 Hasło: ${data.word}`);
    }
  });
}

// Rozpoczęcie gry / pierwsza runda
window.startGame = async function() {
  await startRound();
}

// Nowa runda
window.startRound = async function() {
  const snapshot = await roomRef.get();
  const data = snapshot.val();
  const players = Object.keys(data.players || {});
  if (players.length !== 4) return alert("Musi być dokładnie 4 graczy!");

  const word = words[Math.floor(Math.random() * words.length)];
  const impostor = players[Math.floor(Math.random() * players.length)];
  const roles = {};
  players.forEach(p => roles[p] = p === impostor ? "impostor" : "normal");

  await roomRef.update({ started: true, roles, word });
  showResult("Hasło zostało rozlosowane! 📲 Sprawdź swoje ekrany.");

  // Pokaż przycisk hosta do nowej rundy
  if (isHost) document.getElementById("btnNewWord").classList.remove("hidden");
}

// Funkcja do wyświetlania wyniku / hasła
function showResult(text) {
  document.getElementById("hostPanel").classList.add("hidden");
  document.getElementById("waiting").classList.add("hidden");
  const res = document.getElementById("result");
  res.innerText = text;
  res.classList.remove("hidden");
}

document.getElementById("btnCreate").addEventListener("click", createRoom);
document.getElementById("btnJoin").addEventListener("click", joinRoom);
document.getElementById("btnStart").addEventListener("click", startGame);
document.g

