<template>
  <div class="navbar-container d-flex content align-items-center">

    <!-- Nav Menu Toggler -->
    <ul class="nav navbar-nav d-lg-none">
      <li class="nav-item">
        <b-link
          class="nav-link"
          @click="toggleVerticalMenuActive"
        >
          <b-avatar
            v-if="selected_chain && selected_chain.logo"
            variant="transparent"
            rounded
            size="21"
            :src="selected_chain.logo"
            class="badge-minimal"
          />
          <feather-icon
            v-else
            icon="MenuIcon"
            size="21"
          />
        </b-link>
      </li>
    </ul>

    <!-- Left Col -->
    <div class="bookmark-wrapper align-items-center flex-grow-1 d-none d-lg-flex">
      <b-media
        v-if="selected_chain"
        no-body
      >
        <b-media-aside class="mr-75">
          <b-link
            class="nav-link"
            @click="toggleVerticalMenuActive"
          >
            <b-avatar
              v-b-tooltip.hover.bottom="tips"
              variant="transparent"
              badge
              rounded
              size="42"
              :src="selected_chain.logo"
              class="badge-minimal"
              :badge-variant="variant"
            /></b-link>
        </b-media-aside>
        <b-media-body class="my-auto">
          <h6 class="mb-0 text-uppercase">
            {{ selected_chain.chain_name }}
          </h6>
          <small v-b-tooltip.hover.bottom="'Data Provider'">{{ selected_chain.api }} ({{ selected_chain.sdk_version || '-' }})</small>
        </b-media-body>
      </b-media>
    </div>

    <!-- <dark-Toggler class="d-none d-lg-block" /> -->
    <!-- Right Col -->
    <b-navbar-nav class="nav align-items-center ml-auto">
      <dark-Toggler class="d-none d-lg-block" />
      <search-bar />
      <locale />
      <b-dropdown
        class="ml-1"
        variant="link"
        no-caret
        toggle-class="p-0"
        right
      >

        <template #button-content>
          <b-button
            v-ripple.400="'rgba(255, 255, 255, 0.15)'"
            variant="primary"
            class="btn-icon"
          >
            <feather-icon icon="KeyIcon" />
            {{ walletName }}
          </b-button>
        </template>

        <b-dropdown-item
          :to="{ name: 'portfolio' }"
          class="d-none"
        >
          <feather-icon
            icon="PieChartIcon"
            size="16"
          />
          <span class="align-middle ml-50">Portofolio</span>
        </b-dropdown-item>

        <b-dropdown-item :to="{ name: 'accounts' }">
          <feather-icon
            icon="KeyIcon"
            size="16"
          />
          <span class="align-middle ml-50">Accounts</span>
        </b-dropdown-item>

        <b-dropdown-item :to="{ name: 'delegations' }">
          <feather-icon
            icon="BookOpenIcon"
            size="16"
          />
          <span class="align-middle ml-50">My Delegations</span>
        </b-dropdown-item>

        <b-dropdown-item :to="`/${selected_chain.chain_name}/uptime/my`">
          <feather-icon
            icon="AirplayIcon"
            size="16"
          />
          <span class="align-middle ml-50">My Validators</span>
        </b-dropdown-item>
        <b-dropdown-item :to="`/wallet/transactions`">
          <feather-icon
            icon="LayersIcon"
            size="16"
          />
          <span class="align-middle ml-50">My Transactions</span>
        </b-dropdown-item>
      </b-dropdown>
    </b-navbar-nav>
  </div>
</template>

<script>
import {
  BLink, BNavbarNav, BMedia, BMediaAside, BAvatar, BMediaBody, VBTooltip, BButton,
  BDropdown, BDropdownItem,
} from 'bootstrap-vue'
import Ripple from 'vue-ripple-directive'
import DarkToggler from '@core/layouts/components/app-navbar/components/DarkToggler.vue'
import Locale from '@core/layouts/components/app-navbar/components/Locale.vue'
import SearchBar from '@core/layouts/components/app-navbar/components/SearchBar.vue'
// import CartDropdown from '@core/layouts/components/app-navbar/components/CartDropdown.vue'
import store from '@/store'
import { getLocalAccounts, timeIn, toDay } from '@/libs/utils'
// import UserDropdown from '@core/layouts/components/app-navbar/components/UserDropdown.vue'

export default {
  components: {
    BLink,
    BNavbarNav,
    BAvatar,
    BMedia,
    BMediaAside,
    BMediaBody,
    BButton,
    BDropdown,
    BDropdownItem,

    // Navbar Components
    DarkToggler,
    Locale,
    SearchBar,
    // CartDropdown,
    // UserDropdown,
  },
  directives: {
    'b-tooltip': VBTooltip,
    Ripple,
  },
  props: {
    toggleVerticalMenuActive: {
      type: Function,
      default: () => {},
    },
  },
  data() {
    return {
      variant: 'success',
      tips: 'Synced',
    }
  },
  computed: {
    walletName() {
      const key = this.$store?.state?.chains?.defaultWallet
      return key || 'Wallet'
    },
    selected_chain() {
      this.block()
      return store.state.chains.selected
    },
    chainVariant() {
      return this.variant
    },
  },
  mounted() {
    const accounts = Object.keys(getLocalAccounts() || {})
    if (!this.$store.state.chains.defaultWallet && accounts.length > 0) {
      this.$store.commit('setDefaultWallet', accounts[0])
    }
  },
  methods: {
    block() {
      store.commit('setHeight', 0)
      this.$http.getLatestBlock().then(block => {
        store.commit('setHeight', Number(block.block.header.height))
        if (timeIn(block.block.header.time, 1, 'm')) {
          this.variant = 'danger'
          this.tips = `Halted ${toDay(block.block.header.time, 'from')}, Height: ${store.state.chains.height} `
        } else {
          this.variant = 'success'
          this.tips = 'Synced'
        }
      })
    },
  },
}
</script>
