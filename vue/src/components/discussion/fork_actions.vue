<script lang="coffee">
import Records      from '@/shared/services/records.coffee'
import openModal      from '@/shared/helpers/open_modal'

export default
  props:
    discussion: Object
  methods:
    openMoveCommentsModal: ->
      openModal
        component: 'MoveCommentsModal'
        props:
          discussion: @discussion
</script>

<template lang='pug'>
v-banner.discussion-fork-actions(single-line sticky elevation="4" v-model='discussion.isForking' icon="mdi-call-split" color="accent")
  span(v-t="'discussion_fork_actions.helptext'")
  template(v-slot:actions)
    v-btn(color="primary" @click="openMoveCommentsModal()" v-t="'discussion_fork_actions.move'")
    v-btn(icon @click='discussion.forkedEventIds = []; discussion.isForking = false')
      v-icon mdi-close
</template>
