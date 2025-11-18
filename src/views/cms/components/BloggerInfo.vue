<template>
  <div class="sidebar-wrapper">
    <el-card class="blogger-info-card" shadow="hover" style="background-color: rgba(255,255,255,0.9)">
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

      <el-card class="right-item" shadow="hover" style="background-color: rgba(255,255,255,0.9)">
        <div slot="header" class="attributes">
          <b>分类</b>
        </div>
        <ul class="blog-type-ul" style="margin-top: 5px;">
          <li class="blog-type-li" v-for="cmsType in displayTypeList" :key="cmsType.typeId" @click="selectType(cmsType)">
            <div style="display: flex;align-items: center">
              <el-image style="width: 28px;height: 28px; border-radius: 50%; margin-right: 10px" lazy
                        :src="cmsType.typePicLink" v-if="cmsType.typePicType == '0'">
                <div slot="error" style="width: 28px;height: 28px; border-radius: 50%;">
                  <i class="el-icon-collection" style="margin-left:6px;"></i>
                </div>
              </el-image>
              <el-image style="width: 28px;height: 28px; border-radius: 50%; margin-right: 10px" lazy
                        :src="cmsType.typePic" v-else>
                <div slot="error" style="width: 28px;height: 28px; border-radius: 50%;">
                  <i class="el-icon-collection" style="margin-left:6px;"></i>
                </div>
              </el-image>
              {{cmsType.typeName}}
            </div>
            <div>{{cmsType.blogNum || 0}}</div>
          </li>
        </ul>
        <div class="more" @click="dealType">
          <i v-if="moreType" class="el-icon-arrow-down"></i>
          <i v-else class="el-icon-arrow-up"></i>
        </div>
      </el-card>

      <el-card class="right-item" shadow="hover" style="background-color: rgba(255,255,255,0.9)">
        <div slot="header" class="attributes">
          <b>标签</b>
        </div>
        <div class="tags">
          <div class="tag-item" v-for="tag in displayTagList" :key="tag.tagId" @click="selectTag(tag)">
            <div class="sjx-outer">
              <div class="sjx-inner"></div>
            </div>
            <div class="tag">
              {{tag.tagName}} {{tag.blogNum || 0}}
            </div>
          </div>
        </div>
        <div class="more" @click="dealTag">
          <i v-if="moreTag" class="el-icon-arrow-down"></i>
          <i v-else class="el-icon-arrow-up"></i>
        </div>
      </el-card>

      <el-card class="right-item" shadow="hover" style="background-color: rgba(255,255,255,0.9)">
        <div slot="header" class="attributes">
          <b>最新推荐</b>
        </div>
        <div class="recommend-blog l-text" v-for="blog in recommendList" :key="blog.id" @click="getBlogInfo(blog.id)">
          <a class="recommend-a">{{blog.title}}</a>
        </div>
      </el-card>

    </div>
  </div>
</template>

<script>
import { cmsListRecommend, listBlog } from "@/api/cms/blog";
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
        stats: { articles: 0, categories: 0, tags: 0 }
      },
      // 侧边栏数据
      fullTypeList: [],
      displayTypeList: [],
      moreType: true, // 默认折叠

      fullTagList: [],
      displayTagList: [],
      moreTag: true, // 默认折叠

      recommendList: [],
    };
  },
  computed: {
    ...mapGetters(["name", "avatar"])
  },
  created() {
    this.initData();
    // 恢复个人信息（可选）
    // if (this.name) this.blogger.name = this.name;
    // if (this.avatar) this.blogger.avatar = this.avatar;
  },
  methods: {
    initData() {
      // 1. 获取统计数据
      listBlog().then(res => this.blogger.stats.articles = res.total);

      // 2. 获取分类列表 (使用标准接口 listType 而不是 getBlogDetail)
      listType().then(res => {
        this.blogger.stats.categories = res.total;
        // 处理图片路径
        this.fullTypeList = res.rows.map(item => {
          if (item.typePic && item.typePic.length > 0) {
            // 判断是否已经是绝对路径
            if(!item.typePic.startsWith('http')) {
              item.typePic = process.env.VUE_APP_BASE_API + item.typePic;
            }
          }
          return item;
        });
        // 默认显示前4个
        this.displayTypeList = this.fullTypeList.slice(0, 4);
      });

      // 3. 获取标签列表
      listTag().then(res => {
        this.blogger.stats.tags = res.total;
        this.fullTagList = res.rows;
        // 默认显示前6个
        this.displayTagList = this.fullTagList.slice(0, 6);
      });

      // 4. 获取推荐文章
      cmsListRecommend({ pageNum: 1, pageSize: 5 }).then(res => {
        this.recommendList = res.rows.slice(0, 4);
      });
    },

    // ------------------ 交互逻辑 ------------------

    // 点击分类
    selectType(cmsType) {
      // 跳转到首页并带上分类ID
      this.$router.push({ path: '/cms/main/cmsIndex', query: { typeId: cmsType.typeId } });
    },

    // 点击标签
    selectTag(tag) {
      // 跳转到首页并带上标签ID
      this.$router.push({ path: '/cms/main/cmsIndex', query: { tagId: tag.tagId } });
    },

    // 展开/收起分类
    dealType() {
      if (this.moreType) {
        this.displayTypeList = this.fullTypeList; // 展开
      } else {
        this.displayTypeList = this.fullTypeList.slice(0, 4); // 收起
      }
      this.moreType = !this.moreType;
    },

    // 展开/收起标签
    dealTag() {
      if (this.moreTag) {
        this.displayTagList = this.fullTagList; // 展开
      } else {
        this.displayTagList = this.fullTagList.slice(0, 6); // 收起
      }
      this.moreTag = !this.moreTag;
    },

    // 顶部三个统计图标的跳转
    handleGoTo(type) {
      let path = "/cms/main/cmsIndex";
      this.$router.push(path);
    },

    // 点击文章跳转详情
    getBlogInfo(blogId) {
      let routeUrl = this.$router.resolve({
        path: '/cms/main/blog',
        query: { id: blogId }
      });
      window.open(routeUrl.href, '_blank');
    }
  }
};
</script>

