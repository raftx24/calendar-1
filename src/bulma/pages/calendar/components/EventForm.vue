<template>
    <modal class="event-modal"
        show
        v-on="$listeners">
        <enso-form class="box has-background-light"
            :path="path"
            v-on="$listeners"
            ref="form"
            disable-state
            @ready="init">
            <template v-slot:starts_on="props">
                <form-field v-bind="props"
                    @input="
                        $refs.form.field('ends_on').meta.min = $event;
                        $refs.form.field('recurrence_ends_at').meta.min = $event;
                    "/>
            </template>
            <template v-slot:ends_on="props">
                <form-field v-bind="props"
                    @input="$refs.form.field('starts_on').meta.max = $event;"/>
            </template>
            <template v-slot:frequence="props">
                <form-field v-bind="props" @input="changeFrequence($event)"/>
            </template>
            <template v-slot:update_type="props">
                <form-field v-bind="props"
                    @input="$refs.form.field('update_type').meta.hidden = $event === 1"/>
            </template>
            <template v-slot:reminders="{ field }">
                <div class="field">
                    <label class="label">{{ i18n('Reminders') }}</label>
                    <div class="columns">
                        <div class="column is-3">
                            <fade>
                                <a @click="field.value.push(reminderFactory())"
                                    class="button is-small is-naked has-margin-top-medium is-pulled-right"
                                    v-if="
                                        field.value.length < 3
                                        && !field.value.some(({ remind_at }) => !remind_at)
                                    ">
                                    <span class="icon is-small">
                                        <fa icon="plus"/>
                                    </span>
                                    <span class="icon is-small">
                                        <fa icon="user-clock"/>
                                    </span>
                                </a>
                            </fade>
                        </div>
                        <div class="column">
                            <div class="columns"
                                v-for="(reminder, index) in field.value"
                                :key="index">
                                <div class="column is-9 animated fadeIn">
                                    <p class="has-margin-bottom-small">
                                        <enso-datepicker v-model="reminder.remind_at"
                                            v-bind="field.meta"/>
                                    </p>
                                </div>
                                <div class="column">
                                    <a class="button is-small is-naked has-margin-top-small"
                                        @click="field.value.splice(index, 1)">
                                        <span class="icon is-small">
                                            <fa icon="minus"/>
                                        </span>
                                    </a>
                                </div>
                            </div>
                            <div class="has-text-centered has-margin-top-medium"
                                v-if="field.value.length === 0">
                                {{ i18n('No reminders yet') }}
                            </div>
                        </div>
                    </div>
                </div>
            </template>
            <template v-slot:calendar_id="{field,errors}">
                <color-select :field="field" :errors="errors"/>
            </template>
        </enso-form>
        <event-confirmation v-if="confirm"
                            @confirm="confirm($event); confirm=null"
                            @cancel="confirm=null"/>
    </modal>
</template>

<script>
import { mapState } from 'vuex';
import {
    Modal, EnsoForm, FormField, EnsoDatepicker,
} from '@enso-ui/bulma';
import { library } from '@fortawesome/fontawesome-svg-core';
import { faUserClock, faPlus, faMinus } from '@fortawesome/free-solid-svg-icons';
import { Fade } from '@enso-ui/transitions';
import format from '@enso-ui/ui/src/modules/plugins/date-fns/format';
import EventConfirmation from './EventConfirmation.vue';
import ColorSelect from './ColorSelect.vue';

library.add(faUserClock, faPlus, faMinus);

export default {
    name: 'EventForm',

    components: {
        Modal, EnsoForm, FormField, EnsoDatepicker, Fade, EventConfirmation, ColorSelect,
    },

    inject: ['i18n', 'route'],

    props: {
        event: {
            type: Object,
            required: true,
        },
    },
    data: () => ({
        confirm: null,
    }),
    computed: {
        ...mapState(['meta']),
        isEdit() {
            return this.event.id;
        },
        path() {
            return this.isEdit
                ? this.route('core.calendar.events.edit', { event: this.event.id })
                : this.route('core.calendar.events.create');
        },
        reminderFormat() {
            return `${this.meta.dateFormat} H:i`;
        },
    },

    methods: {
        init() {
            this.$refs.form.field('starts_on').value = this.dateFormat(this.event.startDate);
            this.$refs.form.field('starts_time').value = this.event.startTime.trim();
            this.$refs.form.field('ends_on').value = this.dateFormat(this.event.endDate);
            this.$refs.form.field('ends_time').value = this.event.endTime.trim();
        },
        reminderFactory() {
            return {
                id: null,
                event_id: this.event.id,
                remind_at: null,
            };
        },
        addReminder() {
            this.$refs.form.field('reminders')
                .value.push(this.reminder());
        },
        dateFormat(date) {
            return format(date, this.meta.dateFormat);
        },
        changeFrequence(frequence) {
            console.log('this.isEdit\t', this.isEdit);
            this.$refs.form.field('recurrence_ends_at').meta.hidden = frequence === 1;
            this.$refs.form.field('update_type').meta.hidden = !this.isEdit || frequence === 1;
        },
    },
};
</script>

<style lang="scss">
    .modal.event-modal .modal-content {
        overflow: visible;
        width: 750px;
    }
</style>
