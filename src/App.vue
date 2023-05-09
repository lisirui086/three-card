<script setup>
// 引入组合式API
import { ref, onMounted } from 'vue'

// 引入lodash的debounced 防抖
import { debounced } from 'lodash'

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
camera.position.set(-3.23, 3, 4.06)
camera.aspect = window.innerWidth / window.innerHeight
scene.add(camera)

// 初始化渲染器
const renderer = new THREE.WebGLRenderer( {
  antialias: true
} )
renderer.setSize(window.innerWidth, window.innerHeight)
// 设置色调映射
renderer.outputColorSpace = THREE.SRGBColorSpace
renderer.toneMapping = THREE.ACESFilmicToneMapping
renderer.toneMappingExposure = 0.5
renderer.shadowMap.enabled = true
// renderer.shadowMap.type = THREE.PCFSoftShadowMap
// renderer.useLegacyLights = true
renderer.physicallyCorrectLights = true;
// 声明控制器
let controls

// 添加平行光
const directionalLight = new THREE.DirectionalLight( 0xffffff, 1)
directionalLight.position.set(0, 50, 0)
scene.add(directionalLight)

// 添加点光源
const pointLight = new THREE.PointLight( 0xffffff, 50 )
pointLight.position.set(0.1, 2.4, 0)
pointLight.castShadow = true
scene.add(pointLight)

// 创建点光源组
const pointLightGroup = new THREE.Group()
pointLightGroup.position.set(-8, 2.5, -1.5)
let pointLightArr = []
const ballRadius = 3
for (let i =0; i < 3; i++ ){
  // 创建小灯球
  const ballGeometry = new THREE.SphereGeometry(0.2, 32, 32)
  const ballMaterial = new THREE.MeshStandardMaterial( {
    color: 0xffffff,
    emissive: 0xffffff,
    emissiveIntensity: 10
  } )
  const ball = new THREE.Mesh( ballGeometry, ballMaterial )

  pointLightArr.push(ball)


  // 小灯球位置
  ball.position.set(
    ballRadius * Math.cos( (i * 2 * Math.PI) / 3 ),
    Math.cos((i * 2 * Math.PI) / 3),
    ballRadius * Math.sin((i*2*Math.PI)/3)
  )

  const ballLight = new THREE.PointLight( 0xffffff, 50 )
  ball.add(ballLight)
  pointLightGroup.add(ball)
}
scene.add(pointLightGroup)

// 使用补间动画， 从0到2Π，使用小灯球旋转
let options = {
  angle: 0
}

gsap.to(options, {
  angle: Math.PI * 2,
  duration: 10,
  repeat: -1,
  ease: 'linear',
  onUpdate: () => {
    pointLightGroup.rotation.y = options.angle
    // 实现上下跳动效果
    pointLightArr.forEach((ball, index) => {
      ball.position.set(
        ballRadius * Math.cos( (index * 2 * Math.PI) / 3 ),
        Math.cos( (index * 2 * Math.PI ) / 3 + options.angle * 5 ),
        ballRadius * Math.sin((index * 2 * Math.PI ) / 3)
      )
    })
  }
})

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
    if(child instanceof THREE.Mesh) {
      child.castShadow = true;
      child.receiveShadow = true;
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

// 补间动画 gsap.timeline()
let timeline1 = gsap.timeline()
let timeline2 = gsap.timeline()

// 定义相机移动函数
function tranlateCamera(position, target){
  timeline1.to(camera.position, {
    x: position.x,
    y: position.y,
    z: position.z,
    duration: 1,
    ease: 'power2.inOut'
  })
  // 控制器的焦点，.object的轨道围绕它运行.它可以在任何时候被手动更新，以更改控制器的焦点。
  timeline2.to(controls.target, {
    x: target.x,
    y: target.y,
    z: target.z,
    duration: 1,
    ease: 'power2.inOut'
  })
}


// 场景集合
const scenes = [
  {
    title: '圣诞快乐',
    callback: () => {
      tranlateCamera(
        new THREE.Vector3(-3.23, 3, 4.06),
        new THREE.Vector3(-8, 2, 0)
      )
    }
  },
  {
    title: '世界那么大感谢遇见你',
    callback: () => {
      // 执行切换位置
      tranlateCamera(
        new THREE.Vector3(7, 0, 23),
        new THREE.Vector3(0, 0, 0)
      )
    }
  },
  {
    title: '愿与你探寻世界每一个角落',
    callback: () => {
      // 执行切换位置
      tranlateCamera(
        new THREE.Vector3(10, 3, 0),
        new THREE.Vector3(5, 2, 0)
      )
    }
  },
  {
    title: '愿将天上的星星送给你',
    callback: () => {
      // 执行切换位置
      tranlateCamera(
        new THREE.Vector3(7, 0, 23),
        new THREE.Vector3(0, 0, 0)
      )
    }
  },
  {
    title: '祝大家平安喜乐',
    callback: () => {
      // 执行切换位置
      tranlateCamera(
        new THREE.Vector3(-20, 1.3, 6.6),
        new THREE.Vector3(5, 2, 0)
      )
    }
  }
]

// 场景索引
const sceneIndex = ref(0)


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
  controls.target.set(-8, 2, 0)
  // 启用阻尼（惯性）
  controls.enableDamping = true
  // 因为开启了阻尼 必须在动画循环里调用
  controls.update()

  window.addEventListener('resize', handResize)
  window.addEventListener('wheel', (e) => {
    // if(isAnimate) return
    if(e.deltaY > 0){
      sceneIndex.value++
      if(sceneIndex.value > scenes.length - 1){
        // 重置
        sceneIndex.value = 0
      }
    }
    // 执行对应索引场景的回调函数
    scenes[sceneIndex.value].callback()
    /* setTimeout(()=>{
      isAnimate = false
    }, 1000) */
  }, false)
})

animate()

</script>

<template>
  <div class='canvas' ref='canvas'>
    <div class='scenes' :style="{transform: `translate3d(0, ${-sceneIndex * 100}vh, 0)`}">
      <div class="info" v-for='item in scenes' :key='index'>
        <h1 class='text'> {{ item.title }} </h1>
      </div>
    </div>
  </div>
</template>

<style scoped lang='less'>
  .canvas {
    width: 100vw;
    height: 100vh;
    position: fixed;
    top: 0;
    left: 0;

    .scenes {
      position: fixed;
      left: 0;
      top: 0;
      z-index: 10;
      pointer-events: none;
      transition: all 1s;

      .info {
        width: 100vw;
        height: 100vh;

        .text {
          padding: 100px 50px;
          font-size: 50px;
          color: white;
        }
      }
    }
  }
</style>
