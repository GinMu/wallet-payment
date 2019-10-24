<template>
  <div>
    <div class="header">钱包转账</div>
    <div class="content-container" >
        <el-form :model="form" :rules="rules" ref="form" label-width="140px">
            <el-form-item label="我的钱包地址：" prop="address">
                <el-input type="text" v-model="form.address" autocomplete="off" placeholder="请输入转出钱包地址"></el-input>
            </el-form-item>
            <el-form-item label="我的钱包秘钥：" prop="secret">
                <el-input type="text" v-model="form.secret" autocomplete="off" placeholder="请输入转出钱包秘钥"></el-input>
            </el-form-item>
            <el-form-item label="对方钱包地址：" prop="to">
                <el-input type="text" v-model="form.to" autocomplete="off" placeholder="请输入对方接收的钱包地址"></el-input>
            </el-form-item>
            <div style="position: relative;">
                <el-form-item label="转出数量：" prop="amount">
                    <el-input type="text" v-model="form.amount" autocomplete="off" :placeholder="amountPlaceHolder"></el-input>
                </el-form-item>
                <el-select v-model="form.currency" placeholder="币种" class="choose-token-select" @change="updatePlaceHolder">
                    <el-option v-for="item in tokens" :key="item.value" :label="item.label" :value="item.value"></el-option>
                </el-select>

                <p class="choose-token default-color">选择转账币种</p>
            </div>
            <el-form-item label="转账备注：" prop="memo" class="memo">
                <el-input type="text" v-model="form.memo" autocomplete="off" placeholder="请输入留言"></el-input>
            </el-form-item>
            <el-button type="success" style="margin-left: 140px;" :loading="loading" @click="pay">{{loading ? "转账中": "确认转账"}}</el-button>
            <el-button type="default" style="margin-left: 80px;" @click="reset">清空内容</el-button>
        </el-form>
    </div>
  </div>
</template>

