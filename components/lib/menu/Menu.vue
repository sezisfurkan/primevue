<template>
    <Portal :appendTo="appendTo" :disabled="!popup">
        <transition name="p-connected-overlay" @enter="onEnter" @leave="onLeave" @after-leave="onAfterLeave">
            <div v-if="popup ? overlayVisible : true" :ref="containerRef" :id="id" :class="containerClass" v-bind="$attrs" @click="onOverlayClick">
                <div v-if="$slots.start" class="p-menu-start">
                    <slot name="start"></slot>
                </div>
                <ul
                    :ref="listRef"
                    :id="id + '_list'"
                    class="p-menu-list p-reset"
                    role="menu"
                    :tabindex="tabindex"
                    :aria-activedescendant="focused ? focusedOptionId : undefined"
                    :aria-label="ariaLabel"
                    :aria-labelledby="ariaLabelledby"
                    @focus="onListFocus"
                    @blur="onListBlur"
                    @keydown="onListKeyDown"
                >
                    <template v-for="(item, i) of model" :key="label(item) + i.toString()">
                        <template v-if="item.items && visible(item) && !item.separator">
                            <li v-if="item.items" :id="id + '_' + i" class="p-submenu-header" role="none">
                                <slot name="item" :item="item">{{ label(item) }}</slot>
                            </li>
                            <template v-for="(child, j) of item.items" :key="child.label + i + '_' + j">
                                <PVMenuitem v-if="visible(child) && !child.separator" :id="id + '_' + i + '_' + j" :item="child" :templates="$slots" :exact="exact" :focusedOptionId="focusedOptionId" @item-click="itemClick" />
                                <li v-else-if="visible(child) && child.separator" :key="'separator' + i + j" :class="separatorClass(item)" :style="child.style" role="separator"></li>
                            </template>
                        </template>
                        <li v-else-if="visible(item) && item.separator" :key="'separator' + i.toString()" :class="separatorClass(item)" :style="item.style" role="separator"></li>
                        <PVMenuitem v-else :key="label(item) + i.toString()" :id="id + '_' + i" :item="item" :templates="$slots" :exact="exact" :focusedOptionId="focusedOptionId" @item-click="itemClick" />
                    </template>
                </ul>
                <div v-if="$slots.end" class="p-menu-end">
                    <slot name="end"></slot>
                </div>
            </div>
        </transition>
    </Portal>
</template>

<script>
import OverlayEventBus from 'primevue/overlayeventbus';
import Portal from 'primevue/portal';
import { ConnectedOverlayScrollHandler, DomHandler, UniqueComponentId, ZIndexUtils } from 'primevue/utils';
import Menuitem from './Menuitem.vue';

