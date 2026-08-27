<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Simulação Absoluta de Buraco de Minhoca - Tríade MHD+SAMS+SROS</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }
        body, html {
            width: 100%;
            height: 100%;
            overflow: hidden;
            background-color: #000003;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
        }
        canvas {
            display: block;
            width: 100vw;
            height: 100vh;
            position: absolute;
            top: 0;
            left: 0;
            z-index: 1;
        }
        /* Painel HUD Técnico */
        #hud {
            position: absolute;
            top: 20px;
            left: 20px;
            z-index: 10;
            color: #00ffcc;
            background: rgba(0, 5, 15, 0.85);
            padding: 20px;
            border-radius: 8px;
            border: 1px solid #00ffcc;
            box-shadow: 0 0 20px rgba(0, 255, 204, 0.3);
            pointer-events: none;
            max-width: 320px;
        }
        h1 {
            font-size: 15px;
            text-transform: uppercase;
            letter-spacing: 2px;
            margin-bottom: 12px;
            color: #fff;
            border-bottom: 1px solid #00ffcc;
            padding-bottom: 5px;
        }
        .status-item {
            margin-bottom: 8px;
            font-size: 12px;
        }
        .status-value {
            color: #fff;
            font-weight: bold;
        }
        .flash {
            animation: pulse 1s infinite alternate;
        }
        @keyframes pulse {
            from { opacity: 0.5; }
            to { opacity: 1; text-shadow: 0 0 10px #00ffcc; }
        }
        #controls {
            position: absolute;
            bottom: 20px;
            left: 50%;
            transform: translateX(-50%);
            z-index: 10;
            background: rgba(0, 5, 15, 0.9);
            padding: 10px 20px;
            border-radius: 30px;
            border: 1px solid #ff00ff;
            box-shadow: 0 0 15px rgba(255, 0, 255, 0.2);
            color: #fff;
            font-size: 12px;
            pointer-events: none;
            text-align: center;
        }
    </style>
