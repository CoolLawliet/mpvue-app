<template>
  <div>
    <div class="detail_swiper">
      <swiper
        autoplay
        circular
        indicator-dots
      >
        <swiper-item
          v-for="item in goodsObj.pics"
          :key="item.pics_id"
          @tap="handlePrevewImage"
          :data-url="item.pics_mid"
        >
          <img mode="widthFix" :src="item.pics_mid"/>
        </swiper-item>
      </swiper>
    </div>

    <div class="goods_price">￥{{goodsObj.goods_price}}</div>
    <div class="goods_name_row">
      <div class="goods_name">{{goodsObj.goods_name}}{{goodsObj.goods_name}}</div>
      <div class="goods_collect" @tap="handleCollect">
        <text :class="isCollect?'icon-shoucang1 iconfont':'icon-shoucang iconfont' "></text>
        <div class="collect_text">收藏</div>
      </div>
    </div>

    <div class="goods_info">
      <div class="goods_info_title">图文详情</div>
      <div class="goods_info_content">
        <!-- 富文本 -->
<!--         {{// goodsObj.goods_introduce}}-->
        <wxParse :content="goodsObj.goods_introduce"></wxParse>
      </div>
    </div>

    <div class="btm_tool">
      <div class="tool_item">
        <div class="iconfont icon-kefu"></div>
        <div>客服</div>
        <button open-type="contact"></button>
      </div>
      <div class="tool_item">
        <div class="iconfont icon-yixianshi-"></div>
        <div>分享</div>
        <button open-type="share"></button>
      </div>
      <navigator open-type="switchTab" url="/pages/cart/main" class="tool_item">
        <div class="iconfont icon-gouwuche"></div>
        <div>购物车</div>
      </navigator>
      <div class="tool_item btn_cart " @tap="handleCartAdd">
        加入购物车
      </div>
      <div class="tool_item btn_buy">
        立即购买
      </div>
    </div>
  </div>
</template>

<script>
  import wxParse from 'mpvue-wxparse'
  import {request} from "../../request";

  export default {
    data(){
      return{
        goodsObj: {},
        // 商品是否被收藏
        isCollect:false,
        GoodsInfo: {},
      }
    },
    onShow(){
      let pages = getCurrentPages();
      let currentPage = pages[pages.length - 1];
      let options = currentPage.options;
      const { goods_id } = options;
      this.getGoodsDetail(goods_id);
    },
    methods:{
      // 获取商品详情数据
      async getGoodsDetail(goods_id) {
        const goodsObj = await request({ url: "/goods/detail", data: { goods_id } });
        this.GoodsInfo = goodsObj;
        // 1 获取缓存中的商品收藏的数组
        let collect = mpvue.getStorageSync("collect") || [];
        // 2 判断当前商品是否被收藏
        let isCollect = collect.some(v => v.goods_id === this.GoodsInfo.goods_id);
        this.goodsObj.goods_name = goodsObj.goods_name;
        this.goodsObj.goods_price = goodsObj.goods_price;
        this.goodsObj.goods_introduce = goodsObj.goods_introduce.replace(/\.webp/g, '.jpg');
        this.goodsObj.pics = goodsObj.pics;
        this.isCollect = isCollect;
        console.log(this.isCollect);
      },
      // 点击轮播图 放大预览
      handlePrevewImage(e) {
        // 1 先构造要预览的图片数组
        const urls = this.GoodsInfo.pics.map(v => v.pics_mid);
        // 2 接收传递过来的图片url
        const current = e.mp.currentTarget.dataset.url;
        mpvue.previewImage({
          current,
          urls
        });
      },
      // 点击 加入购物车
      handleCartAdd() {
        // 1 获取缓存中的购物车 数组
        let cart = mpvue.getStorageSync("cart") || [];
        // 2 判断 商品对象是否存在于购物车数组中
        let index = cart.findIndex(v => v.goods_id === this.GoodsInfo.goods_id);
        if (index === -1) {
          //3  不存在 第一次添加
          this.GoodsInfo.num = 1;
          this.GoodsInfo.checked = true;
          cart.push(this.GoodsInfo);
        } else {
          // 4 已经存在购物车数据 执行 num++
          cart[index].num++;
        }
        // 5 把购物车重新添加回缓存中
        mpvue.setStorageSync("cart", cart);
        // 6 弹窗提示
        mpvue.showToast({
          title: '加入成功',
          icon: 'success',
          // true 防止用户 手抖 疯狂点击按钮
          mask: true
        });
      },
      // 点击 商品收藏图标
      handleCollect(){
        let isCollect=false;

        // 1 获取缓存中的商品收藏数组
        let collect=mpvue.getStorageSync("collect")||[];
        // 2 判断该商品是否被收藏过
        let index=collect.findIndex(v=>v.goods_id===this.GoodsInfo.goods_id);
        // 3 当index！=-1表示 已经收藏过
        if(index!==-1){
          // 能找到 已经收藏过了  在数组中删除该商品
          collect.splice(index,1);
          isCollect=false;
          mpvue.showToast({
            title: '取消成功',
            icon: 'success',
            mask: true
          });

        }else{
          // 没有收藏过
          collect.push(this.GoodsInfo);
          isCollect=true;
          mpvue.showToast({
            title: '收藏成功',
            icon: 'success',
            mask: true
          });
        }
        // 4 把数组存入到缓存中
        mpvue.setStorageSync("collect", collect);
        // 5 修改data中的属性  isCollect
        this.isCollect = isCollect

      }
    },
    components:{
      wxParse
    }
  }
</script>

<style lang="wxss">
  page {
    padding-bottom: 45px;
  }
</style>
<style scoped>
  .detail_swiper swiper {
    height: 65vw;
    text-align: center;
  }

  .detail_swiper swiper image {
    width: 60%;
  }

  .goods_price {
    padding: 7.5px;
    font-size: 16px;
    font-weight: 600;
    color: var(--themeColor);
  }

  .goods_name_row {
    border-top: 2.5px solid #dedede;
    border-bottom: 2.5px solid #dedede;
    padding: 5px 0;
    display: flex;
  }

  .goods_name_row .goods_name {
    flex: 5;
    color: #000;
    font-size: 15px;
    padding: 0 5px;
    display: -webkit-box;
    overflow: hidden;
    -webkit-box-orient: vertical;
    -webkit-line-clamp: 2;
  }

  .goods_name_row .goods_collect {
    flex: 1;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    border-left: 0.5px solid #000;
  }

  .goods_name_row .goods_collect .icon-shoucang1 {
    color: orangered;
  }

  .goods_info .goods_info_title {
    font-size: 16px;
    color: #eb4450;
    font-weight: 600;
    padding: 10px;
  }

  .btm_tool {
    border-top: 0.5px solid #ccc;
    position: fixed;
    left: 0;
    bottom: 0;
    width: 100%;
    height: 45px;
    background-color: #fff;
    display: flex;
  }

  .btm_tool .tool_item {
    flex: 1;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    font-size: 12px;
    position: relative;
  }

  .btm_tool .tool_item button {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    opacity: 0;
  }

  .btm_tool .btn_cart {
    flex: 2;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    background-color: #ffa500;
    color: #fff;
    font-size: 15px;
    font-weight: 600;
  }

  .btm_tool .btn_buy {
    flex: 2;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    background-color: #eb4450;
    color: #fff;
    font-size: 15px;
    font-weight: 600;
  }

</style>
