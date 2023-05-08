<script setup>
// 引入组合式API
import { ref, onMounted } from 'vue'

// 引入threejs
import * as THREE from 'three'

// 引入补间动画库
import gsap from 'gsap'

// 引入轨道控制器 
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js';

// GLTF加载器 加载模型
import { GLTFLoader } from 'three/addons/loaders/GLTFLoader.js'

// 引入Draco压缩网格加载器 解压模型
import { DRACOLoader } from 'three/addons/loaders/DRACOLoader.js'

// 引入hdr图像加载器
import { RGBELoader } from 'three/addons/loaders/RGBELoader.js'

// 引入theejs的Water库
import { Water } from 'three/addons/objects/Water2.js'

// 初始化场景
const scene = new THREE.Scene()

// 获取画布节点
const canvas = ref(null)

// 初始化相机
const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000)
camera.position.set(-3.23, 2.98, 4.06)
camera.aspect = window.innerWidth / window.innerHeight
scene.add(camera)

// 初始化渲染器
const renderer = new THREE.WebGLRenderer( { antialias: true } )
renderer.setSize(window.innerWidth, window.innerHeight)
// 设置色调映射
renderer.outputColorSpace = THREE.SRGBColorSpace
renderer.toneMapping = THREE.ACESFilmicToneMapping
renderer.toneMappingExposure = 0.5

// 声明控制器
let controls

// 添加平行光
const directionalLight = new THREE.DirectionalLight( 0xffffff, 1)
directionalLight.position.set(0, 50, 0)
scene.add(directionalLight)

// 创建水面
const waterGeometry = new THREE.CircleGeometry(300, 32)
const water = new Water(waterGeometry, {
  textureWidth: 512,
  textureHeight: 512,
  color: 0xeeeeff,
  flowDirection: new THREE.Vector2(1,1),
  scale: 100
})
water.rotation.x = -Math.PI / 2
water.position.y = -0.4

scene.add(water)

// 初始化GLTF加载器、DRACOLoader 
const loader = new GLTFLoader()
const dracoLoader = new DRACOLoader()
dracoLoader.setDecoderPath('/draco/gltf/')
loader.setDRACOLoader( dracoLoader )

loader.load('/model/scene.glb', (gltf) => {
  const model = gltf.scene
  model.traverse(child=>{
    if(child.name === 'Plane') {
      child.visible = false
    }
  })
  scene.add(model)
})

// 初始化hdr图像加载器
const rgbeLoader = new RGBELoader()
rgbeLoader.load('/textures/sky.hdr', texture => {
  texture.mapping = THREE.EquirectangularRefractionMapping
  scene.background = texture
  scene.environment = texture
})

// 渲染函数
function animate() {
  controls && controls.update()
  requestAnimationFrame (animate)
  renderer.render(scene, camera)
}

// 根据屏幕宽高，修改相机，渲染器宽高
function handResize() {
  camera.aspect = window.innerWidth / window.innerHeight
  camera.updateProjectionMatrix()
  renderer.setSize(window.innerWidth, window.innerHeight)
}

// 组件挂载完毕后
onMounted(() => {
  // 将渲染器追加到画布上
  canvas.value.appendChild(renderer.domElement)

  // 初始化轨道控制器
  controls = new OrbitControls(camera, canvas.value)
  // 启用阻尼（惯性）
  controls.enableDamping = true
  // 因为开启了阻尼 必须在动画循环里调用
  controls.update()

  window.addEventListener('resize', handResize)
}),

animate()

</script>

<template>
  <div class='canvas' ref='canvas'></div>
</template>

<style scoped>
  .canvas {
    width: 100vw;
    height: 100vh;
    position: fixed;
    top: 0;
    left: 0;
  }
</style>
