<template lang="html">

  <section class="login">
    <b-alert :show="dismissCountDown"
      dismissible
      fade
      variant="success"
      @dismissed="dismissCountDown=0"
      @dismiss-count-down="countDownChanged">
      User telah aktif!
    </b-alert>
    <div class="wrapper d-flex align-items-center auth login-full-bg">
      <div class="row w-100">
        <div class="col-lg-6 py-auto mx-auto">
          <div class="auth-form-dark text-left p-5">
            <h2>Masuk</h2>
            <!-- <h4 class="font-weight-light">Hello! let's get started</h4> -->
            <form class="pt-5">
              <form>
                <div class="form-group">
                  <label for="inputEmail">Email</label>
                  <input type="email" class="form-control" id="inputEmail" aria-describedby="emailHelp" placeholder="john.doe@gmail.com" v-model="email">
                  <i class="mdi mdi-account"></i>
                  <span v-show="'email' in this.error" class="text-danger">{{ this.error.email }}</span>
                </div>
                <div class="form-group">
                  <label>atau</label>
                </div>
                <div class="form-group">
                  <label for="inputPhone">Telepon</label>
                  <input type="text" class="form-control" id="inputPhone" aria-describedby="phoneHelp" placeholder="081123456789" v-model="phone">
                  <i class="mdi mdi-account"></i>
                  <span v-show="'phone' in this.error" class="text-danger">{{ this.error.phone }}</span>
                </div>
                <div class="form-group">
                  <label for="inputPassword">Kata Sandi</label>
                  <input type="password" class="form-control" id="inputPassword" placeholder="" v-model="password">
                  <i class="mdi mdi-eye"></i>
                  <span v-show="'password' in this.error" class="text-danger">{{ this.error.password }}</span>
                </div>
                <div class="mt-5">
                  <div class="btn btn-block btn-warning btn-lg font-weight-medium" @click="login">Masuk</div>
                </div>
                <!-- <div class="mt-3 text-center">
                  <a href="#" class="auth-link text-white">Forgot password?</a>
                </div> -->
                <div class="mt-2 text-center">
                  <div class="auth-link text-white">Belum punya akun? Silahkan <router-link class="auth-link text-white font-weight-medium" :to="{ name: 'register' }">daftar</router-link></div>
                </div>
              </form>
            </form>
          </div>
        </div>
      </div>
    </div>
  </section>

</template>

<script lang="js">
export default {
  name: 'login',
  data() {
    return {
      error: {},
      email: null,
      phone: null,
      password: null,
      dismissSecs: 3,
      dismissCountDown: 0
    }
  },
  mounted() {
    if (localStorage.userActivated != undefined) {
      this.dismissCountDown = this.dismissSecs
      localStorage.removeItem('userActivated')
    }
  },
  methods: {
    countDownChanged (dismissCountDown) {
      this.dismissCountDown = dismissCountDown
    },
    login() {
      let self = this
      let data = new URLSearchParams()
      data.append('password', self.password)

      if (self.email)
        data.append('email', self.email)
      else
        data.append('phone', self.phone)

      this.$http.post('login', data)
        .then(resp => {
          self.error = {}
          const response = resp.data
          localStorage.token = response.token
          localStorage.userName = response.user.name
          localStorage.userEmail = response.user.email
          localStorage.company = response.company.name
          localStorage.jobTitle = response.jobtitle[0].name
          self.$http.defaults.headers.common['Authorization'] = 'Bearer ' + localStorage.token
          self.$router.push({ name: 'dashboard' })
        })
        .catch(err => {
          console.log(err)
          const errorResponse = err.response.data

          if ('error' in errorResponse && errorResponse.error == 'Unauthorized')
            self.error = { password: ['Data tidak ditemukan!'] }
          else
            self.error = errorResponse.errors
        })
    }
  }
}
</script>

<style scoped lang="scss">
  .alert {
    position: absolute;
    width: 100%;
  }
</style>
