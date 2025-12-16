#
<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Flappy Bird Sederhana</title>
    <style>
        /* CSS DISINI */
        body {
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            margin: 0;
            background-color: #f0f0f0;
            font-family: sans-serif;
        }

        #game-container {
            position: relative;
            border: 5px solid #333;
            box-shadow: 0 0 20px rgba(0, 0, 0, 0.5);
            background-color: #70c5ce; /* Warna langit */
        }

        #gameCanvas {
            display: block;
        }

        #start-screen, #game-over {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            background: rgba(0, 0, 0, 0.7);
            color: white;
            text-align: center;
            padding: 20px;
            box-sizing: border-box;
        }

        h1 {
            margin-top: 0;
        }

        button {
            padding: 10px 20px;
            font-size: 1.2em;
            cursor: pointer;
            background-color: #4CAF50;
            color: white;
            border: none;
            border-radius: 5px;
            margin-top: 15px;
        }
    </style>
</head>
<body>
    <div id="game-container">
        <canvas id="gameCanvas" width="400" height="600"></canvas>
        <div id="start-screen">
            <h1>Flappy Bird Sederhana</h1>
            <p>Klik atau tekan Spasi untuk Melompat!</p>
            <button id="startButton">Mulai Game</button>
        </div>
        <div id="game-over" style="display:none;">
            <h2>Game Over!</h2>
            <p>Skor Anda: <span id="final-score">0</span></p>
            <button id="restartButton">Main Lagi</button>
        </div>
    </div>

    <script>
        // JAVASCRIPT DISINI
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        const startScreen = document.getElementById('start-screen');
        const gameOverScreen = document.getElementById('game-over');
        const startButton = document.getElementById('startButton');
        const restartButton = document.getElementById('restartButton');
        const finalScoreElement = document.getElementById('final-score');

        // --- Variabel Game ---
        let bird;
        let pipes = [];
        let score;
        let gameInterval;
        let isGameOver;

        const CANVAS_WIDTH = canvas.width;
        const CANVAS_HEIGHT = canvas.height;
        const PIPE_WIDTH = 50;
        const PIPE_GAP = 150; // Jarak vertikal antar pipa
        const PIPE_SPEED = 2;

        // Objek Burung
        const BIRD_SIZE = 20;
        const GRAVITY = 0.25;
        const JUMP_VELOCITY = -5;

        function Bird() {
            this.x = 50;
            this.y = CANVAS_HEIGHT / 2;
            this.velocity = 0;

            this.draw = function() {
                ctx.fillStyle = 'yellow';
                ctx.fillRect(this.x, this.y, BIRD_SIZE, BIRD_SIZE);
                // Gambar mata sederhana
                ctx.fillStyle = 'black';
                ctx.fillRect(this.x + BIRD_SIZE - 5, this.y + 5, 2, 2);
            };

            this.update = function() {
                this.velocity += GRAVITY;
                this.y += this.velocity;

                // Batasan atas
                if (this.y < 0) {
                    this.y = 0;
                    this.velocity = 0;
                }
            };

            this.jump = function() {
                this.velocity = JUMP_VELOCITY;
            };
        }

        // Objek Pipa
        function Pipe(x) {
            this.x = x;
            this.height = Math.floor(Math.random() * (CANVAS_HEIGHT - 300)) + 100; // Tinggi pipa atas
            this.bottomY = this.height + PIPE_GAP;

            this.draw = function() {
                ctx.fillStyle = 'green';
                // Pipa Atas
                ctx.fillRect(this.x, 0, PIPE_WIDTH, this.height);
                // Pipa Bawah
                ctx.fillRect(this.x, this.bottomY, PIPE_WIDTH, CANVAS_HEIGHT - this.bottomY);
            };

            this.update = function() {
                this.x -= PIPE_SPEED;
            };
        }

        // --- Fungsi Game Utama ---

        function initGame() {
            bird = new Bird();
            pipes = [];
            score = 0;
            isGameOver = false;

            // Sembunyikan layar start/game over
            startScreen.style.display = 'none';
            gameOverScreen.style.display = 'none';
        }

        function startGame() {
            initGame();
            // Tambahkan pipa awal
            pipes.push(new Pipe(CANVAS_WIDTH));
            pipes.push(new Pipe(CANVAS_WIDTH + 200));

            gameInterval = setInterval(gameLoop, 1000 / 60); // 60 FPS
            
            // Dengarkan klik untuk melompat
            canvas.addEventListener('mousedown', handleJump);
            window.addEventListener('keydown', handleJump);
        }

        function handleJump(event) {
            if (!isGameOver) {
                // Cek jika event adalah klik atau tombol spasi
                if (event.type === 'mousedown' || event.keyCode === 32) {
                    bird.jump();
                    // Mencegah scroll saat spasi ditekan
                    if (event.keyCode === 32) {
                        event.preventDefault();
                    }
                }
            } else {
                // Jika game over, abaikan input
                event.stopPropagation();
            }
        }


        function checkCollision() {
            // 1. Tabrakan dengan Lantai/Langit
            if (bird.y + BIRD_SIZE > CANVAS_HEIGHT || bird.y < 0) {
                return true;
            }

            // 2. Tabrakan dengan Pipa
            for (let i = 0; i < pipes.length; i++) {
                const p = pipes[i];

                // Cek apakah burung berada di antara koordinat X pipa
                if (bird.x + BIRD_SIZE > p.x && bird.x < p.x + PIPE_WIDTH) {
                    // Cek tabrakan dengan pipa atas ATAU pipa bawah
                    if (bird.y < p.height || bird.y + BIRD_SIZE > p.bottomY) {
                        return true;
                    }
                }
            }
            return false;
        }

        function updateGame() {
            bird.update();

            // Perbarui posisi pipa dan hapus pipa yang sudah lewat
            for (let i = pipes.length - 1; i >= 0; i--) {
                pipes[i].update();

                // Tambah skor jika burung melewati pipa
                if (pipes[i].x + PIPE_WIDTH < bird.x && !pipes[i].scored) {
                    score++;
                    pipes[i].scored = true; // Tandai pipa sudah dihitung skor
                }

                // Hapus pipa yang sudah keluar layar
                if (pipes[i].x + PIPE_WIDTH < 0) {
                    pipes.splice(i, 1);
                }
            }

            // Tambahkan pipa baru secara berkala
            if (pipes.length > 0 && pipes[pipes.length - 1].x < CANVAS_WIDTH - 200) {
                pipes.push(new Pipe(CANVAS_WIDTH));
            }

            // Cek tabrakan
            if (checkCollision()) {
                gameOver();
            }
        }

        function drawGame() {
            // Bersihkan Canvas (warna langit)
            ctx.fillStyle = '#70c5ce';
            ctx.fillRect(0, 0, CANVAS_WIDTH, CANVAS_HEIGHT);

            // Gambar Pipa
            pipes.forEach(p => p.draw());

            // Gambar Burung
            bird.draw();

            // Gambar Lantai
            ctx.fillStyle = '#ded895';
            ctx.fillRect(0, CANVAS_HEIGHT - 20, CANVAS_WIDTH, 20);
            ctx.strokeStyle = '#6d6027';
            ctx.lineWidth = 2;
            ctx.strokeRect(0, CANVAS_HEIGHT - 20, CANVAS_WIDTH, 20);


            // Gambar Skor
            ctx.fillStyle = 'white';
            ctx.font = '30px Arial';
            ctx.textAlign = 'left';
            ctx.fillText('Skor: ' + score, 10, 40);
        }

        function gameLoop() {
            if (!isGameOver) {
                updateGame();
                drawGame();
            }
        }

        function gameOver() {
            isGameOver = true;
            clearInterval(gameInterval);
            
            // Hapus event listener saat game over
            canvas.removeEventListener('mousedown', handleJump);
            window.removeEventListener('keydown', handleJump);

            finalScoreElement.textContent = score;
            gameOverScreen.style.display = 'flex';
        }

        // --- Event Listener Tombol ---

        startButton.addEventListener('click', startGame);
        restartButton.addEventListener('click', startGame);

        // Tampilkan layar awal saat pertama kali dimuat
        drawGame(); 
        ctx.fillStyle = '#333';
        ctx.font = '40px Arial';
        ctx.textAlign = 'center';
        ctx.fillText('Tekan Mulai!', CANVAS_WIDTH / 2, CANVAS_HEIGHT / 2);
    </script>
</body/>
