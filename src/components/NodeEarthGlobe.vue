<script setup lang="ts">
import type { NodeData } from '@/stores/nodes'
import { computed, defineAsyncComponent, useAttrs } from 'vue'
import { useAppStore } from '@/stores/app'

const props = defineProps<{
  nodes?: NodeData[]
}>()

const attrs = useAttrs()
const appStore = useAppStore()

const NodeEarthCobeGlobe = defineAsyncComponent(() => import('@/components/NodeEarthCobeGlobe.vue'))
const NodeEarthRealisticGlobe = defineAsyncComponent(() => import('@/components/NodeEarthRealisticGlobe.vue'))
const NodeEarthTiledMap = defineAsyncComponent(() => import('@/components/NodeEarthTiledMap.vue'))

// WebGL 不可用(部分手机端浏览器/省流量模式)时 realistic/cobe 会渲染空白,
// 自动降级为纯图片的 tiled 平铺地图, 保证顶部不出现空白块
const webglSupported = (() => {
  try {
    const c = document.createElement('canvas')
    return !!(c.getContext('webgl') || c.getContext('experimental-webgl'))
  }
  catch { return false }
})()

const earthComponent = computed(() => {
  const components = {
    realistic: NodeEarthRealisticGlobe,
    cobe: NodeEarthCobeGlobe,
    tiled: NodeEarthTiledMap,
  }
  const renderer = webglSupported ? appStore.earthRenderer : 'tiled'
  return components[renderer] ?? NodeEarthRealisticGlobe
})
</script>

<template>
  <component :is="earthComponent" v-bind="attrs" :nodes="props.nodes" />
</template>
