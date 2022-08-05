<template>
    <div ref="canvasBox"></div>
</template>

<script setup>
import { ref, onMounted } from "vue";
import * as THREE from "three";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls";
import gsap from "gsap";
import { GUI } from "dat.gui";

const canvasBox = ref(null);

let windowWidth = window.innerWidth;
let windowHeight = window.innerHeight;
let aspectRatio = windowWidth / windowHeight;

const factor = 130;
const near = 1;
const far = 10000;

const data = {
    duration: 1.5,
    delay: 0.5,
    colors: {
        background: 0xffad19,
        ground: 0xf99d17,
        cube: 0xe0e0e0,
        plateform: 0x9e9e9e,
    },
};

let scene,
    camera,
    renderer,
    cube,
    ambientlight,
    hemisphereLight,
    directionalLight;

const addScene = () => {
    scene = new THREE.Scene();
    // scene.background = new THREE.Color(0xffad19);
};
const addCamera = () => {
    camera = new THREE.OrthographicCamera(
        windowWidth / -factor,
        windowWidth / factor,
        windowHeight / factor,
        windowHeight / -factor,
        near,
        far
    );
    camera.position.set(7, 5, 5);
    camera.lookAt(scene.position);
};
const addRenderer = () => {
    renderer = new THREE.WebGLRenderer({
        antialias: true,
    });
    renderer.setClearColor(new THREE.Color(data.colors.background), 1);
    renderer.shadowMap.enabled = true;
    renderer.shadowMap.type = THREE.PCFSoftShadowMap;
    renderer.setSize(windowWidth, windowHeight);
    canvasBox.value.appendChild(renderer.domElement);
};

const addAxes = () => {
    const axes = new THREE.AxesHelper(50);
    scene.add(axes);
};

const render = () => {
    renderer.render(scene, camera);
    requestAnimationFrame(render);
};

const onWindowResize = () => {
    windowWidth = window.innerWidth;
    windowHeight = window.innerHeight;
    camera.left = windowWidth / -factor;
    camera.right = windowWidth / factor;
    camera.top = windowHeight / factor;
    camera.bottom = windowHeight / -factor;
    camera.updateProjectionMatrix();
    renderer.setSize(windowWidth, windowHeight);
};

const addLights = () => {
    ambientlight = new THREE.AmbientLight(0xffffff, 0.9);
    hemisphereLight = new THREE.HemisphereLight(0xffffff, 0xffffff, 0.1);
    directionalLight = new THREE.DirectionalLight(0xffffff, 0.1);
    directionalLight.position.set(10, 100, 30);
    directionalLight.castShadow = true;
    scene.add(ambientlight);
    scene.add(hemisphereLight);
    scene.add(directionalLight);
};

class Ground {
    constructor() {
        this.mesh;
        this.addPlane();
    }
    addPlane() {
        this.mesh = new THREE.Object3D();
        const geometry = new THREE.PlaneGeometry(20, 20, 1, 1);
        const material = new THREE.MeshLambertMaterial({
            color: data.colors.ground,
        });
        const plane = new THREE.Mesh(geometry, material);
        plane.receiveShadow = true;
        this.mesh.add(plane);
    }
}

const addGround = () => {
    const newGround = new Ground();
    newGround.mesh.position.y = -2;
    newGround.mesh.rotation.x = (-90 * Math.PI) / 180;
    scene.add(newGround.mesh);
};

class Plateform {
    constructor() {
        this.mesh;
        this.plateformfn();
    }
    plateformfn() {
        this.mesh = new THREE.Object3D();
        const geometry = new THREE.BoxGeometry(1, 0.2, 1);
        const material = new THREE.MeshLambertMaterial({
            color: data.colors.plateform,
        });
        const plateform = new THREE.Mesh(geometry, material);
        plateform.castShadow = true;
        this.mesh.add(plateform);
    }
}

const addPlateform = () => {
    const plateformPosition = [
        [0, -1, 0.5],
        [0, -1, -0.5],
        [-1, -1, -0.5],
    ];
    for (const key in plateformPosition) {
        const plateform = new Plateform();
        plateform.mesh.position.x = plateformPosition[key][0];
        plateform.mesh.position.y = plateformPosition[key][1];
        plateform.mesh.position.z = plateformPosition[key][2];
        scene.add(plateform.mesh);
    }
};

class Cube {
    constructor() {
        this.mesh;
        this.cube;
        this.createCube();
    }
    createCube() {
        this.mesh = new THREE.Object3D();
        const geometry = new THREE.BoxGeometry(1, 1, 1);
        const material = new THREE.MeshLambertMaterial({
            color: data.colors.cube,
            transparent: true,
        });
        this.cube = new THREE.Mesh(geometry, material);
        this.mesh.add(this.cube);
    }
}

const addCube = () => {
    cube = new Cube();
    scene.add(cube.mesh);
    // console.log(cube)
    cube.cube.position.set(0, 2, 0.5);
};

const addControls = () => {
    new OrbitControls(camera, renderer.domElement);
};

const getDegree = (degree) => (degree * Math.PI) / 180;

