<template>
  <div class="imagevote">
    <div class="imagepane">
      <div class="left"  :style="{ backgroundImage: `url(${imageLeftUrl})`  }" />
      <div class="right" :style="{ backgroundImage: `url(${imageRightUrl})` }" />
    </div>

    <p>
      <button href="" @click.prevent="voteLeft">Left</button>
      <button href="" @click.prevent="voteRight">Right</button>
    </p>
    <button href="" @click.prevent="voteUndecided">Unsure</button>
    <button href="" @click.prevent="nextImagePair">Next</button>

    <p>{{resources.length}} votes</p>
    <p v-for="r in resources" :key="r.id">
      <span v-if="r.is_undecided">
        Undecided |
      </span>
      <span v-if="!r.is_undecided">
        Choice: {{r.choice_id}} |
      </span>
      Time: {{r.time_elapsed}} |
      At: {{r.created | formatTimestamp }}
    </p>
    <p>{{error}}</p>
  </div>
</template>

<script>

import $backend from '../backend'

export default {
  name: 'about',
  data () {
    return {
      resources: [],
      error: '',
      imageLeft: 0,
      imageLeftUrl: '/loading.gif',
      imageRight: 0,
      imageRightUrl: '/loading.gif',
      timeStart: Date.now()
    }
  },
  methods: {
    elapsedTime () {
      return Math.round((Date.now() - this.timeStart) / 1000)
    },
    nextImagePair () {
      this.timeStart = Date.now()
      console.log('Fetching image pair')
      $backend.getRandomImages()
        .then(responseData => {
          console.log(responseData)
          this.imageLeft = responseData[0].id
          this.imageLeftUrl = responseData[0].Url
          this.imageRight = responseData[1].id
          this.imageRightUrl = responseData[1].Url
        })
    },
    castVote (isRight) {
      $backend.castVote(
        isRight,
        this.imageLeft,
        this.imageRight,
        this.elapsedTime()
      )
        .then(responseData => {
          if (responseData !== null) {
            this.resources.push(responseData)
            this.nextImagePair()
          }
        }).catch(error => {
          console.error(error.message)
          if (error.message.indexOf('429')) {
            this.error = 'Please slow down'
          } else {
            this.error = 'There was an error, please try again'
          }
        })
    },
    voteLeft () {
      this.castVote(false)
    },
    voteRight () {
      this.castVote(true)
    },
    voteUndecided () {
      this.castVote(null)
    }
  },
  mounted () {
    this.nextImagePair()
  }
}
</script>

<style scoped lang="scss">
.imagepane {
  text-align: center;
  div {
    display: inline-block;
    width: 40%;
    height: 60%;
    min-height: 400px;
    padding: 3px;
    background-size: cover;
    background-repeat: no-repeat;
    background-position: bottom left;
  }
}
</style>