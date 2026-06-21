<template>
  <div class="bg-slate-800 rounded-lg p-4 border border-slate-700">
    <div class="flex items-center justify-between mb-3">
      <h3 class="text-sm font-bold text-slate-400">3D 分子结构</h3>
      <div v-if="store.currentMolecule && !store.benchmarkMolecule" class="text-right">
        <div class="text-lg font-bold text-cyan-400">{{ store.currentMolecule.name }}</div>
        <div class="text-xs text-slate-500">{{ store.currentMolecule.formula }} · MW {{ store.currentMolecule.mw }}</div>
      </div>
    </div>

    <div v-if="store.benchmarkMolecule" class="flex gap-4">
      <div class="flex-1">
        <div class="mb-2">
          <div class="text-lg font-bold text-orange-400">基准: {{ store.benchmarkMolecule.name }}</div>
          <div class="text-xs text-slate-500">{{ store.benchmarkMolecule.formula }} · MW {{ store.benchmarkMolecule.mw }}</div>
        </div>
        <div ref="containerLeftRef" class="w-full h-96 bg-slate-900 rounded-lg border border-slate-700 flex items-center justify-center">
          <span v-if="!store.benchmarkMolecule" class="text-slate-500">选择分子查看3D结构</span>
        </div>
      </div>
      <div class="flex-1">
        <div class="mb-2">
          <div class="text-lg font-bold text-cyan-400">当前: {{ store.currentMolecule?.name }}</div>
          <div class="text-xs text-slate-500">{{ store.currentMolecule?.formula }} · MW {{ store.currentMolecule?.mw }}</div>
        </div>
        <div ref="containerRightRef" class="w-full h-96 bg-slate-900 rounded-lg border border-slate-700 flex items-center justify-center">
          <span v-if="!store.currentMolecule" class="text-slate-500">选择分子查看3D结构</span>
        </div>
      </div>
    </div>

    <div v-else ref="containerRef" class="w-full h-96 bg-slate-900 rounded-lg border border-slate-700 flex items-center justify-center">
      <span v-if="!store.currentMolecule" class="text-slate-500">选择分子查看3D结构</span>
    </div>

    <div class="mt-3 grid grid-cols-4 gap-2 text-xs">
      <div class="flex items-center gap-1"><span class="w-4 h-4 rounded-full bg-gray-500"></span>碳C</div>
      <div class="flex items-center gap-1"><span class="w-4 h-4 rounded-full bg-blue-500"></span>氮N</div>
      <div class="flex items-center gap-1"><span class="w-4 h-4 rounded-full bg-red-500"></span>氧O</div>
      <div class="flex items-center gap-1"><span class="w-4 h-4 rounded-full bg-yellow-500"></span>硫S</div>
    </div>
    <div class="mt-2 text-xs text-slate-500 font-mono">{{ store.currentMolecule?.smiles }}</div>
  </div>
</template>

<script setup lang="ts">
import { ref, watch, onUnmounted, nextTick } from 'vue'
import { useMoleculeStore } from '../store/molecule'
import type { MoleculeData } from '../types'
import * as THREE from 'three'

interface ThreeInstance {
  scene: THREE.Scene
  renderer: THREE.WebGLRenderer
  animationId: number
  group: THREE.Group
  camera: THREE.PerspectiveCamera
}

const store = useMoleculeStore()
const containerRef = ref<HTMLDivElement | null>(null)
const containerLeftRef = ref<HTMLDivElement | null>(null)
const containerRightRef = ref<HTMLDivElement | null>(null)

let singleInstance: ThreeInstance | null = null
let leftInstance: ThreeInstance | null = null
let rightInstance: ThreeInstance | null = null

function createThreeInstance(container: HTMLDivElement): ThreeInstance {
  const scene = new THREE.Scene()
  scene.background = new THREE.Color(0x0f172a)
  const camera = new THREE.PerspectiveCamera(60, container.clientWidth / container.clientHeight, 0.1, 1000)
  camera.position.set(0, 0, 10)
  const ambLight = new THREE.AmbientLight(0xffffff, 0.6)
  scene.add(ambLight)
  const dirLight = new THREE.DirectionalLight(0xffffff, 0.8)
  dirLight.position.set(5, 5, 5)
  scene.add(dirLight)
  const renderer = new THREE.WebGLRenderer({ antialias: true })
  renderer.setSize(container.clientWidth, container.clientHeight)
  container.innerHTML = ''
  container.appendChild(renderer.domElement)

  const group = new THREE.Group()
  scene.add(group)

  let isDragging = false
  let prevX = 0, prevY = 0
  container.addEventListener('mousedown', (e) => { isDragging = true; prevX = e.clientX; prevY = e.clientY })
  container.addEventListener('mouseup', () => { isDragging = false })
  container.addEventListener('mousemove', (e) => {
    if (!isDragging) return
    group.rotation.y += (e.clientX - prevX) * 0.01
    group.rotation.x += (e.clientY - prevY) * 0.01
    prevX = e.clientX; prevY = e.clientY
  })
  container.addEventListener('wheel', (e) => {
    e.preventDefault()
    camera.position.z = Math.max(3, Math.min(30, camera.position.z + e.deltaY * 0.01))
  }, { passive: false })

  let animationId: number
  function animate() {
    animationId = requestAnimationFrame(animate)
    renderer.render(scene, camera)
  }
  animate()

  return { scene, renderer, animationId, group, camera }
}

