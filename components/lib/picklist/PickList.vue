<template>
    <div :class="containerClass">
        <div v-if="showSourceControls" class="p-picklist-buttons p-picklist-source-controls">
            <slot name="sourcecontrolsstart"></slot>
            <PLButton :aria-label="moveUpAriaLabel" :disabled="moveDisabled(0)" type="button" @click="moveUp($event, 0)" v-bind="moveUpButtonProps">
                <template #icon>
                    <slot name="moveupicon">
                        <AngleUpIcon />
                    </slot>
                </template>
            </PLButton>
            <PLButton :aria-label="moveTopAriaLabel" :disabled="moveDisabled(0)" type="button" @click="moveTop($event, 0)" v-bind="moveTopButtonProps">
                <template #icon>
                    <slot name="movetopicon">
                        <AngleDoubleUpIcon />
                    </slot>
                </template>
            </PLButton>
            <PLButton :aria-label="moveDownAriaLabel" :disabled="moveDisabled(0)" type="button" @click="moveDown($event, 0)" v-bind="moveDownButtonProps">
                <template #icon>
                    <slot name="movedownicon">
                        <AngleDownIcon />
                    </slot>
                </template>
            </PLButton>
            <PLButton :aria-label="moveBottomAriaLabel" :disabled="moveDisabled(0)" type="button" @click="moveBottom($event, 0)" v-bind="moveBottomButtonProps">
                <template #icon>
                    <slot name="movebottomicon">
                        <AngleDoubleDownIcon />
                    </slot>
                </template>
            </PLButton>
            <slot name="sourcecontrolsend"></slot>
        </div>
        <div class="p-picklist-list-wrapper p-picklist-source-wrapper">
            <div v-if="$slots.sourceheader" class="p-picklist-header">
                <slot name="sourceheader"></slot>
            </div>
            <transition-group
                ref="sourceList"
                :id="idSource + '_list'"
                name="p-picklist-flip"
                tag="ul"
                class="p-picklist-list p-picklist-source"
                :style="listStyle"
                role="listbox"
                aria-multiselectable="true"
                :aria-activedescendant="focused['sourceList'] ? focusedOptionId : undefined"
                :tabindex="sourceList && sourceList.length > 0 ? tabindex : -1"
                @focus="onListFocus($event, 'sourceList')"
                @blur="onListBlur($event, 'sourceList')"
                @keydown="onItemKeyDown($event, 'sourceList')"
                v-bind="sourceListProps"
            >
                <template v-for="(item, i) of sourceList" :key="getItemKey(item, i)">
                    <li
                        :id="idSource + '_' + i"
                        v-ripple
                        :class="itemClass(item, `${idSource}_${i}`, 0)"
                        @click="onItemClick($event, item, i, 0)"
                        @dblclick="onItemDblClick($event, item, 0)"
                        @touchend="onItemTouchEnd"
                        @mousedown="onOptionMouseDown(i, 'sourceList')"
                        role="option"
                        :aria-selected="isSelected(item, 0)"
                    >
                        <slot name="item" :item="item" :index="i"> </slot>
                    </li>
                </template>
            </transition-group>
        </div>
        <div class="p-picklist-buttons p-picklist-transfer-buttons">
            <slot name="movecontrolsstart"></slot>
            <PLButton :aria-label="moveToTargetAriaLabel" type="button" @click="moveToTarget" :disabled="moveDisabled(0)" v-bind="moveToTargetProps">
                <template #icon>
                    <slot name="movetotargeticon">
                        <AngleRightIcon />
                    </slot>
                </template>
            </PLButton>
            <PLButton :aria-label="moveAllToTargetAriaLabel" type="button" @click="moveAllToTarget" :disabled="moveAllDisabled('sourceList')" v-bind="moveAllToTargetProps">
                <template #icon>
                    <slot name="movealltotargeticon">
                        <AngleDoubleRightIcon />
                    </slot>
                </template>
            </PLButton>
            <PLButton :aria-label="moveToSourceAriaLabel" type="button" @click="moveToSource" :disabled="moveDisabled(1)" v-bind="moveToSourceProps">
                <template #icon>
                    <slot name="movetosourceicon">
                        <AngleLeftIcon />
                    </slot>
                </template>
            </PLButton>
            <PLButton :aria-label="moveAllToSourceAriaLabel" type="button" @click="moveAllToSource" :disabled="moveSourceDisabled('targetList')" v-bind="moveAllToSourceProps">
                <template #icon>
                    <slot name="movealltosourceicon">
                        <AngleDoubleLeftIcon />
                    </slot>
                </template>
            </PLButton>
            <slot name="movecontrolsend"></slot>
        </div>
        <div class="p-picklist-list-wrapper p-picklist-target-wrapper">
            <div v-if="$slots.targetheader" class="p-picklist-header">
                <slot name="targetheader"></slot>
            </div>
            <transition-group
                ref="targetList"
                :id="idTarget + '_list'"
                name="p-picklist-flip"
                tag="ul"
                class="p-picklist-list p-picklist-target"
                :style="listStyle"
                role="listbox"
                aria-multiselectable="true"
                :aria-activedescendant="focused['targetList'] ? focusedOptionId : undefined"
                :tabindex="targetList && targetList.length > 0 ? tabindex : -1"
                @focus="onListFocus($event, 'targetList')"
                @blur="onListBlur($event, 'targetList')"
                @keydown="onItemKeyDown($event, 'targetList')"
                v-bind="targetListProps"
            >
                <template v-for="(item, i) of targetList" :key="getItemKey(item, i)">
                    <li
                        :id="idTarget + '_' + i"
                        v-ripple
                        :class="itemClass(item, `${idTarget}_${i}`, 1)"
                        @click="onItemClick($event, item, i, 1)"
                        @dblclick="onItemDblClick($event, item, 1)"
                        @keydown="onItemKeyDown($event, 'targetList')"
                        @mousedown="onOptionMouseDown(i, 'targetList')"
                        @touchend="onItemTouchEnd"
                        role="option"
                        :aria-selected="isSelected(item, 1)"
                    >
                        <slot name="item" :item="item" :index="i"> </slot>
                    </li>
                </template>
            </transition-group>
        </div>
        <div v-if="showTargetControls" class="p-picklist-buttons p-picklist-target-controls">
            <slot name="targetcontrolsstart"></slot>
            <PLButton :aria-label="moveUpAriaLabel" :disabled="moveDisabled(1)" type="button" @click="moveUp($event, 1)" v-bind="moveUpButtonProps">
                <template #icon>
                    <slot name="moveupicon">
                        <AngleUpIcon />
                    </slot>
                </template>
            </PLButton>
            <PLButton :aria-label="moveTopAriaLabel" :disabled="moveDisabled(1)" type="button" @click="moveTop($event, 1)" v-bind="moveTopButtonProps">
                <template #icon>
                    <slot name="movetopicon">
                        <AngleDoubleUpIcon />
                    </slot>
                </template>
            </PLButton>
            <PLButton :aria-label="moveDownAriaLabel" :disabled="moveDisabled(1)" type="button" @click="moveDown($event, 1)" v-bind="moveDownButtonProps">
                <template #icon>
                    <slot name="movedownicon">
                        <AngleDownIcon />
                    </slot>
                </template>
            </PLButton>
            <PLButton :aria-label="moveBottomAriaLabel" :disabled="moveDisabled(1)" type="button" @click="moveBottom($event, 1)" v-bind="moveBottomButtonProps">
                <template #icon>
                    <slot name="movebottomicon">
                        <AngleDoubleDownIcon />
                    </slot>
                </template>
            </PLButton>
            <slot name="targetcontrolsend"></slot>
        </div>
    </div>
