<script src='https://code.jquery.com/jquery-3.4.1.js'></script>
<template>
  <div class="myApp" v-loading.fullscreen.lock="fullscreenLoading" element-loading-text="拼命加载中...">
    <input type="file" @change="importFile(this)" id="imFile" style="display: none"
           accept="application/vnd.openxmlformats-officedocument.spreadsheetml.sheet, application/vnd.ms-excel"/>
    <a id="downlink"></a>
    <button class="button" @click="uploadFile()">导入</button>
    <br><br>
    <br>
    <button class="button" @click="downloadFile(excelData)">导出</button>
    <br>
    <br>
     <br>
  </div>
</template>
<script src='../assets/js/xlsx.core.min.js' type='javaScript/text'></script>
<script src='../assets/js/xlsx.full.min.js' type='javaScript/text'></script>
<script>

  // 引入xlsx  
  var XLSX = require('xlsx')
  console.log(XLSX)
  export default {
    name: 'HelloWorld',
    data () {
      return {
        fullscreenLoading: false, // 加载中
        imFile: '', // 导入文件el
        outFile: '',  // 导出文件el
        errorDialog: false, // 错误信息弹窗
        errorMsg: '', // 错误信息内容
        textJson:'颜色分类：典雅黑【大气简约  兼容运动商务场景】+收藏加购送运动毛巾',
        textJson1:'颜色分类：专业级摇杯 典雅黑【大气简约  兼容运动商务场景】',
        excelData: []
      }
    },
    mounted () {
      this.imFile = document.getElementById('imFile')
      this.outFile = document.getElementById('downlink')
    },
    methods: {
      uploadFile: function () { // 点击导入按钮
        this.imFile.click()
      },
      downloadFile: function (rs) { // 点击导出按钮
        let data = [{}]
        for (let k in rs[0]) {
          data[0][k] = k
        }
        data = data.concat(rs)
        this.downloadExl(data, '商品清单列表')
      },
      importFile: function () { // 导入excel
        this.fullscreenLoading = true
        let obj = this.imFile
        if (!obj.files) {
          this.fullscreenLoading = false
          return
        }
        var f = obj.files[0]
        var reader = new FileReader()
        let $t = this
        reader.onload = function (e) {
          var data = e.target.result
          if (this.rABS) {
            $t.wb = XLSX.read(btoa(this.fixdata(data)), {  // 手动转化
              type: 'base64'
            })
          } else {
            console.log(XLSX)
            $t.wb = XLSX.read(data, {
              type: 'binary'
            })
          }
          let json = XLSX.utils.sheet_to_json($t.wb.Sheets[$t.wb.SheetNames[0]])
          console.log($t.analyzeData(json))
          $t.dealFile($t.analyzeData(json)) // analyzeData: 解析导入数据
        }
        if (this.rABS) {
          reader.readAsArrayBuffer(f)
        } else {
          reader.readAsBinaryString(f)
        }
      },
      downloadExl: function (json, downName, type) {  // 导出到excel
        let keyMap = [] // 获取键
        for (let k in json[0]) {
          keyMap.push(k)
        }
        console.info('keyMap', keyMap, json)
        let tmpdata = [] // 用来保存转换好的json
        json.map((v, i) => keyMap.map((k, j) => Object.assign({}, {
          v: v[k],
          position: (j > 25 ? this.getCharCol(j) : String.fromCharCode(65 + j)) + (i + 1)
        }))).reduce((prev, next) => prev.concat(next)).forEach(function (v) {
          tmpdata[v.position] = {
            v: v.v
          }
        })
        let outputPos = Object.keys(tmpdata)  // 设置区域,比如表格从A1到D10
        let tmpWB = {
          SheetNames: ['mySheet'], // 保存的表标题
          Sheets: {
            'mySheet': Object.assign({},
              tmpdata, // 内容
              {
                '!ref': outputPos[0] + ':' + outputPos[outputPos.length - 1] // 设置填充区域
              })
          }
        }
        let tmpDown = new Blob([this.s2ab(XLSX.write(tmpWB,
          {bookType: (type === undefined ? 'xlsx' : type), bookSST: false, type: 'binary'} // 这里的数据是用来定义导出的格式类型
        ))], {
          type: ''
        })  // 创建二进制对象写入转换好的字节流
        var href = URL.createObjectURL(tmpDown)  // 创建对象超链接
        this.outFile.download = downName + '.xlsx'  // 下载名称
        this.outFile.href = href  // 绑定a标签
        this.outFile.click()  // 模拟点击实现下载
        setTimeout(function () {  // 延时释放
          URL.revokeObjectURL(tmpDown) // 用URL.revokeObjectURL()来释放这个object URL
        }, 100)
      },
      analyzeData: function (data) {  // 此处可以解析导入数据
        return data
      },
      dealFile: function (data) {   // 处理导入的数据
        console.log(data)
        let arr = this.textJson
        let arr1 = this.textJson1
        // console.log(arr,arr1)
        if(data.length>0){
          for(let i in data){
            let list=data[i].商品属性
            if(list==arr||list==arr1){
              data[i].商品属性 = '黑色漩涡杯'
              
            }            
          }
         this.excelData = data
        }
        this.imFile.value = ''
        this.fullscreenLoading = false
        if (data.length <= 0) {
          this.errorDialog = true
          this.errorMsg = '请导入正确信息'
        } else {
          this.excelData = data
        }
      },
      s2ab: function (s) { // 字符串转字符流
        var buf = new ArrayBuffer(s.length)
        var view = new Uint8Array(buf)
        for (var i = 0; i !== s.length; ++i) {
          view[i] = s.charCodeAt(i) & 0xFF
        }
        return buf
      },
      getCharCol: function (n) { // 将指定的自然数转换为26进制表示。映射关系：[0-25] -> [A-Z]。
        let s = ''
        let m = 0
        while (n > 0) {
          m = n % 26 + 1
          s = String.fromCharCode(m + 64) + s
          n = (n - m) / 26
        }
        return s
      },
      fixdata: function (data) {  // 文件流转BinaryString
        var o = ''
        var l = 0
        var w = 10240
        for (; l < data.byteLength / w; ++l) {
          o += String.fromCharCode.apply(null, new Uint8Array(data.slice(l * w, l * w + w)))
        }
        o += String.fromCharCode.apply(null, new Uint8Array(data.slice(l * w)))
        return o
      }
    }
  }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style>
  .el-table th>.cell {
    text-align: center;
  }
  .button {
    margin-bottom: 20px;
    padding:10px 20px;
    border:1px solid #666;
    font-size:24px ;
    font-weight:bold;
    border-radius:10px ;
  }
</style>