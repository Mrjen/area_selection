<script>
import AreaSelect from './areaSelect.vue'

export default {
  name: 'App',
  components: {
    AreaSelect
  }
}
</script>

<template>
  <div class="app-container">
    <!-- <h2 class="app-title">深圳市罗湖区建筑工务署第三方评估-深圳市罗湖区建筑工务署第三方评估</h2> -->
    <AreaSelect />
  </div>
</template>

<style>
body {
  margin: 0;
  font-family: 'PingFang SC', 'Microsoft YaHei', sans-serif;
}

.app-container {
  padding: 20px;
}

.app-title {
  margin-bottom: 20px;
  color: #17233d;
  text-align: center;
}
</style>
