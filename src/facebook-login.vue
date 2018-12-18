<template>
  <div @click="buttonClicked">
    <slot></slot>
  </div>
</template>

<script>
// Helpers
import {
  loadFbSdk,
  getFbLoginStatus,
  fbLogout,
  fbLogin
} from './helpers.js'

// Resources
import icon from './Facebook.png'

export default {
  name: 'facebook-login',
  props: {
    appId: {
      type: String,
      required: true
    },
    version: {
      type: String,
      default: 'v2.10'
    },
    logoutLabel: {
      type: String,
      default: 'Log out from Facebook'
    },
    loginLabel: {
      type: String,
      default: 'Log in to Facebook'
    },
    loginOptions: {
      type: Object,
      default: function() {
        return {
          scope: 'email'
        }
      }
    }
  },
  data() {
    return {
      isWorking: false,
      isConnected: false,
      icon
    }
  },
  mounted() {
    this.isWorking = true
    loadFbSdk(this.appId, this.version)
      .then(getFbLoginStatus)
      .then(response => {
        if (response.status === 'connected') {
          this.isConnected = true
        }
        this.isWorking = false
        /** Event `get-initial-status` to be deprecated in next major version! */
        this.$emit('get-initial-status', response)
        this.$emit('sdk-loaded', {
          isConnected: this.isConnected,
          FB: window.FB
        })
      })
  },
  computed: {
    getButtonText() {
      switch (this.isConnected) {
        case true:
          return this.logoutLabel
        case false:
          return this.loginLabel
        default:
          return 'this is default'
      }
    }
  },
  methods: {
    buttonClicked() {
      this.$emit('click')
      if (this.isConnected) {
        this.logout()
      } else {
        this.login()
      }
    },
    login() {
      this.isWorking = true
      fbLogin(this.loginOptions)
        .then(response => {
          if (response.status === 'connected') {
            this.isConnected = true
          } else {
            this.isConnected = false
          }
          this.isWorking = false
          this.$emit('login', {
            response,
            FB: window.FB
          })
        })
    },
    logout() {
      this.isWorking = true
      fbLogout()
        .then(response => {
          this.isWorking = false
          this.isConnected = false
          this.$emit('logout', response)
        }
        )
    }
  }
}
</script>
