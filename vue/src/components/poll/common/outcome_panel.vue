<script lang="coffee">
import OutcomeService from '@/shared/services/outcome_service'

export default
  props:
    poll: Object

  created: ->

  computed:
    actions: -> OutcomeService.actions(@outcome)
    outcome: -> @poll.outcome()

</script>

<template lang="pug">
.poll-common-outcome-panel(v-if="outcome")
  h3.lmo-card-subheading(v-t="'poll_common.outcome'")
  .poll-common-outcome-panel__authored-by.caption
    span(v-t="{ path: 'poll_common_outcome_panel.authored_by', args: { name: outcome.authorName() } }")
    time-ago(:date="outcome.createdAt")
  formatted-text(:model="outcome" column="statement")
  document-list(:model="outcome")
  v-layout(align-center)
    reaction-display(:model="outcome")
    action-dock(:model="outcome" :actions="actions")
</template>
