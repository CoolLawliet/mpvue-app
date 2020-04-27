<template>
  <div class="tabs">
    <div class="tabs_title">
      <div
        v-for="(item,index) in tabs"
        :key="item.id"
        :class="'title_item ' + (item.isActive?'active':'') "
        @tap="handleItemTap"
        :data-index="index"
      >
        {{item.value}}
      </div>
    </div>
    <div class="tabs_content">
      <slot></slot>
    </div>
  </div>
</template>

<script>
    export default {
        props:{
          tabs:{
            type:Array,
            value:[]
          }
        },
      methods: {
        // 点击事件
        handleItemTap(e){
          // 1 获取点击的索引
          const {index}=e.mp.currentTarget.dataset;
          // 2 触发 父组件中的事件 自定义
          this.$emit("tabsItemChange",{index});
        }
      }
    }
</script>

<style scoped>
  .tabs_title{
    display: flex;
  }
  .title_item{
    display: flex;
    justify-content: center;
    align-items: center;
    flex: 1;
    padding: 7.5px 0;
  }
  .active{
    color: #eb4450;
    border-bottom: 2.5px solid currentColor;
  }
</style>
