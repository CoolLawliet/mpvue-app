<template>
  <div>
    <Tabs :tabs="tabs" @tabsItemChange="handleTabsItemChange" >
      <div class="fb_main">
        <div class="fb_title">问题的种类</div>
        <div class="fb_tips">
          <p>功能建议</p>
          <p>购买遇到问题</p>
          <p>性能问题</p>
          <p>其他</p>
        </div>
        <div class="fb_content">
          <textarea :value="textVal" @input="handleTextInput" placeholder="请描述一下您的问题"> </textarea>
          <div class="fb_tool">
            <button @tap="handleChooseImg">+</button>
            <div class="up_img_item"
                  v-for="item in chooseImgs"
                  :key="this"
                  @tap="handleRemoveImg"
                  :data-index="index"
            >
              <UpImg :src="item" ></UpImg>
            </div>

          </div>
        </div>
        <div class="form_btn_wrap">
          <button @tap="handleFormSubmit">
            <icon  type="success_no_circle" size="23" color="white">
            </icon>
            提交
          </button>

        </div>
      </div>
    </Tabs>
  </div>
</template>

<script>
  import Tabs from "../../components/Tabs";
  import UpImg from "../../components/UpImg";
  export default {
    data(){
      return{
        tabs: [
          {
            id: 0,
            value: "体验问题",
            isActive: true
          },
          {
            id: 1,
            value: "商品、商家投诉",
            isActive: false
          }
        ],
        // 被选中的图片路径 数组
        chooseImgs: [],
        // 文本域的内容
        textVal: "",
        // 外网的图片的路径数组
        UpLoadImgs: [],
      }
    },
    methods:{
      handleTabsItemChange(e) {
        // 1 获取被点击的标题索引
        const { index } = e;
        // 2 修改源数组
        let tabs = this.tabs;
        tabs.forEach((v, i) => i === index ? v.isActive = true : v.isActive = false);
        // 3 赋值到data中
        this.tabs = tabs
      },
      // 点击 “+” 选择图片
      handleChooseImg() {
        // 2 调用小程序内置的选择图片api
        mpvue.chooseImage({
          // 同时选中的图片的数量
          count: 9,
          // 图片的格式  原图  压缩
          sizeType: ['original', 'compressed'],
          // 图片的来源  相册  照相机
          sourceType: ['album', 'camera'],
          success: (result) => {

            this.chooseImgs=[...this.chooseImgs, ...result.tempFilePaths]
          }
        });

      },
      // 点击 自定义图片组件
      handleRemoveImg(e) {
        // 2 获取被点击的组件的索引
        const { index } = e.mp.currentTarget.dataset;
        // 3 获取data中的图片数组
        let chooseImgs = this.chooseImgs;
        // 4 删除元素
        chooseImgs.splice(index, 1);
        this.chooseImgs = chooseImgs
      },
      // 文本域的输入的事件
      handleTextInput(e) {
        this.textVal=e.mp.detail.value
      },
      // 提交按钮的点击
      handleFormSubmit() {
        // 1 获取文本域的内容 图片数组
        const textVal = this.textVal;
        const chooseImgs = this.chooseImgs;
        // 2 合法性的验证
        if (!textVal.trim()) {
          // 不合法
          mpvue.showToast({
            title: '输入不合法',
            icon: 'none',
            mask: true
          });
          return;
        }
        // 3 准备上传图片 到专门的图片服务器
        // 上传文件的 api 不支持 多个文件同时上传  遍历数组 挨个上传
        // 显示正在等待的图片
        mpvue.showLoading({
          title: "正在上传中",
          mask: true
        });

        // 判断有没有需要上传的图片数组

        if (chooseImgs.length !== 0) {
          chooseImgs.forEach((v, i) => {
            mpvue.uploadFile({
              // 图片要上传到哪里
              url: 'https://images.ac.cn/api/upload/upload',
              // 被上传的文件的路径
              filePath: v,
              // 上传的文件的名称 后台来获取文件  file
              name: "file",
              // 顺带的文本信息
              formData: {},
              success: (result) => {
                console.log(result);
                let url = JSON.parse(result.data).url;
                this.UpLoadImgs.push(url);

                // 所有的图片都上传完毕了才触发
                if (i === chooseImgs.length - 1) {
                  mpvue.hideLoading();
                  console.log("把文本的内容和外网的图片数组 提交到后台中");
                  //  提交都成功了
                  // 重置页面
                  this.textVal = '';
                  this.chooseImgs =[]
                  // 返回上一个页面
                  mpvue.navigateBack({
                    delta: 1
                  });

                }
              }
            });
          })
        }else{
          mpvue.hideLoading();

          console.log("只是提交了文本");
          mpvue.navigateBack({
            delta: 1
          });

        }
      }
    },
    components:{
      Tabs,
      UpImg
    }
  }
</script>

<style lang="wxss">
  page {
    background-color: #eeeeee;
  }
</style>
<style scoped>
  .fb_main {
    padding: 10px;
    color: #666;
  }
  .fb_main .fb_tips {
    display: flex;
    flex-wrap: wrap;
  }
  .fb_main .fb_tips p {
    width: 27%;
    padding: 5px;
    text-align: center;
    background-color: #fff;
    margin: 10px 5px;
  }
  .fb_main .fb_content {
    background-color: #fff;
    margin-top: 10px;
  }
  .fb_main .fb_content textarea {
    padding: 5px;
  }
  .fb_main .fb_tool {
    display: flex;
    flex-wrap: wrap;
    padding-bottom: 15px;
  }
  .fb_main .fb_tool button {
    margin: 0;
    width: 45px;
    height: 45px;
    font-size: 30px;
    padding: 0;
    display: flex;
    justify-content: center;
    align-items: center;
    margin-left: 10px;
    margin-top: 10px;
    color: #ccc;
  }
  .fb_main .fb_tool .up_img_item {
    margin-left: 10px;
    margin-top: 10px;
  }
  .fb_main .form_btn_wrap {
    margin-top: 10px;
    display: flex;
    justify-content: flex-end;
  }
  .fb_main .form_btn_wrap button {
    margin: 0;
    width: 31%;
    display: flex;
    color: white;
    background-color: var(--themeColor);
  }

</style>
