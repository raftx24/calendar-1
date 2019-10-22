<template>
    <modal class="calendar-modal"
        show
        v-on="$listeners">
        <enso-form class="box has-background-light"
            :path="path"
            v-on="$listeners"
            ref="form"
            disable-state>
            <template v-slot:color="{field,errors}">
                <div>
                    <label class="label">
                        {{ i18n(field.label) }}
                    </label>
                    <vue-select v-model="field.value"
                                v-bind="field.meta"
                                v-on="$listeners"
                                @fetch="field.meta.options = $event"
                                :has-error="errors.has(field.name)"
                                @input="errors.clear(field.name); $emit('changed')">
                        <template v-slot:selection="{selection,errors}">
                            <div>
                                <span :class="'calendar-color calendar-'+selection.name"/>
                                <span>{{ selection.name }}</span>
                            </div>
                        </template>
                        <template v-slot:option="{option}">
                            <div>
                                <span :class="'calendar-color calendar-'+option.name"/>
                                <span>{{ option.name }}</span>
                            </div>
                        </template>
                    </vue-select>
                </div>
            </template>
        </enso-form>
    </modal>
</template>

<script>
import { mapState } from 'vuex';
import { EnsoForm, Modal } from '@enso-ui/bulma';
import { library } from '@fortawesome/fontawesome-svg-core';
import { faMinus, faPlus, faUserClock } from '@fortawesome/free-solid-svg-icons';
import { VueSelect } from '@enso-ui/select/bulma';

library.add(faUserClock, faPlus, faMinus);

export default {
    name: 'CalendarForm',

    components: {
        Modal, EnsoForm, VueSelect,
    },

    inject: ['i18n', 'route'],

    props: {
        calendar: {
            type: Object,
            required: true,
        },
    },

    computed: {
        ...mapState(['meta']),
        path() {
            return this.calendar.id
                ? this.route('core.calendar.edit', { calendar: this.calendar.id })
                : this.route('core.calendar.create');
        },
    },
};
</script>

<style lang="scss">
    .calendar-color {
        width: 10px;
        height: 10px;
        border-radius: 10px;
        display: inline-block;
        margin-right: 5px;
    }
    .modal.calendar-modal .modal-content {
        overflow: visible;
    }
</style>
