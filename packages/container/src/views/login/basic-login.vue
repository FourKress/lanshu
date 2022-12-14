<template>
  <div class="basic-login">
    <div class="logo"></div>

    <div class="title" v-if="isWechatLogin">请使用微信扫码登录</div>
    <div class="title" v-else>
      {{ isAccountLogin ? '欢迎使用登录' : '请使用APP扫码登录' }}
    </div>

    <div class="title-tips" v-if="isAccountLogin || isWechatLogin">
      {{
        isWechatLogin
          ? '微信扫码可以同时进行注册和登录'
          : '未注册的手机号，输入后将自动注册'
      }}
    </div>

    <template v-if="isAccountLogin && !isWechatLogin">
      <div class="input-panel">
        <el-form
          :model="loginForm"
          :rules="rules"
          ref="loginForm"
          label-width="0px"
        >
          <div class="phone">
            <el-form-item label="" prop="phoneNum">
              <el-input
                type="text"
                maxlength="13"
                placeholder="输入手机号码，登录或注册"
                v-model="loginForm.phoneNum"
              />
            </el-form-item>
          </div>
        </el-form>

        <div
          class="login-btn"
          :class="activeBtn && 'active'"
          @click="handleLogin"
        >
          下一步
        </div>

        <div class="checkbox-row">
          <el-checkbox v-model="protocolChecked"></el-checkbox>
          <span class="text">我已阅读并同意</span>
          <span class="link" @click="openUrl">《用户服务协议》</span>
          <span class="link" @click="openUrl">《隐私协议》</span>
        </div>
        <div class="checkbox-row">
          <el-checkbox v-model="autoLoginChecked"></el-checkbox>
          <span class="text">自动登录</span>
        </div>
      </div>
    </template>

    <template v-if="!isAccountLogin && !isWechatLogin">
      <div class="qrcode-wrap app-qrcode"></div>
    </template>

    <template v-if="isWechatLogin">
      <div class="qrcode-wrap wechat-qrcode"></div>
      <div class="wechat-checkbox-row">
        <el-checkbox v-model="autoLoginChecked"></el-checkbox>
        <span class="text">自动登录</span>
      </div>
    </template>

    <template v-else>
      <OtherLogin @wechatLogin="handleWechatLogin" />
    </template>
  </div>
</template>

<script>
import { renderProcess } from '@lanshu/render-process';
import OtherLogin from './other-login';

export default {
  name: 'Basic-login',
  props: {
    isAccountLogin: {
      type: Boolean,
      default: true,
    },
  },
  components: {
    OtherLogin,
  },
  data() {
    return {
      isWechatLogin: false,
      loginForm: {
        phoneNum: '',
      },
      rules: {
        phoneNum: [
          {
            required: true,
            message: '请输入有效的电话号码',
            trigger: ['change', 'blur'],
          },
          {
            message: '请输入有效的电话号码',
            max: 13,
            min: 13,
            trigger: ['change', 'blur'],
          },
        ],
      },
      protocolChecked: false,
      autoLoginChecked: false,
      appQrcodeTimer: null,
      wechatQrcodeTimer: null,
    };
  },
  computed: {
    activeBtn() {
      return this.protocolChecked && this.loginForm.phoneNum;
    },
  },
  watch: {
    isAccountLogin(val) {
      clearInterval(this.wechatQrcodeTimer);
      if (val) {
        clearInterval(this.appQrcodeTimer);
      } else if (!this.isWechatLogin) {
        this.loopAppLogin();
      }
      this.isWechatLogin = false;
    },
    'loginForm.phoneNum': function (val, oldVal) {
      if (oldVal?.length > val?.length) return;
      if (val) {
        const matches = /^(\d{3})(\d{4})?(\d{4})?$/.exec(
          this.getRealPhoneNum(val),
        );
        if (matches) {
          let phoneNum = `${matches[1]} `;
          if (matches[2]) {
            phoneNum += `${matches[2]} `;
          }
          if (matches[3]) {
            phoneNum += `${matches[3]}`;
          }
          console.log(phoneNum);
          this.loginForm.phoneNum = phoneNum;
        }
      }
    },
  },
  methods: {
    openUrl(url) {
      renderProcess.openUrl('https://www.baidu.com' || url);
    },
    handleWechatLogin() {
      this.isWechatLogin = true;
      clearInterval(this.appQrcodeTimer);
      this.$emit('update:isAccountLogin', false);
      this.$nextTick(() => {
        this.isWechatLogin = true;
        this.loopWechatLogin();
      });
    },
    getRealPhoneNum(val) {
      return val.replace(/ /g, '');
    },
    handleLogin() {
      if (!this.protocolChecked) {
        this.$message.warning('请阅读并勾选《用户服务协议》《隐私协议》');
        return;
      }
      if (!this.activeBtn) return;

      this.$emit('sendLogin', this.getRealPhoneNum(this.loginForm.phoneNum));
    },
    loopAppLogin() {
      this.appQrcodeTimer = setInterval(() => {
        console.log('APP扫码登录');
        this.$emit('sendLogin');
      }, 500);
    },
    loopWechatLogin() {
      this.wechatQrcodeTimer = setInterval(() => {
        console.log('微信扫码登录');
      }, 500);
    },
  },
};
</script>

<style scoped lang="scss">
.logo {
  width: 60px;
  height: 60px;
  background: #333;
  border-radius: 12px;
}

.title {
  font-size: 30px;
  color: $main-text-color;
  font-weight: bold;
  margin-top: 20px;
}

.title-tips {
  font-size: 14px;
  color: $tips-text-color;
  margin-top: 8px;
}

.input-panel {
  margin: 60px 0;

  .phone,
  .login-btn {
    width: 100%;
    height: 60px;
    line-height: 60px;
    border-radius: 6px;
    box-sizing: border-box;
  }

  .login-btn {
    text-align: center;
    color: $bg-white-color;
    background-color: #87a1cd;
    cursor: pointer;
    margin-bottom: 60px;

    &.active {
      background: $primary-color;
    }
  }

  .phone {
    background-color: $bg-hover-grey-color;
    margin-bottom: 20px;

    ::v-deep .el-input__inner {
      width: 100%;
      height: 60px;
      line-height: 24px;
      font-size: 18px;
      outline: none;
      background-color: transparent;
      border: none;
      padding: 18px 26px;
      box-sizing: border-box;
      color: $main-text-color;

      &::placeholder {
        color: $tips-text-color;
        font-size: 18px;
      }
    }
  }

  .checkbox-row {
    color: $minor-text-color;
    font-size: 14px;
    margin-bottom: 22px;
    flex-direction: column;
    justify-content: flex-start;
    align-items: center;

    .text {
      margin-left: 8px;
    }

    .link {
      cursor: pointer;
      color: $primary-color;
    }

    &:last-child {
      margin-bottom: 0;
    }
  }
}

.wechat-checkbox-row {
  color: $minor-text-color;
  font-size: 14px;
  display: flex;
  justify-content: center;
  align-items: center;

  .text {
    margin-left: 8px;
  }

  .link {
    cursor: pointer;
    color: $primary-color;
  }
}

.qrcode-wrap {
  width: 235px;
  height: 232px;

  background-color: #333333;

  &.app-qrcode {
    margin: 88px auto 90px;
  }

  &.wechat-qrcode {
    margin: 60px auto 90px;
  }
}
</style>
