<template>
    <div class="headbar">
      <h1>坐标服务号平台</h1>
      <div class="right-user">
        <el-dropdown @command="handleCommand" trigger="click" :show-timeout="0">
          <div style="cursor: pointer;">
            <div class="img-wrapper">
              <img :src="this.avatar || '/static/img/avatar.3df55f3.jpg'" alt="">
            </div>
            <div class="text-wrapper">
              <span>{{serviceName}}<i class="el-icon-arrow-down el-icon--right"></i></span>
            </div>
          </div>
          <el-dropdown-menu slot="dropdown">
            <el-dropdown-item command="detail">账号详情</el-dropdown-item>
            <el-dropdown-item command="loginOut">退出登录</el-dropdown-item>
          </el-dropdown-menu>
        </el-dropdown>

      </div>
    </div>
</template>

<script>
    import lockr from 'lockr'
    import { mapState  } from 'vuex'

    export default {
        data(){
            return {
                img: ''
            }
        },
        methods:{
          loginOut(){
            lockr.rm("menuInfo")
            lockr.rm("avatar")
            this.$cookie.delete('oa_adoptToken');
            this.$router.push('/login');
          },
          handleCommand(command) {
            if(command === 'loginOut'){
              this.loginOut()
            }
            if(command === 'detail'){
              this.$router.push('/User/index')
            }
          }
        },
        computed: mapState({
          avatar: function(state){ //箭头函数会有this的问题
            let picPath = lockr.get('avatar')
            if(state.userInfo.avatar == '' && picPath){
              this.$store.commit('updateAvatar',{avatar:picPath})//同步操作
            }
            return state.userInfo.avatar
          },
          serviceName: function(state){
            let serviceName = lockr.get('name')
            if(state.userInfo.name == '' && serviceName){
              this.$store.commit('updateName',{name:serviceName})//同步操作
            }
            return state.userInfo.name
          }
        }),
    }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
 .headbar {
   height: 60px;
   line-height: 60px;
   background: #3a3f51;
   color: #fff;
   position: fixed;
   width: 100%;
   z-index:99;
   h1 {
     font-size: 30px;
     text-align: left;
     padding-left: 28px;
     font-weight: 200;
     margin:0;
   }
   .right-user {
     background: #262a3a;
     height: 60px;
     position: absolute;
     right:0;
     top:0;
     padding: 0 20px;
     .img-wrapper {
       float: left;
       height: 60px;
       img {
         height: 40px;
         width: 40px;
         border-radius: 50%;
         margin-top: 10px;
       }
       margin-right: 10px;
     }
     .text-wrapper {
       color:#fff;
       top: 0;
       height: 60px;
       line-height: 60px;
       float: left;
     }
   }
 }
</style>
