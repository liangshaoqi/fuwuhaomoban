<template>
  <div class="new-article">
    <div class="left">
      <h4>文章列表</h4>
      <div class="list-wrapper">
        <div v-for="(item,index) in form" class="list-item" @mouseover="hoverIndex = index" @mouseleave="hoverIndex = null"  :class="{active:listIndex == index,'list-item-small':index>0}" @click="listIndex = index">
          <div v-if="index == 0">
            <div class="list-img-cover" v-if="item.cover" :style="{backgroundImage: 'url(' +item.cover+ ')'}">
              <span v-if="!item.title">标题</span>
              <span v-if="item.title">{{item.title}}</span>
            </div>
            <div class="list-cover" v-if="!item.cover">
              <img src="http://125.208.1.67:6003/file/download/895b0021-2554-4784-b992-bf417f7229bd.png" alt="">
              <span v-if="!item.title">标题</span>
              <span v-if="item.title">{{item.title}}</span>
            </div>
            <div class="handleIcon" v-if="hoverIndex == index">
              <a href="javascript:;" title="下移"><i class="el-icon-sort-down down" @click="swapItemsDown(index)"></i></a>
            </div>
          </div>
          <div v-if="index != 0">
            <span v-if="!item.title">标题</span>
            <span v-if="item.title">{{item.title}}</span>
            <div class="list-img" v-if="item.cover" :style="{backgroundImage: 'url(' +item.cover+ ')'}">
            </div>
            <div class="list-cover" v-if="!item.cover">
              <img src="http://125.208.1.67:6003/file/download/895b0021-2554-4784-b992-bf417f7229bd.png" alt="">
            </div>
            <div class="handleIcon" v-if="hoverIndex == index">
              <a href="javascript:;" title="上移"><i class="el-icon-sort-up up" @click="swapItemsUp(index)"></i></a>
              <a href="javascript:;" v-if="index != count" title="下移"><i class="el-icon-sort-down down" @click="swapItemsDown(index)"></i></a>
              <a href="javascript:;" title="删除"><i class="el-icon-delete delete" @click="deleteActicle(index)"></i></a>
            </div>
          </div>
        </div>
        <div class="list-add" @click="addArticleList" v-if="count < 3">
          <i class="el-icon-plus"></i>
        </div>
      </div>
    </div>
    <div class="middle"></div>
    <div class="right">
      <el-form :ref="'form'+index" v-for="(item,index) in form" :key="index" :model="item" status-icon label-width="100px" label-position="left" v-show="listIndex == index" :rules="rules">
        <el-form-item label="类型">
          <el-radio-group v-model="item.target">
            <el-radio-button label="1">富文本</el-radio-button>
            <el-radio-button label="2">链接</el-radio-button>
          </el-radio-group>
        </el-form-item>
        <el-form-item label="标题" prop="title">
          <el-input v-model="item.title" :maxlength="50" @change="titleChange" style="width: 350px"></el-input>
        </el-form-item>
        <el-form-item label="正文链接" prop="urlPath">
          <el-input v-model="item.urlPath" style="width: 350px"></el-input>
        </el-form-item>
        <el-form-item label="上传封面图" prop="cover">
          <div class="chooseImage" v-if="index == 0"  @click="toggleShow"  :style="{backgroundImage: 'url(' +item.cover+ ')'}"><i v-if="!item.cover" class="el-icon-plus avatar-uploader-icon"></i>
          </div>
          <div class="chooseImage chooseImage2" v-if="index > 0" @click="toggleShow2"  :style="{backgroundImage: 'url(' +item.cover+ ')'}"><i v-if="!item.cover" class="el-icon-plus avatar-uploader-icon"></i>
          </div>
          <!--<input type="file" id="chooseImage" style="position:absolute; clip:rect(0 0 0 0);" accept="image/png, image/jpeg, image/gif, image/jpg" @change="uploadImg($event)">-->
        </el-form-item>
        <el-form-item label="摘要">
          <el-input type="textarea" :autosize="{ minRows: 6}" v-model="item.summary" style="width: 350px"></el-input>
        </el-form-item>
      </el-form>
    </div>
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
    <my-upload field="img"
               @crop-success="cropSuccess"
               @crop-upload-success="cropUploadSuccess"
               @crop-upload-fail="cropUploadFail"
               v-model="show"
               :noCircle="true"
               :range="true"
               :width="213"
               :height="116"
               ref="uploadCroper"
               url="http://192.168.1.21:9969/materialInfoImage"
               :params="params"
               :headers="headers"
               img-format="png"></my-upload>
    <my-upload field="img"
               @crop-success="cropSuccess2"
               @crop-upload-success="cropUploadSuccess2"
               @crop-upload-fail="cropUploadFail2"
               v-model="show2"
               :noCircle="true"
               :range="true"
               :width="200"
               :height="200"
               ref="uploadCroper"
               url="http://192.168.1.21:9969/materialInfoImage"
               :params="params"
               :headers="headers"
               img-format="png"></my-upload>
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
  import 'babel-polyfill'; // es6 shim
  import myUpload from '@/components/image-cropper.vue';

  export default {
    data(){
      var validatetitle = (rule, value, callback) => {
        if(this.saveOrPush){
          callback();
        }else{
          if (value === '') {
            callback(new Error('请输入标题'));
          } else if (value.length > 50) {
            callback(new Error('不能大于50字符'));
          } else {
            callback();
          }
        }
      };
      var validateurlPath = (rule, value, callback) => {
        let regexp = /((http|https):\/\/([\w\-]+\.)+[\w\-]+(\/[\w\u4e00-\u9fa5\-\.\/?\@\%\!\&=\+\~\:\#\;\,]*)?)/ig;
        if(this.saveOrPush){
          callback();
        }else{
          if (value === '') {
            callback(new Error('请输入链接地址'));
          } else if (!value.match(regexp)) {
            callback(new Error('请输入正确的链接地址'));
          } else {
            callback();
          }
        }
      };
      var validatecover = (rule, value, callback) => {
        if(this.saveOrPush){
          callback();
        }else{
          if (value === '') {
            callback(new Error('请上传封面图'));
          } else {
            callback();
          }
        }
      };
      return {
        dateTime: '',
        dialogVisible:false,
        show: false,
        show2: false,
        src:'',
        rules:{
          title: [
            { validator: validatetitle, trigger: 'blur' }
          ],
          urlPath: [
            { validator: validateurlPath, trigger: 'blur' }
          ],
          cover: [
            { validator: validatecover, trigger: 'blur' }
          ],
        },
        form:[
          {
            title:'',
            urlPath:'',
            target:'2',
            summary:'',
            cover: ''
          }
        ],
        params: {
          token: '123456798',
          name: 'avatar'
        },
        headers: {
          adoptToken: ''
        },
        listIndex: 0,
        count: 0,
        hoverIndex: null,
        postBatchId: null,
        saveOrPush:false
      }
    },
    mounted(){
      this.headers.adoptToken = this.$cookie.get('oa_adoptToken')
      this.init()
    },
    components: {
      'my-upload': myUpload
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
          this.form = res.data.result.result
          this.count = this.form.length-1
        })
      },
      toggleShow(){
        this.show = true
        this.$refs.uploadCroper.refresh()
      },
      toggleShow2(){
        this.show2 = true
        this.$refs.uploadCroper.refresh()
      },
      cropSuccess(imgDataUrl, field){
        console.log('-------- crop success --------');
      },

      cropUploadSuccess(jsonData, field){
        console.log('-------- upload success --------');
        let path = jsonData.result.result[0].path
        this.form[this.listIndex].cover = path;
        console.log('field: ' + field);
      },

      cropUploadFail(status, field){
        this.$message('上传失败')
      },
      cropSuccess2(imgDataUrl, field){
        console.log('-------- crop success --------');
      },
      cropUploadSuccess2(jsonData, field){
        console.log('-------- upload success --------');
        let path = jsonData.result.result[0].path
        this.form[this.listIndex].cover = path;
        console.log('field: ' + field);
      },
      cropUploadFail2(status, field){
        this.$message('上传失败')
      },
      titleChange(data){
        console.log(data)
      },
      saveArticle(){
        this.saveOrPush = true
        this.form.forEach((item,index)=>{
          item.orderId = ++index
          item.publish = 2
        })
        let config = {
          headers: {
            'shouldQs':false
          }
        };
        let data = {
          articles: this.form
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
        let count = 0
        this.form.forEach((item,index)=>{
          this.$refs['form'+index][0].validate((valid) => {
            if (valid) {
              count++
            } else {
              this.$message.error('请填写完整')
              return false;
            }
          });
        })
        if(count === this.form.length){
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
        }
      },
      pushArticle(){
        this.saveOrPush = false
        let data = {
          batchId: this.postBatchId
        }
        let count = 0
        this.form.forEach((item,index)=>{
          this.$refs['form'+index][0].validate((valid) => {
            if (valid) {
              count++
            } else {
              this.$message.error('请填写完整')
              return false;
            }
          });
        })
        if(count === this.form.length){
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
        }
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
      addArticleList(){
        let data = {
          title:'',
          urlPath:'',
          target:'2',
          summary:'',
          cover: ''
        }
        if(this.count >= 4) return
        this.count++
        this.form.push(data)
        this.form[this.count].index = this.count
      },
      deleteActicle(index){
        console.log(this.form.splice(index,1))
      },
      //上移
      swapItemsUp(index){
        this.form[index] = this.form.splice(index-1,1,this.form[index])[0]
      },
      swapItemsDown(index){
        this.form[index] = this.form.splice(index+1,1,this.form[index])[0]
      }
    }
  }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
  .new-article {
    overflow: hidden;
    position: relative;
    .left {
      width: 270px;
      float: left;
      padding:0 20px 0 15px;

      h4 {
        text-align: center;
      }
      .list-wrapper {
        padding-top: 15px;
        .list-item {
          padding: 14px 9px;
          margin-bottom:10px;
          border: 2px solid #eee;
          position: relative;
          .list-cover {
            padding: 40px 30px;
            background: #ececec;
            text-align: center;
            position: relative;
            img {
              width: 30%;
            }
            span {
              display: inline-block;
              width: 100%;
              padding: 0 15px;
              text-align: center;
              position: absolute;
              bottom:0;
              left:0;
              height: 30px;
              line-height: 30px;
              background: rgba(0,0,0,0.4);
              color: #fff;
              overflow: hidden;
              word-wrap:break-word;
              word-break: break-all;
            }
          }
          &.active {
            border: 2px solid #48b54d!important;
          }
          .list-img {
            width: 100%;
            background-size: cover;
            background-repeat: no-repeat;
            background-position: center center;
          }
          .list-img-cover {
            position: relative;
            width: 213px;
            height: 116px;
            overflow: hidden;
            background-size: cover;
            background-repeat: no-repeat;
            background-position: center center;
            span{
              display: block;
              width: 100%;
              text-align: center;
              position: absolute;
              bottom:0;
              left:0;
              height: 30px;
              line-height: 30px;
              background: rgba(0,0,0,0.4);
              color: #fff;
            }
            img {
              width: 100%;
            }
          }
          &.list-item-small {
            overflow: hidden;
            .list-cover {
              width: 100px;
              height: 100px;
              background: #ececec;
              text-align: center;
              float: right;
              display: inline-block;
              vertical-align: middle;
              img {
                width: 30px;
              }
            }
            span {
            }
            .list-img {
              width: 100px;
              height: 100px;
              float: right;
              display: inline-block;
              vertical-align: middle;
              img {
                width: 100%;
              }
            }
          }
          .handleIcon {
            display: inline-block;
            position: absolute;
            bottom: 0;
            left:0;
            width: 100%;
            background: rgba(0,0,0,0.3);
            height: 45px;
            line-height: 45px;
            text-align: center;
            color: #fff;
            font-size: 24px;
            .up {
              position: absolute;
              left: 5%;
              top:23%;
              cursor: pointer;
            }
            .down {
              position: absolute;
              left: 20%;
              top:23%;
              cursor: pointer;
            }
            .delete {
              position: absolute;
              right: 5%;
              top:23%;
              cursor: pointer;
            }
          }
        }
        .list-add {
          padding: 14px 9px;
          border: 2px dashed #eee;
          i {
            width: 213px;
            height: 100px;
            text-align: center;
            line-height:100px;
            font-size: 50px;
          }
          &:hover {
            border-color: #48b54d;
          }
        }
      }
    }
    .middle {
      float: right;
      border-right:1px solid #e7e7eb;
    }
    .right {
      float: left;
      margin-left: 50px;
      .list-cover {
        padding: 35px 25px;
        background: #ececec;
        text-align: center;
        display: inline-block;
        img {
          width: 30%;
        }
      }
      .chooseImage {
        display: inline-block;
        width: 178px;
        height:110px;
        border: 1px dashed #d9d9d9;
        border-radius: 6px;
        cursor: pointer;
        position: relative;
        overflow: hidden;
        background-size: cover;
        background-repeat: no-repeat;
        background-position: center center;
        &:hover {
          border-color: #409EFF;
        }
        .avatar-uploader-icon {
          font-size: 28px;
          color: #8c939d;
          width: 178px;
          height: 110px;
          line-height: 110px;
          text-align: center;
        }
      }
      .chooseImage2 {
        width: 110px;
        height:110px;
        background-size: cover;
        background-repeat: no-repeat;
        background-position: center center;
        &:hover {
          border-color: #409EFF;
        }
        .avatar-uploader-icon {
          font-size: 28px;
          color: #8c939d;
          width: 110px;
          height: 110px;
          line-height: 110px;
          text-align: center;
        }
        img {
          width: 100%;
        }
      }
      .filePic {
        width: 200px;
        vertical-align: bottom;
      }
    }
    .button-group {
      border-top:1px solid #e7e7eb;
      margin-top: 15px;
      padding-top: 20px;
      position: relative;
      width: 100%;
      float: left;
      text-align: center;
    }
  }

</style>
