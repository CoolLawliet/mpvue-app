<template>
<div>

  <div class="user_info_wrap">
    <div v-if="userinfo.avatarUrl" class="user_img_wrap">
      <img class="user_bg" :src="userinfo.avatarUrl" />
      <div class="user_info">
        <img class="user_icon" :src="userinfo.avatarUrl"/>
        <div class="user_name">{{userinfo.nickName}}</div>
      </div>
    </div>
    <div v-else class="user_btn">
      <navigator url="/pages/login/main" >登录</navigator>
    </div>
  </div>
  <div class="user_content">

    <div class="user_main">
      <!-- 历史足迹 -->
      <div class="history_wrap">
        <navigator>
          <div class="his_num">0</div>
          <div class="his_name">收藏的店铺</div>
        </navigator>
        <navigator url="/pages/collect/main">
          <div class="his_num">{{collectNums}}</div>
          <div class="his_name">收藏的商品</div>
        </navigator>
        <navigator>
          <div class="his_num">0</div>
          <div class="his_name">关注的商品</div>
        </navigator>
        <navigator>
          <div class="his_num">0</div>
          <div class="his_name">我的足迹</div>
        </navigator>
      </div>
      <!-- 我的订单 -->
      <div class="orders_wrap">
        <div class="orders_title">我的订单</div>
        <div class="order_content">
          <navigator url="/pages/order/main?type=1">
            <div class="iconfont icon-ding_dan"></div>
            <div class="order_name">全部订单</div>
          </navigator>
          <navigator url="/pages/order/main?type=2">
            <div class="iconfont icon-fukuantongzhi"></div>
            <div class="order_name">待付款</div>
          </navigator>
          <navigator url="/pages/order/main?type=3">
            <div class="iconfont icon-receipt-address"></div>
            <div class="order_name">待收货</div>
          </navigator>
          <navigator>
            <div class="iconfont icon-tuihuotuikuan_dianpu"></div>
            <div class="order_name">退款/退货</div>
          </navigator>
        </div>
      </div>
      <!-- 收货地址管理 -->
      <div class="address_wrap">
        收货地址管理
      </div>
      <!-- 应用信息相关 -->
      <div class="app_info_wrap">
        <div class="app_info_item app_info_contact">
          <p>联系客服</p>
          <p>400-618-4000</p>
        </div>
        <navigator url="/pages/feedback/main" class="app_info_item">意见反馈</navigator>
        <div class="app_info_item">关于我们</div>
      </div>
      <!-- 推荐 -->
      <div class="recommend_wrap">
        把应用推荐给其他人
      </div>
    </div>
  </div>
</div>
</template>

<script>
export default {
  data(){
    return{
      userinfo:{},
      // 被收藏的商品的数量
      collectNums:0
    }
  },
  onShow(){
    const userinfo=mpvue.getStorageSync("userinfo");
    const collect=mpvue.getStorageSync("collect")||[];

    this.userinfo = userinfo
    this.collectNums = collect.length
  }
}
</script>

<style lang="wxss">
  page {
    background-color: #edece8;
  }
</style>
<style scoped>
  .user_info_wrap {
    height: 45vh;
    overflow: hidden;
    background-color: var(--themeColor);
    position: relative;
  }
  .user_info_wrap .user_img_wrap {
    position: relative;
  }
  .user_info_wrap .user_img_wrap .user_bg {
    height: 50vh;
    filter: blur(5px);
  }
  .user_info_wrap .user_img_wrap .user_info {
    position: absolute;
    left: 50%;
    transform: translateX(-50%);
    top: 20%;
    text-align: center;
  }
  .user_info_wrap .user_img_wrap .user_info .user_icon {
    width: 75px;
    height: 75px;
    border-radius: 50%;
  }
  .user_info_wrap .user_img_wrap .user_info .user_name {
    color: #fff;
    margin-top: 20px;
  }
  .user_info_wrap .user_btn {
    position: absolute;
    left: 50%;
    transform: translateX(-50%);
    top: 40%;
    border: 0.5px solid greenyellow;
    color: greenyellow;
    font-size: 19px;
    padding: 15px;
    border-radius: 5px;
  }
  .user_content {
    position: relative;
  }
  .user_content .user_main {
    padding-bottom: 50px;
    color: #666;
    position: absolute;
    width: 90%;
    left: 50%;
    transform: translateX(-50%);
    top: -20px;
  }
  .user_content .user_main .history_wrap {
    background-color: #fff;
    display: flex;
  }
  .user_content .user_main .history_wrap navigator {
    flex: 1;
    text-align: center;
    padding: 5px 0;
  }
  .user_content .user_main .history_wrap navigator .his_num {
    color: var(--themeColor);
  }
  .user_content .user_main .orders_wrap {
    background-color: #fff;
    margin-top: 15px;
  }
  .user_content .user_main .orders_wrap .orders_title {
    padding: 10px;
    border-bottom: 0.5px solid #ccc;
  }
  .user_content .user_main .orders_wrap .order_content {
    display: flex;
  }
  .user_content .user_main .orders_wrap .order_content navigator {
    padding: 7.5px 0;
    flex: 1;
    text-align: center;
  }
  .user_content .user_main .orders_wrap .order_content navigator .iconfont {
    color: var(--themeColor);
    font-size: 20px;
  }
  .user_content .user_main .address_wrap {
    margin-top: 15px;
    background-color: #fff;
    padding: 10px ;
  }
  .user_content .user_main .app_info_wrap {
    margin-top: 15px;
    background-color: #fff;
  }
  .user_content .user_main .app_info_wrap .app_info_item {
    padding: 10px ;
    border-bottom: 0.5px solid #ccc;
  }
  .user_content .user_main .app_info_wrap .app_info_contact {
    display: flex;
    justify-content: space-between;
  }
  .user_content .user_main .recommend_wrap {
    margin-top: 15px;
    background-color: #fff;
    padding: 10px ;
  }

</style>
