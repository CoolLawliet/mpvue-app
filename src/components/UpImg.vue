<template>
  <div class="up_img_wrap">
    <image :src="src"></image>
    <icon type="clear" size="23" color="red">

    </icon>

  </div>
</template>

<script>
    export default {
      props:{
        src:{
          type:String,
          value:""
        }
      }
    }
</script>

<style scoped>
  .up_img_wrap{
    width: 45px;
    height: 45px;
    position: relative;
  }
  .up_img_wrap image{
    width: 100%;
    height: 100%;
    border-radius: 7.5px;
  }
  .up_img_wrap icon{
    position: absolute;
    top:-11px;
    right: -11px;
  }
</style>
