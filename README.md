<!DOCTYPE html>
<html lang="cs">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no, viewport-fit=cover">
  <meta name="apple-mobile-web-app-capable" content="yes">
  <meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
  <meta name="apple-mobile-web-app-title" content="ADHD HUD">
  <title>ADHD OPPS // HUD</title>
  <style>
    :root {
      --bg: #030803;
      --card: #081408;
      --border: #143a14;
      --text: #4ade80;
      --muted: #166534;
      --accent: #22c55e;
      --alert: #ef4444;
      --amber: #f59e0b;
      --blue: #38bdf8;
    }
    * { box-sizing: border-box; -webkit-tap-highlight-color: transparent; }
    body {
      margin: 0;
      padding: env(safe-area-inset-top) 16px env(safe-area-inset-bottom) 16px;
      background: var(--bg);
      color: var(--text);
      font-family: "Menlo", "Courier New", monospace;
      display: flex;
      flex-direction: column;
      align-items: center;
      min-height: 100vh;
    }
    .hud-box {
      width: 100%;
      max-width: 440px;
      background: var(--card);
      border: 1.5px solid var(--border);
      border-radius: 12px;
      padding: 16px;
      margin-top: 10px;
      box-shadow: 0 0 20px rgba(34, 197, 94, 0.08);
    }
    .header {
      display: flex;
      justify-content: space-between;
      align-items: center;
      font-size: 11px;
      font-weight: bold;
      color: var(--muted);
      border-bottom: 1px solid var(--border);
      padding-bottom: 8px;
    }
    .status-badge {
      color: var(--alert);
      font-size: 12px;
    }
    .countdown-box {
      margin: 14px 0;
      text-align: center;
    }
    .countdown-main {
      font-size: 16px;
      font-weight: bold;
      color: var(--text);
    }
    .countdown-sub {
      font-size: 10px;
      color: var(--muted);
      margin-top: 4px;
    }
    .telemetry-grid {
      display: grid;
      grid-template-columns: repeat(4, 1fr);
      gap: 8px;
      margin: 14px 0;
      text-align: center;
    }
    .tele-card {
      background: rgba(20, 58, 20, 0.3);
      border: 1px solid var(--border);
      border-radius: 6px;
      padding: 8px 4px;
    }
    .tele-label { font-size: 9px; color: var(--muted); font-weight: bold; }
    .tele-val { font-size: 11px; font-weight: bold; margin-top: 4px; color: var(--text); }
    
    .actions-grid {
      display: grid;
      grid-template-columns: 1fr 1fr;
      gap: 10px;
      margin-top: 16px;
    }
    button {
      background: #0d220d;
      border: 1.5px solid var(--border);
      color: var(--text);
      font-family: inherit;
      font-size: 13px;
      font-weight: bold;
      padding: 14px 8px;
      border-radius: 8px;
      cursor: pointer;
      display: flex;
      align-items: center;
      justify-content: center;
      gap: 6px;
      transition: all 0.1s;
    }
    button:active {
      background: var(--accent);
      color: #000;
      transform: scale(0.98);
    }
    button.secured {
      border-color: var(--muted);
      color: var(--muted);
      background: rgba(0,0,0,0.4);
    }
    .btn-full { grid-column: span 2; }
    .btn-alert {
      border-color: #5c1414;
      background: #1c0808;
      color: var(--alert);
    }
    .btn-alert:active {
      background: var(--alert);
      color: #fff;
    }
    .footer-bar {
      display: flex;
      justify-content: space-between;
      font-size: 10px;
      color: var(--muted);
      margin-top: 14px;
      border-top: 1px solid var(--border);
      padding-top: 8px;
    }
  </style>
