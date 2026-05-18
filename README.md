# mc-server
<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Minecraft 伺服器狀態監控</title>
    <style>
        :root {
            --bg-color: #1e1e2e;
            --card-bg: #252538;
            --text-color: #cdd6f4;
            --text-muted: #a6adc8;
            --online-color: #a6e3a1;
            --offline-color: #f38ba8;
        }
        body {
            font-family: 'Segoe UI', system-ui, sans-serif;
            background-color: var(--bg-color);
            color: var(--text-color);
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            margin: 0;
        }
        .card {
            background-color: var(--card-bg);
            padding: 30px;
            border-radius: 16px;
            box-shadow: 0 10px 30px rgba(0,0,0,0.4);
            text-align: center;
            width: 380px;
            border: 1px solid #313244;
        }
        h2 { margin-top: 0; font-size: 1.6rem; letter-spacing: 1px; }
        .ip-badge {
            background: #11111b;
            padding: 6px 12px;
            border-radius: 20px;
            font-family: monospace;
            font-size: 0.95rem;
            color: var(--text-muted);
            display: inline-block;
            margin-bottom: 20px;
        }
        .status {
            font-size: 1.1rem;
            padding: 10px;
            border-radius: 8px;
            font-weight: bold;
            margin-bottom: 20px;
            transition: all 0.3s ease;
        }
        .online { background-color: rgba(166, 227, 161, 0.2); color: var(--online-color); border: 1px solid var(--online-color); }
        .offline { background-color: rgba(243, 139, 168, 0.2); color: var(--offline-color); border: 1px solid var(--offline-color); }
        .loading { background-color: rgba(250, 179, 135, 0.2); color: #fab387; border: 1px solid #fab387; }
        
        .info-grid {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 15px;
            margin-top: 20px;
            text-align: left;
        }
        .info-item {
            background: #181825;
            padding: 12px;
            border-radius: 8px;
        }
        .info-label { font-size: 0.85rem; color: var(--text-muted); margin-bottom: 5px; }
        .info-value { font-size: 1.1rem; font-weight: bold; }
        .motd {
            margin-top: 15px;
            font-size: 0.9rem;
            background: #181825;
            padding: 10px;
            border-radius: 8px;
            color: #f5e0dc;
            word-break: break-all;
        }
    </style>
</head>
<body>

<div class="card">
    <h2>伺服器狀態監控</h2>
    <div id="server-ip" class="ip-badge">載入中...</div>
    
    <div id="status-badge" class="status loading">正在連線至 API...</div>
    
    <div id="info-panel" style="display: none;">
        <div class="info-grid">
            <div class="info-item">
                <div class="info-label">遊戲版本</div>
                <div id="version" class="info-value">-</div>
            </div>
            <div class="info-item">
                <div class="info-label">在線人數</div>
                <div id="players" class="info-value">-</div>
            </div>
        </div>
        <div id="motd" class="motd"></div>
    </div>
</div>

<script>
    // 👇【請在這裡修改成你的 Minecraft 伺服器 IP】
    const SERVER_IP = 'gemcsv.ddns.net'; 

    document.getElementById('server-ip').innerText = SERVER_IP;

    async function updateStatus() {
        const badge = document.getElementById('status-badge');
        const panel = document.getElementById('info-panel');
        
        try {
            // 使用 mcsrvstat.us 免費 API (支援 Java 版)
            // 如果是基岩版(Bedrock)，請將網址改為 `https://api.mcsrvstat.us/bedrock/2/${SERVER_IP}`
            const response = await fetch(`https://api.mcsrvstat.us/2/${SERVER_IP}`);
            const data = await response.json();

            if (data.online) {
                badge.innerText = "● 伺服器線上 (Online)";
                badge.className = "status online";
                
                document.getElementById('version').innerText = data.version || '未知';
                document.getElementById('players').innerText = `${data.players.online} / ${data.players.max}`;
                
                // 顯示伺服器簡介 (MOTD)
                if(data.motd && data.motd.clean) {
                    document.getElementById('motd').innerText = data.motd.clean.join('\n');
                    document.getElementById('motd').style.display = 'block';
                } else {
                    document.getElementById('motd').style.display = 'none';
                }
                
                panel.style.display = 'block';
            } else {
                badge.innerText = "○ 伺服器離線 (Offline)";
                badge.className = "status offline";
                panel.style.display = 'none';
            }
        } catch (error) {
            badge.innerText = "❌ 偵測失敗 (API 連線錯誤)";
            badge.className = "status offline";
            console.error(error);
        }
    }

    // 立即執行，並設定每 30 秒自動刷新
    updateStatus();
    setInterval(updateStatus, 30000);
</script>

</body>
</html>