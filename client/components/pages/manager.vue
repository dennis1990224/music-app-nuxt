<template>
  <section class="mt-5">
    <b-container>
      <b-row class="mb-4">
        <b-col>
          <b-card v-if="addState">
            <div class="card-title mb-4">
              <h4>Add Music</h4>
            </div>
            <b-form @submit.prevent="addNewMusic" enctype="multipart/form-data">
              <b-form-group
                label="Title"
                label-for="title"
              >
                <b-form-input
                  id="title"
                  v-model="musicDetails.title"
                  type="text"
                  required
                ></b-form-input>
              </b-form-group>
              <b-form-group
                label="Artist"
                label-for="artist"
              >
                <b-form-input
                  id="artist"
                  v-model="musicDetails.artist"
                  type="text"
                  required
                ></b-form-input>
              </b-form-group>
              <b-form-group>
                <label for="customFile">
                  Music
                </label>
                <div class="custom-file">
                  <input
                    type="file"
                    id="customFile"
                    ref="file"
                    v-on:change="handleFileUpload()"
                    class="custom-file-input"
                  />
                  <label class="custom-file-label" for="customFile">
                    Choose file
                  </label>
                </div>
              </b-form-group>
              <b-form-group>
                <button class="btn btn-primary" :disabled="isDisabled">
                  <span
                    class="spinner-border spinner-border-sm"
                    v-if="addLoading"
                    role="status"
                    aria-hidden="true"
                  ></span>Submit
                </button>
              </b-form-group>
            </b-form>
          </b-card>
        </b-col>
      </b-row>

      <b-row>
        <b-col>
          <b-card class="bg-light p-1 showdow-sm">
            <button
              class="btn btn-info mb-4"
              @click="initForm"
            >
              {{addState ? 'Cancel' : 'Add New Music'}}
            </button>
            <table class="table">
              <thead>
                <tr>
                  <th scope="col">#</th>
                  <th scope="col">Title</th>
                  <th scope="col">Artist</th>
                  <th scope="col">Date created</th>
                  <th scope="col">Action</th>
                </tr>
              </thead>
              <tbody
                v-if="musicLoading"
                role="status"
              >
                <tr>
                  <td colspan="5" class="text-center">
                    <div class="spinner-border">
                      <span class="sr-only">Loading...</span>
                    </div>
                  </td>
                </tr>
              </tbody>
              <tbody v-else>
                <tr v-for="(music, index) in allmusic" :key="index">
                  <td>{{ index + 1 }}</td>
                  <td>{{ music.title }}</td>
                  <td>{{ music.artist }}</td>
                  <td>{{ music.created }}</td>
                  <td>
                    <button class="btn btn-info" @click="deleteMusic(music._id)">
                      Delete
                    </button>
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
        musicDetails: {
          title: '',
          artist: '',
          music: ''
        },
        allmusic: [],
        addState: false,
        addLoading: false,
        musicLoading: false,
        isValid: false
      }
    },
    computed: {
      isDisabled: function() {
        if (
          this.musicDetails.title === '' ||
          this.musicDetails.artist === '' ||
          this.musicDetails.music === ''
        ) {
          return !this.isValid
        }
      }
    },
    methods: {
      initForm() {
        this.addState = !this.addState
      },
      handleFileUpload() {
        this.musicDetails.music = this.$refs.file.files[0]
      },
      async getAllMusics() {
        this.musicLoading = true
        try {
          this.allmusic = await this.$axios.$get('/music')

          this.musicLoading = false
        } catch (err) {
          this.musicLoading = false

          this.$swal.fire({
            title: 'Error!',
            text: 'Error in fetching Musics.',
            icon: 'error'
          })
        }
      },
      addNewMusic() {
        const types = /(\.|\/)(mp3|mp4)$/i
        if (
          types.test(this.musicDetails.music.type) ||
          types.test(this.musicDetails.music.name)
        ) {
          let formData = new FormData()

          formData.append('title', this.musicDetails.title)
          formData.append('artist', this.musicDetails.artist)
          formData.append('music', this.musicDetails.music)

          this.addLoading = true

          this.$axios
            .$post('/music', formData)
            .then(response => {
              this.addLoading = false
              this.addState = false
              this.musicDetails = {}

              this.getAllMusics()

              this.$swal.fire({
                title: 'Success!',
                text: 'New Music has been added successfully.',
                icon: 'success'
              })
            })
            .catch(err => {
              this.addLoading = false

              this.$swal.fire({
                title: 'Error!',
                text: 'Something went wrong.',
                icon: 'error'
              })
            })
        } else {
          this.$swal.fire({
            title: 'Error!',
            text: 'Invalid file type.',
            icon: 'error'
          })

          return !this.isValid
        }
      },
      deleteMusic(id) {
        this.$swal.fire({
          title: 'Are you sure?',
          text: 'Once deleted, you will not be able to recover this Music.',
          icon: 'warning',
          showCancelButton: true
        }).then(({value}) => {
          if (value) {
            this.$axios
              .$delete(`/music/${id}`)
              .then(response => {
                this.getAllMusics()

                this.$swal.fire({
                  title: 'Success!',
                  text: 'Your Music file has been deleted.',
                  icon: 'success'
                })
              })
              .catch(err => {
                this.$swal.fire({
                  title: 'Error!',
                  text: 'Something went wrong.',
                  icon: 'error'
                })
              })
          } else {
            this.$swal.fire({
              title: 'Alert!',
              text: 'Your Music file is safe.',
              icon: 'info'
            })
          }
        })
      }
    },
    created() {
      this.getAllMusics()
    }
  }
</script>

