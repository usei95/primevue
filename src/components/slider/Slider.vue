<template>
    <div :class="containerClass" @click="onBarClick" ref="container">
        <span class="p-slider-range" :style="rangeStyle"></span>
        <span v-if="!range" class="p-slider-handle" :style="handleStyle" @mousedown="onHandleMouseDown($event)" @keydown="onHandleKeyDown($event)" tabindex="0"></span>
        <span v-if="range" class="p-slider-handle" :style="rangeStartHandleStyle" @mousedown="onHandleMouseDown($event, 0)" @keydown="onHandleKeyDown($event, 0)" tabindex="0"></span>
        <span v-if="range" class="p-slider-handle" :style="rangeEndHandleStyle" @mousedown="onHandleMouseDown($event, 1)" @keydown="onHandleKeyDown($event, 1)" tabindex="0"></span>
    </div>
</template>

<script>
import DomHandler from '../utils/DomHandler';

export default {
    props: {
        value: [Number,Array],
		min: {
			type: Number,
			default: 0
		},
		max: {
			type: Number,
			default: 100
		},
		orientation: {
			type: String,
			default: 'horizontal'
		},
		step: {
			type: Number,
			default: null
		},
		range: {
			type: Boolean,
			default: false
		},
        disabled: {
			type: Boolean,
			default: false
		}
    },
    dragging: false,
    handleIndex: null,
    initX: null,
    initY: null,
    barWidth: null,
    barHeight: null,
    dragListener: null,
    mouseupListener: null,
    beforeDestroy() {
        this.unbindDragListeners();
    },
    methods: {
        updateDomData() {
            let rect = this.$refs.container.getBoundingClientRect();
            this.initX = rect.left + DomHandler.getWindowScrollLeft();
            this.initY = rect.top + DomHandler.getWindowScrollTop();
            this.barWidth = this.$refs.container.offsetWidth;
            this.barHeight = this.$refs.container.offsetHeight;
        },
        setValueFromHandlePosition(event, handlePosition) {
            let newValue = (this.max - this.min) * (handlePosition / 100) + this.min;

            if (this.range) {
                if (this.step)
                    this.handleStepChange(event, newValue, this.value[this.handleIndex]);
                else
                    this.updateModel(event, newValue);
            }
            else {
                if (this.step)
                    this.handleStepChange(event, newValue, this.value);
                else
                    this.updateModel(event, newValue);
            }
        },
        onSlide(event) {
            let handlePosition = this.horizontal ? ((event.pageX - this.initX) * 100) / (this.barWidth) : (((this.initY + this.barHeight) - event.pageY) * 100) / (this.barHeight);
            this.setValueFromHandlePosition(event, handlePosition);
        },
        handleStepChange(event, newValue, oldValue) {
            let diff = (newValue - oldValue);
            let val = oldValue;

            if (diff < 0)
                val = oldValue + Math.ceil(newValue / this.step - oldValue / this.step) * this.step;
            else if (diff > 0)
                val = oldValue + Math.floor(newValue / this.step - oldValue / this.step) * this.step;

            this.updateModel(event, val);
        },
        updateModel(event, value) {
            let newValue = value;
            let modelValue;

            if (this.range) {
                if (this.handleIndex == 0) {
                    if (newValue < this.min)
                        newValue = this.min;
                    else if (value > this.value[1])
                        newValue = this.value[1];
                }
                else {
                    if (newValue > this.max)
                        newValue = this.max;
                    else if (newValue < this.value[0])
                        newValue = this.value[0];
                }

                modelValue = [...this.value];
                modelValue[this.handleIndex] = Math.floor(value);
            }
            else {
                if (newValue < this.min)
                    newValue = this.min;
                else if (newValue > this.max)
                    newValue = this.max;

                modelValue = Math.floor(newValue);
            }

            this.$emit('input', modelValue);
            this.$emit('change', modelValue);
        },
        onBarClick(event) {
            if (this.disabled) {
                return;
            }

            if (!DomHandler.hasClass(event.target, 'p-slider-handle')) {
                this.updateDomData();
                this.onSlide(event);
            }
        },
        onHandleMouseDown(event, index) {
            if (this.disabled) {
                return;
            }

            this.dragging = true;
            this.updateDomData();
            this.handleIndex = index;
            this.bindDragListeners();
            event.preventDefault();
        },
        onHandleKeyDown(event, index) {
            this.handleIndex = index;

            switch (event.which) {
                //down
                case 40:
                    if (this.vertical) {
                        this.decrementValue(event, index);
                        event.preventDefault();
                    }
                break;

                //up
                case 38:
                    if (this.vertical) {
                        this.incrementValue(event, index);
                        event.preventDefault();
                    }
                break;

                //left
                case 37:
                    if (this.horizontal) {
                        this.decrementValue(event, index);
                        event.preventDefault();
                    }
                break;

                //right
                case 39:
                    if (this.horizontal) {
                        this.incrementValue(event, index);
                        event.preventDefault();
                    }
                break;

                default:
                break;
            }
        },
        decrementValue(event, index) {
            let newValue;

            if (this.range) {
                if (this.step)
                    newValue = this.value[index] - this.step;
                else
                    newValue = this.value[index] - 1;
            }
            else {
                if (this.step)
                    newValue = this.value - this.step;
                else
                    newValue = this.value - 1;
            }

            this.updateModel(event, newValue);

            event.preventDefault();
        },
        incrementValue(event, index) {
            let newValue;

            if (this.range) {
                if (this.step)
                    newValue = this.value[index] + this.step;
                else
                    newValue = this.value[index] + 1;
            }
            else {
                if (this.step)
                    newValue = this.value + this.step;
                else
                    newValue = this.value + 1;
            }

            this.updateModel(event, newValue);

            event.preventDefault();
        },
        bindDragListeners() {
             if (!this.dragListener) {
                this.dragListener = (event) => {
                    if (this.dragging) {
                        this.onSlide(event);
                    }
                };

                document.addEventListener('mousemove', this.dragListener);
            }

            if (!this.mouseupListener) {
                this.mouseupListener = (event) => {
                    if (this.dragging) {
                        this.dragging = false;
                        this.$emit('slideend', {originalEvent: event, values: this.value});
                    }
                };

                document.addEventListener('mouseup', this.mouseupListener);
            }
        },
        unbindDragListeners() {
            if (this.dragListener) {
                document.removeEventListener('mousemove', this.dragListener);
                this.dragListener = null;
            }

            if (this.mouseupListener) {
                document.removeEventListener('mouseup', this.mouseupListener);
                this.mouseupListener = null;
            }
        }
    },
    computed: {
        containerClass() {
            return ['p-slider p-component', {
                'p-disabled': this.disabled,
                'p-slider-horizontal': (this.orientation === 'horizontal'),
                'p-slider-vertical': (this.orientation === 'vertical')
            }];
        },
        horizontal() {
            return this.orientation === 'horizontal';
        },
        vertical() {
            return this.orientation === 'vertical';
        },
        rangeStyle() {
            if (this.range) {
                if (this.horizontal)
                    return {'left': this.rangeStartPosition + '%', width: (this.rangeEndPosition - this.rangeStartPosition + '%')};
                else
                    return {'bottom': this.rangeStartPosition + '%', height: (this.rangeEndPosition - this.rangeStartHandlePosition + '%')};
            }
            else {
                if (this.horizontal)
                    return {'width': this.handlePosition + '%'};
                else
                    return {'height': this.handlePosition + '%'};
            }
        },
        handleStyle() {
            if (this.horizontal)
                return {'left': this.handlePosition + '%'};
            else
                return {'bottom': this.handlePosition + '%'};

        },
        handlePosition() {
            if (this.value === 0)
                return 0;
            else if (this.value < this.min)
                return 0;
            else if (this.value > this.max)
                return 100;
            else
                return (this.value - this.min) * 100 / (this.max - this.min);
        },
        rangeStartPosition() {
            if (this.value)
                return (this.value[0] < this.min ? 0 : this.value[0] - this.min) * 100 / (this.max - this.min);
            else
                return 0;
        },
        rangeEndPosition() {
            if (this.value)
                return (this.value[1] > this.max ? 100 : this.value[1] - this.min) * 100 / (this.max - this.min);
            else
                return 0;
        },
        rangeStartHandleStyle() {
            if (this.horizontal)
                return {'left': this.rangeStartPosition + '%'};
            else
                return {'bottom': this.rangeStartPosition + '%'};
        },
        rangeEndHandleStyle() {
            if (this.horizontal)
                return {'left': this.rangeEndPosition + '%'};
            else
                return {'bottom': this.rangeEndPosition + '%'};
        },
    }
}
</script>

<style>
.p-slider {
	position: relative;
	text-align: left;
}
.p-slider .p-slider-handle {
	position: absolute;
	width: 1.2em;
	height: 1.2em;
	cursor: default;
	-ms-touch-action: none;
	touch-action: none;
}
.p-slider .p-slider-range {
	position: absolute;
	font-size: .7em;
	display: block;
	border: 0;
	background-position: 0 0;
}

.p-slider-horizontal {
	height: .8em;
}
.p-slider-horizontal .p-slider-handle {
	top: -.3em;
	margin-left: -.6em;
}
.p-slider-horizontal .p-slider-range {
	top: 0;
	height: 100%;
}
.p-slider-vertical {
	width: .8em;
	height: 100px;
}
.p-slider-vertical .p-slider-handle {
	left: -.3em;
	margin-left: 0;
	margin-bottom: -.6em;
}
.p-slider-vertical .p-slider-range {
	left: 0;
    width: 100%;
    bottom: 0;
}
</style>
