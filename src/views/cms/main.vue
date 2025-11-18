<template>
  <router-view :key="key" />
</template>

<script>
export default {
  name: 'cmsMain',
  // props 已被移除
  computed: {
    key() {
      return this.$route.path
    }
  }
}
</script>

<style>
/* * 1. 桌面端布局 ... */
@media screen and (min-width: 1024px) {
  /* ... 大量的 #content-wrapper 和 #info-sidebar 样式 ... */
}

/* * 2. 移动端布局 ... */
@media screen and (max-width: 1023px) {
  /* ... 大量的 #content-wrapper 和 #info-sidebar 样式 ... */
}
</style>
