<template>
  <div class="wrap">
    <div class="excel-container" ref="docxContainer"></div>
  </div>
</template>

<script>
import * as Excel from 'exceljs/dist/exceljs'
import Spreadsheet from "x-data-spreadsheet";
import tinycolor from "tinycolor2";
import { utils, read } from 'xlsx';

export default {
  name: 'excel-preview',
  props: {
    url: {
      required: true,
      type: String,
      default: '',
    },
  },
  data() {
    return {
      excelData: null,
    };
  },
  watch: {
    url: {
      immediate: true,
      handler: 'getData'
    },
  },
  mounted(){
    this.xs = new Spreadsheet(this.$refs.docxContainer,{
      mode: 'read',
      showToolbar: false
    }).loadData({});
  },
  methods: {
    getData(){
      fetch(this.url).then(res=>{
        if(res.status !== 200){
          return Promise.reject(res)
        }
        return res.arrayBuffer()
      }).then(this.renderExcel).catch(e =>{
        this.xs.loadData({})
        this.$emit('error', { type: 'excel', url: this.url, error: e });
      })
    },
    renderExcel(buffer) {
      try {
        const wb = new Excel.Workbook();
        // 微软的 Excel ColorIndex 一个索引数字对应一个颜色
        wb.xlsx.load(buffer)
          .then(workbook => {
            let workbookData = []
            workbook.eachSheet((sheet) => {
              // 构造x-data-spreadsheet 的 sheet 数据源结构
              let sheetData = {
                name: sheet.name,
                styles: [],
                rows: {},
                merges: []
              }
              // console.log(sheet);
              // 收集合并单元格信息
              let mergeAddressData = []
              for (let mergeRange in sheet._merges) {
                sheetData.merges.push(sheet._merges[mergeRange].shortRange)
                let mergeAddress = {}
                // 合并单元格起始地址
                mergeAddress.startAddress = sheet._merges[mergeRange].tl
                // 合并单元格终止地址
                mergeAddress.endAddress = sheet._merges[mergeRange].br
                // Y轴方向跨度
                mergeAddress.YRange = sheet._merges[mergeRange].model.bottom - sheet._merges[mergeRange].model.top
                // X轴方向跨度
                mergeAddress.XRange = sheet._merges[mergeRange].model.right - sheet._merges[mergeRange].model.left
                mergeAddressData.push(mergeAddress)
              }
              sheetData.cols = {}
              const columnsLength = (sheet.columns || []).length;
              for (let i = 0; i < columnsLength; i++) {
                sheetData.cols[i.toString()] = {}
                if (sheet.columns[i].width) {
                  // 不知道为什么从 exceljs 读取的宽度显示到 x-data-spreadsheet 特别小, 这里乘以8
                  sheetData.cols[i.toString()].width = sheet.columns[i].width * 8
                } else {
                  // 默认列宽
                  sheetData.cols[i.toString()].width = 100
                }
              }

              // 遍历行
              sheet.eachRow((row, rowIndex) => {
                sheetData.rows[(rowIndex - 1).toString()] = { cells: {},height:row.height || undefined, }
                //includeEmpty = false 不包含空白单元格
                row.eachCell({ includeEmpty: true }, function (cell, colNumber) {
                  let cellText = ''
                  if (cell.value && cell.value.result) {
                    // Excel 单元格有公式
                    cellText = cell.value.result
                  } else if (cell.value && cell.value.richText) {
                    // Excel 单元格是多行文本
                    for (let text in cell.value.richText) {
                      // 多行文本做累加
                      cellText += cell.value.richText[text].text
                    }
                  } else {
                    // Excel 单元格无公式
                    cellText = cell.value
                  }

                  //解析单元格,包含样式
                  //*********************单元格存在背景色******************************
                  // 单元格存在背景色
                  let backGroundColor = null
                  if (cell.style.fill && cell.style.fill.fgColor && cell.style.fill.fgColor.argb) {
                    // 8位字符颜色先转rgb再转16进制颜色
                    backGroundColor = ((val) => {
                      val = val.trim()
                        .toLowerCase();  //去掉前后空格
                      let color = {};
                      try {
                        let argb = /^#?([a-f\d]{2})([a-f\d]{2})([a-f\d]{2})([a-f\d]{2})$/i.exec(val);
                        color.r = parseInt(argb[2], 16);
                        color.g = parseInt(argb[3], 16);
                        color.b = parseInt(argb[4], 16);
                        color.a = parseInt(argb[1], 16) / 255;
                        return tinycolor(`rgba(${color.r}, ${color.g}, ${color.b}, ${color.a})`)
                          .toHexString()
                      } catch (e) {
                        //
                      }
                    })(cell.style.fill.fgColor.argb)
                  }

                  if (backGroundColor) {
                    cell.style.bgcolor = backGroundColor
                  }
                  //*************************************************************************** */

                  //*********************字体存在背景色******************************
                  // 字体颜色
                  let fontColor = null
                  if (cell.style.font && cell.style.font.color && cell.style.font.color.argb) {
                    // 8位字符颜色先转rgb再转16进制颜色
                    fontColor = ((val) => {
                      val = val.trim()
                        .toLowerCase();  //去掉前后空格
                      let color = {};
                      try {
                        let argb = /^#?([a-f\d]{2})([a-f\d]{2})([a-f\d]{2})([a-f\d]{2})$/i.exec(val)
                        color.r = parseInt(argb[2], 16);
                        color.g = parseInt(argb[3], 16);
                        color.b = parseInt(argb[4], 16);
                        color.a = parseInt(argb[1], 16) / 255;
                        return tinycolor(`rgba(${color.r}, ${color.g}, ${color.b}, ${color.a})`)
                          .toHexString()
                      } catch (e) {
                        //
                      }
                    })(cell.style.font.color.argb)
                  }
                  if (fontColor) {
                    cell.style.color = fontColor
                  }

                  // exceljs 对齐的格式转成 x-date-spreedsheet 能识别的对齐格式
                  if (cell.style.alignment && cell.style.alignment.horizontal) {
                    cell.style.align = cell.style.alignment.horizontal
                    cell.style.valign = cell.style.alignment.vertical
                  }
                  cell.style.border = {};

                  //处理合并单元格
                  let mergeAddress = mergeAddressData.find(function (o) {
                    return o.startAddress == cell._address
                  })
                  if (mergeAddress) {
                    // 遍历的单元格属于合并单元格
                    if (cell.master.address != mergeAddress.startAddress) {
                      // 不是合并单元格中的第一个单元格不需要计入数据源
                      return
                    }
                    // 说明是合并单元格区域的起始单元格
                    sheetData.rows[(rowIndex - 1).toString()].cells[(colNumber - 1).toString()] = {
                      text: cellText,
                      style: 0,
                      merge: [mergeAddress.YRange, mergeAddress.XRange]
                    }
                    sheetData.styles.push(cell.style)
                    //对应的style存放序号
                    sheetData.rows[(rowIndex - 1).toString()].cells[(colNumber - 1).toString()].style = sheetData.styles.length - 1
                  } else {
                    // 非合并单元格
                    sheetData.rows[(rowIndex - 1).toString()].cells[(colNumber - 1).toString()] = {
                      text: cellText,
                      style: 0
                    }
                    //解析单元格,包含样式
                    sheetData.styles.push(cell.style)
                    //对应的style存放序号
                    sheetData.rows[(rowIndex - 1).toString()].cells[(colNumber - 1).toString()].style = sheetData.styles.length - 1
                  }
                });
              })
              workbookData.push(sheetData)
            })
            this.xs.loadData(workbookData);
            // Excel文档渲染完成后触发loaded事件
            this.$emit('loaded', { type: 'excel', url: this.url, sheets: workbookData.length });
          })
      } catch (e) {
        this.xs.loadData({})
      }
    },
    // async render() {
    //   const f = await fetch(this.url);
    //   const ab = await f.arrayBuffer();
    //   const wb = read(ab);
    //   this.excelData = utils.sheet_to_html(wb.Sheets[wb.SheetNames[0]]);
    // },
  },
};
</script>

<style scoped>
  @import "../styles/excel.css";
  .excel-container /deep/td {
    min-width: 50px;
    height: 30px;
    outline: 0.02667rem solid #999;
  }
</style>
