<template>
  <div class="page">
    <PictureList
      :page="page"
      :list="list"
      :page-loading="pageLoading"
      @paging="paging"
      @collection="collection"
      @follow="follow"
    ></PictureList>
    <CornerButtons>
      <Popper placement="top-end" trigger="click" offset="0,4vw" position-fixed>
        <div class="filter-box">
          <div class="input-group">
            <h5 class="sub-name">精准搜索：</h5>
            <Checkbox
              v-model="filterForm.precise"
              is-switch
              color="primary"
            ></Checkbox>
          </div>
          <div class="input-group">
            <h5 class="sub-name">名称：</h5>
            <Field v-model="filterForm.name" block color="primary"></Field>
          </div>
          <div class="input-group center">
            <Btn @click="filter" color="primary">筛选</Btn>
            <Btn @click="reset" color="primary">重置</Btn>
          </div>
        </div>
        <Btn slot="reference" icon big shadow color="white">
          <i class="icon ali-icon-filter"></i>
        </Btn>
      </Popper>
    </CornerButtons>
  </div>
</template>

<script>
import { FliterForm, Pageable } from "../../../assets/script/model"
import PictureList from "../../../components/pages/shared/PictureList"
import CornerButtons from "../../../components/pages/shared/CornerButtons"
import { mapActions } from "vuex"

export default {
  components: {
    PictureList,
    CornerButtons
  },
  watchQuery: true,
  data() {
    return {
      pageLoading: false
    }
  },
  // 在这里不能使用httpUtil
  // 并且嵌套层数超过不知道多少会报错-->坑死我了
  async asyncData({ store, route, $axios }) {
    store.commit("menu/MChangeName", "tag")
    const pageable = new Pageable(0, 16, "likeAmount,desc")
    const filterForm = new FliterForm(
      !!route.query.precise,
      route.query.name,
      route.query.startDate,
      route.query.endDate
    )
    const { data: result } = await $axios.get(`/picture/paging`, {
      params: Object.assign(
        {
          tagList: route.params.tagList
        },
        pageable,
        filterForm
      )
    })
    return {
      page: result.data,
      list: result.data.content,
      pageable,
      filterForm
    }
  },
  head() {
    return { title: this.$route.params.tagList + "的搜索结果 - Secdra" }
  },
  methods: {
    ...mapActions("picture", ["APaging", "ACollection"]),
    ...mapActions("user", ["AFollow"]),
    /**
     * 获取分页数据
     * @returns {Promise<void>}
     */
    async paging() {
      if (this.pageLoading || this.page.last) {
        return
      }
      const sourcePage = ++this.pageable.page
      this.pageLoading = true
      const result = await this.APaging(
        Object.assign(
          {
            tagList: this.$route.params.tagList
          },
          this.pageable,
          this.filterForm
        )
      )
      this.pageLoading = false
      const data = result.data
      if (result.status !== 200) {
        this.$tooltip({ message: result.message })
        this.pageable.page = sourcePage
        return
      }
      this.page = data
      this.list.push(...data.content)
    },
    async collection(picture) {
      const result = await this.ACollection({
        pictureId: picture.id
      })
      if (result.status !== 200) {
        this.$tooltip({ message: result.message })
        return
      }
      picture.focus = result.data
    },
    async follow(id) {
      const result = await this.AFollow({
        followingId: id
      })
      if (result.status !== 200) {
        this.$tooltip({ message: result.message })
        return
      }
      this.list.forEach((item) => {
        if (item.user.id === id) {
          item.user.focus = result.data
        }
      })
    },
    filter() {
      this.$router.replace({
        path: `/picture/search/${encodeURIComponent(
          this.$route.params.tagList
        )}`,
        query: {
          precise: this.filterForm.precise ? 1 : undefined,
          name: this.filterForm.name,
          startDate: this.filterForm.startDate,
          endDate: this.filterForm.endDate
        }
      })
    },
    reset() {
      this.$router.replace(
        `/picture/search/${encodeURIComponent(this.$route.params.tagList)}`
      )
    }
  }
}
</script>

<style type="text/less" lang="less" scoped>
@import "../../../assets/style/color";
@import "../../../assets/style/config";
@import "../../../assets/style/mixin";

.filter-box {
  width: 60vw;
  padding: 3vw;
}
</style>
