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
                <a class="button is-primary is-small">
                    <span class="is-bold">
                        {{ i18n('Today') }}
                    </span>
                    <span class="icon is-small">
                        <fa icon="crosshairs"
                            size="xs"/>
                    </span>
                </a>
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

        <event-confirmation v-if="confirm"
            @confirm="confirm($event); confirm=null"
            @cancel="fetch(); confirm=null"/>
    </div>
</template>

<script>
import { mapState, mapGetters } from 'vuex';
import VueCal from 'vue-cal';
import { library } from '@fortawesome/fontawesome-svg-core';
import { faFlag, faArrowsAltH } from '@fortawesome/free-solid-svg-icons';
import format from '@enso-ui/ui/src/modules/plugins/date-fns/format';
import EventConfirmation from './EventConfirmation';

import('../styles/colors.scss');

library.add(faFlag, faArrowsAltH);

export default {
    name: 'EnsoCalendar',

    components: { VueCal, EventConfirmation },

    inject: ['errorHandler', 'route', 'i18n'],

    props: {
        selectedDate: {
            required: true,
        },
        calendars: {
            required: true,
        },
    },

    data: () => ({
        events: [],
        event: null,
        confirm: null,
        hovering: null,
        interval: null,
    }),

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

    mounted() {
        this.resize();

        window.addEventListener('resize', this.resize);
    },

    beforeDestroy() {
        window.removeEventListener('resize', this.resize);
    },

    methods: {
        resize() {
            this.$el.style.height = `${document.body.clientHeight - 170}px`;
        },
        fetch() {
            if (this.calendars) {
                axios.get(this.route('core.calendar.events.index'), { params: this.params })
                    .then(({ data }) => (this.events = data))
                    .catch(this.errorHandler);
            }
        },
        addEvent(event) {
            this.$emit('edit-event', event);
        },
        update($event, updateType) {
            if ($event.frequence === 1 || updateType !== undefined) {
                axios.patch(
                    this.route('core.calendar.events.update', { event: $event.id }),
                    { end_time: this.timeFormat($event.end), update_type: updateType },
                ).then(({ data }) => {
                    this.$toastr.success(data.message);
                    this.fetch();
                }).catch(this.errorHandler);
                return;
            }
            this.confirm = (updateType) => {
                this.update($event, updateType);
            };
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
        destroy($event, updateType) {
            if ($event.frequence === 1 || updateType !== undefined) {
                axios.delete(
                    this.route(
                        'core.calendar.events.destroy',
                        { event: $event.id, updateType: updateType || 'single' },
                    ),
                ).then(() => (this.fetch())).catch(this.errorHandler);
                return;
            }

            this.confirm = (updateType) => {
                this.destroy($event, updateType);
            };
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
    .calendar-wrapper {
        height: 100%;

        .vuecal {
            border-radius: inherit;

            .vuecal__body {
                overflow: auto;

                .vuecal__bg {
                    overflow: visible;
                }
            }

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

            .vuecal__time-cell .label {
                color: inherit;
                display: inherit;
                font-size: inherit;
                font-weight: inherit;
            }
        }
    }
</style>
