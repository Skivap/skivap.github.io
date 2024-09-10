<script lang="ts">
    import { browser } from '$app/environment'
    import * as THREE from "three"
    import { OBJLoader } from 'three/addons/loaders/OBJLoader.js';
    import { OrbitControls } from 'three/addons/controls/OrbitControls.js';
    import Stats from 'three/examples/jsm/libs/stats.module.js';

    let instaVertices: Float32Array;
    let facebookVertices: Float32Array;
    let githubVertices: Float32Array;
    const geometry = new THREE.BufferGeometry();
    let _spherePosition: number[] = []
    let originalPosition: Float32Array;

    let flag = 0;

    if (browser) {

        const model3D = document.getElementById('3dmodel')

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

        // ====================== ====================== ====================== ======================
        // INSTA

        let _instaVertices: number[] = [];

        const loader = new OBJLoader(); 
        loader.load( 
            'assets/in.obj', 
            function (object) {
                try {
                    object.traverse((child) => {
                        if (child instanceof THREE.Mesh) {
                            const positionAttribute = child.geometry.attributes.position;
                            if (positionAttribute && positionAttribute.array) {
                                const positions = positionAttribute.array;
                                for (let i = 0; i < positions.length; i++) {
                                    _instaVertices.push(positions[i]);
                                }
                            } else {
                                console.error('Position attribute is missing or malformed:', child);
                            }
                            console.log('Vertices:', _instaVertices.length);
                        }
                        else{
                            console.log("noo");
                        }
                    });

                    while(_instaVertices.length < 18000){
                        _instaVertices.push(0);
                    }

                    instaVertices = new Float32Array(_instaVertices);

                    update()
                    
                } catch (error) {
                    console.error('Error processing OBJ file:', error);
                }
            },
            function ( xhr ) { console.log( ( xhr.loaded / xhr.total * 100 ) + '% loaded' ); }, 
            function ( error ) { console.log( 'An error happened' ); } 
        );

        // ====================== ====================== ====================== ======================
        // FB

        let _facebookVertices: number[] = [];

        const loader2 = new OBJLoader(); 
        loader2.load( 
            'assets/facebook.obj', 
            function (object) {
                try {
                    object.traverse((child) => {
                        if (child instanceof THREE.Mesh) {
                            const positionAttribute = child.geometry.attributes.position;
                            if (positionAttribute && positionAttribute.array) {
                                const positions = positionAttribute.array;
                                for (let i = 0; i < positions.length; i++) {
                                    _facebookVertices.push(positions[i]);
                                }
                            } else {
                                console.error('Position attribute is missing or malformed:', child);
                            }
                            console.log('Vertices:', _facebookVertices.length);
                        }
                        else{
                            console.log("noo");
                        }
                    });

                    while(_facebookVertices.length < 18000){
                        _facebookVertices.push(0);
                    }

                    facebookVertices = new Float32Array(_facebookVertices);

                    update()
                    
                } catch (error) {
                    console.error('Error processing OBJ file:', error);
                }
            },
            function ( xhr ) { console.log( ( xhr.loaded / xhr.total * 100 ) + '% loaded' ); }, 
            function ( error ) { console.log( 'An error happened' ); } 
        );

        // ====================== ====================== ====================== ======================
        // GIT

        let _githubVertices: number[] = [];

        const loader3 = new OBJLoader(); 
        loader3.load( 
            'assets/git.obj', 
            function (object) {
                try {
                    object.traverse((child) => {
                        if (child instanceof THREE.Mesh) {
                            const positionAttribute = child.geometry.attributes.position;
                            if (positionAttribute && positionAttribute.array) {
                                const positions = positionAttribute.array;
                                for (let i = 0; i < positions.length; i++) {
                                    _githubVertices.push(positions[i]);
                                }
                            } else {
                                console.error('Position attribute is missing or malformed:', child);
                            }
                            console.log('Vertices:', _githubVertices.length);
                        }
                        else{
                            console.log("noo");
                        }
                    });

                    while(_githubVertices.length < 18000){
                        _githubVertices.push(0);
                    }

                    githubVertices = new Float32Array(_githubVertices);

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
            const _SphereGeometry = new THREE.SphereGeometry(1, 80, 80)
            _spherePosition = Array.from(_SphereGeometry.attributes.position.array);
            console.log(_spherePosition.length)

            while(_spherePosition.length > 18000){
                let idx = Math.floor(Math.random() * _spherePosition.length / 3) * 3
                _spherePosition.splice(idx, 3);
            }

            originalPosition = new Float32Array(_spherePosition)

            console.log(_spherePosition.length)

            const verticesArray = new Float32Array(_spherePosition);
            geometry.setAttribute('position', new THREE.Float32BufferAttribute(verticesArray, 3));

            const sphere = new THREE.Points(geometry, _SphereMaterial)
            scene.add(sphere)
        }

        // ====================== ====================== ====================== ======================
        // Main

        const _SphereMaterial = new THREE.PointsMaterial({
            size: 0.02,
            color: 0x000000,
            transparent: true,
            opacity: 0.9
        })

        // ====================== ====================== ====================== ======================
        // Animation

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

        animate();
    }

    function lerp(start:number, end:number, t:number) {
        return start + (end - start) * t;
    }

    let elapsed = 0;

    function animateToTarget(startPositions:number[], endPositions:Float32Array, duration:number) {
        if (elapsed > duration) return;
        elapsed += 1 / 60;
        const t = Math.min(elapsed / duration, 1);

        const interpolatedPositions = startPositions.map((start, index) => {
            return lerp(start, endPositions[index], t);
        });

        _spherePosition = interpolatedPositions

        const verticesArray = new Float32Array(interpolatedPositions);
        geometry.setAttribute('position', new THREE.Float32BufferAttribute(verticesArray, 3));
        geometry.attributes.position.needsUpdate = true;

        if (t < 1) {
            requestAnimationFrame(() => animateToTarget(startPositions, endPositions, duration));
        }
    }

    function switch_flag(num: number){
        elapsed=0;
        console.log(num)
        flag=num;
        change()
    }

    function change() { 
        if(flag == 0){
            // try{
            //     let ori = originalPosition

            //     for(let i=0; i<_spherePosition.length; i++){
            //         _spherePosition[i] += Math.random() * 0.01 - 0.005
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
        else if(flag == 1){ // Switch to Insta
            animateToTarget(_spherePosition, instaVertices, 100);
        }
        else if(flag == 2){ // Switch to Facebook
            animateToTarget(_spherePosition, facebookVertices, 100);
        }
        else if(flag == 3){ // Switch to Github
            animateToTarget(_spherePosition, githubVertices, 100);
        }
        else if(flag == 4){
            animateToTarget(_spherePosition, originalPosition, 100);
        }
    }
</script>

<style>
    :global(html, body) {
        margin: 0;
        height: 100%;
    }

    :global(h1) {
        font-size: 50px;
    }

    .layout {
        display: flex;
        height: 100%;
    }
    
    .mycanvas {
        position: absolute;
        display: flex;
        justify-content: center;
        align-items: center;    
        width: 100%;
        height: 100%;
    }

    .content{
        max-width: 40%;
        display: block;
        justify-content: left;
        align-items: center;
        color: black;
        height: 100%;
    }

    .outfit {
        font-family: "Outfit", sans-serif;
        font-optical-sizing: auto;
        font-weight: 800;
        font-style: normal;
    }

    .inside-content {
        margin-top: 80px;
        margin-left: 80px;
    }

    .contact{
        position: absolute;
        bottom: 150px;
    }
    
    .icon-list{
        display: flex;
        margin-top: 20px;
    }

    .icon-list svg{
        height: 60px;
    }
    
    .hover-element{
        cursor: pointer;
    }
    
</style>

<svelte:head>
    <title>ae</title>
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossOrigin="true">
    <link href="https://fonts.googleapis.com/css2?family=Outfit:wght@100..900&display=swap" rel="stylesheet">
</svelte:head>
<div class ="mycanvas">
    <div id="3dmodel">

    </div>
</div>
<div class="layout">
    <div class="content">
        <div class="inside-content">
            <h1 class="outfit">
                Hi, my name is Aurick
            </h1>
            <h2 class="outfit">
                I'm interested in Computer Graphics and Deep Learning, especially in Computer Vision
            </h2>
            <div class="contact">
                <h2 class="outfit">
                    Contact me: aurickd@gmail.com
                </h2>
                <div class="icon-list">
                    <div class="hover-element"
                    on:mouseover={() => switch_flag(1)} 
                    on:mouseout={() => switch_flag(4)}>
                        <svg xmlns="http://www.w3.org/2000/svg" x="0px" y="0px" width="100" height="100" viewBox="0 0 30 30">
                            <path d="M 9.9980469 3 C 6.1390469 3 3 6.1419531 3 10.001953 L 3 20.001953 C 3 23.860953 6.1419531 27 10.001953 27 L 20.001953 27 C 23.860953 27 27 23.858047 27 19.998047 L 27 9.9980469 C 27 6.1390469 23.858047 3 19.998047 3 L 9.9980469 3 z M 22 7 C 22.552 7 23 7.448 23 8 C 23 8.552 22.552 9 22 9 C 21.448 9 21 8.552 21 8 C 21 7.448 21.448 7 22 7 z M 15 9 C 18.309 9 21 11.691 21 15 C 21 18.309 18.309 21 15 21 C 11.691 21 9 18.309 9 15 C 9 11.691 11.691 9 15 9 z M 15 11 A 4 4 0 0 0 11 15 A 4 4 0 0 0 15 19 A 4 4 0 0 0 19 15 A 4 4 0 0 0 15 11 z"></path>
                        </svg>
                    </div>
                    
                    <div class="hover-element"
                    on:mouseover={() => switch_flag(2)} 
                    on:mouseout={() => switch_flag(4)}>
                        <svg class="hover-element" xmlns="http://www.w3.org/2000/svg" x="0px" y="0px" width="100" height="100" viewBox="0 0 50 50">
                            <path d="M41,4H9C6.24,4,4,6.24,4,9v32c0,2.76,2.24,5,5,5h32c2.76,0,5-2.24,5-5V9C46,6.24,43.76,4,41,4z M37,19h-2c-2.14,0-3,0.5-3,2 v3h5l-1,5h-4v15h-5V29h-4v-5h4v-3c0-4,2-7,6-7c2.9,0,4,1,4,1V19z"></path>
                        </svg>
                    </div>

                    <div class="hover-element"
                    on:mouseover={() => switch_flag(3)} 
                    on:mouseout={() => switch_flag(4)}>
                        <svg class="hover-element" xmlns="http://www.w3.org/2000/svg" x="0px" y="0px" width="100" height="100" viewBox="0 0 50 50">
                            <path d="M17.791,46.836C18.502,46.53,19,45.823,19,45v-5.4c0-0.197,0.016-0.402,0.041-0.61C19.027,38.994,19.014,38.997,19,39 c0,0-3,0-3.6,0c-1.5,0-2.8-0.6-3.4-1.8c-0.7-1.3-1-3.5-2.8-4.7C8.9,32.3,9.1,32,9.7,32c0.6,0.1,1.9,0.9,2.7,2c0.9,1.1,1.8,2,3.4,2 c2.487,0,3.82-0.125,4.622-0.555C21.356,34.056,22.649,33,24,33v-0.025c-5.668-0.182-9.289-2.066-10.975-4.975 c-3.665,0.042-6.856,0.405-8.677,0.707c-0.058-0.327-0.108-0.656-0.151-0.987c1.797-0.296,4.843-0.647,8.345-0.714 c-0.112-0.276-0.209-0.559-0.291-0.849c-3.511-0.178-6.541-0.039-8.187,0.097c-0.02-0.332-0.047-0.663-0.051-0.999 c1.649-0.135,4.597-0.27,8.018-0.111c-0.079-0.5-0.13-1.011-0.13-1.543c0-1.7,0.6-3.5,1.7-5c-0.5-1.7-1.2-5.3,0.2-6.6 c2.7,0,4.6,1.3,5.5,2.1C21,13.4,22.9,13,25,13s4,0.4,5.6,1.1c0.9-0.8,2.8-2.1,5.5-2.1c1.5,1.4,0.7,5,0.2,6.6c1.1,1.5,1.7,3.2,1.6,5 c0,0.484-0.045,0.951-0.11,1.409c3.499-0.172,6.527-0.034,8.204,0.102c-0.002,0.337-0.033,0.666-0.051,0.999 c-1.671-0.138-4.775-0.28-8.359-0.089c-0.089,0.336-0.197,0.663-0.325,0.98c3.546,0.046,6.665,0.389,8.548,0.689 c-0.043,0.332-0.093,0.661-0.151,0.987c-1.912-0.306-5.171-0.664-8.879-0.682C35.112,30.873,31.557,32.75,26,32.969V33 c2.6,0,5,3.9,5,6.6V45c0,0.823,0.498,1.53,1.209,1.836C41.37,43.804,48,35.164,48,25C48,12.318,37.683,2,25,2S2,12.318,2,25 C2,35.164,8.63,43.804,17.791,46.836z"></path>
                        </svg>
                    </div>

                    <br>
                </div>
            </div>

            
            <!-- <nav>
                <a href="">
                    Experience
                </a>
            </nav> -->
        </div>
    </div>
    
</div>

