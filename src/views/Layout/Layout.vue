<template>
    <div class="app-wrapper">
      <headbar></headbar>
      <sidebar class="sidebar-container"></sidebar>
      <div class="main-container">
        <navbar></navbar>
        <app-main></app-main>
      </div>
    </div>
</template>

<script>
    import { Navbar, Sidebar, AppMain,Headbar } from './components'
    import lockr from 'lockr'
    import { mapMutations, mapState  } from 'vuex'

    export default {
      name: 'layout',
      components: {
        Headbar,
        Navbar,
        Sidebar,
        AppMain,
      },
      methods:{
        ...mapMutations(['updateNavName'])
      },
      watch: {
        $route (to, from) {
          //检测router 存储meta到store
          this.updateNavName({navName:to.meta.title})
        }
      }
    }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="sass" scoped>

</style>
