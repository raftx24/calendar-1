<template>
    <div class="events-wrapper">
        <div class="columns">
            <div class="column is-2-desktop is-8-tablet is-12-mobile">
                <calendar-filter @change-date="selectedDate = $event"
                    @update-selection="calendars = $event;"/>
            </div>
            <div class="column is-10-desktop is-12-tablet is-12-mobile">
                <enso-calendar :selected-date="selectedDate"
                    :calendars="calendars"
                    ref="calendar"
                    @edit-event="event = $event || {}"/>
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
import { mapMutations } from 'vuex';
import EnsoCalendar from './components/EnsoCalendar.vue';
import CalendarFilter from './components/CalendarFilter.vue';
import EventForm from './components/EventForm.vue';

export default {
    name: 'Index',

    components: { EnsoCalendar, CalendarFilter, EventForm },

    inject: ['errorHandler', 'route'],

    data: () => ({
        event: null,
        selectedDate: null,
        calendars: [],
    }),

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
        reloadEvents() {
            this.$refs.calendar.fetch();
            this.event = null;
        },
        resize() {
            this.$el.style.height = `${document.body.clientHeight - 270}px`;
        },
    },
};
</script>
