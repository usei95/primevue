<template>
    <div :class="containerClass">
        <slot></slot>
        <div class="p-treetable-loading" v-if="loading">
            <div class="p-treetable-loading-overlay p-component-overlay"></div>
            <div class="p-treetable-loading-content">
                <i :class="loadingIconClass"></i>
            </div>
        </div>
        <div class="p-treetable-header" v-if="$scopedSlots.header">
            <slot name="header"></slot>
        </div>
         <TTPaginator v-if="paginatorTop" :rows="d_rows" :first="d_first" :totalRecords="totalRecordsLength" :pageLinkSize="pageLinkSize" :template="paginatorTemplate" :rowsPerPageOptions="rowsPerPageOptions"
                :currentPageReportTemplate="currentPageReportTemplate" class="p-paginator-top" @page="onPage($event)" :alwaysShow="alwaysShowPaginator">
            <template #left v-if="$scopedSlots.paginatorLeft">
                <slot name="paginatorLeft"></slot>
            </template>
            <template #right v-if="$scopedSlots.paginatorRight">
                <slot name="paginatorRight"></slot>
            </template>
        </TTPaginator>
        <div class="p-treetable-tablewrapper">
            <table ref="table">
                <thead class="p-treetable-thead">
                    <tr>
                        <th v-for="(col,i) of columns" :key="col.columnKey||col.field||i" :style="col.headerStyle" :class="getColumnHeaderClass(col)" @click="onColumnHeaderClick($event, col)">
                            <span class="p-column-resizer p-clickable" @mousedown="onColumnResizeStart" v-if="resizableColumns"></span>
                            <TTColumnSlot :column="col" type="header" v-if="col.$scopedSlots.header" />
                            <span class="p-column-title" v-if="col.header">{{col.header}}</span>
                            <span v-if="col.sortable" :class="getSortableColumnIcon(col)"></span>
                            <TTColumnSlot :column="col" type="filter" v-if="col.$scopedSlots.filter" />
                        </th>
                    </tr>
                </thead>
                <tfoot class="p-treetable-tfoot" v-if="hasFooter">
                    <tr>
                        <td v-for="(col,i) of columns" :key="col.columnKey||col.field||i" :style="col.footerStyle" :class="col.footerClass">
                            <TTColumnSlot :column="col" type="footer" v-if="col.$scopedSlots.footer" />
                            {{col.footer}}
                        </td>
                    </tr>
                </tfoot>
                <tbody class="p-treetable-tbody">
                    <template v-if="!empty">
                        <TTRow v-for="node of dataToRender" :key="node.key" :columns="columns" :node="node" :level="0"
                        :expandedKeys="d_expandedKeys" @node-toggle="onNodeToggle"
                        :selectionMode="selectionMode" :selectionKeys="selectionKeys" @node-click="onNodeClick" @checkbox-change="onCheckboxChange"></TTRow>
                    </template>
                    <tr v-else class="p-treetable-emptymessage">
                        <td :colspan="columns.length">
                            <slot name="empty"></slot>
                        </td>
                    </tr>
                </tbody>
            </table>
        </div>
        <TTPaginator v-if="paginatorBottom" :rows="d_rows" :first="d_first" :totalRecords="totalRecordsLength" :pageLinkSize="pageLinkSize" :template="paginatorTemplate" :rowsPerPageOptions="rowsPerPageOptions"
                :currentPageReportTemplate="currentPageReportTemplate" class="p-paginator-bottom" @page="onPage($event)" :alwaysShow="alwaysShowPaginator">
            <template #left v-if="$scopedSlots.paginatorLeft">
                <slot name="paginatorLeft"></slot>
            </template>
            <template #right v-if="$scopedSlots.paginatorRight">
                <slot name="paginatorRight"></slot>
            </template>
        </TTPaginator>
        <div class="p-treetable-footer" v-if="$scopedSlots.footer">
            <slot name="footer"></slot>
        </div>
        <div ref="resizeHelper" class="p-column-resizer-helper p-highlight" style="display: none"></div>
    </div>
