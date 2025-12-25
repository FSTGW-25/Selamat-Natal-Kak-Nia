<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
    <title>Selamat Natal Kak Nia</title>
    
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Cormorant+Garamond:wght@600&display=swap" rel="stylesheet">

    <style>
        body { 
            margin: 0; 
            overflow: hidden; 
            background-color: #000000; 
        }
        
        #canvas-container { 
            width: 100vw; 
            height: 100vh; 
            display: block; 
            position: absolute; 
            top: 0; 
            left: 0; 
            z-index: 1;
        }
        
        /* --- 1. JUDUL PERMANEN DI ATAS (FIXED) --- */
        #static-header {
            position: absolute;
            top: 20px;
            width: 100%;
            text-align: center;
            z-index: 20; 
            pointer-events: none;
        }

        .title-text {
            font-family: 'Times New Roman', serif; 
            font-weight: bold;
            font-size: 2.8rem;
            letter-spacing: 4px;
            margin: 0;
            text-transform: uppercase;
            color: #FFD700; 
            text-shadow: 0 0 15px #FFD700, 0 0 30px #FFD700;
        }

        /* --- 2. CONTAINER PESAN (DI TENGAH) --- */
        #message-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            display: flex;
            justify-content: center;
            align-items: center;
            z-index: 10;
            pointer-events: none;
            padding: 20px;
            box-sizing: border-box;
            text-align: center;
        }

        /* --- STYLE PESAN --- */
        .message-text {
            font-family: 'Cormorant Garamond', serif;
            font-weight: 600;
            white-space: pre-wrap;
            line-height: 1.7;
            text-transform: uppercase;
            /* Warna Gradient Pesan (Kuning ke Ungu Muda) */
            background: linear-gradient(180deg, #FFD700 20%, #e5cee8 100%);
            -webkit-background-clip: text;
            -webkit-text-fill-color: transparent;
            background-clip: text;
            color: transparent;
            filter: drop-shadow(0 0 8px rgba(255, 215, 0, 0.3));
            opacity: 1;
            transition: opacity 0.8s ease-in-out, transform 0.8s ease;
        }

        .size-medium { font-size: 2.5rem; letter-spacing: 2px; }
        .size-small { font-size: 1.8rem; letter-spacing: 1px; }

        .hidden { opacity: 0; transform: scale(0.95); }

        #instructions {
            position: absolute; bottom: 20px; width: 100%; text-align: center;
            color: rgba(255, 255, 255, 0.4); font-family: sans-serif; font-size: 0.8rem;
            pointer-events: none; z-index: 15;
        }

        #music-btn {
            position: absolute; bottom: 30px; left: 30px; z-index: 100;
            background: rgba(0, 0, 0, 0.6); border: 1px solid #FFD700; color: #FFD700;
            padding: 10px 15px; font-family: sans-serif; font-size: 0.8rem; cursor: pointer;
            border-radius: 20px; transition: all 0.3s; display: flex; align-items: center; gap: 8px; outline: none;
        }
        #music-btn:hover { background: #FFD700; color: #000; }

        @media (max-width: 768px) {
            .title-text { font-size: 1.8rem; margin-top: 10px; }
            .size-medium { font-size: 1.8rem; }
            .size-small { font-size: 1.2rem; }
        }
    </style>
    <script type="importmap">
        {
            "imports": {
                "three": "https://unpkg.com/three@0.160.0/build/three.module.js",
                "three/addons/": "https://unpkg.com/three@0.160.0/examples/jsm/"
            }
        }
    </script>
