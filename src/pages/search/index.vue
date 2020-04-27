<template>
  <div>
    <div class="search_row">
      <input :value="inpValue" placeholder="请输入您要搜索的商品" @input="handleInput"> </input>
      <button class="search_button" @tap="handleCancel" :hidden="!isFocus">取消</button>
    </div>
    <div class="search_content">
      <a :url="'/pages/goods_detail/main?goods_id='+item.goods_id" class="search_item"
         v-for="item in goods" :key="item.goods_id">
        {{item.goods_name}}
      </a>
    </div>
  </div>
</template>

<script>
  import {request} from "../../request";

  export default {
    data() {
      return {
        goods: [],
        // 取消 按钮 是否显示
        isFocus: false,
        // 输入框的值
        inpValue: "",
        TimeId: -1
      }
    },
    methods: {
      // 输入框的值改变 就会触发的事件
      handleInput(e) {
        // 1 获取输入框的值
        const {value} = e.mp.detail;
        // 2 检测合法性
        if (!value.trim()) {
          this.goods = [];
          this.isFocus = false
          // 值不合法
          return;
        }
        // 3 准备发送请求获取数据
        this.isFocus = true
        clearTimeout(this.TimeId);
        this.TimeId = setTimeout(() => {
          this.qsearch(value);
        }, 1000);
      },
      // 发送请求获取搜索建议 数据
      async qsearch(query) {
        const res = await request({url: "/goods/qsearch", data: {query}});
        // console.log(res);
        this.goods = res
      },
      // 点击 取消按钮
      handleCancel() {
        this.inpValue = "";
        this.isFocus = false;
        this.goods = []
      }
    }
  }
</script>

<style lang="wxss">
  page {
    background-color: #dedede;
    padding: 10px;
  }
</style>
<style scoped>

  .search_row {
    height: 30px;
    display: flex;
  }

  .search_row input {
    background-color: #fff;
    flex: 1;
    height: 100%;
    padding-left: 15px;
  }

  .search_row .search_button {
    width: 55px;
    height: 100%;
    padding: 0;
    margin: 0 5px;
    display: flex;
    justify-content: center;
    align-items: center;
    font-size: 13px;
  }

  .search_content {
    margin-top: 15px;
  }

  .search_content .search_item {
    background-color: #fff;
    font-size: 13px;
    padding: 7.5px 5px;
    border-bottom: 0.5px solid #ccc;
    overflow: hidden;
    white-space: nowrap;
    text-overflow: ellipsis;
  }

</style>
