<script lang="coffee">
import OutcomeService from '@/shared/services/outcome_service'
import EventService from '@/shared/services/event_service'

import { pick } from 'lodash'

export default
  components:
    ThreadItem: -> import('@/components/thread/item.vue')

  props:
    event: Object

  computed:
    eventable: -> @event.model()
    poll: -> @eventable.poll()
    # dockActions: ->
    #   OutcomeService.actions(@eventable, @)
    # menuActions: ->
    #   pick EventService.actions(@event, @), ['pin_event', 'unpin_event', 'notification_history']

</script>

<template lang="pug">
thread-item.outcome-created(:event="event")
  //- template(v-slot:actions)
  //-   v-layout(align-center)
  //-     reaction-display(:model="eventable")
  //-     action-dock(:model="eventable" :actions="dockActions")
  //-     action-menu(:model="eventable" :actions="menuActions")
  formatted-text.thread-item__body(:model="eventable" column="statement")
</template>
