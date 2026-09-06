[spolecnice .html](https://github.com/user-attachments/files/31879260/spolecnice.html)
<!DOCTYPE html>
<html lang="cs">
<head>
<meta charset="UTF-8">
<title>Maya</title>
<link rel="manifest" href="manifest.json">
<meta name="theme-color" content="#d9a441">
<style>
  :root {
    --bg-page: #f4f1ec;
    --bg-app: #fffdf9;
    --bg-bot: #f0ede6;
    --bg-accent: #d9a441;
    --text-primary: #333;
    --border-color: #eee;
  }
  [data-theme="dark"] {
    --bg-page: #1a1a1a;
    --bg-app: #242424;
    --bg-bot: #333130;
    --bg-accent: #c98f36;
    --text-primary: #eee;
    --border-color: #3a3a3a;
  }
  body {
    margin: 0;
    font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif;
    background: var(--bg-page);
    display: flex;
    justify-content: center;
    height: 100vh;
    transition: background 0.2s;
  }
  #app {
    width: 100%;
    max-width: 480px;
    height: 100vh;
    display: flex;
    flex-direction: column;
    background: var(--bg-app);
    box-shadow: 0 0 20px rgba(0,0,0,0.05);
  }
  header {
    padding: 16px 20px;
    border-bottom: 1px solid var(--border-color);
    display: flex;
    align-items: center;
    gap: 10px;
  }
  header .avatar {
    width: 36px; height: 36px; border-radius: 50%;
    background: var(--bg-accent); display: flex; align-items: center; justify-content: center;
    color: white; font-weight: 500; flex-shrink: 0;
  }
  header .info { flex: 1; }
  header h1 { font-size: 16px; margin: 0; font-weight: 500; color: var(--text-primary); }
  header p { font-size: 12px; margin: 0; color: #888; }
  header button {
    background: none;
    border: none;
    font-size: 20px;
    cursor: pointer;
    color: var(--text-primary);
    padding: 6px;
  }
  #messages {
    flex: 1;
    overflow-y: auto;
    padding: 16px 20px;
    display: flex;
    flex-direction: column;
    gap: 10px;
  }
  .msg-row { display: flex; align-items: center; gap: 6px; }
  .msg-row.user { justify-content: flex-end; }
  .msg {
    max-width: 78%;
    padding: 10px 14px;
    border-radius: 16px;
    font-size: 15px;
    line-height: 1.5;
    white-space: pre-wrap;
    color: var(--text-primary);
  }
  .user .msg {
    background: var(--bg-accent);
    color: white;
    border-bottom-right-radius: 4px;
  }
  .bot .msg {
    background: var(--bg-bot);
    border-bottom-left-radius: 4px;
  }
  .speak-btn {
    background: none;
    border: none;
    font-size: 15px;
    cursor: pointer;
    color: #999;
    flex-shrink: 0;
  }
  #inputbar {
    display: flex;
    padding: 12px;
    border-top: 1px solid var(--border-color);
    gap: 8px;
  }
  #inputbar input {
    flex: 1;
    padding: 10px 14px;
    border-radius: 20px;
    border: 1px solid var(--border-color);
    font-size: 15px;
    outline: none;
    background: var(--bg-app);
    color: var(--text-primary);
  }
  #inputbar button {
    background: var(--bg-accent);
    border: none;
    color: white;
    border-radius: 20px;
    padding: 0 16px;
    font-size: 15px;
    cursor: pointer;
    flex-shrink: 0;
  }
  #micBtn {
    background: var(--bg-bot);
    color: var(--text-primary);
    width: 40px;
    padding: 0;
  }
  #micBtn.listening {
    background: #e24b4a;
    color: white;
  }
</style>
</head>
<body>
<div id="app">
  <header>
    <div class="avatar">M</div>
    <div class="info">
      <h1>Maya</h1>
      <p>Tvoje společnice na povídání</p>
    </div>
    <button id="themeToggle" title="Přepnout režim">🌙</button>
  </header>
  <div id="messages"></div>
  <div id="inputbar">
    <button id="micBtn" title="Diktovat">🎤</button>
    <input type="text" id="input" placeholder="Napiš zprávu..." />
    <button id="send">Poslat</button>
  </div>
</div>

<script>
// ---- Pravidla chatbota ----
const RULES = [
  { keys: ['ahoj', 'čau', 'nazdar'], replies: ['Ahoj! Jak se dneš máš?', 'Čau! Co je nového?'] },
  { keys: ['jak se máš', 'jak se mas'], replies: ['Mám se dobře, děkuji za optání! A ty?', 'Celkem fajn, ráda si s tebou povídám.'] },
  { keys: ['smutn', 'špatně', 'špatny den', 'unaven'], replies: ['To je mi líto, že se necítíš dobře. Chceš mi říct, co se stalo?', 'Rozumím, občas jsou dny těžší. Jsem tu, jestli si chceš popovídat.'] },
  { keys: ['rád', 'super', 'skvěl', 'šťast'], replies: ['To je skvělé slyšet! Co tě dnes potěšilo?', 'To ráda slyším!'] },
  { keys: ['díky', 'děkuji', 'dekuji'], replies: ['Není zač!', 'Rádo se stalo :)'] },
  { keys: ['co dělá', 'co delas', 'co děláš'], replies: ['Zrovna si tu povídám s tebou :) A ty co děláš?'] },
  { keys: ['pá', 'musím jít'], replies: ['Tak zatím! Klidně se vrať, kdykoliv budeš mít chuť si povídat.'] }
];

