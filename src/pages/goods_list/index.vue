<template>
  <div>
  <SearchInput></SearchInput>
  <!-- 监听自定义事件 -->
  <Tabs :tabs="tabs" @tabsItemChange="handleTabsItemChange" >

    <block v-if="tabs[0].isActive">
      <div class="first_tab">
        <navigator class="goods_item"
                   v-for="item in goodsList"
                   :key="item.goods_id"
                   :url="'/pages/goods_detail/main?goods_id='+item.goods_id"
        >
          <!-- 左侧 图片容器 -->
          <div class="goods_img_wrap">
            <img mode="widthFix" :src="(item.goods_small_logo?item.goods_small_logo:'https://ww1.sinaimg.cn/large/007rAy9hgy1g24by9t530j30i20i2glm.jpg')"/>
          </div>
          <!-- 右侧 商品容器 -->
          <div class="goods_info_wrap">
            <div class="goods_name">{{item.goods_name}}</div>
            <div class="goods_price">￥{{item.goods_price}}</div>
          </div>
        </navigator>
      </div>
    </block>
    <block v-else-if="tabs[1].isActive">暂无数据</block>
    <block v-else-if="tabs[2].isActive">暂无数据</block>

  </Tabs>

  </div>
</template>

<script>
  import SearchInput from "../../components/SearchInput";
  import Tabs from "../../components/Tabs";
  import {request} from "../../request";

  export default {
    data(){
      return{
        tabs: [
          {
            id: 0,
            value: "综合",
            isActive: true
          },
          {
            id: 1,
            value: "销量",
            isActive: false
          },
          {
            id: 2,
            value: "价格",
            isActive: false
          }
        ],
        goodsList:[],
        // 接口要的参数
        QueryParams:{
          query:"",
          cid:"",
          pagenum:1,
          pagesize:10
        },
        // 总页数
        totalPages:1,
      }
    },
    onLoad(){
      console.log(this.$root);
      this.QueryParams.cid=this.$root.$mp.query.cid||"";
      this.QueryParams.query=this.$root.$mp.query.query||"";
      this.getGoodsList();
    },
    methods:{
      // 获取商品列表数据
      async getGoodsList(){
        const res=await request({url:"/goods/search",data:this.QueryParams});
        // 获取 总条数
        const total=res.total;
        // 计算总页数
        this.totalPages=Math.ceil(total/this.QueryParams.pagesize);
        // console.log(this.totalPages);
        this.goodsList =[...this.goodsList,...res.goods]
        // 关闭下拉刷新的窗口 如果没有调用下拉刷新的窗口 直接关闭也不会报错
        mpvue.stopPullDownRefresh();
      },
      // 标题点击事件 从子组件传递过来
      handleTabsItemChange(e){
        // 1 获取被点击的标题索引
        const {index}=e;
        let tabs = this.tabs
        tabs.forEach((v,i)=>i===index?v.isActive=true:v.isActive=false);
        // 3 赋值到data中
        this.tabs = tabs
      },
      // 页面上滑 滚动条触底事件
      onReachBottom(){
        //  1 判断还有没有下一页数据
        if(this.QueryParams.pagenum>=this.totalPages){
          // 没有下一页数据
          //  console.log('%c'+"没有下一页数据","color:red;font-size:100px;background-image:linear-gradient(to right,#0094ff,pink)");
          mpvue.showToast({ title: '没有下一页数据' });
        }else{
          // 还有下一页数据
          //  console.log('%c'+"有下一页数据","color:red;font-size:100px;background-image:linear-gradient(to right,#0094ff,pink)");
          this.QueryParams.pagenum++;
          this.getGoodsList();
        }
      },
      // 下拉刷新事件
      onPullDownRefresh(){
        // 1 重置数组
        this.goodsList=[]
        // 2 重置页码
        this.QueryParams.pagenum=1;
        // 3 发送请求
        this.getGoodsList();
      }
    },
    components:{
      SearchInput,
      Tabs
    }
  }
</script>

<style scoped>
  .first_tab .goods_item {
    display: flex;
    border-bottom: 1px solid #ccc;
  }
  .first_tab .goods_item .goods_img_wrap {
    flex: 2;
    display: flex;
    justify-content: center;
    align-items: center;
  }
  .first_tab .goods_item .goods_img_wrap image {
    width: 70%;
  }
  .first_tab .goods_item .goods_info_wrap {
    flex: 3;
    display: flex;
    flex-direction: column;
    justify-content: space-around;
  }
  .first_tab .goods_item .goods_info_wrap .goods_name {
    display: -webkit-box;
    overflow: hidden;
    -webkit-box-orient: vertical;
    -webkit-line-clamp: 2;
  }
  .first_tab .goods_item .goods_info_wrap .goods_price {
    color: #eb4450;
    font-size: 16px;
  }
</style>