</head>
<body>

  <div class="hud-box">
    <div class="header">
      <span>/// ADHD_OPPS // HUD</span>
      <span class="status-badge" id="statusBadge">[SYS: IN COMBAT]</span>
    </div>

    <div class="countdown-box">
      <div class="countdown-main" id="countdownMain">>> ODPOČET...</div>
      <div class="countdown-sub" id="countdownSub">STAND DOWN VE 20:00</div>
    </div>

    <div class="telemetry-grid">
      <div class="tele-card">
        <div class="tele-label">MEDIC</div>
        <div class="tele-val" id="teleMedic">[STANDBY]</div>
      </div>
      <div class="tele-card">
        <div class="tele-label">OPS</div>
        <div class="tele-val" id="teleOps">[DROP]</div>
      </div>
      <div class="tele-card">
        <div class="tele-label">PhD</div>
        <div class="tele-val" id="telePhd">[0/3]</div>
      </div>
      <div class="tele-card">
        <div class="tele-label">EXEC</div>
        <div class="tele-val" id="teleExec">[D-00]</div>
      </div>
    </div>

    <div class="actions-grid">
      <button id="btnMedic" onclick="actionMedic()">🧪 DEPLOY MEDIC</button>
      <button id="btnEngaged" onclick="actionEngaged()">🪂 ENGAGED</button>
      <button onclick="actionPhd()">📡 PhD +1h</button>
      <button id="btnExec" onclick="actionExec()">🧱 EXEC_CONTROL</button>
      <button class="btn-full" onclick="actionRetreat()" id="btnRetreat">🚁 TAC_RETREAT</button>
      <button class="btn-full btn-alert" onclick="actionStandDown()">🚨 [PROTOCOL] STAND DOWN</button>
    </div>

    <div class="footer-bar">
      <span id="milestoneTxt">OBJ: T-??D DO 12.</span>
      <span id="debriefTxt" onclick="actionDebrief()" style="cursor:pointer">TAC_DEBRIEF: [□□ 0/2]</span>
    </div>
  </div>

  <script>
    function getTodayStr() {
      const d = new Date();
      return `${d.getFullYear()}-${String(d.getMonth() + 1).padStart(2, '0')}-${String(d.getDate()).padStart(2, '0')}`;
    }

    function getMonthStr() {
      const d = new Date();
      return `${d.getFullYear()}-${String(d.getMonth() + 1).padStart(2, '0')}`;
    }

    let state = {
      date: getTodayStr(),
      currentMonth: getMonthStr(),
      atomoxetin: false,
      engaged: false,
      phdHours: 0,
      streak: 0,
      execConfirmedToday: false,
      debriefCount: 0,
      tacticalRetreat: false
    };

    function loadState() {
      const saved = localStorage.getItem('adhd_opps_state');
      if (saved) {
        try {
          const parsed = JSON.parse(saved);
          const today = getTodayStr();
          const month = getMonthStr();

          if (parsed.currentMonth !== month) {
            parsed.currentMonth = month;
            parsed.debriefCount = 0;
          }

          if (parsed.date !== today) {
            parsed.date = today;
            parsed.atomoxetin = false;
            parsed.engaged = false;
            parsed.phdHours = 0;
            parsed.execConfirmedToday = false;
            parsed.tacticalRetreat = false;
          }
          state = parsed;
        } catch (e) {}
      }
      saveState();
    }

    function saveState() {
      localStorage.setItem('adhd_opps_state', JSON.stringify(state));
      render();
    }

    function actionMedic() {
      state.atomoxetin = true;
      saveState();
    }

    function actionEngaged() {
      state.engaged = true;
      saveState();
    }

    function actionPhd() {
      if (state.phdHours < 3) state.phdHours += 1;
      saveState();
    }

    function actionExec() {
      if (!state.execConfirmedToday) {
        state.streak += 1;
        state.execConfirmedToday = true;
        saveState();
      }
    }

    function actionRetreat() {
      state.tacticalRetreat = !state.tacticalRetreat;
      saveState();
    }

    function actionDebrief() {
      state.debriefCount = state.debriefCount >= 2 ? 0 : state.debriefCount + 1;
      saveState();
    }

    function actionStandDown() {
      alert("⚠️ KLID ZBRANÍ // STAND DOWN\n\n1. Kognitivní klam: Mozek hledá zkratku z vyčerpání.\n2. Vypij velkou sklenici studené vody.\n3. Rychlý cukr (glukóza pro prefrontální exekutivu).\n4. Pravidlo 180 vteřin: Zákaz jakýchkoliv nákupů či finančních operací.");
    }

    function getMilestoneText() {
      const now = new Date();
      const currentDay = now.getDate();
      if (currentDay === 12) return "T-0 (DEN D)";
      let targetYear = now.getFullYear();
      let targetMonth = now.getMonth();
      if (currentDay > 12) {
        targetMonth += 1;
        if (targetMonth > 11) { targetMonth = 0; targetYear += 1; }
      }
      const targetDate = new Date(targetYear, targetMonth, 12);
      const diffDays = Math.ceil((targetDate - now) / (1000 * 60 * 60 * 24));
      return `OBJ: T-${diffDays}D DO 12.`;
    }

    function render() {
      const now = new Date();
      const hr = now.getHours();
      const min = now.getMinutes();
      const dow = now.getDay();
      const isWeekend = (dow === 0 || dow === 6);

      const t20 = new Date(now.getFullYear(), now.getMonth(), now.getDate(), 20, 0, 0);
      const diffTo20 = ((t20 - now) / (1000 * 60 * 60)).toFixed(1);

      // Status badge & odpočet
      const badge = document.getElementById('statusBadge');
      const cMain = document.getElementById('countdownMain');
      const cSub = document.getElementById('countdownSub');

      if (state.tacticalRetreat) {
        badge.innerText = "[SYS: TAC RETREAT]";
        badge.style.color = "var(--amber)";
        cMain.innerText = ">> OPERACE POZASTAVENY";
        cSub.innerText = "REGENERACE POVOLENA";
      } else if (isWeekend) {
        badge.innerText = "[SYS: STRATEGIC HQ]";
        badge.style.color = "var(--blue)";
        cMain.innerText = ">> HQ: VÍKENDOVÝ REŽIM";
        cSub.innerText = "STRATEGICKÉ PLÁNOVÁNÍ";
      } else if (hr >= 20) {
        badge.innerText = "[SYS: AT BASE]";
        badge.style.color = "var(--blue)";
        cMain.innerText = ">> CEASEFIRE V PLATNOSTI";
        cSub.innerText = "ZÁKAZ PRÁCE NA PHD // KLIDOVÝ REŽIM";
      } else {
        badge.innerText = "[SYS: IN COMBAT]";
        badge.style.color = "var(--alert)";
        cMain.innerText = `>> T-${diffTo20}H DO CEASEFIRE (20:00)`;
        cSub.innerText = "STAND DOWN VE 20:00 // MAX 3H PHD ZA DEN";
      }

      // Telemetrie
      const teleMedic = document.getElementById('teleMedic');
      teleMedic.innerText = state.atomoxetin ? "[IN POS ✓]" : "[STANDBY]";
      teleMedic.style.color = state.atomoxetin ? "var(--accent)" : "var(--alert)";

      const teleOps = document.getElementById('teleOps');
      teleOps.innerText = state.engaged ? "[ENGAGED ✓]" : "[DROP ZONE]";
      teleOps.style.color = state.engaged ? "var(--accent)" : "var(--text)";

      const telePhd = document.getElementById('telePhd');
      let bStr = "";
      for (let i = 1; i <= 3; i++) bStr += (i <= state.phdHours) ? "■" : "□";
      telePhd.innerText = `[${bStr} ${state.phdHours}/3]`;
      telePhd.style.color = state.phdHours === 3 ? "var(--accent)" : "var(--text)";

      const teleExec = document.getElementById('teleExec');
      teleExec.innerText = `[D-${String(state.streak).padStart(2, '0')} ${state.execConfirmedToday ? "✓" : "!"}]`;
      teleExec.style.color = state.execConfirmedToday ? "var(--accent)" : "var(--muted)";

      // Tlačítka stav
      document.getElementById('btnMedic').className = state.atomoxetin ? "secured" : "";
      document.getElementById('btnEngaged').className = state.engaged ? "secured" : "";
      document.getElementById('btnExec').className = state.execConfirmedToday ? "secured" : "";
      document.getElementById('btnRetreat').innerText = state.tacticalRetreat ? "🚁 RETURN TO OPS" : "🚁 TAC_RETREAT";

      // Footer
      document.getElementById('milestoneTxt').innerText = getMilestoneText();
      let dBlocks = "";
      for (let i = 1; i <= 2; i++) dBlocks += (i <= state.debriefCount) ? "■" : "□";
      document.getElementById('debriefTxt').innerText = `TAC_DEBRIEF: [${dBlocks} ${state.debriefCount}/2]`;
    }

    loadState();
    setInterval(render, 30000); // refresh každých 30 vteřin
  </script>
</body>
</html>
