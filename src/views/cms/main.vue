<template>
  <div id="content-wrapper" :class="{ 'sidebar-collapsed': !sidebarVisible }">

    <main id="main-content">
      <router-view :key="key" />
    </main>

    <aside id="info-sidebar" :class="{ 'is-visible': sidebarVisible }">
      <div id="sidebar-content-wrapper">

        <div class="p-6">
          <img id="sidebar-avatar" src="" alt="博主头像"
               class="w-24 h-24 rounded-full mx-auto mb-4 border-2 border-white/50 shadow-md object-cover">

          <h3 id="sidebar-username" class="text-xl font-semibold text-center">
            NiHaiCreeper
          </h3>

          <p id="sidebar-bio" class="text-sm text-zinc-700 dark:text-zinc-300 text-center mt-2 mb-5">
            画画、游戏与日常碎片
          </p>

          <div id="sidebar-stats" class="flex justify-around text-center mb-5">
            <div>
              <span id="sidebar-stat-articles-val" class="block text-xl font-semibold">0</span>
              <span class="text-xs text-zinc-500 dark:text-zinc-400">文章</span>
            </div>
            <div>
              <span id="sidebar-stat-tags-val" class="block text-xl font-semibold">0</span>
              <span class="text-xs text-zinc-500 dark:text-zinc-400">标签</span>
            </div>
            <div>
              <span id="sidebar-stat-posts-val" class="block text-xl font-semibold">0</span>
              <span class="text-xs text-zinc-500 dark:text-zinc-400">讨论</span>
            </div>
          </div>

          <a id="sidebar-follow-btn" href="#"
             class="w-full flex items-center justify-center gap-2 px-4 py-2 rounded-lg text-sm font-medium
                    bg-zinc-800 text-white
                    dark:bg-zinc-100 dark:text-zinc-900
                    hover:bg-zinc-700 dark:hover:bg-zinc-200 transition-colors">
            <i class="el-icon-star-off"></i> Follow Me
          </a>

          <div id="sidebar-socials" class="flex justify-center gap-6 mt-5 text-zinc-500 dark:text-zinc-400">
            <a href="/" class="text-xl hover:text-zinc-800 dark:hover:text-zinc-100" title="主页">
              <i class="el-icon-s-home"></i> </a>
            <a id="sidebar-email" href="#" class="text-xl hover:text-zinc-800 dark:hover:text-zinc-100" title="邮件">
              <i class="el-icon-message"></i> </a>
            <a id="sidebar-github" href="#" class="text-xl hover:text-zinc-800 dark:hover:text-zinc-100" title="GitHub">
              <i class="el-icon-user-solid"></i>
            </a>
          </div>

          <hr class="my-5 dark:border-zinc-700" />
          <h3 class="text-xl font-semibold mb-3">标签</h3>
          <div id="sidebar-tag-list" class="flex flex-wrap gap-2">
          </div>

        </div>
      </div>
    </aside>

  </div>
</template>

<script>
export default {
  name: 'cmsMain',
  // 1. 接收来自 index.vue 的 prop
  props: {
    sidebarVisible: {
      type: Boolean,
      default: true
    }
  },
  computed: {
    key() {
      return this.$route.path
    }
  }
}
</script>

<style>
/* * 1. 桌面端布局 (lg: 1024px 及以上)
 * 模仿 nihaiblog(1) 的 .lg:flex .lg:max-w-7xl .lg:mx-auto .lg:gap-6
 */
@media screen and (min-width: 1024px) {
  #content-wrapper {
    display: flex;
    flex-direction: row;
    max-width: 80rem; /* 1280px (lg:max-w-7xl) */
    margin-left: auto;
    margin-right: auto;
    gap: 1.5rem; /* 24px (lg:gap-6) */

    /* 动画：当侧边栏隐藏时，平滑地缩小最大宽度 */
    transition: max-width 0.3s ease-in-out;
  }

  /* 当侧边栏隐藏时，主容器宽度变小 */
  #content-wrapper.sidebar-collapsed {
    max-width: 64rem; /* 1024px (lg:max-w-5xl) */
  }

  /* 左侧主内容 */
  #main-content {
    flex: 1; /* 占据剩余空间 */
    min-width: 0; /* 允许 flex 缩小 */
  }

  /* 右侧博主信息栏 */
  #info-sidebar {
    /* 默认 (隐藏) 状态 */
    width: 0;
    opacity: 0;
    overflow: hidden;
    /* 动画：模仿 nihaiblog(1) 的 transition-all duration-300 */
    transition: all 0.3s ease-in-out;
  }

  /* 激活 (显示) 状态 */
  #info-sidebar.is-visible {
    width: 20rem; /* 320px (w-80) */
    opacity: 1;
  }

  /* 内层容器始终保持宽度，防止内容在动画中变形 */
  #sidebar-content-wrapper {
    width: 20rem; /* 320px */
  }
}

/* * 2. 移动端布局 (1023px 及以下)
 * 模仿 nihaiblog(1) 的默认 (mobile-first) 堆叠布局
 */
@media screen and (max-width: 1023px) {
  #content-wrapper {
    display: flex;
    flex-direction: column; /* 垂直堆叠 */
    width: 100%;
    gap: 0;
    /* 确保在移动端也有内边距 */
    padding-left: 1rem;
    padding-right: 1rem;
    box-sizing: border-box; /* 确保 padding 不会撑开宽度 */
  }

  #main-content {
    width: 100%;
  }

  /* 在移动端，侧边栏始终可见且占满宽度 */
  #info-sidebar {
    width: 100% !important;
    opacity: 1 !important;
  }

  #sidebar-content-wrapper {
    width: 100%;
  }
}
</style>
