<template>
  <button
    type="primary"
    plain
    open-type="getUserInfo"
    @getuserinfo="handleGetUserInfo"
  > 登录 </button>
</template>

<script>
  export default {
    methods:{
      handleGetUserInfo(e){
        // console.log(e);
        const {userInfo}=e.mp.detail;
        mpvue.setStorageSync("userinfo", userInfo);
        mpvue.navigateBack({
          delta: 1
        });
      }
    }
  }
</script>

<style scoped>
  button{
    margin-top: 20px;
    width: 70%;
  }
</style>
