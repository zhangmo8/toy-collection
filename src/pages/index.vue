<script setup lang="ts" generic="T extends any, O extends any">
import * as THREE from 'three'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js'
import EarthMap from '~/assets/map.jpeg'

const earth = ref()
const mixer = ref()
const renderer = ref()
const scene = new THREE.Scene()
const camera = new THREE.PerspectiveCamera(45, 500 / 500, 1, 1500)
const light = new THREE.DirectionalLight(0xFFFFFF)

const group = new THREE.Group()
const dotGroup = new THREE.Group()
const clock = new THREE.Clock()

const startPoint = [0, 40]

const destination = [
  [0, 0],
  [0, 180],
  [0, 90],
  [0, 0],
  [45, 0],
  [90, 0],
  [180, 0],
  [-90, 0],
  [-100, 0],
]

const posTracks: THREE.KeyframeTrack[] = []

function init() {
  renderer.value = new THREE.WebGLRenderer({ canvas: earth.value })
  toRaw(renderer.value).setSize(window.innerWidth, window.innerHeight)
  toRaw(renderer.value).setClearColor(0xFFFFFF)

  camera.position.set(100, 100, 1000)
  camera.lookAt(new THREE.Vector3(0, 0, 0))
  camera.aspect = window.innerWidth / window.innerHeight
  camera.updateProjectionMatrix()
  scene.add(camera)

  light.position.set(0, 200, 200)
  scene.add(light)

  const control = new OrbitControls(camera, toRaw(renderer.value).domElement)

  drawShape(getPositionByLL(startPoint[0], startPoint[1]))

  destination.forEach((ele, index) => {
    drawShape(getPositionByLL(ele[0], ele[1]))
    drawLine(startPoint[0], startPoint[1], ele[0], ele[1], index)
  })
}

function loadMaterial() {
  const geometry = new THREE.SphereGeometry(300, 200, 200)
  const textureLoader = new THREE.TextureLoader()
  textureLoader.load(EarthMap, (texture) => {
    const material = new THREE.MeshLambertMaterial({ map: texture, transparent: true, opacity: 1 })
    const mesh = new THREE.Mesh(geometry, material)

    group.add((mesh))
    scene.add(mesh)
    animate()
  })
}

function getPositionByLL(longitude: number, latitude: number) {
  const lg = THREE.MathUtils.degToRad(longitude)
  const lt = THREE.MathUtils.degToRad(latitude)
  const temp = 300 * Math.cos(lt)
  const x = temp * Math.sin(lg)
  const y = 300 * Math.sin(lt)
  const z = temp * Math.cos(lg)
  return new THREE.Vector3(x, y, z)
}

function drawShape(pos: { x: number; y: number; z: number }) {
  const groupPoint = new THREE.Group()
  // console.log(pos)
  const r = 10
  const geometryLine = new THREE.BoxGeometry()
  const arc = new THREE.ArcCurve(0, 0, r, 0, 2 * Math.PI, true)
  const points = arc.getPoints(200)

  const spherical = new THREE.Spherical()
  spherical.setFromCartesianCoords(pos.x, pos.y, pos.z)

  geometryLine.setFromPoints(points)
  const LineMaterial = new THREE.LineBasicMaterial({ color: 0x772FC4 })
  const line = new THREE.Line(geometryLine, LineMaterial)

  const shapePoint = new THREE.Shape()
  shapePoint.absarc(0, 0, r - 4, 0, 2 * Math.PI, false)

  const arcGeometry = new THREE.ShapeGeometry(shapePoint)
  const arcMaterial = new THREE.MeshBasicMaterial({ color: 0x772FC4 })
  const point = new THREE.Mesh(arcGeometry, arcMaterial)

  groupPoint.add(line)
  groupPoint.add(point)

  groupPoint.rotateX(spherical.phi - Math.PI / 2)
  groupPoint.rotateY(spherical.theta)
  groupPoint.position.set(pos.x, pos.y, pos.z + 1)

  group.add(groupPoint)
}

function drawLine(startLongitude: number, startLatitude: number, endLongitude: number, endLatitude: number, index: number) {
  const geometry = new THREE.BoxGeometry()

  const startPoint = getPositionByLL(startLongitude, startLatitude)
  const endPoint = getPositionByLL(endLongitude, endLatitude)

  const { v1, v2 } = getBezierPoint(startPoint, endPoint)

  const curve = new THREE.CubicBezierCurve3(startPoint, v1, v2, endPoint)

  const curvePoints = curve.getPoints(100)

  geometry.setFromPoints(curvePoints)

  const material = new THREE.LineBasicMaterial({
    color: 0xC44986,
  })

  const line = new THREE.Line(geometry, material)

  group.add(line)

  sport(curvePoints, index)
}

function getBezierPoint(startDot: THREE.Vector3, endDot: THREE.Vector3) {
  // 计算向量夹角
  const angle = (startDot.angleTo(endDot) * 180) / Math.PI // 0 ~ Math.PI

  const aLen = angle * 2.5
  const hLen = angle * angle * 50

  const origin = new THREE.Vector3(0, 0, 0)
  const scale = new THREE.Vector3(0, 0, 303)

  const rayLine = new THREE.Ray(origin, getVCenter(startDot.clone(), endDot.clone()))

  const topDot = rayLine.at(hLen / rayLine.at(1, scale).distanceTo(origin), scale)

  const v1 = getLenVcetor(startDot.clone(), topDot, aLen)

  const v2 = getLenVcetor(endDot.clone(), topDot, aLen)

  return {
    v1,
    v2,
  }
}

function getVCenter(start: THREE.Vector3, end: THREE.Vector3) {
  const vCenter = start.add(end)
  return vCenter.divideScalar(2)
}

function getLenVcetor(v1: THREE.Vector3, v2: THREE.Vector3, len: number) {
  const v1v2Len = v1.distanceTo(v2)
  return v1.lerp(v2, len / v1v2Len)
}

function drawDot(position: THREE.Vector3, name: string) {
  const box = new THREE.SphereGeometry(5, 5, 5)
  const material = new THREE.MeshLambertMaterial({
    color: 0x00BFFF,
  })
  const mesh = new THREE.Mesh(box, material)

  mesh.name = name
  mesh.position.set(position.x, position.y, position.z + 10)
  dotGroup.add(mesh)
  group.add(dotGroup)
  return mesh
}

function sport(curvePoints: THREE.Vector3[], index: number) {
  drawDot(curvePoints[0], `dot${index}`)
  const arr = Array.from(Array(101), (v, k) => k)
  const times = new Float32Array(arr)

  const posArr: number[] = []
  curvePoints.forEach((elem) => {
    posArr.push(elem.x, elem.y, elem.z)
  })

  const values = new Float32Array(posArr)

  const posTrack = new THREE.KeyframeTrack(
    `dot${index}.position`,
    times,
    values,
  )

  posTracks.push(posTrack)

  if (index === destination.length - 1)
    dotAnimation()
}

function dotAnimation() {
  const duration = 101
  const clip = new THREE.AnimationClip('default', duration, posTracks)
  mixer.value = new THREE.AnimationMixer(dotGroup)
  const AnimationAction = toRaw(mixer.value).clipAction(clip)
  AnimationAction.timeScale = 30
  AnimationAction.play()
}

function render() {
  scene.add(group)
  toRaw(renderer.value).render(scene, camera)
}

function animate() {
  render()
  const time = clock.getDelta()
  toRaw(mixer.value).update(time)
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
