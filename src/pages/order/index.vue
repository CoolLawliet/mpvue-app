<template>
  <div>
    <Tabs :tabs="tabs" @tabsItemChange="handleTabsItemChange" >

      <div class="order_main">
        <div
          v-for="item in orders"
          :key="item.order_id"
          class="order_item">

          <div class="order_no_row">
            <div class="order_no_text">订单编号</div>
            <div class="order_no_value">{{item.order_number}}</div>
          </div>
          <div class="order_price_row">
            <div class="order_price_text">订单价格</div>
            <div class="order_price_value">￥{{item.order_price}}</div>
          </div>
          <div class="order_time_row">
            <div class="order_time_text">订单日期</div>
            <div class="order_time_value">{{item.create_time_cn}}</div>
          </div>

        </div>
      </div>
    </Tabs>
  </div>
</template>

<script>
  import Tabs from "../../components/Tabs";
  import {request} from "../../request";

  export default {
    data(){
      return{
        orders: [],
        tabs: [
          {
            id: 0,
            value: "全部",
            isActive: true
          },
          {
            id: 1,
            value: "待付款",
            isActive: false
          },
          {
            id: 2,
            value: "待发货",
            isActive: false
          },
          {
            id: 3,
            value: "退款/退货",
            isActive: false
          }
        ]
      }
    },
    onShow(options) {
      const token = mpvue.getStorageSync("token");
      if (!token) {
        mpvue.navigateTo({
          url: '/pages/auth/main'
        });
        return;
      }
      // 1 获取当前的小程序的页面栈-数组 长度最大是10页面
      let pages = getCurrentPages();
      // 2 数组中 索引最大的页面就是当前页面
      let currentPage = pages[pages.length - 1];
      // 3 获取url上的type参数
      const { type } = currentPage.options;
      // 4 激活选中页面标题 当 type=1 index=0
      this.changeTitleByIndex(type-1);
      this.getOrders(type);
    },
    methods:{
      // 获取订单列表的方法
      async getOrders(type) {
        const res = await request({ url: "/my/orders/all", data: { type } });
        this.orders=res.orders.map(v=>({...v,create_time_cn:(new Date(v.create_time*1000).toLocaleString())}))
      },
      // 根据标题索引来激活选中 标题数组
      changeTitleByIndex(index) {
        // 2 修改源数组
        let tabs = this.tabs;
        tabs.forEach((v, i) => i === index ? v.isActive = true : v.isActive = false);
        // 3 赋值到data中
        this.tabs = tabs
      },
      handleTabsItemChange(e) {
        // 1 获取被点击的标题索引
        const { index } = e;
        this.changeTitleByIndex(index);
        // 2 重新发送请求 type=1 index=0
        this.getOrders(index+1);
      }
    },
    components:{
      Tabs
    }
  }
</script>

<style scoped>
  .order_main .order_item {
    padding: 10px;
    border-bottom: 0.5px solid #ccc;
    color: #666;
  }
  .order_main .order_item .order_no_row {
    display: flex;
    padding: 5px 0;
    justify-content: space-between;
  }
  .order_main .order_item .order_price_row {
    display: flex;
    padding: 5px 0;
    justify-content: space-between;
  }
  .order_main .order_item .order_price_row .order_price_value {
    color: var(--themeColor);
    font-size: 16px;
  }
  .order_main .order_item .order_time_row {
    display: flex;
    padding: 5px 0;
    justify-content: space-between;
  }

</style>
