<template>
  <div>
    <Tabs :tabs="tabs" bindtabsItemChange="handleTabsItemChange">

      <view class="collect_main">
        <view class="collect_title">
          <text class="collect_tips active">全部</text>
          <text class="collect_tips">正在热卖</text>
          <text class="collect_tips">即将上线</text>
        </view>
        <view class="collect_content">
          <navigator class="goods_item" v-for="item in collect" :key="item.goods_id"
                     :url="'/pages/goods_detail/main?goods_id='+item.goods_id">
            <!-- 左侧 图片容器 -->
            <view class="goods_img_wrap">
              <image mode="widthFix"
                     :src="(item.goods_small_logo?item.goods_small_logo:'https://ww1.sinaimg.cn/large/007rAy9hgy1g24by9t530j30i20i2glm.jpg')">
              </image>
            </view>
            <!-- 右侧 商品容器 -->
            <view class="goods_info_wrap">
              <view class="goods_name">{{item.goods_name}}</view>
              <view class="goods_price">￥{{item.goods_price}}</view>
            </view>
          </navigator>
        </view>
      </view>

    </Tabs>
  </div>
</template>

<script>
  import Tabs from "../../components/Tabs";

  export default {
    data() {
      return {
        collect: [],
        tabs: [
          {
            id: 0,
            value: "商品收藏",
            isActive: true
          },
          {
            id: 1,
            value: "品牌收藏",
            isActive: false
          },
          {
            id: 2,
            value: "店铺收藏",
            isActive: false
          },
          {
            id: 3,
            value: "浏览器足迹",
            isActive: false
          }
        ]
      }
    },
    onShow() {
      this.collect = mpvue.getStorageSync("collect") || []
    },
    methods: {
      handleTabsItemChange(e) {
        // 1 获取被点击的标题索引
        const {index} = e.mp.detail;
        // 2 修改源数组
        let tabs = this.tabs;
        tabs.forEach((v, i) => i === index ? v.isActive = true : v.isActive = false);
        // 3 赋值到data中
        this.tabs = tabs
      }
    },
    components: {
      Tabs
    }
  }
</script>

<style scoped>
  .collect_main {
    background-color: #f3f4f6;
  }

  .collect_main .collect_title {
    padding: 20px 0;
  }

  .collect_main .collect_title .collect_tips {
    padding: 7.5px;
    border: 0.5px solid #ccc;
    margin-left: 12.5px;
    background-color: #fff;
  }

  .collect_main .collect_title .active {
    color: var(--themeColor);
    border-color: currentColor;
  }

  .collect_main .collect_content .goods_item {
    display: flex;
    border-bottom: 1px solid #ccc;
    background-color: #fff;
  }

  .collect_main .collect_content .goods_item .goods_img_wrap {
    flex: 2;
    display: flex;
    justify-content: center;
    align-items: center;
  }

  .collect_main .collect_content .goods_item .goods_img_wrap image {
    width: 70%;
  }

  .collect_main .collect_content .goods_item .goods_info_wrap {
    flex: 3;
    display: flex;
    flex-direction: column;
    justify-content: space-around;
  }

  .collect_main .collect_content .goods_item .goods_info_wrap .goods_name {
    display: -webkit-box;
    overflow: hidden;
    -webkit-box-orient: vertical;
    -webkit-line-clamp: 2;
  }

  .collect_main .collect_content .goods_item .goods_info_wrap .goods_price {
    color: var(--themeColor);
    font-size: 16px;
  }

</style>
