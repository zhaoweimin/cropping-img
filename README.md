## 组件介绍
vue 图片裁剪功能

## 安装
```
npm install cropping-im -s
```

## vue  调用
```
<template>
  <div class="hello"> 
      <cropping-img @onCropping="onCropping"/>
  </div>
</template>

<script>
import croppingImg from 'cropping-img'
export default {
  components: { croppingImg },
  methods:{
    onCropping(res) {
      console.log(res)
    }
  }
}
</script>
```


## 入参 props
| 参数      | 类型    |  默认  | 描述  |
| -------- | -----:   :----: || :----: |
| width | string | 100% | 设置显示图片区域样式的width |
| height | string | auto | 设置显示图片区域样式的height |
| hideBtn | boolea | false | 是否显示按钮 |
| btns |Object| ["choose", "cropping", "export"] | choosex选择图片, cropping预览, export导出 |
| btns | | | |

## 方法 action
| 方法名   | 作用  | 返回值 |
| -------- | -----: | :----: |
| onCropping | 生成截图 | 返回裁剪图的 base64 和 file |

## refs
| 方法名   | 参数  | 描述 |
| -------- | -----: | :----: |
| setImage | 图片路径(string/file/base64String) | 设置裁剪图片 |

## github
https://github.com/zhaoweimin/cropping-img