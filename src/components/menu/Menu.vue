<template>
    <transition name="p-input-overlay" @enter="onEnter" @leave="onLeave">
        <div ref="container" :class="containerClass" v-if="popup ? visible : true">
            <ul class="p-menu-list p-reset" role="menu">
                <template v-for="(item, i) of model">
                    <template v-if="item.items && item.visible !== false">
                        <li class="p-submenu-header" :key="item.label+i" v-if="item.items">{{item.label}}</li>
                        <Menuitem v-for="(child, j) of item.items" :key="child.label+i+j" :item="child" @click="itemClick" />
                    </template>
                    <template v-else>
                        <Menuitem :key="item.label+i" :item="item" @click="itemClick" />
                    </template>
                </template>
            </ul>
        </div>
    </transition>
</template>

<script>
import DomHandler from '../utils/DomHandler';
import Menuitem from './Menuitem';

export default {
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
            default: null
        },
        autoZIndex: {
            type: Boolean,
            default: true
        },
        baseZIndex: {
            type: Number,
            default: 0
        }
    },
    data() {
        return {
            visible: false
        };
    },
    target: null,
    outsideClickListener: null,
    resizeListener: null,
    beforeDestroy() {
        this.restoreAppend();
        this.unbindResizeListener();
        this.unbindOutsideClickListener();
        this.target = null;
    },
    methods: {
        itemClick(event) {
            const item = event.item;
            if (item.command) {
                item.command(event);
                event.originalEvent.preventDefault();
            }
            this.hide();
        },
        toggle(event) {
            if (this.visible)
                this.hide();
            else
                this.show(event);
        },
        show(event) {
            this.visible = true;
            this.target = event.currentTarget;
        },
        hide() {
            this.visible = false;
        },
        onEnter() {
            this.appendContainer();
            this.alignOverlay();
            this.bindOutsideClickListener();
            this.bindResizeListener();

            if (this.autoZIndex) {
                this.$refs.container.style.zIndex = String(this.baseZIndex + DomHandler.generateZIndex());
            }
        },
        onLeave() {
            this.unbindOutsideClickListener();
            this.unbindResizeListener();
        },
        alignOverlay() {
            DomHandler.absolutePosition(this.$refs.container, this.target);
        },
        bindOutsideClickListener() {
            if (!this.outsideClickListener) {
                this.outsideClickListener = (event) => {
                    if (this.visible && this.$refs.container && !this.$refs.container.contains(event.target) && !this.isTargetClicked(event)) {
                        this.hide();
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
        bindResizeListener() {
            if (!this.resizeListener) {
                this.resizeListener = () => {
                    if (this.visible) {
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
        isTargetClicked() {
            return this.target && (this.target === event.target || this.target.contains(event.target));
        },
        appendContainer() {
            if (this.appendTo) {
                if (this.appendTo === 'body')
                    document.body.appendChild(this.$refs.container);
                else
                    document.getElementById(this.appendTo).appendChild(this.$refs.container);
            }
        },
        restoreAppend() {
            if (this.$refs.container && this.appendTo) {
                if (this.appendTo === 'body')
                    document.body.removeChild(this.$refs.container);
                else
                    document.getElementById(this.appendTo).removeChild(this.$refs.container);
            }
        }
    },
    computed: {
        containerClass() {
            return ['p-menu p-component', {
                'p-menu-dynamic p-menu-overlay': this.popup
            }]
        }
    },
    components: {
        'Menuitem': Menuitem
    }
}
</script>

<style>
.p-menu {
    width: 12.5em;
    padding: .25em;
}

.p-menu.p-menu-dynamic {
    position: absolute;
}

.p-menu .p-menu-separator {
    border-width: 1px 0 0 0;
}

.p-menu ul {
    list-style: none;
    margin: 0;
    padding: 0;
}

.p-menu .p-submenu-header {
    padding: .25em .5em;
    margin: .125em 0;
}

.p-menu .p-menuitem {
    margin: .125em 0;
}

.p-menu .p-menuitem-link {
    padding: .25em;
    display: block;
    text-decoration: none;
}

.p-menu .p-menuitem-icon {
    margin-right: .25em;
    vertical-align: middle;
}

.p-menu .p-menuitem-text {
    vertical-align: middle;
}
</style>
