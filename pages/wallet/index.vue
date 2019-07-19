<template>
  <div>
    <div class="header">创建钱包</div>
    <div class="content-container">
        <p class="danger">重要提示：</p>
        <p class="default" style="margin-top: 4px;">目前创建钱包网页小工具基于开源代码生成，仅限内部使用和过渡期间使用，后续建议开发人员基于开源代码进行独立的二次开发。</p>
        <p class="default" style="margin-top: 4px;">钱包的创建是在客户端生成的，不会被存档，所以<span class="danger">请相关人员务必抄写好钱包秘钥和地址，妥善保存，若丢失，则不可找回！！！</span></p>
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
