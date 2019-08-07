<template>
  <div class="home">
    <img alt="Vue logo" src="../assets/logo.png">

    Близость камеры
    <input
      v-model.number="positionZ"
      type="range"
      name="position-z"
      id="position-z"
      min="2"
      max="10"
    />

    <button @click="addCube">addCube</button>
    <button @click="initScene">initScene</button>

    <div ref="scene" id="scene"></div>
  </div>
</template>

<script lang="ts">

import { Vue, Component, Watch } from 'vue-property-decorator';

import {
  Scene,
  PerspectiveCamera,
  WebGLRenderer,
  BoxGeometry,
  Mesh,
  DirectionalLight,
  MeshPhongMaterial,
  GridHelper,
  AmbientLight,
  SpotLight,
  LightShadow,
  PlaneBufferGeometry,
  ShadowMaterial,
  MeshBasicMaterial,
  Color,
} from 'three';

import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js';

@Component({})
export default class Home extends Vue {
  positionZ: number = 5;

  SCENE: Scene = {};
  CAMERA: PerspectiveCamera = {};
  RENDERER: WebGLRenderer = {};
  LIGHT: SpotLight = {};
  CONTROLS: OrbitControls = {};
  GRID: GridHelper = {};
  AMBIENT_LIGHT: AmbientLight = {};

  objects: Array<any> = [];

  @Watch('positionZ')
  onPositionZChange (val: number) {
    this.CAMERA.position.z = val;
  }

  private initControls () {
    this.CONTROLS = new OrbitControls(this.CAMERA, this.RENDERER.domElement);
    this.CONTROLS.dampingFactor = 25;
    this.CONTROLS.minPolarAngle = Math.PI * 1 / 4;
    this.CONTROLS.maxPolarAngle = Math.PI * 3 / 4;
    this.CONTROLS.update();
  }

  private addCube () {
    const geometry = new BoxGeometry( 2, 2, 2 );

    const material = new MeshBasicMaterial({ color: 'red' });

    const mesh = new Mesh( geometry, material );

    mesh.position.x = 10;

    //scene is global
    this.SCENE.add(mesh);

    console.log(this.SCENE.children);
  }

  private initScene () {
    this.objects.forEach( obj => this.SCENE.add(obj) );
  }

  async mounted () {
    await this.$nextTick();

    const scene = document.getElementById('scene');
  
    const geometry = new BoxGeometry(1, 2, 1);
    const material = new MeshPhongMaterial({ color: 'gray' });
    const cube = new Mesh( geometry, material );
    const planeGeometry = new PlaneBufferGeometry(2000, 2000).rotateX(-Math.PI / 2);
    const planeMaterial = new ShadowMaterial({ opacity: 0.2 });

    if (scene) {
      this.SCENE = new Scene();
      this.SCENE.background = new Color(0xf0f0f0);
      
      this.CAMERA = new PerspectiveCamera(
        70,
        window.innerWidth / window.innerHeight,
        1,
        10000
      );
      this.CAMERA.position.z = 5;

      this.SCENE.add(new AmbientLight(0xf0f0f0));

      this.LIGHT = new SpotLight('#fff', 1.5);
      this.LIGHT.position.set( 0, 1500, 200 );
      this.LIGHT.castShadow = true;
      this.LIGHT.shadow = new LightShadow( new PerspectiveCamera( 70, 1, 200, 2000 ) );
      this.LIGHT.shadow.bias = - 0.000222;
      this.LIGHT.shadow.mapSize.width = 1024;
      this.LIGHT.shadow.mapSize.height = 1024;
      this.objects.push(this.LIGHT);
      
      this.GRID = new GridHelper( 100, 100, 'red', 'green' );
      this.GRID.position.y = 0;
      this.GRID.material.opacity = 0.25;
      this.GRID.material.transparent = true;
      this.GRID.receiveShadow = true;

      this.SCENE.add(this.GRID);

      cube.castShadow = true;
      cube.receiveShadow = true;

      const plane = new Mesh(planeGeometry, planeMaterial);
      plane.position.y = -1;
      plane.receiveShadow = true;

      this.SCENE.add(plane);

      this.objects.push(cube);

      this.RENDERER = new WebGLRenderer({ alpha: true });
 
      this.RENDERER.setPixelRatio(window.devicePixelRatio);
      this.RENDERER.setSize( window.innerWidth * 0.8, window.innerHeight * 0.8 );
      this.RENDERER.setClearColor('white');
      this.RENDERER.shadowMap.enabled = true;

      scene.appendChild( this.RENDERER.domElement );

      this.initControls();

      const animate = () => {
        requestAnimationFrame( animate );
        this.CONTROLS.update();
        cube.rotation.x += 0.01;
        cube.rotation.y += 0.01;
        this.RENDERER.render( this.SCENE, this.CAMERA );
      };
  
      animate();
      
      window.addEventListener('resize', () => {
        this.CAMERA.aspect = window.innerWidth / window.innerHeight;
        this.CAMERA.updateProjectionMatrix();

        this.RENDERER.setSize(window.innerWidth * 0.8, window.innerHeight * 0.8);
      });
    } else {
      console.error('Scene element is missed');
    }
  }
}

</script>

<style lang="scss">

.home {
  display: flex;
  flex-direction: column;
  align-items: center;
}

</style>

