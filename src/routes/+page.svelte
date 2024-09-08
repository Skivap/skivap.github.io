<script lang="ts">
    import { browser } from '$app/environment'
    import * as THREE from "three"
    import { OBJLoader } from 'three/addons/loaders/OBJLoader.js';
    import { OrbitControls } from 'three/addons/controls/OrbitControls.js';
    import Stats from 'three/examples/jsm/libs/stats.module.js';
    
    if (browser) {

        const model3D = document.getElementById('3dmodel')

        // const stats = new Stats()
        // stats.showPanel(0) 
        
        // if(model3D){
        //     model3D.appendChild(stats.dom)
        // }

        const renderer = new THREE.WebGLRenderer()
        renderer.setSize(700, 700)

        if(model3D){
            model3D.appendChild(renderer.domElement)
        }
        else{
            console.log("model not found")
        }
        
        renderer.setClearColor( 0xffffff, 0);

        const scene = new THREE.Scene()
        const camera = new THREE.PerspectiveCamera(
            75, 1, 0.1, 1000
        )
        camera.position.set(0, 0, 2.2)
        camera.position.y = 2.2 * Math.cos(1.05);

        // const orbit = new OrbitControls(camera, renderer.domElement)
        // orbit.enableZoom = false
        // orbit.enablePan = false
        // orbit.enableRotate = true

        // ====================== ====================== ====================== ======================
        // GRADUATION HAT

        let _gradHatVertices: number[] = [];
        let gradHatVertices: Float32Array;

        const loader = new OBJLoader(); 
        loader.load( 
            'assets/Grat.obj', 
            function (object) {
                try {
                    object.traverse((child) => {
                        if (child instanceof THREE.Mesh) {
                            const positionAttribute = child.geometry.attributes.position;
                            
                            if (positionAttribute && positionAttribute.array) {
                                const positions = positionAttribute.array;
                                for (let i = 0; i < positions.length; i++) {
                                    _gradHatVertices.push(positions[i]);
                                }
                            } else {
                                console.error('Position attribute is missing or malformed:', child);
                            }
                            console.log('Vertices:', _gradHatVertices.length);
                        }
                    });

                    // Log after the OBJ is fully loaded and processed
                    console.log('Vertices collected:', _gradHatVertices.length);
                    try{
                        let filt = new Array(_gradHatVertices.length)
                        for(let i=0; i<_gradHatVertices.length; i+=3){
                            filt[i] = filt[i+1] = filt[i+2] = Math.random()
                        }
                        _gradHatVertices = _gradHatVertices.filter((_, idx) => filt[idx] < 0.02);
                        console.log('After reduceds:', _gradHatVertices.length);
                    }
                    catch (error) {
                        console.log('Error as:', error)
                    }
                    
                    // const geometry = new THREE.BufferGeometry();
                    gradHatVertices = new Float32Array(_gradHatVertices);
                    // geometry.setAttribute('position', new THREE.Float32BufferAttribute(verticesArray, 3));
                    // const material = new THREE.PointsMaterial({
                    //     size: 0.01, // Adjust size as needed
                    //     color: 0x000000, // Color of the points
                    //     transparent: true,
                    //     opacity: 0.9,
                    // });

                    // const points = new THREE.Points(geometry, material);
                    // scene.add(points);
                    update()
                    
                } catch (error) {
                    console.error('Error processing OBJ file:', error);
                }
            },
            function ( xhr ) { console.log( ( xhr.loaded / xhr.total * 100 ) + '% loaded' ); }, 
            function ( error ) { console.log( 'An error happened' ); } 
        );

        // ====================== ====================== ====================== ======================
        // SPHERE

        function update() {
            const _SphereGeometry = new THREE.SphereGeometry(1, 256, 256)
            _spherePosition = Array.from(_SphereGeometry.attributes.position.array);
            console.log(_spherePosition.length)

            while(_spherePosition.length > gradHatVertices.length){
                let idx = Math.floor(Math.random() * _spherePosition.length / 3) * 3
                _spherePosition.splice(idx, 3);
            }

            originalPosition = _spherePosition

            console.log(_spherePosition.length)

            const verticesArray = new Float32Array(_spherePosition);
            geometry.setAttribute('position', new THREE.Float32BufferAttribute(verticesArray, 3));

            const sphere = new THREE.Points(geometry, _SphereMaterial)
            scene.add(sphere)
        }

        // ====================== ====================== ====================== ======================
        // Main

        const geometry = new THREE.BufferGeometry();
        let _spherePosition = []
        let originalPosition = []

        const _SphereMaterial = new THREE.PointsMaterial({
            size: 0.01,
            color: 0x000000,
            transparent: true,
            opacity: 0.9
        })

        // ====================== ====================== ====================== ======================
        // Animation
        function lerp(start, end, t) {
            return start + (end - start) * t;
        }

        let elapsed = 0;

        function animateToTarget(startPositions, endPositions, duration) {
            if (elapsed > duration) return;
            elapsed += 1 / 60; // Assuming 60 FPS
            const t = Math.min(elapsed / duration, 1);

            // Interpolate each vertex position from start to end using the lerp function
            const interpolatedPositions = startPositions.map((start, index) => {
                return lerp(start, endPositions[index], t);
            });

            // Update geometry with new interpolated positions
            const verticesArray = new Float32Array(interpolatedPositions);
            geometry.setAttribute('position', new THREE.Float32BufferAttribute(verticesArray, 3));
            geometry.attributes.position.needsUpdate = true;

            // Continue the animation if not yet complete
            if (t < 1) {
                requestAnimationFrame(() => animateToTarget(startPositions, endPositions, duration));
            }
            else{
                setTimeout(() => {
                    flag = 2
                }, 1000);
            }
        }

        let flag = 0;

        function change() { 
            if(flag == 0){
                try{
                    let ori = originalPosition

                    for(let i=0; i<_spherePosition.length; i++){
                        _spherePosition[i] += Math.random() * 0.01 - 0.005
                        _spherePosition[i] = Math.max(_spherePosition[i], ori[i] - 0.1)
                        _spherePosition[i] = Math.min(_spherePosition[i], ori[i] + 0.1)
                    }
                    const verticesArray = new Float32Array(_spherePosition);
                    geometry.setAttribute('position', new THREE.Float32BufferAttribute(verticesArray, 3));
                }
                catch(error) {
                    console.log("error as", error)
                }
            }
            else if(flag == 2){
                // try{
                //     let ori = gradHatVertices

                //     for(let i=0; i<_spherePosition.length; i++){
                //         _spherePosition[i] += Math.random() * 0.005 - 0.0025
                //         _spherePosition[i] = Math.max(_spherePosition[i], ori[i] - 0.1)
                //         _spherePosition[i] = Math.min(_spherePosition[i], ori[i] + 0.1)
                //     }
                //     const verticesArray = new Float32Array(_spherePosition);
                //     geometry.setAttribute('position', new THREE.Float32BufferAttribute(verticesArray, 3));
                // }
                // catch(error) {
                //     console.log("error as", error)
                // }
            }
            else{
                animateToTarget(_spherePosition, gradHatVertices, 5);
            }
        }

        function animate() {
            // stats.begin()
            
            change()
            requestAnimationFrame(animate);
            // orbit.update()
            camera.position.x = 2.2 * Math.cos(Date.now() * 0.0001)
            camera.position.z = 2.2 * Math.sin(Date.now() * 0.0001)
            camera.lookAt(0, 0, 0)
            renderer.render(scene, camera);

            // stats.end()
        }

        setTimeout(() => {
            flag = 1
        }, 5000);

        animate();
    }
</script>

<style>
    :global(html, body) {
        margin: 0;
        height: 100%;
    }

    .layout {
        display: flex;
        height: 100%;
    }
    
    .mycanvas {
        display: flex;
        justify-content: left;
        align-items: center;    
        width: 0;
        width: 0%;
    }

    .content{
        max-width: 60%;
        right: 0;
        position: absolute;
        display: block;
        justify-content: left;
        align-items: center;
        padding: 100px;
        background-color: rgba(127, 127, 127,0.5);
        color: white;
        height: 100%;
    }

    
</style>

<svelte:head>
    <title>ae</title>
</svelte:head>

<div class="layout">
    <div class ="mycanvas">
        <div id="3dmodel">

        </div>
    </div>
    <div class="content">
        <h1>
            Hi, my name is Aurick
        </h1>
        <h2>
            I'm interested in Computer Graphics and Deep Learning, especially in Computer Vision
        </h2>
        <nav>
            <a href="">
                Experience
            </a>
        </nav>
    </div>
    
</div>

