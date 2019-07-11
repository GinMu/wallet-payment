<template>
  <div>
    <div class="header">钱包转账</div>
    <div style="margin: 58px 0;">
        <el-form :model="form" :rules="rules" ref="form" label-width="140px" style="width: 600px;margin: 58px auto;">
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
                <el-select v-model="form.currency" placeholder="币种" style="position: absolute; left: 620px; top: 0; width: 100px;" @change="updatePlaceHolder">
                    <el-option v-for="item in tokens" :key="item.value" :label="item.label" :value="item.value"></el-option>
                </el-select>

                <p class="choose-token default-color">选择转账币种</p>
            </div>
            <el-form-item label="转账备注：" prop="memo">
                <el-input type="text" v-model="form.memo" autocomplete="off" placeholder="请输入留言"></el-input>
            </el-form-item>

            <el-button type="success" style="margin-left: 140px;" @click="pay">确认转账</el-button>
            <el-button type="default" style="margin-left: 80px;" @click="reset">清空内容</el-button>
        </el-form>
    </div>
  </div>
</template>

<script>
import BigNumber from "bignumber.js";
import * as jtWallet from "jcc_wallet/lib/jingtum";
import { JcExchange } from "jcc_rpc";
import { transferAccount } from "jcc_exchange";
import { isEmptyObject } from "jcc_common";
export default {
  components: {},
  data() {
    return {
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
      exHosts: ["ejedwdjbbl8jgf.jccdex.cn"],
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
  created() {},
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
            this.form.currency
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
    async confirm() {
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
        this.$message.success({
          message: "转账成功",
          duration: 5000
        });
      } catch (error) {
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
.choose-token {
  position: absolute;
  left: 730px;
  top: 0;
  width: 100px;
  text-align: center;
  height: 40px;
  line-height: 40px;
  font-size: 16px;
}
</style>
