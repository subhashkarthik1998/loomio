<script lang="coffee">
import Session        from '@/shared/services/session'
import AbilityService from '@/shared/services/ability_service'
import openModal from '@/shared/helpers/open_modal'

export default
  components:
    ThreadItem: -> import('@/components/thread/item.vue')

  props:
    event: Object

  computed:
    eventable: -> @event.model()
    choiceInHeadline: ->
      @eventable.poll().hasOptionIcons() &&
      @eventable.stanceChoices().length == 1
    canEdit: ->

    componentType:  ->
      if @event.actor()
        'router-link'
      else
        'div'

  created: ->
    @actions =
      edit_stance:
        icon: 'mdi-pencil'
        canPerform: =>
          @eventable.latest && @eventable.poll().isActive() && @eventable.participant() == Session.user()
        perform: =>
          openModal
            component: 'PollCommonEditVoteModal',
            props:
              stance: @eventable.clone()

      translate_stance:
        icon: 'mdi-translate'
        canPerform: =>
          @eventable.reason && AbilityService.canTranslate(@eventable)
        perform: =>
          @eventable.translate(Session.user().locale)

      show_history:
        icon: 'mdi-history'
        canPerform: => @eventable.edited()
        perform: =>
          openModal
            component: 'RevisionHistoryModal'
            props:
              model: @eventable
</script>

<template lang="pug">
thread-item.stance-created(:event="event")
  template(v-slot:actions)
    action-dock(:model="eventable" :actions="actions")
  template(v-if="choiceInHeadline" v-slot:headline)
    v-layout(align-center)
      component(:is="componentType" :to="event.actor() && urlFor(event.actor())") {{event.actorName()}}
      space
      poll-common-stance-choice(:stance-choice="eventable.stanceChoices()[0]")
  poll-common-stance(:stance="eventable" :reason-only="choiceInHeadline")
  attachment-list(:attachments="eventable.attachments")
</template>
