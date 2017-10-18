<template>
  <div class="col-sm-4 col-sm-offset-4">
    <form>
      <div class="form-group">
        <label class="sr-only" for="inputUsername">用户名</label>
        <input type="text" class="form-control" id="inputUsername" placeholder="用户名" 
        v-model="username">
      </div>
      <div class="form-group">
        <label class="sr-only" for="inputPassword">密码</label>
        <input type="password" class="form-control" id="inputPassword" placeholder="密码" 
        v-model="password">
      </div>
      <div class="form-group">
        <label class="sr-only" for="inputPassword2">确认密码</label>
        <input type="password" class="form-control" id="inputPassword2" placeholder="确认密码" 
        v-model="password2">
      </div>
      <button class="btn btn-info btn-block" @click="usernameRegister">注 册</button>
    </form>
    <router-link tag="button" class="btn btn-link" to="/login">返回登录</router-link>
  </div>
</template>

<script>
import axios from 'axios'
import { baseUrl, api } from '../../config/api'
const JSEncrypt = require('jsencrypt').JSEncrypt
const qs = require('qs')

export default {
  name: 'Register',
  data () {
    return {
      username: null,
      password: null,
      password2: null
    }
  },
  methods: {
    usernameRegister () {
      if (this.username === null || this.username === '') {
        alert('用户名不能为空！')
        return
      }
      if (this.password === null || this.password === '') {
        alert('密码不能为空！')
        return
      }
      if (this.password !== this.password2) {
        alert('密码输入不一致！')
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
          const passport = crypt.encrypt(serialNum + this.password2)

          let data = {
            serialNum: serialNum,
            username: this.username,
            passport: passport
          }
          // 注册请求
          axios({
            method: 'post',
            url: api.register,
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
