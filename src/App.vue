<template>
  <div id="app">
    <!-- the router outlet, where all matched components would ber viewed -->
    <div class="navbar is-pulled-right" role="navigation" aria-label="main navigation">
        &nbsp;<a class="navbar-item button is-dark" href="/login" v-if="!isLoggedin">Login</a>
        &nbsp;<a class="navbar-item button is-dark" href="/register" v-if="!isLoggedin">Register</a>&nbsp;&nbsp;
    </div>
    <span class="loggedin is-pulled-right" v-if="isLoggedin">
        Welcome <strong>{{username}}</strong>&nbsp;&nbsp;&nbsp;<button class="button is-small is-info">Logout</button>
    </span>
    <router-view></router-view>
  </div>
</template>

<script>
import Vue from 'vue'
import axios from 'axios';
import VueLocalStorage from 'vue-localstorage'
 
Vue.use(VueLocalStorage, {name: 'ls'})

export default {
  name: 'app',
  data () {
    return {
      isLoggedin: false,
      username: ''
    }
  },
  created () {
    console.log(this.$ls);
    if(this.$ls.get('username')){
      var token = this.$ls.get('token');
      axios.get('http://localhost:3005/v1/account/me',{
        headers: {'Authorization': 'Bearer '+token}
      })
      .then(response => {
        if(response.status==200){
          this.isLoggedin = true
          this.username = this.$ls.get('username')
        }
      })
      .catch(error => {
        console.log(error);
        this.$router.push('/login')
      })
    }
  }
}
</script>
<!-- styling for the component -->
<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 10px;
}
</style>