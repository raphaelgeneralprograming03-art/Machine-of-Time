<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Simulação de Buraco de Minhoca - Tríade MHD+SAMS+SROS</title>
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
            background-color: #000;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
        }
        #canvas-container {
            width: 100%;
            height: 100%;
            position: absolute;
            top: 0;
            left: 0;
            z-index: 1;
        }
        /* Painel HUD de Telemetria Sci-Fi */
        #hud {
            position: absolute;
            top: 20px;
            left: 20px;
            z-index: 10;
            color: #00ffcc;
            background: rgba(0, 10, 20, 0.75);
            padding: 20px;
            border-radius: 8px;
            border: 1px solid #00ffcc;
            box-shadow: 0 0 15px rgba(0, 255, 204, 0.3);
            pointer-events: none;
            max-width: 320px;
        }
        h1 {
            font-size: 16px;
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
            to { opacity: 1; text-shadow: 0 0 8px #00ffcc; }
        }
        /* Controles Interativos */
        #controls {
            position: absolute;
            bottom: 20px;
            left: 50%;
            transform: translateX(-50%);
            z-index: 10;
            background: rgba(0, 10, 20, 0.8);
            padding: 12px 25px;
            border-radius: 30px;
            border: 1px solid #00aaff;
            color: #fff;
            font-size: 13px;
            pointer-events: auto;
            text-align: center;
        }
    </style>
