# 🧠 Mac Mini Assistant Setup with Clawdbot + FastAPI

This guide turns your Mac Mini into a local stock assistant with:
- 🦞 Clawdbot skills + local agent
- ⚙️ FastAPI backend (`polygonio-mcp`)
- 🔐 Secure API access (optional)
- ☁️ Remote control with Tailscale (optional)

---

## 🧰 Requirements

| Tool        | Version / Info                   |
|-------------|----------------------------------|
| Mac Mini    | M2 or M4                        |
| Python      | >= 3.10 (homebrew recommended) |
| Node.js     | LTS (via `nvm` or `brew`)       |
| Clawdbot CLI| `npm install -g clawdbot`       |

---

## 📁 Project Layout

```
~/ai-assistant/
├── polygonio-mcp/         # FastAPI repo backend
├── .clawdbot/             # Skills directory
├── venv/                  # Python environment
└── README.md
```

---

## ✅ Setup Steps

### 1. Clone & Initialize

```bash
mkdir ~/ai-assistant && cd ~/ai-assistant
git clone https://github.com/MCVelasquez45/polygonio-mcp.git
clawdbot init  # Creates .clawdbot
```

### 2. Python Env & Install

```bash
python3 -m venv venv
source venv/bin/activate
pip install -e ./polygonio-mcp
```

### 3. Test FastAPI Backend

```bash
cd polygonio-mcp/agent
uvicorn api:app --reload --port 5001
```
Go to [http://127.0.0.1:5001/docs](http://127.0.0.1:5001/docs) to confirm.

### 4. Register Clawdbot Skill

Inside `.clawdbot/skills/get-stock-summary/index.js`
```js
module.exports = async function getStockSummary({ ticker }) {
  const response = await axios.post('http://localhost:5001/summary', { ticker });
  return `📈 ${ticker} Summary: ${response.data.educational}`;
};
```

Then:
```bash
clawdbot skills check
```

---

## 🧪 Test Locally

```bash
clawdbot agent --local --agent main --message "Get stock summary for AAPL"
```
You should see a real-time response from your FastAPI app.

---

## 🔒 Optional: Secure API

In `polygonio-mcp/agent/api.py`:
```python
from fastapi import Depends, Header, HTTPException

async def verify_key(x_api_key: str = Header(...)):
    if x_api_key != "YOUR_SECRET_KEY":
        raise HTTPException(status_code=403)

@app.post("/summary", dependencies=[Depends(verify_key)])
```

---

## ☁️ Optional: Remote Access

Install [Tailscale](https://tailscale.com) to:
- Access the agent from phone or laptop
- Run `curl http://100.x.x.x:5001/summary`

---

## 🚀 Production Tips

- Use `pm2` or `launchctl` to auto-start backend on reboot
- Use `ngrok` or `cloudflared` to expose the endpoint
- Extend skills: news, alerts, short interest, AI chat

---


