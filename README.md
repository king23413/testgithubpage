<!DOCTYPE html>
<html>
<head>
    <title>Mario-Style Kart Racing</title>
    <style>
        body {
            margin: 0;
            overflow: hidden;
            background: #6ab7ff;
            font-family: 'Arial', sans-serif;
            cursor: pointer;
        }
        #gameCanvas {
            display: block;
            background: #6ab7ff;
        }
        #hud {
            position: absolute;
            top: 10px;
            left: 10px;
            color: white;
            font-size: 20px;
            text-shadow: 2px 2px 4px #000;
            font-family: 'Arial Black', sans-serif;
        }
        #gameOver {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            color: white;
            font-size: 48px;
            text-align: center;
            text-shadow: 3px 3px 6px #000;
            display: none;
            font-family: 'Arial Black', sans-serif;
        }
        #startScreen {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(0, 0, 0, 0.7);
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            color: white;
            font-family: 'Arial Black', sans-serif;
        }
        #startButton {
            margin-top: 20px;
            padding: 15px 30px;
            font-size: 24px;
            background: #ff5555;
            border: none;
            border-radius: 10px;
            color: white;
            cursor: pointer;
            font-family: 'Arial Black', sans-serif;
        }
    </style>
</head>
<body>
    <div id="hud">Lap: 1/3 | Position: 1st | Speed: 0</div>
    <div id="gameOver">Race Finished!<br><span style="font-size: 24px;">Press R to restart</span></div>
    <div id="startScreen">
        <h1 style="font-size: 48px;">MARIO-STYLE KART RACING</h1>
        <p style="font-size: 24px;">Use arrow keys to drive, space to drift!</p>
        <button id="startButton">START RACE!</button>
    </div>
    <canvas id="gameCanvas"></canvas>

    <script>
        // Game setup
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        const hud = document.getElementById('hud');
        const gameOver = document.getElementById('gameOver');
        const startScreen = document.getElementById('startScreen');
        const startButton = document.getElementById('startButton');

        // Canvas size
        canvas.width = 800;
        canvas.height = 600;

        // Game variables
        let gameRunning = false;
        let raceStarted = false;
        let lapCount = 1;
        const totalLaps = 3;
        let playerPosition = 1;

        // Track (Mario-style circuit)
        const track = {
            width: 3000,
            height: 3000,
            checkpoints: [
                { x: 500, y: 500 },
                { x: 2500, y: 500 },
                { x: 2500, y: 2500 },
                { x: 500, y: 2500 }
            ],
            obstacles: [],
            powerups: []
        };

        // Generate Mario-style obstacles (pipes, coins, etc.)
        for (let i = 0; i < 30; i++) {
            track.obstacles.push({
                x: Math.random() * track.width,
                y: Math.random() * track.height,
                width: 60 + Math.random() * 60,
                height: 60 + Math.random() * 60,
                type: Math.random() > 0.5 ? "pipe" : "block",
                color: Math.random() > 0.5 ? "#ff5555" : "#55ff55"
            });
        }

        // Generate power-ups (Mario-style)
        for (let i = 0; i < 10; i++) {
            track.powerups.push({
                x: Math.random() * track.width,
                y: Math.random() * track.height,
                type: ["mushroom", "star", "banana"][Math.floor(Math.random() * 3)],
                active: true
            });
        }

        // Player car (Mario Kart-style)
        const player = {
            x: 1000,
            y: 1000,
            angle: 0,
            speed: 0,
            maxSpeed: 10,
            acceleration: 0.3,
            deceleration: 0.2,
            turnSpeed: 0.06,
            driftPower: 0.15,
            width: 50,
            height: 80,
            color: '#ff3333',
            isDrifting: false,
            lap: 1,
            nextCheckpoint: 0,
            powerup: null,
            invincible: false
        };

        // Bot cars (Mario characters)
        const bots = [
            { x: 950, y: 950, angle: 0, speed: 6, color: '#33cc33', name: "Luigi" },
            { x: 1050, y: 950, angle: 0, speed: 5.5, color: '#ffff33', name: "Toad" },
            { x: 950, y: 1050, angle: 0, speed: 5, color: '#ff9933', name: "Peach" }
        ];

        // Controls
        const keys = {
            ArrowUp: false,
            ArrowDown: false,
            ArrowLeft: false,
            ArrowRight: false,
            " ": false // Space for drifting
        };

        // Event listeners
        window.addEventListener('keydown', (e) => {
            if (keys.hasOwnProperty(e.key)) keys[e.key] = true;
            if (e.key === 'r' && !gameRunning) resetGame();
        });
        window.addEventListener('keyup', (e) => {
            if (keys.hasOwnProperty(e.key)) keys[e.key] = false;
        });

        startButton.addEventListener('click', () => {
            startScreen.style.display = 'none';
            gameRunning = true;
            raceStarted = true;
        });

        // Reset game
        function resetGame() {
            player.x = 1000;
            player.y = 1000;
            player.speed = 0;
            player.lap = 1;
            player.nextCheckpoint = 0;
            player.powerup = null;
            player.invincible = false;
            lapCount = 1;
            gameRunning = true;
            raceStarted = true;
            gameOver.style.display = 'none';
            
            // Reset bots
            bots[0].x = 950; bots[0].y = 950;
            bots[1].x = 1050; bots[1].y = 950;
            bots[2].x = 950; bots[2].y = 1050;
            
            // Reset power-ups
            track.powerups.forEach(p => p.active = true);
        }

        // Update game state
        function update() {
            if (!gameRunning) return;

            // Player controls
            if (keys.ArrowUp) player.speed = Math.min(player.speed + player.acceleration, player.maxSpeed);
            else if (keys.ArrowDown) player.speed = Math.max(player.speed - player.deceleration, -player.maxSpeed / 2);
            else player.speed *= 0.95; // Friction

            // Drifting (hold space)
            player.isDrifting = keys[" "] && Math.abs(player.speed) > 3;
            const turnSpeed = player.isDrifting ? player.turnSpeed * 1.8 : player.turnSpeed;

            if (keys.ArrowLeft) player.angle -= turnSpeed * (player.speed / player.maxSpeed);
            if (keys.ArrowRight) player.angle += turnSpeed * (player.speed / player.maxSpeed);

            // Move player
            player.x += Math.sin(player.angle) * player.speed;
            player.y -= Math.cos(player.angle) * player.speed;

            // Check boundaries
            player.x = Math.max(0, Math.min(track.width, player.x));
            player.y = Math.max(0, Math.min(track.height, player.y));

            // Update bots (simple AI)
            bots.forEach(bot => {
                const targetCheckpoint = track.checkpoints[0]; // Bots follow checkpoints
                const angleToCheckpoint = Math.atan2(targetCheckpoint.x - bot.x, targetCheckpoint.y - bot.y);
                bot.angle = angleToCheckpoint * -1; // Adjust angle for top-down view
                bot.x += Math.sin(bot.angle) * bot.speed;
                bot.y -= Math.cos(bot.angle) * bot.speed;
            });

            // Check checkpoints
            const checkpoint = track.checkpoints[player.nextCheckpoint];
            const distToCheckpoint = Math.sqrt((player.x - checkpoint.x) ** 2 + (player.y - checkpoint.y) ** 2);
            if (distToCheckpoint < 100) {
                player.nextCheckpoint = (player.nextCheckpoint + 1) % track.checkpoints.length;
                if (player.nextCheckpoint === 0) {
                    player.lap++;
                    lapCount = player.lap;
                    if (player.lap > totalLaps) {
                        gameRunning = false;
                        gameOver.style.display = 'block';
                    }
                }
            }

            // Check power-ups
            track.powerups.forEach(powerup => {
                if (powerup.active) {
                    const dist = Math.sqrt((player.x - powerup.x) ** 2 + (player.y - powerup.y) ** 2);
                    if (dist < 50) {
                        powerup.active = false;
                        player.powerup = powerup.type;
                        
                        // Apply power-up effect
                        if (player.powerup === "mushroom") {
                            player.maxSpeed += 2;
                            setTimeout(() => player.maxSpeed -= 2, 5000);
                        } else if (player.powerup === "star") {
                            player.invincible = true;
                            setTimeout(() => player.invincible = false, 5000);
                        } else if (player.powerup === "banana") {
                            // Place banana peel (not implemented here)
                        }
                    }
                }
            });

            // Update HUD
            const positionText = 
                playerPosition === 1 ? '1st' :
                playerPosition === 2 ? '2nd' :
                playerPosition === 3 ? '3rd' :
                `${playerPosition}th`;
            
            let powerupText = "";
            if (player.powerup) {
                powerupText = ` | Power-up: ${player.powerup}`;
                if (player.invincible) powerupText += " (INVINCIBLE!)";
            }
            
            hud.textContent = `Lap: ${lapCount}/${totalLaps} | Position: ${positionText} | Speed: ${Math.abs(player.speed).toFixed(1)}${powerupText}`;
        }

        // Draw everything
        function draw() {
            // Clear canvas
            ctx.clearRect(0, 0, canvas.width, canvas.height);

            // Draw sky background
            const skyGradient = ctx.createLinearGradient(0, 0, 0, canvas.height);
            skyGradient.addColorStop(0, '#6ab7ff');
            skyGradient.addColorStop(1, '#1a75ff');
            ctx.fillStyle = skyGradient;
            ctx.fillRect(0, 0, canvas.width, canvas.height);

            // Draw track (grass & road)
            ctx.fillStyle = '#2a8a3a'; // Grass
            ctx.fillRect(0, 0, canvas.width, canvas.height);
            
            ctx.fillStyle = '#555555'; // Road
            ctx.fillRect(
                canvas.width / 2 - player.x / 5,
                canvas.height / 2 - player.y / 5,
                track.width / 5,
                track.height / 5
            );

            // Draw Mario-style obstacles
            track.obstacles.forEach(obstacle => {
                ctx.fillStyle = obstacle.color;
                if (obstacle.type === "pipe") {
                    // Draw green pipe
                    ctx.fillRect(
                        canvas.width / 2 - player.x / 5 + obstacle.x / 5,
                        canvas.height / 2 - player.y / 5 + obstacle.y / 5,
                        obstacle.width / 5,
                        obstacle.height / 5
                    );
                    ctx.fillStyle = '#333333';
                    ctx.fillRect(
                        canvas.width / 2 - player.x / 5 + obstacle.x / 5 + 10,
                        canvas.height / 2 - player.y / 5 + obstacle.y / 5 + 10,
                        obstacle.width / 5 - 20,
                        obstacle.height / 5 - 20
                    );
                } else {
                    // Draw brick block
                    ctx.fillRect(
                        canvas.width / 2 - player.x / 5 + obstacle.x / 5,
                        canvas.height / 2 - player.y / 5 + obstacle.y / 5,
                        obstacle.width / 5,
                        obstacle.height / 5
                    );
                    ctx.strokeStyle = '#000000';
                    ctx.lineWidth = 2;
                    ctx.strokeRect(
                        canvas.width / 2 - player.x / 5 + obstacle.x / 5,
                        canvas.height / 2 - player.y / 5 + obstacle.y / 5,
                        obstacle.width / 5,
                        obstacle.height / 5
                    );
                }
            });

            // Draw checkpoints (Mario checkered flags)
            track.checkpoints.forEach((checkpoint, i) => {
                ctx.fillStyle = i === player.nextCheckpoint ? '#ffffff' : '#aaaaaa';
                for (let j = 0; j < 4; j++) {
                    ctx.fillRect(
                        canvas.width / 2 - player.x / 5 + checkpoint.x / 5 - 20 + (j % 2) * 20,
                        canvas.height / 2 - player.y / 5 + checkpoint.y / 5 - 20 + Math.floor(j / 2) * 20,
                        20,
                        20
                    );
                }
            });

            // Draw power-ups
            track.powerups.forEach(powerup => {
                if (powerup.active) {
                    if (powerup.type === "mushroom") {
                        // Red mushroom
                        ctx.fillStyle = '#ff5555';
                        ctx.beginPath();
                        ctx.arc(
                            canvas.width / 2 - player.x / 5 + powerup.x / 5,
                            canvas.height / 2 - player.y / 5 + powerup.y / 5,
                            15,
                            0,
                            Math.PI * 2
                        );
                        ctx.fill();
                        ctx.fillStyle = '#ffffff';
                        ctx.beginPath();
                        ctx.arc(
                            canvas.width / 2 - player.x / 5 + powerup.x / 5 - 5,
                            canvas.height / 2 - player.y / 5 + powerup.y / 5 - 5,
                            5,
                            0,
                            Math.PI * 2
                        );
                        ctx.fill();
                    } else if (powerup.type === "star") {
                        // Star
                        ctx.fillStyle = '#ffff00';
                        ctx.beginPath();
                        for (let i = 0; i < 5; i++) {
                            const angle = (i * 2 * Math.PI / 5) - Math.PI / 2;
                            const x = canvas.width / 2 - player.x / 5 + powerup.x / 5 + Math.cos(angle) * 15;
                            const y = canvas.height / 2 - player.y / 5 + powerup.y / 5 + Math.sin(angle) * 15;
                            if (i === 0) ctx.moveTo(x, y);
                            else ctx.lineTo(x, y);
                        }
                        ctx.closePath();
                        ctx.fill();
                    } else if (powerup.type === "banana") {
                        // Banana
                        ctx.fillStyle = '#ffff00';
                        ctx.beginPath();
                        ctx.ellipse(
                            canvas.width / 2 - player.x / 5 + powerup.x / 5,
                            canvas.height / 2 - player.y / 5 + powerup.y / 5,
                            15,
                            8,
                            Math.PI / 4,
                            0,
                            Math.PI * 2
                        );
                        ctx.fill();
                    }
                }
            });

            // Draw bots (Mario characters)
            bots.forEach(bot => {
                ctx.save();
                ctx.translate(
                    canvas.width / 2 - player.x / 5 + bot.x / 5,
                    canvas.height / 2 - player.y / 5 + bot.y / 5
                );
                ctx.rotate(bot.angle);
                
                // Kart body
                ctx.fillStyle = bot.color;
                ctx.fillRect(-25, -40, 50, 80);
                
                // Kart details
                ctx.fillStyle = '#ffffff';
                ctx.fillRect(-20, -30, 40, 20);
                ctx.fillRect(-20, 10, 40, 20);
                
                // Character face
                ctx.fillStyle = '#ffccaa';
                ctx.beginPath();
                ctx.arc(0, -10, 15, 0, Math.PI * 2);
                ctx.fill();
                
                ctx.restore();
            });

            // Draw player car (Mario-style kart)
            ctx.save();
            ctx.translate(canvas.width / 2, canvas.height / 2);
            ctx.rotate(player.angle);
            
            // Kart body
            ctx.fillStyle = player.invincible ? '#ffff00' : player.color;
            ctx.fillRect(-30, -50, 60, 100);
            
            // Kart details
            ctx.fillStyle = '#ffffff';
            ctx.fillRect(-25, -40, 50, 20);
            ctx.fillRect(-25, 20, 50, 20);
            
            // Mario face
            ctx.fillStyle = '#ffccaa';
            ctx.beginPath();
            ctx.arc(0, -20, 20, 0, Math.PI * 2);
            ctx.fill();
            
            // Mario hat
            ctx.fillStyle = '#ff0000';
            ctx.fillRect(-20, -35, 40, 15);
            ctx.fillRect(-15, -40, 30, 5);
            
            // Drift effect (smoke)
            if (player.isDrifting) {
                ctx.fillStyle = 'rgba(255, 255, 255, 0.7)';
                for (let i = 0; i < 5; i++) {
                    ctx.beginPath();
                    ctx.arc(
                        -30 + Math.random() * 20,
                        50 + Math.random() * 20,
                        Math.random() * 15 + 5,
                        0,
                        Math.PI * 2
                    );
                    ctx.fill();
                }
            }
            ctx.restore();
        }

        // Game loop
        function gameLoop() {
            if (raceStarted) update();
            draw();
            requestAnimationFrame(gameLoop);
        }

        // Start the game
        gameLoop();
    </script>
</body>
</html>
