<template>
  <div :id="id" ref="table" class="advanced-table">
    <a-spin :spinning="loading">
      <div :class="['header-bar', size]">
        <div class="info">
          <template v-if="selectedRows && selectedRows.length">
            <a-icon type="info-circle" /> 已选 <span>{{ selectedRows.length }}</span> 项 <a class="clear" @click="onClear">清空</a>
          </template>
        </div>
        <div class="actions">
          <!-- 表格头部操作 -->
          <a-button-group v-if="headerOperation">
            <slot :name="headerOperation" />
          </a-button-group>
          <a-tooltip title="刷新">
            <a-icon class="action" :type="loading ? 'loading' : 'reload'" @click="refresh" />
          </a-tooltip>
          <action-size v-model="sSize" class="action" />
          <a-tooltip title="列配置">
            <action-columns :columns="columns" class="action">
              <template v-for="slot in slots" :slot="slot">
                <slot :name="slot" />
              </template>
            </action-columns>
          </a-tooltip>
          <a-tooltip title="全屏">
            <a-icon class="action" :type="fullScreen ? 'fullscreen-exit' : 'fullscreen'" @click="toggleScreen" />
          </a-tooltip>
        </div>
      </div>
      <a-table
        bordered
        :columns="visibleColumns"
        :data-source="dataSource"
        :pagination="pagination"
        :row-key="rowKey"
        :size="sSize"
        :row-selection="selectedRows ? {selectedRowKeys: selectedRowKeys, onChange: updateSelect} : undefined"
        @expandedRowsChange="onExpandedRowsChange"
        @change="onChange"
        @expand="onExpand"
      >
        <template v-for="slot in scopedSlots " :slot="slot" slot-scope="text, record, index">
          <slot :name="slot" v-bind="{text, record, index}" />
        </template>
        <template v-for="slot in slots" :slot="slot">
          <slot :name="slot" />
        </template>
        <template :slot="$scopedSlots.expandedRowRender ? 'expandedRowRender' : ''" slot-scope="record, index, indent, expanded">
          <slot v-bind="{record, index, indent, expanded}" :name="$scopedSlots.expandedRowRender ? 'expandedRowRender' : ''" />
        </template>
      </a-table>
    </a-spin>
  </div>
</template>

<script>
import mixins from '@/mixins/fullScreen'
import ActionSize from '@/components/table/advance/ActionSize'
import ActionColumns from '@/components/table/advance/ActionColumns'
export default {
  name: 'AdvanceTable',
  components: { ActionColumns, ActionSize },
  mixins: [mixins],
  props: {
    tableLayout: { type: String, default: '' },
    bordered: Boolean,
    childrenColumnName: { type: String, default: 'children' },
    columns: { type: Array, default: () => [] },
    components: { type: Object, default: () => {} },
    dataSource: { type: Array, default: () => [] },
    defaultExpandAllRows: { type: Boolean, default: false },
    expandedRowKeys: { type: Array, default: () => [] },
    selectedRows: {
      type: Array,
      default: () => []
    },
    // eslint-disable-next-line vue/require-default-prop
    expandedRowRender: [Function],
    expandIcon: { type: Function, default: () => {} },
    expandRowByClick: Boolean,
    expandIconColumnIndex: { type: Number, default: 0 },
    footer: { type: Function, default: () => {} },
    indentSize: { type: Number, default: 0 },
    loading: Boolean,
    locale: { type: Object, default: () => {} },
    pagination: {
      type: [Object, Boolean],
      default: false
    },
    rowClassName: { type: Function, default: () => {} },
    rowKey: {
      type: [String, Function],
      default: 'id'
    },
    headerOperation: { type: String, default: '' },
    rowSelection: {
      type: Object,
      default: () => {
        return {
          type: 'checkbox'
        }
      }
    },
    scroll: { type: Object, default: () => {} },
    showHeader: { type: Boolean, default: true },
    size: { type: String, default: '' },
    title: { type: String, default: '' },
    customHeaderRow: { type: Function, default: () => {} },
    customRow: { type: Function, default: () => {} },
    getPopupContainer: { type: Function, default: () => {} },
    transformCellText: { type: Function, default: () => {} },
    formatConditions: { type: Boolean, default: false }
  },
  provide() {
    return {
      table: this
    }
  },
  data() {
    return {
      id: `${new Date().getTime()}-${Math.floor(Math.random() * 10)}`,
      sSize: this.size || 'default',
      fullScreen: false,
      conditions: {},
      isInit: 0
    }
  },
  computed: {
    slots() {
      return Object.keys(this.$slots).filter(slot => slot !== 'title')
    },
    scopedSlots() {
      return Object.keys(this.$scopedSlots).filter(slot => slot !== 'expandedRowRender' && slot !== 'title')
    },
    visibleColumns() {
      return this.columns.filter(col => col.visible)
    },
    selectedRowKeys() {
      return this.selectedRows.map(record => {
        return (typeof this.rowKey === 'function') ? this.rowKey(record) : record[this.rowKey]
      })
    }
  },
  watch: {
    loading(bool) {
      (!bool && this.isInit) && this.$message.success('刷新成功!')
    }
  },
  methods: {
    // 表格数据刷新
    refresh() {
      this.isInit++
      this.$emit('refresh', this.conditions)
    },
    onExpandedRowsChange(expandedRows) {
      this.$emit('expandedRowsChange', expandedRows)
    },
    onChange(pagination, filters, sorter, options) {
      this.$emit('change', pagination, filters, sorter, options)
    },
    onExpand(expanded, record) {
      this.$emit('expand', expanded, record)
    },
    updateSelect(selectedRowKeys, selectedRows) {
      this.$emit('update:selectedRows', selectedRows)
      this.$emit('selectedRowChange', selectedRowKeys, selectedRows)
    },
    toggleScreen() {
      const el = this.$refs.table
      if (this.fullScreen) {
        this.$_outFullScreen(el)
      } else {
        this.$_inFullScreen(el)
      }
    },
    onClear() {
      this.updateSelect([], [])
      this.$emit('clear')
    }
  }
}
</script>

<style scoped lang="less">
.advanced-table{
  overflow-y: auto;
  background-color: @component-background;
  .header-bar{
    display: flex;
    align-items: center;
    justify-content: space-between;
    padding: 10px;
    &.middle{
      padding: 12px 16px;
    }
    &.small{
      padding: 8px 12px;
      border: 1px solid @border-color;
      border-bottom: 0;
      .title{
        font-size: 16px;
      }
    }
    .title{
      transition: all 0.3s;
      font-size: 18px;
      color: @title-color;
      font-weight: 700;
    }
    .search{
      flex: 1;
      text-align: right;
      margin: 0 24px;
    }
    .actions{
      text-align: right;
      font-size: 17px;
      color: @text-color;
      display: flex;
      align-items: center;
      justify-content: flex-end;
      transition: all 0.3s;
      .action{
        margin: 0 8px;
        cursor: pointer;
        &:hover{
          color: @primary-color;
        }
      }
    }
  }
}
</style>
