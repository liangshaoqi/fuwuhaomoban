<template>
  <div class="login-wrapper">
    <transition name="slide-down">
      <div class="login-table" v-show="showTable">
        <div class="login-content">
          <div class="img-wrapper"><img src="../../assets/img/logo2.png" alt=""></div>
          <div class="clearfix nameBar">
            <input type="text" id="userName" placeholder="请输入用户名" v-model="userName" :class="{danger:userEmpty}" @focus="userEmpty = false">
          </div>
          <div class="clearfix pwBar">
            <input type="password" @keydown.enter="login" id="password" placeholder="请输入密码" v-model="password" :class="{danger:pwEmpty}" @focus="pwEmpty = false">
          </div>
        </div>
        <!--<div :class="loginInfo?'success':'error'" v-show="showMsg">{{errorText}}</div>-->
        <button @click="login">登录</button>
        <transition name="fade-in">
          <div id="screen" v-show="showScreen">
            <div class="spinner">
              <div class="double-bounce1"></div>
              <div class="double-bounce2"></div>
            </div>
          </div>
        </transition>
      </div>
    </transition>
  </div>
</template>

<script type="text/ecmascript-6">
  import Vue from 'vue'
  import VueCookie from 'vue-cookie'
  import lockr from 'lockr'
  import qs from 'qs'
  import md5 from 'js-md5'
  import { mapMutations,mapState  } from 'vuex'

  Vue.use(VueCookie);
  export default {
    data(){
      return {
        userName: 'admin',
        password: 'admin',
        errorText: '用户名或密码错误',
        userEmpty: false,
        pwEmpty: false,
        showTable: false,
        showMsg: false,
        loginInfo: false,
        showScreen: false
      }
    },
    methods: {
      login(){
        //如果没有用户名
        if (this.userName == '') {
          this.userEmpty = true;
          this.pwEmpty = true;
          this.showMsg = true;
          this.$message.warning('请输入用户名和密码');
          return;
        }
        //如果没有密码
        if (this.password == '') {
          this.pwEmpty = true;
          this.showMsg = true;
          this.$message.warning('请输入密码');
          return;
        }

        this.showScreen = true;
        let data = {
          loginName:this.userName,
          password: md5(this.password)
        };

        //登录成功后将获取的token保存在cookie中
        this.$http.post('/authc/login', data).then((res)=>{
          if(res.data.status === 0){
            this.$cookie.set('oa_adoptToken', res.data.result.adoptToken, 1);
            let data = {};

//            获取头像存入store
            this.getAvatar()

            //请求
            this.$http.post('/getAuthMenus',data).then((res)=>{
              lockr.set("menuInfo",res.data);      //将数据存入localStorage 以便免登陆应用
              this.$router.push('/User/index');    //跳转
              this.showScreen = false;
              this.userName = '';
              this.password = '';
            },(err)=>{
              this.showScreen = false;
//        //          this.$message.error('网络错误');
              this.showScreen = false;
            })

          }else if(res.status === 1){
            this.showScreen = false;
            this.$message.error('用户名不存在');
          }else
          {
            this.showScreen = false;
            this.$message.error('用户名或密码错误');
          }
        },(err)=>{
//  //          this.$message.error('网络错误');
          this.showScreen = false;
        });
      },
      getAvatar(){
        this.$http.get('/serviceInfoConfig').then( (res) => {
          let data = res.data.result
          this.updateAvatar({avatar:data.picturePath})
          this.updateName({name:data.serviceName})
        })
      },
      toMenuArr(arr){
        let arrTemp = [];
        arr.forEach((item)=>{
          if(!item.children){arrTemp.push(item.alias)}
          if(item.children){
            item.children.forEach((list)=>{
              arrTemp.push(list.alias)
            })
          }
        })
        lockr.set("menuArr",arrTemp);
      },
      ...mapMutations(['updateAvatar','updateName'])
    },
//    created(){
//      let _this = this;
//      document.onkeydown = function(e){
//        let key = e.keyCode;
//        if (key === 13){
//            _this.login();
//        }
//      }
//    },
    mounted(){
      this.showTable = true;
    }
  }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="sass">
  .login-wrapper
    position: fixed
    top: 0
    left: 0
    background: url("../../assets/img/login_bg.jpg") center center no-repeat
    width: 100%
    height: 100%
    .login-table
      box-sizing: border-box
      background: #fff
      position: absolute
      left: 50%
      top: 50%
      width: 400px
      margin-top: -160px
      margin-left: -200px
      padding: 40px 30px
      border-radius: 4px
      label
        float: left
        width: 5%
        padding: 0 0 3px 0
        height: 20px
      .glyphicon
        font-size: 12px
        color: #cccccc
        line-height: 20px
      .img-wrapper
        img
          display: block
          margin: 0 auto 50px
      .nameBar
        overflow: hidden
      .pwBar
        overflow: hidden
      input
        border-top: none
        border-left: none
        border-right: none
        border-bottom: 1px solid #eeeeee
        width: 95%
        float: left
        font-size: 12px
        color: #666666
        outline: none
        margin-bottom: 15px
        padding: 0 8px 3px 12px
        line-height: 21px
        &.danger
          border-color: rgba(255, 11, 36, 0.3)
      button
        border-radius: 3px
        background: rgb(122,143,224)
        color: #fff
        font-size: 14px
        font-weight: bold
        margin-top: 10px
        border: none
        outline: none
        width: 100%
        height: 40px
      .error
        background: rgba(255,11,36,0.1)
        border-radius: 3px
        color: rgba(0,0,0,0.5)
        padding: 5px 8px
        margin: 8px 0 15px 0
        height: 20px
        font-size: 14px
      .success
        background: rgb(87, 194, 47)
        border-radius: 3px
        color: #fff
        padding: 5px 8px
        margin: 8px 0 15px 0
        height: 20px
        font-size: 14px

    /*登录框动画*/
    .slide-down-enter-active
      transition: all .8s ease

    .slide-down-leave-active
      transition: all .8s cubic-bezier(1.0, 0.5, 0.8, 1.0)

    .slide-down-enter, .slide-fade-leave-active
      transform: translateY(50px)
      opacity: 0

    /*fadeIn动画 */
    .fade-in-enter-active
      transition: all .8s ease

    .fade-in-leave-active
      transition: all .8s ease

    .fade-in-enter, .fade-in-leave-active
      opacity: 0


    /*loading*/
    #screen
      position: absolute
      width: 100%
      height: 100%
      left: 0
      top: 0
      background: rgba(255,255,255,0.9)
      .spinner
        width: 60px
        height: 60px
        position: relative
        margin: 100px auto

      .double-bounce1
        width: 100%
        height: 100%
        border-radius: 50%
        background-color: rgba(122, 143, 224,0.5)
        position: absolute
        top: 0
        left: 0
        -webkit-animation: bounce 2.0s infinite ease-in-out
        animation: bounce 2.0s infinite ease-in-out

      .double-bounce2
        width: 100%
        height: 100%
        border-radius: 50%
        background-color: rgba(122, 143, 224,0.5)
        position: absolute
        top: 0
        left: 0
        -webkit-animation: bounce 2.0s infinite ease-in-out
        animation: bounce 2.0s infinite ease-in-out
        -webkit-animation-delay: -1.0s
        animation-delay: -1.0s

      @-webkit-keyframes bounce
        0%, 100%
          -webkit-transform: scale(0)

        50%
          -webkit-transform: scale(1)

      @keyframes bounce
        0%, 100%
          transform: scale(0)
          -webkit-transform: scale(0)
        50%
          transform: scale(1)
          -webkit-transform: scale(1)
</style>
