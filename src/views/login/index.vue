<template>
  <div class="login-container">
    <el-form
      ref="loginForm"
      :model="loginForm"
      :rules="loginRules"
      class="login-form"
      autocomplete="on"
      label-position="left"
    >
      <div class="title-container">
        <h3 class="title">Login Form</h3>
      </div>
      <el-form-item prop="loginAccount">
        <span class="svg-container">
          <svg-icon icon-class="user" />
        </span>
        <el-input
          ref="loginAccount"
          v-model="loginForm.loginAccount"
          placeholder="请输入用户名"
          name="loginAccount"
          type="text"
          tabindex="1"
          autocomplete="on"
        />
      </el-form-item>
      <el-tooltip
        v-model="capsTooltip"
        content="Caps lock is On"
        placement="right"
        manual
      >
        <el-form-item prop="password">
          <span class="svg-container">
            <svg-icon icon-class="password" />
          </span>
          <el-input
            :key="passwordType"
            ref="password"
            v-model="loginForm.password"
            :type="passwordType"
            placeholder="请输入不低于6位的密码"
            name="password"
            tabindex="2"
            autocomplete="on"
            @keyup.native="checkCapslock"
            @blur="capsTooltip = false"
            @keyup.enter.native="handleLogin"
          />
          <span
            class="show-pwd"
            @click="showPwd"
          >
            <svg-icon :icon-class="passwordType === 'password' ? 'eye' : 'eye-open'" />
          </span>
        </el-form-item>
      </el-tooltip>
      <el-button
        :loading="loading"
        type="primary"
        style="width:100%;margin-bottom:30px;"
        @click.native.prevent="handleLogin"
      >Login</el-button>
      <div style="position:relative">
        <div class="tips">
          <span>忘记密码?</span>
          <span><a
            href="javascript:void(0)"
            @click="updateDialogVisible=true"
          >点击修改</a></span>
        </div>
        <div class="tips">
          <span style="margin-right:18px;">没有账号?</span>
          <span><a
            href="javascript:void(0)"
            @click="dialogVisible=true"
          >点击注册</a></span>
        </div>
        <el-button
          class="thirdparty-button"
          type="primary"
          @click="showDialog=true"
        >
          Or connect with
        </el-button>
      </div>
    </el-form>
    <el-dialog
      title="Or connect with"
      :visible.sync="showDialog"
    >
      Can not be simulated on local, so please combine you own business simulation! ! !
      <br>
      <br>
      <br>
      <social-sign />
    </el-dialog>
    <el-dialog
      title="编辑用户信息"
      :visible.sync="updateDialogVisible"
      width="30%"
    >
      <el-form
        ref="updateForm"
        :model="updateForm"
        status-icon
        :rules="updateRules"
        label-width="100px"
        class="demo-ruleForm"
      >
        <el-form-item
          label="用户名"
          prop="loginAccount"
        >
          <el-input
            v-model="updateForm.loginAccount"
            type="text"
            autocomplete="off"
          />
        </el-form-item>
        <el-form-item
          label="旧密码"
          prop="password"
        >
          <el-input
            v-model="updateForm.password"
            type="password"
            autocomplete="off"
          />
        </el-form-item>
        <el-form-item
          label="新密码"
          prop="newPassword"
        >
          <el-input
            v-model="updateForm.newPassword"
            type="password"
            autocomplete="false"
          />
        </el-form-item>
      </el-form>
      <span
        slot="footer"
        class="dialog-footer"
      >
        <el-button @click="updateDialogVisible = false">取 消</el-button>
        <el-button
          type="primary"
          @click="update"
        >确 定</el-button>
      </span>
    </el-dialog>
    <!-- 注册 -->
    <el-dialog
      title="注册用户"
      :visible.sync="dialogVisible"
      width="30%"
    >
      <el-form
        ref="registerForm"
        :model="registerForm"
        status-icon
        :rules="registerRules"
        label-width="100px"
        class="demo-ruleForm"
      >
        <el-form-item
          label="用户名"
          prop="loginAccount"
        >
          <el-input
            v-model="registerForm.loginAccount"
            type="text"
            autocomplete="off"
          />
        </el-form-item>
        <el-form-item
          label="密码"
          prop="password"
        >
          <el-input
            v-model="registerForm.password"
            type="password"
            autocomplete="off"
          />
        </el-form-item>
        <el-form-item
          label="别名"
          prop="alias"
        >
          <el-input
            v-model="registerForm.alias"
            type="text"
            autocomplete="false"
          />
        </el-form-item>
      </el-form>
      <span
        slot="footer"
        class="dialog-footer"
      >
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button
          type="primary"
          @click="register"
        >确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>
