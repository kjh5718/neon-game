<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>NEON OVERDRIVE: COSMIC SURVIVOR</title>
    <style>
        * { box-sizing: border-box; margin: 0; padding: 0; user-select: none; -webkit-user-select: none; }
        body { background: #05050a; color: #fff; font-family: 'Segoe UI', Roboto, Helvetica, Arial, sans-serif; overflow: hidden; width: 100vw; height: 100vh; display: flex; justify-content: center; align-items: center; }
        #gameContainer { position: relative; width: 100%; max-width: 480px; height: 100%; max-height: 850px; background: #080812; overflow: hidden; box-shadow: 0 0 40px rgba(0, 243, 255, 0.2); border: 1px solid rgba(0, 243, 255, 0.3); }
        canvas { display: block; width: 100%; height: 100%; background: #05050f; }
        
        /* UI 레이어 공통 */
        .ui-screen { position: absolute; top: 0; left: 0; width: 100%; height: 100%; display: flex; flex-direction: column; justify-content: center; align-items: center; background: rgba(5, 5, 15, 0.88); backdrop-filter: blur(8px); z-index: 10; padding: 24px; text-align: center; transition: opacity 0.3s ease; }
        .hidden { display: none !important; opacity: 0; pointer-events: none; }

        h1 { font-size: 2rem; font-weight: 900; background: linear-gradient(135deg, #00f3ff, #ff0055, #ffe600); -webkit-background-clip: text; -webkit-text-fill-color: transparent; text-shadow: 0 0 20px rgba(0,243,255,0.4); margin-bottom: 8px; letter-spacing: 2px; }
        p.subtitle { font-size: 0.9rem; color: #8ab4f8; margin-bottom: 24px; letter-spacing: 1px; }

        .input-group { width: 100%; max-width: 300px; margin-bottom: 20px; text-align: left; }
        .input-group label { display: block; font-size: 0.8rem; color: #00f3ff; margin-bottom: 6px; font-weight: bold; letter-spacing: 1px; }
        .input-group input { width: 100%; padding: 12px 16px; background: rgba(0, 243, 255, 0.05); border: 1px solid rgba(0, 243, 255, 0.4); border-radius: 8px; color: #fff; font-size: 1rem; outline: none; transition: all 0.2s; }
        .input-group input:focus { border-color: #00f3ff; box-shadow: 0 0 12px rgba(0, 243, 255, 0.4); background: rgba(0, 243, 255, 0.1); }

        .btn { background: linear-gradient(135deg, #00f3ff, #0066ff); color: #000; border: none; padding: 14px 32px; font-size: 1.1rem; font-weight: 900; border-radius: 8px; cursor: pointer; box-shadow: 0 0 20px rgba(0, 243, 255, 0.5); transition: all 0.2s ease; text-transform: uppercase; letter-spacing: 1px; margin-top: 10px; width: 100%; max-width: 300px; }
        .btn:hover { transform: translateY(-2px); box-shadow: 0 0 30px rgba(0, 243, 255, 0.8); background: linear-gradient(135deg, #33f5ff, #3388ff); }
        .btn:active { transform: translateY(1px); }
        .btn.secondary { background: rgba(255,255,255,0.1); color: #fff; border: 1px solid rgba(255,255,255,0.3); box-shadow: none; margin-top: 10px; }
        .btn.secondary:hover { background: rgba(255,255,255,0.2); }

        /* 리더보드 패널 */
        .leaderboard-box { width: 100%; max-width: 320px; background: rgba(0,0,0,0.4); border: 1px solid rgba(255,255,255,0.1); border-radius: 8px; padding: 12px; margin-bottom: 20px; max-height: 160px; overflow-y: auto; text-align: left; }
        .leaderboard-title { font-size: 0.8rem; color: #ffe600; font-weight: bold; margin-bottom: 8px; display: flex; justify-content: space-between; align-items: center; }
        .lb-item { display: flex; justify-content: space-between; font-size: 0.85rem; padding: 4px 0; border-bottom: 1px solid rgba(255,255,255,0.05); color: #ccc; }
        .lb-item span:first-child { color: #fff; font-weight: 500; }
        .lb-item span.highlight { color: #00f3ff; font-weight: bold; }

        /* 인게임 HUD */
        #hud { position: absolute; top: 0; left: 0; width: 100%; padding: 16px; display: flex; justify-content: space-between; align-items: flex-start; pointer-events: none; z-index: 5; font-weight: bold; text-shadow: 0 0 8px rgba(0,0,0,0.8); }
        .hud-left { display: flex; flex-direction: column; gap: 4px; }
        .pilot-tag { font-size: 0.8rem; color: #00f3ff; background: rgba(0, 243, 255, 0.15); padding: 2px 8px; border-radius: 4px; border: 1px solid rgba(0, 243, 255, 0.3); width: fit-content; }
        .score-display { font-size: 1.3rem; color: #fff; font-family: monospace; }
        .wave-display { font-size: 0.85rem; color: #ffe600; }
        .hud-right { display: flex; flex-direction: column; align-items: flex-end; gap: 6px; }
        .hp-bar-container { width: 100px; height: 10px; background: rgba(255,255,255,0.1); border-radius: 5px; overflow: hidden; border: 1px solid rgba(255,255,255,0.3); }
        #hpFill { width: 100%; height: 100%; background: linear-gradient(90deg, #ff0055, #00f3ff); transition: width 0.2s; }
        .target-rank-info { font-size: 0.75rem; color: #ff0055; background: rgba(255, 0, 85, 0.15); padding: 2px 6px; border-radius: 4px; border: 1px solid rgba(255, 0, 85, 0.3); }

        /* 레벨업 카드 선택 오버레이 */
        #upgradeScreen { position: absolute; top: 0; left: 0; width: 100%; height: 100%; background: rgba(5, 5, 15, 0.9); display: flex; flex-direction: column; justify-content: center; align-items: center; z-index: 20; padding: 20px; }
        .upgrade-grid { display: flex; flex-direction: column; gap: 12px; width: 100%; max-width: 360px; margin-top: 16px; }
        .upgrade-card { background: rgba(0, 243, 255, 0.08); border: 1px solid rgba(0, 243, 255, 0.4); border-radius: 12px; padding: 16px; cursor: pointer; transition: all 0.2s; text-align: left; }
        .upgrade-card:hover { background: rgba(0, 243, 255, 0.2); border-color: #00f3ff; transform: scale(1.02); box-shadow: 0 0 15px rgba(0, 243, 255, 0.3); }
        .upgrade-card h3 { font-size: 1rem; color: #00f3ff; margin-bottom: 4px; }
        .upgrade-card p { font-size: 0.8rem; color: #bbb; }

        /* 토스트 알림 */
        #toast { position: absolute; bottom: 30px; left: 50%; transform: translateX(-50%) translateY(50px); background: rgba(0, 243, 255, 0.9); color: #000; padding: 8px 16px; border-radius: 20px; font-weight: bold; font-size: 0.85rem; z-index: 100; opacity: 0; transition: all 0.3s ease; pointer-events: none; box-shadow: 0 0 15px rgba(0,243,255,0.6); }
        #toast.show { transform: translateX(-50%) translateY(0); opacity: 1; }

        .url-box { width: 100%; max-width: 300px; background: rgba(0,0,0,0.6); border: 1px dashed rgba(0,243,255,0.5); padding: 8px; border-radius: 6px; font-size: 0.75rem; color: #00f3ff; word-break: break-all; margin-top: 8px; display: none; text-align: center; }
    </style>
</head>
<body>

<div id="gameContainer">
    <canvas id="gameCanvas"></canvas>

    <!-- 인게임 HUD -->
    <div id="hud" class="hidden">
        <div class="hud-left">
            <div class="pilot-tag" id="hudPilotName">PILOT-742</div>
            <div class="score-display" id="hudScore">000000</div>
            <div class="wave-display" id="hudWave">WAVE 1</div>
        </div>
        <div class="hud-right">
            <div class="hp-bar-container"><div id="hpFill"></div></div>
            <div class="target-rank-info" id="hudTargetRank">👑 1위 목표: LV.5</div>
        </div>
    </div>

    <!-- 타이틀 화면 -->
    <div id="startScreen" class="ui-screen">
        <h1>NEON OVERDRIVE</h1>
        <p class="subtitle">COSMIC SURVIVOR</p>
        
        <div class="input-group">
            <label>파일럿 호출부호 (닉네임)</label>
            <input type="text" id="startPilotNameInput" placeholder="닉네임 입력 (최대 10자)" maxlength="10" value="">
        </div>

        <div class="leaderboard-box">
            <div class="leaderboard-title"><span>🏆 최고 도달 레벨 랭킹</span><span id="lbCount">TOP 5</span></div>
            <div id="leaderboardList">
                <div class="lb-item"><span>1. 사이버고스트</span><span class="highlight">LV. 14</span></div>
                <div class="lb-item"><span>2. 네온스톰</span><span class="highlight">LV. 11</span></div>
                <div class="lb-item"><span>3. 플라즈마X</span><span class="highlight">LV. 8</span></div>
            </div>
        </div>

        <button class="btn" id="startBtn">🚀 출격하기</button>
    </div>

    <!-- 레벨업 업그레이드 화면 -->
    <div id="upgradeScreen" class="hidden">
        <h1>⚡ SYSTEM UPGRADE</h1>
        <p class="subtitle">전술 모듈을 선택하세요</p>
        <div class="upgrade-grid" id="upgradeGrid"></div>
    </div>

    <!-- 게임 오버 화면 -->
    <div id="gameOverScreen" class="ui-screen hidden">
        <h1>MISSION FAILED</h1>
        <p class="subtitle" id="finalResultText">최종 점수: 0 | 도달 레벨: LV.1</p>
        
        <div class="input-group">
            <label>기록 등록용 파일럿 명칭</label>
            <input type="text" id="pilotNameInput" placeholder="닉네임 입력" maxlength="10">
        </div>

        <button class="btn" id="restartBtn">🔄 다시 출격</button>
        <button class="btn secondary" id="shareBtn">🔗 기록 등록 & 도전장 링크 복사</button>
        <div class="url-box" id="shareUrlBox">링크가 복사되었습니다!</div>
    </div>

    <!-- 토스트 메시지 -->
    <div id="toast">링크가 클립보드에 복사되었습니다!</div>
</div>

<script>
    const canvas = document.getElementById('gameCanvas');
    const ctx = canvas.getContext('2d');

    function resizeCanvas() {
        canvas.width = canvas.parentElement.clientWidth;
        canvas.height = canvas.parentElement.clientHeight;
    }
    window.addEventListener('resize', resizeCanvas);
    resizeCanvas();

    // 오디오 컨텍스트 설정
    let audioCtx = null;
    function initAudio() {
        if (!audioCtx) {
            audioCtx = new (window.AudioContext || window.webkitAudioContext)();
        }
    }
    function playSound(type) {
        if (!audioCtx) return;
        try {
            const osc = audioCtx.createOscillator();
            const gain = audioCtx.createGain();
            osc.connect(gain);
            gain.connect(audioCtx.destination);
            const now = audioCtx.currentTime;

            if (type === 'shoot') {
                osc.type = 'sawtooth';
                osc.frequency.setValueAtTime(400, now);
                osc.frequency.exponentialRampToValueAtTime(100, now + 0.1);
                gain.gain.setValueAtTime(0.15, now);
                gain.gain.linearRampToValueAtTime(0.01, now + 0.1);
                osc.start(now);
                osc.stop(now + 0.1);
            } else if (type === 'explosion') {
                osc.type = 'square';
                osc.frequency.setValueAtTime(120, now);
                osc.frequency.exponentialRampToValueAtTime(30, now + 0.3);
                gain.gain.setValueAtTime(0.3, now);
                gain.gain.linearRampToValueAtTime(0.01, now + 0.3);
                osc.start(now);
                osc.stop(now + 0.3);
            } else if (type === 'levelup') {
                osc.type = 'sine';
                osc.frequency.setValueAtTime(300, now);
                osc.frequency.exponentialRampToValueAtTime(800, now + 0.2);
                gain.gain.setValueAtTime(0.2, now);
                gain.gain.linearRampToValueAtTime(0.01, now + 0.2);
                osc.start(now);
                osc.stop(now + 0.2);
            }
        } catch(e) {}
    }

    // 게임 상태 변수
    let gameState = 'START'; // START, PLAYING, UPGRADE, GAMEOVER
    let score = 0, wave = 1, level = 1, exp = 0, nextExp = 100;
    let player = {
        x: canvas.width / 2, y: canvas.height - 120, radius: 18,
        speed: 6, hp: 100, maxHp: 100, fireRate: 12, fireTimer: 0,
        multishot: 1, laser: false, shield: false, missiles: false
    };

    let bullets = [], enemies = [], particles = [], floatTexts = [], enemyBullets = [];
    let keys = {};
    let touchPos = { x: null, y: null, active: false };
    let screenShakeTimer = 0, screenShakeIntensity = 0;
    let challengePilot = null, challengeLevel = 0;

    // 로컬 리더보드 관리
    let leaderboard = JSON.parse(localStorage.getItem('neon_lb') || JSON.stringify([
        { name: '사이버고스트', level: 14, score: 35000 },
        { name: '네온스톰', level: 11, score: 24000 },
        { name: '플라즈마X', level: 8, score: 16000 }
    ]));

    function updateLeaderboardUI() {
        const listEl = document.getElementById('leaderboardList');
        listEl.innerHTML = '';
        leaderboard.sort((a,b) => b.level - a.level || b.score - a.score);
        leaderboard.slice(0, 5).forEach((item, idx) => {
            const div = document.createElement('div');
            div.className = 'lb-item';
            div.innerHTML = `<span>${idx+1}. ${item.name}</span><span class="highlight">LV. ${item.level}</span>`;
            listEl.appendChild(div);
        });
        if(leaderboard.length > 0) {
            document.getElementById('hudTargetRank').innerText = `👑 1위 목표: LV.${leaderboard[0].level}`;
        }
    }
    updateLeaderboardUI();

    // URL 도전장 체크
    window.addEventListener('load', () => {
        const hash = window.location.hash;
        if (hash.includes('challenge=')) {
            try {
                const dataStr = atob(hash.split('challenge=')[1]);
                const data = JSON.parse(dataStr);
                challengePilot = data.name;
                challengeLevel = data.level;
                showToast(`🔥 [${challengePilot}]님이 LV.${challengeLevel} 도전에 초대했습니다!`);
                document.getElementById('hudTargetRank').innerText = `🔥 도전목표: ${challengePilot} (LV.${challengeLevel})`;
            } catch(e) {}
        }
    });

    // 입력 이벤트 리스너
    window.addEventListener('keydown', e => { keys[e.code] = true; initAudio(); });
    window.addEventListener('keyup', e => { keys[e.code] = false; });

    canvas.addEventListener('pointerdown', e => {
        touchPos.active = true;
        const rect = canvas.getBoundingClientRect();
        touchPos.x = e.clientX - rect.left;
        touchPos.y = e.clientY - rect.top;
        initAudio();
    });
    canvas.addEventListener('pointermove', e => {
        if (!touchPos.active) return;
        const rect = canvas.getBoundingClientRect();
        touchPos.x = e.clientX - rect.left;
        touchPos.y = e.clientY - rect.top;
    });
    canvas.addEventListener('pointerup', () => { touchPos.active = false; });

    // UI 버튼 연결
    document.getElementById('startBtn').addEventListener('click', () => {
        initAudio();
        const pName = document.getElementById('startPilotNameInput').value.trim() || `PILOT-${Math.floor(100+Math.random()*900)}`;
        document.getElementById('pilotNameInput').value = pName;
        document.getElementById('hudPilotName').innerText = pName;
        
        document.getElementById('startScreen').classList.add('hidden');
        document.getElementById('hud').classList.remove('hidden');
        resetGame();
        gameState = 'PLAYING';
    });

    document.getElementById('restartBtn').addEventListener('click', () => {
        document.getElementById('gameOverScreen').classList.add('hidden');
        resetGame();
        gameState = 'PLAYING';
    });

    document.getElementById('shareBtn').addEventListener('click', () => {
        const pName = document.getElementById('pilotNameInput'].value.trim() || '익명의 파일럿';
        const payload = { name: pName, level: level, score: score };
        const encoded = btoa(JSON.stringify(payload));
        const shareUrl = window.location.origin + window.location.pathname + '#challenge=' + encoded;

        navigator.clipboard.writeText(shareUrl).then(() => {
            const box = document.getElementById('shareUrlBox');
            box.style.display = 'block';
            box.innerText = shareUrl;
            showToast('도전장 링크가 복사되었습니다!');
        }).catch(() => {
            prompt('아래 링크를 복사하여 공유하세요:', shareUrl);
        });
    });

    function showToast(msg) {
        const t = document.getElementById('toast');
        t.innerText = msg;
        t.classList.add('show');
        setTimeout(() => t.classList.remove('show'), 3000);
    }

    function resetGame() {
        score = 0; wave = 1; level = 1; exp = 0; nextExp = 100;
        player.x = canvas.width / 2; player.y = canvas.height - 120;
        player.hp = 100; player.maxHp = 100;
        player.multishot = 1; player.laser = false; player.shield = false; player.missiles = false;
        bullets = []; enemies = []; particles = []; floatTexts = []; enemyBullets = [];
    }

    // 업그레이드 모듈 목록
    const upgradePool = [
        { title: '플라즈마 멀티샷 강화', desc: '발사되는 탄환 수가 증가합니다.', apply: () => { player.multishot = Math.min(5, player.multishot + 1); } },
        { title: '이지스 에너지 쉴드', desc: '최대 체력을 증가시키고 체력을 회복합니다.', apply: () => { player.maxHp += 30; player.hp = Math.min(player.maxHp, player.hp + 50); } },
        { title: '네온 추적 미사일', desc: '주기적으로 적을 추적하는 미사일을 발사합니다.', apply: () => { player.missiles = true; } },
        { title: '오비탈 레이저 소형화', desc: '기체 주변을 회전하는 광선 위성을 추가합니다.', apply: () => { player.laser = true; } }
    ];

    function triggerLevelUp() {
        gameState = 'UPGRADE';
        playSound('levelup');
        const grid = document.getElementById('upgradeGrid');
        grid.innerHTML = '';
        
        // 무작위 2개 선택
        const shuffled = [...upgradePool].sort(() => 0.5 - Math.random());
        shuffled.slice(0, 2).forEach(upg => {
            const card = document.createElement('div');
            card.className = 'upgrade-card';
            card.innerHTML = `<h3>⚡ ${upg.title}</h3><p>${upg.desc}</p>`;
            card.onclick = () => {
                upg.apply();
                document.getElementById('upgradeScreen').classList.add('hidden');
                gameState = 'PLAYING';
            };
            grid.appendChild(card);
        });
        document.getElementById('upgradeScreen').classList.remove('hidden');
    }

    function addExplosion(x, y, color = '#00f3ff', count = 20) {
        for (let i = 0; i < count; i++) {
            const angle = Math.random() * Math.PI * 2;
            const speed = Math.random() * 4 + 1;
            particles.push({
                x, y, vx: Math.cos(angle) * speed, vy: Math.sin(angle) * speed,
                life: 1, decay: Math.random() * 0.03 + 0.015, color, size: Math.random() * 3 + 1
            });
        }
        screenShakeTimer = 10;
        screenShakeIntensity = 6;
    }

    // 메인 게임 루프
    let lastTime = 0;
    function gameLoop(timestamp) {
        if (!lastTime) lastTime = timestamp;
        let dt = (timestamp - lastTime) / 1000;
        if (dt > 0.1) dt = 0.1;
        lastTime = timestamp;

        update(dt);
        render();
        requestAnimationFrame(gameLoop);
    }

    function update(dt) {
        if (gameState !== 'PLAYING') return;

        // 플레이어 이동
        let dx = 0, dy = 0;
        if (keys['ArrowLeft'] || keys['KeyA']) dx = -1;
        if (keys['ArrowRight'] || keys['KeyD']) dx = 1;
        if (keys['ArrowUp'] || keys['KeyW']) dy = -1;
        if (keys['ArrowDown'] || keys['KeyS']) dy = 1;

        if (touchPos.active && touchPos.x !== null) {
            const targetDx = touchPos.x - player.x;
            const targetDy = touchPos.y - player.y;
            const dist = Math.hypot(targetDx, targetDy);
            if (dist > 2) {
                player.x += (targetDx / dist) * Math.min(player.speed * 1.5, dist);
                player.y += (targetDy / dist) * Math.min(player.speed * 1.5, dist);
            }
        } else if (dx !== 0 || dy !== 0) {
            const len = Math.hypot(dx, dy);
            player.x += (dx / len) * player.speed;
            player.y += (dy / len) * player.speed;
        }

        // 화면 경계 제한
        player.x = Math.max(25, Math.min(canvas.width - 25, player.x));
        player.y = Math.max(40, Math.min(canvas.height - 40, player.y));

        // 자동 발사
        player.fireTimer++;
        if (player.fireTimer >= player.fireRate) {
            player.fireTimer = 0;
            playSound('shoot');
            if (player.multishot === 1) {
                bullets.push({ x: player.x, y: player.y - 20, vx: 0, vy: -12, radius: 4 });
            } else if (player.multishot === 2) {
                bullets.push({ x: player.x - 8, y: player.y - 20, vx: 0, vy: -12, radius: 4 });
                bullets.push({ x: player.x + 8, y: player.y - 20, vx: 0, vy: -12, radius: 4 });
            } else if (player.multishot >= 3) {
                bullets.push({ x: player.x, y: player.y - 25, vx: 0, vy: -13, radius: 4 });
                bullets.push({ x: player.x - 12, y: player.y - 15, vx: -2, vy: -12, radius: 3 });
                bullets.push({ x: player.x + 12, y: player.y - 15, vx: 2, vy: -12, radius: 3 });
                if (player.multishot >= 5) {
                    bullets.push({ x: player.x - 20, y: player.y - 10, vx: -4, vy: -11, radius: 3 });
                    bullets.push({ x: player.x + 20, y: player.y - 10, vx: 4, vy: -11, radius: 3 });
                }
            }
        }

        // 총알 업데이트
        bullets.forEach(b => { b.x += b.vx; b.y += b.vy; });
        bullets = bullets.filter(b => b.y > -20 && b.x > 0 && b.x < canvas.width);

        // 적 스폰
        if (Math.random() < 0.02 + (wave * 0.003)) {
            const type = Math.random() < 0.8 ? 'drone' : 'tank';
            enemies.push({
                x: Math.random() * (canvas.width - 40) + 20,
                y: -30,
                vx: (Math.random() - 0.5) * 2,
                vy: type === 'drone' ? (Math.random() * 2 + 2) : (Math.random() * 1 + 1),
                radius: type === 'drone' ? 14 : 22,
                hp: type === 'drone' ? 1 : 4,
                maxHp: type === 'drone' ? 1 : 4,
                type: type
            });
        }

        // 적 업데이트
        enemies.forEach(en => {
            en.x += en.vx;
            en.y += en.vy;
            // 플레이어 충돌
            if (Math.hypot(player.x - en.x, player.y - en.y) < player.radius + en.radius) {
                player.hp -= 15;
                en.hp = 0;
                addExplosion(en.x, en.y, '#ff0055', 15);
            }
        });
        enemies = enemies.filter(en => en.y < canvas.height + 50 && en.hp > 0);

        // 탄환과 적 충돌 체크
        bullets.forEach((b, bi) => {
            enemies.forEach((en, ei) => {
                if (Math.hypot(b.x - en.x, b.y - en.y) < b.radius + en.radius) {
                    en.hp--;
                    b.toRemove = true;
                    if (en.hp <= 0) {
                        addExplosion(en.x, en.y, '#00f3ff', 12);
                        playSound('explosion');
                        score += en.type === 'drone' ? 100 : 300;
                        exp += en.type === 'drone' ? 25 : 60;
                        
                        // 레벨업 체크
                        if (exp >= nextExp) {
                            exp -= nextExp;
                            level++;
                            nextExp = Math.floor(nextExp * 1.3);
                            triggerLevelUp();
                        }
                    }
                }
            });
        });
        bullets = bullets.filter(b => !b.toRemove);

        // 파티클 업데이트
        particles.forEach(p => {
            p.x += p.vx; p.y += p.vy;
            p.life -= p.decay;
        });
        particles = particles.filter(p => p.life > 0);

        // HUD 반영
        document.getElementById('hudScore').innerText = String(score).padStart(6, '0');
        document.getElementById('hudWave').innerText = `WAVE ${wave} (LV.${level})`;
        document.getElementById('hpFill').style.width = `${Math.max(0, (player.hp / player.maxHp) * 100)}%`;

        // 게임 오버 체크
        if (player.hp <= 0) {
            gameState = 'GAMEOVER';
            document.getElementById('hud').classList.add('hidden');
            document.getElementById('finalResultText').innerText = `최종 점수: ${score}점 | 도달 레벨: LV.${level}`;
            
            // 자동 랭킹 저장
            const currentName = document.getElementById('pilotNameInput').value.trim() || '익명의 파일럿';
            leaderboard.push({ name: currentName, level: level, score: score });
            updateLeaderboardUI();

            document.getElementById('gameOverScreen').classList.remove('hidden');
        }

        // 웨이브 자동 증가
        if (score > wave * 1500) {
            wave++;
        }
    }

    function render() {
        ctx.save();
        if (screenShakeTimer > 0) {
            screenShakeTimer--;
            ctx.translate((Math.random() - 0.5) * screenShakeIntensity, (Math.random() - 0.5) * screenShakeIntensity);
        }

        ctx.fillStyle = '#05050f';
        ctx.fillRect(0, 0, canvas.width, canvas.height);

        // 네온 배경 사이버 그리드 효과
        ctx.strokeStyle = 'rgba(0, 243, 255, 0.05)';
        ctx.lineWidth = 1;
        const gridSize = 40;
        const offset = (Date.now() / 50) % gridSize;
        for (let y = offset; y < canvas.height; y += gridSize) {
            ctx.beginPath(); ctx.moveTo(0, y); ctx.lineTo(canvas.width, y); ctx.stroke();
        }

        // 파티클 렌더링
        particles.forEach(p => {
            ctx.fillStyle = p.color;
            ctx.globalAlpha = p.life;
            ctx.fillRect(p.x, p.y, p.size, p.size);
            ctx.globalAlpha = 1.0;
        });

        // 총알 렌더링
        ctx.fillStyle = '#00f3ff';
        bullets.forEach(b => {
            ctx.beginPath();
            ctx.arc(b.x, b.y, b.radius, 0, Math.PI * 2);
            ctx.shadowBlur = 10; ctx.shadowColor = '#00f3ff';
            ctx.fill();
            ctx.shadowBlur = 0;
        });

        // 적 렌더링
        enemies.forEach(en => {
            ctx.save();
            ctx.translate(en.x, en.y);
            ctx.fillStyle = en.type === 'drone' ? '#ff0055' : '#ffe600';
            ctx.shadowBlur = 12; ctx.shadowColor = ctx.fillStyle;
            ctx.beginPath();
            if (en.type === 'drone') {
                ctx.moveTo(0, 14); ctx.lineTo(-12, -10); ctx.lineTo(12, -10);
            } else {
                ctx.arc(0, 0, en.radius, 0, Math.PI * 2);
            }
            ctx.closePath();
            ctx.fill();
            ctx.restore();
        });

        // 플레이어 기체 렌더링
        if (gameState === 'PLAYING') {
            ctx.save();
            ctx.translate(player.x, player.y);
            ctx.fillStyle = '#00f3ff';
            ctx.shadowBlur = 15; ctx.shadowColor = '#00f3ff';
            ctx.beginPath();
            ctx.moveTo(0, -22);
            ctx.lineTo(-16, 16);
            ctx.lineTo(0, 8);
            ctx.lineTo(16, 16);
            ctx.closePath();
            ctx.fill();
            ctx.restore();
        }

        ctx.restore();
    }

    requestAnimationFrame(gameLoop);
</script>
</body>
</html>