</head>
<body>

    <!-- Tela de Renderização Nativa -->
    <canvas id="wormholeCanvas"></canvas>

    <!-- HUD de Telemetria Sci-Fi -->
    <div id="hud">
        <h1>Métrica Einstein-Rosen</h1>
        <div class="status-item">STATUS DO PORTAL: <span class="status-value flash" style="color: #00ff55;">MHD + SAMS OPERACIONAIS</span></div>
        <div class="status-item">MHD ENERGETICS: <span class="status-value" id="mhd-val">0.00 Eq/Plck</span></div>
        <div class="status-item">SAMS CINETICS: <span class="status-value" style="color: #ffaa00;">Fluido MR Estabilizado</span></div>
        <div class="status-item">SROS ALIGNMENT: <span class="status-value" id="sros-val">100% Sincronizado</span></div>
        <div class="status-item" style="margin-top: 10px; border-top: 1px dashed #ff00ff; padding-top: 8px;">
            SISTEMA INTEGRADO: <span class="status-value" style="color: #ff00ff;">Motor de Dobra Ativo</span>
        </div>
    </div>

    <div id="controls">
        🌌 <b>Simulação Autônoma:</b> Movendo pelo tecido do vácuo quântico a velocidades relativísticas.
    </div>

    <script>
        const canvas = document.getElementById('wormholeCanvas');
        const ctx = canvas.getContext('2d');

        // Ajusta o tamanho do Canvas para preencher a tela inteira
        function resize() {
            canvas.width = window.innerWidth;
            canvas.height = window.innerHeight;
        }
        window.addEventListener('resize', resize);
        resize();

        // Parâmetros do Algoritmo Cósmico
        const particleCount = 450;
        const particles = [];
        let speed = 0.04; // Velocidade de aproximação do túnel
        let rotationAngle = 0;

        // Configuração inicial das partículas em coordenadas 3D falsas (X, Y, Z)
        for (let i = 0; i < particleCount; i++) {
            // Distribui as partículas em formato circular ao longo de um tubo de profundidade Z
            let angle = Math.random() * Math.PI * 2;
            let radius = 100 + Math.random() * 200; 
            
            particles.push({
                x: Math.cos(angle) * radius,
                y: Math.sin(angle) * radius,
                z: Math.random() * 1000, // Profundidade inicial
                color: Math.random() > 0.5 ? '#00ffff' : '#ff00ff', // Azul SROS ou Magenta de Energia Negativa
                size: 1 + Math.random() * 2
            });
        }

        // Centro da tela (A Garganta do Buraco de Minhoca)
        let centerX = canvas.width / 2;
        let centerY = canvas.height / 2;

        let time = 0;

        // Loop de Renderização Principal
        function draw() {
            // Limpa a tela com um rastro preto semitransparente para criar o efeito "Motion Blur" de velocidade
            ctx.fillStyle = 'rgba(0, 0, 3, 0.12)';
            ctx.fillRect(0, 0, canvas.width, canvas.height);

            centerX = canvas.width / 2;
            centerY = canvas.height / 2;
            time += 0.02;
            rotationAngle += 0.003;

            // Ordena as partículas pela profundidade (Z) para renderizar as do fundo primeiro (Física de Oclusão)
            particles.sort((a, b) => b.z - a.z);

            for (let i = 0; i < particleCount; i++) {
                let p = particles[i];

                // Faz a partícula avançar em direção ao observador (diminuindo a distância Z)
                p.z -= speed * 150;

                // Se a partícula passou da tela (Z <= 0), ela é reinjetada no fundo do túnel (Loop Infinito)
                if (p.z <= 0) {
                    p.z = 1000;
                    let angle = Math.random() * Math.PI * 2;
                    let radius = 120 + Math.random() * 180;
                    p.x = Math.cos(angle) * radius;
                    p.y = Math.sin(angle) * radius;
                    p.color = Math.random() > 0.5 ? '#00ffff' : '#ff00ff';
                }

                // --- MATEMÁTICA DE PROJEÇÃO 3D PARA 2D ---
                // Quanto mais perto (Z menor), mais afastada do centro a partícula é projetada
                let factor = 300 / p.z; 
                
                // Rotaciona as coordenadas levemente para criar o efeito espiral do reator MHD
                let cosR = Math.cos(rotationAngle);
                let sinR = Math.sin(rotationAngle);
                let rotX = p.x * cosR - p.y * sinR;
                let rotY = p.x * sinR + p.y * cosR;

                let screenX = centerX + rotX * factor;
                let screenY = centerY + rotY * factor;
                let radiusOnScreen = p.size * factor;

                // Desenha apenas se estiver dentro dos limites visuais da tela
                if (screenX >= 0 && screenX <= canvas.width && screenY >= 0 && screenY <= canvas.height && p.z > 10) {
                    ctx.beginPath();
                    ctx.arc(screenX, screenY, radiusOnScreen, 0, Math.PI * 2);
                    
                    // Aplica o Brilho Neon (Efeito Bloom via Hardware de forma nativa)
                    ctx.shadowBlur = radiusOnScreen * 3;
                    ctx.shadowColor = p.color;
                    
                    ctx.fillStyle = p.color;
                    ctx.fill();
                }
            }

            // Desenha as "Cerdas de Laser" do SROS conectando os lados do túnel de forma dinâmica
            ctx.shadowBlur = 0; // Desativa o glow pesado para as linhas para não travar o navegador
            for (let i = 0; i < particles.length - 1; i += 25) {
                let p1 = particles[i];
                let p2 = particles[i+1];
                
                let f1 = 300 / p1.z;
                let f2 = 300 / p2.z;

                if (p1.z > 50 && p2.z > 50) {
                    ctx.beginPath();
                    ctx.moveTo(centerX + p1.x * f1, centerY + p1.y * f1);
                    ctx.lineTo(centerX + p2.x * f2, centerY + p2.y * f2);
                    ctx.strokeStyle = 'rgba(0, 255, 204, 0.08)';
                    ctx.lineWidth = 0.5;
                    ctx.stroke();
                }
            }

            // Atualiza os dados matemáticos falsos no painel HUD para dar vida à interface
            document.getElementById('mhd-val').innerText = (143.0 + Math.sin(time * 3) * 1.8).toFixed(2) + " Eq/Plck";
            document.getElementById('sros-val').innerText = "Estável (" + (100 - Math.random()*0.01).toFixed(4) + "%)";

            requestAnimationFrame(draw);
        }

        // Inicia o motor gráfico nativo
        draw();
    </script>
</body>
</html>
