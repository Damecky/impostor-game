<!DOCTYPE html>
<html lang="pl">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Impostor Game</title>
  <style>
    body {
      font-family: 'Segoe UI', sans-serif;
      text-align: center;
      background: #0d1117;
      color: #fff;
      margin: 0;
      padding: 0;
    }
    h1 { color: #00d26a; }
    button {
      background: #00d26a;
      color: black;
      border: none;
      border-radius: 8px;
      padding: 10px 20px;
      font-size: 18px;
      margin: 10px;
      cursor: pointer;
    }
    button:hover { background: #00b85d; }
    input {
      font-size: 18px;
      padding: 10px;
      border-radius: 6px;
      border: 1px solid #ccc;
      width: 150px;
      text-align: center;
    }
    .hidden { display: none; }
    .card {
      margin-top: 50px;
      padding: 30px;
      background: #161b22;
      border-radius: 12px;
      display: inline-block;
    }
  </style>
</head>
<body>
  <h1>🕵️‍♂️ Gra Impostor</h1>

  <div id="menu">
    <button onclick="createRoom()">Stwórz grę</button>
    <br>
    <p>lub</p>
    <input id="joinCode" placeholder="Kod pokoju">
    <button onclick="joinRoom()">Dołącz</button>
  </div>

  <div id="hostPanel" class="hidden">
    <h2>Kod pokoju: <span id="roomCode"></span></h2>
    <p id="players">Gracze: 0/4</p>
    <button onclick="startGame()">Rozdaj role</button>
  </div>

  <div id="waiting" class="hidden">
    <h2>Oczekiwanie na rozpoczęcie...</h2>
  </div>

  <div id="result" class="hidden card"></div>

  <script type="module">
    import { initializeApp } from "https://www.gstatic.com/firebasejs/11.0.1/firebase-app.js";
    import { getDatabase, ref, set, get, onValue, update, push } from "https://www.gstatic.com/firebasejs/11.0.1/firebase-database.js";

    // Import the functions you need from the SDKs you need
import { initializeApp } from "firebase/app";
import { getAnalytics } from "firebase/analytics";
// TODO: Add SDKs for Firebase products that you want to use
// https://firebase.google.com/docs/web/setup#available-libraries

// Your web app's Firebase configuration
// For Firebase JS SDK v7.20.0 and later, measurementId is optional
const firebaseConfig = {
  apiKey: "AIzaSyAy1mE_Q3fJo9W2Aa9EQUqp0L0Bn53XPHc",
  authDomain: "impostor-game-ebc12.firebaseapp.com",
  projectId: "impostor-game-ebc12",
  storageBucket: "impostor-game-ebc12.firebasestorage.app",
  messagingSenderId: "561452405867",
  appId: "1:561452405867:web:24fb4cdf0320c2c3488d2e",
  measurementId: "G-DKBLTBFHJ5"
};

// Initialize Firebase
const app = initializeApp(firebaseConfig);
const analytics = getAnalytics(app);

    const words = [
      "TikTok", "mem", "herbata", "rower", "żaba", "Netflix", "sushi", "domówka", "szkoła",
      "Instagram", "mleko", "pralka", "pizza", "samochód", "cebula", "długopis", "fryzura",
      "kaktus", "Discord", "YouTube", "gra", "zegar", "wakacje", "dżem", "kanapka",
      "Chad", "Sigma", "cringe", "uwu", "sigma face", "drama", "ban", "flex", "emoji",
      "keyboard", "gimby", "team", "snap", "lody", "tramwaj", "pies", "kot", "park", "miłość"
    ];

    let playerId = Math.random().toString(36).substring(2, 9);
    let roomRef = null;
    let isHost = false;

    function randomCode() {
      return Math.floor(1000 + Math.random() * 9000).toString();
    }

    window.createRoom = async function() {
      isHost = true;
      const code = randomCode();
      roomRef = ref(db, "rooms/" + code);
      await set(roomRef, {
        players: {},
        started: false
      });
      document.getElementById("menu").classList.add("hidden");
      document.getElementById("hostPanel").classList.remove("hidden");
      document.getElementById("roomCode").innerText = code;

      const playerRef = ref(db, `rooms/${code}/players/${playerId}`);
      set(playerRef, { ready: true });

      onValue(roomRef, (snapshot) => {
        const data = snapshot.val();
        const count = data.players ? Object.keys(data.players).length : 0;
        document.getElementById("players").innerText = `Gracze: ${count}/4`;
      });
    }

    window.joinRoom = async function() {
      const code = document.getElementById("joinCode").value.trim();
      if (!code) return alert("Podaj kod pokoju!");
      roomRef = ref(db, "rooms/" + code);

      const snapshot = await get(roomRef);
      if (!snapshot.exists()) return alert("Pokój nie istnieje!");

      const playerRef = ref(db, `rooms/${code}/players/${playerId}`);
      await set(playerRef, { ready: true });

      document.getElementById("menu").classList.add("hidden");
      document.getElementById("waiting").classList.remove("hidden");

      onValue(roomRef, (snapshot) => {
        const data = snapshot.val();
        if (data.started && data.roles && data.word) {
          const role = data.roles[playerId];
          showResult(role === "impostor" ? "JESTEŚ IMPOSTOREM 🕵️‍♂️" : `Hasło: ${data.word}`);
        }
      });
    }

    window.startGame = async function() {
      const snap = await get(roomRef);
      const data = snap.val();
      const players = Object.keys(data.players || {});
      if (players.length !== 4) {
        alert("Musi być dokładnie 4 graczy!");
        return;
      }

      const word = words[Math.floor(Math.random() * words.length)];
      const impostor = players[Math.floor(Math.random() * players.length)];
      const roles = {};
      players.forEach(p => roles[p] = p === impostor ? "impostor" : "normal");

      await update(roomRef, { started: true, roles, word });

      showResult(`Hasło zostało rozlosowane! Sprawdź na swoim telefonie.`);
    }

    function showResult(text) {
      document.getElementById("hostPanel").classList.add("hidden");
      document.getElementById("waiting").classList.add("hidden");
      const res = document.getElementById("result");
      res.innerText = text;
      res.classList.remove("hidden");
    }
  </script>
</body>
</html>
