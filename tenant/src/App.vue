<template>
  <div id="app">
    <header class="rf-header">
      <div class="rf-container">
        <MyHeader
          :logged-in="status.loggedIn"
          @on-create-tenant="onCreateTenant"
          @on-create-owner="onCreateOwner"
          @on-logout="onLogout"
          @on-change-lang="changeLang"
          :lang="getLang()"
        />
        <Menu :user="user" />
      </div>
    </header>
    <article class="page">
      <router-view />
    </article>
    <MyFooter />
  </div>
</template>

<script lang="ts">
import { Vue, Component } from "vue-property-decorator";
import MyHeader from "df-shared/src/Header/Header.vue";
import MyFooter from "df-shared/src/Footer/Footer.vue";
import Menu from "@/components/Menu.vue";
import { mapState } from "vuex";
import i18n from "./i18n";

@Component({
  components: {
    MyHeader,
    MyFooter,
    Menu
  },
  computed: {
    ...mapState({
      user: "user",
      status: "status"
    })
  },
  mounted: function() {
    const localScript = document.createElement("script");
    localScript.setAttribute("src", "/js/all.js");
    document.head.appendChild(localScript);
  }
})
export default class App extends Vue {
  OWNER_URL = `//${process.env.VUE_APP_OWNER_URL}`;

  onLogin() {
    this.$router.push("/login");
  }
  onLogout() {
    this.$store.dispatch("logout").then(
      () => {
        console.log("logged out !");
      },
      error => {
        console.dir(error);
      }
    );
  }
  onCreateTenant() {
    this.$router.push("/signup");
  }

  onCreateOwner() {
    window.location.href = this.OWNER_URL;
  }

  getLang() {
    return i18n.locale;
  }

  changeLang() {
    const lang = i18n.locale === "fr" ? "en" : "fr";
    this.$store.dispatch("setLang", lang);
  }
}
</script>

<style lang="scss">
@import "../node_modules/df-shared/src/scss/_main.scss";

#app {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
}

.page {
  flex: auto;
  min-height: 300px;
  display: flex;
  align-items: stretch;
}

</style>

<i18n>
{
"en": {
"home": "Home"
},
"fr": {
"home": "Home"
}
}
</i18n>