function disposeInstance(inst: ThreeInstance | null) {
  if (!inst) return
  cancelAnimationFrame(inst.animationId)
  inst.renderer.dispose()
  inst.scene.traverse((obj) => {
    if (obj instanceof THREE.Mesh) {
      obj.geometry.dispose()
      if (Array.isArray(obj.material)) {
        obj.material.forEach(m => m.dispose())
      } else {
        obj.material.dispose()
      }
    }
  })
}

function renderMoleculeToGroup(group: THREE.Group, mol: MoleculeData) {
  while (group.children.length > 0) group.remove(group.children[0])

  const center = { x: 0, y: 0, z: 0 }
  mol.atoms.forEach(a => { center.x += a.x; center.y += a.y; center.z += a.z })
  center.x /= mol.atoms.length; center.y /= mol.atoms.length; center.z /= mol.atoms.length

  mol.atoms.forEach(atom => {
    const radius = Math.max(0.15, atom.radius * 1.2)
    const geo = new THREE.SphereGeometry(radius, 32, 32)
    const color = new THREE.Color(atom.color)
    const mat = new THREE.MeshPhongMaterial({ color })
    const mesh = new THREE.Mesh(geo, mat)
    mesh.position.set(atom.x - center.x, atom.y - center.y, atom.z - center.z)
    group.add(mesh)
  })

  mol.bonds.forEach(bond => {
    const a1 = mol.atoms[bond.atom1]
    const a2 = mol.atoms[bond.atom2]
    if (!a1 || !a2) return
    const start = new THREE.Vector3(a1.x - center.x, a1.y - center.y, a1.z - center.z)
    const end = new THREE.Vector3(a2.x - center.x, a2.y - center.y, a2.z - center.z)
    const dir = end.clone().sub(start)
    const len = dir.length()
    const geo = new THREE.CylinderGeometry(0.06, 0.06, len, 16)
    const mat = new THREE.MeshPhongMaterial({ color: 0x64748b })
    const mesh = new THREE.Mesh(geo, mat)
    mesh.position.copy(start).add(dir.multiplyScalar(0.5))
    mesh.lookAt(end)
    mesh.rotateX(Math.PI / 2)
    group.add(mesh)
  })
}

function initSingle() {
  if (singleInstance) return
  const container = containerRef.value
  if (!container) return
  singleInstance = createThreeInstance(container)
}

function initDual() {
  if (leftInstance && rightInstance) return
  const leftContainer = containerLeftRef.value
  const rightContainer = containerRightRef.value
  if (!leftContainer || !rightContainer) return
  if (!leftInstance) leftInstance = createThreeInstance(leftContainer)
  if (!rightInstance) rightInstance = createThreeInstance(rightContainer)
}

function disposeAll() {
  disposeInstance(singleInstance)
  singleInstance = null
  disposeInstance(leftInstance)
  leftInstance = null
  disposeInstance(rightInstance)
  rightInstance = null
}

function refreshViews() {
  disposeAll()
  nextTick(() => {
    if (store.benchmarkMolecule) {
      initDual()
      if (store.benchmarkMolecule && leftInstance) {
        renderMoleculeToGroup(leftInstance.group, store.benchmarkMolecule)
      }
      if (store.currentMolecule && rightInstance) {
        renderMoleculeToGroup(rightInstance.group, store.currentMolecule)
      }
    } else {
      initSingle()
      if (store.currentMolecule && singleInstance) {
        renderMoleculeToGroup(singleInstance.group, store.currentMolecule)
      }
    }
  })
}

setTimeout(() => { refreshViews() }, 100)

watch([() => store.currentMolecule, () => store.benchmarkMolecule], () => {
  if (store.benchmarkMolecule) {
    if (!leftInstance || !rightInstance) {
      refreshViews()
    } else {
      if (store.benchmarkMolecule) {
        renderMoleculeToGroup(leftInstance.group, store.benchmarkMolecule)
      }
      if (store.currentMolecule) {
        renderMoleculeToGroup(rightInstance.group, store.currentMolecule)
      }
    }
  } else {
    if (!singleInstance) {
      refreshViews()
    } else {
      if (store.currentMolecule) {
        renderMoleculeToGroup(singleInstance.group, store.currentMolecule)
      }
    }
  }
})

onUnmounted(() => { disposeAll() })
</script>
