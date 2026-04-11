[index.html](https://github.com/user-attachments/files/26643551/index.html)
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <title>미라클 주니 19기 통합 센터</title>
    <style>
        :root { --primary: #F1C40F; --point: #E67E22; --success: #2ECC71; --fail: #E74C3C; --pass: #BDC3C7; --bg: #FFFBF0; --text: #2D3436; }
        body { font-family: 'Pretendard', sans-serif; background: var(--bg); padding: 20px; color: var(--text); }
        .container { max-width: 1400px; margin: 0 auto; }
        .timer-banner { background: #2D3436; color: #F1C40F; text-align: center; padding: 12px; border-radius: 12px; margin-bottom: 20px; font-weight: bold; font-size: 1.1em; letter-spacing: 1px; }
        .tabs { display: flex; gap: 10px; margin-bottom: 20px; }
        .tab { flex: 1; padding: 15px; border-radius: 12px; border: none; background: #eee; font-weight: bold; cursor: pointer; font-size: 1.1em; }
        .tab.active { background: var(--point); color: white; box-shadow: 0 4px 10px rgba(230, 126, 34, 0.3); }
        .card { background: white; padding: 25px; border-radius: 20px; box-shadow: 0 10px 30px rgba(0,0,0,0.05); border: 2px solid #f1f1f1; margin-bottom: 20px; }
        .special-note { background: #FFF9E6; color: #D35400; padding: 8px 15px; border-radius: 8px; font-size: 0.9em; font-weight: bold; margin-bottom: 15px; display: inline-block; border: 1px solid #FFE082; }
        .routine-item { display: flex; justify-content: space-between; align-items: center; padding: 15px 0; border-bottom: 1px solid #eee; }
        .btn-group { display: flex; gap: 5px; }
        .btn { width: 45px; height: 45px; border-radius: 8px; border: 1px solid #ddd; cursor: pointer; font-weight: 900; background: white; }
        .btn.O.active { background: var(--success); color: white; border-color: var(--success); }
        .btn.X.active { background: var(--fail); color: white; border-color: var(--fail); }
        .btn.P.active { background: var(--pass); color: white; border-color: var(--pass); }
        .detail-table-wrapper { overflow-x: auto; margin-top: 15px; }
        .detail-table { width: 100%; border-collapse: collapse; font-size: 0.85em; }
        .detail-table th, .detail-table td { border: 1px solid #eee; padding: 8px; text-align: center; }
        .detail-table th { background: #f8f9fa; }
        .cell-O { color: var(--success); font-weight: bold; }
        .cell-X { color: var(--fail); font-weight: bold; }
        .cell-P { color: var(--pass); font-weight: bold; }
        .admin-area { overflow-x: auto; background: white; padding: 20px; border-radius: 20px; border: 2px solid var(--point); }
        .main-table { width: 100%; border-collapse: collapse; font-size: 11px; min-width: 1200px; }
        .main-table th, .main-table td { border: 1px solid #ddd; padding: 6px 2px; text-align: center; }
        .main-table th { background: #FFF9E6; color: var(--point); position: sticky; top: 0; }
        .sticky-col { position: sticky; left: 0; background: white; z-index: 5; font-weight: bold; border-right: 2px solid var(--point) !important; }
        .rate-col { font-weight: 900; color: #2980b9; background: #f0f7ff; }
        .total-row { background: #FFF9E6; font-weight: 900; }
        #loading { display: none; position: fixed; top:0; left:0; width:100%; height:100%; background:rgba(255,255,255,0.8); z-index:1000; justify-content:center; align-items:center; font-weight:bold; }
    </style>
</head>
<body>

<div id="loading">데이터 처리 중... 🚀</div>

<div class="container">
    <div class="timer-banner" id="deadlineTimer">로딩 중...</div>
    <div class="tabs">
        <button class="tab active" onclick="switchTab('member')">🙋 멤버 인증 및 나의 기록</button>
        <button class="tab" onclick="switchTab('admin')">👑 관리자 대시보드 (비공개)</button>
    </div>

    <div id="memberSection">
        <div class="card">
            <label style="font-weight: 900; color: var(--point);">본인 이름 선택: </label>
            <select id="memberSelect" onchange="loadMemberView()" style="padding:10px; border-radius:8px; width:220px; font-weight:bold; border:2px solid var(--point);">
                <option value="">-- 성함을 선택하세요 --</option>
            </select>
        </div>
        <div id="memberFocusArea" style="display:none;">
            <div class="card" style="border-left: 8px solid var(--success);">
                <h2 id="focusName" style="margin-top:0;"></h2>
                <div id="specialNoteArea"></div>
                <div id="routineList"></div>
            </div>
            <div class="card">
                <h3>📅 나의 4월 상세 루틴 기록표</h3>
                <p id="myRateInfo" style="font-weight:bold; color:var(--point);"></p>
                <div class="detail-table-wrapper"><table class="detail-table" id="myDetailTable"></table></div>
            </div>
        </div>
    </div>

    <div id="adminSection" style="display:none;">
        <div class="admin-area">
            <h2 style="text-align:center; color:var(--point);">📊 4월 실시간 통합 현황</h2>
            <table id="adminTable" class="main-table"></table>
        </div>
    </div>
</div>

<script>
    const SHEET_API_URL = "여기에_URL을_넣으세요"; 
    const ADMIN_PW = "0000"; 
    const DEADLINE_HOUR = 23; 

    const MEMBER_DATA = {
        "곰돌": { items: ["기상시간 5시", "필사", "새벽기도(독서)", "글쓰기"], note: "주말 기상 자유" },
        "글터지기": { items: ["기상시간 5시", "독서", "글쓰기", "산책 (주3회)", "취침시간 10:30분 이전"], note: "일요일 자유" },
        "날위한 나": { items: ["기상시간 6시30분", "아침필사", "글쓰기", "취침시간 10:30분 이전"], note: "취침 시간 금,토 자유" },
        "낭만다정": { items: ["기상시간 6시", "독서(운동, 글쓰기)"], note: "평일 실천" },
        "사랑초": { items: ["기상시간 6시", "독서", "글쓰기", "걷기(수영)"], note: "평일 실천" },
        "들꽃향기": { items: ["기상시간 6시", "글쓰기", "독서", "스트레칭(걷기)"], note: "평일 실천" },
        "보름달": { items: ["기상시간 6시", "독서", "필사"], note: "평일 실천" },
        "비채꿈": { items: ["기상시간 5시30분", "독서", "필사", "경전 읽기", "글쓰기"], note: "평일 실천" },
        "써니허니": { items: ["기상시간 6시", "독서", "공부", "스트레칭(요가)"], note: "평일 실천" },
        "앨리스샘": { items: ["기상시간 4시", "걷기(달리기)", "글쓰기(독서)"], note: "평일 실천" },
        "열정부메랑": { items: ["기상시간 4시30분", "운동(헬스)"], note: "평일 실천" },
        "열정소피아": { items: ["기상시간 4시", "독서", "기도"], note: "" },
        "오빛다": { items: ["기상시간 4시", "영어 공부&리딩", "독서", "필사", "걷기"], note: "평일 실천" },
        "윤슬": { items: ["기상시간 6시30분", "독서", "글쓰기"], note: "주말 기상 7시30분" },
        "즐거운호호씨": { items: ["기상시간 7시", "필사", "말씀"], note: "평일 실천" },
        "지금사랑": { items: ["기상시간 6시", "독서", "저녁식사 조절"], note: "평일 실천" },
        "탈렌트수니": { items: ["기상시간 6시", "운동", "독서", "말씀산책"], note: "" },
        "페이페이": { items: ["기상시간 7시", "글쓰기(주 3회)", "운동(주 3회)"], note: "평일 실천" },
        "하니오웰": { items: ["기상시간 6시", "독서", "글쓰기", "걷기(주3회)"], note: "평일 실천" },
        "mam 마음": { items: ["기상시간 6시", "글쓰기", "걷기"], note: "" },
        "bgmsunny": { items: ["기상시간 5시30분", "명상", "스트레칭(요가)"], note: "평일 실천" }
    };

    let db = [];
    const daysInMonth = 30;
    const weekDays = ['일', '월', '화', '수', '목', '금', '토'];

    // 실시간 날짜를 가져오는 함수
    function getNowInfo() {
        const now = new Date();
        const y = now.getFullYear();
        const m = String(now.getMonth() + 1).padStart(2, '0');
        const d = String(now.getDate()).padStart(2, '0');
        return {
            todayStr: `${y}-${m}-${d}`,
            currentDay: now.getDate()
        };
    }

    async function init() {
        const select = document.getElementById('memberSelect');
        Object.keys(MEMBER_DATA).forEach(name => select.add(new Option(name, name)));
        updateTimer();
        setInterval(updateTimer, 1000);
        await fetchRemoteData();
    }

    function updateTimer() {
        const now = new Date();
        const deadline = new Date(now.getFullYear(), now.getMonth(), now.getDate(), DEADLINE_HOUR, 0, 0);
        let diff = deadline - now;
        if (diff <= 0) { document.getElementById('deadlineTimer').innerHTML = "⏳ 오늘의 루틴 기록이 마감되었습니다!"; return; }
        const h = String(Math.floor((diff / (1000 * 60 * 60)) % 24)).padStart(2, '0');
        const m = String(Math.floor((diff / (1000 * 60)) % 60)).padStart(2, '0');
        const s = String(Math.floor((diff / 1000) % 60)).padStart(2, '0');
        document.getElementById('deadlineTimer').innerHTML = `⏱ 오늘의 기록 마감까지 [ ${h}:${m}:${s} ] 남았습니다`;
    }

    async function fetchRemoteData() {
        if(SHEET_API_URL === "여기에_URL을_넣으세요") {
            db = JSON.parse(localStorage.getItem('miracle_offline_db')) || [];
            renderAdminTable(); return;
        }
        document.getElementById('loading').style.display = 'flex';
        try {
            const response = await fetch(SHEET_API_URL);
            const rawData = await response.json();
            db = rawData.slice(1).map(row => ({ date: row[0], name: row[1], routine: row[2], status: row[3] }));
            renderAdminTable();
        } catch (e) { console.error("Data Load Error"); }
        document.getElementById('loading').style.display = 'none';
    }

    async function syncToRemote(payload) {
        if(SHEET_API_URL === "여기에_URL을_넣으세요") return;
        document.getElementById('loading').style.display = 'flex';
        try { await fetch(SHEET_API_URL, { method: 'POST', body: JSON.stringify(payload) }); } 
        catch (e) { alert("연동 실패"); }
        document.getElementById('loading').style.display = 'none';
    }

    function switchTab(mode) {
        if(mode === 'admin') {
            const pw = prompt("관리자 비밀번호를 입력하세요.");
            if(pw !== ADMIN_PW) { alert("비밀번호가 틀렸습니다."); return; }
        }
        document.getElementById('memberSection').style.display = mode === 'member' ? 'block' : 'none';
        document.getElementById('adminSection').style.display = mode === 'admin' ? 'block' : 'none';
        document.querySelectorAll('.tab').forEach((t, i) => t.classList.toggle('active', (mode==='member'&&i===0)||(mode==='admin'&&i===1)));
        if(mode === 'admin') renderAdminTable();
    }

    async function updateStatus(name, routine, status, targetDate = getNowInfo().todayStr) {
        const payload = { date: targetDate, name: name, routine: routine, status: status };
        db = db.filter(d => !(d.date === targetDate && d.name === name && d.routine === routine));
        if (status !== '대기') db.push(payload);
        localStorage.setItem('miracle_offline_db', JSON.stringify(db));
        await syncToRemote(payload);
        loadMemberView();
        renderAdminTable();
    }

    function getStatus(name, date, routine) {
        const found = db.filter(d => d.date == date && d.name == name && d.routine == routine);
        return found.length > 0 ? found[found.length - 1].status : '대기';
    }

    function loadMemberView() {
        const name = document.getElementById('memberSelect').value;
        if(!name) return;
        const { todayStr, currentDay } = getNowInfo();
        
        document.getElementById('memberFocusArea').style.display = 'block';
        document.getElementById('focusName').innerText = `👋 반갑습니다, ${name}님!`;
        const note = MEMBER_DATA[name].note;
        document.getElementById('specialNoteArea').innerHTML = note ? `<div class="special-note">📌 개인 수칙: ${note}</div>` : '';
        
        const dayName = weekDays[new Date(todayStr).getDay()];
        const list = document.getElementById('routineList');
        list.innerHTML = `<p style="font-weight:bold; color:#666;">오늘(${todayStr} / ${dayName}요일)의 체크</p>`;
        
        MEMBER_DATA[name].items.forEach(r => {
            const s = getStatus(name, todayStr, r);
            list.innerHTML += `<div class="routine-item"><span style="font-size:1.1em; font-weight:800;">${r}</span><div class="btn-group"><button class="btn O ${s==='O'?'active':''}" onclick="updateStatus('${name}','${r}','O')">O</button><button class="btn X ${s==='X'?'active':''}" onclick="updateStatus('${name}','${r}','X')">X</button><button class="btn P ${s==='-'?'active':''}" onclick="updateStatus('${name}','${r}','-')">-</button></div></div>`;
        });

        const myRoutines = MEMBER_DATA[name].items;
        let oCount = 0, pCount = 0;
        let html = `<thead><tr><th>날짜(요일)</th>${myRoutines.map(r => `<th>${r}</th>`).join('')}</tr></thead><tbody>`;
        for(let d=1; d<=daysInMonth; d++) {
            const dObj = new Date(2026, 3, d);
            const dStr = `2026-04-${String(d).padStart(2, '0')}`;
            html += `<tr><td>${d}일(${weekDays[dObj.getDay()]})</td>`;
            myRoutines.forEach(r => {
                const s = getStatus(name, dStr, r);
                if(d <= currentDay) { if(s==='O') oCount++; if(s==='-') pCount++; }
                html += `<td class="cell-${s==='O'?'O':(s==='X'?'X':(s==='-'?'P':''))}">${s==='대기'?'':s}</td>`;
            });
            html += `</tr>`;
        }
        document.getElementById('myDetailTable').innerHTML = html + `</tbody>`;
        const total = (currentDay * myRoutines.length) - pCount;
        const rate = total > 0 ? Math.round((oCount / total) * 100) : 0;
        document.getElementById('myRateInfo').innerText = `🎯 오늘까지 실천 성취율: ${rate}% (패스 ${pCount}회 제외)`;
    }

    function renderAdminTable() {
        const table = document.getElementById('adminTable');
        if (!table) return;
        const { currentDay } = getNowInfo();
        
        let html = `<thead><tr><th class="sticky-col">성함 (수칙)</th><th>루틴 항목</th>`;
        for(let d=1; d<=daysInMonth; d++) html += `<th>${d}<br><small>${weekDays[new Date(2026, 3, d).getDay()]}</small></th>`;
        html += `<th class="rate-col">성취율</th></tr></thead><tbody>`;

        Object.keys(MEMBER_DATA).forEach(name => {
            const myRoutines = MEMBER_DATA[name].items;
            let memberO = 0, memberP = 0;
            myRoutines.forEach((r, idx) => {
                let rO = 0, rP = 0;
                html += `<tr>`;
                if(idx === 0) html += `<td class="sticky-col" rowspan="${myRoutines.length + 1}">${name}<br><small style="color:orange;">${MEMBER_DATA[name].note}</small></td>`;
                html += `<td style="text-align:left;">${r}</td>`;
                for(let d=1; d<=daysInMonth; d++) {
                    const dStr = `2026-04-${String(d).padStart(2, '0')}`;
                    const s = getStatus(name, dStr, r);
                    if(d <= currentDay) { if(s==='O') { rO++; memberO++; } if(s==='-') { rP++; memberP++; } }
                    html += `<td onclick="adminQuickEdit('${name}','${r}','${dStr}')" style="cursor:pointer; font-weight:bold;">${s==='O'?'O':(s==='X'?'X':(s==='-'?'-':''))}</td>`;
                }
                html += `<td class="rate-col">${Math.round((rO / (currentDay - rP)) * 100) || 0}%</td></tr>`;
            });
            const totalRate = Math.round((memberO / (currentDay * myRoutines.length - memberP)) * 100) || 0;
            html += `<tr class="total-row"><td style="background:#FFF9E6;">[종합 성취율]</td>`;
            for(let d=1; d<=daysInMonth; d++) html += `<td style="background:#FFF9E6;"></td>`;
            html += `<td class="rate-col" style="background:var(--primary);">${totalRate}%</td></tr>`;
        });
        table.innerHTML = html + `</tbody>`;
    }

    async function adminQuickEdit(name, routine, date) {
        const cur = getStatus(name, date, routine);
        const next = cur === 'O' ? 'X' : (cur === 'X' ? '-' : (cur === '-' ? '대기' : 'O'));
        db = db.filter(d => !(d.date === date && d.name === name && d.routine === routine));
        if (next !== '대기') db.push({ date: date, name: name, routine: routine, status: next });
        renderAdminTable();
        await syncToRemote({ date: date, name: name, routine: routine, status: next });
    }
    init();
</script>
</body>
</html>