<style lang="scss" scoped>
/* ============ 1. 博主信息卡片样式 ============ */
.blogger-info-card {
  position: relative; text-align: center; border: none; margin-bottom: 20px;
  ::v-deep .el-card__body { padding: 20px 10px; }
  .blogger-avatar img {
    width: 100px; height: 100px; border-radius: 50%;
    border: 4px solid #f5f7fa; transition: transform 0.4s; cursor: pointer;
    &:hover { transform: rotate(360deg); }
  }
  .blogger-name { font-size: 22px; font-weight: 600; color: #303133; margin-top: 15px; }
  .blogger-bio { font-size: 14px; color: #909399; margin-top: 8px; padding: 0 10px; }
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
    position: absolute; top: 10px; right: 10px; padding: 5px; cursor: pointer; color: #909399;
    &:hover { color: #409EFF; }
  }
}

/* ============ 2. 复刻 cmsIndex 的 CSS 样式 ============ */
.right-item { margin-bottom: 20px; border: none; }
.attributes { font-weight: bold; }

.blog-type-ul {
  padding-left: 10px; padding-right: 10px; margin-bottom: 0; border-radius: 5px; list-style: none;
}

.blog-type-li {
  display: flex; justify-content: space-between; align-items: center;
  line-height: 40px; border-bottom: 1px solid rgba(179, 216, 255, 0.5);
  cursor: pointer;
  &:first-child { border-top: 1px solid rgba(179, 216, 255, 0.5); }
  &:hover { background-color: rgba(213, 255, 255, 0.3); }
}

.more { text-align: center; color: #3a8ee6; padding: 8px; cursor: pointer; }

.tags {
  display: flex; flex-wrap: wrap; align-items: center;
  margin: 15px 13px 0; border-bottom: 1px solid rgba(179, 216, 255, 0.5);
}

.tag-item {
  display: flex; justify-content: space-around; align-items: center;
  margin-left: 5px; margin-right: 5px; margin-bottom: 10px; box-sizing: border-box;
  cursor: pointer;
  &:hover .tag { background-color: #409eff; color: white; }
  &:hover .sjx-inner { border-right-color: #409eff; }
}

.tag {
  background-color: #ecf5ff; display: inline-block; height: 22px; padding: 0 10px;
  line-height: 22px; font-size: 10px; color: #409eff; border-radius: 4px;
  white-space: nowrap; border: 1px solid #409eff; border-left: none; transition: .2s;
}

.sjx-outer {
  width: 0; height: 0; border-top: 6px solid transparent; border-bottom: 6px solid transparent;
  border-right: 6px solid #409eff; position: relative; transition: .2s;
}

.sjx-inner {
  border-top: 6px solid transparent; border-bottom: 6px solid transparent;
  border-right: 6px solid #ecf5ff; top: -6px; left: 1px; position: absolute; transition: .2s;
}

.recommend-blog {
  overflow: hidden; text-overflow: ellipsis; white-space: nowrap;
  padding-left: 10px; padding-right: 10px; margin-bottom: 0; border-radius: 5px;
}

.recommend-a {
  border-bottom: 1px solid rgba(34, 36, 38, .15); line-height: 40px;
  display: block; text-decoration: none; color: black; cursor: pointer;
  &:hover { color: #3a8ee6; }
}
</style>
