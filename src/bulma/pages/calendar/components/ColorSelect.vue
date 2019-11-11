<template>
    <div>
        <label class="label">
            {{ i18n(field.label) }}
        </label>
        <vue-select v-model="field.value"
            v-bind="field.meta"
            v-on="$listeners"
            :has-error="errors.has(field.name)"
            @input="errors.clear(field.name); $emit('changed')">
            <template v-slot:selection="{selection,errors}">
                <div>
                    <span :class="`calendar-color calendar-${selection[colorField]}`"/>
                    <span>{{ selection.name }}</span>
                </div>
            </template>
            <template v-slot:option="{option}">
                <div>
                    <span :class="`calendar-color calendar-${option[colorField]}`"/>
                    <span>{{ option.name }}</span>
                </div>
            </template>
        </vue-select>
    </div>
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
import { VueSelect } from '@enso-ui/select/bulma';
import EventConfirmation from './EventConfirmation';

library.add(faUserClock, faPlus, faMinus);

export default {
    name: 'ColorSelect',

    components: {
        VueSelect,
    },

    inject: ['i18n', 'route'],

    props: {
        field: {
            type: Object,
            required: true,
        },
        errors: {
            type: Object,
            default: {},
            required: true,
        },
        colorField: {
            type: String,
            default: 'color',
        },
    },
    data: () => ({
        ready: false,
        confirm: null,
    }),
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
</style>
