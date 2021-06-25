<template>
  <section v-if="allMusic">
    <b-container>
      <b-row>
        <b-col>
          <h3 class="text-center mt-4 mb-4">Music Player</h3>
        </b-col>
      </b-row>
      <b-row>
        <b-col md="6">
          <img
            src="https://images.pexels.com/photos/3624281/pexels-photo-3624281.jpeg?auto=compress&cs=tinysrgb&dpr=1&w=500"
            class="image"
          />
          <b-card class="player_card">
            <h6>
              <b>{{this.current.title}} - {{this.current.artist}}</b>
            </h6>
            <div class="text-center controls">
              <svg class="bi bi-skip-backward" width="1em" height="1em" viewBox="0 0 16 16" fill="currentColor" xmlns="http://www.w3.org/2000/svg" @click="prev()">
                <path fill-rule="evenodd" d="M.5 3.5A.5.5 0 011 4v3.248l6.267-3.636c.52-.302 1.233.043 1.233.696v2.94l6.267-3.636c.52-.302 1.233.043 1.233.696v7.384c0 .653-.713.998-1.233.696L8.5 8.752v2.94c0 .653-.713.998-1.233.696L1 8.752V12a.5.5 0 01-1 0V4a.5.5 0 01.5-.5zm7 1.133L1.696 8 7.5 11.367V4.633zm7.5 0L9.196 8 15 11.367V4.633z" clip-rule="evenodd"/>
              </svg>
              <svg class="bi bi-play" width="1em" height="1em" viewBox="0 0 16 16" fill="currentColor" xmlns="http://www.w3.org/2000/svg" v-if="!isplaying" @click="play()">
                <path fill-rule="evenodd" d="M10.804 8L5 4.633v6.734L10.804 8zm.792-.696a.802.802 0 010 1.392l-6.363 3.692C4.713 12.69 4 12.345 4 11.692V4.308c0-.653.713-.998 1.233-.696l6.363 3.692z" clip-rule="evenodd"/>
              </svg>
              <svg class="bi bi-pause" width="1em" height="1em" viewBox="0 0 16 16" fill="currentColor" xmlns="http://www.w3.org/2000/svg" v-else @click="pause()">
                <path fill-rule="evenodd" d="M6 3.5a.5.5 0 01.5.5v8a.5.5 0 01-1 0V4a.5.5 0 01.5-.5zm4 0a.5.5 0 01.5.5v8a.5.5 0 01-1 0V4a.5.5 0 01.5-.5z" clip-rule="evenodd"/>
              </svg>
              <svg class="bi bi-skip-forward" width="1em" height="1em" viewBox="0 0 16 16" fill="currentColor" xmlns="http://www.w3.org/2000/svg" @click="next()">
                <path fill-rule="evenodd" d="M15.5 3.5a.5.5 0 01.5.5v8a.5.5 0 01-1 0V8.752l-6.267 3.636c-.52.302-1.233-.043-1.233-.696v-2.94l-6.267 3.636C.713 12.69 0 12.345 0 11.692V4.308c0-.653.713-.998 1.233-.696L7.5 7.248v-2.94c0-.653.713-.998 1.233-.696L15 7.248V4a.5.5 0 01.5-.5zM1 4.633v6.734L6.804 8 1 4.633zm7.5 0v6.734L14.304 8 8.5 4.633z" clip-rule="evenodd"/>
              </svg>
            </div>
          </b-card>
        </b-col>
        <b-col md="6">
          <b-card class="shadow">
            <table class="table">
              <thead>
                <tr>
                  <th scope="col">#</th>
                  <th scope="col">Title</th>
                  <th scope="col">Artist</th>
                  <th scope="col">Action</th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="(music, index) in allMusic" :key="index">
                  <th scope="row">{{index + 1}}</th>
                  <td>{{music.title}}</td>
                  <td>{{music.artist}}</td>
                  <td>
                    <button
                      class="btn btn-primary btn-sm"
                      @click="play(music)"
                    >Play</button>
                  </td>
                </tr>
              </tbody>
            </table>
          </b-card>
        </b-col>
      </b-row>
    </b-container>
  </section>
</template>

<script>
  export default {
    data() {
      return {
        current: {
          title: '',
          artist: ''
        },
        song: true,
        isplaying: false,
        allMusic: null,
        index: 0,
        player: ''
      }
    },
    methods: {
      async initPlayer() {
        if (this.allMusic !== []) {
          this.current = await this.allMusic[this.index]
          this.player.src = `${process.env.baseUrl}/${this.current.music.path}`
        } else {
          this.song = true
        }
      },
      async getAllSongs() {
        try {
          const response = await this.$axios.$get('/music')
          if (response === []) {
            this.song = true
            this.current = null
          } else {
            this.song = false
            this.allMusic = response
          }
          await this.initPlayer()
        } catch (err) {
          this.current = null
        }
      },
      play(song) {
        if (song) {
          this.current = song
          this.player.src = `${process.env.baseUrl}/${this.current.music.path}`
        }
        this.player.play()
        this.isplaying = true
      },
      pause() {
        this.player.pause()
        this.isplaying = false
      },
      prev() {
        this.index--
        if (this.index < 0) {
          this.index = this.allMusic.length - 1
        }
        this.current = this.allMusic[this.index]
        this.play(this.current)
      },
      next() {
        this.index++
        if (this.index > this.allMusic.length - 1) {
          this.index = 0
        }
        this.current = this.allMusic[this.index]
        this.play(this.current)
      },
    },
    created() {
      if (process.client) {
        this.player = new Audio()
      }
      this.getAllSongs()
    }
  }
</script>

<style scoped>
  .image {
    position: relative;
    height: 300px;
    width: 100%;
    border-radius: 8px;
  }
  .player_card {
    text-align: center;
    bottom: 20px;
    margin: 0px 40px;
  }
  .controls {
    font-size: 24px;
  }
  .controls svg {
    cursor: pointer;
  }
</style>
