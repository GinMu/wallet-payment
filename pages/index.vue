<template>
  <div class="container">
    <div>
      <div class="empty-wallet-bg"></div>

      <div class="default-color" style="margin-top: 10px;">若暂无钱包，请先创建钱包</div>

      <p style="margin-top: 20px;" v-if="!isWebkit">建议使用Chrome浏览器打开本页面</p>
      <div style="margin-top: 133px;">
        <el-button type="success">
          <nuxt-link to="/wallet">创建钱包</nuxt-link>
        </el-button>
        <el-button type="primary" style="margin-left: 80px;">
          <nuxt-link to="/payment">钱包转账</nuxt-link> 
        </el-button>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  components: {},
  data() {
    return {
      isWebkit: navigator.userAgent.indexOf("AppleWebKit") > -1
    };
  }
};
</script>

<style>
.container {
  margin: 0 auto;
  min-height: 100vh;
  display: flex;
  justify-content: center;
  text-align: center;
}

.empty-wallet-bg {
  background: url("~assets/img/empty-wallet.png");
  background-repeat: no-repeat;
  margin-top: 253px;
  width: 280px;
  height: 153px;
  background-size: contain;
}

.title {
  font-family: "Quicksand", "Source Sans Pro", -apple-system, BlinkMacSystemFont,
    "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif;
  display: block;
  font-weight: 300;
  font-size: 100px;
  color: #35495e;
  letter-spacing: 1px;
}

.subtitle {
  font-weight: 300;
  font-size: 42px;
  color: #526488;
  word-spacing: 5px;
  padding-bottom: 15px;
}

.links {
  padding-top: 15px;
}
</style>
