<!--
 * @Description: 项目根组件
 * @Author: hai-27
 * @Date: 2020-02-07 16:23:00
 * @LastEditors: hai-27
 * @LastEditTime: 2020-03-12 19:30:36
 -->
<template>
  <div id="app" name="app">
    <el-container>
      <!-- 顶部导航栏 -->
      <div class="topbar">
        <div class="nav">
          <ul>

            <li>
              <router-link to="/order">订单</router-link>
            </li>
            <li>
              <router-link to="/collect">喜欢</router-link>
            </li>
            <li :class="getNum > 0 ? 'shopCart-full' : 'shopCart'">
              <router-link to="/shoppingCart">
                <i class="el-icon-shopping-cart-full"></i> 购物车
                <span class="num">({{getNum}})</span>
              </router-link>
            </li>
          </ul>
          <!-- 登录和注册链接 -->
          <ul class="auth-links">
            <li v-if="!this.$store.getters.getUser">
              <el-button type="text" @click="login">登录</el-button>
              <span class="sep">OR</span>
              <el-button type="text" @click="register = true">注册</el-button>
            </li>
            <li v-else>
              Hello
              <el-popover placement="top" width="180" v-model="visible">
                <p>确定退出登录吗？</p>
                <div style="text-align: right; margin: 10px 0 0">
                  <el-button size="mini" type="text" @click="visible = false">取消</el-button>
                  <el-button type="primary" size="mini" @click="logout">确定</el-button>
                </div>
                <el-button type="text" slot="reference">{{this.$store.getters.getUser.username}}</el-button>
              </el-popover>
            </li>
          </ul>
        </div>
      </div>
      <!-- 顶部导航栏END -->

      <!-- 顶栏容器 -->
      <el-header>
        <el-menu
          :default-active="activeIndex"
          class="el-menu-demo"
          mode="horizontal"
          active-text-color="#409eff"
          router
        >
          <div class="logo">
            <router-link to="/">
              <img src="./assets/imgs/logo.png" alt style="width: 300px;"/>
            </router-link>
          </div>
          <el-menu-item index="/">首页</el-menu-item>
          <el-menu-item index="/goods">农产品列表</el-menu-item>
          <el-menu-item index="/seckill">农产品秒杀(HOT)</el-menu-item>
          <!-- <el-menu-item index="/about">关于我们</el-menu-item> -->

          <div class="so">
            <el-input placeholder="你要买什么农产品?" v-model="search">
              <el-button slot="append" icon="el-icon-search" @click="searchClick"></el-button>
            </el-input>
          </div>
        </el-menu>
      </el-header>
      <!-- 顶栏容器END -->

      <!-- 登录模块 -->
      <MyLogin></MyLogin>
      <!-- 注册模块 -->
      <MyRegister :register="register" @fromChild="isRegister"></MyRegister>

      <!-- 主要区域容器 -->
      <el-main>
        <keep-alive>
          <router-view></router-view>
        </keep-alive>
      </el-main>
      <!-- 主要区域容器END -->

      <!-- 底栏容器 -->
      <el-footer>
        <div class="footer">
          <div class="ng-promise-box">
            <div class="ng-promise">
              <p class="text">
                <a class="icon1" href="javascript:;">农产品丰富，品质有保证</a>
                <a class="icon2" href="javascript:;">3天直达，留住新鲜</a>
                <a class="icon3" href="javascript:;">自然呵护，健康美味尽享</a>
                <a class="icon4" style="margin-right: 0" href="javascript:;">天天秒杀，天天低价</a>
              </p>
            </div>
          </div>
<!--          <div class="github">-->
<!--            <a href="https://github.com/ZeroWdd" target="_blank">-->
<!--              <div class="github-but"></div>-->
<!--            </a>-->
<!--          </div>-->


          <div class="mod_help">
<!--            <p>-->
<!--              <router-link to="/">首页</router-link>-->
<!--              <span>|</span>-->
<!--              <router-link to="/goods">全部商品</router-link>-->
<!--              <span>|</span>-->
<!--              <router-link to="/seckill">秒杀</router-link>-->
<!--              <span>|</span>-->
<!--               <router-link to="/about">关于我们</router-link>-->
<!--            </p>-->
            <p class="coty">created by zjm</p>
          </div>
        </div>
      </el-footer>
      <!-- 底栏容器END -->
    </el-container>
  </div>
</template>

<script>
import { mapActions } from "vuex";
import { mapGetters } from "vuex";