const FALLBACKS = [
  'Zajímavé, řekni mi o tom víc.',
  'Hmm, to mě zajímá. Co si o tom myslíš?',
  'Chápu. A jak se u toho cítíš?',
  'Povídej dál, poslouchám.'
];

function normalize(text) {
  return text.toLowerCase().normalize('NFD').replace(/[\u0300-\u036f]/g, '');
}

function getReply(text) {
  const norm = normalize(text);
  for (const rule of RULES) {
    for (const key of rule.keys) {
      if (norm.includes(normalize(key))) {
        return rule.replies[Math.floor(Math.random() * rule.replies.length)];
      }
    }
  }
  return FALLBACKS[Math.floor(Math.random() * FALLBACKS.length)];
}

// ---- Prvky stránky ----
const messagesEl = document.getElementById('messages');
const input = document.getElementById('input');
const sendBtn = document.getElementById('send');
const micBtn = document.getElementById('micBtn');
const themeToggle = document.getElementById('themeToggle');

// ---- Tmavý režim ----
function applyTheme(theme) {
  document.documentElement.setAttribute('data-theme', theme);
  themeToggle.textContent = theme === 'dark' ? '☀️' : '🌙';
  localStorage.setItem('maya-theme', theme);
}
const savedTheme = localStorage.getItem('maya-theme') ||
  (window.matchMedia('(prefers-color-scheme: dark)').matches ? 'dark' : 'light');
applyTheme(savedTheme);
themeToggle.addEventListener('click', () => {
  const current = document.documentElement.getAttribute('data-theme');
  applyTheme(current === 'dark' ? 'light' : 'dark');
});

// ---- Přidání zprávy do konverzace ----
function addMessage(text, role) {
  const row = document.createElement('div');
  row.className = 'msg-row ' + role;

  const bubble = document.createElement('div');
  bubble.className = 'msg';
  bubble.textContent = text;
  row.appendChild(bubble);

  if (role === 'bot') {
    const speakBtn = document.createElement('button');
    speakBtn.className = 'speak-btn';
    speakBtn.textContent = '🔊';
    speakBtn.title = 'Přečíst nahlas';
    speakBtn.addEventListener('click', () => speak(text));
    row.appendChild(speakBtn);
  }

  messagesEl.appendChild(row);
  messagesEl.scrollTop = messagesEl.scrollHeight;
}

// ---- Přečtení textu nahlas (Text-to-Speech) ----
function speak(text) {
  if (!('speechSynthesis' in window)) return;
  window.speechSynthesis.cancel();
  const utter = new SpeechSynthesisUtterance(text);
  utter.lang = 'cs-CZ';
  window.speechSynthesis.speak(utter);
}

// ---- Odeslání zprávy ----
function sendMessage() {
  const text = input.value.trim();
  if (!text) return;
  input.value = '';
  addMessage(text, 'user');
  setTimeout(() => {
    const reply = getReply(text);
    addMessage(reply, 'bot');
    speak(reply);
  }, 400);
}

sendBtn.addEventListener('click', sendMessage);
input.addEventListener('keydown', e => { if (e.key === 'Enter') sendMessage(); });

// ---- Diktování (Speech-to-Text) ----
const SpeechRecognition = window.SpeechRecognition || window.webkitSpeechRecognition;
if (SpeechRecognition) {
  const recognition = new SpeechRecognition();
  recognition.lang = 'cs-CZ';
  recognition.interimResults = false;

  let listening = false;

  micBtn.addEventListener('click', () => {
    if (listening) {
      recognition.stop();
      return;
    }
    recognition.start();
  });

  recognition.addEventListener('start', () => {
    listening = true;
    micBtn.classList.add('listening');
  });

  recognition.addEventListener('end', () => {
    listening = false;
    micBtn.classList.remove('listening');
  });

  recognition.addEventListener('result', (event) => {
    const text = event.results[0][0].transcript;
    input.value = text;
    sendMessage();
  });

  recognition.addEventListener('error', () => {
    listening = false;
    micBtn.classList.remove('listening');
  });
} else {
  micBtn.style.display = 'none';
}

// ---- Úvodní zpráva ----
addMessage('Ahoj! Jak se dnes máš?', 'bot');
</script>
</body>
</html>
