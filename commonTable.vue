<template>
  <!-- 多选返回事件 changeRow -->
  <div>
    <!--表格-->
    <el-table :data="tData"
              row-key="skuNo"
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
              align="center"
              :show-overflow-tooltip="true"
              v-for="(item , index) in getCol"
              :key="item.prop"
              :prop="item.prop"
              :label="item.label"
              :width="item.width">
        <template slot-scope="scope">
          <div v-if="item.filter && item.filter === 'date'">
            {{scope.row[item.prop] | transformDate}}
          </div>
          <div v-if="item.filter && item.filter === 'percent'">
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
          <div v-else>
            {{scope.row[item.prop]}}
          </div>
        </template>
      </el-table-column>

      <el-table-column label="操作" v-if="$props.isHasOpera">
        <template slot-scope="scope">
          <slot name="operate"
                :row="scope.row" :arrIndex="scope.$index"></slot>
        </template>
      </el-table-column>
    </el-table>
  </div>
</template>

<script>
  import { FormatDate } from '@core/filters'
export default {
  name: 'RulesCommonTable',
  props: {
    // 是否显示序列号
    isShowIndex:{
      type: Boolean,
      default: true
    },
    // 是否展示操作
    isHasOpera:{
      type: Boolean,
      default: true
    },
    // 是否展示跨页全选
    isReserveSelect:{
      type: Boolean,
      default: false
    },
    // 当前第几页
    currentPage:{
      type: Number,
      default: 1
    },
    // 默认发射选中的方法名
    emitSelectionName:{
      type: String,
      default: 'changeRow'
    },
    isSelection:{
      type: Boolean,
      default: false
    },
    tColumnData: {
      type: Array,
      default: []
    },
    tData: {
      type: Array,
      default: []
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
  watch:{
    '$props.tData'(val){
      // console.log(this.$props.toSelectedRow , this.$props.currentPage)
    }
  },
  mounted(){
    this.$nextTick(() => {
      let calWidth = 0
      this.$props.tColumnData.forEach((v , k) => {
        calWidth = parseInt(calWidth) + parseInt(v.width)
      })
      let documentWidth = parseInt(this.$refs.commonTableRef.bodyWidth.split('p')[0])

      if(calWidth < documentWidth && document.body.clientWidth > 1366){
        this.$refs.commonTableRef.$el.style.width = calWidth + 51 + 'px'
      }
    })
  },
  methods:{
    /**
     * 时间戳转日期格式
     * @param date 时间
     * @param type 时间类型 yyyy-MM-dd HH:mm:ss ww
     * return string
     */
    FormatDate (date, type) {
      if (!date) return ''
      date = typeof date === 'number' ? new Date(date) : date
      type = type || 'yyyy-MM-dd HH:mm:ss'
      let obj = {
        'y': date.getFullYear(), // 年份，注意必须用getFullYear
        'M': date.getMonth() + 1, // 月份，注意是从0-11
        'd': date.getDate(), // 日期
        'q': Math.floor((date.getMonth() + 3) / 3), // 季度
        'w': date.getDay(), // 星期，注意是0-6
        'H': date.getHours(), // 24小时制
        'h': date.getHours() % 12 === 0 ? 12 : date.getHours() % 12, // 12小时制
        'm': date.getMinutes(), // 分钟
        's': date.getSeconds(), // 秒
        'S': date.getMilliseconds() // 毫秒
      }
      let week = ['天', '一', '二', '三', '四', '五', '六']
      for (let i in obj) {
        type = type.replace(new RegExp(i + '+', 'g'), function (m) {
          let val = obj[i] + ''
          if (i === 'w') return (m.length > 2 ? '星期' : '周') + week[val]
          for (let j = 0, len = val.length; j < m.length - len; j++) val = '0' + val
          return m.length === 1 ? val : val.substring(val.length - m.length)
        })
      }
      return type
    },
    blurInputValue(filed , index , row , validateType , arr){
      // 小数点取后两位
      if(validateType === 'int'){
        this.$props.tData[index][filed] = parseInt(this.$props.tData[index][filed])
      }else if(validateType === 'float'){
        this.$props.tData[index][filed] = parseFloat(parseFloat(this.$props.tData[index][filed]).toFixed(2))
      }else{}
      this.$props.tData.splice(0,0)
      this.$emit('inputBlurCallBack' , {filed: filed , index: index , value: row , arr: arr})
    },
    refreshView(){
      this.tData.splice(0,0)
    },
    updateStatus(col , row , index , opt){
      let currentData = row[col]
      let updateData = ''
      if(opt.indexOf(currentData) === 0){
        updateData = opt[1]
      }else{
        updateData = opt[0]
      }
      this.$emit('switchUpdate' , {col: col , data: row , index: index , updateValue: updateData})
    },
    handleSelectionChange(val){
      this.$emit(this.$props.emitSelectionName , val)
    }
  },
  computed:{
    // 过滤isShow 不展示的值
    getCol(){
      let tmp = []
      this.$props.tColumnData.forEach((v , k) => {
        if(v.isShow){
          tmp.push(v)
        }
      })
      return tmp
    }
  },
  filters: {
    transformDate: function(value) {
      return this.FormatDate(new Date(value))
    }
  }
}
</script>

<style scoped>
.commonTable /deep/ .el-table__body .cell {
  padding: 0px 8px !important;
}
</style>
