<!DOCTYPE html>
<html lang="pl">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>🕵️‍♂️ Gra Impostor</title>
  <style>
    body {
      font-family: 'Segoe UI', sans-serif;
      text-align: center;
      background: #0d1117;
      color: #fff;
      margin: 0;
      padding: 0;
    }
    h1 { color: #00d26a; margin-top: 30px; }
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
      font-size: 22px;
    }
  </style>
</head>
<body>
  <h1>🕵️‍♂️ Gra Impostor</h1>

  <div id="menu">
    <button id="btnCreate">Stwórz grę</button>
    <p>lub</p>
    <input id="joinCode" placeholder="Kod pokoju">
    <button id="btnJoin">Dołącz</button>
  </div>

  <div id="hostPanel" class="hidden">
    <h2>Kod pokoju: <span id="roomCode"></span></h2>
    <p id="players">Gracze: 0/4</p>
    <button id="btnStart">Rozdaj role</button>
  </div>

  <div id="waiting" class="hidden">
    <h2>Oczekiwanie na rozpoczęcie...</h2>
  </div>

  <div id="result" class="hidden card"></div>

  <!-- Firebase CDN -->
  <script src="https://www.gstatic.com/firebasejs/11.0.1/firebase-app.js"></script>
  <script src="https://www.gstatic.com/firebasejs/11.0.1/firebase-database.js"></script>

  <script>
    // Konfiguracja Firebase
    const firebaseConfig = {
      apiKey: "AIzaSyAy1mE_Q3fJo9W2Aa9EQUqp0L0Bn53XPHc",
      authDomain: "impostor-game-ebc12.firebaseapp.com",
      projectId: "impostor-game-ebc12",
      storageBucket: "impostor-game-ebc12.appspot.com",
      messagingSenderId: "561452405867",
      appId: "1:561452405867:web:24fb4cdf0320c2c3488d2e",
      measurementId: "G‑DKBLTBFHJ5",
      databaseURL: "https://impostor-game-ebc12-default-rtdb.europe-west1.firebasedatabase.app/"
    };
    firebase.initializeApp(firebaseConfig);
    const db = firebase.database();

    // Lista haseł
    const words = [
      "TikTok", "mem", "Netflix", "lody", "rower", "żaba", "cebula", "pączek",
      "pizza", "drama", "uwu", "sigma", "snap", "Discord", "YouTube", "szkoła",
      "tramwaj", "wakacje", "park", "kot", "pies", "telefon", "komputer", "fryzura",
      "emoji", "Chad", "Sigma face", "ban", "flex", "kanapka", "długopis", "kaktus",
      "drip", "domówka", "herbata", "miłość", "gra", "keyboard", "film", "sport",
      "pralka", "sushi", "team", "keyboard", "muzyka", "Spotify", "pociąg"
    ];

    let playerId = Math.random().toString(36).substring(2, 9);
    let roomRef = null;
    let isHost = false;

    function randomCode() {
      return Math.floor(1000 + Math.random() * 9000).toString();
    }

    // Funkcja globalna
    window.createRoom = async function() {
      isHost = true;
      const code = randomCode();
      roomRef = db.ref("rooms/" + code);
      await roomRef.set({ players: {}, started: false });

      document.getElementById("menu").classList.add("hidden");
      document.getElementById("hostPanel").classList.remove("hidden");
      document.getElementById("roomCode").innerText = code;

      const playerRef = db.ref(`rooms/${code}/players/${playerId}`);
      await playerRef.set({ ready: true });

      roomRef.on("value", snapshot => {
        const data = snapshot.val();
        const count = data.players ? Object.keys(data.players).length : 0;
        document.getElementById("players").innerText = `Gracze: ${count}/4`;
      });
    }

    window.joinRoom = async function() {
      const code = document.getElementById("joinCode").value.trim();
      if (!code) return alert("Podaj kod pokoju!");
      roomRef = db.ref("rooms/" + code);

      roomRef.get().then(snapshot => {
        if (!snapshot.exists()) return alert("Pokój nie istnieje!");

        const playerRef = db.ref(`rooms/${code}/players/${playerId}`);
        playerRef.set({ ready: true });

        document.getElementById("menu").classList.add("hidden");
        document.getElementById("waiting").classList.remove("hidden");

        roomRef.on("value", snap => {
          const data = snap.val();
          if (data.started && data.roles && data.word) {
            const role = data.roles[playerId];
            showResult(role === "impostor" ? "🕵️‍♂️ JESTEŚ IMPOSTOREM!" : `🔤 Hasło: ${data.word}`);
          }
        });
      });
    }

    window.startGame = async function() {
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
    }

    function showResult(text) {
      document.getElementById("hostPanel").classList.add("hidden");
      document.getElementById("waiting").classList.add("hidden");
      const res = document.getElementById("result");
      res.innerText = text;
      res.classList.remove("hidden");
    }

    // Przycisk start
    document.getElementById("btnCreate").addEventListener("click", createRoom);
    document.getElementById("btnJoin").addEventListener("click", joinRoom);
    document.getElementById("btnStart").addEventListener("click", startGame);
  </script>
</body>
</html>
