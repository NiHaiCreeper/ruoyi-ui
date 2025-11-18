<template>
  <div class="cms-main-container">
    <el-row :gutter="20">
      <el-col :span="mainSpan" class="main-content-col">
        <transition name="fade-transform" mode="out-in">
          <keep-alive :include="cachedViews">
            <router-view :key="key" />
          </keep-alive>
        </transition>
      </el-col>

      <el-col :span="sidebarSpan" class="right-sidebar-col" v-if="isSidebarVisible">
        <BloggerInfo @toggle-sidebar="toggleSidebar" />
      </el-col>
    </el-row>

    <div class="sidebar-toggle-hidden" v-if="!isSidebarVisible" @click="toggleSidebar" title="展开侧边栏">
      <i class="el-icon-arrow-left"></i>
    </div>
  </div>
</template>

<script>
import BloggerInfo from './components/BloggerInfo.vue';

export default {
  name: "CmsMain",
  components: { BloggerInfo },
  data() {
    return {
      isSidebarVisible: true,
      sidebarSpanBase: 6,
      mainSpanBase: 18,
    };
  },
  computed: {
    cachedViews() {
      // 如果您使用了 tagsView，这里可以连接 store 中的 cachedViews
      return this.$store.state.tagsView ? this.$store.state.tagsView.cachedViews : []
    },
    key() {
      return this.$route.path
    },
    mainSpan() {
      return this.isSidebarVisible ? this.mainSpanBase : 24;
    },
    sidebarSpan() {
      return this.isSidebarVisible ? this.sidebarSpanBase : 0;
    }
  },
  methods: {
    toggleSidebar() {
      this.isSidebarVisible = !this.isSidebarVisible;
    }
  }
};
</script>

<style lang="scss" scoped>
/* 保持之前的样式不变 */
.cms-main-container { position: relative; }
.main-content-col, .right-sidebar-col { transition: all 0.3s ease-in-out; }
.right-sidebar-col { overflow: hidden; }
.sidebar-toggle-hidden {
  position: fixed; top: 50%; right: 0; transform: translateY(-50%);
  width: 25px; height: 50px; background-color: #f5f7fa;
  border: 1px solid #e4e7ed; border-right: none; border-radius: 5px 0 0 5px;
  display: flex; align-items: center; justify-content: center;
  cursor: pointer; z-index: 100; box-shadow: -2px 0 5px rgba(0,0,0,0.1);
  &:hover { background-color: #eef2f8; }
}
</style>
