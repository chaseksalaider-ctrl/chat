<!doctype html>
<html lang="th">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1,viewport-fit=cover" />
  <title>Chat (Realtime DB) — ห้อง: รวม</title>
  <link href="https://fonts.googleapis.com/css2?family=Prompt:wght@300;400;600&display=swap" rel="stylesheet">
  <style>
    /* --- Responsive base --- */
    :root{
      --bg:#97AFA3;
      --panel:#f7f7f6;
      --bubble-me:#7b8f84;
      --bubble-other:#f8b8b4;
      --accent:#f7aeb0;
      --muted:#6f7f79;
      --max-frame-w:420px;
      --composer-h:72px;
    }

    /* scale font smoothly from small -> large screens */
    html { font-family: "Prompt", sans-serif; font-size: clamp(14px, 2.6vw, 18px); -webkit-text-size-adjust: 100%; box-sizing: border-box; }
    *, *::before, *::after { box-sizing: inherit; }

    body{
      margin:0;
      min-height:100vh;
      background:var(--bg);
      color:#222;
      -webkit-font-smoothing:antialiased;
      -webkit-overflow-scrolling:touch;
    }

    /* frame that adapts to screen */
    .frame{
      width: min(100%, var(--max-frame-w));
      height:100dvh; /* use dynamic vh for mobile chrome */
      margin:0 auto;
      border-radius: 16px;
      overflow: hidden;
      display:flex;
      flex-direction:column;
      background: linear-gradient(180deg, var(--panel), #fff);
      box-shadow: 0 8px 30px rgba(0,0,0,0.12);
      padding: env(safe-area-inset-top) 0 env(safe-area-inset-bottom); /* respect notch */
    }

    /* header */
    .header{
      padding: 12px 12px;
      display:flex;
      align-items:center;
      justify-content:space-between;
      gap:8px;
      flex-wrap:wrap; /* allow wrapping on narrow screens */
    }
    .title{
      font-size: 1.15rem;
      font-weight:700;
      line-height:1;
    }
    .sub{
      font-size:0.85rem;
      color:var(--muted);
    }

    /* controls: allow wrapping so buttons don't overflow */
    .controls{
      display:flex;
      align-items:center;
      gap:8px;
      flex-wrap:wrap;
      min-width:0;
    }

    .pill{
      background:var(--accent);
      color:#4b2e2e;
      padding:6px 10px;
      border-radius:14px;
      font-weight:600;
      white-space:nowrap;
    }

    /* room area: flexible and won't overflow */
    .room-area{
      display:flex;
      gap:8px;
      align-items:center;
      min-width:0;
    }
    select, input[type="text"]{
      padding:6px 10px;
      border-radius:10px;
      border:1px solid rgba(0,0,0,0.08);
      font-size:1rem;
      min-width:0;
    }
    select{ width:120px; flex:0 0 120px; }
    .room-input{ width:120px; flex:1 1 120px; min-width:80px; }

    .room-create{
      padding:8px 10px;
      border-radius:10px;
      background:#fff;
      border:1px solid rgba(0,0,0,0.06);
      cursor:pointer;
      font-size:0.95rem;
      white-space:nowrap;
    }

    /* Name row under header (stack on small screens) */
    .name-row{
      padding: 8px 12px;
      display:flex;
      gap:8px;
      align-items:center;
      flex-wrap:wrap;
    }
    .name-row input{ flex:1 1 140px; min-width:100px; padding:8px; border-radius:10px; border:1px solid rgba(0,0,0,0.06) }
    .name-row .pill{ white-space:nowrap; }

    /* chat area */
    .chat-wrap{
      flex:1;
      overflow:auto;
      padding:12px;
      display:flex;
      flex-direction:column;
      gap:10px;
      /* give bottom padding so last message not hidden by composer */
      padding-bottom: calc(var(--composer-h) + 20px + env(safe-area-inset-bottom));
    }

    .ts-center{ text-align:center;color:var(--muted);font-size:0.95rem;margin:8px 0; }

    /* meta row above bubble */
    .meta-row{
      display:flex;
      align-items:center;
      gap:10px;
      padding:0 6px;
      flex-wrap:wrap;
    }
    .meta-row.me{ justify-content:flex-end; }
    .sender{ font-size:0.95rem; color:var(--muted); font-weight:600; }
    .time{ font-size:0.85rem; color:var(--muted); }

    /* message bubble */
    .message{
      max-width:85%;
      padding:12px;
      border-radius:16px;
      word-wrap:break-word;
      font-size:1rem;
      line-height:1.3;
      position:relative;
      box-shadow: 0 1px 0 rgba(0,0,0,0.03);
    }
    .message.me{ margin-left:auto; background:var(--bubble-me); color:#fff; border-bottom-right-radius:6px; }
    .message.other{ margin-right:auto; background:var(--bubble-other); color:#2f2f2f; border-bottom-left-radius:6px; }

    .del-btn{
      background:transparent;
      border:0;
      cursor:pointer;
      font-size:0.9rem;
      padding:6px 8px;
      border-radius:8px;
    }
    .del-btn.me{
      color:rgba(255,255,255,0.95);
      position:absolute;
      top:6px;
      right:8px;
    }

    /* composer (sticky to bottom, safe-area aware) */
    .composer{
      position:sticky;
      bottom:0;
      background: linear-gradient(180deg, rgba(255,255,255,0.75), rgba(255,255,255,0.95));
      padding:12px;
      padding-bottom: calc(12px + env(safe-area-inset-bottom));
      border-top:1px solid rgba(0,0,0,0.04);
      backdrop-filter: blur(4px);
    }
    .input-row{
      display:flex;
      align-items:center;
      gap:8px;
      background:#efefef;
      padding:10px;
      border-radius:999px;
    }
    .input-row input[type="text"]{
      flex:1;
      border:0;
      background:transparent;
      padding:6px 8px;
      font-size:1rem;
      outline:none;
      min-width:40px;
    }
    .btn{ width:52px; height:52px; border-radius:50%; display:flex; align-items:center; justify-content:center; border:0; cursor:pointer; font-weight:700; flex:0 0 auto; }
    .btn.send{ background:var(--accent); color:#6b2a2a; }

    /* make bubbles and fonts slightly larger on small screens for readability */
    @media (max-width:420px){
      :root{ --composer-h:82px; }
      .message{ padding:14px; border-radius:18px; }
      .btn{ width:48px; height:48px; }
      select{ width:110px; flex-basis:110px; }
      .title{ font-size:1.05rem; }
    }

    /* tablet and above: slightly larger spacing */
    @media (min-width:560px){
      .frame{ border-radius:18px; }
      .chat-wrap{ padding:18px; gap:12px; }
    }

  </style>
</head>
<body>
  <div class="frame" id="app">
    <div class="header">
      <div>
        <div class="title" id="roomTitle">ห้อง: รวม</div>
        <div class="sub">เริ่มต้นที่ห้องรวม (สร้าง/เปลี่ยนได้)</div>
      </div>

      <div class="controls">
        <div class="room-area">
          <select id="roomSelect" title="เลือกห้อง"></select>
          <input id="roomInput" class="room-input" placeholder="ตั้งชื่อห้องใหม่" />
          <button id="createRoomBtn" class="room-create">สร้าง/เข้าห้อง</button>
        </div>
      </div>
    </div>

    <div class="name-row">
      <input id="nameInput" placeholder="ชื่อมึง (ปล่อยว่าง = ไม่ระบุตัวตน)" />
      <button id="saveNameBtn" class="pill">บันทึก</button>
      <div id="displayName" class="pill">ผู้ใช้: ไม่ระบุตัวตน</div>
    </div>

    <div class="chat-wrap" id="chatWrap">
      <!-- messages injected here -->
    </div>

    <div class="composer">
      <div class="input-row">
        <input id="textInput" type="text" placeholder="ข้อความ" autocomplete="off" />
        <button id="sendBtn" class="btn send">ส่ง</button>
      </div>
    </div>
  </div>

  <script type="module">
    import { initializeApp } from "https://www.gstatic.com/firebasejs/11.10.0/firebase-app.js";
    import {
      getDatabase, ref, push, set, onValue, query, orderByChild, get, remove
    } from "https://www.gstatic.com/firebasejs/11.10.0/firebase-database.js";

    // ใส่ค่าจาก Firebase Console ของมึง (Realtime DB)
    const firebaseConfig = {
      apiKey: "AIzaSyDe23wzWDc5yh1GMKuo8eCpHE1m7_967X0",
      authDomain: "messagesel-3bfce.firebaseapp.com",
      databaseURL: "https://messagesel-3bfce-default-rtdb.firebaseio.com",
      projectId: "messagesel-3bfce",
      storageBucket: "messagesel-3bfce.appspot.com",
      messagingSenderId: "1013439867079",
      appId: "1:1013439867079:web:c6146c9124703d6fc876e2",
      measurementId: "G-311ZX3KWS7"
    };

    const app = initializeApp(firebaseConfig);
    const db = getDatabase(app);

    // UI refs
    const chatWrap = document.getElementById("chatWrap");
    const textInput = document.getElementById("textInput");
    const sendBtn = document.getElementById("sendBtn");
    const nameInput = document.getElementById("nameInput");
    const saveNameBtn = document.getElementById("saveNameBtn");
    const displayName = document.getElementById("displayName");
    const roomSelect = document.getElementById("roomSelect");
    const roomInput = document.getElementById("roomInput");
    const createRoomBtn = document.getElementById("createRoomBtn");
    const roomTitle = document.getElementById("roomTitle");

    // local state
    let myName = localStorage.getItem("chat_myname") || "";
    let currentRoom = localStorage.getItem("chat_room") || "รวม";
    let unsubscribeMessages = null;

    function updateNameUI(){ displayName.textContent = "ผู้ใช้: " + (myName ? myName : "ไม่ระบุตัวตน"); }
    function updateRoomUI(){ roomTitle.textContent = "ห้อง: " + currentRoom; localStorage.setItem("chat_room", currentRoom); }

    updateNameUI();
    updateRoomUI();

    saveNameBtn.addEventListener("click", () => {
      myName = nameInput.value.trim();
      localStorage.setItem("chat_myname", myName);
      nameInput.value = "";
      updateNameUI();
    });

    // rooms
    function sanitizeRoomName(name){
      return name.trim().replace(/\s+/g, "_").replace(/[.#$\/\[\]]/g, "_") || "รวม";
    }
    async function ensureDefaultRoom(){
      const snap = await get(ref(db, 'rooms'));
      if (!snap.exists() || !snap.val().hasOwnProperty('รวม')) {
        await set(ref(db, 'rooms/รวม'), { name: "รวม", createdAt: Date.now() });
      }
    }
    async function fetchRooms(){
      const snap = await get(ref(db, 'rooms'));
      const list = [];
      if(snap.exists()){
        const v = snap.val();
        Object.keys(v).forEach(k => list.push({ id: k, ...(v[k] || {}) }));
      }
      return list;
    }
    async function refreshRoomList(selected = currentRoom){
      const rooms = await fetchRooms();
      if(!rooms.find(r=>r.id === 'รวม')){
        await set(ref(db, 'rooms/รวม'), { name: "รวม", createdAt: Date.now() });
      }
      const updated = await fetchRooms();
      roomSelect.innerHTML = "";
      updated.forEach(r=>{
        const opt = document.createElement("option");
        opt.value = r.id;
        opt.textContent = r.name || r.id;
        roomSelect.appendChild(opt);
      });
      if(!Array.from(roomSelect.options).find(o=>o.value===selected)){
        const opt = document.createElement("option");
        opt.value = selected;
        opt.textContent = selected;
        roomSelect.appendChild(opt);
      }
      roomSelect.value = selected;
    }

    createRoomBtn.addEventListener("click", async ()=>{
      const raw = roomInput.value.trim();
      if(!raw) return alert("ใส่ชื่อห้องก่อนดิ");
      const id = sanitizeRoomName(raw);
      try{
        await set(ref(db, `rooms/${id}`), { name: raw, createdAt: Date.now() });
        roomInput.value = "";
        await refreshRoomList(id);
        await switchRoom(id);
      }catch(e){
        console.error("create room err", e);
        alert("สร้างห้องไม่สำเร็จ");
      }
    });

    roomSelect.addEventListener("change", (e)=> switchRoom(e.target.value));

    // messaging
    async function sendMessage(text){
      if(!text) return;
      const newRef = push(ref(db, `rooms/${currentRoom}/messages`));
      await set(newRef, {
        sender: myName || "ผู้ใช้ไม่ระบุตัวตน",
        text: text,
        createdAt: Date.now()
      });
    }
    sendBtn.addEventListener("click", async ()=>{
      const txt = textInput.value.trim();
      if(!txt) return;
      await sendMessage(txt);
      textInput.value = "";
      // keep focus for mobile keyboard convenience
      textInput.focus();
    });
    textInput.addEventListener("keydown", e => { if(e.key === "Enter") sendBtn.click(); });

    function renderMessageRT(id, data){
      const sender = data.sender || "ผู้ใช้ไม่ระบุตัวตน";
      const isMe = (myName && myName === sender);
      const createdAt = data.createdAt ? new Date(data.createdAt) : new Date();

      const metaRow = document.createElement("div");
      metaRow.className = "meta-row " + (isMe ? "me" : "");

      const nameEl = document.createElement("div");
      nameEl.className = "sender";
      nameEl.textContent = sender;

      const timeEl = document.createElement("div");
      timeEl.className = "time";
      timeEl.textContent = createdAt.toLocaleString();

      if(isMe){
        metaRow.appendChild(timeEl);
        metaRow.appendChild(nameEl);
      } else {
        metaRow.appendChild(nameEl);
        metaRow.appendChild(timeEl);
      }

      const bubble = document.createElement("div");
      bubble.className = "message " + (isMe ? "me" : "other");
      bubble.dataset.id = id;

      if(data.text){
        const p = document.createElement("div");
        p.textContent = data.text;
        bubble.appendChild(p);
      }

      if(isMe){
        const del = document.createElement("button");
        del.className = "del-btn me";
        del.textContent = "ลบ";
        del.title = "ลบข้อความของกู";
        del.addEventListener("click", async (e)=>{
          e.stopPropagation();
          if(!confirm("ลบข้อความจริงไหม? คนอื่นจะไม่เห็นแล้ว")) return;
          try{
            await remove(ref(db, `rooms/${currentRoom}/messages/${id}`));
          }catch(err){
            console.error("delete err", err);
            alert("ลบไม่สำเร็จ");
          }
        });
        bubble.appendChild(del);
      }

      const container = document.createElement("div");
      container.appendChild(metaRow);
      container.appendChild(bubble);
      return container;
    }

    let currentListener = null;
    function listenMessages(roomId){
      if(typeof currentListener === 'function') currentListener(); // unsubscribe
      const q = query(ref(db, `rooms/${roomId}/messages`), orderByChild('createdAt'));
      // onValue returns an unsubscribe function when used with modular SDK — we wrap it
      currentListener = onValue(q, (snap) => {
        chatWrap.innerHTML = "";
        if(!snap.exists()){
          const center = document.createElement("div");
          center.className = "ts-center";
          center.textContent = "ยังไม่มีข้อความ — เริ่มคุยเลย";
          chatWrap.appendChild(center);
          return;
        }
        snap.forEach(childSnap => {
          const id = childSnap.key;
          const data = childSnap.val();
          const el = renderMessageRT(id, data);
          chatWrap.appendChild(el);
        });
        // auto-scroll to bottom smoothly on new messages
        chatWrap.scrollTo({ top: chatWrap.scrollHeight, behavior: 'smooth' });
      }, (err) => {
        console.error("onValue err", err);
      });
    }

    async function switchRoom(roomId){
      currentRoom = roomId || "รวม";
      updateRoomUI();
      await refreshRoomList(currentRoom);
      listenMessages(currentRoom);
    }

    // init
    (async function init(){
      try{ await ensureDefaultRoom(); }catch(e){ console.warn(e); }
      await refreshRoomList(currentRoom);
      listenMessages(currentRoom);

      // helpful: focus input on load (mobile will open keyboard if tapped)
      setTimeout(()=> textInput.focus(), 500);
    })();

    // listen to name changes across tabs
    window.addEventListener("storage", (e)=>{
      if(e.key === "chat_myname"){
        myName = localStorage.getItem("chat_myname") || "";
        updateNameUI();
      }
    });
  </script>
</body>
</html>

