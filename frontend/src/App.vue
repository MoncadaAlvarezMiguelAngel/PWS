<template>
  <v-app>
    
    <v-navigation-drawer expand-on-hover rail permanent v-model="showNavigation">
      
      <v-list density="compact" nav>
        <template v-for="item in navigation" :key="item.title">
          <v-list-item :href="item.href" :prepend-icon="item.icon" :title="item.title" exact v-show="checkIfInRole(user, item.roles)"/>
        </template>
      </v-list>

      <v-divider></v-divider>

      <v-list v-if="user.username">
    <v-list-item>
      <v-avatar>
        <v-img v-for="role in user.roles" :key="role" :src="getRoleIcon(role)" />
      </v-avatar>
      <v-list-item-content >
            <template v-if="showNavigation">
              <v-list-item
          :title="user.username"
        >
        </v-list-item>
            </template>
      </v-list-item-content>
    </v-list-item>
  </v-list>

      <v-list density="compact" nav>
        <v-list-item key="Login" @click="prepareToLogin" prepend-icon="mdi-login" title="Login" exact v-if="!user.username"/>
        <v-list-item key="Logout" @click="logoutConfirmation = true" prepend-icon="mdi-logout" title="Logout" exact v-if="user.username"/>
      </v-list>

    </v-navigation-drawer>

    <v-main>
      <router-view :user="user"></router-view>
    </v-main>

    <v-dialog v-model="loginDialog" width="25em">
      <Login @cancel="loginDialog = false" @loginFailed="loginDialog = false" @loginSuccess="login"/>
    </v-dialog>

    <v-dialog v-model="logoutConfirmation" width="auto">
      <ConfirmationDialog :question="'Are you sure to logout?'" @ok="logout" @cancel="logoutConfirmation = false"/>
    </v-dialog>

  </v-app>
</template>

<script>
import common from './mixins/common'
import Login from './components/Login.vue'
import ConfirmationDialog from './components/ConfirmationDialog.vue'

export default {
  name: 'App',
  components: { Login, ConfirmationDialog },
  mixins: [ common ],
  data() {
    return {
      navigation: [
          { title: 'Dashboard', icon: 'mdi-view-dashboard', href: '#/' },
          { title: 'Persons', icon: 'mdi-account-multiple', href: '#/persons', roles: [ 0, 1 ] },
          { title: 'Projects', icon: 'mdi-sitemap-outline', href: '#/projects' },
          { title: 'Tasks', icon:'mdi-calendar-check-outline' , href: '#/tasks' , roles: [ 0, 1 ] },
          { title: 'Map', icon: 'mdi-map', href: '#/map' },
          { title: 'Chat', icon: 'mdi-chat-processing-outline', href: '#/chat' },

      ],
      showNavigation: false,
      user: {},
      loginDialog: false,
      logoutConfirmation: false
    }
  },
  methods: {
    getRoleIcon(role) {
      switch (role) {
        case 0:
          return require('./assets/adminicon.png');
        case 1:
          return require('./assets/user.png');
        // Puedes añadir más casos según los roles que tengas
        default:
          return require('./assets/vue.png');
      }
    },
    setUser(data) {
      this.loginDialog = false
      Object.keys(this.user).forEach(key => delete this.user[key])
      Object.assign(this.user, data)
      this.$router.push('/')
      this.showNavigation = true
    },
    logout() {
      this.logoutConfirmation = false
      this.showNavigation = false
      fetch('/auth', { method: 'DELETE' })
      .then(res => res.json())
      .then(data => this.setUser(data))
    },
    prepareToLogin() {
      this.showNavigation = false
      this.loginDialog = true
    },
    login(data) {
      this.loginDialog = false
      this.setUser(data)
    }
  },
  mounted() {
    fetch('/auth', { method: 'GET' })
    .then(res => res.json())
    .then(data => this.setUser(data))
  }
}
</script>

<style>
#app {
  font-family: Roboto, Helvetica, Arial, sans-serif;
  margin: 10px 30px;
}
</style>