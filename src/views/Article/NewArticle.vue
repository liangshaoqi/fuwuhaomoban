<template>
    <div class="article">
      <ImageText v-if="type == 'Img'"></ImageText>
      <TextArticle v-if="type == 'Text'"></TextArticle>
    </div>
</template>

<script>
  import ImageText from './ImageText.vue'
  import TextArticle from './Text.vue'
  export default {
      data(){
          return{
            type: ''
          }
      },
      mounted(){
        this.init()
      },
      methods:{
        init(){
          this.type = this.$route.params.type
        }
      },
      components:{
        ImageText,
        TextArticle
      }
    }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">

</style>
