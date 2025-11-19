<template>
  <div class="cms-container" :style="'background-image: url(' + backgroundUrl + ')'">

    <section id="fullpage-cover" class="h-screen flex flex-col items-center justify-center p-6 text-center relative">
      <h1 class="text-4xl md:text-5xl font-bold text-white" id="hero-title" style="text-shadow: 0 2px 4px rgba(0,0,0,0.5);">NiHai 的小站</h1>
      <p class="mt-3 text-lg md:text-xl text-white" id="hero-subtitle" style="text-shadow: 0 1px 3px rgba(0,0,0,0.5);">画画、游戏与日常碎片</p>
      <hr class="my-5 border-white/30 w-1/4" />
      <p class="text-base max-w-lg text-white/90" id="site-bio" style="text-shadow: 0 1px 3px rgba(0,0,0,0.5);">我是 NiHai...</p>

      <div class="absolute bottom-4 animate-bounce">
        <a href="#content-start" aria-label="向下滚动" class="text-2xl text-white/70 hover:text-white">
          <i class="fa fa-angle-down"></i>
        </a>
      </div>
    </section>

    <div id="content-start" class="relative -top-16"></div>
    <nav-bar />
    <router-view />
    <cms-footer/>
    <div id="fab-container" class="fixed bottom-6 right-6 z-50 flex flex-col items-center gap-3 fab-hidden">
      <div id="fab-actions" class="flex flex-col items-center gap-3 transition-all duration-300 opacity-0 transform scale-90 -translate-y-2 visibility-hidden">
        <button id="theme-toggle" class="fab-button bg-white dark:bg-zinc-700 text-zinc-900 dark:text-zinc-100 w-12 h-12 rounded-full shadow-lg flex items-center justify-center" aria-label="toggle theme">🌙</button>
        <a id="rss-link" href="#" class="fab-button bg-white dark:bg-zinc-700 text-zinc-900 dark:text-zinc-100 w-12 h-12 rounded-full shadow-lg flex items-center justify-center" aria-label="rss">📡</a>
        <a id="github-link" href="#" class="fab-button bg-white dark:bg-zinc-700 text-zinc-900 dark:text-zinc-100 w-12 h-12 rounded-full shadow-lg flex items-center justify-center" aria-label="github">🐙</a>
        <a id="email-link" href="#" class="fab-button bg-white dark:bg-zinc-700 text-zinc-900 dark:text-zinc-100 w-12 h-12 rounded-full shadow-lg flex items-center justify-center" aria-label="email">✉️</a>
      </div>

      <button id="fab-toggle" class="fab-button bg-blue-600 text-white w-14 h-14 rounded-full shadow-xl flex items-center justify-center transition-transform duration-200 transform hover:scale-110 active:scale-100">
        <i class="fa fa-cog text-2xl transition-transform duration-300"></i>
      </button>
    </div>
  </div>
</template>

<script>
// 1. 导入 vuex 辅助函数
import { mapState, mapActions } from 'vuex'
import NavBar from './NavBar'

import cmsFooter from './Footer'
import {
  cmsListBlog
} from '@/api/cms/blog'
import backgroundUrl from '@/assets/images/wallhaven-dgojvj.webp'