</head>
<body>

    <div id="canvas-container"></div>

    <!-- HUD de Operação da Tríade -->
    <div id="hud">
        <h1>Métrica Einstein-Rosen</h1>
        <div class="status-item">STATUS DO PORTAL: <span class="status-value flash" style="color: #00ff55;">ESTÁVEL</span></div>
        <div class="status-item">MHD ENERGETICS: <span class="status-value" id="mhd-val">Extraindo Vácuo...</span></div>
        <div class="status-item">SAMS CINETICS: <span class="status-value" style="color: #ffaa00;">Fluido MR Ativo</span></div>
        <div class="status-item">SROS ALIGNMENT: <span class="status-value" id="sros-val">100% Full Duplex</span></div>
        <div class="status-item" style="margin-top: 10px; border-top: 1px dashed #00ffcc; padding-top: 8px;">
            LATÊNCIA CAUSAL: <span class="status-value" style="color: #ff3333;">0.00ms (Dobra Interna)</span>
        </div>
    </div>

    <div id="controls">
        🖱️ <b>Interação:</b> Clique e arraste para distorcer a perspectiva do espaço-tempo.
    </div>

    <!-- Scripts de Renderização (Three.js + OrbitControls) -->
    <script src="https://cloudflare.com"></script>
    <script src="https://jsdelivr.net"></script>

    <script>
        // Configuração Inicial da Cena
        const container = document.getElementById('canvas-container');
        const scene = new THREE.Scene();
        scene.fog = new THREE.FogExp2(0x00000a, 0.015);

        const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
        camera.position.z = 30;

        const renderer = new THREE.WebGLRenderer({ antialias: true });
        renderer.setSize(window.innerWidth, window.innerHeight);
        renderer.setPixelRatio(window.devicePixelRatio);
        container.appendChild(renderer.domElement);

        // Controles de Câmera Interativos
        const controls = new THREE.OrbitControls(camera, renderer.domElement);
        controls.enableDamping = true;
        controls.dampingFactor = 0.05;
        controls.maxDistance = 60;
        controls.minDistance = 5;

        // --- ALGORITMO DE CONSTRUÇÃO DO TUNEL (BURACO DE MINHOCA) ---
        // Criando uma curva geométrica em formato de mola/túnel infinito
        const points = [];
        for (let i = 0; i < 100; i++) {
            // A curva se estreita no meio (a garganta de Einstein-Rosen) e expande nas pontas
            let radius = 6 + Math.pow((i - 50) * 0.25, 2); 
            let angle = i * 0.4;
            points.push(new THREE.Vector3(Math.cos(angle) * radius, Math.sin(angle) * radius, (i - 50) * 1.5));
        }
        
        const tubeCurve = new THREE.CatmullRomCurve3(points);
        // Gera a geometria tubular baseada na curva matemática
        const tubeGeometry = new THREE.TubeGeometry(tubeCurve, 100, 3, 16, false);

        // --- MATERIAIS ENERGÉTICOS (Representando MHD e SAMS) ---
        // Malha de Aramado Dinâmico (Wireframe) para dar o efeito de grade dimensional
        const tubeMaterial = new THREE.MeshBasicMaterial({
            color: 0x0055ff,
            wireframe: true,
            transparent: true,
            opacity: 0.35,
            blending: THREE.AdditiveBlending
        });
        const wormholeMesh = new THREE.Mesh(tubeGeometry, tubeMaterial);
        scene.add(wormholeMesh);

        // --- SISTEMA SROS: Rede de Partículas Laser / Estrelas Guias ---
        const particleCount = 1200;
        const particleGeometry = new THREE.BufferGeometry();
        const positions = new Float32Array(particleCount * 3);
        const colors = new Float32Array(particleCount * 3);

        // Distribui partículas ao longo da geometria do túnel para simular o fluxo quântico
        for (let i = 0; i < particleCount; i++) {
            let t = Math.random();
            let point = tubeCurve.getPoint(t);
            let angle = Math.random() * Math.PI * 2;
            let pRadius = Math.random() * 2.5; // Concentradas perto das paredes

            positions[i * 3] = point.x + Math.cos(angle) * pRadius;
            positions[i * 3 + 1] = point.y + Math.sin(angle) * pRadius;
            positions[i * 3 + 2] = point.z;

            // Cores alternando entre o azul quântico e o ciano do SROS
            let mixColor = Math.random() > 0.4 ? new THREE.Color(0x00ffff) : new THREE.Color(0xff00ff);
            colors[i * 3] = mixColor.r;
            colors[i * 3 + 1] = mixColor.g;
            colors[i * 3 + 2] = mixColor.b;
        }

        particleGeometry.setAttribute('position', new THREE.BufferAttribute(positions, 3));
        particleGeometry.setAttribute('color', new THREE.BufferAttribute(colors, 3));

        const particleMaterial = new THREE.PointsMaterial({
            size: 0.35,
            vertexColors: true,
            transparent: true,
            opacity: 0.8,
            blending: THREE.AdditiveBlending
        });

        const quantumFlow = new THREE.Points(particleGeometry, particleMaterial);
        scene.add(quantumFlow);

        // Iluminação de Ambiente
        const ambientLight = new THREE.AmbientLight(0xffffff, 0.5);
        scene.add(ambientLight);

        // --- LOOP DE ANIMAÇÃO (A Física em Movimento) ---
        let clock = new THREE.Clock();

        function animate() {
            requestAnimationFrame(animate);
            
            const elapsedTime = clock.getElapsedTime();
            const timeData = particleGeometry.attributes.position.array;

            // Faz o túnel rotacionar simulando o arrasto de vácuo do Reator MHD
            wormholeMesh.rotation.z = elapsedTime * 0.08;
            quantumFlow.rotation.z = -elapsedTime * 0.12;

            // Algoritmo de movimentação das partículas (O fluxo atravessando o buraco)
            for (let i = 0; i < particleCount; i++) {
                // Move as partículas no eixo Z (profundidade do túnel)
                timeData[i * 3 + 2] += 0.4; 

                // Se a partícula passar do final do túnel, resgata ela de volta para o início (Infinito)
                if (timeData[i * 3 + 2] > 75) {
                    timeData[i * 3 + 2] = -75;
                }
            }
            particleGeometry.attributes.position.needsUpdate = true;

            // Flutuação cosmética dos dados do HUD simulando telemetria real
            document.getElementById('mhd-val').innerText = (98.4 + Math.sin(elapsedTime * 5) * 1.2).toFixed(2) + " Eq/Plck";
            document.getElementById('sros-val').innerText = "Estável (" + (100 - Math.random()*0.05).toFixed(3) + "%)";

            controls.update();
            renderer.render(scene, camera);
        }

        // Responsividade do tamanho da tela
        window.addEventListener('resize', onWindowResize, false);

        function onWindowResize() {
            camera.aspect = window.innerWidth / window.innerHeight;
            camera.updateProjectionMatrix();
            renderer.setSize(window.innerWidth, window.innerHeight);
        }

        // Inicia a simulação
        animate();
    </script>
</body>
</html>
