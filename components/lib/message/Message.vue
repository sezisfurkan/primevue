<template>
    <transition name="p-message" appear>
        <div v-show="visible" :class="containerClass" role="alert" aria-live="assertive" aria-atomic="true">
            <div class="p-message-wrapper">
                <slot name="messageicon">
                    <component :is="icon ? 'span' : iconComponent" :class="['p-message-icon', icon]"></component>
                </slot>
                <div class="p-message-text">
                    <slot></slot>
                </div>
                <button v-if="closable" v-ripple class="p-message-close p-link" :aria-label="closeAriaLabel" type="button" @click="close($event)" v-bind="closeButtonProps">
                    <slot name="closeicon">
                        <component :is="closeIcon ? 'i' : 'TimesIcon'" :class="['p-message-close-icon', closeIcon]"></component>
                    </slot>
                </button>
            </div>
        </div>
    </transition>
</template>

<script>
import CheckIcon from 'primevue/icon/check';
import ExclamationTriangleIcon from 'primevue/icon/exclamationtriangle';
import InfoCircleIcon from 'primevue/icon/infocircle';
import TimesIcon from 'primevue/icon/times';
import TimesCircleIcon from 'primevue/icon/timescircle';
import Ripple from 'primevue/ripple';

export default {
    name: 'Message',
    emits: ['close'],
    props: {
        severity: {
            type: String,
            default: 'info'
        },
        closable: {
            type: Boolean,
            default: true
        },
        sticky: {
            type: Boolean,
            default: true
        },
        life: {
            type: Number,
            default: 3000
        },
        icon: {
            type: String,
            default: undefined
        },
        closeIcon: {
            type: String,
            default: undefined
        },
        closeButtonProps: {
            type: null,
            default: null
        }
    },
    timeout: null,
    data() {
        return {
            visible: true
        };
    },
    mounted() {
        if (!this.sticky) {
            this.closeAfterDelay();
        }
    },
    methods: {
        close(event) {
            this.visible = false;
            this.$emit('close', event);
        },
        closeAfterDelay() {
            setTimeout(() => {
                this.visible = false;
            }, this.life);
        }
    },
    computed: {
        containerClass() {
            return 'p-message p-component p-message-' + this.severity;
        },
        iconComponent() {
            return {
                info: InfoCircleIcon,
                success: CheckIcon,
                warn: ExclamationTriangleIcon,
                error: TimesCircleIcon
            }[this.severity];
        },
        closeAriaLabel() {
            return this.$primevue.config.locale.aria ? this.$primevue.config.locale.aria.close : undefined;
        }
    },
    directives: {
        ripple: Ripple
    },
    components: {
        TimesIcon,
        InfoCircleIcon,
        CheckIcon,
        ExclamationTriangleIcon,
        TimesCircleIcon
    }
};
</script>

<style>
.p-message-wrapper {
    display: flex;
    align-items: center;
}

.p-message-close {
    display: flex;
    align-items: center;
    justify-content: center;
}

.p-message-close.p-link {
    margin-left: auto;
    overflow: hidden;
    position: relative;
}

.p-message-enter-from {
    opacity: 0;
}

.p-message-enter-active {
    transition: opacity 0.3s;
}

.p-message.p-message-leave-from {
    max-height: 1000px;
}

.p-message.p-message-leave-to {
    max-height: 0;
    opacity: 0;
    margin: 0 !important;
}

.p-message-leave-active {
    overflow: hidden;
    transition: max-height 0.3s cubic-bezier(0, 1, 0, 1), opacity 0.3s, margin 0.15s;
}

.p-message-leave-active .p-message-close {
    display: none;
}
</style>
