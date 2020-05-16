<template>
    <div>
        <h2>3d</h2>
        <div class="scene" ref="scene" />
    </div>
</template>

<script lang="ts">
import { Component, Prop, Vue } from "vue-property-decorator";
import * as THREE from "three";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls";

@Component<Torus>({
    mounted() {
        console.log("mounted");
        const el = this.$refs.scene as Element;
        const canvas = document.getElementById("canvas") as HTMLCanvasElement;
        const texture = new THREE.CanvasTexture(canvas);

        this.camera = new THREE.PerspectiveCamera(
            75,
            el.clientWidth / el.clientHeight,
            0.1,
            1000
        );

        this.controls = new OrbitControls(
            this.camera,
            this.renderer.domElement
        );

        this.renderer.setSize(el.clientWidth, el.clientHeight);

        this.scene.add(
            this.ambientLight,
            this.hemisphereLight,
            this.directionalLight
        );
        this.directionalLight.position.set(150, 350, 350);

        el.appendChild(this.renderer.domElement);

        const geometry = new THREE.TorusGeometry(10, 9, 16, 100);
        this.material = new THREE.MeshPhongMaterial({
            color: 0xffffaa,
            map: texture,
        });
        const torus = new THREE.Mesh(geometry, this.material);
        this.scene.add(torus);
        this.camera.position.z = 40;
        const animate = () => {
            requestAnimationFrame(animate);
            //     torus.rotation.x += 0.01;
            //   torus.rotation.y += 0.01;
            if (this.material.map) {
                this.material.map.needsUpdate = true;
            }
            this.renderer.render(this.scene, this.camera);
        };

        animate();
    },
})
export default class Torus extends Vue {
    private camera!: THREE.PerspectiveCamera;
    private renderer: THREE.WebGLRenderer = new THREE.WebGLRenderer();
    private scene: THREE.Scene = new THREE.Scene();
    private controls!: OrbitControls;
    private material!: THREE.MeshPhongMaterial;

    private hemisphereLight = new THREE.HemisphereLight(
        0xaaaaaa,
        0x000000,
        0.9
    );

    private directionalLight = new THREE.DirectionalLight(0xffffff, 0.9);

    private ambientLight = new THREE.AmbientLight(0xdc8874, 0.5);
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.scene {
    width: 100%;
    height: 500px;
}
</style>
