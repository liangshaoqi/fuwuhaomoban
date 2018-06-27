<template>
  <div class="text-article">
    <el-form ref="form" :model="form" label-width="80px" :rules="rules">
      <el-form-item label="正文" prop="message">
        <el-input type="textarea" :rows="6" v-model="form.message"></el-input>
      </el-form-item>
    </el-form>
    <div class="button-group">
      <el-button type="primary">返回</el-button>
      <el-button type="primary" @click="saveArticle">保存草稿</el-button>
      <el-button type="primary">预览</el-button>
      <el-dropdown @command="handleCommand" trigger="click" split-button type="primary" style="margin-left: 8px;" :show-timeout="0">
        发布
        <el-dropdown-menu slot="dropdown">
          <el-dropdown-item command="a">定时发布</el-dropdown-item>
          <el-dropdown-item command="b">立即发布</el-dropdown-item>
        </el-dropdown-menu>
      </el-dropdown>
    </div>
    <el-dialog
      title="选择时间"
      :visible.sync="dialogVisible"
      width="400px"
      style="text-align: center;">
      <el-date-picker
        v-model="dateTime"
        type="datetime"
        placeholder="选择日期时间">
      </el-date-picker>
      <span slot="footer" class="dialog-footer">
          <el-button @click="dialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="dialogOk">确 定</el-button>
        </span>
    </el-dialog>
  </div>
</template>

<script>
  export default {
    data(){
      var validatemessage = (rule, value, callback) => {
        if(this.saveOrPush){
          callback();
        }else{
          if (value === '') {
            callback(new Error('请输入正文'));
          } else {
            callback();
          }
        }
      };
      return {
        form:{
          message:'',
          target: 3,
          orderId:1
        },
        dialogVisible:false,
        dateTime:'',
        rules:{
          message: [
            { validator: validatemessage, trigger: 'blur' }
          ],
        },
      }
    },
    mounted(){
      this.init()
    },
    methods:{
      init(){
        if(this.$route.params.id != 123){
          this.postBatchId = this.$route.params.id
          this.getEditeData()
        }
      },
      getEditeData(){
        this.$http.get('/serviceInfoArticles/selectByBatchId',{
          params:{
            batchId: this.postBatchId
          }
        }).then( (res) => {
          this.form = res.data.result.result[0]
        })
      },
      handleCommand(command) {
        if(command == 'a'){
          this.dialogVisible = true
        }
        if(command == 'b'){
          this.pushArticle()
        }
      },
      dialogOk(){
        this.timePushArticle()
      },
      saveArticle(){
        this.saveOrPush = true
        this.form.publish = 2
        let config = {
          headers: {
            'shouldQs':false
          }
        };
        let data = {
          articles: [this.form]
        }
        if(this.$route.params.id != 123){
          this.$http.put('/serviceInfoArticles',data,config).then( (res) => {
            this.$message({
              showClose: true,
              message: '保存成功...即将返回列表页',
              type: 'success',
              duration: 1000
            });
            setTimeout(()=>{this.$router.push('/Article/index')},1000)
          })
        }else{
          this.$http.post('/serviceInfoArticles',data,config).then( (res) => {
            this.$message({
              showClose: true,
              message: '保存成功...即将返回列表页',
              type: 'success',
              duration: 1000
            });
            setTimeout(()=>{this.$router.push('/Article/index')},1000)
          })
        }
      },
      timePushArticle(){
        this.saveOrPush = false
        let data = {
          batchId: this.postBatchId,
          time: new Date(this.dateTime).getTime()
        }
        this.$refs.form.validate((valid) => {
          if (valid) {
            this.$http.post('/serviceInfoArticles/timingRelease',data).then( (res) => {
              if(res.data.status === 0){
                this.dialogVisible = false
                this.$message({
                  showClose: true,
                  message: '发布成功...即将返回列表页',
                  type: 'success',
                  duration: 1000
                });
                setTimeout(()=>{this.$router.push('/Article/index')},1000)
              }
            })
          } else {
            this.$message.error('请填写完整')
            return false;
          }
        });
      },
      pushArticle(){
        this.saveOrPush = false
        let data = {
          batchId: this.postBatchId
        }
        this.$refs.form.validate((valid) => {
          if (valid) {
            this.$http.post('/article-publish/publish',data).then( (res) => {
              if(res.data.status === 0){
                this.$message({
                  showClose: true,
                  message: '发布成功...即将返回列表页',
                  type: 'success',
                  duration: 1000
                });
                setTimeout(()=>{this.$router.push('/Article/index')},1000)
              }
            })
          } else {
            this.$message.error('请填写完整')
            return false;
          }
        });
      },

    }
  }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
  .text-article {
    overflow: hidden;
    .button-group {
      position: relative;
      width: 100%;
      float: left;
      text-align: center;
    }
  }

</style>
