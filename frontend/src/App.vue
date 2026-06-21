<template>
  <div class="min-h-screen bg-slate-900 text-slate-200">
    <header class="border-b border-slate-700 px-6 py-4">
      <h1 class="text-2xl font-bold text-cyan-400">药物分子 3D 结构与 ADMET 性质预测</h1>
      <p class="text-sm text-slate-500 mt-1">SMILES解析 · Three.js球棍模型 · ADMET预测 · Tanimoto相似搜索</p>
    </header>
    <div v-if="store.benchmarkMolecule" class="bg-amber-500/10 border border-amber-500/50 mx-4 mt-4 px-4 py-3 rounded-lg flex items-center justify-between">
      <div class="flex items-center gap-2 text-amber-300">
        <span>📌</span>
        <span>基准分子：{{ store.benchmarkMolecule.name }} ({{ store.benchmarkMolecule.formula }})</span>
      </div>
      <div class="flex items-center gap-2">
        <button
          v-if="store.currentMolecule && store.benchmarkMolecule.id !== store.currentMolecule.id"
          @click="store.setBenchmark(store.currentMolecule!)"
          class="px-3 py-1 text-sm bg-amber-500/20 hover:bg-amber-500/30 text-amber-300 border border-amber-500/50 rounded transition-colors"
        >
          切换到当前分子为基准
        </button>
        <button
          @click="store.clearBenchmark()"
          class="px-3 py-1 text-sm bg-amber-500/20 hover:bg-amber-500/30 text-amber-300 border border-amber-500/50 rounded transition-colors"
        >
          取消基准
        </button>
      </div>
    </div>
    <div class="flex flex-col lg:flex-row gap-4 p-4">
      <div class="lg:w-1/4 space-y-4">
        <MoleculeSearch />
      </div>
      <div class="lg:w-1/2 space-y-4">
        <MoleculeViewer />
      </div>
      <div class="lg:w-1/4 space-y-4">
        <AdmetPanel />
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { onMounted } from 'vue'
import { useMoleculeStore } from './store/molecule'
import MoleculeViewer from './components/MoleculeViewer.vue'
import AdmetPanel from './components/AdmetPanel.vue'
import MoleculeSearch from './components/MoleculeSearch.vue'

const store = useMoleculeStore()
onMounted(() => store.loadMolecules())
</script>
