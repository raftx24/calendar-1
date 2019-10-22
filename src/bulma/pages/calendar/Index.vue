<template>
    <div class="events-wrapper">
        <div class="columns">
            <div class="column is-2-desktop is-8-tablet is-12-mobile">
                <enso-calendar-filter
                    @change-date="selectedDate = $event"
                    @change-calendars="selectedCalendars = $event; fetch();"/>
            </div>
            <div class="column is-10-desktop is-12-tablet is-12-mobile">
                <enso-calendar :events="events"
                               :on-event-dblclick="selectEvent"
                               :selected-date="selectedDate"
                               @ready="updateInterval"
                               @view-change="updateInterval"
                               @event-duration-change="update"
                               @event-delete="destroy"
                               @add-event="event = $event || {}"/>
            </div>
        </div>

        <event-form :event="event"
                    @submit="reloadEvents"
                    @close="event = null"
                    @destroy="reloadEvents"
                    v-if="event"/>
    </div>
</template>

<script>
import { mapState, mapMutations } from 'vuex';
import { EnsoSelect } from '@enso-ui/bulma';
import format from '@enso-ui/ui/src/modules/plugins/date-fns/format';
import VueCal from 'vue-cal';
import EnsoCalendar from './components/EnsoCalendar.vue';
import EnsoCalendarFilter from './components/EnsoCalendarFilter.vue';
import EventForm from './components/EventForm.vue';

export default {
    name: 'Index',

    components: {
        EnsoCalendar, EnsoCalendarFilter, EventForm, EnsoSelect, VueCal,
    },

    inject: ['errorHandler', 'route'],

    data: () => ({
        calendar: '1',
        event: null,
        events: [],
        interval: null,
        test: null,
        selectedDate: null,
        selectedCalendars: null,
    }),

    computed: {
        ...mapState(['enums']),
        ...mapState(['meta']),
        params() {
            if (!this.interval) {
                return { calendars: this.selectedCalendars };
            }

            if (this.interval.view === 'month') {
                return {
                    calendars: this.selectedCalendars,
                    startDate: `${this.dateFormat(this.interval.firstCellDate)} 00:00:00`,
                    endDate: `${this.dateFormat(this.interval.lastCellDate)} 23:59:59`,
                };
            }
            return {
                calendars: this.selectedCalendars,
                startDate: `${this.dateFormat(this.interval.startDate)} 00:00`,
                endDate: `${this.dateFormat(this.interval.endDate)} 23:59:59`,
            };
        },
    },
    created() {
        this.hideFooter();
    },

    mounted() {
        this.resize();

        window.addEventListener('resize', this.resize);
    },

    beforeDestroy() {
        window.removeEventListener('resize', this.resize);

        this.showFooter();
    },

    methods: {
        ...mapMutations('layout', ['showFooter', 'hideFooter']),
        fetch() {
            axios.get(this.route('core.calendar.events.index'), { params: this.params })
                .then(({ data }) => (this.events = data))
                .catch(this.errorHandler);
        },
        selectEvent(event, e) {
            if (event.route) {
                this.$router.push(event.route);
                return;
            }

            if (!event.readonly) {
                this.event = event;
            }
            e.stopPropagation();
        },
        update($event) {
            axios.patch(
                this.route('core.calendar.events.update', { event: $event.id }),
                { ends_time_at: `${this.timeFormat($event.end)}` },
            ).then(({ data }) => {
                this.$toastr.success(data.message);
                this.reloadEvents();
            }).catch(this.errorHandler);
        },
        destroy($event) {
            axios.delete(
                this.route('core.calendar.events.destroy', { event: $event.id }),
            ).then(() => (this.fetch())).catch(this.errorHandler);
        },
        updateInterval(interval) {
            this.interval = interval;
            this.fetch();
        },
        reloadEvents() {
            this.fetch();
            this.event = null;
        },
        resize() {
            this.$el.style.height = `${document.body.clientHeight - 270}px`;
        },
        dateFormat(date) {
            return format(date, `${this.meta.dateFormat}`);
        },
        timeFormat(dateTime) {
            return format(dateTime, 'H:i');
        },
    },
};
</script>
