<template>
  <div>
    <div class="header">创建钱包</div>
    <div class="content-container">
        <p class="danger">重要提示：</p>
        <p class="default" style="margin-top: 4px;">数字资产去中心化交易平台，不保存您的钱包数据，特别是钱包密钥，<span class="danger">一旦丢失，无法找回，请复制保存并抄写，</span>务必妥善保管！！！</p>
        <el-form ref="form" :model="form" label-width="100px" style="margin-top: 63px;">
            <el-form-item label="钱包地址：">
                <el-input v-model="form.address" :readonly="true"></el-input>
            </el-form-item>
            <el-form-item label="钱包秘钥：">
                <el-input v-model="form.secret" :readonly="true"></el-input>
            </el-form-item>
        </el-form>
        <div style="text-align: center; margin-top: 40px;">
            <el-button type="success">
                <nuxt-link to="/">确认</nuxt-link> 
            </el-button>

            <el-button type="primary" @click="generate">
                重新生成
            </el-button>
        </div>
    </div>
  </div>
</template>

<script>
import * as jtWallet from "jcc_wallet/lib/jingtum";
export default {
  components: {},
  data() {
    return {
      form: {
        address: "",
        secret: ""
      }
    };
  },
  created() {
    this.generate();
  },
  methods: {
    generate() {
      const wallet = jtWallet.createWallet();
      this.form = wallet;
    }
  }
};
</script>

<style>
</style>
