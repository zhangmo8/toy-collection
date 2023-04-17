<script setup lang="ts" generic="T extends any, O extends any">
import * as THREE from 'three'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js'
import Stats from 'three/examples/jsm/libs/stats.module.js'

import EarthMap from '~/assets/map.jpeg'

const earth = ref()
const renderer = ref()
const scene = new THREE.Scene()
const camera = new THREE.PerspectiveCamera(45, 500 / 500, 1, 500)

const stats = new Stats()
const group = new THREE.Group()

const points = [
  [39.9, 116.3],
  [129.58, 115.05],
]

function init() {
  document.body.appendChild(stats.dom)

  renderer.value = new THREE.WebGLRenderer({
    canvas: earth.value,
    antialias: true,
    logarithmicDepthBuffer: true,
    alpha: true,
    precision: 'highp',
  })
  toRaw(renderer.value).setSize(window.innerWidth, window.innerHeight)
  toRaw(renderer.value).setPixelRatio(window.devicePixelRatio)
  toRaw(renderer.value).outputEncoding = THREE.sRGBEncoding
  toRaw(renderer.value).shadowMap.enabled = true
  toRaw(renderer.value).shadowMap.type = THREE.PCFSoftShadowMap
  toRaw(renderer.value).setClearColor(0x000000)

  camera.position.set(40, 40, 40)
  camera.lookAt(new THREE.Vector3(0, 0, 0))
  camera.aspect = window.innerWidth / window.innerHeight
  camera.updateProjectionMatrix()
  scene.add(camera)

  const light = new THREE.SpotLight(0xF1F1F1)
  light.position.set(50, 150, 50)
  light.castShadow = true
  light.shadow.mapSize.width = 1024 * 3
  light.shadow.mapSize.height = 1024 * 3
  scene.add(light)

  // eslint-disable-next-line no-new
  new OrbitControls(camera, toRaw(renderer.value).domElement)
}

async function loadMaterial() {
  const geometry = new THREE.SphereGeometry(15, 64, 32)
  const texture = await new THREE.TextureLoader().loadAsync(EarthMap)

  const material = new THREE.MeshBasicMaterial({ map: texture })
  const sphere = new THREE.Mesh(geometry, material)
  sphere.castShadow = true
  scene.add(sphere)

  // const planeGeometry = new THREE.PlaneGeometry(200, 200, 32, 32)
  // const planeMaterial = new THREE.MeshLambertMaterial({ color: 0xFFFFFF })
  // const plane = new THREE.Mesh(planeGeometry, planeMaterial)
  // plane.position.set(0, -25, 0)
  // plane.rotateX(-180 / 360 * Math.PI)
  // plane.receiveShadow = true
  // scene.add(plane)

  const pointVectors: THREE.Vector3[] = points.map(item => getPositionByLL(item[0], item[1]))

  const line = getBezierPoint(pointVectors[0], pointVectors[1])

  const curve = new THREE.CubicBezierCurve3(
    pointVectors[0],
    line.v1,
    line.v2,
    pointVectors[1],
  )

  const curvePoints = curve.getPoints(150)
  const curveGeometry = new THREE.BufferGeometry().setFromPoints(curvePoints)
  const curveMaterial = new THREE.LineBasicMaterial({ color: 0xCCCFF, linewidth: 20 })
  const curveObject = new THREE.Line(curveGeometry, curveMaterial)

  scene.add(curveObject)
  animate()
}

function getPositionByLL(longitude: number, latitude: number) {
  const lg = THREE.MathUtils.degToRad(longitude)
  const lt = THREE.MathUtils.degToRad(latitude)
  const temp = 15 * Math.cos(lt)
  const x = temp * Math.sin(lg)
  const y = 15 * Math.sin(lt)
  const z = temp * Math.cos(lg)
  return new THREE.Vector3(x, y, z)
}

function getBezierPoint(startDot: THREE.Vector3, endDot: THREE.Vector3) {
  const angle = (startDot.angleTo(endDot) * 180) / Math.PI

  const aLen = angle * 0.1
  const hLen = angle * angle * 10

  const origin = new THREE.Vector3(0, 0, 0)
  const scale = new THREE.Vector3(0, 1, 0)

  const rayLine = new THREE.Ray(origin, getVCenter(startDot.clone(), endDot.clone()))

  const topDot = rayLine.at(hLen / rayLine.at(1, scale).distanceTo(origin), scale)

  const v1 = getLenVcetor(startDot.clone(), topDot, aLen)

  const v2 = getLenVcetor(endDot.clone(), topDot, aLen)

  return {
    v1,
    v2,
  }
}

// function getBezierPoint(startDot: THREE.Vector3, endDot: THREE.Vector3) {
//   const angle = (startDot.angleTo(endDot) * 180) / Math.PI

//   const aLen = angle * 0.2
//   const hLen = angle * angle * 50

//   const origin = new THREE.Vector3(0, 0, 0)

//   const topV = new THREE.Vector3(0, 1, 0)
//   const centerDot = getVCenter(startDot.clone(), endDot.clone())

//   const rayLine = new THREE.Ray(centerDot, topV)

//   const topDot = rayLine.at(hLen / rayLine.at(1, origin).distanceTo(centerDot), origin)

//   const v1 = getLenVcetor(startDot.clone(), topDot, aLen)

//   const v2 = getLenVcetor(endDot.clone(), topDot, aLen)

//   return {
//     v1,
//     v2,
//   }
// }

function getVCenter(start: THREE.Vector3, end: THREE.Vector3) {
  const vCenter = start.add(end)
  return vCenter.divideScalar(2)
}

function getLenVcetor(v1: THREE.Vector3, v2: THREE.Vector3, len: number) {
  const v1v2Len = v1.distanceTo(v2)
  return v1.lerp(v2, len / v1v2Len)
}

function render() {
  scene.add(group)
  toRaw(renderer.value).render(scene, camera)
}

function animate() {
  stats.update()
  render()
  requestAnimationFrame(animate)
}

onMounted(() => {
  init()
  loadMaterial()

  window.addEventListener('resize', () => {
    camera.aspect = window.innerWidth / window.innerHeight

    camera.updateProjectionMatrix()

    toRaw(renderer.value).setSize(window.innerWidth, window.innerHeight)
    toRaw(renderer.value).setPixelRatio(window.devicePixelRatio)
  })
})
</script>

<template>
  <canvas ref="earth" />
</template>
