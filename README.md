<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Simulação de Buraco de Minhoca - Efeito Bloom Neon</title>
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
        /* HUD de Telemetria */
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
            box-shadow: 0 0 20px rgba(0, 255, 204, 0.4);
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
            to { opacity: 1; text-shadow: 0 0 10px #00ffcc; }
        }
        #controls {
            position: absolute;
            bottom: 20px;
            left: 50%;
            transform: translateX(-50%);
            z-index: 10;
            background: rgba(0, 5, 15, 0.9);
            padding: 12px 25px;
            border-radius: 30px;
            border: 1px solid #ff00ff;
            box-shadow: 0 0 15px rgba(255, 0, 255, 0.3);
            color: #fff;
            font-size: 13px;
            pointer-events: auto;
            text-align: center;
        }
    </style>
</head>
<body>

    <div id="canvas-container"></div>

    <div id="hud">
        <h1>Métrica Einstein-Rosen</h1>
        <div class="status-item">STATUS DO PORTAL: <span class="status-value flash" style="color: #00ff55;">HIPER-ESTÁVEL</span></div>
        <div class="status-item">MHD ENERGY: <span class="status-value" id="mhd-val">Extraindo Vácuo...</span></div>
        <div class="status-item">SAMS CINETICS: <span class="status-value" style="color: #ffaa00;">Fluido Ativo (Filtro Bloom)</span></div>
        <div class="status-item">SROS ALIGNMENT: <span class="status-value" id="sros-val">100% Full Duplex</span></div>
        <div class="status-item" style="margin-top: 10px; border-top: 1px dashed #ff00ff; padding-top: 8px;">
            FILTRO QUANTICO: <span class="status-value" style="color: #ff00ff;">Pós-Processamento Ativo</span>
        </div>
    </div>

    <div id="controls">
        🖱️ <b>Interação Ativa:</b> Clique, arraste e use o scroll para navegar pelo vácuo quântico.
    </div>

    <!-- Scripts do Three.js na mesma versão para evitar conflito -->
    <script src="https://jsdelivr.net"></script>
    <script src="https://jsdelivr.net"></script>
    
    <!-- Scripts de Pós-Processamento (Essenciais para o efeito Bloom/Neon) -->
    <script src="https://jsdelivr.net"></script>
    <script src="https://jsdelivr.net"></script>
    <script src="https://jsdelivr.net"></script>
    <script src="https://jsdelivr.net"></script>
    <script src="https://jsdelivr.net"></script>
    <script src="https://jsdelivr.net"></script>

    <script>
        const container = document.getElementById('canvas-container');
        const scene = new THREE.Scene();
        scene.fog = new THREE.FogExp2(0x000005, 0.02);

        const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
        camera.position.z = 25;

        const renderer = new THREE.WebGLRenderer({ antialias: true });
        renderer.setSize(window.innerWidth, window.innerHeight);
        renderer.setPixelRatio(window.devicePixelRatio);
        container.appendChild(renderer.domElement);

        const controls = new THREE.OrbitControls(camera, renderer.domElement);
        controls.enableDamping = true;
        controls.dampingFactor = 0.05;
        controls.maxDistance = 50;
        controls.minDistance = 2;

        // --- PIPELINE DE PÓS-PROCESSAMENTO (BLOOM) ---
        const renderScene = new THREE.RenderPass(scene, camera);
        
        // Parâmetros do UnrealBloomPass: (resolução, intensidade do brilho, raio do brilho, corte de luz)
        const bloomPass = new THREE.UnrealBloomPass(
            new THREE.Vector2(window.innerWidth, window.innerHeight), 
            2.5,  // Intensidade do Brilho (Neon forte)
            0.4,  // Raio do espalhamento da luz
            0.1   // Limiar de luminância (quais cores brilham)
        );

        const composer = new THREE.EffectComposer(renderer);
        composer.addPass(renderScene);
        composer.addPass(bloomPass);

        // --- GEOMETRIA DO TÚNEL (BURACO DE MINHOCA) ---
        const points = [];
        for (let i = 0; i < 100; i++) {
            let radius = 5 + Math.pow((i - 50) * 0.25, 2); 
            let angle = i * 0.4;
            points.push(new THREE.Vector3(Math.cos(angle) * radius, Math.sin(angle) * radius, (i - 50) * 1.5));
        }
        
        const tubeCurve = new THREE.CatmullRomCurve3(points);
        const tubeGeometry = new THREE.TubeGeometry(tubeCurve, 100, 3.5, 20, false);

        // Material com cor forte para reagir intensamente ao Bloom
        const tubeMaterial = new THREE.MeshBasicMaterial({
            color: 0x0066ff,
            wireframe: true,
            transparent: true,
            opacity: 0.4,
            blending: THREE.AdditiveBlending
        });
        const wormholeMesh = new THREE.Mesh(tubeGeometry, tubeMaterial);
        scene.add(wormholeMesh);

        // --- FLUXO DE PARTÍCULAS QUANTICAS ---
        const particleCount = 2000; // Aumentado para gerar mais pontos de luz
        const particleGeometry = new THREE.BufferGeometry();
        const positions = new Float32Array(particleCount * 3);
        const colors = new Float32Array(particleCount * 3);

        for (let i = 0; i < particleCount; i++) {
            let t = Math.random();
            let point = tubeCurve.getPoint(t);
            let angle = Math.random() * Math.PI * 2;
            let pRadius = Math.random() * 3.2; 

            positions[i * 3] = point.x + Math.cos(angle) * pRadius;
            positions[i * 3 + 1] = point.y + Math.sin(angle) * pRadius;
            positions[i * 3 + 2] = point.z;

            // Paleta Neon: Ciano (SROS) e Magenta (Energia Negativa)
            let mixColor = Math.random() > 0.5 ? new THREE.Color(0x00ffff) : new THREE.Color(0xff00ff);
            colors[i * 3] = mixColor.r;
            colors[i * 3 + 1] = mixColor.g;
            colors[i * 3 + 2] = mixColor.b;
        }

        particleGeometry.setAttribute('position', new THREE.BufferAttribute(positions, 3));
        particleGeometry.setAttribute('color', new THREE.BufferAttribute(colors, 3));

        const particleMaterial = new THREE.PointsMaterial({
            size: 0.4,
            vertexColors: true,
            transparent: true,
            opacity: 0.9,
            blending: THREE.AdditiveBlending
        });

        const quantumFlow = new THREE.Points(particleGeometry, particleMaterial);
        scene.add(quantumFlow);

        // --- LOOP DE ANIMAÇÃO ---
        let clock = new THREE.Clock();

        function animate() {
            requestAnimationFrame(animate);
            
            const elapsedTime = clock.getElapsedTime();
            const timeData = particleGeometry.attributes.position.array;

            wormholeMesh.rotation.z = elapsedTime * 0.05;
            quantumFlow.rotation.z = -elapsedTime * 0.08;

            // Movimento das partículas acelerado (Simulando Dobra Cósmica)
            for (let i = 0; i < particleCount; i++) {
                timeData[i * 3 + 2] += 0.6; 

                if (timeData[i * 3 + 2] > 75) {
                    timeData[i * 3 + 2] = -75;
                }
            }
            particleGeometry.attributes.position.needsUpdate = true;

            // Efeito pulsar dinâmico na intensidade do Bloom simulando oscilação do Reator MHD
            bloomPass.bloomIntensity = 2.0 + Math.sin(elapsedTime * 4) * 0.5;

            document.getElementById('mhd-val').innerText = (120.5 + Math.sin(elapsedTime * 6) * 2.1).toFixed(2) + " Eq/Plck";
            document.getElementById('sros-val').innerText = "Sincronizado (" + (100 - Math.random()*0.02).toFixed(4) + "%)";

            controls.update();
            
            // Renderização através do COMPOSER (Aplica os filtros de luz)
            composer.render();
        }

        window.addEventListener('resize', onWindowResize, false);

        function onWindowResize() {
            camera.aspect = window.innerWidth / window.innerHeight;
            camera.updateProjectionMatrix();
