<template>
  <div class="user" :class="{victim: user.isVictim, show: isShown}" @click="showHandler">
    <div class="user__images">
      <div class="self">
        <p>User {{ user.id }}</p>
        <img :src="`https://avatars.dicebear.com/api/micah/${user.id}.svg`" class="avatar" alt="">
      </div>
      <div class="target" v-if="user.target >= 0 && isShown">
        <img :src="`https://avatars.dicebear.com/api/micah/${user.target}.svg`" class="avatar avatar--target" alt="">
        <p>{{ user.target ? 'User ' + user.target : 'Вы' }}</p>
      </div>
    </div>
    <div class="observers">
      <div v-if="isShown" class="observers__list">
        <img v-for="obs in observers" :key="obs.id"
             :src="`https://avatars.dicebear.com/api/micah/${obs.id}.svg`" class="avatar" alt="">
      </div>

    </div>
    <div class="toolbar">
      <button v-if="isButtonShown" @click.stop="killHandler" type="button" class="button">Убить</button>
    </div>
  </div>
</template>

<script>
  export default {
    name: "UserCard",
    props: {
      user: Object,
      users: Object,
      isGameOver: Boolean,
      murdererTarget: Number,
      changeTime: Number,
      killMode: String
    },
    data: () => ({
      timer: null,
    }),
    computed: {
      isShown() {
        return this.user.id === this.murdererTarget
      },
      observers() {
        return Object.values(this.users).filter(user => user.target === this.user.id)
      },
      isButtonShown() {
        if (this.killMode === 'sight') {
          return this.user.target === 0 && this.isShown
        }
        return this.isShown;
      }
    },
    methods: {
      init() {
        this.$emit('set-target', this.user.id);
        this.intervalHandler();
      },
      intervalHandler() {
        let randInterval = Math.floor(+(1 + Math.random() * (this.changeTime - 1)).toFixed(3) * 1000);
        this.timer = setTimeout(() => {
          this.$emit('set-target', this.user.id);
          this.destroyTimer();
          this.intervalHandler();
        }, randInterval)
      },
      destroyTimer() {
        clearTimeout(this.timer);
      },
      killHandler() {
        this.destroyTimer();
        this.$emit('kill-user', this.user.id);
      },
      showHandler() {
        this.$emit('show-user', this.user.id);
      }
    },
    mounted() {
      this.init()
    },
    beforeDestroy() {
      this.destroyTimer();
    }
  }
</script>

<style lang="stylus" scoped>
  .target
    margin 38px 0 0

  .user
    margin 20px
    padding 15px 0
    background lightgrey
    width 250px
    height 300px
    text-align center
    display flex
    flex-direction column
    align-items center
    justify-content space-between
    cursor pointer
    user-select none

    &.victim
      opacity 0.5
      pointer-events none

    &.show
      background lightgreen

  .user__images
    flex-grow 1
    display flex
    justify-content center

  .observers
    display flex
    align-items center
    justify-content center
    width 100%
    height 30px
    margin 0 0 20px
    .avatar
      width 30px
      height 30px
      margin 5px

  .toolbar
    height 38px
    width 100%
</style>