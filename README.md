# YODA-<!DOCTYPE html>
<html lang="tr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>FRC BIOCORE™: Ultimate Driver Station</title>
    <style>
        body { 
            margin: 0; padding: 0;
            display: flex; flex-direction: column; justify-content: center; align-items: center; 
            background: #0a0f0a; font-family: 'Segoe UI', Arial, sans-serif; 
            height: 100vh; color: white; overflow: hidden;
        }
        
        .screen { display: none; text-align: center; z-index: 5; }
        .active { display: flex; flex-direction: column; align-items: center; justify-content: center; }

        /* İttifak Seçim Ekranı */
        #allianceScreen h1 { color: #8BC34A; letter-spacing: 2px; margin-bottom: 30px; }
        .btn-container { display: flex; gap: 20px; }
        .btn-alliance {
            padding: 20px 40px; font-size: 20px; font-weight: bold; border: none; border-radius: 12px;
            cursor: pointer; transition: 0.3s; color: white; text-shadow: 1px 1px 5px #000;
        }
        .red-alliance { background: #D32F2F; box-shadow: 0 0 20px rgba(211, 47, 47, 0.5); }
        .blue-alliance { background: #1976D2; box-shadow: 0 0 20px rgba(25, 118, 210, 0.5); }
        .btn-alliance:hover { transform: scale(1.08); }

        /* Oyun Alanı */
        #gameContainer { position: relative; }
        canvas { 
            border: 4px solid #8BC34A; border-radius: 16px; 
            touch-action: none; background: #112211;
            transition: filter 0.5s ease;
        }
        
        /* Dashboard & Göstergeler */
        #dashboard {
            display: flex; justify-content: space-between; width: 400px;
            margin-bottom: 10px; font-weight: bold; font-size: 16px;
        }
        .status-badge { padding: 4px 10px; border-radius: 6px; text-transform: uppercase; }
        .auto-mode { background: #FF9800; color: black; }
        .teleop-mode { background: #4CAF50; color: white; }
        .endgame-mode { background: #E91E63; color: white; }

        /* Arıza Efekti */
        #glitchOverlay {
            display: none; position: absolute; top: 0; left: 0; width: 100%; height: 100%;
            background: rgba(255, 0, 0, 0.15); color: #FF3D00; font-size: 28px; font-weight: bold;
            justify-content: center; align-items: center; border-radius: 16px; z-index: 8;
            pointer-events: none; letter-spacing: 2px; text-shadow: 0 0 10px #000;
        }

        /* Skor ve Kayıt Ekranı */
        input[type="text"] {
            padding: 10px; font-size: 16px; border-radius: 6px; border: 2px solid #8BC34A;
            background: #222; color: white; margin-bottom: 15px; text-align: center; width: 200px;
        }
        .btn-submit { background: #8BC34A; color: black; padding: 10px 20px; font-weight: bold; border: none; border-radius: 6px; cursor: pointer; }

        /* Liderlik Tablosu */
        table { width: 280px; margin: 15px 0; border-collapse: collapse; }
        th, td { padding: 8px; border-bottom: 1px solid #444; text-align: center; }
        th { color: #8BC34A; }

        .water-game-active { filter: hue-rotate(140deg) saturate(1.5); }
    </style>
</head>
<body>

    <div id="allianceScreen" class="screen active">
        <h1>FRC BIOCORE™ SIMULATOR</h1>
        <p style="color: #aaa; margin-bottom: 20px;">Alliance takımınızı seçin ve maça başlayın!</p>
        <div class="btn-container">
            <button id="btnRedAlliance" class="btn-alliance red-alliance">RED ALLIANCE</button>
            <button id="btnBlueAlliance" class="btn-alliance blue-alliance">BLUE ALLIANCE</button>
        </div>
    </div>

    <div id="gameScreen" class="screen">
        <div id="dashboard">
            <div>MODE: <span id="modeBadge" class="status-badge auto-mode">AUTO</span></div>
            <div>TIMER: <span id="timerText" style="color: #FF9800;">70s</span></div>
            <div>SCORE: <span id="scoreText" style="color: #8BC34A;">0</span></div>
        </div>
        <div id="gameContainer">
            <div id="glitchOverlay">⚠️ WATCHDOG TIMEOUT ⚠️</div>
            <canvas id="gameCanvas" width="400" height="500"></canvas>
        </div>
        <p id="controlsHint" style="font-size: 12px; color: #777; margin-top: 8px;">Otonomda robot kendi hareket eder. Teleopta Fare veya YÖN / A-D tuşlarını kullanın.</p>
    </div>

    <div id="endScreen" class="screen">
        <h2 id="endStatusTitle" style="color: #F44336; margin: 0 0 10px 0;">MATCH FMS DISABLE</h2>
        <p>Toplam Skorunuz: <strong id="finalScoreText" style="color: #8BC34A; font-size: 24px;">0</strong></p>
        
        <div id="rankInputArea">
            <p style="font-size: 14px; color: #ccc;">Sürücü listesine girmek için bilgilerinizi yazın:</p>
            <input type="text" id="playerName" placeholder="Adınız" maxlength="10"><br>
            <input type="text" id="playerTeam" placeholder="Takım Numaranız (Örn: 9999)" maxlength="5"><br>
            <button id="btnSaveScore" class="btn-submit">SKORU KAYDET</button>
        </div>

        <div id="leaderboardArea">
            <h3>🏆 EN İYİ SÜRÜCÜLER (TOP DRIVERS)</h3>
            <table id="leaderboardTable">
                <thead>
                    <tr><th>Sıra</th><th>Sürücü</th><th>Takım</th><th>Skor</th></tr>
                </thead>
                <tbody id="leaderboardBody"></tbody>
            </table>
            <button id="btnMenu" class="btn-submit" style="background: #555; color: white;">MENÜYE DÖN</button>
            <button id="btnReplay" class="btn-submit" style="margin-left: 10px;">MATCH REPLAY</button>
        </div>
    </div>

    <script>
        const canvas = document.getElementById("gameCanvas");
        const ctx = canvas.getContext("2d");
        
        let alliance = 'blue';
        let score = 0;
        let matchTimer = 70; 
        let isGameOver = false;
        let gameLoopId;
        let matchIntervalId;
        
        let currentPhase = 'AUTO'; 
        let watchdogActive = false;
        let waterGameActive = false;
        let waterGameTimer = 0;
        let isClimbed = false;
        let climbProgress = 0; 

        // Bebek Yoda Görseli Tanımlaması
        const yodaImg = new Image();
        let yoda = { x: 160, y: 410, width: 80, height: 80, speed: 8 }; // Tuş hassasiyeti için speed optimize edildi
        let items = [];
        let itemSpawnInterval;
        let mouseX = 160;

        // Klavye kontrol hafızası
        let keys = {};

        function executeClimbAction() {
            if (currentPhase === 'ENDGAME' && !watchdogActive && !isGameOver) {
                if (yoda.x > 80 && yoda.x < 240) {
                    climbProgress += 10; 
                    if (climbProgress >= 100) {
                        climbProgress = 100;
                        if(!isClimbed) {
                            score += 100; 
                            isClimbed = true;
                        }
                    }
                }
            }
        }

        // Fare hareket takibi
        canvas.addEventListener("mousemove", e => {
            if (currentPhase !== 'AUTO' && !watchdogActive && climbProgress === 0) {
                const rect = canvas.getBoundingClientRect();
                mouseX = e.clientX - rect.left - yoda.width / 2;
            }
        });

        // Mobil/Tablet parmak hareketi takibi
        canvas.addEventListener("touchmove", e => {
            if (currentPhase !== 'AUTO' && !watchdogActive && climbProgress === 0) {
                const rect = canvas.getBoundingClientRect();
                mouseX = e.touches[0].clientX - rect.left - yoda.width / 2;
            }
            e.preventDefault();
        }, { passive: false });

        // Tıklama veya ekrana dokunma ile tırmanma desteği
        canvas.addEventListener("mousedown", executeClimbAction);
        canvas.addEventListener("touchstart", e => {
            executeClimbAction();
            e.preventDefault();
        }, { passive: false });

        // Gelişmiş Klavye Dinleyicileri (Sürüş + Tırmanma tek çatı altında)
        window.addEventListener("keydown", e => {
            keys[e.code] = true;
            if (e.code === "Space") {
                executeClimbAction();
            }
        });

        window.addEventListener("keyup", e => {
            keys[e.code] = false;
        });

        function switchScreen(screenId) {
            document.querySelectorAll('.screen').forEach(s => s.classList.remove('active'));
            document.getElementById(screenId).classList.add('active');
        }

        function startGame(selectedAlliance) {
            alliance = selectedAlliance;
            score = 0;
            matchTimer = 70;
            currentPhase = 'AUTO';
            isGameOver = false;
            isClimbed = false;
            climbProgress = 0;
            watchdogActive = false;
            waterGameActive = false;
            items = [];
            keys = {};
            yoda.y = 410;
            yoda.x = 160;
            mouseX = 160;
            
            document.getElementById("modeBadge").className = "status-badge auto-mode";
            document.getElementById("modeBadge").innerText = "AUTO";
            document.getElementById("timerText").style.color = "#FF9800";
            document.getElementById("controlsHint").innerText = "🤖 OTONOM DÖNEMİ: Robot kendi kendine tohum topluyor!";

            switchScreen('gameScreen');
            
            if(gameLoopId) cancelAnimationFrame(gameLoopId);
            if(matchIntervalId) clearInterval(matchIntervalId);
            if(itemSpawnInterval) clearInterval(itemSpawnInterval);

            itemSpawnInterval = setInterval(spawnItem, 800);
            matchIntervalId = setInterval(updateMatchTimer, 1000);
            
            loop();
        }

        function updateMatchTimer() {
            matchTimer--;
            document.getElementById("timerText").innerText = matchTimer + "s";

            if (matchTimer === 60) {
                currentPhase = 'TELEOP';
                document.getElementById("modeBadge").className = "status-badge teleop-mode";
                document.getElementById("modeBadge").innerText = "TELEOP";
                document.getElementById("timerText").style.color = "#4CAF50";
                document.getElementById("controlsHint").innerText = "🕹️ TELEOP DÖNEMİ: Kontrol sizde! Plastiklerden kaçın.";
            } else if (matchTimer === 15) {
                currentPhase = 'ENDGAME';
                document.getElementById("modeBadge").className = "status-badge endgame-mode";
                document.getElementById("modeBadge").innerText = "ENDGAME";
                document.getElementById("timerText").style.color = "#E91E63";
                document.getElementById("controlsHint").innerText = "🧗 ENDGAME: Bar YEŞİL olduğunda tırmanmak için EKRANA TIKLAYIN veya SPACE'e basın!";
            }

            if (currentPhase === 'TELEOP' && Math.random() < 0.08 && !watchdogActive) {
                triggerWatchdog();
            }

            if (matchTimer <= 0) {
                endGame();
            }
        }

        function triggerWatchdog() {
            watchdogActive = true;
            document.getElementById("glitchOverlay").style.display = "flex";
            setTimeout(() => {
                watchdogActive = false;
                document.getElementById("glitchOverlay").style.display = "none";
            }, 1500);
        }

        function spawnItem() {
            if (isGameOver) return;
            const types = [
                { text: "🌱", score: 15, type: 'seed' },
                { text: "🥤", score: -25, type: 'plastic' },
                { text: "🌊", score: 10, type: 'water' } 
            ];
            
            let chosen = types[0];
            let r = Math.random();
            if (r > 0.5 && r < 0.9) chosen = types[1];
            else if (r >= 0.9) chosen = types[2];

            items.push({
                x: Math.random() * (canvas.width - 30),
                y: -20,
                size: 30,
                speed: 4 + Math.random() * 3,
                ...chosen
            });
        }

        function loop() {
            if (isGameOver) return;
            update();
            draw();
            gameLoopId = requestAnimationFrame(loop);
        }

        function update() {
            document.getElementById("scoreText").innerText = score;

            if (waterGameActive) {
                waterGameTimer--;
                if (waterGameTimer <= 0) {
                    waterGameActive = false;
                    canvas.classList.remove("water-game-active");
                }
            }

            // --- HİBRİT SÜRÜŞ SİSTEMİ YAPISI ---
            if (currentPhase === 'AUTO') {
                let targetSeed = items.find(i => i.type === 'seed');
                if (targetSeed) {
                    let diff = targetSeed.x - (yoda.x + yoda.width / 2);
                    yoda.x += Math.sign(diff) * 4.5;
                }
            } else if (currentPhase === 'ENDGAME' && climbProgress > 0) {
                yoda.y = 410 - (climbProgress * 2.5);
            } else if (!watchdogActive) {
                // Eğer klavyedeki Sol ok veya A tuşuna basılıyorsa
                if (keys["ArrowLeft"] || keys["KeyA"]) {
                    yoda.x -= yoda.speed;
                    mouseX = yoda.x; // Fareyi senkronize et ki çakışmasın
                } 
                // Eğer Sağ ok veya D tuşuna basılıyorsa
                else if (keys["ArrowRight"] || keys["KeyD"]) {
                    yoda.x += yoda.speed;
                    mouseX = yoda.x; // Fareyi senkronize et ki çakışmasın
                } 
                // Hiçbir tuşa basılmıyorsa akıcı fare kontrolüne dön
                else {
                    let diff = mouseX - yoda.x;
                    yoda.x += diff * 0.3;
                }
            }

            if (yoda.x < 0) yoda.x = 0;
            if (yoda.x > canvas.width - yoda.width) yoda.x = canvas.width - yoda.width;

            items.forEach((item, index) => {
                item.y += item.speed;

                if (item.x < yoda.x + yoda.width * 0.8 && item.x + item.size > yoda.x + yoda.width * 0.2 &&
                    item.y < yoda.y + yoda.height * 0.85 && item.y + item.size > yoda.y + yoda.height * 0.15) {
                    
                    if (item.type === 'water') {
                        waterGameActive = true;
                        waterGameTimer = 200;
                        canvas.classList.add("water-game-active");
                        items = []; 
                        score += 30;
                    } else {
                        if (item.type === 'plastic' && waterGameActive) {
                            score += 10;
                        } else {
                            score += item.score;
                        }
                    }
                    items.splice(index, 1);
                }

                if (item.y > canvas.height) {
                    items.splice(index, 1);
                }
            });

            if (score < -80) endGame(); 
        }

        function draw() {
            ctx.clearRect(0, 0, canvas.width, canvas.height);

            let greenVal = Math.max(15, Math.min(45, 25 + score / 15));
            let grayVal = Math.max(5, Math.min(30, 15 - score / 20));
            ctx.fillStyle = `rgb(${grayVal}, ${greenVal}, ${grayVal})`;
            ctx.fillRect(0, 0, canvas.width, canvas.height);

            let treeCount = Math.min(12, Math.floor(score / 40));
            ctx.font = "20px Arial";
            for (let i = 0; i < treeCount; i++) {
                ctx.fillText("🌳", 15 + (i * 32), 490);
            }

            ctx.strokeStyle = "rgba(255,255,255,0.08)";
            ctx.lineWidth = 2;
            ctx.beginPath(); ctx.moveTo(0, 120); ctx.lineTo(canvas.width, 120); ctx.stroke();

            if (currentPhase === 'ENDGAME') {
                let robotInZone = (yoda.x > 80 && yoda.x < 240);
                
                ctx.strokeStyle = robotInZone ? "#00E676" : "#E91E63";
                ctx.lineWidth = 6;
                ctx.beginPath(); ctx.moveTo(120, 140); ctx.lineTo(280, 140); ctx.stroke();
                
                ctx.fillStyle = robotInZone ? "#00E676" : "rgba(233, 30, 99, 0.8)";
                ctx.font = "bold 12px sans-serif";
                if(robotInZone && !isClimbed) {
                    ctx.fillText("READY TO CLIMB! CLICK NOW!", 115, 125);
                } else if (isClimbed) {
                    ctx.fillText("ROBOT CLIMBED! +100 RP", 125, 125);
                } else {
                    ctx.fillText("ALIGN TO CENTER ZONE", 130, 125);
                }
                
                ctx.strokeStyle = "rgba(255,255,255,0.3)";
                ctx.lineWidth = 2;
                ctx.beginPath(); ctx.moveTo(120, 0); ctx.lineTo(120, 140); ctx.moveTo(280, 0); ctx.lineTo(280, 140); ctx.stroke();
            }

            // Bebek Yoda Çizim Alanı
            if (yodaImg.complete && yodaImg.width > 0) {
                ctx.drawImage(yodaImg, yoda.x, yoda.y, yoda.width, yoda.height);
            } else {
                ctx.font = "50px Arial";
                // Tatlı bir bebek yoda emojisi kombinasyonu (👶 + 💚)
                ctx.fillText("👶💚", yoda.x + 5, yoda.y + 55);
            }

            ctx.fillStyle = alliance === 'red' ? '#D32F2F' : '#1976D2';
            ctx.fillRect(yoda.x + 5, yoda.y + yoda.height - 8, yoda.width - 10, 8);
            ctx.fillStyle = "white";
            ctx.font = "bold 9px sans-serif";
            ctx.fillText("TEAM", yoda.x + yoda.width/2 - 14, yoda.y + yoda.height - 1);

            items.forEach(item => {
                ctx.font = `${item.size}px Arial`;
                ctx.fillText(item.text, item.x, item.y + item.size * 0.8);
            });
        }

        function endGame() {
            isGameOver = true;
            clearInterval(matchIntervalId);
            clearInterval(itemSpawnInterval);
            document.getElementById("finalScoreText").innerText = score;
            
            if(score <= -80) {
                document.getElementById("endStatusTitle").innerText = "ALLIANCE DISABLED!";
                document.getElementById("endStatusTitle").style.color = "#D32F2F";
            } else {
                document.getElementById("endStatusTitle").innerText = "MATCH COMPLETED";
                document.getElementById("endStatusTitle").style.color = "#4CAF50";
            }

            document.getElementById("rankInputArea").style.display = "block";
            switchScreen('endScreen');
            showLeaderboard();
        }

        function saveScore() {
            const name = document.getElementById("playerName").value.trim() || "Anonymous";
            const team = document.getElementById("playerTeam").value.trim() || "0000";
            
            let data = JSON.parse(localStorage.getItem("frc_biocore_board")) || [];
            data.push({ name: name, team: team, score: score });
            data.sort((a, b) => b.score - a.score);
            data = data.slice(0, 5);
            
            localStorage.setItem("frc_biocore_board", JSON.stringify(data));
            document.getElementById("rankInputArea").style.display = "none";
            showLeaderboard();
        }

        function showLeaderboard() {
            const data = JSON.parse(localStorage.getItem("frc_biocore_board")) || [];
            const body = document.getElementById("leaderboardBody");
            body.innerHTML = "";
            
            data.forEach((player, i) => {
                const row = `<tr><td>${i+1}</td><td>${player.name}</td><td>#${player.team}</td><td style="color:#8BC34A; font-weight:bold;">${player.score}</td></tr>`;
                body.innerHTML += row;
            });
        }

        function resetGame() {
            startGame(alliance);
        }

        function restartToMenu() {
            switchScreen('allianceScreen');
        }

        // Event Listener'lar
        document.getElementById("btnRedAlliance").addEventListener("click", () => startGame('red'));
        document.getElementById("btnBlueAlliance").addEventListener("click", () => startGame('blue'));
        document.getElementById("btnSaveScore").addEventListener("click", saveScore);
        document.getElementById("btnMenu").addEventListener("click", restartToMenu);
        document.getElementById("btnReplay").addEventListener("click", resetGame);

        // Bebek Yoda (Grogu) İkon Kaynağı
        yodaImg.src = "https://i.pinimg.com/736x/61/37/37/613737961ec47dbb6ce882dce56b45b7.jpg"; 
    </script>
</body>
</html>
