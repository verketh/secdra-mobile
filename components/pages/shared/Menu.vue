<template>
  <div>
    <transition
      name="fade"
      enter-active-class="fadeIn mask-duration"
      leave-active-class="fadeOut mask-duration"
    >
      <div v-show="show" @click.stop="close" class="mask"></div>
    </transition>
    <div :class="{ active: show }" class="menu">
      <div
        v-lazy:background-image="$img.backLazy(user.background, `backCard`)"
        class="cover"
      ></div>
      <div class="center" style="margin-top: -10vw;margin-bottom: 5vw">
        <a v-ripple @click="to(`/user/${user.id}`)" class="head-box">
          <img v-lazy="$img.headLazy(user.head, `small100`)" />
        </a>
      </div>
      <ul class="list">
        <li :class="{ active: activeName === `new` }">
          <a v-ripple @click="to(`/new`)">
            <i class="icon ali-icon-new"></i>
            最新发现
          </a>
        </li>
        <li :class="{ active: activeName === `collection` }">
          <a v-ripple @click="to(`/collection`)">
            <i class="icon ali-icon-like"></i>
            我的收藏
          </a>
        </li>
        <li :class="{ active: activeName === `works` }">
          <a v-ripple @click="to(`/works`)">
            <i class="icon ali-icon-pic"></i>
            我的作品
          </a>
        </li>
        <li :class="{ active: activeName === `footprint` }">
          <a v-ripple @click="to(`/footprint`)">
            <i class="icon ali-icon-footprint"></i>
            我的足迹
          </a>
        </li>
        <li :class="{ active: activeName === `follower` }">
          <a v-ripple @click="to(`/follower`)">
            <i class="icon ali-icon-friendfavor"></i>
            我的粉丝
          </a>
        </li>
        <li :class="{ active: activeName === `following` }">
          <a v-ripple @click="to(`/following`)">
            <i class="icon ali-icon-friendadd"></i>
            关注用户
          </a>
        </li>
        <li :class="{ active: activeName === `upload` }">
          <a v-ripple @click="to(`/upload`)">
            <i class="icon ali-icon-upload"></i>
            我要上传
          </a>
        </li>
        <li :class="{ active: activeName === `message` }">
          <a v-ripple @click="to(`/message`)">
            <i class="icon ali-icon-community"></i>
            通知信息
          </a>
        </li>
        <li>
          <a v-ripple @click="logout">
            <i class="icon ali-icon-logout"></i>
            退出登录
          </a>
        </li>
      </ul>
    </div>
  </div>
</template>

<script>
import Cookie from "js-cookie"
import { mapState, mapMutations } from "vuex"

export default {
  componentName: "Menu",

  computed: {
    ...mapState("menu", {
      show: "show",
      activeName: "name"
    }),
    ...mapState("user", ["user"])
  },
  methods: {
    ...mapMutations("menu", ["MChangeShow"]),
    close() {
      this.MChangeShow(false)
    },
    to(path) {
      if (this.$route.path === path) return
      this.close()
      this.$router.push(path)
    },
    logout() {
      this.close()
      this.$confirm({
        message: `你确定要退出登录吗？`,
        okCallback: () => {
          Cookie.remove("token")
          this.$router.replace(`/login?r=${this.$route.fullPath}`)
        }
      })
    }
  }
}
</script>

<style type="text/less" lang="less" scoped>
@import "../../../assets/style/color.less";
@import "../../../assets/style/config.less";
@import "../../../assets/style/mixin";

.menu {
  @width: 70vw;
  height: 100vh;
  width: @width;
  background-color: white;
  position: fixed;
  top: 0;
  left: 0;
  transform: translateX(-@width - 1vw);
  transition: @short-animate-time;
  z-index: @mask-index+1;
  &.active {
    transform: translateX(0);
  }
  .cover {
    height: @width / 2;
  }
  .head-box {
    display: inline-block;
    position: relative;
    border-radius: 50%;
    img {
      border-radius: 50%;
      width: @width / 3;
    }
  }
  .list {
    li {
      line-height: 10vw;
      display: flex;
      align-items: center;
      a {
        font-size: @default-font-size;
        display: block;
        padding: 0 8vw;
        flex: 1;
      }
      .icon {
        margin-right: 3vw;
        font-size: @big-font-size;
      }
      &.active {
        background-color: @theme-background-color;
        a {
          color: @theme-color;
        }
        .icon {
          color: @theme-color;
        }
      }
    }
  }
}
</style>