export default {
  name: 'Cms',
  components: {
    NavBar,
    cmsFooter
  },
  data() {
    return {
      backgroundUrl,
      isFabVisible: false,
      showThreshold: 300,
      hideThreshold: 220,
      boundFabToggleClick: null,
      boundFabScrollHandler: null,
      boundThemeToggleClick: null
    }
  },
  computed: {
    // 2. 监听 settings 模块中的 sideTheme
    ...mapState('settings', ['sideTheme'])
  },
  methods: {
    // 3. 引入 changeSetting 动作
    ...mapActions('settings', ['changeSetting']),

    // 4. 创建新的切换方法
    toggleCmsTheme() {
      const newTheme = this.sideTheme === 'theme-dark' ? 'theme-light' : 'theme-dark';
      this.changeSetting({ key: 'sideTheme', value: newTheme });
      if (newTheme === 'theme-dark') {
        document.documentElement.classList.add('dark');
      } else {
        document.documentElement.classList.remove('dark');
      }
      this.updateThemeToggleUI(newTheme);
    },
    updateThemeToggleUI(theme = this.sideTheme) {
      const btn = document.getElementById('theme-toggle');
      if (!btn) return;
      const isDark = theme === 'theme-dark';
      btn.textContent = isDark ? '☀️' : '🌙';
      btn.setAttribute('aria-label', isDark ? '切换为浅色主题' : '切换为暗色主题');
      btn.setAttribute('title', isDark ? '切换为浅色主题' : '切换为暗色主题');
    },


  },
  watch: {
    sideTheme(newTheme) {
      if (newTheme === 'theme-dark') {
        document.documentElement.classList.add('dark');
      } else {
        document.documentElement.classList.remove('dark');
      }
      this.updateThemeToggleUI(newTheme);
    }
  },
  mounted() {

    this.$nextTick(() => {
      const themeToggle = document.getElementById('theme-toggle');
      const fabToggle = document.getElementById('fab-toggle'); // 主齿轮按钮
      const fabContainer = document.getElementById('fab-container'); // 容器

      if (themeToggle) {
        this.boundThemeToggleClick = this.toggleCmsTheme;
        themeToggle.addEventListener('click', this.boundThemeToggleClick);
        this.updateThemeToggleUI(this.sideTheme);
      }

      // nihaiblog(1) 中用于展开/折叠 FAB 的逻辑
      if (fabToggle && fabContainer) {
        this.boundFabToggleClick = () => {
          fabContainer.classList.toggle('is-active');
        };
        fabToggle.addEventListener('click', this.boundFabToggleClick);

        // 初始化为收起状态
        fabContainer.classList.add('fab-hidden');
        this.isFabVisible = false;

        // 滚动时显示/收起 FAB（带缩回动画，含阈值回差）
        this.boundFabScrollHandler = () => {
          const y = window.scrollY;
          if (!this.isFabVisible && y >= this.showThreshold) {
            fabContainer.classList.add('fab-visible');
            fabContainer.classList.remove('fab-hidden');
            this.isFabVisible = true;
          } else if (this.isFabVisible && y <= this.hideThreshold) {
            fabContainer.classList.add('fab-hidden');
            fabContainer.classList.remove('fab-visible');
            this.isFabVisible = false;
          }
        };
        window.addEventListener('scroll', this.boundFabScrollHandler, { passive: true });

        // 初始可见性校准（根据当前滚动位置）
        {
          const y0 = window.scrollY;
          if (y0 >= this.showThreshold) {
            fabContainer.classList.add('fab-visible');
            fabContainer.classList.remove('fab-hidden');
            this.isFabVisible = true;
          } else {
            fabContainer.classList.add('fab-hidden');
            fabContainer.classList.remove('fab-visible');
            this.isFabVisible = false;
          }
        }
      }
      // 导航栏滚动隐藏逻辑由 NavBar.vue 负责

    });


    // 8. 页面加载时，根据 Vuex 状态初始化 'dark' 类
    if (this.sideTheme === 'theme-dark') {
      document.documentElement.classList.add('dark');
    } else {
      document.documentElement.classList.remove('dark');
    }
    this.updateThemeToggleUI(this.sideTheme);
  },
  beforeDestroy() {
    const themeToggle = document.getElementById('theme-toggle');
    const fabToggle = document.getElementById('fab-toggle');
    if (themeToggle && this.boundThemeToggleClick) {
      themeToggle.removeEventListener('click', this.boundThemeToggleClick);
      this.boundThemeToggleClick = null;
    }
    if (fabToggle && this.boundFabToggleClick) {
      fabToggle.removeEventListener('click', this.boundFabToggleClick);
      this.boundFabToggleClick = null;
    }
    if (this.boundFabScrollHandler) {
      window.removeEventListener('scroll', this.boundFabScrollHandler);
      this.boundFabScrollHandler = null;
    }
    // 导航栏滚动事件由 NavBar.vue 管理，无需在此移除
  }
}
</script>

<style scoped>
  .el-footer {
    background-color: rgba(84, 92, 100, 0.5);
  }

  .background {
    background-repeat: no-repeat;
    background-size: cover;
    margin: 0px;
    padding: 0px;
    top: 0;
    width: 100%;
    height: 120vh;
    position: fixed;
    z-index:-1;
  }

  @media screen and (max-width: 768px) {
    .title {
      width: 100%;
      background-position-x: center;
      background-position-y: 0;
    }
  }
#fab-container { transition: transform 280ms cubic-bezier(0.22, 1, 0.36, 1), opacity 220ms ease-out; will-change: transform, opacity; }
#fab-container.fab-visible { opacity: 1; transform: translateY(0) scale(1); }
#fab-container.fab-hidden { opacity: 0; transform: translateY(24px) scale(0.85); pointer-events: none; }
#fab-actions { transition: transform 220ms ease-out, opacity 220ms ease-out; opacity: 0; transform: translateY(8px) scale(0.95); will-change: transform, opacity; }
#fab-container.is-active #fab-actions { opacity: 1; transform: translateY(-8px) scale(1); }
</style>
