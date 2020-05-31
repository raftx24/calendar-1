<template>
    <modal show
        v-on="$listeners">
        <div class="box">
            <h5 class="subtitle is-5">
                {{ i18n("Update") }}
            </h5>
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
                        <button class="button is-success"
                            @click="$emit('confirm', update.OnlyThis)">
                            {{ i18n(update._get(update.OnlyThis)) }}
                        </button>
                        <button class="button is-warning has-margin-left-medium"
                            @click="$emit('confirm', update.ThisAndFuture)"
                            v-if="!event.isLast">
                            {{ i18n(update._get(update.ThisAndFuture)) }}
                        </button>
                        <button class="button is-danger has-margin-left-medium"
                            @click="$emit('confirm', update.All)"
                            v-if="event.parentId">
                            {{ i18n(update._get(update.All)) }}
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
import { Modal } from '@enso-ui/modal/bulma';

import('../styles/colors.scss');

library.add(faFlag, faArrowsAltH);

export default {
    name: 'EventConfirmation',

    components: { Modal },

    inject: ['i18n'],

    props: {
        event: {
            type: Object,
            required: true,
        },
    },

    computed: {
        ...mapState(['enums']),
        update() {
            return this.enums.eventUpdateType;
        },
    },
};
</script>
