<template>
    <div ref="container" :class="containerClass" @click="onClick($event)">
        <div class="p-hidden-accessible">
            <input ref="focusInput" type="text" role="listbox" readonly :disabled="disabled" @focus="onFocus" @blur="onBlur" @keydown="onKeyDown" :tabindex="tabindex"/>
        </div>
        <input v-if="editable" type="text" class="p-dropdown-label p-inputtext" :disabled="disabled" @focus="onFocus" @blur="onBlur" :placeholder="placeholder" :value="editableInputValue" @input="onEditableInput">
        <label v-if="!editable" :class="labelClass">{{label}}</label>
        <i v-if="showClear && value != null" class="p-dropdown-clear-icon pi pi-times" @click="onClearClick($event)"></i>
        <div class="p-dropdown-trigger">
            <span class="p-dropdown-trigger-icon pi pi-chevron-down p-clickable"></span>
        </div>
        <transition name="p-input-overlay" @enter="onOverlayEnter" @leave="onOverlayLeave">
            <div ref="overlay" class="p-dropdown-panel" v-if="overlayVisible">
                <div v-if="filter" class="p-dropdown-filter-container">
                    <input type="text" v-model="filterValue" autoComplete="off" class="p-dropdown-filter p-inputtext p-component" :placeholder="filterPlaceholder" @keydown="onFilterKeyDown" />
                    <span class="p-dropdown-filter-icon pi pi-search"></span>
                </div>
                <div ref="itemsWrapper" class="p-dropdown-items-wrapper" :style="{'max-height': scrollHeight}">
                    <ul class="p-dropdown-items p-dropdown-list p-component">
                        <li v-for="(option, i) of visibleOptions" :class="['p-dropdown-item', {'p-highlight': isSelected(option), 'p-disabled': isOptionDisabled(option)}]"
                            :aria-label="getOptionLabel(option)" :key="getOptionLabel(option)" @click="onOptionSelect($event, option)">
                            <slot name="option" :option="option" :index="i">
                                {{getOptionLabel(option)}}
                            </slot>
                        </li>
                    </ul>
                </div>
            </div>
        </transition>
    </div>
</template>

<script>
import ObjectUtils from '../utils/ObjectUtils';
import DomHandler from '../utils/DomHandler';

