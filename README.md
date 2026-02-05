
html_content = '''<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Banana Commando: Grape Blaster</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }
        
        body {
            background: linear-gradient(135deg, #1a1a2e 0%, #16213e 50%, #0f3460 100%);
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            overflow: hidden;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
        }
        
        #gameContainer {
            position: relative;
            box-shadow: 0 0 50px rgba(255, 215, 0, 0.3);
            border-radius: 10px;
            overflow: hidden;
        }
        
        canvas {
            display: block;
            background: linear-gradient(to bottom, #87CEEB 0%, #E0F6FF 50%, #FFE4B5 100%);
            cursor: crosshair;
        }
        
        #ui {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            padding: 15px;
            display: flex;
            justify-content: space-between;
            pointer-events: none;
            text-shadow: 2px 2px 4px rgba(0,0,0,0.5);
        }
        
        .stat-box {
            background: rgba(0, 0, 0, 0.6);
            padding: 10px 20px;
            border-radius: 25px;
            color: #FFD700;
            font-weight: bold;
            font-size: 18px;
            border: 2px solid #FFD700;
        }
        
        #health-bar {
            position: absolute;
            bottom: 20px;
            left: 50%;
            transform: translateX(-50%);
            width: 300px;
            height: 30px;
            background: rgba(0, 0, 0, 0.6);
            border-radius: 15px;
            overflow: hidden;
            border: 2px solid #FFD700;
        }
        
        #health-fill {
            height: 100%;
            width: 100%;
            background: linear-gradient(90deg, #ff4444, #ff8844, #ffaa44);
            transition: width 0.3s;
        }
        
        #startScreen, #gameOverScreen, #victoryScreen {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(0, 0, 0, 0.85);
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            color: white;
            z-index: 100;
        }
        
        h1 {
            font-size: 48px;
            margin-bottom: 20px;
            text-align: center;
            background: linear-gradient(45deg, #FFD700, #FFA500, #FFD700);
            -webkit-background-clip: text;
            -webkit-text-fill-color: transparent;
            background-clip: text;
            text-shadow: none;
            filter: drop-shadow(0 0 10px rgba(255, 215, 0, 0.5));
        }
        
        .subtitle {
            font-size: 24px;
            margin-bottom: 30px;
            color: #FFA500;
        }
        
        button {
            padding: 15px 40px;
            font-size: 24px;
            background: linear-gradient(45deg, #FFD700, #FFA500);
            border: none;
            border-radius: 30px;
            color: #1a1a2e;
            font-weight: bold;
            cursor: pointer;
            transition: transform 0.2s, box-shadow 0.2s;
            margin: 10px;
        }
        
        button:hover {
            transform: scale(1.1);
            box-shadow: 0 0 30px rgba(255, 215, 0, 0.6);
        }
        
        .instructions {
            margin-top: 30px;
            text-align: center;
            color: #ccc;
            line-height: 1.6;
        }
        
        .hidden {
            display: none !important;
        }
        
        #weapon-indicator {
            position: absolute;
            bottom: 60px;
            left: 50%;
            transform: translateX(-50%);
            background: rgba(0, 0, 0, 0.6);
            padding: 8px 16px;
            border-radius: 20px;
            color: #FFD700;
            font-weight: bold;
        }
        
        .particle {
            position: absolute;
            pointer-events: none;
        }
    </style>
</head>
<body>
    <div id="gameContainer">
        <canvas id="gameCanvas" width="1000" height="700"></canvas>
        
        <div id="ui">
            <div class="stat-box">Score: <span id="score">0</span></div>
            <div class="stat-box">Wave: <span id="wave">1</span></div>
            <div class="stat-box">Aliens: <span id="aliensLeft">0</span></div>
        </div>
        
        <div id="health-bar">
            <div id="health-fill"></div>
        </div>
        
        <div id="weapon-indicator">🍌 Banana Gun (Grapes)</div>
        
        <div id="startScreen">
            <h1>🍌 BANANA COMMANDO 🍌</h1>
            <div class="subtitle">Grape Blaster Edition</div>
            <button onclick="startGame()">START MISSION</button>
            <div class="instructions">
                <p>🎯 Move mouse to aim | Click to shoot grapes</p>
                <p>🚀 WASD or Arrow keys to move</p>
                <p>👽 Defeat all aliens to save the city!</p>
                <p>💛 Collect yellow orbs for health</p>
            </div>
        </div>
        
        <div id="gameOverScreen" class="hidden">
            <h1>GAME OVER</h1>
            <div class="subtitle">The aliens have taken the city!</div>
            <div class="stat-box" style="margin: 20px;">Final Score: <span id="finalScore">0</span></div>
            <button onclick="restartGame()">TRY AGAIN</button>
        </div>
        
        <div id="victoryScreen" class="hidden">
            <h1>🎉 VICTORY! 🎉</h1>
            <div class="subtitle">The city is safe thanks to you!</div>
            <div class="stat-box" style="margin: 20px;">Final Score: <span id="victoryScore">0</span></div>
            <button onclick="restartGame()">PLAY AGAIN</button>
        </div>
    </div>

    <script>
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        
        // Game state
        let gameRunning = false;
        let score = 0;
        let wave = 1;
        let frame = 0;
        
        // Player (Banana)
        const player = {
            x: canvas.width / 2,
            y: canvas.height - 150,
            width: 50,
            height: 70,
            speed: 5,
            health: 100,
            maxHealth: 100,
            vx: 0,
            vy: 0,
            angle: 0,
            recoil: 0
        };
        
        // Input handling
        const keys = {};
        let mouseX = canvas.width / 2;
        let mouseY = canvas.height / 2;
        
        // Game objects
        let bullets = [];
        let aliens = [];
        let particles = [];
        let powerUps = [];
        let buildings = [];
        let clouds = [];
        
        // Initialize city background
        function initCity() {
            buildings = [];
            clouds = [];
            
            // Generate buildings
            for (let i = 0; i < 15; i++) {
                buildings.push({
                    x: i * 80 + Math.random() * 20,
                    y: canvas.height - 100 - Math.random() * 200,
                    width: 60 + Math.random() * 40,
                    height: 150 + Math.random() * 250,
                    color: `hsl(${200 + Math.random() * 40}, 20%, ${30 + Math.random() * 20}%)`,
                    windows: []
                });
                
                // Add windows to buildings
                const building = buildings[i];
                for (let w = 0; w < 5; w++) {
                    for (let h = 0; h < 8; h++) {
                        if (Math.random() > 0.3) {
                            building.windows.push({
                                x: building.x + 10 + w * 12,
                                y: building.y + 10 + h * 20,
                                lit: Math.random() > 0.5
                            });
                        }
                    }
                }
            }
            
            // Generate clouds
            for (let i = 0; i < 5; i++) {
                clouds.push({
                    x: Math.random() * canvas.width,
                    y: 50 + Math.random() * 150,
                    width: 100 + Math.random() * 100,
                    speed: 0.2 + Math.random() * 0.3
                });
            }
        }
        
        // Spawn aliens
        function spawnAliens(count) {
            for (let i = 0; i < count; i++) {
                const type = Math.random();
                let alien = {
                    x: Math.random() * (canvas.width - 60) + 30,
                    y: -50 - Math.random() * 200,
                    width: 40,
                    height: 40,
                    speed: 1 + wave * 0.3 + Math.random(),
                    health: 1,
                    maxHealth: 1,
                    type: 'basic',
                    angle: 0,
                    wobble: Math.random() * Math.PI * 2
                };
                
                if (type > 0.7 && wave > 2) {
                    alien.type = 'tank';
                    alien.width = 60;
                    alien.height = 60;
                    alien.health = 3;
                    alien.maxHealth = 3;
                    alien.speed *= 0.6;
                    alien.color = '#8B0000';
                } else if (type > 0.4 && wave > 1) {
                    alien.type = 'fast';
                    alien.width = 35;
                    alien.height = 35;
                    alien.speed *= 1.5;
                    alien.color = '#00CED1';
                } else {
                    alien.color = '#32CD32';
                }
                
                aliens.push(alien);
            }
            updateUI();
        }
        
        // Create particle effect
        function createParticles(x, y, color, count = 8) {
            for (let i = 0; i < count; i++) {
                particles.push({
                    x: x,
                    y: y,
                    vx: (Math.random() - 0.5) * 8,
                    vy: (Math.random() - 0.5) * 8,
                    life: 30,
                    color: color,
                    size: 3 + Math.random() * 5
                });
            }
        }
        
        // Draw banana player
        function drawBanana() {
            ctx.save();
            ctx.translate(player.x, player.y);
            
            // Recoil effect
            const recoilOffset = player.recoil * 0.3;
            
            // Banana body (curved)
            ctx.beginPath();
            ctx.moveTo(-20, -30);
            ctx.quadraticCurveTo(0, -35, 20, -30);
            ctx.quadraticCurveTo(25, 0, 15, 30);
            ctx.quadraticCurveTo(0, 35, -15, 30);
            ctx.quadraticCurveTo(-25, 0, -20, -30);
            ctx.closePath();
            
            const gradient = ctx.createLinearGradient(-20, -30, 20, 30);
            gradient.addColorStop(0, '#FFE135');
            gradient.addColorStop(0.5, '#FFD700');
            gradient.addColorStop(1, '#FFA500');
            ctx.fillStyle = gradient;
            ctx.fill();
            ctx.strokeStyle = '#DAA520';
            ctx.lineWidth = 2;
            ctx.stroke();
            
            // Banana stem
            ctx.fillStyle = '#8B4513';
            ctx.fillRect(-3, -38, 6, 10);
            
            // Face
            ctx.fillStyle = 'black';
            // Left eye
            ctx.beginPath();
            ctx.arc(-8, -10, 3, 0, Math.PI * 2);
            ctx.fill();
            // Right eye
            ctx.beginPath();
            ctx.arc(8, -10, 3, 0, Math.PI * 2);
            ctx.fill();
            
            // Determined mouth
            ctx.beginPath();
            ctx.moveTo(-10, 5);
            ctx.lineTo(0, 10);
            ctx.lineTo(10, 5);
            ctx.stroke();
            
            // Banana gun
            ctx.rotate(player.angle);
            ctx.translate(25 - recoilOffset, 0);
            
            // Gun body
            ctx.fillStyle = '#8B4513';
            ctx.fillRect(0, -8, 40, 16);
            
            // Gun barrel
            ctx.fillStyle = '#654321';
            ctx.fillRect(35, -6, 15, 12);
            
            // Banana decoration on gun
            ctx.fillStyle = '#FFE135';
            ctx.beginPath();
            ctx.arc(15, 0, 8, 0, Math.PI * 2);
            ctx.fill();
            
            ctx.restore();
            
            // Reduce recoil
            player.recoil = Math.max(0, player.recoil - 1);
        }
        
        // Draw grape bullet
        function drawGrape(bullet) {
            ctx.save();
            ctx.translate(bullet.x, bullet.y);
            
            // Grape cluster (3 circles)
            const colors = ['#663399', '#4B0082', '#8B008B'];
            const positions = [
                {x: 0, y: -3},
                {x: -3, y: 2},
                {x: 3, y: 2}
            ];
            
            positions.forEach((pos, i) => {
                ctx.beginPath();
                ctx.arc(pos.x, pos.y, 4, 0, Math.PI * 2);
                ctx.fillStyle = colors[i];
                ctx.fill();
                ctx.strokeStyle = '#4B0082';
                ctx.lineWidth = 1;
                ctx.stroke();
                
                // Shine
                ctx.beginPath();
                ctx.arc(pos.x - 1, pos.y - 1, 1.5, 0, Math.PI * 2);
                ctx.fillStyle = 'rgba(255,255,255,0.4)';
                ctx.fill();
            });
            
            // Trail effect
            ctx.globalAlpha = 0.3;
            ctx.beginPath();
            ctx.arc(-bullet.vx * 2, -bullet.vy * 2, 3, 0, Math.PI * 2);
            ctx.fillStyle = '#663399';
            ctx.fill();
            
            ctx.restore();
        }
        
        // Draw alien
        function drawAlien(alien) {
            ctx.save();
            ctx.translate(alien.x, alien.y);
            
            // Hover animation
            const hover = Math.sin(frame * 0.1 + alien.wobble) * 5;
            ctx.translate(0, hover);
            
            // Alien body (UFO shape)
            ctx.beginPath();
            ctx.ellipse(0, 0, alien.width/2, alien.height/3, 0, 0, Math.PI * 2);
            ctx.fillStyle = alien.color || '#32CD32';
            ctx.fill();
            ctx.strokeStyle = '#228B22';
            ctx.lineWidth = 2;
            ctx.stroke();
            
            // Alien head (dome)
            ctx.beginPath();
            ctx.arc(0, -alien.height/4, alien.width/3, Math.PI, 0);
            ctx.fillStyle = 'rgba(144, 238, 144, 0.6)';
            ctx.fill();
            ctx.strokeStyle = '#32CD32';
            ctx.stroke();
            
            // Eyes
            ctx.fillStyle = 'red';
            ctx.beginPath();
            ctx.arc(-8, -5, 4, 0, Math.PI * 2);
            ctx.arc(8, -5, 4, 0, Math.PI * 2);
            ctx.fill();
            
            // Evil pupils
            ctx.fillStyle = 'yellow';
            ctx.beginPath();
            ctx.arc(-8, -5, 1.5, 0, Math.PI * 2);
            ctx.arc(8, -5, 1.5, 0, Math.PI * 2);
            ctx.fill();
            
            // Antennae
            ctx.strokeStyle = alien.color;
            ctx.lineWidth = 2;
            ctx.beginPath();
            ctx.moveTo(-10, -alien.height/3);
            ctx.lineTo(-15, -alien.height/2);
            ctx.moveTo(10, -alien.height/3);
            ctx.lineTo(15, -alien.height/2);
            ctx.stroke();
            
            // Health bar for tank aliens
            if (alien.type === 'tank' && alien.health < alien.maxHealth) {
                ctx.fillStyle = 'red';
                ctx.fillRect(-20, -alien.height/2 - 15, 40, 5);
                ctx.fillStyle = '#00ff00';
                ctx.fillRect(-20, -alien.height/2 - 15, 40 * (alien.health / alien.maxHealth), 5);
            }
            
            ctx.restore();
        }
        
        // Draw city background
        function drawCity() {
            // Sky gradient is handled by CSS, draw clouds
            clouds.forEach(cloud => {
                ctx.fillStyle = 'rgba(255, 255, 255, 0.8)';
                ctx.beginPath();
                ctx.arc(cloud.x, cloud.y, 30, 0, Math.PI * 2);
                ctx.arc(cloud.x + 25, cloud.y - 10, 35, 0, Math.PI * 2);
                ctx.arc(cloud.x + 50, cloud.y, 30, 0, Math.PI * 2);
                ctx.fill();
                
                // Move clouds
                cloud.x += cloud.speed;
                if (cloud.x > canvas.width + 100) cloud.x = -100;
            });
            
            // Buildings
            buildings.forEach(building => {
                // Building body
                ctx.fillStyle = building.color;
                ctx.fillRect(building.x, building.y, building.width, canvas.height - building.y);
                
                // Building outline
                ctx.strokeStyle = 'rgba(0,0,0,0.3)';
                ctx.lineWidth = 1;
                ctx.strokeRect(building.x, building.y, building.width, canvas.height - building.y);
                
                // Windows
                building.windows.forEach(window => {
                    ctx.fillStyle = window.lit ? '#FFFF99' : '#2C3E50';
                    ctx.fillRect(window.x, window.y, 8, 12);
                });
            });
            
            // Street level
            ctx.fillStyle = '#34495E';
            ctx.fillRect(0, canvas.height - 20, canvas.width, 20);
            
            // Street lines
            ctx.strokeStyle = '#F1C40F';
            ctx.lineWidth = 2;
            ctx.setLineDash([20, 20]);
            ctx.beginPath();
            ctx.moveTo(0, canvas.height - 10);
            ctx.lineTo(canvas.width, canvas.height - 10);
            ctx.stroke();
            ctx.setLineDash([]);
        }
        
        // Draw power-up
        function drawPowerUp(powerUp) {
            ctx.save();
            ctx.translate(powerUp.x, powerUp.y);
            
            // Floating animation
            const float = Math.sin(frame * 0.1) * 5;
            ctx.translate(0, float);
            
            // Glow
            ctx.shadowBlur = 20;
            ctx.shadowColor = '#FFD700';
            
            // Heart/health shape
            ctx.fillStyle = '#FF6B6B';
            ctx.beginPath();
            const topCurveHeight = 10;
            ctx.moveTo(0, 5);
            ctx.bezierCurveTo(-15, -10, -15, -topCurveHeight, 0, -topCurveHeight);
            ctx.bezierCurveTo(15, -topCurveHeight, 15, -10, 0, 5);
            ctx.fill();
            
            // Plus sign
            ctx.fillStyle = 'white';
            ctx.fillRect(-2, -8, 4, 10);
            ctx.fillRect(-5, -5, 10, 4);
            
            ctx.restore();
        }
        
        // Update game logic
        function update() {
            if (!gameRunning) return;
            
            frame++;
            
            // Player movement
            if (keys['KeyW'] || keys['ArrowUp']) player.y = Math.max(50, player.y - player.speed);
            if (keys['KeyS'] || keys['KeyDown']) player.y = Math.min(canvas.height - 100, player.y + player.speed);
            if (keys['KeyA'] || keys['ArrowLeft']) player.x = Math.max(30, player.x - player.speed);
            if (keys['KeyD'] || keys['ArrowRight']) player.x = Math.min(canvas.width - 30, player.x + player.speed);
            
            // Calculate angle to mouse
            const dx = mouseX - player.x;
            const dy = mouseY - player.y;
            player.angle = Math.atan2(dy, dx);
            
            // Update bullets
            bullets = bullets.filter(bullet => {
                bullet.x += bullet.vx;
                bullet.y += bullet.vy;
                
                // Remove off-screen bullets
                if (bullet.x < 0 || bullet.x > canvas.width || bullet.y < 0 || bullet.y > canvas.height) {
                    return false;
                }
                
                // Check collision with aliens
                for (let i = aliens.length - 1; i >= 0; i--) {
                    const alien = aliens[i];
                    const dist = Math.hypot(bullet.x - alien.x, bullet.y - alien.y);
                    
                    if (dist < alien.width/2 + 10) {
                        alien.health--;
                        createParticles(bullet.x, bullet.y, '#663399', 5);
                        
                        if (alien.health <= 0) {
                            createParticles(alien.x, alien.y, alien.color, 15);
                            score += alien.type === 'tank' ? 30 : alien.type === 'fast' ? 20 : 10;
                            aliens.splice(i, 1);
                            
                            // Chance to drop power-up
                            if (Math.random() < 0.15) {
                                powerUps.push({
                                    x: alien.x,
                                    y: alien.y,
                                    type: 'health'
                                });
                            }
                        }
                        
                        updateUI();
                        return false;
                    }
                }
                
                return true;
            });
            
            // Update aliens
            aliens.forEach(alien => {
                // Move towards player
                const dx = player.x - alien.x;
                const dy = player.y - alien.y;
                const dist = Math.hypot(dx, dy);
                
                if (dist > 0) {
                    alien.x += (dx / dist) * alien.speed;
                    alien.y += (dy / dist) * alien.speed;
                }
                
                // Check collision with player
                if (dist < 40) {
                    player.health -= 10;
                    createParticles(player.x, player.y, '#FFD700', 10);
                    alien.y -= 50; // Bounce back
                    updateUI();
                    
                    if (player.health <= 0) {
                        gameOver();
                    }
                }
            });
            
            // Update particles
            particles = particles.filter(p => {
                p.x += p.vx;
                p.y += p.vy;
                p.vy += 0.3; // Gravity
                p.life--;
                p.size *= 0.95;
                return p.life > 0;
            });
            
            // Update power-ups
            powerUps = powerUps.filter(powerUp => {
                const dist = Math.hypot(player.x - powerUp.x, player.y - powerUp.y);
                if (dist < 40) {
                    player.health = Math.min(player.maxHealth, player.health + 25);
                    createParticles(powerUp.x, powerUp.y, '#FFD700', 8);
                    updateUI();
                    return false;
                }
                return true;
            });
            
            // Wave management
            if (aliens.length === 0 && gameRunning) {
                wave++;
                if (wave > 5) {
                    victory();
                } else {
                    spawnAliens(3 + wave * 2);
                }
            }
            
            // Update health bar
            document.getElementById('health-fill').style.width = player.health + '%';
        }
        
        // Draw everything
        function draw() {
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            
            drawCity();
            
            // Draw particles
            particles.forEach(p => {
                ctx.fillStyle = p.color;
                ctx.globalAlpha = p.life / 30;
                ctx.beginPath();
                ctx.arc(p.x, p.y, p.size, 0, Math.PI * 2);
                ctx.fill();
            });
            ctx.globalAlpha = 1;
            
            // Draw power-ups
            powerUps.forEach(drawPowerUp);
            
            // Draw bullets
            bullets.forEach(drawGrape);
            
            // Draw aliens
            aliens.forEach(drawAlien);
            
            // Draw player
            drawBanana();
        }
        
        // Game loop
        function gameLoop() {
            update();
            draw();
            if (gameRunning) {
                requestAnimationFrame(gameLoop);
            }
        }
        
        // UI updates
        function updateUI() {
            document.getElementById('score').textContent = score;
            document.getElementById('wave').textContent = wave;
            document.getElementById('aliensLeft').textContent = aliens.length;
        }
        
        // Start game
        function startGame() {
            document.getElementById('startScreen').classList.add('hidden');
            document.getElementById('gameOverScreen').classList.add('hidden');
            document.getElementById('victoryScreen').classList.add('hidden');
            
            // Reset game state
            player.x = canvas.width / 2;
            player.y = canvas.height - 150;
            player.health = 100;
            player.recoil = 0;
            score = 0;
            wave = 1;
            bullets = [];
            aliens = [];
            particles = [];
            powerUps = [];
            
            initCity();
            spawnAliens(5);
            updateUI();
            
            gameRunning = true;
            gameLoop();
        }
        
        // Game over
        function gameOver() {
            gameRunning = false;
            document.getElementById('finalScore').textContent = score;
            document.getElementById('gameOverScreen').classList.remove('hidden');
        }
        
        // Victory
        function victory() {
            gameRunning = false;
            document.getElementById('victoryScore').textContent = score;
            document.getElementById('victoryScreen').classList.remove('hidden');
        }
        
        // Restart
        function restartGame() {
            startGame();
        }
        
        // Event listeners
        canvas.addEventListener('mousemove', (e) => {
            const rect = canvas.getBoundingClientRect();
            mouseX = e.clientX - rect.left;
            mouseY = e.clientY - rect.top;
        });
        
        canvas.addEventListener('mousedown', (e) => {
            if (!gameRunning) return;
            
            // Shoot grape
            const speed = 12;
            const vx = Math.cos(player.angle) * speed;
            const vy = Math.sin(player.angle) * speed;
            
            bullets.push({
                x: player.x + Math.cos(player.angle) * 50,
                y: player.y + Math.sin(player.angle) * 50,
                vx: vx,
                vy: vy
            });
            
            player.recoil = 10;
            createParticles(player.x + Math.cos(player.angle) * 50, player.y + Math.sin(player.angle) * 50, '#8B4513', 3);
        });
        
        window.addEventListener('keydown', (e) => {
            keys[e.code] = true;
        });
        
        window.addEventListener('keyup', (e) => {
            keys[e.code] = false;
        });
        
        // Initialize
        initCity();
        draw();
    </script>
</body>
</html>'''

# Save to file
with open('/mnt/kimi/output/banana_commando.html', 'w', encoding='utf-8') as f:
    f.write(html_content)

print("Game created successfully!")
print("File saved to: /mnt/kimi/output/banana_commando.html")
print("\nGame Features:")
print("- You are a banana with a banana gun that shoots grape clusters")
print("- WASD or Arrow keys to move")
print("- Mouse to aim, click to shoot")
print("- 5 waves of aliens (basic, fast, and tank types)")
print("- City background with buildings and clouds")
print("- Particle effects and animations")
print("- Health pickups dropped by defeated aliens")
print("- Health bar, score, and wave tracking")
