<template>
  <div id="app">
    <!-- the router outlet, where all matched components would be viewed -->
    <div class="navbar is-pulled-right no-print" role="navigation" aria-label="main navigation">
        &nbsp;<a class="navbar-item button is-dark" href="/login" v-if="!isLoggedin">Login</a>
        &nbsp;<a class="navbar-item button is-dark" href="/register" v-if="!isLoggedin">Register</a>&nbsp;&nbsp;
    </div>
    <span class="loggedin is-pulled-right no-print" v-if="isLoggedin==true">
        Welcome <strong>{{username}}</strong>&nbsp;&nbsp;&nbsp;<button class="button is-small is-info" v-on:click="logout">Logout</button>
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
      username: '',
      token: ''
    }
  },
  created () {
    //console.log(this.$ls);
    if(this.$ls.get('username')){
      this.token = this.$ls.get('token');
      axios.get('http://localhost:3005/v1/account/me',{
        headers: {'Authorization': 'Bearer '+this.token}
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
  },
  methods:{
    logout (){
      axios.get('http://localhost:3005/v1/account/logout',{
        headers: {'Authorization': 'Bearer '+this.token}
      })
      .then(response => {
        if(response.status==200){
          this.isLoggedin = false
          this.$router.push('/login')
          this.$ls.remove('token')
        }
      })
      .catch(error => {
        console.log(error);
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
@media print
{    
    .no-print, .no-print *
    {
        display: none !important;
    }
}
@media screen
{    
    .no-screen, .no-screen *
    {
        display: none !important;
    }
}
</style>