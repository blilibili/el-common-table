<template>
    <div>
        <el-cascader
                v-model="ruleForm.name10"
                size="large"
                :options="options"
                :props="cascaderProps"
                collapse-tags
                clearable
                @change="changeAddressMethods"
                style="width:300px;"
        />
    </div>
</template>

<script>
  import { regionData, TextToCode, CodeToText } from 'element-china-area-data'
  export default {
    name: "Address",
    mounted() {
      let tmpRegionObj = {
        children: [
          {
            label: this.$props.allText,
            value: this.$props.allText,
            children: [
              {
                label: this.$props.allText,
                value: this.$props.allText,
              }
            ]
          }
        ],
        label: this.$props.allText,
        value: this.$props.allText
      }

      let regionOrigin = JSON.parse(JSON.stringify(regionData))
      regionOrigin.forEach((item) => {
        let tmpObj = {
          label: this.$props.allText,
          value: this.$props.allText
        }
        item.children.unshift(tmpObj)
        item.children.forEach((childItem) => {
          let tmpObj = {
            label: this.$props.allText,
            value: this.$props.allText
          }
          if(childItem.children) {
            childItem.children.unshift(tmpObj)
          }
        })
      })
      let textCodeOrigin = JSON.parse(JSON.stringify(TextToCode))

      const filterChar = regionOrigin.filter((res) => {
        return res.label === this.$props.allText
      })
      if(filterChar.length === 0) {
        regionOrigin.unshift(tmpRegionObj)
      }
      this.options = regionOrigin
      this.textCode = textCodeOrigin
    },
    props: {
      allText: {
        type: String,
        default: '不限'
      }
    },
    data() {
      return {
        codeText: CodeToText,
        textCode: TextToCode,
        ruleForm: {
          name10: ''
        },
        cascaderProps: {
          multiple: true
        },
        options: regionData,
      }
    },
    methods: {
      changeAddressMethods(val) {
        let returnArr = []
        val.forEach((item) => {
          let tmp = {
            province: item[0] === this.$props.allText?this.$props.allText:CodeToText[item[0]],
            city: item[1] === this.$props.allText?this.$props.allText:CodeToText[item[1]],
            area: item[2] === this.$props.allText?this.$props.allText:CodeToText[item[2]]
          }
          returnArr.push(tmp)
        })
        this.$emit('changeSelectionAddress', returnArr)
      }
    }
  }
</script>

<style scoped>

</style>