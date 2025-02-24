<template>
    <button v-ripple :class="buttonClass" type="button" :aria-label="defaultAriaLabel" :disabled="disabled">
        <slot></slot>
        <template v-if="!$slots.default">
            <slot v-if="loading" name="loadingicon" :class="iconStyleClass">
                <component :is="loadingIcon ? 'span' : 'SpinnerIcon'" :class="iconStyleClass" spin />
            </slot>
            <slot v-else name="icon" :class="iconStyleClass">
                <span v-if="icon" :class="iconStyleClass"></span>
            </slot>
            <span class="p-button-label">{{ label || '&nbsp;' }}</span>
            <span v-if="badge" :class="badgeStyleClass">{{ badge }}</span>
        </template>
    </button>
</template>

<script>
import SpinnerIcon from 'primevue/icon/spinner';
import Ripple from 'primevue/ripple';

export default {
    name: 'Button',
    props: {
        label: {
            type: String,
            default: null
        },
        icon: {
            type: String,
            default: null
        },
        iconPos: {
            type: String,
            default: 'left'
        },
        iconClass: {
            type: String,
            default: null
        },
        badge: {
            type: String,
            default: null
        },
        badgeClass: {
            type: String,
            default: null
        },
        loading: {
            type: Boolean,
            default: false
        },
        loadingIcon: {
            type: String,
            default: undefined
        },
        link: {
            type: Boolean,
            default: false
        },
        severity: {
            type: String,
            default: null
        },
        raised: {
            type: Boolean,
            default: false
        },
        rounded: {
            type: Boolean,
            default: false
        },
        text: {
            type: Boolean,
            default: false
        },
        outlined: {
            type: Boolean,
            default: false
        },
        size: {
            type: String,
            default: null
        },
        plain: {
            type: Boolean,
            default: false
        }
    },
    computed: {
        buttonClass() {
            return [
                'p-button p-component',
                {
                    'p-button-icon-only': this.hasIcon && !this.label,
                    'p-button-vertical': (this.iconPos === 'top' || this.iconPos === 'bottom') && this.label,
                    'p-disabled': this.$attrs.disabled || this.loading,
                    'p-button-loading': this.loading,
                    'p-button-loading-label-only': this.loading && !this.hasIcon && this.label,
                    'p-button-link': this.link,
                    [`p-button-${this.severity}`]: this.severity,
                    'p-button-raised': this.raised,
                    'p-button-rounded': this.rounded,
                    'p-button-text': this.text,
                    'p-button-outlined': this.outlined,
                    'p-button-sm': this.size === 'small',
                    'p-button-lg': this.size === 'large',
                    'p-button-plain': this.plain
                }
            ];
        },
        iconStyleClass() {
            return [
                this.loading ? 'p-button-loading-icon ' + (this.loadingIcon || '') : this.icon,
                'p-button-icon',
                this.iconClass,
                {
                    'p-button-icon-left': this.iconPos === 'left' && this.label,
                    'p-button-icon-right': this.iconPos === 'right' && this.label,
                    'p-button-icon-top': this.iconPos === 'top' && this.label,
                    'p-button-icon-bottom': this.iconPos === 'bottom' && this.label
                }
            ];
        },
        badgeStyleClass() {
            return [
                'p-badge p-component',
                this.badgeClass,
                {
                    'p-badge-no-gutter': this.badge && String(this.badge).length === 1
                }
            ];
        },
        disabled() {
            return this.$attrs.disabled || this.loading;
        },
        defaultAriaLabel() {
            return this.label ? this.label + (this.badge ? ' ' + this.badge : '') : this.$attrs['aria-label'];
        },
        hasIcon() {
            return this.icon || this.$slots.icon;
        }
    },
    components: {
        SpinnerIcon
    },
    directives: {
        ripple: Ripple
    }
};
</script>
