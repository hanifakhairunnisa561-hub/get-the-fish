<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Game Hiu Pemakan Ikan</title>
    <style>
        body {
            margin: 0;
            padding: 20px;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            background: linear-gradient(180deg, #0077be, #001f3f);
            font-family: Arial, sans-serif;
        }
        
        .game-container {
            text-align: center;
        }
        
        canvas {
            border: 3px solid #00ffff;
            border-radius: 10px;
            box-shadow: 0 0 30px rgba(0, 255, 255, 0.5);
            background: linear-gradient(180deg, #006994, #003d5c);
        }
        
        .score-board {
            color: white;
            font-size: 24px;
            margin-top: 10px;
            text-shadow: 2px 2px 4px rgba(0,0,0,0.5);
        }
        
        .controls {
            color: white;
            margin-top: 10px;
            font-size: 16px;
            text-shadow: 1px 1px 2px rgba(0,0,0,0.5);
        }
    </style>
</head>
<body>
    <div class="game-container">
        <canvas id="gameCanvas"></canvas>
        <div class="score-board">
            🦈 Skor: <span id="score">0</span> | 🐟 Ikan: <span id="fishCount">0</span>
        </div>
        <div class="controls">
            🎮 Gunakan tombol WASD atau Panah untuk menggerakkan hiu | Spasi untuk mulai ulang
        </div>
    </div>

    <script>
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        
        // Ukuran canvas
        canvas.width = 800;
        canvas.height = 500;
        
        // Variabel game
        let score = 0;
        let gameRunning = true;
        
        // Pergerakan hiu
        const keys = {
            w: false,
            a: false,
            s: false,
            d: false,
            ArrowUp: false,
            ArrowLeft: false,
            ArrowDown: false,
            ArrowRight: false
        };
        
        // Objek hiu
        const shark = {
            x: canvas.width / 2,
            y: canvas.height / 2,
            width: 80,
            height: 40,
            speed: 5,
            direction: 1 // 1 = kanan, -1 = kiri
        };
        
        // Array ikan
        let fishes = [];
        const maxFishes = 10;
        
        // Kelas Ikan
        class Fish {
            constructor() {
                this.width = 30;
                this.height = 20;
                this.reset();
            }
            
            reset() {
                // Muncul dari sisi acak
                const side = Math.floor(Math.random() * 4);
                switch(side) {
                    case 0: // atas
                        this.x = Math.random() * canvas.width;
                        this.y = -30;
                        break;
                    case 1: // kanan
                        this.x = canvas.width + 30;
                        this.y = Math.random() * canvas.height;
                        break;
                    case 2: // bawah
                        this.x = Math.random() * canvas.width;
                        this.y = canvas.height + 30;
                        break;
                    case 3: // kiri
                        this.x = -30;
                        this.y = Math.random() * canvas.height;
                        break;
                }
                
                // Kecepatan acak
                this.speedX = (Math.random() - 0.5) * 2;
                this.speedY = (Math.random() - 0.5) * 2;
                
                // Pastikan ikan bergerak
                if (Math.abs(this.speedX) < 0.5 && Math.abs(this.speedY) < 0.5) {
                    this.speedX = (Math.random() - 0.5) * 3;
                    this.speedY = (Math.random() - 0.5) * 3;
                }
                
                // Warna ikan
                this.color = `hsl(${Math.random() * 60 + 180}, 70%, 60%)`;
                this.direction = this.speedX > 0 ? 1 : -1;
            }
            
            update() {
                this.x += this.speedX;
                this.y += this.speedY;
                
                // Update arah
                if (this.speedX !== 0) {
                    this.direction = this.speedX > 0 ? 1 : -1;
                }
                
                // Cek jika ikan keluar dari layar
                if (this.x < -50 || this.x > canvas.width + 50 || 
                    this.y < -50 || this.y > canvas.height + 50) {
                    return true; // Harus di-reset
                }
                return false;
            }
            
            draw(ctx) {
                ctx.save();
                ctx.translate(this.x, this.y);
                ctx.scale(this.direction, 1);
                
                // Badan ikan
                ctx.fillStyle = this.color;
                ctx.beginPath();
                ctx.ellipse(0, 0, 15, 10, 0, 0, Math.PI * 2);
                ctx.fill();
                
                // Sirip ekor
                ctx.beginPath();
                ctx.moveTo(-15, 0);
                ctx.lineTo(-25, -8);
                ctx.lineTo(-25, 8);
                ctx.closePath();
                ctx.fill();
                
                // Mata
                ctx.fillStyle = 'white';
                ctx.beginPath();
                ctx.arc(10, -3, 4, 0, Math.PI * 2);
                ctx.fill();
                ctx.fillStyle = 'black';
                ctx.beginPath();
                ctx.arc(11, -3, 2, 0, Math.PI * 2);
                ctx.fill();
                
                ctx.restore();
            }
        }
        
        // Inisialisasi ikan
        function initFishes() {
            fishes = [];
            for (let i = 0; i < maxFishes; i++) {
                fishes.push(new Fish());
            }
        }
        
        // Gambar hiu
        function drawShark() {
            ctx.save();
            ctx.translate(shark.x, shark.y);
            ctx.scale(shark.direction, 1);
            
            // Badan hiu
            ctx.fillStyle = '#808080';
            ctx.beginPath();
            ctx.moveTo(-40, 0);
            ctx.lineTo(-20, -15);
            ctx.lineTo(20, -15);
            ctx.lineTo(40, 0);
            ctx.lineTo(20, 15);
            ctx.lineTo(-20, 15);
            ctx.closePath();
            ctx.fill();
            
            // Sirip atas
            ctx.fillStyle = '#606060';
            ctx.beginPath();
            ctx.moveTo(-5, -15);
            ctx.lineTo(0, -30);
            ctx.lineTo(10, -15);
            ctx.closePath();
            ctx.fill();
            
            // Sirip ekor
            ctx.beginPath();
            ctx.moveTo(-40, 0);
            ctx.lineTo(-55, -15);
            ctx.lineTo(-55, 15);
            ctx.closePath();
            ctx.fill();
            
            // Mata
            ctx.fillStyle = 'white';
            ctx.beginPath();
            ctx.arc(25, -5, 6, 0, Math.PI * 2);
            ctx.fill();
            ctx.fillStyle = 'black';
            ctx.beginPath();
            ctx.arc(27, -5, 3, 0, Math.PI * 2);
            ctx.fill();
            
            // Mulut
            ctx.strokeStyle = 'black';
            ctx.lineWidth = 2;
            ctx.beginPath();
            ctx.moveTo(35, 2);
            ctx.lineTo(25, 5);
            ctx.stroke();
            
            ctx.restore();
        }
        
        // Update pergerakan hiu
        function updateShark() {
            let moveX = 0;
            let moveY = 0;
            
            if (keys.w || keys.ArrowUp) moveY -= 1;
            if (keys.s || keys.ArrowDown) moveY += 1;
            if (keys.a || keys.ArrowLeft) moveX -= 1;
            if (keys.d || keys.ArrowRight) moveX += 1;
            
            // Normalisasi gerakan diagonal
            if (moveX !== 0 && moveY !== 0) {
                moveX *= 0.707;
                moveY *= 0.707;
            }
            
            // Update posisi
            shark.x += moveX * shark.speed;
            shark.y += moveY * shark.speed;
            
            // Update arah hiu
            if (moveX > 0) shark.direction = 1;
            if (moveX < 0) shark.direction = -1;
            
            // Batasi hiu di dalam canvas
            shark.x = Math.max(40, Math.min(canvas.width - 40, shark.x));
            shark.y = Math.max(20, Math.min(canvas.height - 20, shark.y));
        }
        
        // Deteksi tabrakan
        function checkCollision(shark, fish) {
            const dx = shark.x - fish.x;
            const dy = shark.y - fish.y;
            const distance = Math.sqrt(dx * dx + dy * dy);
            return distance < 40; // Jarak tabrakan
        }
        
        // Update game
        function update() {
            if (!gameRunning) return;
            
            // Update hiu
            updateShark();
            
            // Update ikan dan cek tabrakan
            for (let i = fishes.length - 1; i >= 0; i--) {
                const fish = fishes[i];
                const outOfBounds = fish.update();
                
                // Cek tabrakan
                if (checkCollision(shark, fish)) {
                    score++;
                    fish.reset();
                    document.getElementById('score').textContent = score;
                } else if (outOfBounds) {
                    fish.reset();
                }
            }
            
            // Update jumlah ikan
            document.getElementById('fishCount').textContent = maxFishes;
        }
        
        // Render game
        function render() {
            // Clear canvas
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            
            // Gambar efek air
            for (let i = 0; i < 20; i++) {
                ctx.fillStyle = 'rgba(255, 255, 255, 0.03)';
                ctx.fillRect(
                    Math.random() * canvas.width,
                    Math.random() * canvas.height,
                    20, 2
                );
            }
            
            // Gambar semua ikan
            fishes.forEach(fish => fish.draw(ctx));
            
            // Gambar hiu
            drawShark();
            
            // Tampilkan pesan jika game over (tidak ada game over di game ini)
            if (!gameRunning) {
                ctx.fillStyle = 'rgba(0, 0, 0, 0.5)';
                ctx.fillRect(0, 0, canvas.width, canvas.height);
                ctx.fillStyle = 'white';
                ctx.font = '40px Arial';
                ctx.textAlign = 'center';
                ctx.fillText('GAME PAUSED', canvas.width/2, canvas.height/2);
            }
        }
        
        // Game loop
        function gameLoop() {
            update();
            render();
            requestAnimationFrame(gameLoop);
        }
        
        // Event listeners
        document.addEventListener('keydown', (e) => {
            if (e.key in keys) {
                keys[e.key] = true;
                e.preventDefault();
            }
            
            // Spasi untuk reset
            if (e.key === ' ') {
                score = 0;
                document.getElementById('score').textContent = score;
                shark.x = canvas.width / 2;
                shark.y = canvas.height / 2;
                initFishes();
                gameRunning = true;
                e.preventDefault();
            }
        });
        
        document.addEventListener('keyup', (e) => {
            if (e.key in keys) {
                keys[e.key] = false;
                e.preventDefault();
            }
        });
        
        // Mulai game
        initFishes();
        gameLoop();
    </script>
</body>
</html>
