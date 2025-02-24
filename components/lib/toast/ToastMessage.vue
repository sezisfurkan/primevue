<template>
    <div :class="containerClass" role="alert" aria-live="assertive" aria-atomic="true">
        <div class="p-toast-message-content" :class="message.contentStyleClass">
            <template v-if="!templates.message">
                <component :is="templates.icon ? templates.icon : iconComponent.name ? iconComponent : 'span'" :class="iconClass" class="p-toast-message-icon" />
                <div class="p-toast-message-text">
                    <span class="p-toast-summary">{{ message.summary }}</span>
                    <div class="p-toast-detail">{{ message.detail }}</div>
                </div>
            </template>
            <component v-else :is="templates.message" :message="message"></component>{{ message.closable }}
            <div v-if="message.closable !== false">
                <button v-ripple class="p-toast-icon-close p-link" type="button" :aria-label="closeAriaLabel" @click="onCloseClick" autofocus v-bind="closeButtonProps">
                    <component :is="templates.closeicon || 'TimesIcon'" :class="['p-toast-icon-close-icon', closeIcon]" />
                </button>
            </div>
        </div>
    </div>
</template>

<script>
import CheckIcon from 'primevue/icon/check';
import ExclamationTriangleIcon from 'primevue/icon/exclamationtriangle';
import InfoCircleIcon from 'primevue/icon/infocircle';
import TimesIcon from 'primevue/icon/times';
import TimesCircleIcon from 'primevue/icon/timescircle';
import Ripple from 'primevue/ripple';

export default {
    name: 'ToastMessage',
    emits: ['close'],
    props: {
        message: {
            type: null,
            default: null
        },
        templates: {
            type: Object,
            default: null
        },
        closeIcon: {
            type: String,
            default: null
        },
        infoIcon: {
            type: String,
            default: null
        },
        warnIcon: {
            type: String,
            default: null
        },
        errorIcon: {
            type: String,
            default: null
        },
        successIcon: {
            type: String,
            default: null
        },
        closeButtonProps: {
            type: null,
            default: null
        }
    },
    closeTimeout: null,
    mounted() {
        if (this.message.life) {
            this.closeTimeout = setTimeout(() => {
                this.close({ message: this.message, type: 'life-end' });
            }, this.message.life);
        }
    },
    beforeUnmount() {
        this.clearCloseTimeout();
    },
    methods: {
        close(params) {
            this.$emit('close', params);
        },
        onCloseClick() {
            this.clearCloseTimeout();
            this.close({ message: this.message, type: 'close' });
        },
        clearCloseTimeout() {
            if (this.closeTimeout) {
                clearTimeout(this.closeTimeout);
                this.closeTimeout = null;
            }
        }
    },
    computed: {
        containerClass() {
            return [
                'p-toast-message',
                this.message.styleClass,
                {
                    'p-toast-message-info': this.message.severity === 'info',
                    'p-toast-message-warn': this.message.severity === 'warn',
                    'p-toast-message-error': this.message.severity === 'error',
                    'p-toast-message-success': this.message.severity === 'success'
                }
            ];
        },
        iconComponent() {
            return {
                info: !this.infoIcon && InfoCircleIcon,
                success: !this.successIcon && CheckIcon,
                warn: !this.warnIcon && ExclamationTriangleIcon,
                error: !this.errorIcon && TimesCircleIcon
            }[this.message.severity];
        },
        iconClass() {
            return [
                {
                    [this.infoIcon]: this.message.severity === 'info',
                    [this.warnIcon]: this.message.severity === 'warn',
                    [this.errorIcon]: this.message.severity === 'error',
                    [this.successIcon]: this.message.severity === 'success'
                }
            ];
        },
        closeAriaLabel() {
            return this.$primevue.config.locale.aria ? this.$primevue.config.locale.aria.close : undefined;
        }
    },
    components: {
        TimesIcon: TimesIcon,
        InfoCircleIcon: InfoCircleIcon,
        CheckIcon: CheckIcon,
        ExclamationTriangleIcon: ExclamationTriangleIcon,
        TimesCircleIcon: TimesCircleIcon
    },
    directives: {
        ripple: Ripple
    }
};
</script>