const timeline = () => {
    const t1 = gsap.timeline({
        repeat: -1,
        repeatDelay: data.delay,
    });
    // console.log(cube.cube)
    t1.set(cube.cube.material, {
        opacity: 0,
    });

    t1.to(cube.cube.position, {
        duration: 0.8,
        y: -0.4,
        ease: "Bounce.easeOut",
    });

    t1.to(
        cube.cube.scale,
        {
            duration: 0.8,
            y: 1,
            ease: "Bounce.easeOut",
        },
        "-=0.8"
    );

    t1.to(
        cube.cube.material,
        {
            duration: 0.5,
            opacity: 1,
        },
        "<"
    ); //插入到上一个动画的开始位置

    t1.to(
        cube.cube.rotation,
        {
            duration: 0.8,
            x: getDegree(-90),
        },
        "+=0.2"
    );

    t1.to(
        cube.cube.position,
        {
            duration: 0.3,
            y: -0.2,
        },
        "<"
    );

    t1.to(
        cube.cube.position,
        {
            duration: 0.8,
            z: -0.5,
        },
        "<"
    );

    t1.to(
        cube.cube.position,
        {
            duration: 0.3,
            y: -0.4,
        },
        "-=0.4"
    );

    t1.to(cube.cube.rotation, {
        duration: 0.8,
        y: getDegree(-90),
    });

    t1.to(
        cube.cube.position,
        {
            duration: 0.3,
            y: -0.2,
        },
        "<"
    );

    t1.to(
        cube.cube.position,
        {
            duration: 0.8,
            x: -1,
        },
        "<"
    );

    t1.to(
        cube.cube.position,
        {
            duration: 0.3,
            y: -0.4,
        },
        "-=0.4"
    );

    t1.to(cube.cube.rotation, {
        duration: 0.8,
        x: 0,
        ease: "Power3.easeOut",
    });

    t1.to(
        cube.cube.position,
        {
            duration: 0.8,
            z: 0.8,
            ease: "Power3.easeOut",
        },
        "<"
    );

    t1.to(
        cube.cube.position,
        {
            duration: 0.6,
            y: -4,
            ease: "Power3.easeIn",
        },
        "<"
    );

    t1.to(
        cube.cube.scale,
        {
            duration: 0.8,
            y: 1.5,
        },
        "-=0.5"
    );

    t1.to(
        cube.cube.material,
        {
            duration: 0.25,
            opacity: 0,
        },
        "-=0.85"
    );

    t1.timeScale(data.duration);
};

const init = () => {
    addScene();
    // addAxes();
    addCamera();
    addRenderer();
    render();
    addLights();
    addGround();
    addPlateform();
    addCube();
    addControls();
    timeline();
    window.addEventListener("resize", onWindowResize);
};



const guiFn = () => {

  const ctrlObj = {
    ambientlightVisible: true,
    ambientlightColor: 0xffffff,
    ambientlightIntensity: 0.9,

    hemisphereLightVisible: true,
    hemisphereLightGroundColor: 0xffffff,
    hemisphereLightSkyColor: 0xffffff,
    hemisphereIntensity: 0.1,

    directionalLightColor: 0xffffff,
    directionalLightIntensity: 0.1,
    directionalLightVisible: true,

    cubeColor: 0xffffff,
};

    const ctrl = new GUI();
    const ambientlightFolder = ctrl.addFolder("ambientlight");
    ambientlightFolder.add(ctrlObj, "ambientlightVisible").onChange((e) => {
        ambientlight.visible = e;
    });
    ambientlightFolder.addColor(ctrlObj, "ambientlightColor").onChange((e) => {
        ambientlight.color = new THREE.Color(e);
    });
    ambientlightFolder
        .add(ctrlObj, "ambientlightIntensity", 0, 1)
        .onChange((e) => {
            ambientlight.intensity = e;
        });

    const directionalLightFolder = ctrl.addFolder("directionalLight");
    directionalLightFolder
        .add(ctrlObj, "directionalLightVisible")
        .onChange((e) => {
            directionalLight.visible = e;
        });
    directionalLightFolder
        .addColor(ctrlObj, "directionalLightColor")
        .onChange((e) => {
            directionalLight.color = new THREE.Color(e);
        });
    directionalLightFolder
        .add(ctrlObj, "directionalLightIntensity", 0, 0.1)
        .onChange((e) => {
            directionalLight.intensity = e;
        });

    const hemisphereLightFolder = ctrl.addFolder("hemisphereLight");
    hemisphereLightFolder
        .add(ctrlObj, "hemisphereLightVisible")
        .onChange((e) => {
            hemisphereLight.visible = e;
        });
    hemisphereLightFolder
        .addColor(ctrlObj, "hemisphereLightGroundColor")
        .onChange((e) => {
            hemisphereLight.groundColor = new THREE.Color(e);
        });
    hemisphereLightFolder
        .addColor(ctrlObj, "hemisphereLightSkyColor")
        .onChange((e) => {
            hemisphereLight.color = new THREE.Color(e);
        });
    hemisphereLightFolder.add(ctrlObj, "cubeColor").onChange((e) => {
        hemisphereLight.intensity = e;
    });
    const cubeFolder = ctrl.addFolder("cube");
    cubeFolder.addColor(ctrlObj, "hemisphereLightGroundColor").onChange((e) => {
        cube.cube.material.color = new THREE.Color(e);
    });
    
};

onMounted(() => {
  init();
  guiFn()
});
</script>

<style lang="less">
body {
    margin: 0;
}
</style>
