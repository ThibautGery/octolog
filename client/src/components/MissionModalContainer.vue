<template>
  <div class="modal-mask" v-if="active">
    <div class="modal-wrapper">
      <div class="modal-container">
        <div class="mission">
          <div class="mission-logo-time">
            <img v-bind:src="mission.customerLogo"/>
            <input v-model="mission.customerLogo">
            <input v-model="mission.from">
            <input v-model="mission.to">
          </div>
          <div class="mission-desc">
            <h2><input v-model="mission.role"> for <input v-model="mission.customer"></h2>
            <textarea v-model="missionDescription" rows="5" cols="100"></textarea>
            <p class="mission-keywords"><input placeholder="tag1, tag2" v-model.lazy="missionTags"></p>
          </div>
        </div>
        <button class="modal-default-button" v-on:click="close">Cancel</button>
        <button class="modal-default-button" v-on:click="updateMission">OK</button>
      </div>
    </div>
  </div>
</template>

<script>
import MissionModalHub from '@/components/events/MissionModalHub'
import axios from 'axios'

export default {
  data () {
    return {
      active: false,
      trigram: '',
      mission: {},
      missionTags: '',
      missionDescription: '-... \n-...'
    }
  },
  mounted () {
    this.$nextTick(function () {
      MissionModalHub.$on('set-modal-data', this.set)
      MissionModalHub.$on('open-modal', this.open)
    }.bind(this))
  },
  destroyed () {
    MissionModalHub.$off('set-modal-data', this.set)
    MissionModalHub.$off('open-modal', this.open)
  },
  methods: {
    open () {
      this.active = true
    },
    set (mission, trigram) {
      this.trigram = trigram
      this.mission = mission
      if (mission.tags) {
        this.missionTags = mission.tags.join(',')
      }
      if (mission.description) {
        this.missionDescription = '- ' + mission.description.join('\n- ')
      }
    },
    close () {
      this.active = false
    },
    updateMission () {
      this.mission.tags = this.missionTags.split(',')
      this.mission.description = this.missionDescription.substring(2).split('\n- ')
      return axios.put(process.env.API_URL + process.env.UPDATE_EXPERIENCE_PATH.replace('{id}', this.mission.id).replace('{trigram}', this.trigram))
        .then((response) => {
          this.mission = response.data
          this.active = false
        })
        .catch(e => {
          console.log(e)
          this.errors.push(e)
        })
    }
  }
}
</script>