export default {
    props: {
        value: null,
        options: Array,
        optionLabel: null,
        optionValue: null,
        optionDisabled: null,
		scrollHeight: {
			type: String,
			default: '200px'
		},
		filter: Boolean,
		filterPlaceholder: String,
		editable: Boolean,
		placeholder: String,
		disabled: Boolean,
        dataKey: null,
		showClear: Boolean,
		tabindex: String
    },
    data() {
        return {
            focused: false,
            filterValue: null,
            overlayVisible: false
        };
    },
    outsideClickListener: null,
    searchTimeout: null,
    currentSearchChar: null,
    previousSearchChar: null,
    searchValue: null,
    beforeDestroy() {
        this.unbindOutsideClickListener();
    },
    updated() {
        if (this.overlayVisible && this.filterValue) {
            this.alignOverlay();
        }
    },
    methods: {
        getOptionLabel(option) {
            return ObjectUtils.resolveFieldData(option, this.optionLabel);
        },
        getOptionValue(option) {
            return this.optionValue ? ObjectUtils.resolveFieldData(option, this.optionValue) : option;
        },
        isOptionDisabled(option) {
            return this.optionDisabled ? ObjectUtils.resolveFieldData(option, this.optionDisabled) : false;
        },
        getSelectedOption() {
            let selectedOption;

            if (this.value != null && this.options) {
                for (let option of this.options) {
                    if ((ObjectUtils.equals(this.value, this.getOptionValue(option), this.dataKey))) {
                        selectedOption = option;
                        break;
                    }
                }
            }

            return selectedOption;
        },
        isSelected(option) {
            return ObjectUtils.equals(this.value, this.getOptionValue(option), this.dataKey);
        },
        getSelectedOptionIndex() {
            let selectedOptionIndex = -1;

            if (this.value != null && this.visibleOptions) {
                for (let i = 0; i < this.visibleOptions.length; i++) {
                    if ((ObjectUtils.equals(this.value, this.getOptionValue(this.visibleOptions[i]), this.dataKey))) {
                        selectedOptionIndex = i;
                        break;
                    }
                }
            }

            return selectedOptionIndex;
        },
        onFocus() {
            this.focused = true;
        },
        onBlur() {
            this.focused = false;
        },
        onKeyDown(event) {
            switch(event.which) {
                //down
                case 40:
                    this.onDownKey(event);
                break;

                //up
                case 38:
                    this.onUpKey(event);
                break;

                //space
                case 32:
                    if (!this.overlayVisible) {
                        this.overlayVisible = true;
                        event.preventDefault();
                    }
                break;

                //enter and escape
                case 13:
                case 27:
                    if (this.overlayVisible) {
                        this.overlayVisible = false;
                        event.preventDefault();
                    }
                break;

                //tab
                case 9:
                    this.overlayVisible = false;
                break;

                default:
                    this.search(event);
                break;
            }
        },
        onFilterKeyDown(event) {
            switch (event.which) {
                //down
                case 40:
                    this.onDownKey(event);
                    break;

                //up
                case 38:
                    this.onUpKey(event);
                    break;

                //enter and escape
                case 13:
                case 27:
                    this.overlayVisible = false;
                    event.preventDefault();
                break;

                default:
                break;
            }
        },
        onDownKey(event) {
            if (this.visibleOptions) {
                if (!this.overlayVisible && event.altKey) {
                    this.overlayVisible = true;
                }
                else {
                    let nextOption = this.findNextOption(this.getSelectedOptionIndex());

                    if (nextOption) {
                        this.updateModel(event, this.getOptionValue(nextOption));
                    }
                }
            }

            event.preventDefault();
        },
        onUpKey(event) {
            if (this.visibleOptions) {
                let prevOption = this.findPrevOption(this.getSelectedOptionIndex());

                if (prevOption) {
                    this.updateModel(event, this.getOptionValue(prevOption));
                }
            }

            event.preventDefault();
        },
        findNextOption(index) {
            let i = index + 1;
            if (i === this.visibleOptions.length) {
                return null;
            }

            let option = this.visibleOptions[i];
            if (this.isOptionDisabled(option))
                return this.findNextOption(i);
            else
                return option;

        },
        findPrevOption(index) {
            let i = index - 1;
            if (i < 0) {
                return null;
            }

            let option = this.visibleOptions[i];
            if (this.isOptionDisabled(option))
                return this.findPrevOption(i);
            else
                return option;
        },
        onClearClick(event) {
            this.updateModel(event, null);
        },
        onClick(event) {
            if (DomHandler.hasClass(event.target, 'p-dropdown-clear-icon')) {
                return;
            }
            else if (!this.$refs.overlay || !this.$refs.overlay.contains(event.target)) {
                this.overlayVisible = !this.overlayVisible;
                this.$refs.focusInput.focus();
            }
        },
        onOptionSelect(event, option) {
            let value = this.getOptionValue(option);
            this.updateModel(event, value);
            this.$refs.focusInput.focus();

            setTimeout(() => {
                this.overlayVisible = false;
            }, 100);
        },
        onEditableInput(event) {
            this.$emit('input', event.target.value);
        },
        onOverlayEnter() {
            this.alignOverlay();
            this.bindOutsideClickListener();
        },
        onOverlayLeave() {
            this.unbindOutsideClickListener();
        },
        alignOverlay() {
            DomHandler.relativePosition(this.$refs.overlay, this.$refs.container);
        },
        updateModel(event, value) {
            this.$emit('input', value);
            this.$emit('change', {originalEvent: event, value: value});
        },
        bindOutsideClickListener() {
            if (!this.outsideClickListener) {
                this.outsideClickListener = (event) => {
                    if (this.overlayVisible && this.$refs.overlay && !this.$refs.container.contains(event.target)) {
                        this.overlayVisible = false;
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
        search(event) {
            if (!this.visibleOptions) {
                return;
            }

            if (this.searchTimeout) {
                clearTimeout(this.searchTimeout);
            }

            const char = String.fromCharCode(event.keyCode);
            this.previousSearchChar = this.currentSearchChar;
            this.currentSearchChar = char;

            if (this.previousSearchChar === this.currentSearchChar)
                this.searchValue = this.currentSearchChar;
            else
                this.searchValue = this.searchValue ? this.searchValue + char : char;

            let searchIndex = this.getSelectedOptionIndex();
            let newOption = this.searchOption(++searchIndex);

            if (newOption) {
                this.updateModel(event, this.getOptionValue(newOption));
            }

            this.searchTimeout = setTimeout(() => {
                this.searchValue = null;
            }, 250);
        },
        searchOption(index) {
            let option;

            if (this.searchValue) {
                option = this.searchOptionInRange(index, this.visibleOptions.length);

                if (!option) {
                    option = this.searchOptionInRange(0, index);
                }
            }

            return option;
        },
        searchOptionInRange(start, end) {
            for (let i = start; i < end; i++) {
                let opt = this.visibleOptions[i];
                let label = this.getOptionLabel(opt).toLowerCase();
                if (label.startsWith(this.searchValue.toLowerCase())) {
                    return opt;
                }
            }

            return null;
        }
    },
    computed: {
        visibleOptions() {
            if (this.filterValue && this.filterValue.trim().length > 0)
                return this.options.filter(option => this.getOptionLabel(option).toLowerCase().indexOf(this.filterValue.toLowerCase()) > -1);
            else
                return this.options;
        },
        containerClass() {
            return [
                'p-dropdown p-component p-unselectable-text',
                {
                    'p-disabled': this.disabled,
                    'p-dropdown-clearable': this.showClear && !this.disabled,
                    'p-focus': this.focused
                }
            ];
        },
        labelClass() {
            return [
                'p-dropdown-label p-inputtext',
                {
                    'p-placeholder': this.label === this.placeholder,
                    'p-dropdown-label-empty': (this.label === 'p-emptylabel' || this.label.length === 0)
                }
            ];
        },
        label() {
            let selectedOption = this.getSelectedOption();
            if (selectedOption)
                return this.getOptionLabel(selectedOption);
            else
                return this.placeholder||'p-emptylabel';
        },
        editableInputValue() {
            let selectedOption = this.getSelectedOption();
            if (selectedOption)
                return this.getOptionLabel(selectedOption);
            else
                return this.value;
        }
    }
}
</script>

<style>
.p-dropdown {
    display: inline-block;
    position: relative;
    cursor: pointer;
    vertical-align: middle;
}

.p-dropdown .p-dropdown-clear-icon {
    position: absolute;
    right: 2em;
    top: 50%;
    font-size: 1em;
    height: 1em;
    margin-top: -.5em;
}

.p-dropdown .p-dropdown-trigger {
    border-right: none;
    border-top: none;
    border-bottom: none;
    cursor: pointer;
    width: 1.5em;
    height: 100%;
    position: absolute;
    right: 0;
    top: 0;
    padding: 0 .25em;
}

.p-dropdown .p-dropdown-trigger .p-dropdown-trigger-icon {
    top: 50%;
    left: 50%;
    margin-top: -.5em;
    margin-left: -.5em;
    position: absolute;
}

.p-dropdown .p-dropdown-label  {
    display: block;
    border: none;
    white-space: nowrap;
    overflow: hidden;
    font-weight: normal;
    width: 100%;
    padding-right: 1.5em;
}

.p-dropdown .p-dropdown-item-empty,
.p-dropdown .p-dropdown-label-empty {
    overflow: hidden;
    visibility: hidden;
}

.p-dropdown.p-disabled .p-dropdown-trigger,
.p-dropdown.p-disabled .p-dropdown-label {
    cursor: default;
}

.p-dropdown label.p-dropdown-label  {
    cursor: pointer;
}

.p-dropdown input.p-dropdown-label  {
    cursor: default;
}

.p-dropdown .p-dropdown-panel {
    min-width: 100%;
    z-index: 1;
}

.p-dropdown-panel {
    position: absolute;
    height: auto;
}

.p-dropdown-panel .p-dropdown-items-wrapper {
    overflow: auto;
}

.p-dropdown-panel .p-dropdown-item {
    font-weight: normal;
    border: 0 none;
    cursor: pointer;
    margin: 1px 0;
    padding: .125em .25em;
    text-align: left;
}

.p-dropdown-panel .p-dropdown-item-group {
    font-weight: bold;
}

.p-dropdown-panel .p-dropdown-list {
    padding: 0.4em;
    border: 0 none;
    margin: 0;
    list-style-type: none;
}

.p-dropdown-panel .p-dropdown-filter {
    width: 100%;
    box-sizing: border-box;
    padding-right: 1.5em;
}

.p-dropdown-panel .p-dropdown-filter-container {
    position: relative;
    margin: 0;
    padding: 0.4em;
    display: inline-block;
}

.p-dropdown-panel .p-dropdown-filter-container .p-dropdown-filter-icon {
    position: absolute;
    top: .8em;
    right: 1em;
}

/** Dropdown **/
.p-fluid .p-dropdown {
    width: 100%;
}
</style>