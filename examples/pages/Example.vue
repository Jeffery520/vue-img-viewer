<template>
  <div class="container">
    <h3 class="mb16">第一种用法，自定义传递图片url集合，自己控制显隐</h3>
    <img
      v-for="(img, index) in imageUrls"
      :key="index"
      :src="img"
      style="width: 100px; height: 100px; margin-right: 10px;"
      @click="handleImgPreview(index)">
    <image-preview
      :image-urls="imageUrls"
      :visible.sync="visible"
      :start-position="startPosition" />

    <h3 class="mb16">第二种用法，组件包裹元素，自动识别内部图片并添加点击事件预览, 显隐由组件内部控制，此模式可传递缩放相关的Props</h3>
    <h4 class="mb16">includeSelector 和 excludeSelector过滤</h4>
    <image-preview
      max-scale="3"
      min-scale="0.6"
      scale-step="0.2"
      include-selector=".img , .s-img"
      exclude-selector=".img2 , .img3"
      :url-mapper="urlMapper">
      <img
        v-for="(img, index) in imageUrls"
        :key="index"
        :src="img"
        :class="['img', index > 1 ? `img${index}` : '']"
        style="width: 100px; height: 100px; margin-right: 10px;">
      <img
        v-for="(img, index) in imageUrls"
        :key="imageUrls.length + index"
        :src="img"
        :class="['s-img', index > 1 ? `img${index}` : '']"
        style="width: 100px; height: 100px; margin-right: 10px;">
    </image-preview>
    <h4 class="mb16">filter 函数过滤</h4>
    <image-preview
      max-scale="2"
      min-scale="0.5"
      scale-step="0.1"
      angle="45"
      :filter="filter">
      <img
        v-for="(img, index) in imageUrls"
        :key="index"
        :src="img"
        :class="['img', index > 1 ? `img${index}` : '']"
        style="width: 100px; height: 100px; margin-right: 10px;">
      <img
        v-for="(img, index) in imageUrls"
        :key="imageUrls.length + index"
        :src="img"
        :class="['s-img', index > 1 ? `img${index}` : '']"
        style="width: 100px; height: 100px; margin-right: 10px;">
    </image-preview>
    <h3 class="mb16">自定义底部操作栏和 loading</h3>
    <image-preview ref="imgPreview">
      <img
        v-for="(img, index) in imageUrls"
        :key="index"
        :src="img"
        class="img"
        style="width: 100px; height: 100px; margin-right: 10px;">
      <span slot="operate">
        <button @click="$refs.imgPreview.rotate(30)">旋转 30 度</button>
        <button @click="rotateExtraAngle">根据已有角度增加 30 度</button>
        <button @click="$refs.imgPreview.zoom(2)">放大至 2 倍</button>
        <button @click="zoom"> 在自身基础上放大 50% </button>
        <button @click="$refs.imgPreview.reset()">重置</button>
      </span>
      <span slot="loading" slot-scope="{loading}">
        <button v-if="loading">loading</button>
      </span>
    </image-preview>
  </div>
</template>
<script>

export default {
  name: 'Demo',
  data () {
    return {
      visible: false,
      visible2: false,
      startPosition: 0,
      imageUrls: [
        'https://images.unsplash.com/photo-1536420111820-d84dee5c90c5?ixlib=rb-0.3.5&ixid=eyJhcHBfaWQiOjEyMDd9&s=d76602c3cafa0599d42cfdf255c5eb8d&auto=format&fit=crop&w=700&q=80',
        'https://images.unsplash.com/photo-1536484049453-85de4ea3db6a?ixlib=rb-0.3.5&ixid=eyJhcHBfaWQiOjEyMDd9&s=902f2f3c5fbf8d85a2643ae073f39d39&auto=format&fit=crop&w=1222&q=80',
        'https://images.unsplash.com/photo-1536420095395-a592ce76a37e?ixlib=rb-0.3.5&ixid=eyJhcHBfaWQiOjEyMDd9&s=986b742530a59130ea65a65ea461653d&auto=format&fit=crop&w=700&q=80',
        'https://images.unsplash.com/photo-1536420124982-bd9d18fc47ed?ixlib=rb-0.3.5&ixid=eyJhcHBfaWQiOjEyMDd9&s=2d98a0cbfe7514bbe11cbd95ba2554f7&auto=format&fit=crop&w=701&q=80',
        'https://wimg.ruan8.com/uploadimg/image/20190402/20190402163328_31815.jpg'
      ],
      exampleSetting: {
        props: [
          {
            name: 'visible.sync',
            desc: '控制显隐。常规模式可用，插槽模式无效',
            type: 'Boolean',
            default: false
          },
          {
            name: 'imageUrls',
            desc: 'url集合。常规模式可用，插槽模式无效',
            type: 'Array',
            default: []
          },
          {
            name: 'maxScale',
            desc: '最大缩放比例, 5 代表 5倍、500%, 不得小于1',
            type: 'Number',
            default: 5
          },
          {
            name: 'minScale',
            desc: '最小缩放比例, 0.1 代表 0.1倍、10%, 不得大于1',
            type: 'Number',
            default: 0.1
          },
          {
            name: 'scaleStep',
            desc: '单次缩放的比例（放大、缩小按钮）, 0.1 代表 每次变化0.1倍，变化10%,',
            type: 'Number',
            default: 0.1
          },
          {
            name: 'angle',
            desc: '每次旋转的角度，默认90度',
            type: 'Number',
            default: 90
          },
          {
            name: 'includeSelector',
            desc: 'css选择器筛选指定图片，插槽模式下有效。<br> eg: include-selector = ".my-img" 实际筛选则为 img.my-img ',
            type: 'String',
            default: ''
          },
          {
            name: 'excludeSelector',
            desc: 'css 选择器过滤指定图片，插槽模式下有效。<br> eg: exclude-selector = ".other-img" 实际筛选则为 img:not(.ohter-img)',
            type: 'String',
            default: ''
          },
          {
            name: 'filter',
            desc: '同 Array.prototype.filter 函数，插槽模式下有效。<br> 过滤 imageList 集合，此参数存在时，includeSelector 和 excludeSelector 无效。',
            type: 'Function',
            default: '() => true'
          },
          {
            name: 'closeOnPressEscape',
            desc: '按ESC键是否关闭弹窗',
            type: 'Boolean',
            default: true
          }
        ]
      }
    }
  },
  methods: {
    handleImgPreview (index) {
      this.visible = true
      this.startPosition = index
    },
    filter (img, index) {
      return index > 2 && index < 5
    },
    rotateExtraAngle () {
      // angle 为旋转前的角度
      return this.$refs.imgPreview.rotate(angle => angle + 30)
    },
    zoom () {
      // scale 为缩放前的比例
      return this.$refs.imgPreview.zoom(scale => scale * 1.5)
    },
    urlMapper (url, index) {
      console.log(url, index)
      return 'https://images.unsplash.com/photo-1536420124982-bd9d18fc47ed?ixlib=rb-0.3.5&ixid=eyJhcHBfaWQiOjEyMDd9&s=2d98a0cbfe7514bbe11cbd95ba2554f7&auto=format&fit=crop&w=701&q=80'
    }
  }
}
</script>

<style scoped>
.container {
  height: 110vh;
}

.mb16 {
  margin-top: 16px;
}

.dib {
  display: inline-block;
}

img {
  border: 1px solid #ececec;
}

</style>
