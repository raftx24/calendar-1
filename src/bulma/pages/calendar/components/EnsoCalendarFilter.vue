<template>
    <div>
        <div class="calendar-container">
            <vue-cal xsmall
                     :time="false"
                     hide-view-selector
                     default-view="month"
                     :disable-views="['years', 'year', 'week', 'day']"
                     @cell-focus="$emit('change-date',$event)"
                     class="small-calendar vuecal--green-theme vuecal--rounded-theme"/>
        </div>
        <div class="is-relative">
            <span class="is-bold">
                Calenders
            </span>
            <span class="icon is-small calendar-icon" @click="calendar = {}">
                <fa icon="plus"/>
            </span>
        </div>
        <div class="calendars">
            <div class="item has-padding-small is-relative" v-for="calendar in calendars">
                <input type="checkbox"
                       :id="'checkbox-'+calendar.id "
                       :name="'checkbox-'+calendar.id"
                       :value="calendar.id"
                       v-model="selectedCalendars"
                       @change="$emit('change-calendars', selectedCalendars)"
                       class="is-checkradio "
                       :class="'calendar-'+calendar.color">
                <label class="is-marginless" :for="'checkbox-'+calendar.id">
                    {{ calendar.name }}
                </label>
                <div class="calendar-icon">
                    <span class="icon is-small"
                          v-if="! calendar.readonly" @click="setCalendar(calendar)">
                        <fa icon="pencil-alt"/>
                    </span>
                </div>
            </div>
        </div>


        <calendar-form :calendar="calendar"
                    @submit="updateCalendar"
                    @close="calendar = null"
                    @destroy="destroy"
                    v-if="calendar"/>
    </div>
</template>

<script>
import VueCal from 'vue-cal';
import { library } from '@fortawesome/fontawesome-svg-core';
import { faPlus, faFlag, faArrowsAltH } from '@fortawesome/free-solid-svg-icons';
import 'vue-cal/dist/vuecal.css';
import CalendarForm from './CalendarForm.vue';

import('../styles/colors.scss');

library.add(faPlus, faFlag, faArrowsAltH);

export default {
    name: 'EnsoCalendarFilter',

    components: { CalendarForm, VueCal },

    inject: ['i18n', 'route'],

    data: () => ({
        calendars: [],
        calendar: null,
        selectedCalendars: [],
    }),

    created() {
        this.fetch().then(() => {
            this.calendars.forEach((calendar) => {
                this.selectedCalendars.push(calendar.id);
            });

            this.$emit('change-calendars', this.selectedCalendars);
        });
    },

    methods: {
        fetch() {
            return axios.get(this.route('core.calendar.index'))
                .then(({ data }) => {
                    this.calendars = data;
                }).catch(this.errorHandler);
        },
        setCalendar(calendar) {
            this.calendar = calendar;
        },
        updateCalendar({ calendar }) {
            this.fetch().then(() => {
                if (!this.calendar.id) {
                    this.selectedCalendars.push(calendar.id);
                }
                this.$emit('change-calendars', this.selectedCalendars);
                this.calendar = null;
            });
        },
        destroy() {
            this.fetch().then(() => {
                const index = this.selectedCalendars.findIndex(id => id === this.calendar.id);
                this.selectedCalendars.splice(index, 1);
                this.$emit('change-calendars', this.selectedCalendars);
                this.calendar = null;
            });
        },
    },
};
</script>

<style lang="scss">
    .small-calendar{
        height: 290px;
    }
    .calendar-container{
        margin-bottom: 20px;;
    }
    .calendars {
        margin-top: 10px;
    }
    .calendar-icon {
        font-size: 0.9em;
        position: absolute;
        right: 0;
        opacity: 0.6;
        top: 4px;
    }
</style>
