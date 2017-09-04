<template>
    <div class="vue-calendar">
        <calendar-header
                :locale="locale"
                :full-month-names="fullMonthNames"
                :first-day-of-month="currentMonthStart"
                @changeMonth="changeMonth"
        >
        </calendar-header>

        <div class="calendar-body">
            <div class="days-header">
                <strong class="day-label" v-for="dayNumber in 7">{{ (dayNumber - 1) | dayToString(locale, firstDay, fullDayNames) }}</strong>
            </div>

            <div class="days-container">
                <div class="week-row" v-for="week in calendar">
                    <div class="week-day-cell" :class="{'today': day.isToday, 'not-current-month': !day.isCurrentMonth}"
                         v-for="day in week"
                         @click.stop="dayClick(day)"
                    >
                        <div class="day-number">
                            {{ day.monthDay }}
                        </div>
                        <events-box
                                :events="day.events"
                                :show-limit="showLimit"
                                @eventClicked="eventClicked"
                                @showMore="showEventsModal"
                        ></events-box>
                    </div>
                </div>
            </div>
        </div>

        <events-modal v-if="showModal" :events="currentEventsList" @hideModal="hideEventsModal"></events-modal>
    </div>
</template>
<script>
	  import dateHelper from '../utils/date-tools';
    import calHeader from './calendar-header.vue';
    import eventsBox from './events-box.vue';
    import eventsModal from './events-modal.vue';

    export default {
        props : {
            events : {
              type : Array,
              default : () => []
            },
            locale : {
              type : String,
              default : 'en'
            },
            firstDay : {
              type : Number | String,
              default : 0
            },
	          fullMonthNames:  {
            	type: Boolean,
              default: true
            },
            fullDayNames: {
	            type: Boolean,
	            default: false
            },
            showLimit: {
            	type: Number,
              default: 3
            },
            moreText: {
            	type: String,
              default: 'Show more'
            },
            disabled: {
            	type: Object,
            },
            highlight: {
            	type: Object
            }
        },
        data: function () {
          return {
          	showModal: false,
            currentEventsList: null,
            currentMonthStart: dateHelper.firstDateOfMonth()
          }
        },
        computed: {
          calendar () {
            return dateHelper.buildCalendar(this.currentMonthStart, this.events, this.firstDay);
          }
        },
        methods: {
          changeMonth: function (newMonth) {
            this.currentMonthStart = newMonth;
	          this.$emit('monthChanged', newMonth);
          },
          dayClick: function (day) {
            this.$emit('dayClicked', day);
          },
          eventClicked: function(event) {
          	this.$emit('eventClicked', event);
          },
          showEventsModal: function(events) {
            this.currentEventsList = events;
            this.showModal = true;
          },
          hideEventsModal: function () {
            this.showModal = false;
          }
        },
        filters: {
            dayToString (dayNumber, locale, dayOffset = 0, fullNames = 0) {
              dayNumber = parseInt(dayNumber + dayOffset) % 7;
              return dateHelper.localDayName(locale, dayNumber, fullNames);
            }
        },
        components: {
          'calendar-header': calHeader,
          'events-box': eventsBox,
          'events-modal': eventsModal,
        }
    }
</script>
<style>
    .vue-calendar {
        background: #fff;
        margin:0 auto;
    }
    .calendar-body {
        margin: 10px 0;
    }

    .days-header{
        display: flex;
        border-top:1px solid #e0e0e0;
        border-bottom:1px solid #e0e0e0;
        border-left:1px solid #e0e0e0;
    }
    .day-label{
        flex:1;
        text-align: center;
        border-right:1px solid #e0e0e0;
    }
    .day-number{
        text-align: right;
        margin-right: 10px;
    }

    .week-row{
        border-left:1px solid #e0e0e0;
        display: flex;
    }
    .week-day-cell{
        flex:1;
        min-height: 112px;
        padding:4px;
        border-right:1px solid #e0e0e0;
        border-bottom:1px solid #e0e0e0;
    }
    .week-day-cell.disabled-day{
        background-color: rgb(245, 245, 245);
    }
    .week-day-cell.not-current-month>.day-number{
        color: rgb(195,195,195);
    }
    .week-day-cell.today > .day-number{
        font-size: 25px;
        font-weight: bold;
        color: red;
    }
</style>