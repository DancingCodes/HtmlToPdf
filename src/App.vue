<template>
  <div class="viewHtml" ref="viewHtml">
    <div ref="items" v-for="item in 100" class="items">{{ item }}</div>
  </div>
  <div @click="save" class="saveBotton">保存</div>
</template>

<script setup>
import { ref, reactive } from "vue";
import html2Canvas from 'html2canvas'
import jsPDF from 'jspdf'

// 待转换Html的Ref
const viewHtml = ref()
const items = ref()

const pdf = new jsPDF('p', 'pt', 'a4', true)
// a4的规格
const a4Specs = reactive({
  width: 595.28,
  height: 841.89
})

const save = async () => {
  // 设置pdf
  await setPdf(getPaging())
  // 下载pdf文件
  pdf.save('demo.pdf')
}

// 设置pdf
const setPdf = async (pagingList) => {
  let canvasY = 0
  for (let i = 0; i < pagingList.length; i++) {
    // 添加水印
    addWatermark()
    // 将Html转为Canvas
    let canvasEle = await html2Canvas(viewHtml.value, {
      useCORS: true,
      backgroundColor: null,
      height: pagingList[i].htmlHeight,
      y: canvasY
    })
    canvasY += pagingList[i].htmlHeight

    // 将canvas转换为图片并添加到pdf中
    pdf.addImage(canvasEle.toDataURL('image/png'), 'PNG', 0, 0, a4Specs.width, pagingList[i].a4Height)
    if (i + 1 < pagingList.length) {
      pdf.addPage()
    }
  }
}

import shuiYin from '@/assets/pdfImg/shuiYin.png'
// 添加水印
const addWatermark = () => {
  pdf.addImage(shuiYin, 'PNG', 0, 0, 595.28, 841.89)
}

// 获取分页数据
const getPaging = () => {
  // 记录分页数据
  const paging = []
  // 计算html的高为多少可以盛满一页a4 (a4宽 / a4高 = Html宽 / Html高  ===> Html高 = Html宽 / (a4宽 / a4高))
  const htmlHeightToA4 = viewHtml.value.offsetWidth / (a4Specs.width / a4Specs.height)
  // 根据原子元素灵活分页
  let count = 0
  for (let i = 0; i < items.value.length; i++) {
    // 处理临界值
    if (count + items.value[i].offsetHeight > htmlHeightToA4) {
      paging.push({
        htmlHeight: count,
        a4Height: getA4HeightToHtmlHeight(count),
      })
      count = 0
    }
    count += items.value[i].offsetHeight
  }
  paging.push({
    htmlHeight: count,
    a4Height: getA4HeightToHtmlHeight(count),
  })

  return paging
}

// 根据html的高获取在a4中的高
const getA4HeightToHtmlHeight = (htmlHeight) => {
  // (a4宽 / a4高 = Html宽 / Html高  ===> a4高 = a4宽 / (Html宽 / Html高))
  return a4Specs.width / (viewHtml.value.offsetWidth / htmlHeight)
}

</script>

<style lang="scss" scoped>
.viewHtml {

  .items {
    height: 30px;
    text-align: center;
    display: flex;
    align-items: center;
    justify-content: center;
    border: 1px solid #ccc;
    box-sizing: border-box;
  }

  .items:nth-child(even) {
    height: 50px;
  }
}

.saveBotton {
  margin: 0 auto;
  height: 30px;
  display: flex;
  align-items: center;
  justify-content: center;
  position: fixed;
  bottom: 20px;
  right: 0;
  width: 50px;
  background-color: #ccc;
  border-radius: 10px;
}
</style>