<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1.0" />
<title>Lucky Anish — ask my repos</title>
<meta name="description" content="A self-contained, offline agent that answers questions about Lucky Anish's projects." />
<style>
@import url('https://fonts.googleapis.com/css2?family=IBM+Plex+Mono:wght@400;500;600&family=IBM+Plex+Sans:wght@400;500&display=swap');

* { box-sizing: border-box; }
body { margin: 0; padding: 40px 16px; background: #050706; min-height: 100vh; display: flex; align-items: center; justify-content: center; }

.pa-root {
  --bg: #0A0C0B;
  --panel: #121614;
  --panel-2: #161B18;
  --amber: #FFB000;
  --cyan: #4FD1C5;
  --text: #E7E5DE;
  --muted: #7C8580;
  --line: #232B27;

  background: var(--bg);
  color: var(--text);
  font-family: 'IBM Plex Sans', sans-serif;
  border: 1px solid var(--line);
  border-radius: 10px;
  overflow: hidden;
  display: grid;
  grid-template-columns: 220px 1fr;
  min-height: 560px;
  max-width: 920px;
  width: 100%;
  margin: 0 auto;
}

@media (max-width: 640px) {
  .pa-root { grid-template-columns: 1fr; }
  .pa-sidebar { display: none; }
}

.pa-sidebar {
  background: var(--panel);
  border-right: 1px solid var(--line);
  padding: 20px 16px;
  font-family: 'IBM Plex Mono', monospace;
  font-size: 12.5px;
}

.pa-sidebar .prompt { color: var(--muted); margin-bottom: 10px; }
.pa-sidebar .prompt span { color: var(--amber); }

.pa-repo-list { list-style: none; margin: 0; padding: 0; }
.pa-repo-list li {
  padding: 6px 0;
  color: var(--text);
  cursor: pointer;
  opacity: 0;
  animation: pa-fadein 0.35s ease forwards;
  border-radius: 4px;
  transition: color 0.15s;
}
.pa-repo-list li:hover { color: var(--amber); }
.pa-repo-list li .tag { color: var(--muted); font-size: 11px; display: block; }

@keyframes pa-fadein { to { opacity: 1; } }

.pa-main {
  display: flex;
  flex-direction: column;
  min-height: 560px;
}

.pa-header {
  padding: 16px 20px;
  border-bottom: 1px solid var(--line);
  font-family: 'IBM Plex Mono', monospace;
  font-size: 13px;
  color: var(--muted);
  display: flex;
  justify-content: space-between;
  align-items: center;
}
.pa-header b { color: var(--cyan); font-weight: 500; }
.pa-dot { width: 6px; height: 6px; border-radius: 50%; background: var(--cyan); display: inline-block; margin-right: 6px; box-shadow: 0 0 6px var(--cyan); }

.pa-messages {
  flex: 1;
  padding: 20px;
  overflow-y: auto;
  display: flex;
  flex-direction: column;
  gap: 16px;
}

.pa-msg { max-width: 90%; opacity: 0; animation: pa-fadein 0.3s ease forwards; }
.pa-msg.user { align-self: flex-end; }
.pa-msg.agent { align-self: flex-start; }

.pa-bubble {
  padding: 10px 14px;
  border-radius: 8px;
  font-size: 14.5px;
  line-height: 1.55;
  white-space: pre-wrap;
}
.pa-msg.user .pa-bubble { background: var(--panel-2); border: 1px solid var(--line); color: var(--text); }
.pa-msg.agent .pa-bubble { background: transparent; border-left: 2px solid var(--cyan); padding-left: 12px; color: var(--text); }
.pa-msg .pa-label { font-family: 'IBM Plex Mono', monospace; font-size: 10.5px; color: var(--muted); margin-bottom: 4px; letter-spacing: 0.04em; }

.pa-inputbar {
  border-top: 1px solid var(--line);
  padding: 14px 16px;
  display: flex;
  gap: 10px;
  background: var(--panel);
}
.pa-inputbar input {
  flex: 1;
  background: var(--bg);
  border: 1px solid var(--line);
  border-radius: 6px;
  padding: 10px 12px;
  color: var(--text);
  font-family: 'IBM Plex Mono', monospace;
  font-size: 13.5px;
  outline: none;
}
.pa-inputbar input:focus { border-color: var(--cyan); }
.pa-inputbar button {
  background: var(--amber);
  color: #0A0C0B;
  border: none;
  border-radius: 6px;
  padding: 0 18px;
  font-family: 'IBM Plex Mono', monospace;
  font-weight: 600;
  font-size: 13px;
  cursor: pointer;
  transition: opacity 0.15s;
}
.pa-inputbar button:hover { opacity: 0.85; }

.pa-suggestions { display: flex; gap: 8px; flex-wrap: wrap; padding: 0 16px 14px; }
.pa-suggestions button {
  background: var(--panel-2);
  border: 1px solid var(--line);
  color: var(--muted);
  font-family: 'IBM Plex Mono', monospace;
  font-size: 11.5px;
  padding: 6px 10px;
  border-radius: 20px;
  cursor: pointer;
}
.pa-suggestions button:hover { color: var(--amber); border-color: var(--amber); }
</style>
</head>
<body>

<div class="pa-root">
  <div class="pa-sidebar">
    <div class="prompt"><span>$</span> ls ~/repos</div>
    <ul class="pa-repo-list" id="pa-repo-list"></ul>
  </div>
  <div class="pa-main">
    <div class="pa-header">
      <span><span class="pa-dot"></span>ask-my-repos<b>.agent</b></span>
      <span id="pa-status">ready · offline</span>
    </div>
    <div class="pa-messages" id="pa-messages"></div>
    <div class="pa-suggestions" id="pa-suggestions"></div>
    <div class="pa-inputbar">
      <input id="pa-input" type="text" placeholder="ask about VICTOR, AgentOS, the MCP server..." />
      <button id="pa-send">run</button>
    </div>
  </div>
</div>

<script>
(function() {
  // Everything below runs entirely in the browser. No network calls, no API key, no backend.
  const REPOS = [
    { key: "victor", name: "victor/", tag: "bot detection · SHAP",
      keywords: ["victor", "bot", "detection", "shap", "isolation", "xgboost", "streamlit"],
      desc: "VICTOR is a bot-detection system built on an Isolation Forest + XGBoost ensemble, with SHAP explainability so every flagged account comes with a reason. It ships with a live Streamlit/Plotly dashboard for exploring predictions." },
    { key: "agentos", name: "agentos/", tag: "LangGraph",
      keywords: ["agentos", "agent", "langgraph", "orchestration", "multi-agent"],
      desc: "AgentOS is a multi-agent orchestration system built with LangGraph, coordinating multiple agents through a shared graph to execute multi-step tasks." },
    { key: "rag", name: "rag-chatbot/", tag: "retrieval",
      keywords: ["rag", "retrieval", "chatbot", "ragas", "critic", "research agent"],
      desc: "A RAG chatbot currently being upgraded with a Critic Agent (scored with RAGAs metrics) and a re-retrieval Research Agent to catch and fix weak answers before they reach the user." },
    { key: "bert", name: "bert-sentiment/", tag: "FastAPI",
      keywords: ["bert", "sentiment", "fastapi", "nlp", "classification"],
      desc: "A BERT-based sentiment analysis model served as a REST API via FastAPI." },
    { key: "cnn", name: "cnn-classifier/", tag: "computer vision",
      keywords: ["cnn", "image", "classifier", "vision", "pytorch", "tensorflow"],
      desc: "A CNN image classifier built with PyTorch/TensorFlow." },
    { key: "jarvis-cloud", name: "jarvis-cloud/", tag: "Groq · Llama 3.3 70B",
      keywords: ["jarvis", "cloud", "groq", "llama", "voice", "assistant"],
      desc: "A voice-driven desktop assistant powered by Groq-hosted Llama 3.3 70B." },
    { key: "jarvis-local", name: "jarvis-local/", tag: "Ollama · Qwen3",
      keywords: ["jarvis", "local", "offline", "ollama", "qwen", "rtx"],
      desc: "A fully offline desktop assistant using Ollama and Qwen3, hardware-optimized to run well on an RTX 2050." },
    { key: "jobbot", name: "job-bot/", tag: "Playwright",
      keywords: ["job", "bot", "playwright", "application", "greenhouse", "lever", "automation"],
      desc: "An autonomous job application bot that discovers and applies to roles end-to-end using Playwright and the Greenhouse/Lever APIs." },
    { key: "mcp", name: "mcp-server/", tag: "FastMCP · SQLite",
      keywords: ["mcp", "server", "fastmcp", "sqlite", "remotive", "model context protocol"],
      desc: "A Job Search MCP Server exposing 6 tools (search, application tracking, cover-letter drafting) over the Remotive API, with SQLite persistence. Anthropic MCP certified." },
    { key: "agentlayer", name: "agentlayer/", tag: "startup",
      keywords: ["agentlayer", "startup", "infrastructure", "agent-native"],
      desc: "Agentlayer is agent-native software infrastructure, built on the thesis that AI agents will become a major class of internet users, not just humans." },
    { key: "retix", name: "retix/", tag: "startup",
      keywords: ["retix", "startup", "inventory", "gst", "khata", "distribution", "wholesale", "android"],
      desc: "Retix is a GST-ready inventory, distributor, and Udhari/Khata credit-tracking platform for Indian wholesale distribution, with an Android app in progress." }
  ];

  const FALLBACK = "I don't have anything on that yet — I only know about the 11 repos in the sidebar. Try asking about VICTOR, AgentOS, the MCP server, or click a name on the left.";

  function findMatches(question) {
    const q = question.toLowerCase();
    const scored = REPOS.map(r => {
      let score = 0;
      if (q.includes(r.key)) score += 5;
      r.keywords.forEach(k => { if (q.includes(k)) score += 2; });
      return { r, score };
    }).filter(x => x.score > 0).sort((a, b) => b.score - a.score);
    return scored.map(x => x.r);
  }

  function answer(question) {
    const matches = findMatches(question);
    if (matches.length === 0) return FALLBACK;
    if (matches.length === 1) return matches[0].desc;
    return matches.slice(0, 2).map(r => r.name + " — " + r.desc).join("\n\n");
  }

  const listEl = document.getElementById('pa-repo-list');
  REPOS.forEach((r, i) => {
    const li = document.createElement('li');
    li.style.animationDelay = (i * 0.05) + 's';
    li.innerHTML = r.name + '<span class="tag">' + r.tag + '</span>';
    li.addEventListener('click', () => ask('What does ' + r.key + ' do?'));
    listEl.appendChild(li);
  });

  const suggestions = ["what's VICTOR?", "explain the MCP server", "what stack for AgentOS?", "tell me about Retix"];
  const sugEl = document.getElementById('pa-suggestions');
  suggestions.forEach(s => {
    const b = document.createElement('button');
    b.textContent = s;
    b.addEventListener('click', () => ask(s));
    sugEl.appendChild(b);
  });

  const messagesEl = document.getElementById('pa-messages');
  const inputEl = document.getElementById('pa-input');
  const sendBtn = document.getElementById('pa-send');

  function addMessage(role, text) {
    const wrap = document.createElement('div');
    wrap.className = 'pa-msg ' + (role === 'user' ? 'user' : 'agent');
    const label = document.createElement('div');
    label.className = 'pa-label';
    label.textContent = role === 'user' ? '> you' : '~ agent';
    const bubble = document.createElement('div');
    bubble.className = 'pa-bubble';
    bubble.textContent = text;
    wrap.appendChild(label);
    wrap.appendChild(bubble);
    messagesEl.appendChild(wrap);
    messagesEl.scrollTop = messagesEl.scrollHeight;
    return bubble;
  }

  function ask(question) {
    if (!question || !question.trim()) return;
    inputEl.value = '';
    addMessage('user', question);
    setTimeout(() => { addMessage('agent', answer(question)); }, 220);
  }

  sendBtn.addEventListener('click', () => ask(inputEl.value));
  inputEl.addEventListener('keydown', (e) => { if (e.key === 'Enter') ask(inputEl.value); });

  setTimeout(() => {
    addMessage('agent', "Indexed 11 repos, running fully offline — no API calls. Ask me anything, or click a name on the left.");
  }, 300);
})();
</script>
</body>
</html>