</template>

<script>
import ObjectUtils from '../utils/ObjectUtils';
import FilterUtils from '../utils/FilterUtils';
import DomHandler from '../utils/DomHandler';
import TreeTableColumnSlot from './TreeTableColumnSlot';
import TreeTableRowLoader from './TreeTableRowLoader';
import Paginator from '../paginator/Paginator';

export default {
    props: {
        value: {
            type: null,
            default: null
        },
        expandedKeys: {
            type: null,
            default: null
        },
        selectionKeys: {
            type: null,
            default: null
        },
        selectionMode: {
            type: String,
            default: null
        },
        metaKeySelection: {
            type: Boolean,
            default: true
        },
        rows: {
            type: Number,
            default: 0
        },
        first: {
            type: Number,
            default: 0
        },
        totalRecords: {
            type: Number,
            default: 0
        },
        paginator: {
            type: Boolean,
            default: false
        },
        paginatorPosition: {
            type: String,
            default: 'bottom'
        },
        alwaysShowPaginator: {
            type: Boolean,
            default: true
        },
        paginatorTemplate: {
            type: String,
            default: 'FirstPageLink PrevPageLink PageLinks NextPageLink LastPageLink RowsPerPageDropdown'
        },
        pageLinkSize: {
            type: Number,
            default: 5
        },
        rowsPerPageOptions: {
            type: Array,
            default: null
        },
        currentPageReportTemplate: {
            type: String,
            default: '({currentPage} of {totalPages})'
        },
        lazy: {
            type: Boolean,
            default: false
        },
        loading: {
            type: Boolean,
            default: false
        },
        loadingIcon: {
            type: String,
            default: 'pi pi-spinner'
        },
        rowHover: {
            type: Boolean,
            default: false
        },
        autoLayout: {
            type: Boolean,
            default: false
        },
        sortField: {
            type: String,
            default: null
        },
        sortOrder: {
            type: Number,
            default: null
        },
        defaultSortOrder: {
            type: Number,
            default: 1
        },
        multiSortMeta: {
            type: Array,
            default: null
        },
        sortMode: {
            type: String,
            default: 'single'
        },
        filters: {
            type: Object,
            default: null
        },
        filterMode: {
            type: String,
            default: 'lenient'
        },
        resizableColumns: {
            type: Boolean,
            default: false
        },
        columnResizeMode: {
            type: String,
            default: 'fit'
        }
    },
    documentColumnResizeListener: null,
    documentColumnResizeEndListener: null,
    lastResizeHelperX: null,
    resizeColumnElement: null,
    data() {
        return {
            allChildren: null,
            d_expandedKeys: this.expandedKeys || {},
            d_first: this.first,
            d_rows: this.rows,
            d_sortField: this.sortField,
            d_sortOrder: this.sortOrder,
            d_multiSortMeta: this.multiSortMeta ? [...this.multiSortMeta] : [],
        }
    },
    watch: {
        expandedKeys(newValue) {
            this.d_expandedKeys = newValue;
        },
        first(newValue) {
            this.d_first = newValue;
        },
        rows(newValue) {
            this.d_rows = newValue;
        },
        sortField(newValue) {
            this.d_sortField = newValue;
        },
        sortOrder(newValue) {
            this.d_sortOrder = newValue;
        },
        multiSortMeta(newValue) {
            this.d_multiSortMeta = newValue;
        }
    },
    mounted() {
        this.allChildren = this.$children;
    },
    methods: {
        onNodeToggle(node) {
            const key = node.key;

            if (this.d_expandedKeys[key]) {
                delete this.d_expandedKeys[key];
                this.$emit('node-collapse', node);
            }
            else {
                this.d_expandedKeys[key] = true;
                this.$emit('node-expand', node);
            }

            this.d_expandedKeys = {...this.d_expandedKeys};
            this.$emit('update:expandedKeys', this.d_expandedKeys);
        },
        onNodeClick(event) {
            if (this.rowSelectionMode && event.node.selectable !== false) {
                const metaSelection = event.nodeTouched ? false : this.metaKeySelection;
                const _selectionKeys = metaSelection ? this.handleSelectionWithMetaKey(event) : this.handleSelectionWithoutMetaKey(event);

                this.$emit('update:selectionKeys', _selectionKeys);
            }
        },
        handleSelectionWithMetaKey(event) {
            const originalEvent = event.originalEvent;
            const node = event.node;
            const metaKey = (originalEvent.metaKey||originalEvent.ctrlKey);
            const selected = this.isNodeSelected(node);
            let _selectionKeys;

            if (selected && metaKey) {
                if (this.isSingleSelectionMode()) {
                    _selectionKeys = {};
                }
                else {
                    _selectionKeys = {...this.selectionKeys};
                    delete _selectionKeys[node.key];
                }

                this.$emit('node-unselect', node);
            }
            else {
                if (this.isSingleSelectionMode()) {
                    _selectionKeys = {};
                }
                else if (this.isMultipleSelectionMode()) {
                    _selectionKeys = !metaKey ? {} : (this.selectionKeys ? {...this.selectionKeys} : {});
                }

                _selectionKeys[node.key] = true;
                this.$emit('node-select', node);
            }

            return _selectionKeys;
        },
        handleSelectionWithoutMetaKey(event) {
            const node = event.node;
            const selected = this.isNodeSelected(node);
            let _selectionKeys;

            if (this.isSingleSelectionMode()) {
                if (selected) {
                    _selectionKeys = {};
                    this.$emit('node-unselect', node);
                }
                else {
                    _selectionKeys = {};
                    _selectionKeys[node.key] = true;
                    this.$emit('node-select', node);
                }
            }
            else {
                if (selected) {
                    _selectionKeys = {...this.selectionKeys};
                    delete _selectionKeys[node.key];

                    this.$emit('node-unselect', node);
                }
                else {
                    _selectionKeys = this.selectionKeys ? {...this.selectionKeys} : {};
                    _selectionKeys[node.key] = true;

                    this.$emit('node-select', node);
                }
            }

            return _selectionKeys;
        },
        onCheckboxChange(event) {
            this.$emit('update:selectionKeys', event.selectionKeys);

            if (event.check)
                this.$emit('node-select', event.node);
            else
                this.$emit('node-unselect', event.node);
        },
        isSingleSelectionMode() {
            return this.selectionMode === 'single';
        },
        isMultipleSelectionMode() {
            return this.selectionMode === 'multiple';
        },
        onPage(event) {
            this.d_first = event.first;
            this.d_rows = event.rows;

            this.$emit('update:first', this.d_first);
            this.$emit('update:rows', this.d_rows);
            this.$emit('page', event);
        },
        resetPage() {
            this.d_first = 0;
            this.$emit('update:first', this.d_first);
        },
        getColumnHeaderClass(column) {
            const sorted = this.isColumnSorted(column);

            return [column.headerClass,
                    {'p-sortable-column': column.sortable},
                    {'p-resizable-column': this.resizableColumns},
                    {'p-highlight': sorted}
            ];
        },
        isColumnSorted(column) {
            if (column.sortable)
                return this.sortMode === 'single' ? (this.d_sortField === (column.field || column.sortField)) : this.getMultiSortMetaIndex(column) > -1;
            else
                return false;
        },
        getSortableColumnIcon(column) {
            let sorted = false;
            let sortOrder = null;

            if (this.sortMode === 'single') {
                sorted =  this.d_sortField === (column.field || column.sortField);
                sortOrder = sorted ? this.d_sortOrder: 0;
            }
            else if (this.sortMode === 'multiple') {
                let metaIndex = this.getMultiSortMetaIndex(column);
                if (metaIndex > -1) {
                    sorted = true;
                    sortOrder = this.d_multiSortMeta[metaIndex].order;
                }
            }

            return [
                'p-sortable-column-icon pi pi-fw',
                {'pi-sort': !sorted},
                {'pi-sort-up': sorted && sortOrder > 0},
                {'pi-sort-down': sorted && sortOrder < 0},
            ];
        },
        getMultiSortMetaIndex(column) {
            let index = -1;

            for (let i = 0; i < this.d_multiSortMeta.length; i++) {
                let meta = this.d_multiSortMeta[i];
                if (meta.field === (column.field || column.sortField)) {
                    index = i;
                    break;
                }
            }

            return index;
        },
        onColumnHeaderClick(event, column) {
            if (column.sortable) {
                this.resetPage();
                const targetNode = event.target;

                if (DomHandler.hasClass(targetNode, 'p-sortable-column') || DomHandler.hasClass(targetNode, 'p-column-title')
                    || DomHandler.hasClass(targetNode, 'p-sortable-column-icon') || DomHandler.hasClass(targetNode.parentElement, 'p-sortable-column-icon')) {
                    DomHandler.clearSelection();

                    const columnField = column.field || column.sortField;
                    this.d_sortOrder = (this.d_sortField === columnField) ? this.d_sortOrder * -1 : this.defaultSortOrder;
                    this.d_sortField = columnField;

                    if(this.sortMode === 'multiple') {
                        let metaKey = event.metaKey || event.ctrlKey;
                        if (!metaKey) {
                            this.d_multiSortMeta = [];
                        }

                        this.addSortMeta({field: this.d_sortField, order: this.d_sortOrder});
                    }

                    this.$emit('update:sortField', this.d_sortFied);
                    this.$emit('update:sortOrder', this.d_sortOrder);
                    this.$emit('update:multiSortMeta', this.d_multiSortMeta);

                    this.$emit('sort', {
                        originalEvent: event,
                        sortField: this.d_sortField,
                        sortOrder: this.d_sortOrder,
                        multiSortMeta: this.d_multiSortMeta
                    });
                }
            }
        },
        addSortMeta(meta) {
            let index = -1;
            for (let i = 0; i < this.d_multiSortMeta.length; i++) {
                if (this.d_multiSortMeta[i].field === meta.field) {
                    index = i;
                    break;
                }
            }

            if(index >= 0)
                this.d_multiSortMeta[index] = meta;
            else
                this.d_multiSortMeta.push(meta);

            this.d_multiSortMeta = [...this.d_multiSortMeta];
        },
        sortSingle(nodes) {
            return this.sortNodesSingle(nodes);
        },
        sortNodesSingle(nodes) {
            let _nodes = [...nodes];

            _nodes.sort((node1, node2) => {
                const value1 = ObjectUtils.resolveFieldData(node1.data, this.d_sortField);
                const value2 = ObjectUtils.resolveFieldData(node2.data, this.d_sortField);
                let result = null;

                if (value1 == null && value2 != null)
                    result = -1;
                else if (value1 != null && value2 == null)
                    result = 1;
                else if (value1 == null && value2 == null)
                    result = 0;
                else if (typeof value1 === 'string' && typeof value2 === 'string')
                    result = value1.localeCompare(value2, undefined, { numeric: true });
                else
                    result = (value1 < value2) ? -1 : (value1 > value2) ? 1 : 0;

                return (this.d_sortOrder * result);
            });

            for (let i = 0; i < _nodes.length; i++) {
                if (_nodes[i].children && _nodes[i].children.length) {
                    _nodes[i].children = this.sortNodesSingle(_nodes[i].children);
                }
            }

            return _nodes;
        },
        sortMultiple(nodes) {
            return this.sortNodesMultiple(nodes);
        },
        sortNodesMultiple(nodes) {
            let _nodes = [...nodes];
            _nodes.sort((node1, node2) => {
                return this.multisortField(node1, node2, 0);
            });

            for (let i = 0; i < _nodes.length; i++) {
                if (_nodes[i].children && _nodes[i].children.length) {
                    _nodes[i].children = this.sortNodesMultiple(_nodes[i].children);
                }
            }

            return _nodes;
        },
        multisortField(node1, node2, index) {
            const value1 = ObjectUtils.resolveFieldData(node1.data, this.d_multiSortMeta[index].field);
            const value2 = ObjectUtils.resolveFieldData(node2.data, this.d_multiSortMeta[index].field);
            let result = null;

            if (value1 == null && value2 != null)
                result = -1;
            else if (value1 != null && value2 == null)
                result = 1;
            else if (value1 == null && value2 == null)
                result = 0;
            else {
                if (value1 === value2)  {
                    return (this.d_multiSortMeta.length - 1) > (index) ? (this.multisortField(node1, node2, index + 1)) : 0;
                }
                else {
                    if ((typeof value1 === 'string' || value1 instanceof String) && (typeof value2 === 'string' || value2 instanceof String))
                        return (this.d_multiSortMeta[index].order * value1.localeCompare(value2, undefined, { numeric: true }));
                    else
                        result = (value1 < value2) ? -1 : 1;
                }
            }

            return (this.d_multiSortMeta[index].order * result);
        },
        filter(value) {
            let filteredNodes = [];
            const strict = this.filterMode === 'strict';
            let valueChanged = false;

            for (let node of value) {
                let copyNode = {...node};
                let localMatch = true;
                let globalMatch = false;

                for (let j = 0; j < this.columns.length; j++) {
                    let col = this.columns[j];
                    let filterField = col.field;

                    //local
                    if (this.filters.hasOwnProperty(col.field)) {
                        let filterMatchMode = col.filterMatchMode;
                        let filterValue = this.filters[col.field];
                        let filterConstraint = FilterUtils[filterMatchMode];
                        let paramsWithoutNode = {filterField, filterValue, filterConstraint, strict};

                        if ((strict && !(this.findFilteredNodes(copyNode, paramsWithoutNode) || this.isFilterMatched(copyNode, paramsWithoutNode))) ||
                            (!strict && !(this.isFilterMatched(copyNode, paramsWithoutNode) || this.findFilteredNodes(copyNode, paramsWithoutNode)))) {
                                localMatch = false;
                        }

                        if (!localMatch) {
                            break;
                        }
                    }

                    //global
                    if (this.hasGlobalFilter && !globalMatch) {
                        let copyNodeForGlobal = {...copyNode};
                        let globalFilterValue = this.props.globalFilter;
                        let globalFilterConstraint = ObjectUtils.filterConstraints['contains'];
                        let globalFilterParamsWithoutNode = {filterField, globalFilterValue, globalFilterConstraint, strict};

                        if ((strict && (this.findFilteredNodes(copyNodeForGlobal, globalFilterParamsWithoutNode) || this.isFilterMatched(copyNodeForGlobal, globalFilterParamsWithoutNode))) ||
                            (!strict && (this.isFilterMatched(copyNodeForGlobal, globalFilterParamsWithoutNode) || this.findFilteredNodes(copyNodeForGlobal, globalFilterParamsWithoutNode)))) {
                                globalMatch = true;
                                copyNode = copyNodeForGlobal;
                        }
                    }
                }

                let matches = localMatch;
                if (this.hasGlobalFilter) {
                    matches = localMatch && globalMatch;
                }

                if (matches) {
                    filteredNodes.push(copyNode);
                }

                valueChanged = valueChanged || !localMatch || globalMatch;
            }

            this.$emit('filter', {
                filters: this.filters,
                filteredValue: filteredNodes
            });

            return valueChanged ? filteredNodes : value;
        },
        findFilteredNodes(node, paramsWithoutNode) {
            if (node) {
                let matched = false;
                if (node.children) {
                    let childNodes = [...node.children];
                    node.children = [];
                    for (let childNode of childNodes) {
                        let copyChildNode = {...childNode};
                        if (this.isFilterMatched(copyChildNode, paramsWithoutNode)) {
                            matched = true;
                            node.children.push(copyChildNode);
                        }
                    }
                }

                if (matched) {
                    return true;
                }
            }
        },
        isFilterMatched(node, {filterField, filterValue, filterConstraint, strict}) {
            let matched = false;
            let dataFieldValue = ObjectUtils.resolveFieldData(node.data, filterField);
            if (filterConstraint(dataFieldValue, filterValue)) {
                matched = true;
            }

            if (!matched || (strict && !this.isNodeLeaf(node))) {
                matched = this.findFilteredNodes(node, {filterField, filterValue, filterConstraint, strict}) || matched;
            }

            return matched;
        },
        isNodeSelected(node) {
            return (this.selectionMode && this.selectionKeys) ? this.selectionKeys[node.key] === true : false;
        },
        isNodeLeaf(node) {
            return node.leaf === false ? false : !(node.children && node.children.length);
        },
        onColumnResizeStart(event) {
            let containerLeft = DomHandler.getOffset(this.$el).left;
            this.resizeColumnElement = event.target.parentElement;
            this.columnResizing = true;
            this.lastResizeHelperX = (event.pageX - containerLeft + this.$el.scrollLeft);

            this.bindColumnResizeEvents();
        },
        onColumnResize(event) {
            let containerLeft = DomHandler.getOffset(this.$el).left;
            DomHandler.addClass(this.$el, 'p-unselectable-text');
            this.$refs.resizeHelper.style.height = this.$el.offsetHeight + 'px';
            this.$refs.resizeHelper.style.top = 0 + 'px';
            this.$refs.resizeHelper.style.left = (event.pageX - containerLeft + this.$el.scrollLeft) + 'px';

            this.$refs.resizeHelper.style.display = 'block';
        },
        onColumnResizeEnd() {
            let delta = this.$refs.resizeHelper.offsetLeft - this.lastResizeHelperX;
            let columnWidth = this.resizeColumnElement.offsetWidth;
            let newColumnWidth = columnWidth + delta;
            let minWidth = this.resizeColumnElement.style.minWidth||15;

            if(columnWidth + delta > parseInt(minWidth, 10)) {
                if(this.columnResizeMode === 'fit') {
                    let nextColumn = this.resizeColumnElement.nextElementSibling;
                    let nextColumnWidth = nextColumn.offsetWidth - delta;

                    if(newColumnWidth > 15 && nextColumnWidth > 15) {
                        this.resizeColumnElement.style.width = newColumnWidth + 'px';
                        if(nextColumn) {
                            nextColumn.style.width = nextColumnWidth + 'px';
                        }
                    }
                }
                else if(this.columnResizeMode === 'expand') {
                    this.$refs.table.style.width = this.$refs.table.offsetWidth + delta + 'px';
                    this.resizeColumnElement.style.width = newColumnWidth + 'px';
                }

                this.$emit('column-resize-end', {
                    element: this.resizeColumnElement,
                    delta: delta
                });
            }

            this.$refs.resizeHelper.style.display = 'none';
            this.resizeColumn = null;
            DomHandler.removeClass(this.$el, 'p-unselectable-text');

            this.unbindColumnResizeEvents();
        },
        bindColumnResizeEvents() {
            if (!this.documentColumnResizeListener) {
                this.documentColumnResizeListener = document.addEventListener('mousemove', () => {
                    if(this.columnResizing) {
                        this.onColumnResize(event);
                    }
                });
            }

            if (!this.documentColumnResizeEndListener) {
                this.documentColumnResizeEndListener = document.addEventListener('mouseup', () => {
                    if(this.columnResizing) {
                        this.columnResizing = false;
                        this.onColumnResizeEnd();
                    }
                });
            }

        },
        unbindColumnResizeEvents() {
            if (this.documentColumnResizeListener) {
                document.removeEventListener('document', this.documentColumnResizeListener);
                this.documentColumnResizeListener = null;
            }

            if (this.documentColumnResizeEndListener) {
                document.removeEventListener('document', this.documentColumnResizeEndListener);
                 this.documentColumnResizeEndListener = null;
            }
        }
    },
    computed: {
        containerClass() {
            return ['p-treetable p-component', {
                'p-treetable-hoverable-rows': (this.rowHover || this.rowSelectionMode),
                'p-treetable-auto-layout': this.autoLayout,
                'p-treetable-resizable': this.resizableColumns,
                'p-treetable-resizable-fit': this.resizableColumns && this.columnResizeMode === 'fit',
            }];
        },
        columns() {
            if (this.allChildren) {
                return this.allChildren.filter(child =>  child.$options._propKeys.indexOf('columnKey') !== -1);
            }
            return [];
        },
        processedData() {
            if (this.lazy) {
                return this.value;
            }
            else {
                if (this.value && this.value.length) {
                    let data = this.value;

                    if (this.sorted) {
                        if(this.sortMode === 'single')
                            data = this.sortSingle(data);
                        else if(this.sortMode === 'multiple')
                            data = this.sortMultiple(data);
                    }

                    if (this.hasFilters) {
                        data = this.filter(data);
                    }

                    return data;
                }
                else {
                    return null;
                }
            }
        },
        dataToRender() {
            const data = this.processedData;

            if (this.paginator) {
                const first = this.lazy ? 0 : this.d_first;
                return data.slice(first, first + this.d_rows);
            }
            else {
                return data;
            }
        },
        empty() {
            const data = this.processedData;
            return (!data || data.length === 0);
        },
        sorted() {
            return this.d_sortField || (this.d_multiSortMeta && this.d_multiSortMeta.length > 0);
        },
        hasFooter() {
            let hasFooter = false;

            for (let col of this.columns) {
                if (col.footer || col.$scopedSlots.footer) {
                    hasFooter = true;
                    break;
                }
            }

            return hasFooter;
        },
        hasFilters() {
            return this.filters && Object.keys(this.filters).length > 0 && this.filters.constructor === Object;
        },
        hasGlobalFilter() {
            return this.filters && this.filters.hasOwnProperty('global');
        },
        paginatorTop() {
            return this.paginator && (this.paginatorPosition !== 'bottom' || this.paginatorPosition === 'both');
        },
        paginatorBottom() {
            return this.paginator && (this.paginatorPosition !== 'top' || this.paginatorPosition === 'both');
        },
        singleSelectionMode() {
            return this.selectionMode && this.selectionMode === 'single';
        },
        multipleSelectionMode() {
            return this.selectionMode && this.selectionMode === 'multiple';
        },
        rowSelectionMode() {
            return this.singleSelectionMode || this.multipleSelectionMode;
        },
        totalRecordsLength() {
            if (this.lazy) {
                return this.totalRecords;
            }
            else {
                const data = this.processedData;
                return data ? data.length : 0;
            }
        },
        loadingIconClass() {
            return ['p-treetable-loading-icon pi-spin', this.loadingIcon];
        }
    },
    components: {
        'TTColumnSlot': TreeTableColumnSlot,
        'TTRow': TreeTableRowLoader,
        'TTPaginator': Paginator,
    }
}
</script>

