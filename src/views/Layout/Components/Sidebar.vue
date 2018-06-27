<template>
  <div>
    <el-menu
      class="el-menu-vertical-demo"
      @open="handleOpen"
      @close="handleClose"
      background-color="#F1F1F1"
      text-color="#666"
      active-text-color="#459ffc">
      <router-link :to="{ name: 'Article'}">
        <el-menu-item index="1">
          <i class="el-icon-menu"></i>
          <span slot="title">文章管理</span>
        </el-menu-item>
      </router-link>
      <router-link to="/Menu/EditeMenu">
        <el-menu-item index="2">
          <i class="el-icon-setting"></i>
          <span slot="title">菜单设置</span>
        </el-menu-item>
      </router-link>
    </el-menu>
  </div>
</template>


<style>
  .el-menu-vertical-demo:not(.el-menu--collapse) {
    width: 200px;
    min-height: 400px;
  }
</style>

<script>
  export default {
    data() {
      return {
        isCollapse: true
      };
    },
    methods: {
      handleOpen(key, keyPath) {
        console.log(key, keyPath);
      },
      handleClose(key, keyPath) {
        console.log(key, keyPath);
      }
    }
  }
</script>
