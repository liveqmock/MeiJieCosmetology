<template>
  <nav class="site-navbar" :class="navbarClasses">
    <div class="site-navbar__header">
      <h1 class="site-navbar__brand" @click="$router.push({ name: 'home' })">
        <a class="site-navbar__brand-lg" href="javascript:;"><img src="~@/assets/img/logo.png" height="40" /></a>
        <a class="site-navbar__brand-mini" href="javascript:;">logo</a>
      </h1>
    </div>
    <div class="site-navbar__body clearfix">
      <el-menu class="site-navbar__menu" mode="horizontal">
        <el-menu-item class="site-navbar__switch" index="0">
          <!-- 医疗美容后台 -->
        </el-menu-item>
      </el-menu>
      <el-menu class="site-navbar__menu site-navbar__menu--right" mode="horizontal">
        <!-- <el-menu-item index="1" @click="$router.push({ name: 'setting' })">
          <template slot="title">
            <el-badge value="new">
              <icon-svg name="shezhi" class="el-icon-setting"></icon-svg>
            </el-badge>
          </template>
        </el-menu-item>
        <el-menu-item index="2">
          <el-badge value="hot">
            <a href="//www.renren.io/" target="_blank">官方社区</a>
          </el-badge>
        </el-menu-item> -->
        <!-- <el-submenu index="3">
          <template slot="title">Git源码</template>
          <el-menu-item index="2-1">
            <a href="//github.com/daxiongYang/renren-fast-vue" target="_blank">前端</a>
          </el-menu-item>
          <el-menu-item index="2-2">
            <a href="//git.oschina.net/renrenio/renren-fast" target="_blank">后台</a>
          </el-menu-item>
          <el-menu-item index="2-3">
            <a href="//git.oschina.net/renrenio/renren-generator" target="_blank">代码生成器</a>
          </el-menu-item>
        </el-submenu> -->
        <el-submenu index="1">
          <template slot="title">
            <img src="~@/assets/img/main/xx-w.png" height="30" />
            <span style="color:#fff;">信息提醒</span>
          </template>
          <el-menu-item index="1-1" v-if="isAuth('appoint:appointment:list')">
            <a href="#/appointmentTip" style="color:#fff;"><img src="~@/assets/img/main/xx-w.png" height="30" />预约提醒</a>
          </el-menu-item>
          <el-menu-item index="1-1" v-if="isAuth('member:member:list')">
            <a @click="tipInfo()" style="color:#fff;"><img src="~@/assets/img/main/xx-w.png" height="30" />特殊日期</a>
          </el-menu-item>
          <el-menu-item index="1-1" v-if="isAuth('sys:log:list')">
            <a href="#/sysLogInfo" style="color:#fff;"><img src="~@/assets/img/main/xx-w.png" height="30" />系统信息</a>
          </el-menu-item>
          <!-- <el-menu-item index="1-2">
            <a href="//git.oschina.net/renrenio/renren-fast" style="color:#fff;">信息提示</a>
          </el-menu-item>
          <el-menu-item index="1-3">
            <a href="//git.oschina.net/renrenio/renren-generator" style="color:#fff;">信息提示</a>
          </el-menu-item> -->
        </el-submenu>
        <el-menu-item index="2" style="margin-right:30px;" v-if="isAuth('knowledge:knowledge:list')">
          <el-badge value="hot">
            <a href="#/knowledgegrid" style="vertical-align:top;"><img src="~@/assets/img/main/zsk-w.png" height="30" />知识库</a>
          </el-badge>
        </el-menu-item>
        <el-menu-item class="site-navbar__avatar" index="3">
          <el-dropdown :show-timeout="0" placement="bottom">
            <span class="el-dropdown-link">
              <img v-if="$store.state.user.headPortrait" :src="parseUrl($store.state.user.headPortrait)" :alt="$store.state.user.name">
              <img v-else src="~@/assets/img/head.png" :alt="$store.state.user.name"> {{ $store.state.user.name }}
            </span>
            <el-dropdown-menu slot="dropdown">
              <el-dropdown-item @click.native="updatePasswordHandle()">修改密码</el-dropdown-item>
              <!-- <el-dropdown-item>商户信息</el-dropdown-item> -->
              <el-dropdown-item @click.native="logoutHandle()">退出</el-dropdown-item>
            </el-dropdown-menu>
          </el-dropdown>
        </el-menu-item>
      </el-menu>
    </div>
    <!-- 弹窗, 修改密码 -->
    <update-password v-if="updatePassowrdVisible" ref="updatePassowrd"></update-password>
    <info-tip v-if="tipVisible" ref="infoTip"></info-tip>
  </nav>
</template>

<script>
import UpdatePassword from "./update-password";
import InfoTip from "./info-tip";
import API from "@/api";
import { mapMutations } from "vuex";
import parseUrl from "@/utils/parseUrl";
export default {
  data() {
    return {
      updatePassowrdVisible: false,
      tipVisible: false
    };
  },
  components: {
    UpdatePassword,
    InfoTip
  },
  computed: {
    navbarClasses() {
      let type = this.$store.state.navbarLayoutType;
      return [!/\S/.test(type) || type === "default" ? "" : `site-navbar--${type}`];
    }
  },
  methods: {
    parseUrl,
    // 切换侧边栏, 水平折叠收起状态
    switchSidebarCollapseHandle() {
      this.SWITCH_SIDEBAR_COLLAPSE({ collapse: !this.$store.state.sidebarCollapse });
    },
    // 修改密码
    updatePasswordHandle() {
      this.updatePassowrdVisible = true;
      this.$nextTick(() => {
        this.$refs.updatePassowrd.init();
      });
    },
    tipInfo() {
      this.tipVisible = true;
      this.$nextTick(() => {
        this.$refs.infoTip.init();
      });
    },
    // 退出
    logoutHandle() {
      this.$confirm(`确定进行[退出]操作?`, "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      })
        .then(() => {
          API.common.logout().then(({ data }) => {
            if (data && data.code === 0) {
              this.DELETE_CONTENT_TABS();
              this.$cookie.delete("token");
              this.$router.replace({ name: "login" });
            }
          });
        })
        .catch(() => {});
    },
    ...mapMutations(["SWITCH_SIDEBAR_COLLAPSE", "DELETE_CONTENT_TABS"])
  }
};
</script>
<style scoped>
.site-navbar__header {
  width: 183px;
  background-color: #c99f63;
}
.site-navbar__body {
  margin-left: 190px;
  background-color: #c99f63;
  padding-left: 20px;
}
.site-content__wrapper {
  margin-left: 200px;
}
.el-menu-item,
.site-navbar__switch {
  border-bottom-color: none !important;
  background-color: #c99f63 !important;
}
.el-menu-item,
.site-navbar__switch:hover {
  border-bottom-color: none !important;
  background-color: #c99f63 !important;
}
</style>
