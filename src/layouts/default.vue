<script setup lang="ts">
import { useTheme } from 'vuetify'

const theme = useTheme()

const toggleTheme = () => {
  if (isDark.value) {
    // 恢复亮色主题
    localStorage.setItem('diyfile-theme', 'light')
    theme.global.name.value = 'light'
  } else {
    // 设置为暗黑主题
    localStorage.setItem('diyfile-theme', 'dark')
    theme.global.name.value = 'dark'
  }
  toggleDark()
}
</script>

<template>
  <div>
    <n-layout position="absolute">
      <n-layout-header style="height: 60px" bordered>
        <Header @toggleTheme="toggleTheme" />
      </n-layout-header>
      <n-layout style="height: calc(100% - 120px)">
        <n-layout-content content-style="padding: 0.5rem;">
          <RouterView />
        </n-layout-content>
      </n-layout>
      <n-layout-footer style="height: 60px" bordered>
        <Footer />
      </n-layout-footer>
    </n-layout>
  </div>
</template>
