Berikut adalah kode lengkap game balapan 3D sederhana menggunakan Three.js. Simpan sebagai file HTML dan buka di browser.
```html
<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
    <title>Game Balapan 3D - Arcade Racer</title>
    <style>
        body {
            margin: 0;
            overflow: hidden;
            font-family: 'Segoe UI', 'Courier New', monospace;
        }
        #info {
            position: absolute;
            top: 20px;
            left: 20px;
            color: white;
            background: rgba(0,0,0,0.7);
            padding: 10px 18px;
            border-radius: 8px;
            backdrop-filter: blur(5px);
            pointer-events: none;
            z-index: 10;
            font-weight: bold;
            letter-spacing: 1px;
            border-left: 4px solid #ff5500;
            box-shadow: 0 2px 10px rgba(0,0,0,0.3);
        }
        #speed-panel {
            position: absolute;
            bottom: 30px;
            right: 30px;
            background: rgba(0,0,0,0.8);
            color: #0f0;
            font-family: monospace;
            font-size: 28px;
            padding: 12px 24px;
            border-radius: 12px;
            font-weight: bold;
            backdrop-filter: blur(4px);
            pointer-events: none;
            z-index: 10;
            text-align: center;
            border: 1px solid #0f0;
            box-shadow: 0 0 15px rgba(0,255,0,0.3);
        }
        #speed-label {
            font-size: 14px;
            color: #aaa;
            letter-spacing: 2px;
        }
        #controls-hint {
            position: absolute;
            bottom: 20px;
            left: 20px;
            background: rgba(0,0,0,0.6);
            color: #ccc;
            padding: 8px 15px;
            border-radius: 20px;
            font-size: 14px;
            font-weight: bold;
            font-family: monospace;
            pointer-events: none;
            z-index: 10;
        }
        @media (max-width: 600px) {
            #speed-panel { font-size: 20px; padding: 8px 16px; bottom: 15px; right: 15px; }
            #info { font-size: 12px; top: 10px; left: 10px; }
            #controls-hint { font-size: 10px; }
        }
    </style>
</head>
<body>
    <div id="info">
        🏁 ARCADE RACER 🏁 | Hindari rintangan! 
    </div>
    <div id="speed-panel">
        <div id="speed-label">SPEED</div>
        <span id="speed-value">0</span> <span style="font-size:18px;">km/h</span>
    </div>
    <div id="controls-hint">
        🎮 ← →  untuk menggeser mobil | 🚗 Koleksi kotak orange +10 poin
    </div>

    <!-- Import Three.js core dan addons -->
    <script type="importmap">
        {
            "imports": {
                "three": "https://unpkg.com/three@0.128.0/build/three.module.js",
                "three/addons/": "https://unpkg.com/three@0.128.0/examples/jsm/"
            }
        }
    </script>

    <script type="module">
        import * as THREE from 'three';
        import { OrbitControls } from 'three/addons/controls/OrbitControls.js';
        import { CSS2DRenderer, CSS2DObject } from 'three/addons/renderers/CSS2DRenderer.js';

        // --- Setup Scene, Camera, Renderers ---
        const scene = new THREE.Scene();
        scene.background = new THREE.Color(0x071a3b); // langit malam
        scene.fog = new THREE.FogExp2(0x071a3b, 0.008); // kabut tipis

        // Kamera perspektif yang mengikuti mobil dari belakang
        const camera = new THREE.PerspectiveCamera(65, window.innerWidth / window.innerHeight, 0.1, 1000);
        camera.position.set(0, 4, 8);
        camera.lookAt(0, 0, -5);

        // Renderer WebGL
        const renderer = new THREE.WebGLRenderer({ antialias: true });
        renderer.setSize(window.innerWidth, window.innerHeight);
        renderer.shadowMap.enabled = true; // bayangan realistis
        renderer.shadowMap.type = THREE.PCFSoftShadowMap;
        document.body.appendChild(renderer.domElement);

        // CSS2DRenderer untuk teks skor
        const labelRenderer = new CSS2DRenderer();
        labelRenderer.setSize(window.innerWidth, window.innerHeight);
        labelRenderer.domElement.style.position = 'absolute';
        labelRenderer.domElement.style.top = '0px';
        labelRenderer.domElement.style.left = '0px';
        labelRenderer.domElement.style.pointerEvents = 'none';
        document.body.appendChild(labelRenderer.domElement);

        // --- Pencahayaan ---
        // Ambient light
        const ambientLight = new THREE.AmbientLight(0x404060);
        scene.add(ambientLight);
        // Directional light (matahari)
        const sunLight = new THREE.DirectionalLight(0xfff5d1, 1.2);
        sunLight.position.set(10, 20, 5);
        sunLight.castShadow = true;
        sunLight.receiveShadow = true;
        sunLight.shadow.mapSize.width = 1024;
        sunLight.shadow.mapSize.height = 1024;
        sunLight.shadow.camera.near = 0.5;
        sunLight.shadow.camera.far = 30;
        sunLight.shadow.camera.left = -10;
        sunLight.shadow.camera.right = 10;
        sunLight.shadow.camera.top = 10;
        sunLight.shadow.camera.bottom = -10;
        scene.add(sunLight);
        
        // Fill light dari bawah (pantulan aspal)
        const fillLight = new THREE.PointLight(0x5577aa, 0.4);
        fillLight.position.set(0, -2, 0);
        scene.add(fillLight);
        
        // Lampu belakang dramatis
        const backLight = new THREE.PointLight(0xff6600, 0.5);
        backLight.position.set(0, 1, -3);
        scene.add(backLight);

        // --- Dekorasi Lantai & Track ---
        // Track utama: bidang aspal dengan garis pembatas
        const trackMat = new THREE.MeshStandardMaterial({ color: 0x2c2e3a, roughness: 0.7, metalness: 0.1 });
        const trackPlane = new THREE.Mesh(new THREE.PlaneGeometry(20, 200), trackMat);
        trackPlane.rotation.x = -Math.PI / 2;
        trackPlane.position.y = -0.2;
        trackPlane.receiveShadow = true;
        scene.add(trackPlane);

        // Garis putih pembatas jalan (kiri dan kanan)
        const lineMat = new THREE.MeshStandardMaterial({ color: 0xdddddd, emissive: 0x222222 });
        const leftBarrier = new THREE.Mesh(new THREE.BoxGeometry(0.3, 0.2, 200), lineMat);
        leftBarrier.position.set(-3.8, 0.05, 0);
        leftBarrier.receiveShadow = true;
        scene.add(leftBarrier);
        
        const rightBarrier = new THREE.Mesh(new THREE.BoxGeometry(0.3, 0.2, 200), lineMat);
        rightBarrier.position.set(3.8, 0.05, 0);
        rightBarrier.receiveShadow = true;
        scene.add(rightBarrier);
        
        // Garis putus-putus di tengah (efek gerak)
        const dashMat = new THREE.MeshStandardMaterial({ color: 0xffdd99, emissive: 0x442200 });
        const dashCount = 40;
        const dashSpacing = 4;
        for (let i = -40; i <= 40; i++) {
            const dash = new THREE.Mesh(new THREE.BoxGeometry(0.5, 0.1, 1.5), dashMat);
            dash.position.set(0, 0.05, i * dashSpacing);
            dash.castShadow = true;
            scene.add(dash);
        }

        // Tambahkan efek lampu track (titik-titik kecil di pinggir)
        const edgeMat = new THREE.MeshStandardMaterial({ color: 0xffaa55, emissive: 0x442200 });
        for (let z = -50; z <= 50; z += 2.5) {
            const leftMarker = new THREE.Mesh(new THREE.SphereGeometry(0.12, 6, 6), edgeMat);
            leftMarker.position.set(-3.5, 0.1, z);
            scene.add(leftMarker);
            
            const rightMarker = new THREE.Mesh(new THREE.SphereGeometry(0.12, 6, 6), edgeMat);
            rightMarker.position.set(3.5, 0.1, z);
            scene.add(rightMarker);
        }

        // --- Mobil Pemain (dengan warna sporty merah) ---
        const carGroup = new THREE.Group();
        
        // Body
        const bodyGeo = new THREE.BoxGeometry(0.9, 0.4, 1.8);
        const bodyMat = new THREE.MeshStandardMaterial({ color: 0xdd3333, roughness: 0.2, metalness: 0.8 });
        const body = new THREE.Mesh(bodyGeo, bodyMat);
        body.castShadow = true;
        body.receiveShadow = true;
        body.position.y = 0.2;
        carGroup.add(body);
        
        // Kaca depan
        const glassGeo = new THREE.BoxGeometry(0.8, 0.25, 0.6);
        const glassMat = new THREE.MeshStandardMaterial({ color: 0x88aaff, metalness: 0.9, roughness: 0.1, emissive: 0x112233 });
        const glass = new THREE.Mesh(glassGeo, glassMat);
        glass.position.set(0, 0.45, -0.3);
        glass.castShadow = true;
        carGroup.add(glass);
        
        // Atap
        const roofGeo = new THREE.BoxGeometry(0.85, 0.2, 1.0);
        const roofMat = new THREE.MeshStandardMaterial({ color: 0xaa2222, metalness: 0.5 });
        const roof = new THREE.Mesh(roofGeo, roofMat);
        roof.position.set(0, 0.55, 0);
        roof.castShadow = true;
        carGroup.add(roof);
        
        // Roda
        const wheelMat = new THREE.MeshStandardMaterial({ color: 0x222222, metalness: 0.7, roughness: 0.4 });
        const wheelGeo = new THREE.CylinderGeometry(0.28, 0.28, 0.4, 24);
        const positions = [[-0.65, 0.1, -0.7], [0.65, 0.1, -0.7], [-0.65, 0.1, 0.7], [0.65, 0.1, 0.7]];
        positions.forEach(pos => {
            const wheel = new THREE.Mesh(wheelGeo, wheelMat);
            wheel.rotation.z = Math.PI / 2;
            wheel.position.set(pos[0], pos[1], pos[2]);
            wheel.castShadow = true;
            carGroup.add(wheel);
        });
        
        // Lampu depan
        const headMat = new THREE.MeshStandardMaterial({ color: 0xffaa66, emissive: 0xff4400 });
        const leftHead = new THREE.Mesh(new THREE.SphereGeometry(0.12, 16, 16), headMat);
        leftHead.position.set(-0.45, 0.2, 0.95);
        const rightHead = new THREE.Mesh(new THREE.SphereGeometry(0.12, 16, 16), headMat);
        rightHead.position.set(0.45, 0.2, 0.95);
        carGroup.add(leftHead);
        carGroup.add(rightHead);
        
        scene.add(carGroup);
        
        // Posisi awal mobil
        carGroup.position.set(0, 0.1, 0);
        
        // --- Variable Game State ---
        let score = 0;
        let speed = 25;        // kecepatan dasar (km/h terasa, skala gerak dunia)
        let baseSpeed = 22;
        let maxSpeed = 58;
        let carX = 0;
        const carLimit = 2.7;   // batas kiri-kanan
        let gameActive = true;
        
        // Elemen UI skor
        const scoreDiv = document.getElementById('speed-value');
        
        // Array untuk rintangan (musuh / kotak powerup)
        let obstacles = [];
        let powerups = [];
        
        // World offset untuk gerakan (track bergerak ke belakang)
        let worldOffset = 0;
        let lastTimestamp = 0;
        
        // --- Helper membuat rintangan (kotak merah yang harus dihindari) ---
        function createObstacle(x, z) {
            const group = new THREE.Group();
            const boxMat = new THREE.MeshStandardMaterial({ color: 0xcc3333, emissive: 0x331100, metalness: 0.2 });
            const box = new THREE.Mesh(new THREE.BoxGeometry(0.9, 0.6, 0.9), boxMat);
            box.castShadow = true;
            box.position.y = 0.3;
            group.add(box);
            
            // paku / spike kecil di atas
            const spikeMat = new THREE.MeshStandardMaterial({ color: 0xaa8866 });
            const spike = new THREE.Mesh(new THREE.ConeGeometry(0.2, 0.3, 6), spikeMat);
            spike.position.y = 0.7;
            group.add(spike);
            
            group.position.set(x, 0, z);
            scene.add(group);
            return group;
        }
        
        function createPowerup(x, z) {
            const group = new THREE.Group();
            const coreMat = new THREE.MeshStandardMaterial({ color: 0xffaa44, emissive: 0xaa4422 });
            const box = new THREE.Mesh(new THREE.BoxGeometry(0.7, 0.7, 0.7), coreMat);
            box.castShadow = true;
            group.add(box);
            
            // efek floating ring
            const ringMat = new THREE.MeshStandardMaterial({ color: 0xffaa66, emissive: 0xff6600 });
            const ring = new THREE.Mesh(new THREE.TorusGeometry(0.5, 0.05, 16, 32), ringMat);
            ring.rotation.x = Math.PI / 2;
            ring.position.y = 0.45;
            group.add(ring);
            
            group.position.set(x, 0.2, z);
            scene.add(group);
            return group;
        }
        
        // --- Spawn objek pada jarak tertentu ---
        function spawnRandomObjects() {
            // spawn rintangan (probabilitas 40% jika jumlah kurang dari 6)
            if (obstacles.length < 6 && Math.random() < 0.022) {
                const randX = (Math.random() - 0.5) * 4.5;
                // batasi di dalam track
                const clampedX = Math.min(Math.max(randX, -3.2), 3.2);
                const zPos = -25 - Math.random() * 15;
                const obs = createObstacle(clampedX, zPos);
                obstacles.push(obs);
            }
            // spawn powerup (orange) untuk koleksi +10 poin
            if (powerups.length < 4 && Math.random() < 0.018) {
                const randX = (Math.random() - 0.5) * 4.2;
                const clampedX = Math.min(Math.max(randX, -3), 3);
                const zPos = -22 - Math.random() * 20;
                const power = createPowerup(clampedX, zPos);
                powerups.push(power);
            }
        }
        
        // --- Update posisi semua objek bergerak berdasarkan kecepatan ---
        function updateWorldMovement(deltaTime) {
            if (!gameActive) return;
            // deltaTime dalam detik, kecepatan gerak dunia: speed * 5 unit per detik (agar responsif)
            const moveDist = speed * deltaTime * 3.2;
            // Gerakkan rintangan
            for (let i = obstacles.length-1; i >= 0; i--) {
                const obs = obstacles[i];
                obs.position.z += moveDist;
                if (obs.position.z > 8) {
                    scene.remove(obs);
                    obstacles.splice(i,1);
                }
            }
            for (let i = powerups.length-1; i >= 0; i--) {
                const pow = powerups[i];
                pow.position.z += moveDist;
                if (pow.position.z > 8) {
                    scene.remove(pow);
                    powerups.splice(i,1);
                }
            }
            
            // Animasi garis tengah (agar efek berjalan) - rotasi sederhana tidak perlu, cukup pindah? tidak perlu karena sudah infinite
            // tetapi kita bisa geser marker garis dengan material offset? skip
            
            // Spawn objek baru
            spawnRandomObjects();
        }
        
        // --- Deteksi Tabrakan (bounding box sederhana) ---
        function checkCollisions() {
            if (!gameActive) return false;
            
            const carBox = new THREE.Box3().setFromObject(carGroup);
            
            // Cek rintangan (merah)
            for (let i=0; i<obstacles.length; i++) {
                const obsBox = new THREE.Box3().setFromObject(obstacles[i]);
                if (carBox.intersectsBox(obsBox)) {
                    gameActive = false;
                    // efek crash: ubah warna mobil dan tampilkan pesan
                    body.material.color.setHex(0x884444);
                    showGameOverMessage();
                    return true;
                }
            }
            
            // Cek powerup
            for (let i=0; i<powerups.length; i++) {
                const powBox = new THREE.Box3().setFromObject(powerups[i]);
                if (carBox.intersectsBox(powBox)) {
                    // ambil powerup
                    scene.remove(powerups[i]);
                    powerups.splice(i,1);
                    score += 10;
                    updateScoreUI();
                    // efek percepatan kecil
                    speed = Math.min(maxSpeed, speed + 5);
                    // efek visual kilat
                    document.body.style.backgroundColor = "rgba(255,200,100,0.2)";
                    setTimeout(() => { document.body.style.backgroundColor = ""; }, 100);
                    break;
                }
            }
            return false;
        }
        
        function updateScoreUI() {
            scoreDiv.innerText = Math.floor(speed * 1.8 + score * 0.5);
            // Tampilkan speed dalam km/h (stylized)
            document.getElementById('speed-value').innerText = Math.floor(speed * 3.2);
        }
        
        function showGameOverMessage() {
            const msgDiv = document.createElement('div');
            msgDiv.id = 'gameover-msg';
            msgDiv.innerText = '💥 GAME OVER 💥\nTekan R untuk restart';
            msgDiv.style.position = 'absolute';
            msgDiv.style.top = '45%';
            msgDiv.style.left = '50%';
            msgDiv.style.transform = 'translate(-50%, -50%)';
            msgDiv.style.backgroundColor = 'black';
            msgDiv.style.color = 'red';
            msgDiv.style.fontSize = '32px';
            msgDiv.style.fontWeight = 'bold';
            msgDiv.style.padding = '20px 40px';
            msgDiv.style.borderRadius = '20px';
            msgDiv.style.border = '3px solid red';
            msgDiv.style.fontFamily = 'monospace';
            msgDiv.style.zIndex = '100';
            msgDiv.style.textAlign = 'center';
            msgDiv.style.backdropFilter = 'blur(8px)';
            document.body.appendChild(msgDiv);
        }
        
        function removeGameOverMessage() {
            const existing = document.getElementById('gameover-msg');
            if (existing) existing.remove();
        }
        
        function restartGame() {
            // Hapus semua objek dinamis
            obstacles.forEach(obs => scene.remove(obs));
            powerups.forEach(pow => scene.remove(pow));
            obstacles = [];
            powerups = [];
            
            // Reset posisi mobil
            carGroup.position.set(0, 0.1, 0);
            carX = 0;
            speed = 25;
            score = 0;
            gameActive = true;
            updateScoreUI();
            body.material.color.setHex(0xdd3333);
            removeGameOverMessage();
            
            // Spawn beberapa objek awal
            for (let i=0; i<3; i++) {
                spawnRandomObjects();
            }
        }
        
        // --- Input Keyboard (mobil kiri/kanan) ---
        const keyState = { ArrowLeft: false, ArrowRight: false };
        window.addEventListener('keydown', (e) => {
            if (e.key === 'ArrowLeft') keyState.ArrowLeft = true;
            if (e.key === 'ArrowRight') keyState.ArrowRight = true;
            if (e.key === 'r' || e.key === 'R') {
                if (!gameActive) restartGame();
            }
            // Mencegah scroll halaman
            if (e.key === 'ArrowLeft' || e.key === 'ArrowRight') e.preventDefault();
        });
        window.addEventListener('keyup', (e) => {
            if (e.key === 'ArrowLeft') keyState.ArrowLeft = false;
            if (e.key === 'ArrowRight') keyState.ArrowRight = false;
        });
        
        // --- Update Kecepatan berdasarkan waktu, geser mobil, dan kamera mengikuti ---
        let lastTime = performance.now();
        
        function animateGame(now) {
            let delta = Math.min(0.033, (now - lastTime) / 1000);
            lastTime = now;
            if (delta <= 0) delta = 0.016;
            
            if (gameActive) {
                // Kontrol mobil horizontal
                let move = 0;
                if (keyState.ArrowLeft) move = -1;
                if (keyState.ArrowRight) move = 1;
                carX += move * 7.5 * delta;
                carX = Math.min(carLimit, Math.max(-carLimit, carX));
                carGroup.position.x = carX;
                
                // Efek kemiringan (roll) saat belok
                const tilt = carX * 0.25;
                carGroup.rotation.z = tilt * 0.3;
                
                // Penyesuaian kecepatan: semakin banyak skor semakin cepat, tapi ada batas
                speed = baseSpeed + (score / 35);
                speed = Math.min(maxSpeed, Math.max(18, speed));
                if (score > 150) speed = maxSpeed;
                
                // Update dunia bergerak
                updateWorldMovement(delta);
                
                // Deteksi tabrakan setelah pergerakan
                const crashed = checkCollisions();
                if (crashed) {
                    gameActive = false;
                    showGameOverMessage();
                }
                
                // Update UI kecepatan secara realtime
                document.getElementById('speed-value').innerText = Math.floor(speed * 3.2);
                
                // Efek lampu belakang berkedip berdasarkan kecepatan
                const intensity = 0.4 + (speed / maxSpeed) * 0.8;
                backLight.intensity = intensity;
            }
            
            // Update posisi kamera mengikuti mobil dengan efek lag halus
            const targetCamX = carGroup.position.x * 0.5;
            const targetCamZ = 6.5;
            camera.position.x += (targetCamX - camera.position.x) * 0.1;
            camera.position.z = targetCamZ;
            camera.position.y = 3.2 + Math.abs(carGroup.position.x) * 0.2;
            camera.lookAt(carGroup.position.x, 0.2, carGroup.position.z - 3);
            
            // Animasi sederhana untuk powerup (berputar)
            powerups.forEach(p => {
                if (p.children[1]) p.children[1].rotation.y += 0.03;
                p.children[0].rotation.y += 0.02;
            });
            obstacles.forEach(o => {
                if (o.children[1]) o.children[1].rotation.y += 0.01;
            });
            
            // Render scene
            renderer.render(scene, camera);
            labelRenderer.render(scene, camera);
            requestAnimationFrame(animateGame);
        }
        
        // --- Efek ambient / partikel debu kecil di track ---
        const dustGeometry = new THREE.BufferGeometry();
        const dustCount = 800;
        const dustPositions = new Float32Array(dustCount * 3);
        for (let i = 0; i < dustCount; i++) {
            dustPositions[i*3] = (Math.random() - 0.5) * 25;
            dustPositions[i*3+1] = -0.1;
            dustPositions[i*3+2] = (Math.random() - 0.5) * 120;
        }
        dustGeometry.setAttribute('position', new THREE.BufferAttribute(dustPositions, 3));
        const dustMat = new THREE.PointsMaterial({ color: 0xaa9977, size: 0.08 });
        const dustSystem = new THREE.Points(dustGeometry, dustMat);
        scene.add(dustSystem);
        
        // Animasi debu bergerak mundur
        function animateDust() {
            if (!gameActive) return;
            const positions = dustGeometry.attributes.position.array;
            for (let i = 0; i < dustCount; i++) {
                positions[i*3+2] += speed * 0.05;
                if (positions[i*3+2] > 20) positions[i*3+2] = -60;
            }
            dustGeometry.attributes.position.needsUpdate = true;
            requestAnimationFrame(animateDust);
        }
        animateDust();
        
        // Generate beberapa objek awal
        for (let i = 0; i < 5; i++) {
            setTimeout(() => spawnRandomObjects(), i * 300);
        }
        
        // Menambahkan skybox sederhana (gradient sky via scene bg sudah cukup)
        // Tambahkan lampu kota di pinggir (efek)
        const poleMat = new THREE.MeshStandardMaterial({ color: 0xaa9966 });
        for (let z = -40; z <= 40; z += 6) {
            const pole = new THREE.Mesh(new THREE.CylinderGeometry(0.15, 0.25, 1.8, 6), poleMat);
            pole.position.set(-4.5, 0.9, z);
            pole.castShadow = true;
            scene.add(pole);
            const poleR = new THREE.Mesh(new THREE.CylinderGeometry(0.15, 0.25, 1.8, 6), poleMat);
            poleR.position.set(4.5, 0.9, z);
            poleR.castShadow = true;
            scene.add(poleR);
            
            const lampMat = new THREE.MeshStandardMaterial({ color: 0xffaa66, emissive: 0x442200 });
            const lamp = new THREE.Mesh(new THREE.SphereGeometry(0.22, 8, 8), lampMat);
            lamp.position.set(-4.5, 1.7, z);
            scene.add(lamp);
            const lampR = new THREE.Mesh(new THREE.SphereGeometry(0.22, 8, 8), lampMat);
            lampR.position.set(4.5, 1.7, z);
            scene.add(lampR);
        }
        
        // Jalankan game loop
        lastTime = performance.now();
        requestAnimationFrame(animateGame);
        
        // Handle resize window
        window.addEventListener('resize', onWindowResize, false);
        function onWindowResize() {
            camera.aspect = window.innerWidth / window.innerHeight;
            camera.updateProjectionMatrix();
            renderer.setSize(window.innerWidth, window.innerHeight);
            labelRenderer.setSize(window.innerWidth, window.innerHeight);
        }
        
        // Petunjuk kecil di console
        console.log('Game Balapan 3D siap! Gunakan panah kiri/kanan, kumpulkan kotak orange, hindari merah.');
    </script>
</body>
</html>
```# balapan-3d
