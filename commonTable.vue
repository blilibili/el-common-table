<template>
  <!-- 多选返回事件 changeRow -->
  <div>
    <!--表格-->
    <el-table :data="tData"
              row-key="skuNo"
              style="width: 100%;position: relative;"
              class="commonTable"
              border
              :max-height="tHeight"
              @selection-change="handleSelectionChange"
              ref="commonTableRef"
    >
      <el-table-column
              :reserve-selection="$props.isReserveSelect"
              v-if="$props.isSelection"
              type="selection"
              width="55">
      </el-table-column>
      <el-table-column
              v-if="$props.isShowIndex"
              type="index"
              width="50"
              label="序号"
      ></el-table-column>
      <el-table-column
              align="left"
              :show-overflow-tooltip="true"
              v-for="(item , index) in getCol"
              :key="item.prop + '-' + index + '-' + 'only'"
              :prop="item.prop"
              :label="item.label"
              :width="item.width">
        <template slot-scope="scope">
<!--          <div v-if="item.filter && item.filter === 'date'">-->
<!--            {{scope.row[item.prop] | transformDate}}-->
<!--          </div>-->
<!--          <div v-else-if="item.filter && item.filter === 'times'">-->
<!--            {{scope.row[item.prop] | transformTimes}}-->
<!--          </div>-->
          <div v-else-if="item.filter && item.filter === 'percent'">
            {{scope.row[item.prop]}}%
          </div>
          <!--有值存在-->
          <div v-else-if="item.filter && item.filter === 'keyValue' && scope.row[item.prop] !== undefined && scope.row[item.prop] !== null">
            {{item.options[scope.row[item.prop]]}}
          </div>
          <div v-else-if="item.isEdit && item.inputType === 'string'">
            <input type="text" style="padding: 3px;border-radius: 4px;border: 1px solid lightgrey;" v-model="scope.row[item.prop]" @blur="blurInputValue(item.prop , scope.$index , scope.row[item.prop] , item.validateType , scope.row)" />
          </div>
          <!--开关控件-->
          <div v-else-if="item.isEdit && item.inputType === 'switch'">
            <el-switch
                    @change="updateStatus(item.prop , scope.row , scope.$index , item.switchOpt)"
                    :value="scope.row[item.prop]"
                    :active-value="item.switchOpt[0]"
                    :inactive-value="item.switchOpt[1]"
                    active-color="#13ce66"
                    inactive-color="#999999">
            </el-switch>
          </div>
          <!--单选下拉控件-->
          <div v-else-if="item.isEdit && item.inputType === 'select'">
            <el-select v-model="scope.row[item.prop]" @change="refreshView" placeholder="请选择">
              <el-option
                      v-for="(im , idx) in item.dataSource"
                      :key="idx"
                      :label="im.label"
                      :value="im.value">
              </el-option>
            </el-select>
          </div>
          <!--自定义 slotName 插槽名称-->
          <div v-else-if="item.prop === 'slot'">
            <slot :name="item.slotName"
                  :row="scope.row" :arrIndex="scope.$index"></slot>
          </div>
          <div v-html="scope.row[item.prop]" v-else>

          </div>
        </template>
      </el-table-column>

      <el-table-column fixed="right" :width="parseInt($props.operatingWidth)" label="操作" v-if="$props.isHasOpera">
        <template slot-scope="scope">
          <slot name="operate"
                :row="scope.row" :arrIndex="scope.$index"></slot>
        </template>
      </el-table-column>
    </el-table>
  </div>
</template>

<script>

// import {FormatDate} from '@/utils/filters'
export default {
  name: 'RulesCommonTable',
  props: {
    // 控制操作的长度默认是空
    operatingWidth: {
      type: String || Number,
      default: ''
    },
    // 是否显示序列号
    isShowIndex: {
      type: Boolean,
      default: true
    },
    // 是否展示操作
    isHasOpera: {
      type: Boolean,
      default: true
    },
    // 是否展示跨页全选
    isReserveSelect: {
      type: Boolean,
      default: false
    },
    // 当前第几页
    currentPage: {
      type: Number,
      default: 1
    },
    // 默认发射选中的方法名
    emitSelectionName: {
      type: String,
      default: 'changeRow'
    },
    isSelection: {
      type: Boolean,
      default: false
    },
    tColumnData: {
      type: Array,
      default: () => []
    },
    tData: {
      type: Array,
      default: () => []
    },
    isSelect: {
      type: Boolean,
      default: false
    },
    tHeight: {
      type: Number,
      default: 520
    }
  },
  mounted() {
    this.$nextTick(() => {
      let calWidth = 0
      this.$props.tColumnData.forEach((v) => {
        calWidth = parseInt(calWidth, 10) + parseInt(v.width, 10)
      })
      const documentWidth = parseInt(this.$refs.commonTableRef.bodyWidth.split('p')[0], 10)

      if (calWidth < documentWidth && document.body.clientWidth > 1366) {
        this.$refs.commonTableRef.$el.style.width = '100%'
      }
    })
  },
  methods: {
    blurInputValue(filed, index, row, validateType, arr) {
      // 小数点取后两位
      if (validateType === 'int') {
        this.$props.tData[index][filed] = parseInt(this.$props.tData[index][filed], 10)
      } else if (validateType === 'float') {
        this.$props.tData[index][filed] = parseFloat(parseFloat(this.$props.tData[index][filed]).toFixed(2))
      }
      this.$props.tData.splice(0, 0)
      this.$emit('inputBlurCallBack', {
        filed: filed, index: index, value: row, arr: arr
      })
    },
    refreshView() {
      this.tData.splice(0, 0)
    },
    updateStatus(col, row, index, opt) {
      const currentData = row[col]
      let updateData = ''
      if (opt.indexOf(currentData) === 0) {
        [updateData] = [opt[1]]
      } else {
        [updateData] = [opt[0]]
      }
      this.$emit('switchUpdate', {
        col: col, data: row, index: index, updateValue: updateData
      })
    },
    handleSelectionChange(val) {
      this.$emit(this.$props.emitSelectionName, val)
    }
  },
  computed: {
    // 过滤isShow 不展示的值
    getCol() {
      const tmp = []
      this.$props.tColumnData.forEach((v) => {
        if (v.isShow) {
          tmp.push(v)
        }
      })
      return tmp
    }
  },
  filters: {
    // transformDate(value) {
    //   if (value) {
    //     return FormatDate(new Date(value))
    //   }
    //   return ''
    // },
    // transformTimes(value) {
    //   if (value) {
    //     return FormatDate(new Date(value)).split(' ')[1]
    //   }
    //   return ''
    // }
  }
}
</script>

<style scoped>
.commonTable /deep/ .el-table__body .cell {
  padding: 0px 8px !important;
}
</style>
