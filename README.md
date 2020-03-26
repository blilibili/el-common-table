# server_components
日常工作组件封装

commonTable (数据控制表格组件)
方便于日常工作表格 或者表格增删改查渲染渲染 
数据格式：

[
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
