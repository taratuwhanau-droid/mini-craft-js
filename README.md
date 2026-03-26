# mini-craft-js
Minecraft 2.0
<!DOCTYPE html>
<html>
<head>
    <title>Mini-Craft JS</title>
    <style>
        body { margin: 0; overflow: hidden; font-family: sans-serif; }
        #crosshair {
            position: absolute; top: 50%; left: 50%;
            width: 20px; height: 20px;
            border: 2px solid white; border-radius: 50%;
            transform: translate(-50%, -50%);
            pointer-events: none;
        }
        #instructions {
            position: absolute; top: 10px; left: 10px;
            color: white; background: rgba(0,0,0,0.5); padding: 10px;
        }
    </style>
</head>
<body>
    <div id="crosshair"></div>
    <div id="instructions">Click to Play | WASD to Move | Space to Jump</div>
    
    <script type="module">
        import * as THREE from 'https://cdn.skypack.dev/three@0.136.0';
        import { PointerLockControls } from 'https://cdn.skypack.dev/three@0.136.0/examples/jsm/controls/PointerLockControls.js';

        // 1. Scene Setup
        const scene = new THREE.Scene();
        scene.background = new THREE.Color(0x87CEEB); // Sky Blue
        scene.fog = new THREE.FogExp2(0x87CEEB, 0.02);

        const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
        const renderer = new THREE.WebGLRenderer({ antialias: false }); // Pixelated feel
        renderer.setSize(window.innerWidth, window.innerHeight);
        document.body.appendChild(renderer.domElement);

        // 2. Lights
        const ambientLight = new THREE.AmbientLight(0xcccccc, 0.5);
        scene.add(ambientLight);
        const directionalLight = new THREE.DirectionalLight(0xffffff, 0.8);
        directionalLight.position.set(1, 1, 0.5).normalize();
        scene.add(directionalLight);

        // 3. Controls
        const controls = new PointerLockControls(camera, document.body);
        document.body.addEventListener('click', () => controls.lock());

        // 4. World Generation (Simple 20x20 Grass Plane)
        const boxGeo = new THREE.BoxGeometry(1, 1, 1);
        const grassMat = new THREE.MeshLambertMaterial({ color: 0x44aa44 });
        const dirtMat = new THREE.MeshLambertMaterial({ color: 0x8b4513 });

        for (let x = -10; x < 10; x++) {
            for (let z = -10; z < 10; z++) {
                const mesh = new THREE.Mesh(boxGeo, grassMat);
                mesh.position.set(x, 0, z);
                scene.add(mesh);
                
                // Add some random "hills"
                if (Math.random() > 0.9) {
                    const tree = new THREE.Mesh(boxGeo, dirtMat);
                    tree.position.set(x, 1, z);
                    scene.add(tree);
                }
            }
        }

        // 5. Movement Logic
        let moveForward = false, moveBackward = false, moveLeft = false, moveRight = false, canJump = false;
        const velocity = new THREE.Vector3();
        const direction = new THREE.Vector3();

        const onKeyDown = (e) => {
            switch (e.code) {
                case 'KeyW': moveForward = true; break;
                case 'KeyS': moveBackward = true; break;
                case 'KeyA': moveLeft = true; break;
                case 'KeyD': moveRight = true; break;
                case 'Space': if (canJump) velocity.y += 5; canJump = false; break;
            }
        };
        const onKeyUp = (e) => {
            switch (e.code) {
                case 'KeyW': moveForward = false; break;
                case 'KeyS': moveBackward = false; break;
                case 'KeyA': moveLeft = false; break;
                case 'KeyD': moveRight = false; break;
            }
        };
        document.addEventListener('keydown', onKeyDown);
        document.addEventListener('keyup', onKeyUp);

        // 6. Game Loop
        let prevTime = performance.now();
        function animate() {
            requestAnimationFrame(animate);
            const time = performance.now();
            const delta = (time - prevTime) / 1000;

            if (controls.isLocked) {
                // Gravity & Friction
                velocity.x -= velocity.x * 10.0 * delta;
                velocity.z -= velocity.z * 10.0 * delta;
                velocity.y -= 9.8 * delta; // Gravity

                direction.z = Number(moveForward) - Number(moveBackward);
                direction.x = Number(moveRight) - Number(moveLeft);
                direction.normalize();

                if (moveForward || moveBackward) velocity.z -= direction.z * 100.0 * delta;
                if (moveLeft || moveRight) velocity.x -= direction.x * 100.0 * delta;

                controls.moveRight(-velocity.x * delta);
                controls.moveForward(-velocity.z * delta);
                controls.getObject().position.y += (velocity.y * delta);

                // Floor Collision
                if (controls.getObject().position.y < 2) {
                    velocity.y = 0;
                    controls.getObject().position.y = 2;
                    canJump = true;
                }
            }
            renderer.render(scene, camera);
            prevTime = time;
        }
        animate();
    </script>
</body>
</html>
