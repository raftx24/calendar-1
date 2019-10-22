<template>
    <div class="calendar-wrapper box is-paddingless raises-on-hover">
        <vue-cal class="vuecal--green-theme"
            :time-from="7 * 60"
            :locale="lang"
            :selected-date="selectedDate"
            :events="events"
            show-all-day-events
            events-count-on-year-view
            v-bind="$attrs"
            @event-mouse-enter="hovering = $event.id"
            @event-mouse-leave="hovering = null"
            :on-event-create="addEvent"
            editable-events
            resize-x
            v-on="$listeners">
            <template v-slot:title="{ title, view }">
                <div>
                    {{ title }}
                    <a class="button is-primary is-small is-rounded has-margin-left-large"
                        @click.stop="$emit('add-event')">
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
                                {{ dateFormat(event.daysCount,event.start) }}
                                <fa icon="arrows-alt-h"/>
                                {{ dateFormat(event.daysCount,event.end) }}
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

    inject: ['i18n'],

    props: {
        events: {
            type: Array,
            required: true,
        },
        selectedDate: {
            required: true,
        },
    },

    data: () => ({
        event: null,
        hovering: null,
    }),
    computed: {
        ...mapState(['enums']),
        ...mapGetters('preferences', ['lang']),
    },
    methods: {
        addEvent(event) {
            [event.startDate, event.startTime] = event.start.split(' ');
            [event.endDate, event.endTime] = event.end.split(' ');
            this.$emit('add-event', event);
        },
        dateFormat(daysCount, date) {
            return daysCount > 1
                ? format(date, 'm-d h:i')
                : format(date, 'h:i');
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
