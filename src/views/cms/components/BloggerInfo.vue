<template>
  <div class="sidebar-wrapper">
    <el-card class="blogger-info-card" shadow="hover">
      <div class="sidebar-toggle-internal" @click="$emit('toggle-sidebar')" title="收起侧边栏">
        <i class="el-icon-arrow-right"></i>
      </div>
      <div class="blogger-profile">
        <div class="blogger-avatar">
          <img :src="blogger.avatar" alt="avatar" />
        </div>
        <div class="blogger-name">{{ blogger.name }}</div>
        <div class="blogger-bio">{{ blogger.bio }}</div>
      </div>
      <el-divider class="info-divider"></el-divider>
      <div class="blogger-stats">
        <div class="stat-item" @click="handleGoTo('blog')">
          <div class="stat-count">{{ blogger.stats.articles }}</div>
          <div class="stat-label">文章</div>
        </div>
        <div class="stat-item" @click="handleGoTo('type')">
          <div class="stat-count">{{ blogger.stats.categories }}</div>
          <div class="stat-label">分类</div>
        </div>
        <div class="stat-item" @click="handleGoTo('tag')">
          <div class="stat-count">{{ blogger.stats.tags }}</div>
          <div class="stat-label">标签</div>
        </div>
      </div>
    </el-card>

    <div class="original-sidebar-content">

      <el-card class="sidebar-card" shadow="hover">
        <div slot="header" class="card-header">
          <span><i class="el-icon-folder-opened"></i> 分类</span>
          <span class="more" @click="handleGoTo('type')">更多 >></span>
        </div>
        <ul class="category-list">
          <li v-for="item in typeList" :key="item.typeId" @click="handleFilter('typeId', item.typeId)">
            <span class="cate-name">{{ item.typeName }}</span>
          </li>
          <div v-if="typeList.length === 0" class="empty-text">暂无分类</div>
        </ul>
      </el-card>

      <el-card class="sidebar-card" shadow="hover">
        <div slot="header" class="card-header">
          <span><i class="el-icon-collection-tag"></i> 标签</span>
          <span class="more" @click="handleGoTo('tag')">更多 >></span>
        </div>
        <div class="tag-cloud">
          <el-tag
            v-for="item in tagList"
            :key="item.tagId"
            size="small"
            class="tag-item"
            :type="randomTagType()"
            @click="handleFilter('tagId', item.tagId)"
          >
            {{ item.tagName }}
          </el-tag>
          <div v-if="tagList.length === 0" class="empty-text">暂无标签</div>
        </div>
      </el-card>

      <el-card class="sidebar-card" shadow="hover">
        <div slot="header" class="card-header">
          <span><i class="el-icon-trophy"></i> 最新推荐</span>
        </div>
        <ul class="recommend-list">
          <li v-for="item in recommendList" :key="item.id" @click="handleArticleClick(item.id)">
            <div class="rec-title">{{ item.title }}</div>
            <div class="rec-date">{{ parseTime(item.createTime, '{y}-{m}-{d}') }}</div>
          </li>
          <div v-if="recommendList.length === 0" class="empty-text">暂无推荐</div>
        </ul>
      </el-card>

    </div>
  </div>
</template>

<script>
// 引入需要的 API
import { listBlog } from "@/api/cms/blog";
import { listType } from "@/api/cms/type";
import { listTag } from "@/api/cms/tag";
import { mapGetters } from "vuex";

export default {
  name: "BloggerInfo",
  data() {
    return {
      // 博主信息
      blogger: {
        avatar: require("@/assets/images/profile.jpg"),
        name: "NiHaiCreeper",
        bio: "Talk is cheap. Show me the code.",
        stats: {
          articles: 0,
          categories: 0,
          tags: 0
        }
      },
      // 列表数据
      typeList: [],
      tagList: [],
      recommendList: [],
      // 标签颜色类型
      tagTypes: ['', 'success', 'info', 'warning', 'danger']
    };
  },
  computed: {
    ...mapGetters(["name", "avatar"])
  },
  created() {
    this.initData();
    // 如果想使用当前登录用户信息覆盖默认头像和名称
    // if (this.name) this.blogger.name = this.name;
    // if (this.avatar) this.blogger.avatar = this.avatar;
  },
  methods: {
    // 初始化所有数据
    initData() {
      this.getBloggerStats();
      this.getSideLists();
    },

    // 获取统计数据（顶部卡片用）
    getBloggerStats() {
      listBlog().then(res => this.blogger.stats.articles = res.total);
      listType().then(res => this.blogger.stats.categories = res.total);
      listTag().then(res => this.blogger.stats.tags = res.total);
    },

    // 获取下方列表数据
    getSideLists() {
      // 1. 获取分类列表 (取前 6 个)
      listType({ pageNum: 1, pageSize: 6 }).then(res => {
        this.typeList = res.rows;
      });

      // 2. 获取标签列表 (取前 15 个)
      listTag({ pageNum: 1, pageSize: 15 }).then(res => {
        this.tagList = res.rows;
      });

      // 3. 获取最新推荐文章
      // 注意：这里假设您想显示“推荐”的文章。如果没有推荐字段，去掉 isRecommend 参数即为“最新文章”
      // 常见的推荐字段可能是 'isRecommend': '1' 或 'support': '1'，请根据您的后端调整
      let queryParams = {
        pageNum: 1,
        pageSize: 5,
        orderByColumn: 'createTime',
        isAsc: 'desc',
        // status: '1' // 确保是已发布的
      };
      // 如果您的系统有推荐功能，取消下面注释
      // queryParams.isRecommend = '1';

      listBlog(queryParams).then(res => {
        this.recommendList = res.rows;
      });
    },

    // 随机标签颜色
    randomTagType() {
      const index = Math.floor(Math.random() * this.tagTypes.length);
      return this.tagTypes[index];
    },

    // 页面跳转 (顶部统计栏)
    handleGoTo(type) {
      let path = "/cms/index";
      if (type === 'blog') path = "/cms/index";
      // 根据您的路由配置，如果有单独的分类墙/标签墙页面，请修改这里
      // if (type === 'type') path = "/cms/type/wall";
      // if (type === 'tag') path = "/cms/tag/wall";
      this.$router.push(path);
    },

    // 点击分类/标签 -> 跳转首页并筛选
    handleFilter(key, value) {
      this.$router.push({
        path: '/cms/index',
        query: { [key]: value }
      });
      // 触发事件让父组件重新加载列表(如果父组件在监听路由变化则不需要这步)
      // this.$emit('filter-change');
    },

    // 点击文章 -> 跳转详情页
    handleArticleClick(blogId) {
      // 根据您的路由配置调整详情页路径
      this.$router.push(`/cms/blog/${blogId}`);
      // 或者: this.$router.push({ path: '/cms/detail', query: { id: blogId } });
    }
  }
};
</script>

