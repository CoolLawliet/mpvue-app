<template>
  <div>
  <!-- 收货地址 -->
  <div class="revice_address_row">
    <div class="user_info_row" >
      <div class="user_info">
        <div>{{address.userName}}</div>
        <div>{{address.all}}</div>
      </div>
      <div class="user_phone">{{address.telNumber}}</div>
    </div>
  </div>
  <!-- 购物车内容 -->
  <div class="cart_content">
    <div class="cart_title">购物车</div>
    <div class="cart_main">
      <div class="cart_item"
            v-for="item in cart"
            :key="item.goods_id"
      >
        <!-- 商品图片 -->
        <navigator class="cart_img_wrap">
          <img mode="widthFix" :src="item.goods_small_logo" />
        </navigator>
        <!-- 商品信息 -->
        <div class="cart_info_wrap">
          <div class="goods_name">{{item.goods_name}}</div>
          <div class="goods_price_wrap">
            <div class="goods_price">￥{{item.goods_price}}</div>
            <div class="cart_num_tool">
              <div class="goods_num">X {{item.num}}</div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
  <!-- 底部工具栏 -->
  <div class="footer_tool">
    <!-- 总价格 -->
    <div class="total_price_wrap">
      <div class="total_price">
        合计: <text class="total_price_text">￥{{totalPrice}}</text>
      </div>
      <div>包含运费</div>
    </div>
    <div class="order_pay_wrap" @tap="handleOrderPay"  >
      支付({{totalNum}})
    </div>
  </div>
  </div>
</template>

<script>
  import { getSetting, chooseAddress, openSetting, showModal, showToast, requestPayment } from "../../utils/asyncWx.js";
 import {request} from "../../request";

 export default {
   data(){
     return{
       address: {},
       cart: [],
       totalPrice: 0,
       totalNum: 0
     }
   },
   onShow() {
     // 1 获取缓存中的收货地址信息
     const address = mpvue.getStorageSync("address");
     // 1 获取缓存中的购物车数据
     let cart = mpvue.getStorageSync("cart") || [];
     // 过滤后的购物车数组
     cart = cart.filter(v => v.checked);
     this.address = address;

     // 1 总价格 总数量
     let totalPrice = 0;
     let totalNum = 0;
     cart.forEach(v => {
       totalPrice += v.num * v.goods_price;
       totalNum += v.num;
     })
     this.cart = cart;
     this.totalPrice = totalPrice;
     this.totalNum = totalNum;
     this.address = address;
   },
   methods:{
     // 点击 支付
     async handleOrderPay() {
       try {
         // 1 判断缓存中有没有token
         const token = mpvue.getStorageSync("token");
         // 2 判断
         if (!token) {
           mpvue.navigateTo({
             url: '/pages/auth/main'
           });
           return;
         }
         // 3 创建订单
         // 3.1 准备 请求头参数
         // const header = { Authorization: token };
         // 3.2 准备 请求体参数
         const order_price = this.totalPrice;
         const consignee_addr = this.address.all;
         const cart = this.cart;
         let goods = [];
         cart.forEach(v => goods.push({
           goods_id: v.goods_id,
           goods_number: v.num,
           goods_price: v.goods_price
         }))
         const orderParams = { order_price, consignee_addr, goods };
         // 4 准备发送请求 创建订单 获取订单编号
         const { order_number } = await request({ url: "/my/orders/create", method: "POST", data: orderParams });
         // 5 发起 预支付接口
         const { pay } = await request({ url: "/my/orders/req_unifiedorder", method: "POST", data: { order_number } });
         // 6 发起微信支付
         await requestPayment(pay);
         // 7 查询后台 订单状态
         const res = await request({ url: "/my/orders/chkOrder", method: "POST", data: { order_number } });
         await showToast({ title: "支付成功" });
         // 8 手动删除缓存中 已经支付了的商品
         let newCart=mpvue.getStorageSync("cart");
         newCart=newCart.filter(v=>!v.checked);
         mpvue.setStorageSync("cart", newCart);

         // 8 支付成功了 跳转到订单页面
         mpvue.navigateTo({
           url: '/pages/order/main'
         });

       } catch (error) {
         await showToast({ title: "暂无支付功能,支付失败" })
         console.log(error);
       }
     }
   }
 }
</script>

<style lang="wxss">
  page {
  padding-bottom: 45px;
  }
</style>
<style scoped>

  .revice_address_row .user_info_row {
    display: flex;
    padding: 10px;
  }
  .revice_address_row .user_info_row .user_info {
    flex: 5;
  }
  .revice_address_row .user_info_row .user_phone {
    flex: 3;
    text-align: right;
  }
  .cart_content .cart_title {
    padding: 10px;
    font-size: 13px;
    color: var(--themeColor);
    border-top: 0.5px solid currentColor;
    border-bottom: 0.5px solid currentColor;
  }
  .cart_content .cart_main .cart_item {
    display: flex;
    padding: 5px;
    border-bottom: 0.5px solid #ccc;
  }
  .cart_content .cart_main .cart_item .cart_img_wrap {
    flex: 2;
    display: flex;
    justify-content: center;
    align-items: center;
  }
  .cart_content .cart_main .cart_item .cart_img_wrap image {
    width: 80%;
  }
  .cart_content .cart_main .cart_item .cart_info_wrap {
    flex: 4;
    display: flex;
    flex-direction: column;
    justify-content: space-around;
  }
  .cart_content .cart_main .cart_item .cart_info_wrap .goods_name {
    display: -webkit-box;
    overflow: hidden;
    -webkit-box-orient: vertical;
    -webkit-line-clamp: 2;
    color: #666;
  }
  .cart_content .cart_main .cart_item .cart_info_wrap .goods_price_wrap {
    display: flex;
    justify-content: space-between;
  }
  .cart_content .cart_main .cart_item .cart_info_wrap .goods_price_wrap .goods_price {
    color: var(--themeColor);
    font-size: 17px;
  }
  .cart_content .cart_main .cart_item .cart_info_wrap .goods_price_wrap .cart_num_tool {
    display: flex;
  }
  .cart_content .cart_main .cart_item .cart_info_wrap .goods_price_wrap .cart_num_tool .goods_num {
    width: 22.5px;
    height: 22.5px;
    display: flex;
    justify-content: center;
    align-items: center;
  }
  .footer_tool {
    position: fixed;
    bottom: 0;
    left: 0;
    width: 100%;
    height: 45px;
    background-color: #fff;
    display: flex;
    border-top: 0.5px solid #ccc;
  }
  .footer_tool .total_price_wrap {
    flex: 5;
    padding-right: 7.5px;
    text-align: right;
  }
  .footer_tool .total_price_wrap .total_price .total_price_text {
    color: var(--themeColor);
    font-size: 17px;
    font-weight: 600;
  }
  .footer_tool .order_pay_wrap {
    flex: 3;
    background-color: var(--themeColor);
    color: #fff;
    font-size: 16px;
    font-weight: 600;
    display: flex;
    justify-content: center;
    align-items: center;
  }

</style>
