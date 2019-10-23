<template>
    <div class="calendar-wrapper box is-paddingless raises-on-hover">
        <vue-cal class="vuecal--green-theme"
            :time-from="7 * 60"
            :locale="lang"
            :selected-date="selectedDate"
            :events="events"
            show-all-day-events
            today-button
            watch-real-time
            events-count-on-year-view
            v-bind="$attrs"
            @ready="updateInterval"
            @view-change="updateInterval"
            @event-mouse-enter="hovering = $event.id"
            @event-mouse-leave="hovering = null"
            @event-delete="destroy"
            @event-duration-change="update"
            :on-event-dblclick="selectEvent"
            :on-event-create="addEvent"
            editable-events
            v-on="$listeners">
            <template v-slot:today-button>
                <a class="button is-primary is-small is-rounded has-margin-left-large">
                        <span class="is-bold">
                            {{ i18n('Today') }}
                        </span>
                </a>
            </template>
            <template v-slot:title="{ title, view }">
                <div>
                    {{ title }}
                    <a class="button is-primary is-small is-rounded has-margin-left-large"
                        @click.stop="$emit('edit-event')">
                        <span class="is-bold">
                            {{ i18n('Add Event') }}
                        </span>
                        <span class="icon is-small">
                            <fa icon="plus"/>
                        </span>
                    </a>
                </div>
            </template>
            <template v-slot:event-renderer="{ event, view }">
                <div>
                    <b class="has-text-centered">
                        {{ event.title }}
                    </b>
                    <p class="event-body has-margin-bottom-small"
                        v-if="event.body"
                        v-html="event.body"/>
                    <div v-if="!event.allDay">
                        <p class="has-text-centered"
                            v-if="hovering === event.id">
                                {{ dateTimeFormat(event.daysCount,event.start) }}
                                <fa icon="arrows-alt-h"/>
                                {{ dateTimeFormat(event.daysCount,event.end) }}
                        </p>
                    </div>
                </div>
            </template>
        </vue-cal>
    </div>
</template>

<script>
import { mapState, mapGetters } from 'vuex';
import VueCal from 'vue-cal';
import { library } from '@fortawesome/fontawesome-svg-core';
import { faPlus, faFlag, faArrowsAltH } from '@fortawesome/free-solid-svg-icons';
import format from '@enso-ui/ui/src/modules/plugins/date-fns/format';

import('../styles/colors.scss');

library.add(faPlus, faFlag, faArrowsAltH);

export default {
    name: 'EnsoCalendar',

    components: { VueCal },

    inject: ['errorHandler', 'route', 'i18n'],

    props: {
        selectedDate: {
            required: true,
        },
        calendars: {
            required: true,
        },
    },
    computed: {
        ...mapState(['meta']),
        ...mapGetters('preferences', ['lang']),
        params() {
            if (!this.interval) {
                return { calendars: this.calendars };
            }

            if (this.interval.view === 'month') {
                return {
                    calendars: this.calendars,
                    startDate: `${this.dateFormat(this.interval.firstCellDate)} 00:00:00`,
                    endDate: `${this.dateFormat(this.interval.lastCellDate)} 23:59:59`,
                };
            }
            return {
                calendars: this.calendars,
                startDate: `${this.dateFormat(this.interval.startDate)} 00:00:00`,
                endDate: `${this.dateFormat(this.interval.endDate)} 23:59:59`,
            };
        },
    },
    watch: {
        calendars() {
            this.fetch();
        },
    },
    data: () => ({
        events: [],
        hovering: null,
        interval: null,
    }),
    methods: {
        fetch() {
            if (this.calendars) {
                axios.get(this.route('core.calendar.events.index'), { params: this.params })
                    .then(({ data }) => (this.events = data))
                    .catch(this.errorHandler);
            }
        },
        addEvent(event) {
            [event.startDate, event.startTime] = event.start.split(' ');
            [event.endDate, event.endTime] = event.end.split(' ');
            this.$emit('edit-event', event);
        },
        update($event) {
            axios.patch(
                this.route('core.calendar.events.update', { event: $event.id }),
                { ends_time_at: this.timeFormat($event.end) },
            ).then(({ data }) => {
                this.$toastr.success(data.message);
                this.fetch();
            }).catch(this.errorHandler);
        },
        updateInterval(interval) {
            this.interval = interval;
            this.fetch();
        },
        selectEvent(event, e) {
            if (event.route) {
                this.$router.push(event.route);
                return;
            }

            if (!event.readonly) {
                this.$emit('edit-event', event);
            }

            e.stopPropagation();
        },
        destroy($event) {
            axios.delete(
                this.route('core.calendar.events.destroy', { event: $event.id }),
            ).then(() => (this.fetch())).catch(this.errorHandler);
        },
        dateTimeFormat(daysCount, date) {
            return daysCount > 1
                ? format(date, 'm-d h:i')
                : format(date, 'h:i');
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

<style lang="scss">
    .vuecal__cell-content {align-self: flex-start;}
    .vuecal__cell-date {text-align: right;padding: 4px;}

    .vuecal--week-view .vuecal__bg .vuecal__event--all-day.love,
    .vuecal--day-view .vuecal__bg .vuecal__event--all-day.love {right: 50%;}
    .vuecal--week-view .vuecal__bg .vuecal__event--all-day.leisure,
    .vuecal--day-view .vuecal__bg .vuecal__event--all-day.leisure {left: 50%;}

    .calendar-wrapper {
        .vuecal {
            border-radius: inherit;

            .vuecal__cell:hover {
                cursor: pointer;
            }
        }
        .vuecal__event {
            .event-body {
                white-space: pre;
            }
        }


        .vuecal__event-resize-handle {
                &:after {
                    top: 5px;
                    left: calc(50% - 10px);
                }
                &:before {
                    top: 10px;
                    left: calc(50% - 10px);
                }
                &:after, &:before {
                    display: block;
                    content: "";
                    position: absolute;
                    width: 20px;
                    height: 2px;
                    transition-timing-function: ease;
                    transition-duration: .15s;
                    transition-property: transform;
                    border-radius: 4px;
                    background-color: #fff;
                }
            }
        .vuecal__time-column {
            height: auto;
        }
    }
</style>
