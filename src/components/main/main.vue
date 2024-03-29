<template>
  <Layout style="height: 100%" class="main">
    <Sider
      hide-trigger
      collapsible
      :width="256"
      :collapsed-width="64"
      v-model="collapsed"
      class="left-sider"
      :style="{ overflow: 'hidden' }"
    >
      <side-menu
        accordion
        ref="sideMenu"
        :active-name="$route.name"
        :collapsed="collapsed"
        @on-select="turnToPage"
        :menu-list="menuList"
        theme="dark"
      >
        <!-- 需要放在菜单上面的内容，如Logo，写在side-menu标签内部，如下 -->
        <div class="logo-con">
          <div class="logBox" v-show="!collapsed" key="max-logo">
            <!-- <img :src="logo" class="logo" /> -->
            <p>中铁隧道局Human-AI协同平台</p>
          </div>
          <!-- <img v-show="collapsed" :src="logo" key="min-logo" class="logo" /> -->
        </div>
      </side-menu>
      <footer style="text-align:center;position: absolute; bottom: 2%; width:100%;font-family: 'Helvetica Neue',Helvetica,'PingFang SC','Hiragino Sans GB','Microsoft YaHei','\5FAE\8F6F\96C5\9ED1',Arial,sans-serif;">
        <div class="lc-content">
          <div class="lc-company">技术支持：北京令才科技有限公司</div>
          <div class="lc-phone">服务电话400-167-8089</div>
        </div>
      </footer>
    </Sider>
    <Layout>
      <Header class="header-con">
        <header-bar
          :collapsed="collapsed"
          @on-coll-change="handleCollapsedChange"
        >
          <user
            v-if="userToken"
            :message-unread-count="unreadCount"
            :user-avatar="userAvatar"
            :userName="userName"
          />
        </header-bar>
      </Header>
      <Content class="main-content-con">
        <Layout class="main-layout-con">
          <Content class="content-wrapper">
            <keep-alive :include="cacheList">
              <router-view />
            </keep-alive>
          </Content>
        </Layout>
      </Content>
    </Layout>
  </Layout>
</template>
<script>
import SideMenu from "./components/side-menu";
import HeaderBar from "./components/header-bar";
import User from "./components/user";
import { mapMutations, mapActions, mapGetters } from "vuex";
import { getNewTagList, routeEqual, localSave } from "@/libs/util";
import routers from "@/router/routers";
import "./main.less";
export default {
  name: "Main",
  components: {
    SideMenu,
    HeaderBar,
    User,
  },
  data() {
    return {
      collapsed: false,
      minLogo: "",
      maxLogo: "",
      isFullscreen: false,
    };
  },
  computed: {
    ...mapGetters(["errorCount"]),
    tagNavList() {
      return this.$store.state.app.tagNavList;
    },
    tagRouter() {
      return this.$store.state.app.tagRouter;
    },
    userAvatar() {
      return this.$store.state.user.avatarImgPath;
    },
    adminNo() {
      return this.$store.state.user.adminNo;
    },
    cacheList() {
      const list = [
        "ParentView",
        ...(this.tagNavList.length
          ? this.tagNavList
              .filter((item) => !(item.meta && item.meta.notCache))
              .map((item) => item.name)
          : []),
      ];
      return list;
    },
    menuList() {
      return this.$store.getters.menuList;
    },
    local() {
      return this.$store.state.app.local;
    },
    unreadCount() {
      return this.$store.state.user.unreadCount;
    },
    userName() {
      return this.$store.state.user.userName;
    },
    userToken() {
      return this.$store.state.user.token;
    },
  },
  methods: {
    ...mapMutations([
      "setBreadCrumb",
      "setTagNavList",
      "addTag",
      "setLocal",
      "setHomeRoute",
      "closeTag",
      "setCollapse",
    ]),
    ...mapActions(["handleLogin", "getRobotList", "getFormList"]),
    turnToPage(route) {
      let { name, params, query } = {};
      if (typeof route === "string") name = route;
      else {
        name = route.name;
        params = route.params;
        query = route.query;
      }
      if (name.indexOf("isTurnByHref_") > -1) {
        window.open(name.split("_")[1]);
        return;
      }
      this.$router.push({
        name,
        params,
        query,
      });
    },
    handleCollapsedChange(state) {
      this.collapsed = state;
      this.setCollapse(state);
      localSave("collapsed", state);
    },
    handleCloseTag(res, type, route) {
      if (type !== "others") {
        if (type === "all") {
          this.turnToPage(this.$config.homeName);
        } else {
          if (routeEqual(this.$route, route)) {
            this.closeTag(route);
          }
        }
      }
      this.setTagNavList(res);
    },
    handleClick(item) {
      this.turnToPage(item);
    },
  },
  watch: {
    $route(newRoute) {
      const { name, query, params, meta } = newRoute;
      this.addTag({
        route: { name, query, params, meta },
        type: "push",
      });
      this.setBreadCrumb(newRoute);
      this.setTagNavList(getNewTagList(this.tagNavList, newRoute));
      // this.$refs.sideMenu.updateOpenName(newRoute.name);
    },
  },
  mounted() {
    /**
     * @description 初始化设置面包屑导航和标签导航
     */
    this.setTagNavList();
    this.setHomeRoute(routers);
    const { name, params, query, meta } = this.$route;
    this.addTag({
      route: { name, params, query, meta },
    });
    this.setBreadCrumb(this.$route);
    // 设置初始语言
    this.setLocal(this.$i18n.locale);
    // 如果当前打开页面不在标签栏中，跳到homeName页
    if (!this.tagNavList.find((item) => item.name === this.$route.name)) {
      this.$router.push({
        name: this.$config.homeName,
      });
    }
    // 获取未读消息条数
    // this.getUnreadMessageCount();
    // 机器人列表
    this.getRobotList();
    // 表单名称列表
    this.getFormList();
  },
};
</script>
