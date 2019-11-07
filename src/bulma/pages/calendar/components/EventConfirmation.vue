<template>
    <modal show
        v-on="$listeners">
        <div class="box">
            <h5 class="subtitle is-5">
                {{ i18n("Choose change type for the recurring event") }}
            </h5>
            <div>
                <div class="field">
                    <input class="is-checkradio"
                        v-model="type"
                        :id="enums.eventUpdateType.OnlyThisEvent"
                        type="radio"
                        name="type"
                        checked="checked"
                        :value="enums.eventUpdateType.OnlyThisEvent">
                    <label :for="enums.eventUpdateType.OnlyThisEvent">
                        {{ i18n(enums.eventUpdateType._get(enums.eventUpdateType.OnlyThisEvent)) }}
                    </label>
                </div>
                <div class="field">
                    <input class="is-checkradio"
                        v-model="type"
                        :id="enums.eventUpdateType.ThisAndFutureEvents"
                        type="radio"
                        name="type"
                        checked="checked"
                        :value="enums.eventUpdateType.ThisAndFutureEvents">
                    <label :for="enums.eventUpdateType.ThisAndFutureEvents">
                        {{ i18n(enums.eventUpdateType._get(enums.eventUpdateType.ThisAndFutureEvents)) }}
                    </label>
                </div>
                <div class="field" v-if="!isParent">
                    <input class="is-checkradio"
                        v-model="type"
                        :id="enums.eventUpdateType.All"
                        type="radio"
                        name="type"
                        checked="checked"
                        :value="enums.eventUpdateType.All">
                    <label :for="enums.eventUpdateType.All">
                        {{ i18n(enums.eventUpdateType._get(enums.eventUpdateType.All)) }}
                    </label>
                </div>
            </div>
            <hr>
            <div class="level">
                <div class="level-left">
                    <div class="level-item">
                        <button class="button"
                            @click="$emit('cancel')">
                            {{ i18n('Cancel') }}
                        </button>
                    </div>
                </div>
                <div class="level-right">
                    <div class="level-item">
                        <button class="button is-danger has-margin-left-small"
                            @click="$emit('confirm', type)">
                            {{ i18n('Confirm') }}
                        </button>
                    </div>
                </div>
            </div>
        </div>
    </modal>
</template>

<script>
import { mapState } from 'vuex';
import { library } from '@fortawesome/fontawesome-svg-core';
import { faFlag, faArrowsAltH } from '@fortawesome/free-solid-svg-icons';
import { Modal } from '@enso-ui/bulma';

import('../styles/colors.scss');

library.add(faFlag, faArrowsAltH);

export default {
    name: 'EventConfirmation',

    components: { Modal },

    inject: ['errorHandler', 'route', 'i18n'],

    props: {
        isParent: {
            type: Boolean,
            default: false,
        },
    },

    data: () => ({
        type: 'futures',
    }),

    computed: {
        ...mapState(['enums']),
    },
};
</script>
