<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Real 2-Person WhatsApp Clone</title>
  <script src="https://www.gstatic.com/firebasejs/9.22.0/firebase-app-compat.js"></script>
  <script src="https://www.gstatic.com/firebasejs/9.22.0/firebase-database-compat.js"></script>
  <style>
    body {
      margin: 0; padding: 0; font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
      height: 100vh; background: #0b141a; display: flex; justify-content: center; align-items: center;
    }
    .chat {
      width: 380px; height: 680px; background: #111b21; color: white;
      border-radius: 16px; overflow: hidden; box-shadow: 0 10px 30px rgba(0,0,0,0.5);
      display: flex; flex-direction: column;
    }
    .header {
      background: #1f2c34; padding: 15px; text-align: center; font-size: 18px;
    }
    .name-input { text-align: center; padding: 20px; background: #111b21; }
    .name-input input {
      padding: 12px; width: 80%; border: none; border-radius: 8px; font-size: 16px;
    }
    .name-input button {
      padding: 12px 20px; background: #00a884; color: white; border: none; border-radius: 8px; margin-top: 10px; cursor: pointer;
    }
    .messages {
      flex: 1; overflow-y: auto; padding: 10px; background: #0b141a;
      display: flex; flex-direction: column; gap: 8px;
    }
    .msg {
      max-width: 75%; padding: 8px 12px; border-radius: 12px; word-wrap: break-word;
      position: relative; font-size: 15px;
    }
    .received {
      align-self: flex-start; background: #202c33; border-bottom-left-radius: 4px;
    }
    .sent {
      align-self: flex-end; background: #005c4b; border-bottom-right-radius: 4px;
    }
    .time {
      font-size: 11px; opacity: 0.7; margin-top: 4px; text-align: right;
    }
    .input-area {
      display: flex; padding: 10px; background: #1f2c34;
    }
    #msgInput {
      flex: 1; background: #2a3942; color: white; border: none; border-radius: 25px;
      padding: 12px 18px; font-size: 15px; outline: none;
    }
    button {
      margin-left: 10px; background: #00a884; color: white; border: none;
      width: 48px; height: 48px; border-radius: 50%; cursor: pointer; font-size: 20px;
    }
    .typing { padding: 0 15px; color: #8696a0; font-size: 13px; font-style: italic; }
    .error { color: #ff6b6b; text-align: center; padding: 10px; }
  </style>
</head>
<body>

<div class="chat">
  <div class="header">Live WhatsApp Clone</div>

  <div class="name-input" id="nameSection">
    <p>Type your name to start chatting:</p>
    <input type="text" id="nameInput" placeholder="Your name..." autofocus>
    <button onclick="setName()">Join Chat</button>
    <div class="error" id="error" style="display:none;"></div>
  </div>

  <div id="chatSection" style="display:none; flex-direction:column; height:100%">
    <div class="messages" id="messages"></div>
    <div class="typing" id="typing"></div>
    <div class="input-area">
      <input type="text" id="msgInput" placeholder="Type a message" autocomplete="off">
      <button onclick="send()">➤</button>
    </div>
  </div>
</div>

<script>
  // PASTE YOUR FIREBASE CONFIG HERE (from Project Settings)
  const firebaseConfig = {
    // Example - replace with yours!
    apiKey: "YOUR_API_KEY",
    authDomain: "YOUR_PROJECT.firebaseapp.com",
    databaseURL: "https://YOUR_PROJECT-default-rtdb.firebaseio.com/",
    projectId: "YOUR_PROJECT_ID",
    storageBucket: "YOUR_PROJECT.appspot.com",
    messagingSenderId: "123456789",
    appId: "YOUR_APP_ID"
  };

  // Initialize Firebase
  firebase.initializeApp(firebaseConfig);
  const db = firebase.database();
  const messagesRef = db.ref("messages");

  let myName = "";

  function setName() {
    myName = document.getElementById("nameInput").value.trim() || "Anonymous";
    if (myName) {
      document.getElementById("nameSection").style.display = "none";
      document.getElementById("chatSection").style.display = "flex";
      document.getElementById("msgInput").focus();
      loadMessages(); // Load any existing messages
    }
  }

  // Send message
  function send() {
    const input = document.getElementById("msgInput");
    const text = input.value.trim();
    if (!text) return;

    messagesRef.push({
      name: myName,
      text: text,
      time: Date.now()
    });

    input.value = "";
  }

  // Load initial messages
  function loadMessages() {
    messagesRef.once("value", snapshot => {
      snapshot.forEach(child => {
        const msg = child.val();
        displayMessage(msg.name, msg.text, msg.time);
      });
    });
  }

  // Listen for new messages in real-time
  messagesRef.on("child_added", snapshot => {
    const msg = snapshot.val();
    displayMessage(msg.name, msg.text, msg.time);
  });

  // Display message
  function displayMessage(name, text, timestamp) {
    const div = document.createElement("div");
    const isMe = name === myName;
    div.className = `msg ${isMe ? "sent" : "received"}`;
    
    const time = new Date(timestamp).toLocaleTimeString([], {hour: '2-digit', minute: '2-digit'});
    
    div.innerHTML = `
      ${!isMe ? `<strong>${name}</strong><br>` : ''}
      ${text}
      <div class="time">${time}</div>
    `;
    document.getElementById("messages").appendChild(div);
    div.scrollIntoView({ behavior: "smooth" });
  }

  // Enter key to send
  document.getElementById("msgInput").addEventListener("keypress", e => {
    if (e.key === "Enter") send();
  });

  // Error handling (e.g., if config is wrong)
  messagesRef.on("error", err => {
    document.getElementById("error").textContent = "Oops! Check your Firebase config. Error: " + err.message;
    document.getElementById("error").style.display = "block";
  });
</script>

</body>
</html>
