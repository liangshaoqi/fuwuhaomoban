<template>
    <div class="navbar-container">
      <h2>{{name}}</h2>
    </div>
</template>

<script>
    import lockr from 'lockr'
    import { mapMutations, mapState  } from 'vuex'

    export default {
      methods:{
        ...mapMutations(['updateNavName'])
      },
      computed: mapState({
        name: function(state){//箭头函数会有this的问题
          let navName = lockr.get('navName')
          if(state.userInfo.navName == '' && navName){
            this.$store.commit('updateNavName',{navName:navName})//同步操作
          }
          return state.userInfo.navName
        },
      })
    }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style>
  .navbar-container {
    margin-top: 60px;
  }
</style>
