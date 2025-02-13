<template>
    <ul ref="element" :class="containerClass" role="menu">
        <template v-for="(item, i) of model">
            <li role="menuitem" :class="getItemClass(item)" :style="item.style" v-if="item.visible !== false && !item.separator" :key="item.label + i"
                @mouseenter="onItemMouseEnter($event, item)">
                <router-link v-if="item.to" :to="item.to" class="p-menuitem-link"
                    @click.native="onItemClick($event, item)" @keydown.native="onItemKeyDown($event, item)">
                    <span :class="['p-menuitem-icon', item.icon]"></span>
                    <span class="p-menuitem-text">{{item.label}}</span>
                </router-link>
                <a v-else :href="item.url||'#'" class="p-menuitem-link" :target="item.target"
                    @click="onItemClick($event, item)" @keydown="onItemKeyDown($event, item)">
                    <span :class="['p-menuitem-icon', item.icon]"></span>
                    <span class="p-menuitem-text">{{item.label}}</span>
                    <span class="p-submenu-icon pi pi-fw pi-caret-right" v-if="item.items"></span>
                </a>
                <sub-menu :model="item.items" v-if="item.visible !== false && item.items" :key="item.label + '_sub_'"
                    @leaf-click="onLeafClick" @keydown-item="onChildItemKeyDown" :parentActive="item === activeItem" />
            </li>
            <li class="p-menu-separator" :style="item.style" v-if="item.visible !== false && item.separator" :key="'separator' + i"></li>
        </template>
    </ul>
</template>

<script>
import DomHandler from '../utils/DomHandler';

export default {
    name: 'sub-menu',
    props: {
        model: {
            type: Array,
            default: null
        },
        root: {
            type: Boolean,
            default: false
        },
        popup: {
            type: Boolean,
            default: false
        },
        parentActive: {
            type: Boolean,
            default: false
        }
    },
    documentClickListener: null,
    watch: {
        parentActive(newValue) {
            if (!newValue) {
                this.activeItem = null;
            }
        }
    },
    data() {
        return {
            activeItem: null
        }
    },
    updated() {
        if (this.root && this.activeItem) {
            this.bindDocumentClickListener();
        }
    },
    beforeDestroy() {
        this.unbindDocumentClickListener();
    },
    methods: {
        onItemMouseEnter(event, item) {
            if (item.disabled) {
                event.preventDefault();
                return;
            }

            if (this.root) {
                if (this.activeItem || this.popup) {
                    this.activeItem = item;
                }
            }
            else {
                this.activeItem = item;
            }
        },
        onItemClick(event, item) {
            if (item.disabled) {
                event.preventDefault();
                return;
            }

            if (!item.url) {
                event.preventDefault();
            }

            if (item.command) {
                item.command({
                    originalEvent: event,
                    item: item
                });
            }

            if (this.root && item.items) {
                if (this.activeItem && item === this.activeItem)
                    this.activeItem = null;
                else
                   this.activeItem = item;
            }

            if (!item.items) {
                this.onLeafClick();
            }
        },
        onLeafClick() {
            this.activeItem = null;

            if (!this.root) {
                this.$emit('leaf-click');
            }
        },
        onItemKeyDown(event, item) {
            let listItem = event.currentTarget.parentElement;

            switch (event.which) {
                //down
                case 40:
                    var nextItem = this.findNextItem(listItem);
                    if (nextItem) {
                        nextItem.children[0].focus();
                    }

                    event.preventDefault();
                break;

                //up
                case 38:
                    var prevItem = this.findPrevItem(listItem);
                    if (prevItem) {
                        prevItem.children[0].focus();
                    }

                    event.preventDefault();
                break;

                //right
                case 39:
                    if (item.items) {
                        this.activeItem = item;

                        setTimeout(() => {
                            listItem.children[1].children[0].children[0].focus();
                        }, 50);
                    }

                    event.preventDefault();
                break;

                default:
                break;
            }

            this.$emit('keydown-item', {
                originalEvent: event,
                element: listItem
            });
        },
        onChildItemKeyDown(event) {
            //left
            if (event.originalEvent.which === 37) {
                this.activeItem = null;
                event.element.parentElement.previousElementSibling.focus();
            }
        },
        findNextItem(item) {
            let nextItem = item.nextElementSibling;

            if (nextItem)
                return DomHandler.hasClass(nextItem, 'p-disabled') || !DomHandler.hasClass(nextItem, 'p-menuitem') ? this.findNextItem(nextItem) : nextItem;
            else
                return null;
        },
        findPrevItem(item) {
            let prevItem = item.previousElementSibling;

            if (prevItem)
                return DomHandler.hasClass(prevItem, 'p-disabled') || !DomHandler.hasClass(prevItem, 'p-menuitem') ? this.findPrevItem(prevItem) : prevItem;
            else
                return null;
        },
        getItemClass(item) {
            return [
                'p-menuitem', item.class, {
                    'p-menuitem-active': this.activeItem === item,
                    'p-disabled': item.disabled,
                }
            ]
        },
        bindDocumentClickListener() {
            if (!this.documentClickListener) {
                this.documentClickListener = (event) => {
                    if (this.$el && !this.$el.contains(event.target)) {
                        this.activeItem = null;
                        this.unbindDocumentClickListener();
                    }
                };

                document.addEventListener('click', this.documentClickListener);
            }
        },
        unbindDocumentClickListener() {
            if (this.documentClickListener) {
                document.removeEventListener('click', this.documentClickListener);
                this.documentClickListener = null;
            }
        }
    },
    computed: {
        containerClass() {
            return {'p-submenu-list': !this.root};
        }
    }
}
</script>