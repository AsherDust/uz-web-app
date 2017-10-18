<template>
  <div class="col-sm-4 col-sm-offset-4">
    <form>
      <div class="form-group">
        <label class="sr-only" for="inputAccount">账号</label>
        <input type="text" class="form-control" id="inputAccount" placeholder="账号" v-model="account">
      </div>
      <div class="form-group">
        <label class="sr-only" for="inputPassword">密码</label>
        <input type="password" class="form-control" id="inputPassword" placeholder="密码" v-model="password">
      </div>
      <button class="btn btn-info btn-block" @click="accountLogin">登 录</button>
    </form>
    <router-link tag="button" class="btn btn-link" to="/register">注册</router-link>
  </div>
</template>

<script>
import axios from 'axios'
import { baseUrl, api } from '../../config/api'
const JSEncrypt = require('jsencrypt').JSEncrypt
const qs = require('qs')

export default {
  name: 'Login',
  data () {
    return {
      account: null,
      password: null
    }
  },
  methods: {
    accountLogin () {
      if (this.account === null || this.account === '') {
        alert('账号不能为空！')
        return
      }
      if (this.password === null || this.password === '') {
        alert('密码不能为空！')
        return
      }

      // 获取密钥
      axios({
        method: 'get',
        url: api.getKey,
        baseURL: baseUrl
      })
        .then(response => {
          const serialNum = response.data.data.serialNum
          let publicKey = response.data.data.key

          // 加密密码
          const crypt = new JSEncrypt()
          crypt.setKey(publicKey)
          const passport = crypt.encrypt(serialNum + this.password)

          let data = {
            serialNum: serialNum,
            account: this.account,
            passport: passport
          }
          // 登录请求
          axios({
            method: 'post',
            url: api.login,
            baseURL: baseUrl,
            data: data,
            transformRequest: function (data) {
              return qs.stringify(data)
            }
          })
            .then(response => {
              if (response.data.code !== 666) {
                alert(response.data.msg)
              } else {
                console.log(response.data.data)
                sessionStorage.setItem('token', response.data.data)
                this.$router.push('/')
              }
            })
            .catch(error => {
              console.log(error)
            })
        })
        .catch(error => {
          console.log(error)
        })
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style>

</style>
