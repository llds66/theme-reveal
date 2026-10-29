<script setup lang="ts">
import { ref } from 'vue'

type Theme = 'light' | 'dark'

const theme = ref<Theme>('light')

async function toggleTheme(event: MouseEvent) {
  const nextTheme = theme.value === 'light' ? 'dark' : 'light'

  const updateTheme = () => {
    theme.value = nextTheme
    document.documentElement.dataset.theme = nextTheme
    document.documentElement.style.colorScheme = nextTheme
  }

  // 浏览器不支持 View Transitions 时直接切换
  if (
    !document.startViewTransition ||
    window.matchMedia('(prefers-reduced-motion: reduce)').matches
  ) {
    updateTheme()
    return
  }

  // 鼠标点击坐标
  const x = event.clientX
  const y = event.clientY

  // 计算圆形覆盖整个屏幕所需的半径
  const endRadius = Math.hypot(
    Math.max(x, window.innerWidth - x),
    Math.max(y, window.innerHeight - y),
  )

  // 开始视图过渡
  const transition = document.startViewTransition(updateTheme)

  try {
    await transition.ready

    // 从点击位置向外扩散新主题
    await document.documentElement.animate(
      {
        clipPath: [`circle(0px at ${x}px ${y}px)`, `circle(${endRadius}px at ${x}px ${y}px)`],
        // 从上到下
        // clipPath: ['inset(0 0 100% 0)','inset(0 0 0 0)'],
        // 从下到上
        // clipPath: ['inset(100% 0 0 0)','inset(0 0 0 0)'],
        // 从左到右
        // clipPath: ['inset(0 100% 0 0)','inset(0 0 0 0)'],
        // 从右到左
        // clipPath: ['inset(0 0 0 100%)', 'inset(0 0 0 0)'],
        // 对角线扩散
        // clipPath: ['polygon(0 0, 0 0, 0 0)','polygon(0 0, 200% 0, 0 200%)'],
        // 从中心向两侧展开
        // clipPath: ['inset(0 50% 0 50%)', 'inset(0 0 0 0)']
      },
      {
        duration: 650,
        easing: 'ease-in-out',
        pseudoElement: '::view-transition-new(root)',
      },
    ).finished
  } catch {
    // 过渡被跳过或取消时，主题已经更新
  }
}
</script>

<template>
  <main class="flex min-h-dvh cursor-default items-center justify-center bg-app-bg text-app-fg" @click="toggleTheme">
    <h1 class="select-none font-mono text-xl font-semibold tracking-tight">
      单击页面
    </h1>
  </main>
</template>