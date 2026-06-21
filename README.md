<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0">
<title>Claude Voz</title>
<style>
  @import url('https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500&family=Space+Grotesk:wght@400;600&display=swap');

  *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

  :root {
    --bg: #0a0a0f;
    --surface: #13131a;
    --border: #1e1e2e;
    --accent: #7c6af7;
    --accent-glow: rgba(124, 106, 247, 0.25);
    --text: #e8e8f0;
    --muted: #6b6b80;
    --danger: #f76a6a;
    --success: #6af7a8;
  }

  html, body {
    height: 100%;
    background: var(--bg);
    color: var(--text);
    font-family: 'Inter', sans-serif;
    overflow: hidden;
  }

  body {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: space-between;
    padding: 24px 20px 36px;
    gap: 16px;
  }

  /* Header */
  header {
    width: 100%;
    display: flex;
    align-items: center;
    justify-content: space-between;
  }

  .logo {
    font-family: 'Space Grotesk', sans-serif;
    font-size: 15px;
    font-weight: 600;
    letter-spacing: 0.04em;
    color: var(--accent);
  }

  .status-dot {
    width: 8px;
    height: 8px;
    border-radius: 50%;
    background: var(--muted);
    transition: background 0.3s;
  }
  .status-dot.ready { background: var(--success); box-shadow: 0 0 8px var(--success); }
  .status-dot.listening { background: var(--accent); box-shadow: 0 0 8px var(--accent); animation: pulse 1s infinite; }
  .status-dot.thinking { background: #f7c76a; box-shadow: 0 0 8px #f7c76a; animation: pulse 0.6s infinite; }
  .status-dot.speaking { background: var(--success); box-shadow: 0 0 8px var(--success); animation: pulse 0.8s infinite; }

  @keyframes pulse {
    0%, 100% { opacity: 1; }
    50% { opacity: 0.4; }
  }

  /* Conversation area */
  .conversation {
    flex: 1;
    width: 100%;
    max-width: 480px;
    overflow-y: auto;
    display: flex;
    flex-direction: column;
    gap: 12px;
    padding: 4px 0;
    scrollbar-width: none;
  }
  .conversation::-webkit-scrollbar { display: none; }

  .bubble {
    padding: 12px 16px;
    border-radius: 16px;
    font-size: 14px;
    line-height: 1.6;
    max-width: 85%;
    animation: fadeUp 0.3s ease;
  }

  @keyframes fadeUp {
    from { opacity: 0; transform: translateY(8px); }
    to { opacity: 1; transform: translateY(0); }
  }

  .bubble.user {
    background: var(--accent);
    color: #fff;
    align-self: flex-end;
    border-bottom-right-radius: 4px;
  }

  .bubble.claude {
    background: var(--surface);
    border: 1px solid var(--border);
    color: var(--text);
    align-self: flex-start;
    border-bottom-left-radius: 4px;
  }

  .bubble.typing {
    background: var(--surface);
    border: 1px solid var(--border);
    align-self: flex-start;
    border-bottom-left-radius: 4px;
  }

  .typing-dots {
    display: flex;
    gap: 5px;
    align-items: center;
    height: 18px;
  }

  .typing-dots span {
    width: 6px;
    height: 6px;
    background: var(--muted);
    border-radius: 50%;
    animation: bounce 1.2s infinite;
  }
  .typing-dots span:nth-child(2) { animation-delay: 0.2s; }
  .typing-dots span:nth-child(3) { animation-delay: 0.4s; }

  @keyframes bounce {
    0%, 80%, 100% { transform: translateY(0); }
    40% { transform: translateY(-6px); }
  }

  /* Center orb area */
  .orb-area {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 20px;
  }

  .orb-container {
    position: relative;
    width: 120px;
    height: 120px;
    display: flex;
    align-items: center;
    justify-content: center;
  }

  .orb-ring {
    position: absolute;
    border-radius: 50%;
    border: 1px solid var(--accent-glow);
    animation: expandRing 2s infinite;
    opacity: 0;
  }

  .orb-ring:nth-child(1) { width: 100%; height: 100%; animation-delay: 0s; }
  .orb-ring:nth-child(2) { width: 130%; height: 130%; animation-delay: 0.5s; }
  .orb-ring:nth-child(3) { width: 160%; height: 160%; animation-delay: 1s; }

  .orb-ring.active {
    animation: expandRing 1.5s infinite;
    border-color: rgba(124, 106, 247, 0.4);
  }

  @keyframes expandRing {
    0% { opacity: 0.6; transform: scale(0.8); }
    100% { opacity: 0; transform: scale(1.2); }
  }

  .mic-btn {
    position: relative;
    z-index: 2;
    width: 80px;
    height: 80px;
    border-radius: 50%;
    border: none;
    background: var(--surface);
    border: 1.5px solid var(--border);
    color: var(--muted);
    cursor: pointer;
    display: flex;
    align-items: center;
    justify-content: center;
    transition: all 0.2s ease;
    outline: none;
    -webkit-tap-highlight-color: transparent;
  }

  .mic-btn:active { transform: scale(0.94); }

  .mic-btn.listening {
    background: var(--accent);
    border-color: var(--accent);
    color: #fff;
    box-shadow: 0 0 30px var(--accent-glow);
  }

  .mic-btn.thinking {
    background: var(--surface);
    border-color: #f7c76a;
    color: #f7c76a;
  }

  .mic-btn.speaking {
    background: var(--surface);
    border-color: var(--success);
    color: var(--success);
    box-shadow: 0 0 20px rgba(106, 247, 168, 0.2);
  }

  .mic-btn svg {
    width: 28px;
    height: 28px;
    transition: all 0.2s;
  }

  /* Status label */
  .status-label {
    font-size: 12px;
    color: var(--muted);
    letter-spacing: 0.08em;
    text-transform: uppercase;
    font-weight: 500;
    min-height: 16px;
    transition: all 0.3s;
  }

  /* Controls */
  .controls {
    display: flex;
    gap: 12px;
    align-items: center;
  }

  .ctrl-btn {
    background: var(--surface);
    border: 1px solid var(--border);
    color: var(--muted);
    border-radius: 10px;
    padding: 8px 14px;
    font-size: 12px;
    font-family: 'Inter', sans-serif;
    cursor: pointer;
    transition: all 0.2s;
    -webkit-tap-highlight-color: transparent;
  }

  .ctrl-btn:hover, .ctrl-btn:active { border-color: var(--accent); color: var(--accent); }

  /* API key modal */
  .modal-overlay {
    position: fixed;
    inset: 0;
    background: rgba(0,0,0,0.85);
    display: flex;
    align-items: center;
    justify-content: center;
    z-index: 100;
    padding: 24px;
  }

  .modal {
    background: var(--surface);
    border: 1px solid var(--border);
    border-radius: 20px;
    padding: 28px 24px;
    width: 100%;
    max-width: 380px;
    display: flex;
    flex-direction: column;
    gap: 16px;
  }

  .modal h2 {
    font-family: 'Space Grotesk', sans-serif;
    font-size: 18px;
    font-weight: 600;
    color: var(--text);
  }

  .modal p {
    font-size: 13px;
    color: var(--muted);
    line-height: 1.6;
  }

  .modal input {
    background: var(--bg);
    border: 1px solid var(--border);
    border-radius: 10px;
    padding: 12px 14px;
    color: var(--text);
    font-size: 14px;
    font-family: 'Inter', sans-serif;
    width: 100%;
    outline: none;
    transition: border-color 0.2s;
  }

  .modal input:focus { border-color: var(--accent); }

  .modal-btn {
    background: var(--accent);
    color: #fff;
    border: none;
    border-radius: 10px;
    padding: 13px;
    font-size: 14px;
    font-weight: 500;
    font-family: 'Inter', sans-serif;
    cursor: pointer;
    transition: opacity 0.2s;
  }

  .modal-btn:active { opacity: 0.85; }

  .error-msg {
    font-size: 12px;
    color: var(--danger);
    display: none;
  }

  /* Voice select */
  .voice-row {
    display: flex;
    gap: 8px;
    align-items: center;
  }
  .voice-row label { font-size: 12px; color: var(--muted); white-space: nowrap; }
  .voice-row select {
    flex: 1;
    background: var(--bg);
    border: 1px solid var(--border);
    border-radius: 8px;
    padding: 6px 10px;
    color: var(--text);
    font-size: 12px;
    font-family: 'Inter', sans-serif;
    outline: none;
  }

  .empty-state {
    flex: 1;
    display: flex;
    align-items: center;
    justify-content: center;
  }

  .empty-state p {
    font-size: 13px;
    color: var(--muted);
    text-align: center;
    line-height: 1.7;
  }
</style>
</head>
<body>

<header>
  <span class="logo">CLAUDE VOZ</span>
  <div class="status-dot" id="statusDot"></div>
</header>

<div class="conversation" id="conversation">
  <div class="empty-state" id="emptyState">
    <p>Toque no microfone e fale.<br>Eu ouço, processo e respondo em voz.</p>
  </div>
</div>

<div class="orb-area">
  <div class="orb-container">
    <div class="orb-ring" id="ring1"></div>
    <div class="orb-ring" id="ring2"></div>
    <div class="orb-ring" id="ring3"></div>
    <button class="mic-btn" id="micBtn" aria-label="Falar">
      <!-- Mic icon -->
      <svg id="iconMic" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
        <path d="M12 1a3 3 0 0 0-3 3v8a3 3 0 0 0 6 0V4a3 3 0 0 0-3-3z"/>
        <path d="M19 10v2a7 7 0 0 1-14 0v-2"/>
        <line x1="12" y1="19" x2="12" y2="23"/>
        <line x1="8" y1="23" x2="16" y2="23"/>
      </svg>
      <!-- Stop icon -->
      <svg id="iconStop" viewBox="0 0 24 24" fill="currentColor" style="display:none;">
        <rect x="6" y="6" width="12" height="12" rx="2"/>
      </svg>
      <!-- Spinner -->
      <svg id="iconSpin" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" style="display:none; animation: spin 1s linear infinite;">
        <path d="M12 2v4M12 18v4M4.93 4.93l2.83 2.83M16.24 16.24l2.83 2.83M2 12h4M18 12h4M4.93 19.07l2.83-2.83M16.24 7.76l2.83-2.83"/>
      </svg>
      <!-- Speaker icon -->
      <svg id="iconSpeak" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" style="display:none;">
        <polygon points="11 5 6 9 2 9 2 15 6 15 11 19 11 5"/>
        <path d="M19.07 4.93a10 10 0 0 1 0 14.14"/>
        <path d="M15.54 8.46a5 5 0 0 1 0 7.07"/>
      </svg>
    </button>
  </div>
  <div class="status-label" id="statusLabel">Pronto</div>
</div>

<div class="controls">
  <div class="voice-row">
    <label>Voz:</label>
    <select id="voiceSelect"></select>
  </div>
  <button class="ctrl-btn" id="clearBtn">Limpar</button>
  <button class="ctrl-btn" id="keyBtn">Chave API</button>
</div>

<!-- API Key Modal -->
<div class="modal-overlay" id="modal">
  <div class="modal">
    <h2>Chave da API Anthropic</h2>
    <p>Insira sua chave para usar o Claude. Ela fica salva apenas no seu navegador.</p>
    <input type="password" id="apiKeyInput" placeholder="sk-ant-..." autocomplete="off" />
    <p class="error-msg" id="errorMsg">Chave inválida ou erro de conexão.</p>
    <button class="modal-btn" id="saveKeyBtn">Salvar e começar</button>
  </div>
</div>

<style>
@keyframes spin { to { transform: rotate(360deg); } }
</style>

<script>
// ── State ──────────────────────────────────────────────────────────────────
let apiKey = localStorage.getItem('claude_voice_key') || '';
let state = 'idle'; // idle | listening | thinking | speaking
let recognition = null;
let synth = window.speechSynthesis;
let voices = [];
let conversationHistory = [];
let typingBubble = null;

// ── DOM ────────────────────────────────────────────────────────────────────
const micBtn = document.getElementById('micBtn');
const statusLabel = document.getElementById('statusLabel');
const statusDot = document.getElementById('statusDot');
const conversation = document.getElementById('conversation');
const emptyState = document.getElementById('emptyState');
const modal = document.getElementById('modal');
const apiKeyInput = document.getElementById('apiKeyInput');
const saveKeyBtn = document.getElementById('saveKeyBtn');
const errorMsg = document.getElementById('errorMsg');
const clearBtn = document.getElementById('clearBtn');
const keyBtn = document.getElementById('keyBtn');
const voiceSelect = document.getElementById('voiceSelect');
const rings = [document.getElementById('ring1'), document.getElementById('ring2'), document.getElementById('ring3')];
const iconMic = document.getElementById('iconMic');
const iconStop = document.getElementById('iconStop');
const iconSpin = document.getElementById('iconSpin');
const iconSpeak = document.getElementById('iconSpeak');

// ── Init ───────────────────────────────────────────────────────────────────
if (!apiKey) showModal();
else { setStatus('ready'); }

function showModal() {
  modal.style.display = 'flex';
  apiKeyInput.value = apiKey;
  errorMsg.style.display = 'none';
}

function hideModal() { modal.style.display = 'none'; }

saveKeyBtn.addEventListener('click', () => {
  const k = apiKeyInput.value.trim();
  if (!k.startsWith('sk-')) { errorMsg.style.display = 'block'; return; }
  apiKey = k;
  localStorage.setItem('claude_voice_key', k);
  hideModal();
  setStatus('ready');
});

keyBtn.addEventListener('click', showModal);
clearBtn.addEventListener('click', () => {
  conversationHistory = [];
  conversation.innerHTML = '';
  conversation.appendChild(emptyState);
  emptyState.style.display = 'flex';
});

// ── Voice list ─────────────────────────────────────────────────────────────
function loadVoices() {
  voices = synth.getVoices();
  voiceSelect.innerHTML = '';
  const ptVoices = voices.filter(v => v.lang.startsWith('pt'));
  const allVoices = ptVoices.length ? ptVoices : voices;
  allVoices.forEach((v, i) => {
    const opt = document.createElement('option');
    opt.value = i;
    opt.textContent = v.name.replace('Google ', '').substring(0, 22);
    opt.dataset.globalIndex = voices.indexOf(v);
    voiceSelect.appendChild(opt);
  });
}
loadVoices();
if (speechSynthesis.onvoiceschanged !== undefined) speechSynthesis.onvoiceschanged = loadVoices;

function getSelectedVoice() {
  const opt = voiceSelect.options[voiceSelect.selectedIndex];
  if (!opt) return null;
  return voices[parseInt(opt.dataset.globalIndex)];
}

// ── Status management ──────────────────────────────────────────────────────
function setStatus(s) {
  state = s;
  statusDot.className = 'status-dot ' + (s === 'idle' ? '' : s);

  const labels = { idle: 'Pronto', ready: 'Pronto', listening: 'Ouvindo...', thinking: 'Processando...', speaking: 'Respondendo...' };
  statusLabel.textContent = labels[s] || '';

  micBtn.className = 'mic-btn ' + (s === 'idle' || s === 'ready' ? '' : s);

  // Icons
  iconMic.style.display = (s === 'idle' || s === 'ready') ? 'block' : 'none';
  iconStop.style.display = s === 'listening' ? 'block' : 'none';
  iconSpin.style.display = s === 'thinking' ? 'block' : 'none';
  iconSpeak.style.display = s === 'speaking' ? 'block' : 'none';

  // Rings
  rings.forEach(r => r.className = 'orb-ring ' + (s === 'listening' || s === 'speaking' ? 'active' : ''));
}

// ── Conversation bubbles ───────────────────────────────────────────────────
function addBubble(text, role) {
  if (emptyState.parentNode) emptyState.style.display = 'none';
  const b = document.createElement('div');
  b.className = 'bubble ' + role;
  b.textContent = text;
  conversation.appendChild(b);
  conversation.scrollTop = conversation.scrollHeight;
  return b;
}

function showTyping() {
  if (emptyState.parentNode) emptyState.style.display = 'none';
  typingBubble = document.createElement('div');
  typingBubble.className = 'bubble typing';
  typingBubble.innerHTML = '<div class="typing-dots"><span></span><span></span><span></span></div>';
  conversation.appendChild(typingBubble);
  conversation.scrollTop = conversation.scrollHeight;
}

function removeTyping() {
  if (typingBubble) { typingBubble.remove(); typingBubble = null; }
}

// ── Speech Recognition ─────────────────────────────────────────────────────
function startListening() {
  if (!('webkitSpeechRecognition' in window) && !('SpeechRecognition' in window)) {
    alert('Seu navegador não suporta reconhecimento de voz. Use Chrome.');
    return;
  }
  const SR = window.SpeechRecognition || window.webkitSpeechRecognition;
  recognition = new SR();
  recognition.lang = 'pt-BR';
  recognition.interimResults = false;
  recognition.maxAlternatives = 1;

  recognition.onstart = () => setStatus('listening');

  recognition.onresult = (e) => {
    const transcript = e.results[0][0].transcript;
    addBubble(transcript, 'user');
    askClaude(transcript);
  };

  recognition.onerror = (e) => {
    console.error('Speech error:', e.error);
    setStatus('ready');
    if (e.error === 'no-speech') statusLabel.textContent = 'Nada detectado';
  };

  recognition.onend = () => {
    if (state === 'listening') setStatus('ready');
  };

  recognition.start();
}

function stopListening() {
  if (recognition) { recognition.stop(); recognition = null; }
  setStatus('ready');
}

// ── Claude API ─────────────────────────────────────────────────────────────
async function askClaude(userText) {
  if (synth.speaking) synth.cancel();
  setStatus('thinking');
  showTyping();

  conversationHistory.push({ role: 'user', content: userText });

  try {
    const res = await fetch('https://api.anthropic.com/v1/messages', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'x-api-key': apiKey,
        'anthropic-version': '2023-06-01',
        'anthropic-dangerous-direct-browser-access': 'true'
      },
      body: JSON.stringify({
        model: 'claude-sonnet-4-6',
        max_tokens: 1024,
        system: 'Você é o Claude, um assistente de IA conversando por voz. Responda de forma clara e concisa, como se estivesse falando, não escrevendo. Evite listas com marcadores, asteriscos ou formatação markdown — use linguagem natural e fluida. Seja direto e objetivo. Responda sempre em português brasileiro.',
        messages: conversationHistory
      })
    });

    if (!res.ok) {
      const err = await res.json();
      throw new Error(err.error?.message || 'Erro na API');
    }

    const data = await res.json();
    const reply = data.content[0].text;

    conversationHistory.push({ role: 'assistant', content: reply });
    removeTyping();
    addBubble(reply, 'claude');
    speakText(reply);

  } catch (err) {
    removeTyping();
    addBubble('Erro: ' + err.message, 'claude');
    setStatus('ready');
    console.error(err);
  }
}

// ── TTS ────────────────────────────────────────────────────────────────────
function speakText(text) {
  if (!synth) { setStatus('ready'); return; }
  setStatus('speaking');

  const utter = new SpeechSynthesisUtterance(text);
  const voice = getSelectedVoice();
  if (voice) utter.voice = voice;
  utter.lang = 'pt-BR';
  utter.rate = 1.05;
  utter.pitch = 1;

  utter.onend = () => setStatus('ready');
  utter.onerror = () => setStatus('ready');

  synth.speak(utter);
}

// ── Mic button ─────────────────────────────────────────────────────────────
micBtn.addEventListener('click', () => {
  if (!apiKey) { showModal(); return; }

  if (state === 'speaking') {
    synth.cancel();
    setStatus('ready');
    return;
  }

  if (state === 'listening') {
    stopListening();
    return;
  }

  if (state === 'thinking') return;

  startListening();
});

// Prevent double-tap zoom on mobile
micBtn.addEventListener('touchend', (e) => e.preventDefault());
</script>
</body>
</html>
