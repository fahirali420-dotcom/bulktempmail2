<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Bhai-Mail: Public Bulk Generator</title>
    <style>
        body { font-family: 'Segoe UI', sans-serif; background: #0f172a; color: #f8fafc; padding: 20px; }
        .box { max-width: 900px; margin: auto; background: #1e293b; padding: 30px; border-radius: 15px; box-shadow: 0 10px 15px rgba(0,0,0,0.3); }
        input, button { padding: 12px; border-radius: 8px; border: none; margin: 5px; }
        input { background: #334155; color: white; width: 200px; }
        button { background: #3b82f6; color: white; cursor: pointer; font-weight: bold; transition: 0.3s; }
        button:hover { background: #2563eb; }
        .grid { display: grid; grid-template-columns: 1fr 1fr; gap: 20px; margin-top: 20px; }
        .card { background: #0f172a; padding: 15px; border-radius: 10px; border-left: 5px solid #3b82f6; }
        .scroll { max-height: 400px; overflow-y: auto; }
        .msg-box { background: #334155; padding: 10px; margin-top: 10px; border-radius: 5px; font-size: 14px; }
    </style>
</head>
<body>

<div class="box">
    <h1>📩 Bhai-Mail Public Tool</h1>
    
    <!-- Section 1: Bulk Generator -->
    <div style="border-bottom: 1px solid #334155; padding-bottom: 20px;">
        <h3>1. Bulk Generate (with Tokens)</h3>
        <input type="number" id="bulkCount" value="5" placeholder="Qty">
        <button onclick="generateBulk()">Generate & Download CSV</button>
        <p id="status" style="color: #94a3b8; font-size: 14px;"></p>
    </div>

    <div class="grid">
        <!-- Section 2: Public Inbox Search -->
        <div>
            <h3>2. Check Public Inbox</h3>
            <input type="text" id="searchEmail" placeholder="email@example.com">
            <input type="password" id="searchPass" placeholder="Password">
            <button onclick="checkInbox()">Check Mail</button>
        </div>

        <!-- Section 3: Messages Display -->
        <div class="card scroll">
            <h3>Recent Messages</h3>
            <div id="messages">Inbox khali hai ya details galat hain...</div>
        </div>
    </div>
</div>

<script>
    const API = "https://mail.tm";

    // 1. Bulk Generation Logic
    async function generateBulk() {
        const count = document.getElementById('bulkCount').value;
        const status = document.getElementById('status');
        let csv = "Email,Password,Token\n";

        try {
            const domRes = await fetch(`${API}/domains`);
            const domData = await domRes.json();
            const domain = domData['hydra:member'][0].domain;

            for(let i=0; i<count; i++) {
                status.innerText = `Bana raha hoon: ${i+1}/${count}`;
                const user = `bhai_${Math.random().toString(36).substring(7)}@${domain}`;
                const pass = "BhaiPass786!";

                await fetch(`${API}/accounts`, {
                    method: 'POST',
                    headers: {'Content-Type': 'application/json'},
                    body: JSON.stringify({address: user, password: pass})
                });

                const tkRes = await fetch(`${API}/token`, {
                    method: 'POST',
                    headers: {'Content-Type': 'application/json'},
                    body: JSON.stringify({address: user, password: pass})
                });
                const tkData = await tkRes.json();
                
                csv += `${user},${pass},${tkData.token}\n`;
                await new Promise(r => setTimeout(r, 1000));
            }

            downloadCSV(csv);
            status.innerText = "✅ CSV Download ho gayi!";
        } catch (e) { status.innerText = "Error: " + e.message; }
    }

    // 2. Inbox Check Logic
    async function checkInbox() {
        const email = document.getElementById('searchEmail').value;
        const pass = document.getElementById('searchPass').value;
        const msgDiv = document.getElementById('messages');

        try {
            msgDiv.innerHTML = "Loading...";
            const tkRes = await fetch(`${API}/token`, {
                method: 'POST',
                headers: {'Content-Type': 'application/json'},
                body: JSON.stringify({address: email, password: pass})
            });
            const tkData = await tkRes.json();

            const msgRes = await fetch(`${API}/messages`, {
                headers: {'Authorization': `Bearer ${tkData.token}`}
            });
            const msgData = await msgRes.json();

            msgDiv.innerHTML = msgData['hydra:member'].length ? "" : "Koi message nahi mila.";
            msgData['hydra:member'].forEach(m => {
                msgDiv.innerHTML += `<div class="msg-box"><b>From:</b> ${m.from.address}<br><b>Sub:</b> ${m.subject}</div>`;
            });
        } catch (e) { msgDiv.innerHTML = "❌ Login Failed!"; }
    }

    function downloadCSV(content) {
        const blob = new Blob([content], {type: 'text/csv'});
        const url = window.URL.createObjectURL(blob);
        const a = document.createElement('a');
        a.href = url;
        a.download = 'bhai_bulk_mail.csv';
        a.click();
    }
</script>

</body>
</html>