<style>
.p-treetable {
    position: relative;
}

.p-treetable table {
    border-collapse: collapse;
    width: 100%;
    table-layout: fixed;
}

.p-treetable-toggler {
    cursor: pointer;
    display: inline-block;
}

.p-treetable .p-treetable-thead > tr > th,
.p-treetable .p-treetable-tbody > tr > td,
.p-treetable .p-treetable-tfoot > tr > td {
    padding: .25em .5em;
}

.p-treetable .p-treetable-thead > tr > th .p-column-title {
    vertical-align: middle;
    user-select: none;
}

.p-treetable .p-sortable-column {
    cursor: pointer;
}

.p-treetable .p-sortable-column-icon {
    vertical-align: middle;
}

.p-treetable-auto-layout > .p-treetable-wrapper {
    overflow-x: auto;
}

.p-treetable-auto-layout > .p-treetable-wrapper > table {
    table-layout: auto;
}

.p-treetable-hoverable-rows .p-treetable-tbody > tr.p-highlight {
    cursor: pointer;
}

/* Sections */
.p-treetable-header,
.p-treetable-footer {
    padding: .25em .5em;
    text-align: center;
    font-weight: bold;
}

.p-treetable-header {
    border-bottom: 0 none;
}

.p-treetable-footer {
    border-top: 0 none;
}

