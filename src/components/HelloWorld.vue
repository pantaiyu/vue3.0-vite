<template>
  <h1>{{ msg }}</h1>
  <!-- <el-table :data="tableData" style="width: 100%">
      <el-table-column prop="date" label="日期" width="180"></el-table-column>
      <el-table-column prop="'开票名称'" label="姓名" width="180"></el-table-column>
      <el-table-column prop="address" label="地址"></el-table-column>
  </el-table>-->
  <hr />
  <el-upload
    class="upload"
    action
    :multiple="false"
    :show-file-list="false"
    :file-list="fileList"
    accept="csv, application/vnd.ms-excel, application/vnd.openxmlformats-officedocument.spreadsheetml.sheet"
    :http-request="httpRequest"
  >
    <el-button size="small" type="primary" :loading="upLoading">上传</el-button>
  </el-upload>
  <Composition></Composition>
</template>

<script>
import { reactive, computed, onMounted, onUnmounted, ref, toRefs } from "vue";
import Composition from "./Composition.vue";
import XLSX from "xlsx";
export default {
  name: "HelloWorld",
  props: {
    msg: String
  },
  data() {
    return {
      limitUpload: 1,
      fileTemp: null,
      fileListUpload: [],
      tableData: [],
      fileList: [],
      upLoading: false
    };
  },
  components: {
    Composition
  },
  methods: {
    httpRequest(e) {
      let file = e.file; // 文件信息
      console.log(e.file);
      if (!file) {
        // 没有文件
        return false;
      } else if (!/\.(xls|xlsx)$/.test(file.name.toLowerCase())) {
        // 格式根据自己需求定义
        this.$message.error("上传格式不正确，请上传xls或者xlsx格式");
        return false;
      }
      this.fileList = [
        {
          name: file.name,
          url: file.uid
        }
      ];
      this.upLoading = true;
      const fileReader = new FileReader();
      let this_ = this;
      fileReader.onload = ev => {
        try {
          const data = ev.target.result;
          const workbook = XLSX.read(data, {
            type: "binary" // 以字符编码的方式解析
          });
          console.log(workbook);
          const exlname = workbook.SheetNames[0]; // 取第一张表
          console.log(XLSX);
          let exl = XLSX.utils.sheet_to_json(workbook.Sheets[exlname]); // 生成json表格内容

          // console.log(exl);
          this.tableData = exl;
          this.upLoading = false;

          this.$alert(
            `文件读取成功！总计${exl.length}条数据,点击确定为您转化数据`,
            "提示",
            {
              confirmButtonText: "确定",
              callback: action => {
                // this.$message({
                //   type: 'info',
                //   message: `action: ${ action }`
                // });
                // let data=exl.map((item)=>{
                //   return{
                //     customerName:item['客户名称'],
                //     salesmanName:item['业务员姓名'],
                //   }
                // })
                this.initUpData(exl);
              }
            }
          );
        } catch (e) {
          console.log("出错了：：");
          return false;
        }
      };
      fileReader.readAsBinaryString(file);
    },
    initUpData(data) {
      let newdata = {};
      console.log(data);
      data.map((item, index) => {
         item.wareTime = this.getNextDate("1900-01-01", item['出库时间'] - 2);
        if (newdata[item["客户名称"] + item["业务员姓名"]]) {
          newdata[item["客户名称"] + item["业务员姓名"]].push(item);
        } else {
          newdata[item["客户名称"] + item["业务员姓名"]] = [item];
        }
      });
      console.log(newdata);
    },
    getExcel(res) {
      require.ensure([], () => {
        const {
          export_json_to_excel
        } = require("../../public/excel/Export2Excel.js");
        const tHeader = ["商品编号", "商品名称", "单位", "数量", "价格"];
        const filterVal = [
          "skuNo",
          "skuName",
          "unit",
          "itemQty",
          "currentPrice"
        ];
        const list = res;
        const data = this.formatJson(filterVal, list);
        export_json_to_excel(tHeader, data, "商品明细");
      });
    },
    formatJson(filterVal, jsonData) {
      return jsonData.map(v => filterVal.map(j => v[j]));
    },
    getNextDate(date,day) {
      var dd = new Date(date);
      dd.setDate(dd.getDate() + day);
      var y = dd.getFullYear();
      var m =
        dd.getMonth() + 1 < 10 ? "0" + (dd.getMonth() + 1) : dd.getMonth() + 1;
      var d = dd.getDate() < 10 ? "0" + dd.getDate() : dd.getDate();
      return y + "-" + m + "-" + d;
    },
  }
};
</script>
