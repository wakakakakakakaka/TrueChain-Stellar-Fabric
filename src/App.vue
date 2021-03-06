<template>
  <div id="tc-app">
    <nav :class="{'tc-nav-login': $route.name === null}">
      <img class="tc-nav-logo" src="./assets/logo.png" alt="logo" height="60">
      <transition name="fly-right">
        <div style="position: absolute; width: 100%;" v-if="$route.name !== null">
          <div class="tc-nav-title" v-for="item in routes" :key="routes.indexOf(item)" :class="{'tc-nav-title-focus': $route.name === item.name}" @click="$router.push(item.path)">{{item.name}}</div>
        </div>
      </transition>
      <transition name="fly-left">
        <div style="position: absolute;" class="tc-login" v-if="$route.name === null">
          <div>Name<input type="text" v-model="username"></div>
          <div>Password<input type="password" v-model="password"></div>
          <span @click.stop="signIn">Sign in</span>
          <loading-animation :color="'#fff'" :isActive="signInWaiting"></loading-animation>
        </div>
      </transition>
    </nav>
    <div id="tc-content">
      <hello v-if="$route.name === null"></hello>
      <div id="tc-user" @mouseleave="closeUserCtrl" :class="{'tc-user-hide': $route.name === null, 'tc-user-ctrl-open': userCtrlisOpen}">
        <div class="tc-user-info" @mouseover="openUserCtrl">
          {{userInfo.name}}
        </div>
        <div class="tc-user-ctrl">
          <span @click.stop="signOut">Sign out</span>
        </div>
      </div>
      <router-view @routerinit="initRouterEl"/>
    </div>
    <div id="tc-notice"></div>
  </div>
</template>

<script>
import routes from '@/router/routes.js'
import api from '@/api-config'

import Hello from '@/components/Hello'
import loadingAnimation from '@/components/common/gui/loading'

export default {
  name: 'App',
  data: () => {
    return {
      routes,
      userInfo: {
        name: ''
      },
      username: '',
      password: '',
      signInWaiting: false,
      userCtrlisOpen: false,
      noticeBox: null,
      noticeBoxTimer: 0
    }
  },
  mounted () {
    window.el = this
    this.userInfo.name = window.cookieStorage.getItem('userName')
    this.noticeBox = this.$el.querySelector('#tc-notice')
    window.notice = this.notice
  },
  methods: {
    signIn () {
      if (this.signInWaiting) {
        return
      }
      this.signInWaiting = true
      api.login(this.username, this.password).then((res) => {
        let data = res.data
        let d = new Date()
        d.setMinutes(d.getMinutes + 30)
        window.cookieStorage.setItem('userToken', data.token, {expires: d})
        window.cookieStorage.setItem('userName', this.username, {expires: d})
        this.userInfo.name = this.username
        this.notice('#4596f5', `Log in as ${this.username}.`, 3000)
        this.$router.push('/chaincode')
      }).catch((err) => {
        if (/403/.test(err)) {
          this.notice('#f78432', 'Check out your Name & Password.', 3000)
        } else {
          this.notice('#d21107', 'Network Error!', 3000)
        }
      }).then(() => {
        this.signInWaiting = false
      })
    },
    signOut () {
      window.cookieStorage.setItem('userToken', 'anyValue', {expires: new Date()})
      window.cookieStorage.setItem('userName', 'anyValue', {expires: new Date()})
      if (window.installedCcInfo) {
        window.installedCcInfo = false
      }
      this.$router.push('/')
    },
    openUserCtrl () {
      this.userCtrlisOpen = true
    },
    closeUserCtrl () {
      this.userCtrlisOpen = false
    },
    notice (color, text, time) {
      let el = this.noticeBox
      if (!el) {
        return
      }
      let delay = 0
      if (this.noticeBoxTimer) {
        clearTimeout(this.noticeBoxTimer)
        delay = 300
      }
      el.style.transform = 'translateY(110%)'
      el.style.backgroundColor = color
      el.innerHTML = text
      setTimeout(() => {
        el.style.transform = 'translateY(0%)'
      }, delay)
      this.noticeBoxTimer = setTimeout(() => {
        el.style.transform = 'translateY(110%)'
        this.noticeBoxTimer = 0
      }, delay + time)
    },
    initRouterEl (obj) {
      if (!window.cookieStorage.getItem('userToken')) {
        this.$router.push('/')
      }
      obj.updateSize = function () {
        obj.height = obj.$el.getBoundingClientRect().height
        obj.onMousewheel()
      }
      obj.onMousewheel = function (e) {
        if (e) {
          obj.pageTranslateY += e.deltaY > 0 ? -90 : 90
        }
        if (obj.pageTranslateY > 0) {
          obj.pageTranslateY = 0
        }
        let contentH = window.innerHeight - 60
        if (obj.height + obj.pageTranslateY < contentH) {
          obj.pageTranslateY = Math.min(contentH - obj.height, 0)
        }
      }
    }
  },
  components: {
    Hello,
    loadingAnimation
  }
}
</script>

