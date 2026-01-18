<script setup lang="ts">
import { useRouteCacheStore } from '@/stores'
import { computed, onMounted, ref, nextTick } from 'vue'

useHead({
  meta: [
    {
      name: 'description',
      content: 'Air724 管理面板',
    },
    {
      name: 'theme-color',
      content: () => isDark.value ? '#00aba9' : '#ffffff',
    },
  ],
  link: [
    {
      rel: 'icon',
      type: 'image/svg+xml',
      href: () => preferredDark.value ? '/favicon-dark.svg' : '/favicon.svg',
    },
  ],
})

const routeCacheStore = useRouteCacheStore()
const appWrapperRef = ref<HTMLElement>()

const keepAliveRouteNames = computed(() => {
  return routeCacheStore.routeCaches
})

const mode = computed(() => {
  return isDark.value ? 'dark' : 'light'
})

// 更新容器高度
function updateContainerHeights() {
  const appWrapper = appWrapperRef.value
  if (!appWrapper) return
  
  const navBar = document.querySelector('.van-nav-bar') as HTMLElement
  const tabBar = document.querySelector('.van-tabbar') as HTMLElement
  
  if (navBar && tabBar) {
    const navHeight = navBar.offsetHeight
    const tabHeight = tabBar.offsetHeight
    const availableHeight = window.innerHeight - navHeight - tabHeight
    
    // 设置CSS变量供子组件使用
    document.documentElement.style.setProperty('--nav-height', `${navHeight}px`)
    document.documentElement.style.setProperty('--tab-height', `${tabHeight}px`)
    
    // 设置.app-wrapper高度
    appWrapper.style.height = `${availableHeight}px`
    appWrapper.style.minHeight = `${availableHeight}px`
    
    // 解除所有子容器的高度限制
    const childContainers = appWrapper.querySelectorAll(
      '.device-list, .van-pull-refresh__track, .van-pull-refresh, .van-cell-group, .van-cell-group--inset'
    )
    
    childContainers.forEach(container => {
      const el = container as HTMLElement
      el.style.minHeight = '100%'
      el.style.height = 'auto'
      el.style.maxHeight = 'none'
    })
  }
}

onMounted(() => {
  nextTick(() => {
    updateContainerHeights()
    
    // 监听窗口大小变化
    window.addEventListener('resize', updateContainerHeights)
    
    // 监听DOM变化，确保动态内容也能正确计算
    const observer = new MutationObserver(() => {
      updateContainerHeights()
    })
    
    if (appWrapperRef.value) {
      observer.observe(appWrapperRef.value, {
        childList: true,
        subtree: true
      })
    }
  })
})
</script>

<template>
  <van-config-provider :theme="mode">
    <nav-bar />
    <router-view v-slot="{ Component }">
      <section class="app-wrapper" ref="appWrapperRef">
        <keep-alive :include="keepAliveRouteNames">
          <component :is="Component" />
        </keep-alive>
      </section>
    </router-view>
    <tab-bar />
  </van-config-provider>
</template>

<style scoped>
.app-wrapper {
  position: fixed;
  top: var(--nav-height, 74px); /* 使用动态高度 */
  bottom: var(--tab-height, 80px); /* 使用动态高度 */
  left: 0;
  right: 0;
  padding: 16px;
  overflow-y: auto;
  overflow-x: hidden;
  -webkit-overflow-scrolling: touch;
  box-sizing: border-box;
  background: var(--van-background-color);
  z-index: 1;
}

/* 响应式高度计算 */
@media screen and (max-width: 768px) {
  .app-wrapper {
    top: var(--nav-height, 74px);
    bottom: calc(var(--tab-height, 80px) + env(safe-area-inset-bottom, 0px));
  }
}
</style>

<style>
/* 全局样式：解除所有可能的高度限制 */
:root {
  --nav-height: 74px;
  --tab-height: 80px;
}

/* 穿透到子组件，确保设备列表可以完全展开 */
.device-list,
.van-pull-refresh__track,
.van-pull-refresh,
.van-cell-group,
.van-cell-group--inset {
  min-height: 100% !important;
  height: auto !important;
  max-height: none !important;
}

/* 确保设备列表项正常显示 */
.van-cell {
  min-height: 54px !important;
}

/* 优化移动端滚动体验 */
@media (hover: none) and (pointer: coarse) {
  .app-wrapper {
    -webkit-overflow-scrolling: touch !important;
    scroll-behavior: smooth !important;
  }
}
</style>