<script>
import { validateLoginAccount } from '@/utils/validate'
import SocialSign from './components/SocialSignin'
import { login } from '@/api/user'
import { setToken } from '@/utils/auth'

export default {
  name: 'Login',
  components: { SocialSign },
  data() {
    const validateLogin = (rule, value, callback) => {
      if (!validateLoginAccount(value)) {
        callback(new Error('请输入用户名'))
      } else {
        callback()
      }
    }
    const validatePassword = (rule, value, callback) => {
      if (value.length < 6) {
        callback(new Error('请输入不低于6位的密码'))
      } else {
        callback()
      }
    }
    // const validatePasswordAndNewPassword = (rule, value, callback) => {
    //   if (value.password === value.newPassword) {
    //     callback(new Error('新旧密码一致'))
    //   } else if (value.password.length < 6) {
    //     callback(new Error('请输入不低于6位的密码'))
    //   } else if (value.newPassword.length < 6) {
    //     callback(new Error('请输入不低于6位的密码'))
    //   } else {
    //     callback()
    //   }
    // }
    return {
      loginForm: {
        loginAccount: 'admin',
        password: 'admin123456'
      },
      loginRules: {
        loginAccount: [{ required: true, trigger: 'blur', validator: validateLogin }],
        password: [{ required: true, trigger: 'blur', validator: validatePassword }]
      },
      registerForm: {
        loginAccount: '',
        password: '',
        alias: ''
      },
      registerRules: {
        loginAccount: [{ required: true, message: '登录账号不能为空', trigger: 'blur' }],
        password: [{ required: true, message: '登录尼玛不能为空', trigger: 'blur' }],
        alias: [{ required: true, message: '别名不能为空', trigger: 'blur' }]
      },
      updateForm: {
        loginAccount: '',
        password: '',
        newPassword: ''
      },
      updateRules: {
        loginAccount: [{ required: true, message: '登录账号不能为空', trigger: 'blur' }],
        password: [{ required: true, message: '旧密码不能为空', trigger: 'blur' }],
        newPassword: [{ required: true, message: '新密码不能为空', trigger: 'blur' }]
      },
      passwordType: 'password',
      capsTooltip: false,
      loading: false,
      showDialog: false,
      redirect: undefined,
      dialogVisible: false,
      updateDialogVisible: false,
      otherQuery: {}
    }
  },
  watch: {
    $route: {
      handler: function(route) {
        const query = route.query
        if (query) {
          this.redirect = query.redirect
          this.otherQuery = this.getOtherQuery(query)
        }
      },
      immediate: true
    }
  },
  created() {
    // window.addEventListener('storage', this.afterQRScan)
  },
  mounted() {
    if (this.loginForm.loginAccount === '') {
      this.$refs.loginAccount.focus()
    } else if (this.loginForm.password === '') {
      this.$refs.password.focus()
    }
    if (this.registerForm.loginAccount === '') {
      this.$refs.loginAccount.focus()
    } else if (this.registerForm.password === '') {
      this.$refs.password.focus()
    }
    if (this.updateForm.loginAccount === '') {
      this.$refs.loginAccount.focus()
    } else if (this.updateForm.password === '') {
      this.$refs.password.focus()
    }
  },
  destroyed() {
    // window.removeEventListener('storage', this.afterQRScan)
  },
  methods: {
    register() {
      alert('注册')
    },
    update() {
      alert('修改用户信息')
    },
    // handleClose(done) {
    //   this.$confirm('确认关闭？')
    //     .then(_ => {
    //       done()
    //     })
    //     .catch(_ => { })
    // },
    checkCapslock({ shiftKey, key } = {}) {
      if (key && key.length === 1) {
        if (shiftKey && (key >= 'a' && key <= 'z') || !shiftKey && (key >= 'A' && key <= 'Z')) {
          this.capsTooltip = true
        } else {
          this.capsTooltip = false
        }
      }
      if (key === 'CapsLock' && this.capsTooltip === true) {
        this.capsTooltip = false
      }
    },
    showPwd() {
      if (this.passwordType === 'password') {
        this.passwordType = ''
      } else {
        this.passwordType = 'password'
      }
      this.$nextTick(() => {
        this.$refs.password.focus()
      })
    },
    handleLogin() {
      this.$refs.loginForm.validate(valid => {
        if (valid) {
          this.loading = true
          // this.$store.dispatch('user/login', this.loginForm)
          //   .then(() => {
          //     this.$router.push({ path: this.redirect || '/', query: this.otherQuery })
          //     this.loading = false
          //   })
          //   .catch(() => {
          //     this.loading = false
          //   })
          login(this.loginForm)
            .then((res) => {
              if (res.data.code === 200) {
                setToken('admin-token')
                this.$router.push({ path: '/' })
                this.loading = false
              }
            })
            .catch((err) => {
              console.log(err)
              this.loading = false
            })
        } else {
          console.log('error submit!!')
          return false
        }
      })
    },
    getOtherQuery(query) {
      return Object.keys(query).reduce((acc, cur) => {
        if (cur !== 'redirect') {
          acc[cur] = query[cur]
        }
        return acc
      }, {})
    }
    // afterQRScan() {
    //   if (e.key === 'x-admin-oauth-code') {
    //     const code = getQueryObject(e.newValue)
    //     const codeMap = {
    //       wechat: 'code',
    //       tencent: 'code'
    //     }
    //     const type = codeMap[this.auth_type]
    //     const codeName = code[type]
    //     if (codeName) {
    //       this.$store.dispatch('LoginByThirdparty', codeName).then(() => {
    //         this.$router.push({ path: this.redirect || '/' })
    //       })
    //     } else {
    //       alert('第三方登录失败')
    //     }
    //   }
    // }
  }
}

