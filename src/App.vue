<template>
  <div id="app">
    <div class="header">
      <div>
        <img :src="`https://avatars.dicebear.com/api/micah/0.svg`" class="user__avatar" alt="">
        <p>Убийца (вы)</p>
      </div>
      <p>Незамеченных убийств: {{ victimCount }}</p>
    </div>

    <div v-if="isGameActive" class="list">
      <user-card v-for="user in users"
                 :key="user.id"
                 :is-game-over="isGameOver"
                 :user="user"
                 :users="users"
                 :change-time="changeTime"
                 :murderer-target="murdererTarget"
                 :kill-mode="killMode"
                 @set-target="setUserTarget"
                 @kill-user="killUser"
                 @show-user="showUser"></user-card>
    </div>

    <div v-else class="modal" >
      <div class="modal__blur"></div>
      <div class="modal__content">
        <template v-if="isGameOver">
          <h3>Убийца пойман на месте преступления</h3>
          <p>Незамеченных убийств: {{ victimCount - 1 }} </p>
          <div class="spectators">
            <div v-for="spectator in spectators" class="spectators__item">
              <img :src="`https://avatars.dicebear.com/api/micah/${spectator.id}.svg`" class="avatar" alt="">
              <p>User {{ spectator.id }}</p>
            </div>
          </div>
        </template>
        <template v-if="isGameWin">
          <h3>Вы победили!</h3>
          <p>Убийства остались незамеченными</p>
          <div class="spectators">
            <div class="spectators__item">
              <img :src="`https://avatars.dicebear.com/api/micah/0.svg`" class="avatar" alt="">
              <p>Победитель</p>
            </div>
          </div>
        </template>
        <button class="button" @click="startNewGame">Начать заново</button>
      </div>
    </div>
  </div>
</template>

<script>
  import UserCard from "./components/UserCard";
  export default {
    name: 'App',
    components: { UserCard },
    data: () => ({
      users: {},
      usersCount: 8,
      changeTime: 3,  //seconds
      activeUsersIds: [],
      spectators: [],
      murdererTarget: null,
      killMode: 'sight'
    }),
    mounted() {
      this.startNewGame()
    },
    computed: {
      victimCount() {
        return this.usersCount - this.activeUsersIds.length + 1
      },
      isGameWin() {
        return !this.isGameOver && this.activeUsersIds.length === 2
      },
      isGameOver() {
        return !!this.spectators.length
      },
      isGameActive() {
        return !this.isGameOver && !this.isGameWin
      },
    },
    methods: {
      startNewGame() {
        this.spectators = [];
        this.checkUrlParams();
        this.initUsers();
      },
      checkUrlParams() {
        let url = new URLSearchParams(window.location.search);
        if(url.get('users')) this.usersCount = +url.get('users');
        if(url.get('time')) this.changeTime = +url.get('time');
        if(url.get('kill')) this.killMode = url.get('kill');
      },
      initUsers() {
        let users = {};
        for (let i = 1; i <= this.usersCount; i++) {
          users[i] = {id: i, target: 0, isVictim: false}
        }
        this.activeUsersIds = [0, ...Object.keys(users).map(Number)];
        this.users = users;
      },
      chooseRandomUserId(selfId) {
        let index = Math.floor(Math.random() * this.activeUsersIds.length);
        let targetId = this.activeUsersIds[index];
        return (targetId === selfId) ? this.chooseRandomUserId(selfId) : targetId;
      },
      setUserTarget(userId) {
        this.users[userId].target = this.chooseRandomUserId(userId);
      },
      showUser(userId) {
        this.murdererTarget = userId;
      },
      killUser(userId) {
        let victimId = this.activeUsersIds.findIndex(id => id === userId);
        this.murdererTarget = null;
        this.users[userId].isVictim = true;
        this.users[userId].target = -1;
        this.activeUsersIds.splice(victimId, 1);
        this.spectators = Object.values(this.users).filter(user => user.target === 0);
      }
    }
  }
</script>

<style lang="stylus">
  *
    box-sizing border-box

  #app
    font-family Avenir, Helvetica, Arial, sans-serif
    -webkit-font-smoothing antialiased
    -moz-osx-font-smoothing grayscale
    color #2c3e50
    margin 20px auto
    max-width: 1160px

  p
    margin 10px 0

  .header
    display flex
    align-items flex-end
    justify-content space-between
    padding 0 20px
    font-weight 600

  .avatar
    width 100px
    height 100px
    pointer-events none
    &--target
      transform scale(-1, 1)

  .button
    background crimson
    color white
    border 0
    padding 10px 40px
    cursor pointer
    font-size 16px

  .list
    display flex
    align-items center
    flex-wrap wrap

  .modal
    position fixed
    top 0
    left 0
    display flex
    align-items center
    justify-content center
    width 100%
    height 100%
    z-index 999

  .modal__blur
    position absolute
    width 100%
    height 100%
    background white
    z-index: -1;

  .modal__content
    width 530px
    min-height 350px
    padding 20px
    background lightgreen
    text-align center

  .spectators
    margin 30px 0 10px
    display flex
    align-items center
    justify-content center
    flex-wrap wrap

  .spectators__item
    max-width 100px
    margin 0 10px
</style>
