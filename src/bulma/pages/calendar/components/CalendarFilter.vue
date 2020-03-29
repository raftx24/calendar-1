<template>
    <div class="wrapper">
        <calendar-form :calendar="calendar"
            @submit="updateCalendar"
            @close="calendar = null"
            @destroy="destroy"
            v-if="calendar"/>
        <div class="box has-padding-small raises-on-hover">
            <vue-cal class="small-calendar is-paddingless"
                xsmall
                today-button
                :locale="lang"
                :time="false"
                hide-view-selector
                default-view="month"
                :disable-views="['years', 'year', 'week', 'day']"
                @cell-focus="$emit('change-date', $event)">
                <template v-slot:today-button>
                    <span class="icon is-small is-clickable is-naked"
                        @click="$emit('change-date', new Date())">
                        <fa icon="crosshairs"
                            size="xs"/>
                    </span>
                </template>
            </vue-cal>
        </div>
        <div class="level is-marginless">
            <div class="level-left">
                <div class="level-item">
                    <label class="label">
                        {{ i18n('Calendars') }}
                    </label>
                </div>
            </div>
            <div class="level-right">
                <div class="level-item">
                    <a class="button is-naked">
                        <span class="icon"
                            @click="calendar = {}">
                            <fa icon="plus"/>
                        </span>
                    </a>
                </div>
            </div>
        </div>
        <div class="level is-marginless calendar-item"
            v-for="calendar in calendars"
            :key="calendar.id">
            <div class="level-left">
                <div class="level-item">
                    <label class="checkbox">
                        <input class="is-hidden"
                            v-model="selected"
                            type="checkbox"
                            :value="calendar.id"
                            @change="updateSelection">
                        <span class="calendar-color"
                            :class="`calendar-${selected.includes(calendar.id) ? calendar.color : 'gray'}`"/>
                            {{ calendar.name }}
                    </label>
                </div>
            </div>
            <div class="level-right">
                <div class="level-item">
                    <a class="button is-naked">
                        <span class="icon"
                            @click="setCalendar(calendar)"
                            v-if="!calendar.readonly">
                            <fa icon="pencil-alt"/>
                        </span>
                    </a>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
import { mapGetters } from 'vuex';
import VueCal from 'vue-cal';
import { library } from '@fortawesome/fontawesome-svg-core';
import {
    faPlus, faFlag, faArrowsAltH, faCrosshairs,
} from '@fortawesome/free-solid-svg-icons';
import 'vue-cal/dist/vuecal.css';
import CalendarForm from './CalendarForm.vue';

import('../styles/colors.scss');

library.add(faPlus, faFlag, faArrowsAltH, faCrosshairs);

export default {
    name: 'CalendarFilter',

    components: { CalendarForm, VueCal },

    inject: ['errorHandler', 'i18n', 'route'],

    data: () => ({
        calendars: [],
        calendar: null,
        selected: [],
    }),

    created() {
        this.fetch().then(this.selectAll);
    },

    computed: {
        ...mapGetters('preferences', ['lang']),
    },

    methods: {
        fetch() {
            return axios.get(this.route('core.calendar.index'))
                .then(({ data }) => {
                    this.calendars = data;
                }).catch(this.errorHandler);
        },
        updateSelection() {
            this.$emit('update-selection', this.selected);
        },
        selectAll() {
            this.selected = this.calendars.map(({ id }) => id);
            this.updateSelection();
        },
        setCalendar(calendar) {
            this.calendar = calendar;
        },
        updateCalendar({ calendar }) {
            this.fetch().then(() => {
                if (!this.calendar.id) {
                    this.selected.push(calendar.id);
                }

                this.calendar = null;
                this.updateSelection();
            });
        },
        destroy() {
            this.fetch().then(() => {
                const index = this.selected
                    .findIndex(id => id === this.calendar.id);

                this.selected.splice(index, 1);
                this.calendar = null;
                this.updateSelection();
            });
        },
    },
};
</script>

<style lang="scss">
    .small-calendar {
        height: 290px;
    }

    .calendar-item .level-left {
        flex-shrink: 1;
        overflow: hidden;
    }
</style>
