<template>
  <div class="search_input">
    <navigator url="/pages/search/main" open-type="navigate">
      搜索
    </navigator>
  </div>
</template>

<script>
    export default {
        name: "SearchInput"
    }
</script>

<style scoped>
  .search_input {
    height: 45px;
    padding: 10px;
    background-color: #eb4450;
  }
  .search_input navigator {
    height: 100%;
    display: flex;
    justify-content: center;
    align-items: center;
    background-color: #fff;
    border-radius: 7.5px;
    color: #666;
  }

</style>
