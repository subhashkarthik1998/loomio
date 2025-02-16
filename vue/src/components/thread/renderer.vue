<script lang="coffee">
import { reverse, throttle, debounce, map, range, min, max, times, first, last, clone, cloneDeep, sortedUniq, sortBy, difference, isNumber, isEqual, uniq, without, pull } from 'lodash'
import Records from '@/shared/services/records'
import EventBus from '@/shared/services/event_bus'
import RecordLoader from '@/shared/services/record_loader'
import EventHeights from '@/shared/services/event_heights'

export default
  props:
    discussion: Object
    parentEvent: Object
    fetch: Function
    viewportIsBelow: Boolean
    viewportIsAbove: Boolean

  created: ->
    @fetchMissing = debounce ->
      @fetch(@missingSlots)
    , 500

    @loader = new RecordLoader
      collection: 'events'
      params:
        per: @padding * 2

    @watchRecords
      key: 'parentEvent'+@parentEvent.id
      collections: ['events']
      query: @renderSlots

    EventBus.$on 'focusedEvent', @focused

    @renderSlots()

  beforeDestroy: ->
    delete @loader
    EventBus.$off 'focusedEvent', @focused

  data: ->
    loader: null
    eventsBySlot: {}
    visibleSlots: []
    recentSlots: []
    missingSlots: []
    slots: []
    padding: 20
    focusedEvent: null

  methods:
    focused: (event) ->
      @focusedEvent = event          if event.parentId == @parentEvent.id
      @focusedEvent = event.parent() if event.parent() && event.parent().parentId == @parentEvent.id

    renderSlots: ->
      return if @parentEvent.childCount == 0
      defaultFirst = 1
      defaultLast = 1
      if @parentEvent.depth == 0 && ((@discussion.newestFirst && !@viewportIsBelow) ||
                                    (!@discussion.newestFirst &&  @viewportIsBelow))
        defaultFirst = @parentEvent.childCount
        defaultLast = @parentEvent.childCount

      if @parentEvent.depth == 0
        # replace items with slots
        firstRendered = max([1, (first(@visibleSlots) || defaultFirst) - @padding])
        lastRendered = min([(last(@visibleSlots) || defaultLast) + @padding, @parentEvent.childCount])
        firstSlot = max([1, firstRendered - (@padding * 2)])
        lastSlot = min([lastRendered + (@padding * 2), @parentEvent.childCount])
      else
        firstSlot = firstRendered = 1
        lastSlot = lastRendered = @parentEvent.childCount

      eventsBySlot = {}

      for i in [firstSlot..lastSlot]
        eventsBySlot[i] = null

      presentPositions = []
      Records.events.collection.chain().
      find(parentId: @parentEvent.id).
      find(position: {$between: [firstRendered, lastRendered]}).
      simplesort('position').
      data().forEach (event) =>
        presentPositions.push(event.position)
        eventsBySlot[event.position] = event

      expectedPositions = range(firstRendered, lastRendered+1)
      @eventsBySlot = eventsBySlot
      @missingSlots = difference(expectedPositions, presentPositions)
      @eventsBySlot[@focusedEvent.position] = @focusedEvent if @focusedEvent

      if @discussion.newestFirst && @parentEvent.depth == 0
        @slots = reverse([firstSlot..lastSlot])
      else
        @slots = [firstSlot..lastSlot]

    slotVisible: (isVisible, slot) ->
      slot = parseInt(slot)
      if isVisible
        @visibleSlots = sortedUniq(sortBy(@visibleSlots.concat([slot])))
      else
        @visibleSlots = without(@visibleSlots, slot)

  watch:
    visibleSlots: (newVal, oldVal) ->
      # console.log "visibleSlots #{@visibleSlots}"
      @renderSlots() unless isEqual(newVal, oldVal)

    missingSlots: (newVal, oldVal) ->
      # console.log "fetch pid #{@parentEvent.id} missing #{newVal}"
      @fetchMissing(newVal) unless isEqual(newVal, oldVal)

    'discussion.newestFirst': -> @visibleSlots = []
    # 'viewportIsBelow': (newVal, oldVal) -> @visibleSlots = [] if newVal
    # 'viewportIsAbove': (newVal, oldVal) -> @visibleSlots = [] if newVal

</script>
<template lang="pug">
.thread-renderer
  .thread-item-slot(v-for="slot in slots" :id="'position-'+slot" :key="slot" v-observe-visibility="{callback: (isVisible) => slotVisible(isVisible, slot)}" )
    thread-item-wrapper(:parent-id="parentEvent.id" :event="eventsBySlot[slot]" :position="parseInt(slot)")
  //- div
    | depth {{parentEvent.depth}}
    | position {{parentEvent.position}}
    | visible {{visibleSlots}}
    | missing {{missingSlots}}
    | viewportIsBelow {{viewportIsBelow}}
</template>
