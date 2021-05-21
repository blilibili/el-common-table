# tables
基于饿了么表格组件封装

commonTable (数据控制表格组件)
方便于日常工作表格 或者表格增删改查渲染渲染 
数据格式：

const selectData = [
     // 一般的展示
   
    {
        label: '',   // 表格列名称
        prop: 'stockPrice',  // 后端对应的数据字段
        width: 120,  // 表格列的宽度
        isShow: true,  // 该列是否展示
        isEdit: false  // 改列是否可编辑
    },
    
    {
            label: '',
            prop: '',
            width: 160,
            isShow: true,
            isEdit: true,  // 可编辑
            inputType: 'string',  // 编辑的控件类型  string 输入框  select 下拉  switch 开关
            validateType: 'int'  // int 最后取整  float 最后取浮点数
     },
     // key value 展示
     {
            label: '渠道',
            prop: 'labelChannel',
            filter: 'keyValue',
            options: ['' , '钻石会员'],
            width: 120,
            isShow: true,
            isEdit: false
    },
    // 自定义slot插槽
     {
                label: '图片',
                prop: 'slot',
                slotName: 'labelImgSlot',
                width: 150,
                isShow: true,
                isEdit: false
      },
]

表格代码示例：
<CommonTable               
  :is-selection="true"
  :is-has-opera="false"
  :is-show-index="false"
  :t-column-data="selectData"
  :t-data="[{stockPrice: 'test'}]"
  @changeRow="changeRowPersonList"
/>

若是插槽模式：
<div slot="labelImgSlot" slot-scope="scope">
       插槽内容
</div>

操作的插槽
<div slot="operate" slot-scope="scope">
   
</div>

表格的属性 ：
1. is-show-index  boolean  是否展示序列号
2. operating-width  string 操作栏的长度
3. t-column-data list  col 表格字段的数组
4. t-data   list   表格的数据
5. isHasOpera  bolean 是否展示操作
6. isReserveSelect  boolean  是否跨页全选
7. isSelection boolean  是否多选
8. emitSelectionName  string  自定义多选时的回调方法

表格的方法: 
1. inputBlurCallBack  返回{filed: filed 字段名, index: index 数组下标, value: row 值, arr: arr 整体数组 }
2. emitSelectionName  返回当前选中的全选数组


2021/02/10  更新基于饿了么城市选择组件  Address
基于 element-china-area-data 的城市码
allText ---- 设置 全选的名称  像’全部‘， ’不限‘
@changeSelectionAddress  接收选择的数组，目前是匹配文字数组返回
