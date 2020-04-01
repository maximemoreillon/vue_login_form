<template>
  <div class="login_form_wrapper">

    <!-- Check if processing a login operation -->
    <template v-if="!logging_in">

      <!-- not logged in => show form -->
      <form
        v-if="!logged_in"
        v-on:submit.prevent="login()">
        <input type="text" v-model="input_data.username" placeholder="username">
        <input type="password" v-model="input_data.password" placeholder="password">
        <input type="submit" value="login">

        <!-- TODO: Error message might need to be not only here -->
        <div class="error_message">{{error_message}}</div>
      </form>

      <!-- logged in (has JWT) => show logout button (to delete JWT) -->
      <template v-else>

        <!-- Show user info if available -->
        <div class="" v-if="user">
          Logged in as {{user.properties.username}}
        </div>

        <!-- loader for user info -->
        <Loader v-else message="Loading user info..." />


        <!-- Show logout button if JWT in cokies -->
        <form
          v-on:submit.prevent="logout()">
          <input type="submit" value="logout">
        </form>

      </template>
    </template>

    <!-- TODO: Replace by a proper loader -->
    <Loader v-else message="Processing..." />

  </div>

</template>

<script>
import axios from 'axios'
import VueCookies from 'vue-cookies'

import Loader from '@moreillon/vue_loader'

export default {
  name: 'LoginForm',
  props: {
    api: String,
  },
  components: {
    Loader,
  },
  data(){
    return {
      input_data: {
        username: '',
        password: '',
      },

      error_message: '',
      logging_in: false,
      logged_in: false,

      user: null,
      user_loading: false,

    }
  },
  mounted(){
    this.check_cookies()

  },
  methods: {
    check_cookies(){
      var jwt = VueCookies.get('jwt')
      if(jwt) {
        this.logged_in = true
        this.whoami()
      }
      else this.logged_in = false
    },
    whoami(){
      var jwt = VueCookies.get('jwt')
      if(jwt){
        this.user = null
        this.user_loading = true;
        axios.post(`${this.api}/whoami`, {},
        { headers: { Authorization : `Bearer ${jwt}`} })
        .then(response => { this.user = response.data})
        .catch(error => {
          if(error.response) this.error_message = error.response.data
          else this.error_message = "System error"
         })
        .finally( () => { this.user_loading = false })
      }

    },
    login(){
      this.logging_in = true;

      axios.post(`${this.api}/login`, {
        username: this.input_data.username,
        password: this.input_data.password
      })
      .then(response => {
        // save the cookie in the current domain
        VueCookies.set("jwt", response.data.jwt, '14d', null, null)
        // Clear the error message
        this.error_message = null;

        this.check_cookies()
      })
      .catch(error => {
        if(error.response) this.error_message = error.response.data
        else this.error_message = "System error"
       })
      .finally( () => { this.logging_in = false })
    },
    logout(){
      VueCookies.remove('jwt')
      this.check_cookies()
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>

</style>
