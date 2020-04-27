<template>
  <div>
    <!-- 收货地址 -->
    <div class="revice_address_row">
      <!-- 当收货地址 不存在 按钮显示  对象 空对象 bool类型也是true  -->
      <div class="address_btn" v-if="!address.userName">
        <button @tap="handleChooseAddress" type="primary" plain>获取收货地址</button>
      </div>
      <!-- 当收货地址 存在 详细信息就显示 -->
      <div v-else class="user_info_row">
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
        <!-- 当cart数组 长度不为0 显示 商品信息 -->
        <block v-if="cart.length!==0">
          <div class="cart_item"
               v-for="item in cart"
               :key="item.goods_id"
          >
            <!-- 复选框 -->
            <div class="cart_chk_wrap">
              <checkbox-group :data-id="item.goods_id" @change="handeItemChange">
                <checkbox :checked="item.checked"></checkbox>
              </checkbox-group>
            </div>
            <!-- 商品图片 -->
            <navigator class="cart_img_wrap">
              <img mode="widthFix" :src="item.goods_small_logo"/>
            </navigator>
            <!-- 商品信息 -->
            <div class="cart_info_wrap">
              <div class="goods_name">{{item.goods_name}}</div>
              <div class="goods_price_wrap">
                <div class="goods_price">￥{{item.goods_price}}</div>
                <div class="cart_num_tool">
                  <div @tap="handleItemNumEdit" :data-id="item.goods_id" :data-operation="-1" class="num_edit">-</div>
                  <div class="goods_num">{{item.num}}</div>
                  <div @tap="handleItemNumEdit" :data-id="item.goods_id" :data-operation="1" class="num_edit">+</div>
                </div>
              </div>
            </div>
          </div>
        </block>
        <block v-else>
          <img mode="widthFix"
               src="http://hbimg.b0.upaiyun.com/e1b1467beea0a9c7d6a56b32bac6d7e5dcd914f7c3e6-YTwUd6_fw658"/>
        </block>
      </div>
    </div>

    <!-- 底部工具栏 -->
    <div class="footer_tool">
      <!-- 全选 -->
      <div class="all_chk_wrap">
        <checkbox-group @change="handleItemAllCheck">
          <checkbox :checked="allChecked">全选</checkbox>
        </checkbox-group>
      </div>
      <!-- 总价格 -->
      <div class="total_price_wrap">
        <div class="total_price">
          合计: <p class="total_price_text">￥{{totalPrice}}</p>
        </div>
        <div>包含运费</div>
      </div>
      <!-- 结算 -->
      <div class="order_pay_wrap" @tap="handlePay">
        结算({{totalNum}})
      </div>
    </div>
  </div>
</template>

