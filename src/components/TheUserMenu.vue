<template>
  <q-list
    :class='compactMode ? "flex row no-wrap justify-around items-center full-width" : "text-right"'
    :dense='compactMode'
  >
    <q-item
      v-if='$store.state.keys.pub'
      class='col no-padding flex items-center'
      :class="compactMode ? 'justify-center' : 'justify-end'"
      clickable
      @click='toProfile($store.state.keys.pub)'
    >
      <BaseUserCard
        v-if='!compactMode'
        :pubkey='$store.state.keys.pub'
        :align-right='true'
        :wrap='true'
        class='gt-sm'
      />
      <BaseUserAvatar
        v-if='!compactMode'
        :pubkey="$store.state.keys.pub"
        :align-right='true'
        :show-verified='true'
        size='lg'
        class='q-mr-sm lt-md'
      />
      <BaseUserAvatar
        v-if='$store.state.keys.pub && compactMode'
        :pubkey="$store.state.keys.pub"
        :align-right='true'
        size='1.5rem'
      />
    </q-item>
    <!-- <q-separator v-if='!compactMode' color='accent' spaced/> -->
    <div v-if='!compactMode' style='min-height: 1rem;'/>
    <q-item
      v-for='item in filteredUserMenuItems'
      clickable
      class='menu-item'
      :dense='compactMode'
      :style='compactMode ? "" : "min-height: 2.75rem;"'
      :active="$route.name === item.title"
      active-class=''
      @click='(event) => handleClick(event, item)'
      :key='item.title'
      :class="($route.name === item.title ? 'menu-item-active text-accent ' : '') +
        (compactMode ? 'no-margin no-padding col' : 'self-end q-px-none')"
    >
      <q-item-section v-if='!compactMode' class='gt-sm text-uppercase' style='font-size: 1rem;'>
        <div>
          {{ $t(item.title) }}
        </div>
      </q-item-section>

      <q-item-section
        v-if="item.icon && !compactMode"
        avatar
        class="relative-position no-padding"
        style="min-width: 2.5rem;"
      >
        <q-icon outline :name="item.icon"/>
        <!-- <q-badge
          v-if="item.badge && $store.getters[item.badge]"
          color="secondary"
          floating
          class='q-mr-md text-bold'
          outline
        >
          {{ $store.getters[item.badge] }}
        </q-badge> -->
        <q-badge
          v-if="item.title === 'notifications' && $store.state.unreadNotifications"
          color="secondary"
          floating
          class='q-mr-md text-bold'
          outline
        >
          {{ $store.state.unreadNotifications }}
        </q-badge>
        <q-badge
          v-if="item.title === 'messages' && $store.getters.unreadChats"
          color="secondary"
          floating
          class='q-mr-md text-bold'
          outline
        >
          {{ $store.getters.unreadChats }}
        </q-badge>
      </q-item-section>

      <div
        v-if="item.icon && compactMode"
        avatar
        class="q-mx-auto no-padding relative-position"
      >
        <q-icon :name="item.icon" size='sm'/>
        <q-badge
          v-if="item.badge && $store.getters[item.badge]"
          color="secondary"
          floating
          rounded
          style='margin-top: .2rem; margin-left: .1rem;'
        />
      </div>
    </q-item>
    <!-- <q-separator v-if='!compactMode' color='accent' spaced/> -->
    <div v-if='!compactMode' style='min-height: 1rem;'/>
      <div
        color="primary"
        class='flex '
        :class='compactMode ? "col justify-center" : "q-my-md justify-end"'
      >
        <BaseButtonPost
          v-if='$store.state.keys.pub'
          :is-open='posting'
          :verbose='true'
          @open="$emit('toggle-post-entry')"
          :outline='!compactMode'
          :flat='compactMode'
          color='primary'
          :size='compactMode ? "sm" : "lg"'
          :class='compactMode ? "" : "q-px-sm"'
        />
        <BaseButtonSetUser
          v-if='!$store.state.keys.pub'
          @click='(event) => handleClick(event, {title: "set-user"})'
          :verbose='true'
          :outline='!compactMode'
          :flat='compactMode'
          color='primary'
          :size='compactMode ? "sm" : "lg"'
          :class='compactMode ? "" : "q-px-sm"'
        />
      </div>
    <!-- <q-dialog
      v-model='post'
      seamless
      position='bottom'
      transition-show='slide-up'
      transition-hide='slide-down'
    >
      <q-card unelevated class='flex column no-wrap post-entry relative-position'
      >
        <div class='absolute-top-right'>
          <q-btn class='z-top' icon="close" flat dense v-close-popup/>
        </div>
        <BasePostEntry class='q-pa-md' @sent='post = false'/>
        <div class='compact-user-menu-space'/>
      </q-card>
    </q-dialog> -->
    <!-- <input type='color' /> -->
  </q-list>