</head>
<body>

    <div id="static-header">
        <h1 class="title-text">MERRY CHRISTMAS BOSSSS 😈👻🤍</h1>
    </div>

    <div id="message-overlay">
        <h1 id="dynamic-msg" class="message-text hidden"></h1>
    </div>

    <div id="instructions">Click Anywhere to Open Message</div>
    <div id="canvas-container"></div>

    <button id="music-btn">▶ Play Music</button>
    <audio id="bg-music" loop><source src="Pentatonix - That's Christmas to Me (Official Video).mp3" type="audio/mpeg"></audio>

    <script type="module">
        import * as THREE from 'three';
        import { OrbitControls } from 'three/addons/controls/OrbitControls.js';

        // --- DAFTAR PESAN ---
        const MESSAGES = [
            {
                text: "TUHAN YESUS\nMEMBERKATI SELALU KAKS 🤟",
                style: "size-medium"
            },
            {
                text: "JANGAN MARAH-MARAH YA KAK PA QUEEN\nDENGAN SORRY KATU NDA BALAS CHAT\nDI GRUP TADI MALAM\nNANTI BERIKUT SO NDA MO BAGITU",
                style: "size-small"
            },
            {
                text: "KIRANYA DAMAI DAN SUKACITA NATAL\nMELINGKUPI HATI KAK DAN KELUARGA.\nSELAMAT MENYAMBUT TAHUN BARU YANG PENUH BERKAT! 🤍🌼",
                style: "size-small"
            }
        ];

        const PARTICLE_COUNT = 2500; 
        const TREE_HEIGHT = 45;
        const TREE_RADIUS = 18;
        
        let currentState = -1; 
        let isAnimating = false;
        
        // --- SCENE SETUP ---
        const scene = new THREE.Scene();
        scene.background = new THREE.Color(0x000000);
        // Mengurangi kabut agar warna cerah tetap terlihat dari jauh
        scene.fog = new THREE.FogExp2(0x000000, 0.002);

        const camera = new THREE.PerspectiveCamera(45, window.innerWidth / window.innerHeight, 0.1, 1000);
        camera.position.set(0, 5, 130); 

        const renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true });
        renderer.setSize(window.innerWidth, window.innerHeight);
        renderer.setPixelRatio(window.devicePixelRatio);
        // Meningkatkan exposure agar lebih terang
        renderer.toneMapping = THREE.ACESFilmicToneMapping;
        renderer.toneMappingExposure = 1.5; 
        document.getElementById('canvas-container').appendChild(renderer.domElement);

        const controls = new OrbitControls(camera, renderer.domElement);
        controls.enableDamping = true;
        controls.autoRotate = true;
        controls.autoRotateSpeed = 1.0;
        controls.enableZoom = false;
        controls.target.set(0, 5, 0);

        // --- CAHAYA (DIPERKUAT AGAR TIDAK GELAP) ---
        // Ambient light tinggi agar warna dasar keluar
        const ambientLight = new THREE.AmbientLight(0xffffff, 3.0);
        scene.add(ambientLight);

        // Lampu sorot point lights diperkuat intensitasnya
        const lights = [];
        lights[0] = new THREE.PointLight(0xffd700, 600, 300); // Kuning Emas Kuat
        lights[0].position.set(30, 50, 30);
        
        lights[1] = new THREE.PointLight(0xd000ff, 600, 300); // Ungu Neon Kuat
        lights[1].position.set(-30, 20, -30);
        
        lights[2] = new THREE.DirectionalLight(0xffffff, 2.0); // Cahaya depan
        lights[2].position.set(0, 0, 100);

        scene.add(lights[0]); scene.add(lights[1]); scene.add(lights[2]);

        // --- MATERIAL & PARTIKEL ---
        // Material disetting agar BERSINAR (Emissive)
        const mat = new THREE.MeshStandardMaterial({
            color: 0xffffff, 
            metalness: 0.6, // Dikurangi sedikit agar warna asli lebih terlihat (tidak hitam memantul)
            roughness: 0.1, // Sangat mengkilap
            emissive: 0x444444, // Glow dasar putih/abu agar tidak mati saat gelap
            emissiveIntensity: 0.5
        });

        const sphereGeo = new THREE.SphereGeometry(0.5, 16, 16); // Segmen diperhalus
        const cubeGeo = new THREE.BoxGeometry(0.8, 0.8, 0.8);
        const sphereMesh = new THREE.InstancedMesh(sphereGeo, mat, PARTICLE_COUNT);
        const cubeMesh = new THREE.InstancedMesh(cubeGeo, mat, PARTICLE_COUNT);
        scene.add(sphereMesh); scene.add(cubeMesh);

        const sphereData = []; const cubeData = []; const dummy = new THREE.Object3D();

        function getTreePos(i, total) {
            const y = (1 - (i / total)) * TREE_HEIGHT - (TREE_HEIGHT / 2);
            const r = (1 - (y + TREE_HEIGHT / 2) / TREE_HEIGHT) * TREE_RADIUS * (1 + Math.random()*0.3);
            const angle = i * 0.5 + Math.random();
            return new THREE.Vector3(Math.cos(angle)*r, y, Math.sin(angle)*r);
        }
        function getExplodePos() {
            const v = new THREE.Vector3(Math.random()-0.5, Math.random()-0.5, Math.random()-0.5);
            return v.normalize().multiplyScalar(120 + Math.random() * 200);
        }

        function initData(mesh, dataArr) {
            const color = new THREE.Color();
            for (let i = 0; i < PARTICLE_COUNT; i++) {
                // WARNA POHON CERAH (BUKAN PUCAT)
                if (Math.random() > 0.5) {
                    // Kuning Emas Cerah
                    color.setHex(0xffcc00); 
                } else {
                    // Ungu Neon Cerah Metalik (Bukan ungu muda teks)
                    color.setHex(0xbf00ff); 
                }
                
                mesh.setColorAt(i, color);
                const treeP = getTreePos(i, PARTICLE_COUNT);
                dataArr.push({ currentPos: treeP.clone(), treePos: treeP.clone(), explodePos: getExplodePos() });
                dummy.position.copy(treeP); dummy.updateMatrix(); mesh.setMatrixAt(i, dummy.matrix);
            }
            mesh.instanceMatrix.needsUpdate = true; mesh.instanceColor.needsUpdate = true;
        }
        initData(sphereMesh, sphereData); initData(cubeMesh, cubeData);

        // BINTANG DI PUNCAK
        const topper = new THREE.Mesh(
            new THREE.OctahedronGeometry(2.5, 0),
            new THREE.MeshStandardMaterial({ 
                color: 0xffff00, // Kuning murni
                emissive: 0xffaa00, // Glow oranye
                emissiveIntensity: 3.0, // Sangat terang
                toneMapped: false // Agar tidak kena batas exposure (sangat silau)
            })
        );
        topper.position.set(0, TREE_HEIGHT/2 + 2, 0);
        scene.add(topper);

        // --- LOGIKA INTERAKSI ---
        const contentEl = document.getElementById('dynamic-msg');
        
        function updateTextAndScene() {
            contentEl.classList.add('hidden'); 
            
            setTimeout(() => {
                currentState++;
                if (currentState >= MESSAGES.length) currentState = -1; 

                if (currentState === -1) {
                    animateParticles('treePos');
                    topper.visible = true;
                } else {
                    const msg = MESSAGES[currentState];
                    contentEl.innerText = msg.text;
                    contentEl.className = 'message-text hidden ' + msg.style;
                    
                    animateParticles('explodePos');
                    topper.visible = false;

                    setTimeout(() => { contentEl.classList.remove('hidden'); }, 100);
                }
            }, 800);
        }

        function animateParticles(targetProp) {
            isAnimating = true; let progress = 0; const duration = 100;
            function step() {
                progress++; const alpha = progress / duration; const ease = 1 - Math.pow(1 - alpha, 3);
                updateMesh(sphereMesh, sphereData, targetProp, ease); updateMesh(cubeMesh, cubeData, targetProp, ease);
                if (progress < duration) requestAnimationFrame(step); else isAnimating = false;
            }
            step();
        }

        function updateMesh(mesh, data, targetProp, alpha) {
            for (let i = 0; i < data.length; i++) {
                const d = data[i]; d.currentPos.lerp(d[targetProp], 0.05);
                dummy.position.copy(d.currentPos);
                // Rotasi lebih cepat agar efek metaliknya kelihatan (berkedip)
                dummy.rotation.x += 0.03; dummy.rotation.y += 0.03;
                const scale = (targetProp === 'explodePos') ? 0.3 : 1.0;
                dummy.scale.setScalar(scale); dummy.updateMatrix(); mesh.setMatrixAt(i, dummy.matrix);
            }
            mesh.instanceMatrix.needsUpdate = true;
        }

        window.addEventListener('mousedown', onInput); window.addEventListener('touchstart', onInput, {passive: false});
        function onInput(e) {
            if (e.target.id === 'music-btn') return;
            if (e.type === 'touchstart') e.preventDefault();
            const audio = document.getElementById('bg-music');
            if (audio.paused && !isPlaying) toggleMusic();
            if (!isAnimating) updateTextAndScene();
        }

        let isPlaying = false;
        const musicBtn = document.getElementById('music-btn');
        musicBtn.addEventListener('click', (e) => { e.stopPropagation(); toggleMusic(); });
        function toggleMusic() {
            const audio = document.getElementById('bg-music');
            if (audio.paused) { audio.play(); musicBtn.innerHTML = "|| Pause"; isPlaying = true; }
            else { audio.pause(); musicBtn.innerHTML = "▶ Play Music"; isPlaying = false; }
        }

        function animate() {
            requestAnimationFrame(animate); controls.update();
            if (!isAnimating) {
                const target = (currentState === -1) ? 'treePos' : 'explodePos';
                updateMesh(sphereMesh, sphereData, target, 0.1); updateMesh(cubeMesh, cubeData, target, 0.1);
            }
            renderer.render(scene, camera);
        }
        animate();

        window.addEventListener('resize', () => {
            camera.aspect = window.innerWidth / window.innerHeight; camera.updateProjectionMatrix();
            renderer.setSize(window.innerWidth, window.innerHeight);
        });
    </script>
</body>
</html>
