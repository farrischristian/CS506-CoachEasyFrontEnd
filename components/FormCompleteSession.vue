<template>
  <div>
    <Loading v-if="loading" />
    <MessageError v-if="error" :message="errorMessage" />
    <div v-if="!loading">
      <HeadingSection :text="session.name" />
      <SpacerExtraSmall />
      <div class="mainDisplay">
        <div class="secondaryDisplay">
          <v-text-field
            label="Body Weight"
            v-model="session.client_weight">
          </v-text-field>
        </div>
        <div class="exerciseClientGrid exerciseClientGridHeader">
          <p class="exerciseClientCol exerciseFirstCol">Name</p>
          <p class="exerciseClientCol">Sets</p>
          <p class="exerciseClientCol">Reps</p>
          <p class="exerciseClientCol">Weight</p>
        </div>
        <FormClientExercise v-for="(exercise, index) in exerciseList" 
          :single="!loading"
          :key="index"
          :exercise="exercise" 
          :edit=true />
        <div class="secondaryDisplay">
          <v-text-field
            label="Comment"
            v-model="session.comment">
          </v-text-field>
        </div>
        <div v-if="role==='COACH'">
          <ViewCoachExercise v-for="(exercise, index) in exerciseList" 
            :key="index" 
            :exercise="exercise"  />
        </div>
        <ButtonViewSession v-if="type==='Session'" @complete="completeSession()" :session="this.session" action="Complete" />
      </div>
      <SpacerSmall />
    </div>
  </div>
</template>

<script>

import axios from 'axios'
axios.defaults.withCredentials = true;
const url = 'https://coach-easy-deploy.herokuapp.com';

import HeadingSection from '~/components/HeadingSection'
import SpacerExtraSmall from '~/components/SpacerExtraSmall'
import FormClientExercise from '~/components/FormClientExercise'
import ViewCoachExercise from '~/components/ViewCoachExercise'
import ButtonViewSession from "~/components/ButtonViewSession"
import SpacerSmall from "~/components/SpacerSmall"
import Loading from "~/components/Loading"
import MessageError from "~/components/MessageError"

export default {
  props: {
    session: Object,
    type: String,
  },
  components: {
    SpacerExtraSmall,
    FormClientExercise,
    ViewCoachExercise,
    HeadingSection,
    ButtonViewSession,
    SpacerSmall,
    Loading,
    MessageError
  },
  data() {
    return {
      exerciseList: [],
      role: '',
      loading: true,
      loadingFailed: false,
    }
  },
  methods: {
    getSessionRole: function(){
      Promise.all([ this.$store.state.userData ]).then( () => {
        this.role = this.$store.state.userData.role
        this.updateExerciseList();
        // this.loading = false
      },() => {
        this.loadingFailed = true
      })
    },
    updateExerciseList: function() {
      if(this.$props.session.training_entries.length > 0){
        this.exerciseList = this.$props.session.training_entries
      } else if(this.$props.role === 'COACH'){
        this.exerciseList = this.$props.session.coach_exercises ;
      } else if(this.$props.session.exercises){
        this.exerciseList = this.$props.session.exercises;
      } else {
        this.exerciseList = this.$props.session.client_exercises
      }
      this.loading = false;
    },
    completeSession: function() {
      this.$emit('complete');
    }
  },
  mounted() {
    this.getSessionRole();
    console.log('form view session mounted');
  }
}
</script>

<style lang="scss">
.mainDisplay{
  border-radius: 16px;
  box-shadow: $elevation2;
  overflow: hidden;
}
.secondaryDisplay{
  display: flex;
  margin: 8px 16px 8px 16px !important;
}
.exerciseClientGridHeader{
  p{
    font-weight: 500;
  }
}
.completedCheckbox{
  justify-content: flex-end;
}
.v-input--selection-controls{
  margin-top: 0px;
}
</style>
