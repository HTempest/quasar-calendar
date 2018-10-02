<template>
  <div>
    <div
      v-for="thisEvent in monthGetDateEvents(thisDay.dateObject)[0]"
      :key="thisEvent.id"
    >
      <calendar-event
        :event-object="thisEvent"
        :month-style="true"
        :event-ref="eventRef"
        :prevent-event-detail="preventEventDetail"
        :current-calendar-day="thisDay.dateObject"
        :has-previous-day="thisEvent.hasPrev"
        :has-next-day="thisEvent.hasNext"
        :first-day-of-week="(weekDayIndex === 0)"
        :last-day-of-week="(weekDayIndex === (thisWeek.length -1))"
        :allow-editing="allowEditing"
      />
    </div>
    <div class="event-overflow-ellipsis text-center" v-if="monthGetDateEvents(thisDay.dateObject)[1].length > 0">
      &bull;&bull;&bull; {{ monthGetDateEvents(thisDay.dateObject)[1].length }} 
    </div>
    {{ thisDay }}
  </div>
</template>

<script>
  import CalendarMixin from './mixins/CalendarMixin'
  import CalendarEventMixin from './mixins/CalendarEventMixin'
  import CalendarParentComponentMixin from './mixins/CalendarParentComponentMixin'
  import CalendarEvent from './CalendarEvent'
  
  export default {
    name: 'CalendarMonthDay',
    components: {
      CalendarEvent
    },
    mixins: [CalendarParentComponentMixin, CalendarMixin, CalendarEventMixin],
    props: {
      monthStyle: Boolean,
      eventRef: String,
      preventEventDetail: Boolean,
      currentCalendarDay: Object,
      hasPreviousDay: Boolean,
      hasNextDay: Boolean,
      firstDayOfWeek: Boolean,
      lastDayOfWeek: Boolean,
      allowEditing: Boolean,
      thisDay: Object,
      maxEvents: Number
    },
    data () {
      return {
        // dayCellHeight: 5,
        // dayCellHeightUnit: 'rem',
        // workingDate: new Date(),
        // weekArray: [],
        parsed: this.getDefaultParsed(),
        // eventDetailEventObject: {}

        visibleEvents: [],
        remainingEvents: []
      }
    },
    computed: {
      calendarDaysAreClickable: function () {
        return (this.fullComponentRef && this.fullComponentRef.length > 0)
      }
    },
    methods: {
      monthGetDateEvents: function (dateObject) {
        console.log(dateObject)
        const events = this.dateGetEvents(dateObject)
        console.info("maxEvents", this.maxEvents)
        var a = [
          events.slice(0, this.maxEvents),
          events.slice(this.maxEvents)
        ]
        console.log(a);
        return a;
      }
    },
    mounted () {
      // this.doUpdate()
      // getEvents()
      this.handlePassedInEvents()
      this.$root.$on(
        this.eventRef + ':navMovePeriod',
        this.handleNavMove
      )
      this.$root.$on(
        'click-event-' + this.eventRef,
        this.handleEventDetailEvent
      )
      this.$root.$on(
        'update-event-' + this.eventRef,
        this.handleEventUpdate
      )
    }
  }
</script>

<style lang="stylus">
  @import 'calendar.vars.styl'
  .event-overflow-ellipsis
    font-weight 1000
    letter-spacing 0.5em
</style>