/* Paginator */
.p-treetable .p-paginator-top {
    border-bottom: 0 none;
}

.p-treetable .p-paginator-bottom {
    border-top: 0 none;
}

/* Scrollable */
.p-treetable-scrollable-wrapper {
    position: relative;
}
.p-treetable-scrollable-header,
.p-treetable-scrollable-footer {
    overflow: hidden;
    border: 0 none;
}

.p-treetable-scrollable-body {
    overflow: auto;
    position: relative;
}

.p-treetable-scrollable-body > table > .p-treetable-tbody > tr:first-child > td {
    border-top: 0 none;
}

.p-treetable-virtual-table {
    position: absolute;
}

/* Frozen Columns */
.p-treetable-frozen-view .p-treetable-scrollable-body {
    overflow: hidden;
}

.p-treetable-frozen-view > .p-treetable-scrollable-body > table > .p-treetable-tbody > tr > td:last-child {
    border-right: 0 none;
}

.p-treetable-unfrozen-view {
    position: absolute;
    top: 0px;
}

/* Filter */
.p-column-filter {
    width: 100%;
}

/* Resizable */
.p-treetable-resizable > .p-treetable-tablewrapper {
    overflow-x: auto;
}

.p-treetable-resizable .p-treetable-thead > tr > th,
.p-treetable-resizable .p-treetable-tfoot > tr > td,
.p-treetable-resizable .p-treetable-tbody > tr > td {
    overflow: hidden;
}

