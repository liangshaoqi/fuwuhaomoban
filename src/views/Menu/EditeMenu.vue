<template>
    <div class="edite-menu">
      <div class="tip">
        <h4>菜单编辑中</h4>
        <p>菜单未发布，请确认菜单编辑完成后点击“发布”同步到手机。</p>
      </div>
      <!--设置页面-->
      <div class="menu_preview_area">
        <div class="mobile_menu_preview">
          <div class="mobile_bd">
            <ul class="pre_menu_list">
              <draggable v-model="form" :options="{disabled:!isSorting,animation:150}" @start="drag=true" @end="drag=true" @update="datadragEnd">
                <!--有菜单的时候三个-->
                <li v-for="(item,index) in form" :key="index" class="pre_menu_list_li" :class="{active:(activeIndex == index) && activeMenuBoolean}" @click="activeMenu(index)">
                  <span v-if="item.menuName != ''">{{item.menuName}}</span>
                  <span v-if="item.menuName == ''">菜单名称</span>
                  <ul class="pre_menu_list_child" v-show="activeIndex == index" v-if="form.length != 0" :class="{noBorder: isSorting && item['sub_menu'].length == 0 }">
                    <draggable v-model="item['sub_menu']" :options="{group:'people'}" @start="drag=true" @end="drag=false" @update="datadragEnd2">
                      <li v-for="(list,index1) in item['sub_menu']" :class="{noTopBorder:index1 == 0,active:(activeChildIndex == index1) && activeChilMenuBoolean}" @click.stop="activeChilMenu(index1);">
                        <span v-if="list.menuName != ''">{{list.menuName}}</span>
                        <span v-if="list.menuName == ''">子菜单名称</span>
                      </li>
                      <li class="addChildMenu" :class="{noTopBorder:item['sub_menu'].length == 0}" @click.stop="addChildMenu(index)" v-if="item['sub_menu'].length <= 4 && !isSorting">
                        <span><i class="el-icon-plus"></i></span>
                      </li>
                    </draggable>
                  </ul>
                </li>
                <!--没有菜单的时候或者小于2个的时候-->
                <li class="addMenu pre_menu_list_li" @click="addMenu" v-if="form.length <= 2 && !isSorting">
                  <span><i class="el-icon-plus"></i></span>
                  <span v-if="form.length == 0">添加菜单</span>
                </li>
              </draggable>
            </ul>
          </div>
        </div>
      </div>
      <!--表单-->
      <div class="menu_form_area">
        <div class="js_none" v-if="isSorting">
          请通过拖拽左边的菜单进行排序
        </div>
        <div class="js_rightBox" v-if="!isSorting">
          <div class="editor_inner">
            <div class="editor_header">
              <span style="float: left;">{{menuForm.name ? menuForm.name : menuTopName}}</span>
              <span style="float: right;color: #337ab7;cursor: pointer;" @click="deleteMenu">删除菜单</span>
            </div>
            <div class="editor_body">
              <el-form ref="form" :model="menuForm" label-width="80px" label-position="left" :rules="rules">
                <el-form-item label="菜单名称" prop="name">
                  <el-input style="width: 300px" v-model="menuForm.name" placeholder="请输入菜单名称" @blur="saveMenuName"></el-input>
                  <br>
                  <span style="color: #8d8d8d;">字数不超过4个汉字或8个字母</span>
                </el-form-item>
                <el-form-item label="菜单内容" v-if="!isCatalog">
                  <el-radio-group v-model="menuForm.type" size="medium">
                    <el-radio border label="跳转网页"></el-radio>
                  </el-radio-group>
                </el-form-item>
                <div class="url-wrapper" v-if="!isCatalog">
                  <span style="color: #8d8d8d;">阅读者点击该子菜单会跳到以下链接</span>
                  <el-form-item label="页面地址" prop="urlPath">
                    <el-input style="width: 300px" v-model="menuForm.urlPath" placeholder="请输入跳转链接地址" @blur="saveUrlPath"></el-input>
                    <br>
                  </el-form-item>
                </div>
              </el-form>
            </div>

          </div>
        </div>
        <div class="editor_footer"  v-if="!isSorting">
          <el-button @click="validateForm">保存</el-button>
        </div>
      </div>
      <!--按钮组-->
      <div class="btn-wrapper">
        <el-button type="success" class="sort_btn" @click="sortMenu" v-if="!isSorting" :disabled="cantSort">菜单排序</el-button>
        <el-button type="success" class="sort_btn" @click="sortMenuDone" v-if="isSorting">完成</el-button>
        <el-button type="primary" class="save_btn" v-if="!isSorting" @click="pushMenu">发布</el-button>
        <el-button class="preview_btn" v-if="!isSorting" @click="previewPhone">预览</el-button>
      </div>
      <!--预览-->
      <div class="preview_phone" v-if="phonePre">
        <div class="phone_bg">
          <ul class="pre_menu_list">
            <li v-for="(item,index) in form" :key="index" class="pre_menu_list_li" @click="preActiveIndex = index">
              <span v-if="item.menuName != ''">{{item.menuName}}</span>
              <span v-if="item.menuName == ''">菜单名称</span>
              <ul class="pre_menu_list_child" v-show="preActiveIndex == index" v-if="item['sub_menu'].length != 0" :class="{noBorder: isSorting && item['sub_menu'].length == 0 }">
                <li v-for="(list,index1) in item['sub_menu']" :class="{noTopBorder:index1 == 0,active:(activeChildIndex == index1) && activeChilMenuBoolean}">
                  <span v-if="list.menuName != ''">{{list.menuName}}</span>
                  <span v-if="list.menuName == ''">子菜单名称</span>
                </li>
                <i class="arrow arrow_in"></i>
              </ul>
            </li>
          </ul>
        </div>
        <el-button  class="preOutBtn" @click="phonePre = false">退出预览</el-button>
      </div>
    </div>
