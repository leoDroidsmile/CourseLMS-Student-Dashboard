<script>
import simplebar from "simplebar-vue";
import i18n from "../i18n";
import axios from 'axios';

export default {
  data() {
    return {
      languages: [
        {
          flag: require("@/assets/images/flags/us.jpg"),
          language: "en",
          title: "English"
        },
        {
          flag: require("@/assets/images/flags/french.jpg"),
          language: "fr",
          title: "French"
        },
        {
          flag: require("@/assets/images/flags/spain.jpg"),
          language: "es",
          title: "spanish"
        },
        {
          flag: require("@/assets/images/flags/chaina.png"),
          language: "zh",
          title: "Chinese"
        },
        {
          flag: require("@/assets/images/flags/arabic.png"),
          language: "ar",
          title: "Arabic"
        }
      ],
      current_language: "en",
      search : '',
      notifications : []
    };
  },
  components: { simplebar },
  methods: {
    toggleMenu() {
      this.$parent.toggleMenu();
    },
    initFullScreen() {
      document.body.classList.toggle("fullscreen-enable");
      if (
        !document.fullscreenElement &&
        /* alternative standard method */ !document.mozFullScreenElement &&
        !document.webkitFullscreenElement
      ) {
        // current working methods
        if (document.documentElement.requestFullscreen) {
          document.documentElement.requestFullscreen();
        } else if (document.documentElement.mozRequestFullScreen) {
          document.documentElement.mozRequestFullScreen();
        } else if (document.documentElement.webkitRequestFullscreen) {
          document.documentElement.webkitRequestFullscreen(
            Element.ALLOW_KEYBOARD_INPUT
          );
        }
      } else {
        if (document.cancelFullScreen) {
          document.cancelFullScreen();
        } else if (document.mozCancelFullScreen) {
          document.mozCancelFullScreen();
        } else if (document.webkitCancelFullScreen) {
          document.webkitCancelFullScreen();
        }
      }
    },
    toggleRightSidebar() {
      this.$parent.toggleRightSidebar();
    },
    setLanguage(locale) {
      i18n.locale = locale;
      this.current_language = i18n.locale;
    },

    searchCourse(){
      this.$router.push({
        path: `/courses/0/${this.search}`
      });
    },
    
    downloadNotifications(){
      let config = {
        params: {
          is_read: false
        },
      }

      axios.get(this.$api_host + 'user/notifications', config)
      .then((response) => {
        this.notifications = response.data.notifications;
        this.notifications.forEach(element => {
          element.time = this.toUTCDate(element.created_at);
        });
      }).catch((error) => {
        if (error.response && error.response.status == 401){
          this.$router.push({ name: 'login' })  
        }
      });

      setTimeout(this.downloadNotifications, 5000);
    }
  },

  created(){
    if(this.$route.name  == "Courses" && this.$route.params.teacher == 0)
      this.search = this.$route.params.category;

    this.downloadNotifications();
  }
};
</script>

<template>
  <header id="page-topbar">
    <div class="navbar-header">
      <div class="d-flex">
        <!-- LOGO -->
        <div class="navbar-brand-box">
          <a href="" class="logo logo-dark">
            <span class="logo-sm">
              <img src="@/assets/images/logo-light.png" alt height="22" />
            </span>
            <span class="logo-lg">
              <img src="@/assets/images/logo-light.png" alt height="20" />
            </span>
          </a>

          <a href="" class="logo logo-light">
            <span class="logo-sm">
              <img src="@/assets/images/logo-light.png" alt height="22" />
            </span>
            <span class="logo-lg">
              <img src="@/assets/images/logo-light.png" alt height="20" />
            </span>
          </a>
        </div>

        <button
          @click="toggleMenu"
          type="button"
          class="btn btn-sm px-3 font-size-24 header-item waves-effect"
          id="vertical-menu-btn"
        >
          <i class="ri-menu-2-line align-middle"></i>
        </button>

        <!-- App Search-->
        <form class="app-search d-none d-lg-block" v-on:submit.prevent="searchCourse">
          <div class="position-relative">
            <input type="text" class="form-control" :placeholder="$t('navbar.search.text')" v-model='search'/>
            <span class="ri-search-line"></span>
          </div>
        </form>
      </div>

      <div class="d-flex">
        <!-- <div class="dropdown d-inline-block d-lg-none ml-2">
          <button
            type="button"
            class="btn header-item noti-icon waves-effect"
            id="page-header-search-dropdown"
            data-toggle="dropdown"
            aria-haspopup="true"
            aria-expanded="false"
          >
            <i class="ri-search-line"></i>
          </button>
          <div
            class="dropdown-menu dropdown-menu-lg dropdown-menu-right p-0"
            aria-labelledby="page-header-search-dropdown"
          >
            <form class="p-3">
              <div class="form-group m-0">
                <div class="input-group">
                  <input type="text" class="form-control" placeholder="Search ..." />
                  <div class="input-group-append">
                    <button class="btn btn-primary" type="submit">
                      <i class="ri-search-line"></i>
                    </button>
                  </div>
                </div>
              </div>
            </form>
          </div>
        </div> -->

        
        <b-dropdown
          right
          menu-class="dropdown-menu-lg p-0"
          toggle-class="header-item noti-icon"
          variant="black"
        >
          <template v-slot:button-content>
            <i class="ri-notification-3-line"></i>
            <span class="noti-dot" v-if='notifications.length'></span>
          </template>
          <div class="p-3">
            <div class="row align-items-center">
              <div class="col">
                <h6 class="m-0">{{ $t('navbar.dropdown.notification.text')}}</h6>
              </div>
              <div class="col-auto">
                <router-link to="/notifications" class="small">{{ $t('navbar.dropdown.notification.subtext')}}</router-link>
              </div>
            </div>
          </div>
          <simplebar style="max-height: 230px;">
            
            <a href class="text-reset notification-item" v-for='notification in notifications' :key="notification.id">
              <div class="media">
                <div class="media-body">
                  <h6 class="mt-0 mb-1">{{ notification.data["body"]}}</h6>
                  <div class="font-size-12 text-muted">
                    <p class="mb-0">
                      <i class="mdi mdi-clock-outline"></i>
                      {{ notification.time}}
                    </p>
                  </div>
                </div>
              </div>
            </a>
          </simplebar>
        </b-dropdown>
        
        <div class="dropdown d-inline-block">
          <button
            type="button"
            class="btn header-item noti-icon right-bar-toggle waves-effect toggle-right"
            @click="toggleRightSidebar"
          >
          </button>
        </div>
      </div>
    </div>
  </header>
</template>

<style lang="scss" scoped>
.notify-item {
  .active {
    color: #16181b;
    background-color: #f8f9fa;
  }
}

@media (min-width: 332px){
  .d-lg-block {
      display: block !important;
  }
}

#page-topbar > div > div:nth-child(1) > div > a.logo.logo-light > span.logo-lg > img{
  position:relative;
  right:16%;
}

@media screen and (max-width: 576px) {
  .navbar-brand-box{
    display:none;
  }
}

.logo-sm > img{
  display:none;
}
</style>

