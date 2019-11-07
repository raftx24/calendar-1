<template>
    <modal :show="true">
        <div class="box">
            <h5 class="subtitle is-5">
                {{ i18n("Edit recurring event") }}
            </h5>
            <div>
                <div class="field">
                    <input class="is-checkradio" id="single" type="radio"
                           name="type" checked="checked" value="single" v-model="type">
                    <label for="single">This event</label>
                </div>

                <div class="field">
                    <input class="is-checkradio" id="futures" type="radio"
                           name="type" checked="checked" value="futures" v-model="type">
                    <label for="futures">This and following events</label>
                </div>

                <div class="field" v-if="! isParent">
                    <input class="is-checkradio" id="all" type="radio"
                           name="type" checked="checked" value="all" v-model="type">
                    <label for="all">All events</label>
                </div>
            </div>
            <hr>
            <div class="level">
                <div class="level-left">
                    <div class="level-item">
                        <button class="button is-outlined" @click="$emit('cancel')">
                            {{ i18n('Cancel') }}
                        </button>
                    </div>
                </div>

                <div class="level-right">
                    <div class="level-item">
                        <button class="button is-danger has-margin-left-small"
                                @click="$emit('confirm', type)">
                            {{ i18n('Ok') }}
                        </button>
                    </div>
                </div>
            </div>
        </div>
    </modal>
</template>

<script>
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
    },
    methods: {
    },
};
</script>