<script>
  import {getSetting, chooseAddress, openSetting, showModal, showToast} from "../../utils/asyncWx.js";

  export default {
    data() {
      return {
        address: {},
        cart: [],
        allChecked: false,
        totalPrice: 0,
        totalNum: 0
      }
    },
    onShow() {
      // 1 获取缓存中的收货地址信息
      const address = mpvue.getStorageSync("address");
      // 1 获取缓存中的购物车数据
      const cart = mpvue.getStorageSync("cart") || [];

      this.address =address;
      this.setCart(cart);
    },
    methods:{
      async handleChooseAddress() {
        try {
          // 1 获取 权限状态
          const res1 = await getSetting();
          const scopeAddress = res1.authSetting["scope.address"];
          // 2 判断 权限状态
          if (scopeAddress === false) {
            await openSetting();
          }
          // 4 调用获取收货地址的 api
          let address = await chooseAddress();
          address.all = address.provinceName + address.cityName + address.countyName + address.detailInfo;
          // 5 存入到缓存中
          mpvue.setStorageSync("address", address);
        } catch (error) {
          console.log(error);
        }
      },
      // 商品的选中
      handeItemChange(e) {
        // 1 获取被修改的商品的id
        const goods_id = e.mp.currentTarget.dataset.id;
        // 2 获取购物车数组
        let cart = this.cart;
        // 3 找到被修改的商品对象
        let index = cart.findIndex(v => v.goods_id === goods_id);
        // 4 选中状态取反
        cart[index].checked = !cart[index].checked;

        this.setCart(cart);

      },
      // 设置购物车状态同时 重新计算 底部工具栏的数据 全选 总价格 购买的数量
      setCart(cart) {
        let allChecked = true;
        // 1 总价格 总数量
        let totalPrice = 0;
        let totalNum = 0;
        cart.forEach(v => {
          if (v.checked) {
            totalPrice += v.num * v.goods_price;
            totalNum += v.num;
          } else {
            allChecked = false;
          }
        })
        // 判断数组是否为空
        allChecked = cart.length !== 0 ? allChecked : false;
        this.cart = cart
        this.totalPrice = totalPrice
        this.totalNum = totalNum
        this.allChecked = allChecked
        mpvue.setStorageSync("cart", cart);
      },
      // 商品全选功能
      handleItemAllCheck() {
        // 1 获取data中的数据
        let cart =this.cart
        let allChecked = this.allChecked
        // 2 修改值
        allChecked = !allChecked;
        // 3 循环修改cart数组 中的商品选中状态
        cart.forEach(v => v.checked = allChecked);
        // 4 把修改后的值 填充回data或者缓存中
        this.setCart(cart);
      },
      // 商品数量的编辑功能
      async handleItemNumEdit(e) {
        // 1 获取传递过来的参数
        const { operation, id } = e.mp.currentTarget.dataset;
        // 2 获取购物车数组
        let cart = this.cart;
        // 3 找到需要修改的商品的索引
        const index = cart.findIndex(v => v.goods_id === id);
        // 4 判断是否要执行删除
        if (cart[index].num === 1 && operation === -1) {
          // 4.1 弹窗提示
          const res = await showModal({ content: "您是否要删除？" });
          if (res.confirm) {
            cart.splice(index, 1);
            this.setCart(cart);
          }
        } else {
          // 4  进行修改数量
          cart[index].num += operation;
          // 5 设置回缓存和data中
          this.setCart(cart);
        }
      },
      // 点击 结算
      async handlePay(){
        // 1 判断收货地址
        let address = this.address;
        let totalNum = this.totalNum;
        if(!address.userName){
          await showToast({title:"您还没有选择收货地址"});
          return;
        }
        // 2 判断用户有没有选购商品
        if(totalNum===0){
          await showToast({title:"您还没有选购商品"});
          return ;
        }
        // 3 跳转到 支付页面
        mpvue.navigateTo({
          url: '/pages/pay/main'
        });

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


  .revice_address_row .address_btn {
    padding: 10px;
  }

  .revice_address_row .address_btn button {
    width: 60%;
  }

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
    color: #eb4450;
    border-top: 0.5px solid currentColor;
    border-bottom: 0.5px solid currentColor;
  }

  .cart_content .cart_main .cart_item {
    display: flex;
    padding: 5px;
    border-bottom: 0.5px solid #ccc;
  }

  .cart_content .cart_main .cart_item .cart_chk_wrap {
    flex: 1;
    display: flex;
    justify-content: center;
    align-items: center;
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

  .cart_content .cart_main .cart_item .cart_info_wrap .goods_price_wrap .cart_num_tool .num_edit {
    width: 22.5px;
    height: 22.5px;
    display: flex;
    justify-content: center;
    align-items: center;
    border: 0.5px solid #ccc;
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

  .footer_tool .all_chk_wrap {
    flex: 2;
    display: flex;
    justify-content: center;
    align-items: center;
  }

  .footer_tool .total_price_wrap {
    flex: 5;
    padding-right: 7.5px;
    text-align: right;
  }

  .footer_tool .total_price_wrap .total_price .total_price_text {
    color: #eb4450;
    font-size: 17px;
    font-weight: 600;
  }

  .footer_tool .order_pay_wrap {
    flex: 3;
    background-color: #eb4450;
    color: #fff;
    font-size: 16px;
    font-weight: 600;
    display: flex;
    justify-content: center;
    align-items: center;
  }

</style>