<style>
#tc-app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #333;
  display: flex;
}
nav {
  color: #fff;
  width: 200px;
  height: 100vh;
  flex: 0 0 200px;
  background-color: #014676;
  box-shadow: 4px 0 8px #0001;
  position: relative;
  z-index: 10;
  transition: flex 1s;
  overflow: hidden;
}
.tc-nav-login {
  flex: 0 0 400px;
}
.tc-nav-logo {
  display: block;
  margin: auto;
  height: 60px;
  padding: 30px 0;
}
.tc-nav-title {
  cursor: pointer;
  padding-left: 20px;
  height: 50px;
  line-height: 50px;
  opacity: .6;
  transition: opacity 1s, background 0.6s;
  margin: 5px 0;
}
.tc-nav-title-focus {
  opacity: 1;
  color: #014676;
  background-color: #fff;
}
.tc-nav-title:hover {
  opacity: 1;
}
.tc-login {
  padding: 60px;
  width: 280px;
}
.tc-login input {
  float: right;
  width: 200px;
}
.tc-login div {
  height: 40px;
  line-height: 40px;
  margin-bottom: 20px;
  clear: both;
}
.tc-login span {
  width: 80px;
  float: left;
  line-height: 40px;
  /* clear: both; */
  cursor: pointer;
}
#tc-content {
  position: relative;
  flex: auto;
  background-color: #f0f4f8;
}
#tc-user {
  position: relative;
  z-index: 9;
  height: 60px;
  background-color: #fff;
  box-shadow: 0 2px 4px #0001;
  transition: transform .4s;
}
.tc-user-hide {
  transform: translateY(-70px);
}
.tc-user-info {
  float: right;
  padding: 0 20px;
  margin-right: 10px;
  line-height: 60px;
  cursor: pointer;
  background-color: #fff;
  position: relative;
  z-index: 11;
  transition: background-color .6s;
}
.tc-user-ctrl {
  line-height: 60px;
  float: right;
  transition: .6s;
  transform: translateX(100%)
}
.tc-user-ctrl span {
  padding: 0 10px;
  font-size: 14px;
  color: #666;
  cursor: pointer;
  transition: color .6s;
}
.tc-user-ctrl span:hover {
  color: #014676;
}
.tc-user-ctrl-open .tc-user-info {
  background-color: #eef4f9;
}
.tc-user-ctrl-open .tc-user-ctrl {
  transform: translateX(0)
}
#tc-notice {
  position: fixed;
  width: 100vw;
  min-height: 60px;
  bottom: 0;
  left: 0;
  z-index: 20;
  box-shadow: 0px -4px 4px #0001;
  transition: transform .6s;
  transform: translateY(110%);
  padding: 20px;
  box-sizing: border-box;
  line-height: 20px;
  color: #fff;
}

.fly-right-enter-active, .fly-right-leave-active {
  transition: transform 1s;
}
.fly-right-enter, .fly-right-leave-to {
  transform: translateX(100%);
}

.fly-left-enter-active, .fly-left-leave-active {
  transition: transform 1s, opacity 1s;
}
.fly-left-enter, .fly-left-leave-to {
  transform: translateX(-50%);
  opacity: 0;
}
</style>
