<template>
  <div class="bg-slate-800 rounded-lg p-4 border border-slate-700">
    <div v-if="store.benchmarkMolecule" class="mb-3 flex items-center justify-between bg-orange-900/30 border border-orange-700/50 rounded px-3 py-2">
      <div class="flex items-center gap-2">
        <span class="text-orange-400">⇄</span>
        <span class="text-xs text-slate-300">正在对比基准：</span>
        <span class="text-sm font-bold text-orange-300">{{ store.benchmarkMolecule.name }}</span>
      </div>
      <button
        @click="store.clearBenchmark()"
        class="text-xs px-2 py-1 rounded bg-slate-700 hover:bg-slate-600 text-slate-300 transition"
      >
        取消基准
      </button>
    </div>

    <h3 class="text-sm font-bold text-slate-400 mb-3">ADMET 性质预测</h3>

    <div v-if="store.admet" class="space-y-3">
      <div class="grid grid-cols-2 gap-2 text-sm">
        <div class="bg-slate-900 rounded p-2">
          <div class="text-xs text-slate-500">LogP (脂溶性)</div>
          <div v-if="!store.benchmarkAdmet" class="text-lg font-bold" :class="store.admet.logP > 3 ? 'text-red-400' : 'text-green-400'">{{ store.admet.logP }}</div>
          <div v-else class="space-y-1">
            <div class="flex items-center gap-1">
              <span class="text-xs text-slate-500 w-8">基准</span>
              <span class="text-sm font-medium text-orange-400">{{ store.benchmarkAdmet.logP }}</span>
            </div>
            <div class="flex items-center gap-1">
              <span class="text-xs text-slate-500 w-8">当前</span>
              <span class="text-lg font-bold" :class="store.admet.logP > 3 ? 'text-red-400' : 'text-green-400'">{{ store.admet.logP }}</span>
              <span v-if="diffValue(store.admet.logP, store.benchmarkAdmet.logP) !== 0" class="text-xs font-bold" :class="diffValue(store.admet.logP, store.benchmarkAdmet.logP) > 0 ? 'text-green-400' : 'text-red-400'">
                {{ formatDiff(diffValue(store.admet.logP, store.benchmarkAdmet.logP)) }}
              </span>
            </div>
          </div>
        </div>

        <div class="bg-slate-900 rounded p-2">
          <div class="text-xs text-slate-500">LogS (溶解度)</div>
          <div v-if="!store.benchmarkAdmet" class="text-lg font-bold text-cyan-400">{{ store.admet.logS }}</div>
          <div v-else class="space-y-1">
            <div class="flex items-center gap-1">
              <span class="text-xs text-slate-500 w-8">基准</span>
              <span class="text-sm font-medium text-orange-400">{{ store.benchmarkAdmet.logS }}</span>
            </div>
            <div class="flex items-center gap-1">
              <span class="text-xs text-slate-500 w-8">当前</span>
              <span class="text-lg font-bold text-cyan-400">{{ store.admet.logS }}</span>
              <span v-if="diffValue(store.admet.logS, store.benchmarkAdmet.logS) !== 0" class="text-xs font-bold" :class="diffValue(store.admet.logS, store.benchmarkAdmet.logS) > 0 ? 'text-green-400' : 'text-red-400'">
                {{ formatDiff(diffValue(store.admet.logS, store.benchmarkAdmet.logS)) }}
              </span>
            </div>
          </div>
        </div>

        <div class="bg-slate-900 rounded p-2">
          <div class="text-xs text-slate-500">蛋白结合率</div>
          <div v-if="!store.benchmarkAdmet" class="text-lg font-bold text-purple-400">{{ store.admet.proteinBinding }}%</div>
          <div v-else class="space-y-1">
            <div class="flex items-center gap-1">
              <span class="text-xs text-slate-500 w-8">基准</span>
              <span class="text-sm font-medium text-orange-400">{{ store.benchmarkAdmet.proteinBinding }}%</span>
            </div>
            <div class="flex items-center gap-1">
              <span class="text-xs text-slate-500 w-8">当前</span>
              <span class="text-lg font-bold text-purple-400">{{ store.admet.proteinBinding }}%</span>
              <span v-if="diffValue(store.admet.proteinBinding, store.benchmarkAdmet.proteinBinding) !== 0" class="text-xs font-bold" :class="diffValue(store.admet.proteinBinding, store.benchmarkAdmet.proteinBinding) > 0 ? 'text-green-400' : 'text-red-400'">
                {{ formatDiff(diffValue(store.admet.proteinBinding, store.benchmarkAdmet.proteinBinding)) }}%
              </span>
            </div>
          </div>
        </div>

        <div class="bg-slate-900 rounded p-2">
          <div class="text-xs text-slate-500">生物利用度</div>
          <div v-if="!store.benchmarkAdmet" class="text-lg font-bold" :class="store.admet.bioavailability > 50 ? 'text-green-400' : 'text-orange-400'">{{ store.admet.bioavailability }}%</div>
          <div v-else class="space-y-1">
            <div class="flex items-center gap-1">
              <span class="text-xs text-slate-500 w-8">基准</span>
              <span class="text-sm font-medium text-orange-400">{{ store.benchmarkAdmet.bioavailability }}%</span>
            </div>
            <div class="flex items-center gap-1">
              <span class="text-xs text-slate-500 w-8">当前</span>
              <span class="text-lg font-bold" :class="store.admet.bioavailability > 50 ? 'text-green-400' : 'text-orange-400'">{{ store.admet.bioavailability }}%</span>
              <span v-if="diffValue(store.admet.bioavailability, store.benchmarkAdmet.bioavailability) !== 0" class="text-xs font-bold" :class="diffValue(store.admet.bioavailability, store.benchmarkAdmet.bioavailability) > 0 ? 'text-green-400' : 'text-red-400'">
                {{ formatDiff(diffValue(store.admet.bioavailability, store.benchmarkAdmet.bioavailability)) }}%
              </span>
            </div>
          </div>
        </div>
      </div>

      <div class="bg-slate-900 rounded p-3 text-sm">
        <div class="flex justify-between items-center mb-2">
          <span class="text-slate-500">毒性评估</span>
          <div class="flex items-center gap-2">
            <span v-if="store.benchmarkAdmet" class="text-xs font-medium text-orange-400">[{{ store.benchmarkAdmet.toxicity }}]</span>
            <span :class="store.admet.toxicity.includes('高') ? 'text-red-400' : store.admet.toxicity.includes('中') ? 'text-orange-400' : 'text-green-400'">{{ store.admet.toxicity }}</span>
          </div>
        </div>
        <div class="flex justify-between items-center mb-2">
          <span class="text-slate-500">代谢稳定性</span>
          <div class="flex items-center gap-2">
            <span v-if="store.benchmarkAdmet" class="text-xs font-medium text-orange-400">[{{ store.benchmarkAdmet.metabolicStability }}]</span>
            <span :class="store.admet.metabolicStability === '稳定' ? 'text-green-400' : store.admet.metabolicStability === '中等' ? 'text-orange-400' : 'text-red-400'">{{ store.admet.metabolicStability }}</span>
          </div>
        </div>
        <div class="flex justify-between items-center">
          <span class="text-slate-500">Lipinski五规则</span>
          <div class="flex items-center gap-2">
            <span v-if="store.benchmarkAdmet" class="text-xs font-medium text-orange-400">
              [{{ store.benchmarkAdmet.ruleOfFive ? '✓ 通过' : '✗ 违反' }} ({{ store.benchmarkAdmet.violations }})]
            </span>
            <span :class="store.admet.ruleOfFive ? 'text-green-400' : 'text-red-400'">{{ store.admet.ruleOfFive ? '✓ 通过' : '✗ 违反' }} ({{ store.admet.violations }})</span>
          </div>
        </div>
      </div>
    </div>
    <div v-else class="text-slate-500 text-sm">选择分子查看ADMET</div>

    <div v-if="store.similarMolecules.length > 0" class="mt-4">
      <h4 class="text-xs font-bold text-slate-500 mb-2">
        相似分子 (Tanimoto)
        <span v-if="store.benchmarkMolecule" class="ml-1 text-orange-400/70">· 含基准对比</span>
      </h4>
      <div class="space-y-1">
        <div
          v-for="mol in similarMoleculesWithBenchmark"
          :key="mol.id"
          @click="store.selectMolecule(mol)"
          class="cursor-pointer flex items-center justify-between bg-slate-900 rounded p-2 hover:bg-slate-700 transition"
        >
          <span class="text-sm text-slate-200">{{ mol.name }}</span>
          <div class="flex items-center gap-2">
            <span v-if="mol.benchmarkSimilarity !== undefined" class="text-xs text-orange-400/80">基准{{ mol.benchmarkSimilarity }}%</span>
            <span class="text-xs font-bold" :style="{ color: mol.similarity > 60 ? '#22c55e' : mol.similarity > 30 ? '#eab308' : '#94a3b8' }">{{ mol.similarity }}%</span>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { computed } from 'vue'
import { useMoleculeStore } from '../store/molecule'

const store = useMoleculeStore()

function diffValue(current: number, benchmark: number): number {
  return Math.round((current - benchmark) * 100) / 100
}

function formatDiff(diff: number): string {
  if (diff > 0) return `+${diff}`
  return `${diff}`
}

const similarMoleculesWithBenchmark = computed(() => {
  if (!store.benchmarkMolecule) {
    return store.similarMolecules.map(m => ({ ...m, benchmarkSimilarity: undefined as number | undefined }))
  }
  return store.similarMolecules.map(mol => {
    const benchmarkSimilarity = store.computeTanimoto(store.benchmarkMolecule!.smiles, mol.smiles)
    return { ...mol, benchmarkSimilarity }
  })
})
</script>