</template>

<script>
import Button from 'primevue/button';
import AngleDoubleDownIcon from 'primevue/icon/angledoubledown';
import AngleDoubleLeftIcon from 'primevue/icon/angledoubleleft';
import AngleDoubleRightIcon from 'primevue/icon/angledoubleright';
import AngleDoubleUpIcon from 'primevue/icon/angledoubleup';
import AngleDownIcon from 'primevue/icon/angledown';
import AngleLeftIcon from 'primevue/icon/angleleft';
import AngleRightIcon from 'primevue/icon/angleright';
import AngleUpIcon from 'primevue/icon/angleup';
import Ripple from 'primevue/ripple';
import { DomHandler, ObjectUtils, UniqueComponentId } from 'primevue/utils';

export default {
    name: 'PickList',
    emits: ['update:modelValue', 'reorder', 'update:selection', 'selection-change', 'move-to-target', 'move-to-source', 'move-all-to-target', 'move-all-to-source', 'focus', 'blur'],
    props: {
        modelValue: {
            type: Array,
            default: () => [[], []]
        },
        selection: {
            type: Array,
            default: () => [[], []]
        },
        dataKey: {
            type: String,
            default: null
        },
        listStyle: {
            type: null,
            default: null
        },
        metaKeySelection: {
            type: Boolean,
            default: true
        },
        responsive: {
            type: Boolean,
            default: true
        },
        breakpoint: {
            type: String,
            default: '960px'
        },
        stripedRows: {
            type: Boolean,
            default: false
        },
        showSourceControls: {
            type: Boolean,
            default: true
        },
        showTargetControls: {
            type: Boolean,
            default: true
        },
        targetListProps: {
            type: null,
            default: null
        },
        sourceListProps: {
            type: null,
            default: null
        },
        moveUpButtonProps: {
            type: null,
            default: null
        },
        moveTopButtonProps: {
            type: null,
            default: null
        },
        moveDownButtonProps: {
            type: null,
            default: null
        },
        moveBottomButtonProps: {
            type: null,
            default: null
        },
        moveToTargetProps: {
            type: null,
            default: null
        },
        moveAllToTargetProps: {
            type: null,
            default: null
        },
        moveToSourceProps: {
            type: null,
            default: null
        },
        moveAllToSourceProps: {
            type: null,
            default: null
        },
        tabindex: {
            type: Number,
            default: 0
        }
    },
    itemTouched: false,
    reorderDirection: null,
    styleElement: null,
    data() {
        return {
            id: this.$attrs.id,
            d_selection: this.selection,
            focused: {
                sourceList: false,
                targetList: false
            },
            focusedOptionIndex: -1
        };
    },
    watch: {
        '$attrs.id': function (newValue) {
            this.id = newValue || UniqueComponentId();
        },
        selection(newValue) {
            this.d_selection = newValue;
        }
    },
    updated() {
        if (this.reorderDirection) {
            this.updateListScroll(this.$refs.sourceList.$el);
            this.updateListScroll(this.$refs.targetList.$el);
            this.reorderDirection = null;
        }
    },
    beforeUnmount() {
        this.destroyStyle();
    },
    mounted() {
        this.id = this.id || UniqueComponentId();

        if (this.responsive) {
            this.createStyle();
        }
    },
    methods: {
        getItemKey(item, index) {
            return this.dataKey ? ObjectUtils.resolveFieldData(item, this.dataKey) : index;
        },
        isSelected(item, listIndex) {
            return ObjectUtils.findIndexInList(item, this.d_selection[listIndex]) != -1;
        },
        onListFocus(event, listType) {
            const selectedFirstItem = DomHandler.findSingle(this.$refs[listType].$el, 'li.p-picklist-item.p-highlight');
            const findIndex = ObjectUtils.findIndexInList(selectedFirstItem, this.$refs[listType].$el.children);

            this.focused[listType] = true;

            const index = this.focusedOptionIndex !== -1 ? this.focusedOptionIndex : selectedFirstItem ? findIndex : -1;

            this.changeFocusedOptionIndex(index, listType);
            this.$emit('focus', event);
        },
        onListBlur(event, listType) {
            this.focused[listType] = false;
            this.focusedOptionIndex = -1;
            this.$emit('blur', event);
        },
        onOptionMouseDown(index, listType) {
            this.focused[listType] = true;
            this.focusedOptionIndex = index;
        },
        moveUp(event, listIndex) {
            if (this.d_selection && this.d_selection[listIndex]) {
                let valueList = [...this.modelValue[listIndex]];
                let selectionList = this.d_selection[listIndex];

                for (let i = 0; i < selectionList.length; i++) {
                    let selectedItem = selectionList[i];
                    let selectedItemIndex = ObjectUtils.findIndexInList(selectedItem, valueList);

                    if (selectedItemIndex !== 0) {
                        let movedItem = valueList[selectedItemIndex];
                        let temp = valueList[selectedItemIndex - 1];

                        valueList[selectedItemIndex - 1] = movedItem;
                        valueList[selectedItemIndex] = temp;
                    } else {
                        break;
                    }
                }

                let value = [...this.modelValue];

                value[listIndex] = valueList;

                this.reorderDirection = 'up';
                this.$emit('update:modelValue', value);
                this.$emit('reorder', {
                    originalEvent: event,
                    value: value,
                    direction: this.reorderDirection,
                    listIndex: listIndex
                });
            }
        },
        moveTop(event, listIndex) {
            if (this.d_selection) {
                let valueList = [...this.modelValue[listIndex]];
                let selectionList = this.d_selection[listIndex];

                for (let i = 0; i < selectionList.length; i++) {
                    let selectedItem = selectionList[i];
                    let selectedItemIndex = ObjectUtils.findIndexInList(selectedItem, valueList);

                    if (selectedItemIndex !== 0) {
                        let movedItem = valueList.splice(selectedItemIndex, 1)[0];

                        valueList.unshift(movedItem);
                    } else {
                        break;
                    }
                }

                let value = [...this.modelValue];

                value[listIndex] = valueList;

                this.reorderDirection = 'top';
                this.$emit('update:modelValue', value);
                this.$emit('reorder', {
                    originalEvent: event,
                    value: value,
                    direction: this.reorderDirection,
                    listIndex: listIndex
                });
            }
        },
        moveDown(event, listIndex) {
            if (this.d_selection) {
                let valueList = [...this.modelValue[listIndex]];
                let selectionList = this.d_selection[listIndex];

                for (let i = selectionList.length - 1; i >= 0; i--) {
                    let selectedItem = selectionList[i];
                    let selectedItemIndex = ObjectUtils.findIndexInList(selectedItem, valueList);

                    if (selectedItemIndex !== valueList.length - 1) {
                        let movedItem = valueList[selectedItemIndex];
                        let temp = valueList[selectedItemIndex + 1];

                        valueList[selectedItemIndex + 1] = movedItem;
                        valueList[selectedItemIndex] = temp;
                    } else {
                        break;
                    }
                }

                let value = [...this.modelValue];

                value[listIndex] = valueList;

                this.reorderDirection = 'down';
                this.$emit('update:modelValue', value);
                this.$emit('reorder', {
                    originalEvent: event,
                    value: value,
                    direction: this.reorderDirection,
                    listIndex: listIndex
                });
            }
        },
        moveBottom(event, listIndex) {
            if (this.d_selection) {
                let valueList = [...this.modelValue[listIndex]];
                let selectionList = this.d_selection[listIndex];

                for (let i = selectionList.length - 1; i >= 0; i--) {
                    let selectedItem = selectionList[i];
                    let selectedItemIndex = ObjectUtils.findIndexInList(selectedItem, valueList);

                    if (selectedItemIndex !== valueList.length - 1) {
                        let movedItem = valueList.splice(selectedItemIndex, 1)[0];

                        valueList.push(movedItem);
                    } else {
                        break;
                    }
                }

                let value = [...this.modelValue];

                value[listIndex] = valueList;

                this.reorderDirection = 'bottom';
                this.$emit('update:modelValue', value);
                this.$emit('reorder', {
                    originalEvent: event,
                    value: value,
                    direction: this.reorderDirection,
                    listIndex: listIndex
                });
            }
        },
        moveToTarget(event) {
            let selection = this.d_selection && this.d_selection[0] ? this.d_selection[0] : null;
            let sourceList = [...this.modelValue[0]];
            let targetList = [...this.modelValue[1]];

            if (selection) {
                for (let i = 0; i < selection.length; i++) {
                    let selectedItem = selection[i];

                    if (ObjectUtils.findIndexInList(selectedItem, targetList) == -1) {
                        targetList.push(sourceList.splice(ObjectUtils.findIndexInList(selectedItem, sourceList), 1)[0]);
                    }
                }

                let value = [...this.modelValue];

                value[0] = sourceList;
                value[1] = targetList;
                this.$emit('update:modelValue', value);

                this.$emit('move-to-target', {
                    originalEvent: event,
                    items: selection
                });

                this.d_selection[0] = [];
                this.$emit('update:selection', this.d_selection);
                this.$emit('selection-change', {
                    originalEvent: event,
                    value: this.d_selection
                });
            }
        },
        moveAllToTarget(event) {
            if (this.modelValue[0]) {
                let sourceList = [...this.modelValue[0]];
                let targetList = [...this.modelValue[1]];

                this.$emit('move-all-to-target', {
                    originalEvent: event,
                    items: sourceList
                });

                targetList = [...targetList, ...sourceList];
                sourceList = [];

                let value = [...this.modelValue];

                value[0] = sourceList;
                value[1] = targetList;
                this.$emit('update:modelValue', value);

                this.d_selection[0] = [];
                this.$emit('update:selection', this.d_selection);
                this.$emit('selection-change', {
                    originalEvent: event,
                    value: this.d_selection
                });
            }
        },
        moveToSource(event) {
            let selection = this.d_selection && this.d_selection[1] ? this.d_selection[1] : null;
            let sourceList = [...this.modelValue[0]];
            let targetList = [...this.modelValue[1]];

            if (selection) {
                for (let i = 0; i < selection.length; i++) {
                    let selectedItem = selection[i];

                    if (ObjectUtils.findIndexInList(selectedItem, sourceList) == -1) {
                        sourceList.push(targetList.splice(ObjectUtils.findIndexInList(selectedItem, targetList), 1)[0]);
                    }
                }

                let value = [...this.modelValue];

                value[0] = sourceList;
                value[1] = targetList;
                this.$emit('update:modelValue', value);

                this.$emit('move-to-source', {
                    originalEvent: event,
                    items: selection
                });

                this.d_selection[1] = [];
                this.$emit('update:selection', this.d_selection);
                this.$emit('selection-change', {
                    originalEvent: event,
                    value: this.d_selection
                });
            }
        },
        moveAllToSource(event) {
            if (this.modelValue[1]) {
                let sourceList = [...this.modelValue[0]];
                let targetList = [...this.modelValue[1]];

                this.$emit('move-all-to-source', {
                    originalEvent: event,
                    items: targetList
                });

                sourceList = [...sourceList, ...targetList];
                targetList = [];

                let value = [...this.modelValue];

                value[0] = sourceList;
                value[1] = targetList;
                this.$emit('update:modelValue', value);

                this.d_selection[1] = [];
                this.$emit('update:selection', this.d_selection);
                this.$emit('selection-change', {
                    originalEvent: event,
                    value: this.d_selection
                });
            }
        },
        onItemClick(event, item, index, listIndex) {
            const listType = listIndex === 0 ? 'sourceList' : 'targetList';

            this.itemTouched = false;
            const selectionList = this.d_selection[listIndex];
            const selectedIndex = ObjectUtils.findIndexInList(item, this.d_selection);
            const selected = selectedIndex != -1;
            const metaSelection = this.itemTouched ? false : this.metaKeySelection;
            const selectedId = DomHandler.find(this.$refs[listType].$el, '.p-picklist-item')[index].getAttribute('id');

            this.focusedOptionIndex = selectedId;
            let _selection;

            if (metaSelection) {
                let metaKey = event.metaKey || event.ctrlKey;

                if (selected && metaKey) {
                    _selection = selectionList.filter((val, index) => index !== selectedIndex);
                } else {
                    _selection = metaKey ? (selectionList ? [...selectionList] : []) : [];
                    _selection.push(item);
                }
            } else {
                if (selected) {
                    _selection = selectionList.filter((val, index) => index !== selectedIndex);
                } else {
                    _selection = selectionList ? [...selectionList] : [];
                    _selection.push(item);
                }
            }

            let newSelection = [...this.d_selection];

            newSelection[listIndex] = _selection;
            this.d_selection = newSelection;

            this.$emit('update:selection', this.d_selection);
            this.$emit('selection-change', {
                originalEvent: event,
                value: this.d_selection
            });
        },
        onItemDblClick(event, item, listIndex) {
            if (listIndex === 0) this.moveToTarget(event);
            else if (listIndex === 1) this.moveToSource(event);
        },
        onItemTouchEnd() {
            this.itemTouched = true;
        },
        onItemKeyDown(event, listType) {
            switch (event.code) {
                case 'ArrowDown':
                    this.onArrowDownKey(event, listType);
                    break;

                case 'ArrowUp':
                    this.onArrowUpKey(event, listType);
                    break;

                case 'Home':
                    this.onHomeKey(event, listType);
                    break;

                case 'End':
                    this.onEndKey(event, listType);
                    break;

                case 'Enter':
                    this.onEnterKey(event, listType);
                    break;

                case 'Space':
                    this.onSpaceKey(event, listType);
                    break;

                case 'KeyA':
                    if (event.ctrlKey) {
                        this.d_selection = [...this.modelValue];
                        this.$emit('update:selection', this.d_selection);
                    }

                default:
                    break;
            }
        },
        onArrowDownKey(event, listType) {
            const optionIndex = this.findNextOptionIndex(this.focusedOptionIndex, listType);

            this.changeFocusedOptionIndex(optionIndex, listType);

            if (event.shiftKey) {
                this.onEnterKey(event, listType);
            }

            event.preventDefault();
        },
        onArrowUpKey(event, listType) {
            const optionIndex = this.findPrevOptionIndex(this.focusedOptionIndex, listType);

            this.changeFocusedOptionIndex(optionIndex, listType);

            if (event.shiftKey) {
                this.onEnterKey(event, listType);
            }

            event.preventDefault();
        },
        onEnterKey(event, listType) {
            const items = DomHandler.find(this.$refs[listType].$el, 'li.p-picklist-item');
            const focusedItem = DomHandler.findSingle(this.$refs[listType].$el, `li.p-picklist-item[id=${this.focusedOptionIndex}]`);
            const matchedOptionIndex = [...items].findIndex((item) => item === focusedItem);
            const listId = listType === 'sourceList' ? 0 : 1;

            this.onItemClick(event, this.modelValue[listId][matchedOptionIndex], matchedOptionIndex, listId);

            event.preventDefault();
        },
        onSpaceKey(event, listType) {
            event.preventDefault();

            if (event.shiftKey) {
                const listId = listType === 'sourceList' ? 0 : 1;
                const items = DomHandler.find(this.$refs[listType].$el, 'li.p-picklist-item');
                const selectedItemIndex = ObjectUtils.findIndexInList(this.d_selection[listId][0], [...this.modelValue[listId]]);
                const focusedItem = DomHandler.findSingle(this.$refs[listType].$el, `li.p-picklist-item[id=${this.focusedOptionIndex}]`);
                const matchedOptionIndex = [...items].findIndex((item) => item === focusedItem);

                this.d_selection[listId] = [...this.modelValue[listId]].slice(Math.min(selectedItemIndex, matchedOptionIndex), Math.max(selectedItemIndex, matchedOptionIndex) + 1);
                this.$emit('update:selection', this.d_selection);
            } else {
                this.onEnterKey(event, listType);
            }
        },
        onHomeKey(event, listType) {
            if (event.ctrlKey && event.shiftKey) {
                const listId = listType === 'sourceList' ? 0 : 1;
                const items = DomHandler.find(this.$refs[listType].$el, 'li.p-picklist-item');
                const focusedItem = DomHandler.findSingle(this.$refs[listType].$el, `li.p-picklist-item[id=${this.focusedOptionIndex}]`);
                const matchedOptionIndex = [...items].findIndex((item) => item === focusedItem);

                this.d_selection[listId] = [...this.modelValue[listId]].slice(0, matchedOptionIndex + 1);
                this.$emit('update:selection', this.d_selection);
            } else {
                this.changeFocusedOptionIndex(0, listType);
            }

            event.preventDefault();
        },
        onEndKey(event, listType) {
            const items = DomHandler.find(this.$refs[listType].$el, 'li.p-picklist-item');

            if (event.ctrlKey && event.shiftKey) {
                const listId = listType === 'sourceList' ? 0 : 1;
                const focusedItem = DomHandler.findSingle(this.$refs[listType].$el, `li.p-picklist-item[id=${this.focusedOptionIndex}]`);
                const matchedOptionIndex = [...items].findIndex((item) => item === focusedItem);

                this.d_selection[listId] = [...this.modelValue[listId]].slice(matchedOptionIndex, items.length);
                this.$emit('update:selection', this.d_selection);
            } else {
                this.changeFocusedOptionIndex(items.length - 1, listType);
            }

            event.preventDefault();
        },
        findNextOptionIndex(index, listType) {
            const items = DomHandler.find(this.$refs[listType].$el, 'li.p-picklist-item');

            const matchedOptionIndex = [...items].findIndex((link) => link.id === index);

            return matchedOptionIndex > -1 ? matchedOptionIndex + 1 : 0;
        },
        findPrevOptionIndex(index, listType) {
            const items = DomHandler.find(this.$refs[listType].$el, 'li.p-picklist-item');
            const matchedOptionIndex = [...items].findIndex((link) => link.id === index);

            return matchedOptionIndex > -1 ? matchedOptionIndex - 1 : 0;
        },
        changeFocusedOptionIndex(index, listType) {
            const items = DomHandler.find(this.$refs[listType].$el, 'li.p-picklist-item');

            let order = index >= items.length ? items.length - 1 : index < 0 ? 0 : index;

            this.focusedOptionIndex = items[order].getAttribute('id');
            this.scrollInView(items[order].getAttribute('id'), listType);
        },
        scrollInView(id, listType) {
            const element = DomHandler.findSingle(this.$refs[listType].$el, `li[id="${id}"]`);

            if (element) {
                element.scrollIntoView && element.scrollIntoView({ block: 'nearest', inline: 'start' });
            }
        },
        updateListScroll(listElement) {
            const listItems = DomHandler.find(listElement, '.p-picklist-item.p-highlight');

            if (listItems && listItems.length) {
                switch (this.reorderDirection) {
                    case 'up':
                        DomHandler.scrollInView(listElement, listItems[0]);
                        break;

                    case 'top':
                        listElement.scrollTop = 0;
                        break;

                    case 'down':
                        DomHandler.scrollInView(listElement, listItems[listItems.length - 1]);
                        break;

                    case 'bottom':
                        listElement.scrollTop = listElement.scrollHeight;
                        break;

                    default:
                        break;
                }
            }
        },
        createStyle() {
            if (!this.styleElement) {
                this.$el.setAttribute(this.attributeSelector, '');
                this.styleElement = document.createElement('style');
                this.styleElement.type = 'text/css';
                document.head.appendChild(this.styleElement);

                let innerHTML = `
@media screen and (max-width: ${this.breakpoint}) {
    .p-picklist[${this.attributeSelector}] {
        flex-direction: column;
    }

    .p-picklist[${this.attributeSelector}] .p-picklist-buttons {
        padding: var(--content-padding);
        flex-direction: row;
    }

    .p-picklist[${this.attributeSelector}] .p-picklist-buttons .p-button {
        margin-right: var(--inline-spacing);
        margin-bottom: 0;
    }

    .p-picklist[${this.attributeSelector}] .p-picklist-buttons .p-button:last-child {
        margin-right: 0;
    }

    .p-picklist[${this.attributeSelector}] .pi-angle-right:before {
        content: "\\e930"
    }

    .p-picklist[${this.attributeSelector}] .pi-angle-double-right:before {
        content: "\\e92c"
    }

    .p-picklist[${this.attributeSelector}] .pi-angle-left:before {
        content: "\\e933"
    }

    .p-picklist[${this.attributeSelector}] .pi-angle-double-left:before {
        content: "\\e92f"
    }
}
`;

                this.styleElement.innerHTML = innerHTML;
            }
        },
        destroyStyle() {
            if (this.styleElement) {
                document.head.removeChild(this.styleElement);
                this.styleElement = null;
            }
        },
        moveDisabled(index) {
            if (!this.d_selection[index] || !this.d_selection[index].length) {
                return true;
            }
        },
        moveAllDisabled(list) {
            return ObjectUtils.isEmpty(this[list]);
        },
        moveSourceDisabled() {
            return ObjectUtils.isEmpty(this.targetList);
        },
        itemClass(item, id, listIndex) {
            return ['p-picklist-item', { 'p-highlight': this.isSelected(item, listIndex), 'p-focus': id === this.focusedOptionId }];
        }
    },
    computed: {
        idSource() {
            return `${this.id}_source`;
        },
        idTarget() {
            return `${this.id}_target`;
        },
        focusedOptionId() {
            return this.focusedOptionIndex !== -1 ? this.focusedOptionIndex : null;
        },
        containerClass() {
            return [
                'p-picklist p-component',
                {
                    'p-picklist-striped': this.stripedRows
                }
            ];
        },
        sourceList() {
            return this.modelValue && this.modelValue[0] ? this.modelValue[0] : null;
        },
        targetList() {
            return this.modelValue && this.modelValue[1] ? this.modelValue[1] : null;
        },
        attributeSelector() {
            return UniqueComponentId();
        },
        moveUpAriaLabel() {
            return this.$primevue.config.locale.aria ? this.$primevue.config.locale.aria.moveUp : undefined;
        },
        moveTopAriaLabel() {
            return this.$primevue.config.locale.aria ? this.$primevue.config.locale.aria.moveTop : undefined;
        },
        moveDownAriaLabel() {
            return this.$primevue.config.locale.aria ? this.$primevue.config.locale.aria.moveDown : undefined;
        },
        moveBottomAriaLabel() {
            return this.$primevue.config.locale.aria ? this.$primevue.config.locale.aria.moveBottom : undefined;
        },
        moveToTargetAriaLabel() {
            return this.$primevue.config.locale.aria ? this.$primevue.config.locale.aria.moveToTarget : undefined;
        },
        moveAllToTargetAriaLabel() {
            return this.$primevue.config.locale.aria ? this.$primevue.config.locale.aria.moveAllToTarget : undefined;
        },
        moveToSourceAriaLabel() {
            return this.$primevue.config.locale.aria ? this.$primevue.config.locale.aria.moveToSource : undefined;
        },
        moveAllToSourceAriaLabel() {
            return this.$primevue.config.locale.aria ? this.$primevue.config.locale.aria.moveAllToSource : undefined;
        }
    },
    components: {
        PLButton: Button,
        AngleRightIcon: AngleRightIcon,
        AngleLeftIcon: AngleLeftIcon,
        AngleDownIcon: AngleDownIcon,
        AngleUpIcon: AngleUpIcon,
        AngleDoubleRightIcon: AngleDoubleRightIcon,
        AngleDoubleLeftIcon: AngleDoubleLeftIcon,
        AngleDoubleDownIcon: AngleDoubleDownIcon,
        AngleDoubleUpIcon: AngleDoubleUpIcon
    },
    directives: {
        ripple: Ripple
    }
};
</script>

<style>
.p-picklist {
    display: flex;
}

.p-picklist-buttons {
    display: flex;
    flex-direction: column;
    justify-content: center;
}

.p-picklist-list-wrapper {
    flex: 1 1 50%;
}

.p-picklist-list {
    list-style-type: none;
    margin: 0;
    padding: 0;
    overflow: auto;
    min-height: 12rem;
    max-height: 24rem;
}

.p-picklist-item {
    cursor: pointer;
    overflow: hidden;
    position: relative;
}

.p-picklist-item.p-picklist-flip-enter-active.p-picklist-flip-enter-to,
.p-picklist-item.p-picklist-flip-leave-active.p-picklist-flip-leave-to {
    transition: none !important;
}
</style>