.p-treetable-resizable .p-resizable-column {
    background-clip: padding-box;
    position: relative;
}

.p-treetable-resizable-fit .p-resizable-column:last-child .p-column-resizer {
    display: none;
}

.p-treetable .p-column-resizer {
    display: block;
    position: absolute !important;
    top: 0;
    right: 0;
    margin: 0;
    width: .5em;
    height: 100%;
    padding: 0px;
    cursor:col-resize;
    border: 1px solid transparent;
}

.p-treetable .p-column-resizer-helper {
    width: 1px;
    position: absolute;
    z-index: 10;
    display: none;
}

/* Selection */
.p-treetable .p-treetable-checkbox {
    margin: 0 .5em 0 .25em;
    vertical-align: middle;
}

/* Edit */
.p-treetable .p-treetable-tbody > tr > td.p-cell-editing .p-component {
    width: 100%;
}

/* Reorder */
.p-treetable-reorder-indicator-up,
.p-treetable-reorder-indicator-down {
    position: absolute;
    display: none;
}

/* Responsive */
.p-treetable-responsive .p-treetable-tbody > tr > td .p-column-title {
    display: none;
}

/* Loader */
.p-treetable .p-treetable-loading-overlay {
    position: absolute;
    width: 100%;
    height: 100%;
    -ms-filter: "progid:DXImageTransform.Microsoft.Alpha(Opacity=10)";
    opacity: 0.1;
    z-index: 1;
}

.p-treetable .p-treetable-loading-content {
    position: absolute;
    left: 50%;
    top: 50%;
    z-index: 2;
    margin-top: -1em;
    margin-left: -1em;
}

.p-treetable .p-treetable-loading-icon {
    font-size: 2em;
}
</style>