</template>

<script>
    import draggable from 'vuedraggable'

    export default {
      data(){
          var validatename = (rule, value, callback) => {
              let len = this.getByteLen(value)
              if (value === '') {
                callback(new Error('请输入菜单名'));
              } else if (len > 8) {
                callback(new Error('不能大于8个字符'));
              } else {
                callback();
              }
          };
          return {
            form:[],
            menuForm:{
              name:'',
              type: '跳转网页',
              urlPath:''
            },
            activeIndex:0,
            activeChildIndex:0,
            activeChilMenuBoolean:false,
            activeMenuBoolean:true,
            isCatalog: true,
            menuTopName: '',
            isSorting: false,
            phonePre: false,
            cantSort: false,
            preActiveIndex: 0,
            rules:{
              name: [
                { validator: validatename, trigger: 'blur',required: true },
              ],
              urlPath: [
                { required: true, message: '请输入文章地址', trigger: 'blur' },
                { type:"url", message: '请输入正确的链接', trigger: 'blur' }
              ],
            }
          }
      },
      mounted(){
        this.init()
      },
      components: {
        draggable,
      },
      methods:{
        init(){
          this.getMenu()
        },
        getMenu(){
          this.$http.get('/serviceInfoMenus').then( (res) => {
            this.form = res.data.result
            if(this.form[0].menuName != ''){
              this.menuForm.name = this.form[0].menuName
            }else {
              this.menuTopName = '菜单名称'
            }
            if(this.form[0]['sub_menu'].length == 0 ){
                this.isCatalog = false
            }
          })
        },
        addMenu(){
          let data = {
            menuName :"",
            menuType  :2,
            urlPath  : '',
            target :2,
            'sub_menu':[]
          }

          this.form.push(data)

          this.pushForm()

        },
        addChildMenu(index){
          let data = {
            menuName : '',
            urlPath  : '',
            menuType  :2,
            target :2,
          }

          this.form[index].menuType = 1

          this.isCatalog = true

          this.form[index]['sub_menu'].push(data)

          this.pushForm()
        },
        validateForm(){
          this.$refs['form'].validate((valid) => {
            if (valid) {
              this.saveEditeForm()
            } else {
              this.$message.error('请填写完整后再进行保存')
              return false;
            }
          });
        },
        saveEditeForm(){
          if(this.getByteLen(this.menuForm.name) > 8){
            this.$message.error('字数不超过4个汉字或8个字母')
            return
          }
          if(this.activeMenuBoolean){
            this.form[this.activeIndex].menuName = this.menuForm.name
            this.form[this.activeIndex].urlPath = this.menuForm.urlPath
          }
          if(this.activeChilMenuBoolean){
            this.form[this.activeIndex]['sub_menu'][this.activeChildIndex].menuName = this.menuForm.name
            this.form[this.activeIndex]['sub_menu'][this.activeChildIndex].urlPath = this.menuForm.urlPath
          }
          this.pushForm(this.$message.success('保存成功'))
        },
        saveMenuName(){
        },
        saveUrlPath(){
        },
        //选中主菜单,如果没有子菜单就是跳转,有的话就是修改名字
        activeMenu(index){
          this.menuTopName = '菜单名称'
          this.activeChilMenuBoolean = false
          this.activeMenuBoolean = true
          this.activeIndex = index
          this.menuForm.name = this.form[this.activeIndex].menuName
          this.menuForm.urlPath = this.form[this.activeIndex].urlPath
          if(this.form[this.activeIndex]['sub_menu'].length > 0){
              this.isCatalog = true
          }else {
            this.isCatalog = false
          }
        },
        activeChilMenu(index){
          this.menuTopName = '子菜单名称'
          this.isCatalog = false
          this.activeChilMenuBoolean = true
          this.activeMenuBoolean = false
          this.activeChildIndex = index
          this.menuForm.name = this.form[this.activeIndex]['sub_menu'][this.activeChildIndex].menuName
          this.menuForm.urlPath = this.form[this.activeIndex]['sub_menu'][this.activeChildIndex].urlPath
        },
        deleteMenu(){
          if(this.activeMenuBoolean){
            this.form.splice(this.activeIndex,1)
          }
          if(this.activeChilMenuBoolean){
            this.form[this.activeIndex]['sub_menu'].splice(this.activeChildIndex,1)
          }
          this.pushForm()
          if(this.form.length != 0){
            this.menuForm.name = this.form[this.activeIndex].menuName
            this.menuForm.urlPath = this.form[this.activeIndex].urlPath
          }
          this.menuForm.name = this.form[this.activeIndex]['sub_menu'][this.activeChildIndex].menuName
          this.menuForm.urlPath = this.form[this.activeIndex]['sub_menu'][this.activeChildIndex].urlPath
        },
        pushForm(callback){
          let config = {
            headers: {
              'Content-Type': 'application/json',
              'shouldQs':false
            }
          };
          this.$http.put('/serviceInfoMenus',this.form,config).then((res) => {
              if(res.data.status == 0){
                  if(callback){
                    callback()
                  }
              }
          })
        },
        sortMenu(){
          this.isSorting = true
        },
        sortMenuDone(){
          this.isSorting = false
          this.pushForm()
        },
        datadragEnd (evt) {
          this.activeIndex = evt.newIndex
        },
        datadragEnd2 (evt){
          this.activeChildIndex = evt.newIndex
        },
        previewPhone(){
          this.phonePre = true
        },
        pushMenu(){
          for (var i = 0;i<this.form.length;i++){
              if(this.form[i].menuName == ''){
                let tempName = this.menuForm.name
                let tempUrl = this.menuForm.urlPath
                this.activeIndex = i
                this.activeChilMenuBoolean = false
                this.activeMenuBoolean = true
                this.activeMenu(i)
                this.menuForm.name = tempName
                this.menuForm.urlPath = tempUrl
                this.$refs['form'].validate((valid) => {
                  if (!valid) {
                    this.$message.error('请将相关项填写完整后再进行发布')
                    return false;
                  }
                });
              }else {
                  for (var j = 0;j<this.form[i]['sub_menu'].length;j++){
                      if(this.form[i]['sub_menu'][j].menuName == ''){
                          let tempName = this.menuForm.name
                          let tempUrl = this.menuForm.urlPath
                          this.activeIndex = i
                          this.activeChildIndex = j
                          this.activeChilMenuBoolean = true
                          this.activeMenuBoolean = false
                          this.activeChilMenu(j)
                          this.menuForm.name = tempName
                          this.menuForm.urlPath = tempUrl
                          this.$refs['form'].validate((valid) => {
                            if (!valid) {
                              this.$message.error('请将相关项填写完整后再进行发布')
                              return false;
                            }
                          });
                      }
                  }
              }
          }
          this.$http.post('/serviceInfoMenus/release').then( (res) => {
              if(res.data.status == 0){
                  this.$message.success('发布成功')
              }
          })
        },
        //字符长度
        getByteLen(val) {
          var len = 0;
          for (let i = 0; i < val.length; i++) {
            var length = val.charCodeAt(i);
            if(length>=0&&length<=128)
            {
              len += 1;
            }
            else
            {
              len += 2;
            }
          }
          return len;
        }
      }

    }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="scss" scoped>
  .edite-menu {
    padding: 0 20px 40px;
    overflow: hidden;
    .tip {
      padding: 8px 16px;
      background-color: #ecf8ff;
      border-radius: 4px;
      border-left: 5px solid #50bfff;
      margin: 20px 0;
      h4 {
        font-size: 14px;
      }
      p {
        font-size: 12px;
      }
    }
    .menu_preview_area {
      float: left;
      margin-right: 20px;
      position: relative;
      .mobile_menu_preview {
        width: 320px;
        height: 580px;
        -webkit-background-size: contain;
        background-size: contain;
        position: relative;
        background: transparent url('../../assets/img/bg_mobile_head.png') no-repeat 0 0;
        background-position: 0 0;
        border: 1px solid #e7e7eb;
        .mobile_bd {
          .pre_menu_list{
            position: absolute;
            bottom: 0;
            left: 0;
            right: 0;
            border-top: 1px solid #e7e7eb;
            background: transparent url('../../assets/img/bg_mobile_foot.png') no-repeat 0 0;
            background-position: 0 0;
            background-repeat: no-repeat;
            padding-left: 43px;
            height: 50px;
            &>div {
              display: flex;
              .pre_menu_list_li {
                flex:1;
                line-height: 50px;
                text-align: center;
                font-size:12px;
                border-left: 1px solid #e7e7eb;
                position: relative;
                &.active {
                  border: 1px solid #44b549;
                  color: #44b549;
                  height: 50px;
                }
                .pre_menu_list_child {
                  border: 1px solid #d0d0d0;
                  width: 100%;
                  position: absolute;
                  left:0;
                  bottom: 60px;
                  li {
                    display: inline-block;
                    width: 100%;
                    color: #616161;
                    border-top: 1px solid #d0d0d0;
                    &.noTopBorder {
                      border-top: none;
                    }
                    &.active {
                      border: 1px solid #44b549;
                      color: #44b549;
                    }
                  }
                  &.noBorder {
                    border: none;
                  }
                }
              }
            }
          }
        }
      }
    }
    .menu_form_area {
      padding: 10px 30px;
      min-height:580px;
      background: #F4F5F9;
      position: relative;
      overflow: hidden;
      .js_rightBox {
        padding-left: 12px;
        position: relative;
        .editor_header{
          overflow: hidden;
          font-size: 14px;
          padding-bottom: 10px;
          border-bottom:1px solid #e7e7eb;
        }
        .editor_body {
          padding-top: 20px;
          .url-wrapper {
            padding: 20px;
            background: #fff;
            border:1px solid #e7e7eb;
            span {
              display: inline-block;
              padding-bottom: 20px;
            }
          }
        }
      }
      .editor_footer {
        position: absolute;
        bottom: 20px;
        text-align: center;
        width: 100%;
      }
      .js_none {
        text-align: center;
        padding-top: 200px;
        color: #8d8d8d;
      }
    }
    .btn-wrapper {
      float: left;
      margin-top: 30px;
      .sort_btn {
        margin-left: 120px;
      }
      .save_btn {
        margin-left: 170px;
      }
    }
    .preview_phone {
      position: fixed;
      top: 0;
      right: 0;
      bottom: 0;
      left: 0;
      overflow: auto;
      background: rgba(0,0,0,0.5);
      z-index:2001;
      .phone_bg {
        width: 320px;
        height: 640px;
        position: relative;
        left: 50%;
        top: 10%;
        margin-left: -160px;
        background: transparent url('../../assets/img/iphone-x.png') no-repeat 0 0;
        background-position: 0 0;
        background-repeat: no-repeat;
        z-index:2002;
        .pre_menu_list{
          position: absolute;
          bottom: 43px;
          left: 60px;
          right: 21px;
          border-top: 1px solid #e7e7eb;
          background: #FAFAFA;
          display: flex;
          height: 43px;
          .pre_menu_list_li {
            flex:1;
            line-height: 43px;
            text-align: center;
            font-size:12px;
            border-left: 1px solid #e7e7eb;
            position: relative;
            &.active {
              border: 1px solid #44b549;
              color: #44b549;
              height: 50px;
            }
            .pre_menu_list_child {
              border: 1px solid #d0d0d0;
              width: 100%;
              position: absolute;
              left:0;
              bottom: 55px;
              background: #FAFAFA;
              li {
                display: inline-block;
                width: 100%;
                color: #616161;
                border-top: 1px solid #d0d0d0;
                &.noTopBorder {
                  border-top: none;
                }
                &.active {
                  border: 1px solid #44b549;
                  color: #44b549;
                }
              }
              &.noBorder {
                border: none;
              }
            }
            .arrow_in {
              bottom: -5px;
              display: inline-block;
              width: 0;
              height: 0;
              border-width: 6px;
              border-style: dashed;
              border-color: transparent;
              border-bottom-width: 0;
              border-top-color: #fafafa;
              border-top-style: solid;
            }
            .arrow {
              position: absolute;
              left: 50%;
              margin-left: -6px;
            }
          }
        }
      }
      .preOutBtn {
        position: absolute;
        bottom: 120px;
        left: 46.5%;
      }
    }
  }
</style>