<style lang="scss" scoped>
.sidebar-wrapper {
  // 整体容器样式
}

// ============ 1. 博主信息卡片样式 (复用之前) ============
.blogger-info-card {
  position: relative;
  text-align: center;
  border: none;
  margin-bottom: 20px; // 与下方内容拉开间距
  ::v-deep .el-card__body { padding: 20px 10px; }

  .blogger-avatar img {
    width: 100px; height: 100px; border-radius: 50%;
    border: 4px solid #f5f7fa; transition: transform 0.4s; cursor: pointer;
    &:hover { transform: rotate(360deg); }
  }
  .blogger-name { font-size: 22px; font-weight: 600; color: #303133; margin-top: 15px; }
  .blogger-bio { font-size: 14px; color: #909399; margin-top: 8px; }
  .info-divider { margin: 20px 0; }
  .blogger-stats {
    display: flex; justify-content: space-around;
    .stat-item {
      cursor: pointer; padding: 5px 10px; border-radius: 8px; transition: 0.3s;
      &:hover { background-color: #f5f7fa; .stat-count { color: #409EFF; } }
      .stat-count { font-size: 20px; font-weight: bold; color: #303133; margin-bottom: 5px; }
      .stat-label { font-size: 13px; color: #909399; }
    }
  }
  .sidebar-toggle-internal {
    position: absolute; top: 10px; right: 10px; padding: 5px;
    cursor: pointer; color: #909399; &:hover { color: #409EFF; }
  }
}

// ============ 2. 原侧边栏样式 (新增) ============
.original-sidebar-content {
  .sidebar-card {
    margin-bottom: 20px;
    border: none; // 扁平化风格
    // border: 1px solid #e6e6e6; // 或保留边框

    .card-header {
      display: flex; justify-content: space-between; align-items: center;
      font-size: 16px; font-weight: 600; color: #333;
      .more { font-size: 12px; color: #999; cursor: pointer; &:hover { color: #409EFF; } }
      i { margin-right: 5px; color: #409EFF; }
    }

    ::v-deep .el-card__body { padding: 15px; }
  }

  // 分类列表样式
  .category-list {
    list-style: none; padding: 0; margin: 0;
    li {
      display: flex; justify-content: space-between; padding: 10px 5px;
      border-bottom: 1px solid #f0f0f0; cursor: pointer; transition: all 0.3s;
      &:last-child { border-bottom: none; }
      &:hover {
        background-color: #f9f9f9; padding-left: 10px; // 悬停向右动一点
        .cate-name { color: #409EFF; }
      }
      .cate-name { font-size: 14px; color: #606266; }
      .cate-count { font-size: 12px; color: #909399; background: #f0f2f5; padding: 2px 6px; border-radius: 10px; }
    }
  }

  // 标签云样式
  .tag-cloud {
    display: flex; flex-wrap: wrap; gap: 10px;
    .tag-item { cursor: pointer; transition: all 0.3s; &:hover { transform: scale(1.1); } }
  }

  // 推荐列表样式
  .recommend-list {
    list-style: none; padding: 0; margin: 0;
    li {
      padding: 10px 0; border-bottom: 1px dashed #eee; cursor: pointer;
      &:last-child { border-bottom: none; }
      &:hover .rec-title { color: #409EFF; }
      .rec-title {
        font-size: 14px; color: #333; margin-bottom: 5px;
        overflow: hidden; text-overflow: ellipsis; white-space: nowrap; // 单行省略
      }
      .rec-date { font-size: 12px; color: #999; }
    }
  }

  .empty-text { text-align: center; color: #999; font-size: 13px; padding: 10px 0; }
}
</style>