export default {
  beforeUpdate() {
    this.activeIndex = this.$route.path;
  },
  data() {
    return {
      activeIndex: "", // 头部导航栏选中的标签
      search: "", // 搜索条件
      register: false, // 是否显示注册组件
      visible: false // 是否退出登录
    };
  },
  created() {
    // 对用户信息进行校验，并刷新cookie
    if (this.getCookie('XM_TOKEN') != null) {
      this.$axios
        .get("/api/user/token")
        .then(res => {
          if (res.data.code === "001") {
            // 001 为成功
            this.setUser(res.data.data);
          }
        });
      let cookie = this.getCookie('XM_TOKEN');
      let user = {};
      user.userId = cookie.split("|")[1];
      user.username = cookie.split("|")[2];
      this.setUser(user);
    }

  },
  computed: {
    ...mapGetters(["getUser", "getNum"])
  },
  watch: {
    // 获取vuex的登录状态
    getUser: function(val) {
      if (val === "") {
        // 用户没有登录
        this.setShoppingCart([]);
      } else {
        // 用户已经登录,获取该用户的购物车信息
        this.$axios
          .get("/api/cart/user/" + val.userId)
          .then(res => {
            if (res.data.code === "001") {
              // 001 为成功, 更新vuex购物车状态
              this.setShoppingCart(res.data.data);
            } else {
              // 提示失败信息
              this.notifyError(res.data.msg);
            }
          })
          .catch(err => {
            return Promise.reject(err);
          });
      }
    }
  },
  methods: {
    ...mapActions(["setUser", "setShowLogin", "setShoppingCart"]),
    login() {
      // 点击登录按钮, 通过更改vuex的showLogin值显示登录组件
      this.setShowLogin(true);
    },
    // 退出登录
    logout() {
      this.visible = false;
      // 清空cookie中的token
      this.delCookie("XM_TOKEN");
      // 清空vuex登录信息
      this.setUser("");
      this.notifySucceed("成功退出登录");
    },
    // 接收注册子组件传过来的数据
    isRegister(val) {
      this.register = val;
    },
    // 点击搜索按钮
    searchClick() {
      if (this.search != "") {
        // 跳转到全部商品页面,并传递搜索条件
        this.$router.push({ path: "/goods", query: { search: this.search } });
        this.search = "";
      }
    },
    getCookie(name) { //获取指定名称的cookie值
      // (^| )name=([^;]*)(;|$),match[0]为与整个正则表达式匹配的字符串，match[i]为正则表达式捕获数组相匹配的数组；
      let arr = document.cookie.match(new RegExp("(^| )"+name+"=([^;]*)(;|$)"));
      if(arr != null) {
        return unescape(arr[2]);
      }
      return null;
    },
    delCookie(name) {
      let exp = new Date();
      exp.setTime(exp.getTime() - 1);
      let cval = this.getCookie(name);
      if (cval != null)
        document.cookie = name + "=" + cval + ";expires=" + exp.toGMTString();
    }
  }
};
</script>

<style>
/* 全局CSS */
* {
  padding: 0;
  margin: 0;
  border: 0;
  list-style: none;
}
#app .el-header {
  padding: 0;
}
#app .el-main {
  min-height: 300px;
  padding: 20px 0;
}
#app .el-footer {
  padding: 0;
}
a,
a:hover {
  text-decoration: none;
}
/* 全局CSS END */

/* 顶部导航栏CSS */
.topbar {
  height: 40px;
  background-color: #fff;
  margin-bottom: 20px;
}
.topbar .nav {
  width: 1225px;
  margin: 0 auto;
}
.topbar .nav ul {
  float: left;
}
.topbar .nav li {
  float: right;
  height: 40px;
  color: #3b2d2d;
  font-size: 14px;
  text-align: center;
  line-height: 40px;
  margin-left: 20px;
}
.topbar .nav .sep {
  color: #3b2d2d;
  font-size: 12px;
  margin: 0 5px;
}
.topbar .nav li .el-button {
  color: #3b2d2d;
}
.topbar .nav .el-button:hover {
  color: #ccb340;
}
.topbar .nav li a {
  color: #3b2d2d;
}
.topbar .nav a:hover {
  color: #ccb340;
}
.topbar .nav .shopCart {
  width: 120px;
  background: #dee46f;
}
.topbar .nav .shopCart:hover {
  background: #fff;
}
.topbar .nav .shopCart:hover a {
  color: #ff6700;
}
.topbar .nav .shopCart-full {
  width: 120px;
  background: #ff6700;
}
.topbar .nav .shopCart-full a {
  color: white;
}
.topbar .nav .auth-links {
  float: right;
}
/* 顶部导航栏CSS END */

/* 顶栏容器CSS */
.el-header .el-menu {
  max-width: 1225px;
  margin: 0 auto;
}
.el-header .logo {
  height: 60px;
  width: 189px;
  float: left;
  margin-right: 100px;
}
.el-header .so {
  margin-top: 10px;
  width: 600px;
  float: right;
  color: #9dffb5;
}
/* 顶栏容器CSS END */

/* 底栏容器CSS */
.footer {
  width: 100%;
  text-align: center;
  background: #eaeaea;
  padding-bottom: 20px;
}
.footer .ng-promise-box {
  border-bottom: 1px solid #3d3d3d;
  line-height: 145px;
}
.footer .ng-promise-box {
  margin: 0 auto;
  border-bottom: 1px solid #3d3d3d;
  line-height: 145px;
}
.footer .ng-promise-box .ng-promise p a {
  color: #181717;
  font-size: 20px;
  margin-right: 110px;
  padding-left: 44px;
  height: 42px;
  display: inline-block;
  line-height: 40px;
  text-decoration: none;
  /*background: url("./assets/imgs/us-icon.png") no-repeat left 0;*/
  background: url("./assets/imgs/img.png") no-repeat left 0;
}

.footer .ng-promise-box .ng-promise .text .icon2  {
  background-position: 0 -43px
}

.footer .ng-promise-box .ng-promise .text .icon3 {
  background-position: 0 -86px
}

.footer .ng-promise-box .ng-promise .text .icon4 {
  background-position: 0 -129px
}
.footer .github {
  height: 50px;
  line-height: 50px;
  margin-top: 20px;
}
.footer .github .github-but {
  width: 50px;
  height: 50px;
  margin: 0 auto;

}
.footer .mod_help {
  text-align: center;
  color: #888888;
}
.footer .mod_help p {
  margin: 20px 0 16px 0;
}

.footer .mod_help p a {
  color: #888888;
  text-decoration: none;
}
.footer .mod_help p a:hover {
  color: #fff;
}
.footer .mod_help p span {
  padding: 0 22px;
}
/* 底栏容器CSS END */
</style>