</script>
<style lang="scss">
/* 修复input 背景不协调 和光标变色 */
/* Detail see https://github.com/PanJiaChen/vue-element-admin/pull/927 */

$bg: #283443;
$light_gray: #fff;
$cursor: #fff;

@supports (-webkit-mask: none) and (not (cater-color: $cursor)) {
  .login-container .el-input input {
    color: $cursor;
  }
}

/* reset element-ui css */
.login-container {
  .el-input {
    display: inline-block;
    height: 47px;
    width: 85%;

    input {
      background: transparent;
      border: 0px;
      -webkit-appearance: none;
      border-radius: 0px;
      padding: 12px 5px 12px 15px;
      color: $light_gray;
      height: 47px;
      caret-color: $cursor;

      &:-webkit-autofill {
        box-shadow: 0 0 0px 1000px $bg inset !important;
        -webkit-text-fill-color: $cursor !important;
      }
    }
  }

  .el-form-item {
    border: 1px solid rgba(255, 255, 255, 0.1);
    background: rgba(0, 0, 0, 0.1);
    border-radius: 5px;
    color: #454545;
  }
}
</style>
<style lang="scss" scoped>
$bg: #2d3a4b;
$dark_gray: #889aa4;
$light_gray: #eee;

.login-container {
  min-height: 100%;
  width: 100%;
  background-color: $bg;
  overflow: hidden;

  .login-form {
    position: relative;
    width: 520px;
    max-width: 100%;
    padding: 160px 35px 0;
    margin: 0 auto;
    overflow: hidden;
  }

  .tips {
    font-size: 14px;
    color: #fff;
    margin-bottom: 10px;

    span {
      &:first-of-type {
        margin-right: 16px;
      }
    }
  }

  .svg-container {
    padding: 6px 5px 6px 15px;
    color: $dark_gray;
    vertical-align: middle;
    width: 30px;
    display: inline-block;
  }

  .title-container {
    position: relative;

    .title {
      font-size: 26px;
      color: $light_gray;
      margin: 0px auto 40px auto;
      text-align: center;
      font-weight: bold;
    }
  }

  .show-pwd {
    position: absolute;
    right: 10px;
    top: 7px;
    font-size: 16px;
    color: $dark_gray;
    cursor: pointer;
    user-select: none;
  }

  .thirdparty-button {
    position: absolute;
    right: 0;
    bottom: 6px;
  }

  @media only screen and (max-width: 470px) {
    .thirdparty-button {
      display: none;
    }
  }
}
</style>
