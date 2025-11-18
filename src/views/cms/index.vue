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
    <cms-main />
    <cms-footer />
    <div id="fab-container" class="fixed bottom-6 right-6 z-50 flex flex-col items-center gap-3">
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
import cmsMain from './main'
import cmsFooter from './Footer'
import {
  cmsListBlog
} from '@/api/cms/blog'
import backgroundUrl from '@/assets/images/wallhaven-dgojvj.webp'

export default {
  name: 'Cms',
  components: {
    NavBar,
    cmsMain,
    cmsFooter
  },
  data() {
    // ... 您的 data 不变 ...
    return {
      backgroundUrl,
      // ...
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
      // 确定新主题
      const newTheme = this.sideTheme === 'theme-dark' ? 'theme-light' : 'theme-dark';

      // 5. 提交 Vuex 状态变更
      this.changeSetting({ key: 'sideTheme', value: newTheme });

      // 6. 切换 CSS 所需的 'dark' 类
      if (newTheme === 'theme-dark') {
        document.documentElement.classList.add('dark');
      } else {
        document.documentElement.classList.remove('dark');
      }
    },
    handleScroll() {
      // 确保 headerEl 已经被获取
      if (!this.headerEl) return;

      const currentScrollY = window.scrollY;

      if (currentScrollY > this.lastScrollY && currentScrollY > this.hideTriggerPoint) {
        // 1. 向下滚动 且 滚过了封面的一半 -> 隐藏
        this.headerEl.classList.add('header-hidden');

      } else if (currentScrollY < this.lastScrollY) {
        // 2. 向上滚动 (无论在何处) -> 显示
        // (这符合您 "在内容以下的时候向上滚动它就会出来" 的要求)
        this.headerEl.classList.remove('header-hidden');

      } else if (currentScrollY <= this.hideTriggerPoint) {
        // 3. 在封面顶部区域 (0 到 封面一半) -> 始终显示
        this.headerEl.classList.remove('header-hidden');
      }

      // 更新最后滚动位置 (防止负值)
      this.lastScrollY = currentScrollY <= 0 ? 0 : currentScrollY;
    }
  },
  mounted() {

    this.$nextTick(() => {
      const themeToggle = document.getElementById('theme-toggle');
      const fabToggle = document.getElementById('fab-toggle'); // 主齿轮按钮
      const fabContainer = document.getElementById('fab-container'); // 容器

      if (themeToggle) {
        themeToggle.addEventListener('click', this.toggleCmsTheme);
      }

      // nihaiblog(1) 中用于展开/折叠 FAB 的逻辑
      if (fabToggle && fabContainer) {
        fabToggle.addEventListener('click', () => {
          fabContainer.classList.toggle('is-active');
        });

        // (可选) 滚动时显示 FAB
        window.addEventListener('scroll', () => {
          if (window.scrollY > 300) {
            fabContainer.classList.add('fab-visible');
          } else {
            fabContainer.classList.remove('fab-visible');
          }
        });
      }
      // --- START: 新增滚动监听逻辑 ---

      // 1. 获取导航栏和封面元素
      this.headerEl = document.getElementById('main-header');
      const coverEl = document.getElementById('fullpage-cover');

      if (coverEl && this.headerEl) {
        // 2. 计算并存储封面高度和触发点
        this.coverHeight = coverEl.offsetHeight;
        // 触发点为封面高度的一半
        this.hideTriggerPoint = this.coverHeight / 2;
      }

      // 3. 绑定滚动事件
      window.addEventListener('scroll', this.handleScroll);
      // --- END: 新增滚动监听逻辑 ---

    });


    // 8. 页面加载时，根据 Vuex 状态初始化 'dark' 类
    if (this.sideTheme === 'theme-dark') {
      document.documentElement.classList.add('dark');
    } else {
      document.documentElement.classList.remove('dark');
    }
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
</style>
