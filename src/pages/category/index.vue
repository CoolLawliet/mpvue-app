<template>
  <div class="cates">
    <SearchInput></SearchInput>
    <div class="cates_container">
      <!-- 左侧菜单 -->
      <scroll-view scroll-y class="left_menu`">
        <div
          :class="'menu_item '+ (index===currentIndex?'active':'')"
          v-for="(item,index) in leftMenuList"
          :key="index"
          @tap="handleItemTap"
          :data-index="index"
        >{{item}}</div>
      </scroll-view>
      <!-- 右侧商品 -->
      <scroll-view :scroll-top="scrollTop" scroll-y class="right_content">
        <div class="goods_group"
              v-for="(item1,index1) in rightContent"
        >
          <div class="goods_title">
            <span class="delimiter">/</span>
            <span class="title">{{item1.cat_name}}</span>
            <span class="delimiter">/</span>
          </div>
          <div class="goods_list">
            <navigator v-for="(item2,index2) in item1.children"
                       :key="item2.cat_id"
                       :url="'/pages/goods_list/main?cid='+item2.cat_id"
            >
              <img mode="widthFix" :src="item2.cat_icon" />
              <div class="goods_name">{{item2.cat_name}}</div>
            </navigator>
          </div>
        </div>
      </scroll-view>
    </div>
  </div>
</template>

<script>
  import SearchInput from "../../components/SearchInput";
  import {request} from "../../request";

  export default {
    data(){
      return{
        // 左侧的菜单数据
        leftMenuList: [],
        // 右侧的商品数据
        rightContent: [],
        // 被点击的左侧的菜单
        currentIndex: 0,
        // 右侧内容的滚动条距离顶部的距离
        scrollTop: 0,
        // 接口的返回数据
        Cates:[],
      }
    },
    onLoad(){
      //  1 获取本地存储中的数据  (小程序中也是存在本地存储 技术)
      const Cates = mpvue.getStorageSync("cates");
      // 2 判断
      if (!Cates) {
        // 不存在  发送请求获取数据
        this.getCates();
      } else {
        // 有旧的数据 定义过期时间  10s 改成 5分钟
        if (Date.now() - Cates.time > 1000 * 10) {
          // 重新发送请求
          this.getCates();
        } else {
          // 可以使用旧的数据
          this.Cates = Cates.data;
          let leftMenuList = this.Cates.map(v => v.cat_name);
          let rightContent = this.Cates[0].children;
          this.setData({
            leftMenuList,
            rightContent
          })
        }
      }
    },
    methods:{
      async getCates() {
        // 1 使用es7的async await来发送请求
        const res = await request({ url: "/categories" });
        // this.Cates = res.data.message;
        this.Cates = res;
        // 把接口的数据存入到本地存储中
        mpvue.setStorageSync("cates", { time: Date.now(), data: this.Cates });
        // 构造左侧的大菜单数据
        let leftMenuList = this.Cates.map(v => v.cat_name);
        // 构造右侧的商品数据
        let rightContent = this.Cates[0].children;
        this.leftMenuList = leftMenuList;
        this.rightContent = rightContent
      },
      // 左侧菜单的点击事件
      handleItemTap(e) {
        /*
        1 获取被点击的标题身上的索引
        2 给data中的currentIndex赋值就可以了
        3 根据不同的索引来渲染右侧的商品内容
         */
        const { index } = e.mp.currentTarget.dataset;

        let rightContent = this.Cates[index].children;
        this.currentIndex = index
        this.rightContent = rightContent
        this.scrollTop = 0
      }
    },
      components:{
        SearchInput
      }
    }
</script>

<style lang="wxss">
  page {
    height: 100%;
  }
</style>
<style scoped>
  .cates {
    height: 100%;
  }
  .cates .cates_container {
    height: calc(100vh - 45px);
    display: flex;
  }
  .cates .cates_container .left_menu {
    flex: 2;
  }
  .cates .cates_container .left_menu .menu_item {
    height: 40px;
    display: flex;
    justify-content: center;
    align-items: center;
    font-size: 15px;
  }
  .cates .cates_container .left_menu .active {
    color: #eb4450;
    border-left: 5px solid currentColor;
  }
  .cates .cates_container .right_content {
    flex: 5;
  }
  .cates .cates_container .right_content .goods_group .goods_title {
    height: 40px;
    display: flex;
    justify-content: center;
    align-items: center;
  }
  .cates .cates_container .right_content .goods_group .goods_title .delimiter {
    color: #ccc;
    padding: 0 5px;
  }
  .cates .cates_container .right_content .goods_group .goods_list {
    display: flex;
    flex-wrap: wrap;
  }
  .cates .cates_container .right_content .goods_group .goods_list navigator {
    width: 33.33%;
    text-align: center;
  }
  .cates .cates_container .right_content .goods_group .goods_list navigator image {
    width: 50%;
  }

</style>
