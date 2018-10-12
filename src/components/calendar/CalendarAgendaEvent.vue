<template>
  <div
    :class="getDotEventClass()"
    :style="getEventStyle()"
    @mouseup="handleClick"
  >
    <!-- dot style -->
    <div class="col-auto calendar-agenda-event-dot" :class="getDotClass()"></div>
    <div>
      <div>
        <span
          v-if="showTime"
          class="col-auto calendar-agenda-event-time"
        >
          {{ formatTimeRange(eventObject.start.dateObject,
          eventObject.end.dateObject) }}
        </span>
        <q-chip v-show="showPeople" dense square :icon="eventObject.attendees.length > 1 ? 'people' : 'person'" :color="getPhotographerCountColor">
          {{ eventObject.attendees.length }}
        </q-chip>
        <span v-show="showSummary" class="calendar-agenda-event-summary">
          {{ eventObject.summary }}
        </span>
      </div>
      <span v-show="showSummaryWrap" class="calendar-agenda-event-summary">
        {{ eventObject.summary }}
      </span>
    </div>
  </div>
</template>

<script>
  import CalendarMixin from './mixins/CalendarMixin'
  import {
    QBtn,
    QTooltip
  } from 'quasar'
  const { DateTime } = require('luxon')
  export default {
    name: 'CalendarAgendaEvent',
    props: {
      eventObject: {
        type: Object,
        default: () => {}
      },
      agendaStyle: {
        type: String,
        default: 'block'
      },
      color: {
        type: String,
        default: 'primary'
      },
      textColor: {
        type: String,
        default: 'white'
      },
      showTime: {
        type: Boolean,
        default: true
      },
      monthStyle: {
        type: Boolean,
        default: false
      },
      eventRef: String,
      calendarLocale: {
        type: String,
        default: () => { return DateTime.local().locale }
      },
      calendarTimezone: {
        type: String,
        default: () => { return DateTime.local().zoneName }
      },
      allowEditing: {
        type: Boolean,
        default: false
      }
    },
    components: {
      QBtn,
      QTooltip
    },
    mixins: [CalendarMixin],
    data () {
      return {
        elementWidth: 0
      }
    },
    computed: {
      showPeople () {
        return this.elementWidth > 130
      },
      showSummary () {
        return this.elementWidth > 300
      },
      showSummaryWrap () {
        return this.elementWidth <= 300 && this.elementWidth > 210
      },
      getPhotographerCountColor () {
        const attendees = this.eventObject.attendees
        if (!this.$userProfileData) {
          console.error("$userProfileData not found. This error will only happen if there was an error with logic elsewhere!")
          return 'negative'
        }
        const photographerId = this.$userProfileData.photographerId
        let colour = 'info'
        if (Array.isArray(attendees) && photographerId) {
          colour = attendees.some(a => a.photographerId == photographerId) ? 'positive': 'info'
        }
        return colour
      }
    },
    methods: {
      getDotClass: function () {
        return this.addCssColorClasses({}, this.eventObject)
      },
      getDotEventClass: function () {
        return {
          'row': true,
          'items-center': true,
          'justify-start': true,
          'cursor-pointer': true,
          'calendar-agenda-event': true,
          'calendar-agenda-event-dot-style': true,
          'calendar-agenda-event-empty-slot': this.eventObject.start.isEmptySlot
        }
      },
      getEventClass: function () {
        return this.addCssColorClasses(
          {
            'calendar-agenda-event': true,
            'calendar-agenda-event-empty-slot': this.eventObject.start.isEmptySlot
          },
          this.eventObject
        )
      },
      getEventStyle: function () {
        return {}
      },
      formatTimeRange: function (startTime, endTime) {
        let returnString = ''
        // start time
        returnString += this.simplifyTimeFormat(
          this.makeDT(startTime).toLocaleString(DateTime.TIME_SIMPLE),
          (this.formatDate(startTime, 'a') === this.formatDate(endTime, 'a'))
        )
        if (this.$q.screen.gt.md) {
          returnString += ' - '
          // end time
          returnString += this.simplifyTimeFormat(
            this.makeDT(endTime).toLocaleString(DateTime.TIME_SIMPLE),
            false
          )
        }
        return returnString
      },
      handleClick: function (e) {
        this.eventObject.allowEditing = this.allowEditing
        this.$emit('click', this.eventObject)
        this.triggerEventClick(this.eventObject, this.eventRef)
      },
      updateWidth () {
        this.elementWidth = this.$el.clientWidth
      }
    },
    mounted () {
      this.$nextTick(() => {
        window.addEventListener('resize', this.updateWidth)
        this.updateWidth()
      })
    }
  }
</script>

<style lang="stylus">
  @import 'calendar.vars.styl'
  .calendar-agenda-event-empty-slot
    display none
    background green

  .calendar-agenda-event-dot-style
    width 100%
    background-color inherit
    transition background-color 0.3s ease
    &:hover
      background-color $whiteHighlightBackgroundColor
      transition background-color 0.3s ease
    .calendar-agenda-event-time
    .calendar-agenda-event-summary
      margin-left 1em
    .calendar-agenda-event-time
      width 170px
      @media screen and (max-width: $breakpoint-sm)
        width 60px
    .calendar-agenda-event-dot
      border-radius 12px
      width 12px
      height 12px
</style>
