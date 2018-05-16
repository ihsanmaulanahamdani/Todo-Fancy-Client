<template>
  <div>
    <button class="btn btn-outline-success" data-toggle="modal" data-target="#modalLogin"><i class="fas fa-sign-in-alt"></i> LOGIN</button>
    <button class="btn btn-outline-primary" data-toggle="modal" data-target="#modalRegister">REGISTER</button>
    <div>
      <div class="modal fade" id="modalLogin" tabindex="-1" role="dialog" aria-labelledby="exampleModalLabel" aria-hidden="true">
        <div class="modal-dialog" role="document">
          <div class="modal-content">
            <div class="modal-header">
              <h5 class="modal-title" id="exampleModalLabel"><i class="fas fa-user"></i> Login</h5>
              <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                <span aria-hidden="true">&times;</span>
              </button>
            </div>
            <div class="modal-body">
              <form>
                <div class="form-group">
                  <label for="inputEmail">Email</label>
                  <input type="text" class="form-control" v-model="email" placeholder="Email">
                </div>
                <div class="form-group">
                  <label for="inputPassword">Password</label>
                  <input type="password" class="form-control" v-model="password" placeholder="Password">
                </div>
              </form>
            </div>
            <div class="modal-footer">
              <button type="button" class="btn btn-outline-success" @click="loginUser">LOGIN</button>
              <facebook-login class="button"
                appId="1957935024247708"
                @login="getUserData">
              </facebook-login>
            </div>
          </div>
        </div>
      </div>

      <div class="modal fade" id="modalRegister" tabindex="-1" role="dialog" aria-labelledby="exampleModalLabel" aria-hidden="true">
        <div class="modal-dialog" role="document">
          <div class="modal-content">
            <div class="modal-header">
              <h5 class="modal-title" id="exampleModalLabel"><i class="fas fa-user"></i> Register</h5>
              <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                <span aria-hidden="true">&times;</span>
              </button>
            </div>
            <div class="modal-body">
              <form>
                <div class="form-group">
                  <label for="inputFullName">Full Name</label>
                  <input type="text" class="form-control" v-model="full_name" placeholder="Full Name">
                </div>
                <div class="form-group">
                  <label for="inputUsername">Username</label>
                  <input type="text" class="form-control" v-model="username" placeholder="Username">
                </div>
                <div class="form-group">
                  <label for="inputEmail">Email</label>
                  <input type="text" class="form-control" v-model="email" placeholder="Email">
                </div>
                <div class="form-group">
                  <label for="inputPassword">Password</label>
                  <input type="password" class="form-control" v-model="password" placeholder="Password">
                </div>
              </form>
            </div>
            <div class="modal-footer">
              <button type="button" class="btn btn-outline-primary" @click="registerUser">REGISTER</button>
            </div>
          </div>
        </div>
      </div>
    </div>
    </div>
</template>

<script>
  import axios from 'axios'
  import facebookLogin from 'facebook-login-vuejs'

  export default {
    data () {
      return {
        full_name: '',
        username: '',
        email: '',
        password: ''
      }
    },
    components: {
      facebookLogin
    },
    methods: {
      registerUser () {
        let self = this

        axios
          .post('https://safe-plains-12311.herokuapp.com/register', {
            full_name: self.full_name,
            username: self.username,
            email: self.email,
            password: self.password
          })
          .then(response => {
            if (response.data.user.email) {
              window.location.reload()
            }
          })
          .catch(err => {
            console.log(err)
          })
      },

      loginUser () {
        let self = this

        axios
          .post('https://safe-plains-12311.herokuapp.com/login', {
            email: self.email,
            password: self.password
          })
          .then(response => {
            if (response.data.token) {
              localStorage.setItem('token', response.data.token)
              localStorage.setItem('id', response.data.user.id)
              localStorage.setItem('username', response.data.user.username)
              localStorage.setItem('role', response.data.user.role)
              localStorage.setItem('email', response.data.user.email)
              
              window.location.reload()
            }
          })
          .catch(err => {
            console.log(err)
          })
      },

      getUserData () {
        let self = this

        FB.getLoginStatus(function (response) {
          if (response.status === 'connected') {
            FB.api('/me', { fields: [ 'name', 'email' ] }, function(data) {
              axios
                .post('https://safe-plains-12311.herokuapp.com/loginfacebook', {
                  full_name: data.name,
                  username: data.name,
                  email: data.email,
                  password: 'Abc12345'
                })
                .then(result => {
                  console.log(result);
                  if (result.data.token) {
                    localStorage.setItem('token', result.data.token)
                    localStorage.setItem('id', result.data.user.id)
                    localStorage.setItem('username', result.data.user.username)
                    localStorage.setItem('role', result.data.user.role)
                    localStorage.setItem('email', result.data.user.email)
                    
                    window.location.reload()
                  }
                })
                .catch(err => {
                  console.log(err)
                })
            })
          }
        })
      }
    }
}
</script>