export default {
    name: 'Menu',
    inheritAttrs: false,
    emits: ['show', 'hide', 'focus', 'blur'],
    props: {
        popup: {
            type: Boolean,
            default: false
        },
        model: {
            type: Array,
            default: null
        },
        appendTo: {
            type: String,
            default: 'body'
        },
        autoZIndex: {
            type: Boolean,
            default: true
        },
        baseZIndex: {
            type: Number,
            default: 0
        },
        exact: {
            type: Boolean,
            default: true
        },
        tabindex: {
            type: Number,
            default: 0
        },
        'aria-label': {
            type: String,
            default: null
        },
        'aria-labelledby': {
            type: String,
            default: null
        }
    },
    data() {
        return {
            id: this.$attrs.id,
            overlayVisible: false,
            focused: false,
            focusedOptionIndex: -1,
            selectedOptionIndex: -1
        };
    },
    watch: {
        '$attrs.id': function (newValue) {
            this.id = newValue || UniqueComponentId();
        }
    },
    target: null,
    outsideClickListener: null,
    scrollHandler: null,
    resizeListener: null,
    container: null,
    list: null,
    mounted() {
        this.id = this.id || UniqueComponentId();

        if (!this.popup) {
            this.bindResizeListener();
            this.bindOutsideClickListener();
        }
    },
    beforeUnmount() {
        this.unbindResizeListener();
        this.unbindOutsideClickListener();

        if (this.scrollHandler) {
            this.scrollHandler.destroy();
            this.scrollHandler = null;
        }

        this.target = null;

        if (this.container && this.autoZIndex) {
            ZIndexUtils.clear(this.container);
        }

        this.container = null;
    },
    methods: {
        itemClick(event) {
            const item = event.item;

            if (this.disabled(item)) {
                return;
            }

            if (item.command) {
                item.command(event);
            }

            if (item.to && event.navigate) {
                event.navigate(event.originalEvent);
            }

            if (this.overlayVisible) this.hide();

            if (!this.popup && this.focusedOptionIndex !== event.id) {
                this.focusedOptionIndex = event.id;
            }
        },
        onListFocus(event) {
            this.focused = true;

            if (!this.popup) {
                if (this.selectedOptionIndex !== -1) {
                    this.changeFocusedOptionIndex(this.selectedOptionIndex);
                    this.selectedOptionIndex = -1;
                } else this.changeFocusedOptionIndex(0);
            }

            this.$emit('focus', event);
        },
        onListBlur(event) {
            this.focused = false;
            this.focusedOptionIndex = -1;
            this.$emit('blur', event);
        },
        onListKeyDown(event) {
            switch (event.code) {
                case 'ArrowDown':
                    this.onArrowDownKey(event);
                    break;

                case 'ArrowUp':
                    this.onArrowUpKey(event);
                    break;

                case 'Home':
                    this.onHomeKey(event);
                    break;

                case 'End':
                    this.onEndKey(event);
                    break;

                case 'Enter':
                    this.onEnterKey(event);
                    break;

                case 'Space':
                    this.onSpaceKey(event);
                    break;

                case 'Escape':
                    if (this.popup) {
                        DomHandler.focus(this.target);
                        this.hide();
                    }

                case 'Tab':
                    this.overlayVisible && this.hide();
                    break;

                default:
                    break;
            }
        },
        onArrowDownKey(event) {
            const optionIndex = this.findNextOptionIndex(this.focusedOptionIndex);

            this.changeFocusedOptionIndex(optionIndex);
            event.preventDefault();
        },
        onArrowUpKey(event) {
            if (event.altKey && this.popup) {
                DomHandler.focus(this.target);
                this.hide();
                event.preventDefault();
            } else {
                const optionIndex = this.findPrevOptionIndex(this.focusedOptionIndex);

                this.changeFocusedOptionIndex(optionIndex);
                event.preventDefault();
            }
        },
        onHomeKey(event) {
            this.changeFocusedOptionIndex(0);
            event.preventDefault();
        },
        onEndKey(event) {
            this.changeFocusedOptionIndex(DomHandler.find(this.container, 'li.p-menuitem:not(.p-disabled)').length - 1);
            event.preventDefault();
        },
        onEnterKey(event) {
            const element = DomHandler.findSingle(this.list, `li[id="${`${this.focusedOptionIndex}`}"]`);
            const anchorElement = element && DomHandler.findSingle(element, '.p-menuitem-link');

            this.popup && DomHandler.focus(this.target);
            anchorElement ? anchorElement.click() : element && element.click();

            event.preventDefault();
        },
        onSpaceKey(event) {
            this.onEnterKey(event);
        },
        findNextOptionIndex(index) {
            const links = DomHandler.find(this.container, 'li.p-menuitem:not(.p-disabled)');
            const matchedOptionIndex = [...links].findIndex((link) => link.id === index);

            return matchedOptionIndex > -1 ? matchedOptionIndex + 1 : 0;
        },
        findPrevOptionIndex(index) {
            const links = DomHandler.find(this.container, 'li.p-menuitem:not(.p-disabled)');
            const matchedOptionIndex = [...links].findIndex((link) => link.id === index);

            return matchedOptionIndex > -1 ? matchedOptionIndex - 1 : 0;
        },
        changeFocusedOptionIndex(index) {
            const links = DomHandler.find(this.container, 'li.p-menuitem:not(.p-disabled)');
            let order = index >= links.length ? links.length - 1 : index < 0 ? 0 : index;

            order > -1 && (this.focusedOptionIndex = links[order].getAttribute('id'));
        },
        toggle(event) {
            if (this.overlayVisible) this.hide();
            else this.show(event);
        },
        show(event) {
            this.overlayVisible = true;
            this.target = event.currentTarget;
        },
        hide() {
            this.overlayVisible = false;
            this.target = null;
        },
        onEnter(el) {
            this.alignOverlay();
            this.bindOutsideClickListener();
            this.bindResizeListener();
            this.bindScrollListener();

            if (this.autoZIndex) {
                ZIndexUtils.set('menu', el, this.baseZIndex + this.$primevue.config.zIndex.menu);
            }

            if (this.popup) {
                DomHandler.focus(this.list);
                this.changeFocusedOptionIndex(0);
            }

            this.$emit('show');
        },
        onLeave() {
            this.unbindOutsideClickListener();
            this.unbindResizeListener();
            this.unbindScrollListener();
            this.$emit('hide');
        },
        onAfterLeave(el) {
            if (this.autoZIndex) {
                ZIndexUtils.clear(el);
            }
        },
        alignOverlay() {
            DomHandler.absolutePosition(this.container, this.target);
            this.container.style.minWidth = DomHandler.getOuterWidth(this.target) + 'px';
        },
        bindOutsideClickListener() {
            if (!this.outsideClickListener) {
                this.outsideClickListener = (event) => {
                    const isOutsideContainer = this.container && !this.container.contains(event.target);
                    const isOutsideTarget = !(this.target && (this.target === event.target || this.target.contains(event.target)));

                    if (this.overlayVisible && isOutsideContainer && isOutsideTarget) {
                        this.hide();
                    } else if (!this.popup && isOutsideContainer && isOutsideTarget) {
                        this.focusedOptionIndex = -1;
                    }
                };

                document.addEventListener('click', this.outsideClickListener);
            }
        },
        unbindOutsideClickListener() {
            if (this.outsideClickListener) {
                document.removeEventListener('click', this.outsideClickListener);
                this.outsideClickListener = null;
            }
        },
        bindScrollListener() {
            if (!this.scrollHandler) {
                this.scrollHandler = new ConnectedOverlayScrollHandler(this.target, () => {
                    if (this.overlayVisible) {
                        this.hide();
                    }
                });
            }

            this.scrollHandler.bindScrollListener();
        },
        unbindScrollListener() {
            if (this.scrollHandler) {
                this.scrollHandler.unbindScrollListener();
            }
        },
        bindResizeListener() {
            if (!this.resizeListener) {
                this.resizeListener = () => {
                    if (this.overlayVisible && !DomHandler.isTouchDevice()) {
                        this.hide();
                    }
                };

                window.addEventListener('resize', this.resizeListener);
            }
        },
        unbindResizeListener() {
            if (this.resizeListener) {
                window.removeEventListener('resize', this.resizeListener);
                this.resizeListener = null;
            }
        },
        visible(item) {
            return typeof item.visible === 'function' ? item.visible() : item.visible !== false;
        },
        disabled(item) {
            return typeof item.disabled === 'function' ? item.disabled() : item.disabled;
        },
        label(item) {
            return typeof item.label === 'function' ? item.label() : item.label;
        },
        separatorClass(item) {
            return ['p-menuitem-separator', item.class];
        },
        onOverlayClick(event) {
            OverlayEventBus.emit('overlay-click', {
                originalEvent: event,
                target: this.target
            });
        },
        containerRef(el) {
            this.container = el;
        },
        listRef(el) {
            this.list = el;
        }
    },
    computed: {
        containerClass() {
            return [
                'p-menu p-component',
                {
                    'p-menu-overlay': this.popup,
                    'p-input-filled': this.$primevue.config.inputStyle === 'filled',
                    'p-ripple-disabled': this.$primevue.config.ripple === false
                }
            ];
        },
        focusedOptionId() {
            return this.focusedOptionIndex !== -1 ? this.focusedOptionIndex : null;
        }
    },
    components: {
        PVMenuitem: Menuitem,
        Portal: Portal
    }
};
</script>

<style>
.p-menu-overlay {
    position: absolute;
    top: 0;
    left: 0;
}

.p-menu ul {
    margin: 0;
    padding: 0;
    list-style: none;
}

.p-menu .p-menuitem-link {
    cursor: pointer;
    display: flex;
    align-items: center;
    text-decoration: none;
    overflow: hidden;
    position: relative;
}

.p-menu .p-menuitem-text {
    line-height: 1;
}
</style>
