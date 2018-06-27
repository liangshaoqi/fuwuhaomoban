<template>
  <div class="article-wrapper">
    <div class="header">
      <h1>文章共{{totalCount}}条</h1>
      <div class="search">
        <el-input
          clearable
          style="width: 200px"
          placeholder="请输入内容"
          prefix-icon="el-icon-search"
          @input="searchArticle"
          v-model="search">
        </el-input>
        <el-dropdown @command="handleCommand" trigger="click" :show-timeout="0">
          <el-button type="primary">新建</el-button>
          <el-dropdown-menu slot="dropdown">
            <el-dropdown-item command="a">新建图文消息</el-dropdown-item>
            <el-dropdown-item command="b">新建文本消息</el-dropdown-item>
          </el-dropdown-menu>
        </el-dropdown>

      </div>
    </div>
    <div class="table-list">
      <el-table
        :data="tableData"
        style="width: 100%">
        <el-table-column
          label="内容">
          <template slot-scope="scope">
            <img :src="scope.row.cover" style="vertical-align:middle;" alt="">
            <span style="display:inline-block;vertical-align:middle;padding-left: 10px;" v-if="scope.row.target == 2 ">{{scope.row.title}}</span>
            <span style="display:inline-block;vertical-align:middle;width: 500px;word-wrap: break-word;overflow: hidden;" v-if="scope.row.target == 3 ">{{scope.row.message}}</span>
          </template>
        </el-table-column>
        <el-table-column
          label="更新时期"
          width="200">
          <template slot-scope="scope">
            <span>{{(new Date(scope.row.updateTime)).Format("yyyy-M-d h:m:s")}}</span>
          </template>
        </el-table-column>
        <el-table-column
          label="状态"
          width="100">
          <template slot-scope="scope">
            <span v-if="scope.row.publish == 1">已发布</span>
            <span v-if="scope.row.publish == 2">未发布</span>
            <span v-if="scope.row.publish == 3">定时发布</span>
          </template>
        </el-table-column>
        <el-table-column
          label="操作"
          width="220">
          <template slot-scope="scope">
            <el-button
              size="mini"
              type="success"
              @click="handlePreview(scope.$index, scope.row)">预览</el-button>
            <el-button
              size="mini"
              type="primary"
              @click="handleEdit(scope.$index, scope.row)">编辑</el-button>
            <el-button
              size="mini"
              type="danger"
              @click="handleDelete(scope.$index, scope.row)">删除</el-button>
          </template>
        </el-table-column>
      </el-table>
      <div class="pagination-wrapper">
        <el-pagination
          @size-change="handleSizeChange"
          @current-change="handleCurrentChange"
          :current-page="currentPage"
          :page-sizes="[10, 20, 50, 100]"
          :page-size="pageSize"
          layout="total, sizes, prev, pager, next, jumper"
          :total="totalCount">
        </el-pagination>
      </div>
    </div>
  </div>
</template>
<script>
    export default {
      data(){
          return {
            number: 0,
            search: '',
            currentPage:1,
            pageSize:10,
            totalCount: 30,
            tableData: [{
              date: '2016-05-02',
              src:'http://125.208.1.67:6003/file/download/af64824f-0293-4848-b666-6b50a8142605.png',
              text:'123123',
              status: '已发布',
              address: '上海市普陀区金沙江路 1518 弄'
            }]
          }
      },
      mounted(){
        this.getAllArticle()
      },
      methods: {
        getAllArticle(){
          let data = {
            pageSize: this.pageSize,
            currentPage: this.currentPage,
            title: this.search
          }
          this.$http.post('/serviceInfoArticles/page',data).then( (res) => {
            if(res.data.status == 0) {
              this.totalCount = Number(res.data.result.totalCount)
              this.tableData = res.data.result.result
            }
          })
        },
        handleEdit(index, row) {
          if(row.target == 2){
            this.$router.push('/Article/NewArticle/Img/'+row.batchId)
          }
          if(row.target == 3){
            this.$router.push('/Article/NewArticle/Text/'+row.batchId)
          }
        },
        handleDelete(index, row) {
          this.$http.delete('/serviceInfoArticles',{
              params:{
                batchId: row.batchId
              }
          }).then( (res) => {
            if(res.data.status == 0){
              this.$message.success('删除成功')
              this.getAllArticle()
            }
          })
        },
        handleSizeChange(val) {
          this.pageSize = val
          this.getAllArticle()
        },
        handleCurrentChange(val) {
          this.currentPage = val
          this.getAllArticle()
        },
        handleCommand(command) {
          if(command == 'a'){
              this.$router.push('/Article/NewArticle/Img/123')
          }
          if(command == 'b'){
            this.$router.push('/Article/NewArticle/Text/123')
          }
        },
        searchArticle(data){
          this.getAllArticle()
        }
      }
    }
</script>
<style lang="scss" scoped>
  .header {
    overflow: hidden;
    height:40px;
    h1 {
      float: left;
      font-size: 20px;
      line-height: 40px;
    }
    .search {
      float: right;
    }
  }
</style>