<script>
import BigNumber from "bignumber.js";
import * as jtWallet from "jcc_wallet/lib/jingtum";
import { JcExchange, JcConfig, JcExplorer } from "jcc_rpc";
import { transferAccount } from "jcc_exchange";
import { isEmptyObject } from "jcc_common";
import axios from "axios";
export default {
  components: {},
  data() {
    return {
      loading: false,
      form: {
        address: "",
        secret: "",
        to: "",
        amount: "",
        memo: "",
        currency: ""
      },
      tokens: [
        {
          value: "SWT",
          label: "SWTC"
        },
        {
          value: "COCO",
          label: "COCO"
        }
      ],
      amountPlaceHolder: "可转出 - - 数量：- - 个",
      balance: null,
      exHosts: [],
      scanHosts: [],
      rules: {
        address: {
          required: true,
          validator: (rule, value, callback) => {
            if (jtWallet.isValidAddress(value.trim())) {
              return callback();
            }
            callback(new Error("请输入正确的钱包地址"));
          }
        },
        to: {
          required: true,
          validator: (rule, value, callback) => {
            if (jtWallet.isValidAddress(value.trim())) {
              return callback();
            }
            callback(new Error("请输入正确的钱包地址"));
          }
        },
        secret: {
          required: true,
          validator: (rule, value, callback) => {
            if (jtWallet.isValidSecret(value.trim())) {
              return callback();
            }
            callback(new Error("请输入正确的钱包秘钥"));
          }
        },
        amount: {
          required: true,
          validator: (rule, value, callback) => {
            const bn = new BigNumber(value);
            if (BigNumber.isBigNumber(bn) && bn.gt(0)) {
              return callback();
            }
            callback(new Error("请输入大于0的数字"));
          }
        }
      }
    };
  },
  created() {
    axios
      .get("https://jcconfig.jccdex.cn/jc_config.json?t=" + Date.now())
      .then(res => {
        if (res && res.status === 200 && res.data) {
          this.exHosts = res.data.exHosts;
          this.scanHosts = res.data.scanHosts;
        }
      })
      .catch(error => {
        console.log(error);
      });
  },
  methods: {
    updatePlaceHolder() {
      if (!this.balance) {
        this.requestBalance().then(() => {
          if (this.balance) {
            this.setAmountPlaceHolder();
          } else {
            this.amountPlaceHolder = "可转出 - - 数量：- - 个";
          }
        });
      } else {
        this.setAmountPlaceHolder();
      }
    },
    setAmountPlaceHolder() {
      const b = this.balance.find(
        b => b.currency.toLowerCase() === this.form.currency.toLowerCase()
      );
      if (b) {
        this.amountPlaceHolder = `可转出${b.title}数量：${b.value}个`;
      } else {
        this.amountPlaceHolder = "可转出 - - 数量：- - 个";
      }
    },
    async requestBalance() {
      if (!jtWallet.isValidAddress(this.form.address)) {
        return;
      }
      const hosts = this.exHosts;
      const port = 443;
      const https = true;
      const instance = new JcExchange(hosts, port, https);
      const res = await instance.getBalances(this.form.address);
      if (res.result) {
        this.balance = res.data;
      }
    },
    pay() {
      this.$refs.form.validate(valid => {
        if (!valid) {
          return;
        }
        if (!this.form.currency) {
          return this.$message.error({
            message: "请选择转账币种",
            duration: 5000
          });
        }
        this.$confirm(
          `确认向 ${this.form.to} 转 ${this.form.amount} ${
            this.form.currency === "SWT" ? "SWTC" : this.form.currency
          }?`,
          "转账确认",
          {
            confirmButtonText: "确定",
            cancelButtonText: "取消"
          }
        )
          .then(() => {
            this.confirm();
          })
          .catch(() => {});
      });
    },
    orderDetail(instance, hash) {
      return new Promise((resolve, reject) => {
        setTimeout(async () => {
          try {
            const orderDetail = await instance.orderDetail(Date.now(), hash);
            resolve(orderDetail);
          } catch (error) {
            return reject(error);
          }
        }, 15000);
      });
    },

    async getOrderDetail(hash) {
      const instance = new JcExplorer(this.scanHosts, 443, true);
      let orderDetail = null;
      let count = 0;
      while (orderDetail === null) {
        // try 10
        if (count === 10) {
          break;
        }
        try {
          orderDetail = await this.orderDetail(instance, hash);
        } catch (error) {
          console.log("request order error:", error);
        } finally {
          count = count + 1;
        }
      }
      if (!orderDetail) {
        throw new Error("获取转账详情失败，请去浏览器再次确认转账是否成功");
      }
      return orderDetail;
    },

    async confirm() {
      if (
        !Array.isArray(this.exHosts) ||
        this.exHosts.length === 0 ||
        !Array.isArray(this.scanHosts) ||
        this.scanHosts.length === 0
      ) {
        this.$message.error({
          message: "请重新刷新网页",
          duration: 5000
        });
        return;
      }
      this.loading = true;
      const obj = {
        currency: this.form.currency,
        amount: this.form.amount,
        address: this.form.address.trim(),
        secret: this.form.secret.trim(),
        to: this.form.to.trim(),
        issuer: "jGa9J9TkqtBcUoHe2zqhVFFbgUVED6o9or",
        memo: this.form.memo,
        hosts: this.exHosts,
        https: true,
        port: 443
      };
      try {
        const hash = await transferAccount(obj);
        const orderDetail = await this.getOrderDetail(hash);
        if (
          orderDetail &&
          orderDetail.result &&
          orderDetail.data &&
          orderDetail.data.succ === "tesSUCCESS"
        ) {
          this.$message.success({
            message: "转账成功",
            duration: 5000
          });
        } else {
          this.$message.error({
            message: "转账失败",
            duration: 5000
          });
        }
        this.loading = false;
      } catch (error) {
        this.loading = false;
        this.$message.error({
          message: error.message,
          duration: 5000
        });
      }
    },
    reset() {
      this.$refs.form.resetFields();
    }
  }
};
</script>

<style>
</style>
