<template>
  <div class="pyg_index">
    <SearchInput></SearchInput>
    <div class="index_swiper">
      <swiper autoplay indicator-dots circular>
        <swiper-item
          v-for="item in swiperList"
          :key="item.goods_id"
        >
          <navigator :url="item.navigator_url">
            <img mode="widthFix" :src="item.image_src"></img>
          </navigator>
        </swiper-item>
      </swiper>
    </div>
    <!-- 轮播图 结束 -->

    <!-- 导航 开始 -->
    <div class="index_cate">
      <navigator
        v-for="item in catesList"
        :key="item.name"
        url="/pages/category/main"
        open-type="switchTab"
      >
        <img mode="widthFix" :src="item.image_src"></img>
      </navigator>
    </div>
    <!-- 导航 结束 -->

    <!-- 楼层 开始 -->
    <div class="index_floor">
      <div class="floor_group"
           v-for="(item1,index1) in floorList"
           :key="item1.floor_title"
      >
        <!-- 标题 -->
        <div class="floor_title">
          <img mode="widthFix" :src="item1.floor_title.image_src"></img>
        </div>
        <!-- 内容 -->
        <div class="floor_list">
          <navigator
            v-for="(item2,index2) in item1.product_list"
            :key="item2.name"
            :url="item2.navigator_url"
          >
            <img :mode="index2===0?'widthFix':'scaleToFill'" :src="item2.image_src"></img>
          </navigator>
        </div>
      </div>
    </div>
    <!-- 楼层 结束 -->
  </div>
</template>

<script>
  import {request} from "../../request/index";
  import SearchInput from "../../components/SearchInput";
  export default {
// 0 引入 用来发送请求的 方法 一定要把路径补全
    data() {
      return {
        // 轮播图数组
        swiperList: [],
        // 导航 数组
        catesList: [],
        // 楼层数据
        floorList: []
      }
    },
    created() {
      this.getSwiperList();
      this.getCateList();
      this.getFloorList();
    },
    methods: {
      // 获取轮播图数据
      getSwiperList() {
        request({url: "/home/swiperdata"})
          .then(result => {
            this.swiperList = result
          })
      },
      // 获取 分类导航数据
      getCateList() {
        request({url: "/home/catitems"})
          .then(result => {
            this.catesList = result
          })
      },
      // 获取 楼层数据
      getFloorList() {
        request({url: "/home/floordata"})
          .then(result => {
            this.floorList = result
          })
      },
    },
    components:{
      SearchInput
    }

  }
</script >

<style scoped>
  .index_swiper swiper {
    width: 375px;
    height: 170px;
  }
  .index_swiper swiper image {
    width: 100%;
  }
  .index_cate {
    display: flex;
  }
  .index_cate navigator {
    padding: 10px;
    flex: 1;
  }
  .index_cate navigator image {
    width: 100%;
  }
  .index_floor .floor_group .floor_title {
    padding: 5px 0;
  }
  .index_floor .floor_group .floor_title image {
    width: 100%;
  }
  .index_floor .floor_group .floor_list {
    overflow: hidden;
  }
  .index_floor .floor_group .floor_list navigator {
    float: left;
    width: 33.33%;
    /* 后四个超链接 */
    /* 2 3 两个超链接 */
  }
  .index_floor .floor_group .floor_list navigator:nth-last-child(-n+4) {
    /* 原图的宽高 232 *386 */
    height: 27.72711207vw;
    border-left: 5px solid #fff;
  }
  .index_floor .floor_group .floor_list navigator:nth-child(2),
  .index_floor .floor_group .floor_list navigator:nth-child(3) {
    border-bottom: 5px solid #fff;
  }
  .index_floor .floor_group .floor_list navigator image {
    width: 100%;
    height: 100%;
  }

</style>