</template>

<script>
import { defineComponent } from 'vue'
import helpersMixin from '../utils/mixin'
import BaseUserCard from 'components/BaseUserCard.vue'
import BaseButtonPost from 'components/BaseButtonPost.vue'
import BaseButtonSetUser from 'components/BaseButtonSetUser.vue'

export default defineComponent({
  name: 'TheUserMenu',
  mixins: [helpersMixin],
  emits: ['toggle-post-entry', 'scroll-to-rect', 'set-user'],
  props: {
    iconMode: {
      type: Boolean,
      default: false
    },
    compactMode: {
      type: Boolean,
      default: false,
    },
    showCompactModeItems: {
      type: Boolean,
      default: false
    },
    posting: {
      type: Boolean,
      required: true
    }
  },

  data() {
    return {
      // isOpen: this.posting,
      // position: 'bottom',
      userMenuItems: [
        {
          title: 'feed',
          icon: 'newspaper',
          to: '/',
          match: 'feed',
        },
        {
          title: 'notifications',
          badge: 'unreadNotifications',
          icon: 'notifications',
          to: '/notifications',
          match: 'notifications',
        },
        {
          title: 'messages',
          badge: 'unreadChats',
          icon: 'mail_lock',
          to: '/messages/inbox',
          match: 'messages',
        },
        {
          title: 'settings',
          icon: 'settings',
          to: '/settings',
          match: 'settings',
        },
        {
          title: 'follow',
          icon: 'search',
          to: '/follow',
          match: 'follow',
          compactMenuOnly: true,
        },
      ],
    }
  },

  components: {
    BaseUserCard,
    BaseButtonPost,
    BaseButtonSetUser,
  },

  computed: {
    filteredUserMenuItems() {
      if (!this.$store.state.keys.pub) return this.userMenuItems.filter(item => item.title === 'feed')
      if (!this.compactMode && !this.showCompactModeItems) return this.userMenuItems.filter(item => !item.compactMenuOnly)
      return this.userMenuItems
    },
    // isOpen() {
    //   return this.posting
    // }
  },

  methods: {
    handleClick(event, item) {
      if (item.title === 'set-user') {
        this.$emit('scroll-to-rect', {top: 0})
        this.$emit('set-user')
        return
      }
      if (item.title === 'feed' && this.$route.name === 'feed') {
        event.preventDefault()
        this.$emit('scroll-to-rect', {top: 0})
      } else this.$router.push(item.to)
    }
  }
})
</script>

<style lang='css' scoped>
.q-item {
  transition: all .2s ease-in-out
}
.menu-item {
 letter-spacing: .1rem;
 opacity: .7;
}
.menu-item:hover {
  opacity: 1;
  font-weight: bold;
}
.menu-item-active {
  color: var(--q-accent);
  opacity: 1;
  font-weight: bold;
}
.q-dialog .post-entry {
  width: 600px;
  overflow: auto;
}
.compact-user-menu-space {
  height: 2rem;
}

@media screen and (min-width: 700px) {
  .compact-user-menu-space {
    display: none;
  }
}
</style>
