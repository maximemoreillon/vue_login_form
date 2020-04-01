<template>
  <div id="app">
    <LoginForm
      api="https://authentication.maximemoreillon.com"/>
  </div>
</template>

<script>
import LoginForm from './LoginForm.vue'

export default {
  name: 'App',
  components: {
    LoginForm
  }
}
</script>

<style>

</style>
