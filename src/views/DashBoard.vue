<template>
<NavbarComponent></NavbarComponent>
<div class="container">
<router-view/>
</div>
</template>

<script>
import NavbarComponent from '../components/NavbarComponent.vue'
export default {
  created () {
    const token = document.cookie.replace(/(?:(?:^|.*;\s*)vueshop\s*=\s*([^;]*).*$)|^.*$/, '$1')
    this.$http.defaults.headers.common.Authorization = token
    const api = `${process.env.VUE_APP_API}api/user/check`
    this.$http.post(api, this.user)
      .then((res) => {
        if (!res.data.success) {
          this.$router.push('/login')
        }
      })
  },
  components: {
    NavbarComponent
  }
}
</script>
