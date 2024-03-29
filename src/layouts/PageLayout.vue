<template>
  <div class="page-layout">
    <div ref="pageHeader" class="page-header">
      <h1 class="title">{{ pageTitle }}</h1>
    </div>
    <div ref="page" :class="['page-content', layout, pageWidth]">
      <a-card :body-style="{padding: '20px 10px'}" :bordered="false">
        <slot />
      </a-card>
    </div>
  </div>
</template>

<script>
import { mapState, mapMutations } from 'vuex'
import { getI18nKey } from '@/utils/routerUtil'
export default {
  name: 'PageLayout',
  props: {
    title: {
      type: String,
      default: ''
    }
  },
  data() {
    return {
      page: {},
      pageHeaderHeight: 0
    }
  },
  computed: {
    ...mapState('setting', ['layout', 'multiPage', 'pageMinHeight', 'pageWidth', 'customTitles']),
    pageTitle() {
      const pageTitle = this.page && this.page.title
      return this.customTitle || (pageTitle && this.$t(pageTitle)) || this.title || this.routeName
    },
    routeName() {
      const route = this.$route
      return this.$t(getI18nKey(route.matched[route.matched.length - 1].path))
    },
    marginCorrect() {
      return this.multiPage ? 24 : 0
    }
  },
  watch: {
    $route() {
      this.page = this.$route.meta.page
    }
  },
  updated() {
    if (!this._inactive) {
      this.updatePageHeight()
    }
  },
  activated() {
    this.updatePageHeight()
  },
  deactivated() {
    this.updatePageHeight(0)
  },
  mounted() {
    this.updatePageHeight()
  },
  created() {
    this.page = this.$route.meta.page
  },
  beforeDestroy() {
    this.updatePageHeight(0)
  },
  methods: {
    ...mapMutations('setting', ['correctPageMinHeight']),
    getRouteBreadcrumb() {
      const routes = this.$route.matched
      const path = this.$route.path
      const breadcrumb = []
      routes.filter(item => path.includes(item.path))
        .forEach(route => {
          const path = route.path.length === 0 ? '/home' : route.path
          breadcrumb.push(this.$t(getI18nKey(path)))
        })
      const pageTitle = this.page && this.page.title
      if (this.customTitle || pageTitle) {
        breadcrumb[breadcrumb.length - 1] = this.customTitle || pageTitle
      }
      return breadcrumb
    },
    /**
     * 用于计算页面内容最小高度
     * @param newHeight
     */
    updatePageHeight(newHeight = this.$refs.pageHeader.offsetHeight + this.marginCorrect) {
      this.correctPageMinHeight(this.pageHeaderHeight - newHeight)
      this.pageHeaderHeight = newHeight
    }
  }
}
</script>

<style lang="less" scoped>
.page-header{
  padding: 10px;
  background-color: @base-bg-color;
  .title{
    font-size: 14px;
    color: @title-color;
    padding-left: 10px;
    margin: 0;
    position: relative;
    &::before{
      content: '';
      width: 3px;
      height: 14px;
      position: absolute;
      left: 0;
      top: 4px;
      background-color: @title-color;
    }
  }
}
  .link{
    /*margin-top: 16px;*/
    line-height: 24px;
    a{
      font-size: 14px;
      margin-right: 32px;
      i{
        font-size: 22px;
        margin-right: 8px;
      }
    }
  }
  .page-content{
    position: relative;
    padding: 10px 0 0;
    &.head.fixed{
      margin: 0 auto;
      max-width: 1400px;
    }
  }
</style>
