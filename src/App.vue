<template>
  <div id="app">
    <div class="container">
      <h1>config更新器</h1>
      <div class="head">
        <el-select class="choose-company" v-model="company" placeholder="请选择公司">
          <el-option v-for="item in companys" :key="item.value" :label="item.label" :value="item.value">
            <span class="company-name" style="float: left">{{ item.value }}</span>
          </el-option>
        </el-select>
        <input id="myfile" type="file" />
        <el-button class="upload-file" size="small" type="success">上传</el-button>
      </div>
      <!-- 商品列表编辑框 -->
      <div ref="lists" class="lists">
        <div class="list1">
          <h6>#list1name</h6>
          <div class="text-area">
            <el-input type="textarea" :rows="4" placeholder="填写商品id,用逗号“,”隔开">
            </el-input>
          </div>
        </div>
      </div>
      <div class="btn">
        <el-button type="success" @click="creatConfig">生成（新）</el-button>
        <el-button type="success" @click="save">下载（新）</el-button>
      </div>
    </div>
    <div class="footer">
      <div>1、点击【生成（新）】即按上传的config文件生成一个带新数据的config。</div>
      <div>1、点击【下载（新）】即下载生成的最新数据的config。</div>
    </div>
  </div>
</template>

<script>

export default {
  name: "jsonEditor",
  data() {
    return {
      companys: [
        {
          value: "7mall"
        },
        {
          value: "Dbuy"
        }
      ],
      company: "",
      fileList: [],
      lists: [],
      config: ""
    };
  },

  methods: {
    // 生成list编辑框
    generateListEditors() {
      var listsDom = this.$refs.lists;

      var listNames = this.lists.map(function(list) {
        return list[1];
      });

      var html = "";

      var html = listNames.reduce(function(html, name) {
        html += `<div class="list1">
                   <h6>${name}</h6>
                   <div class="text-area">
                     <textarea rows="8" cols="120" placeholder="填写商品id,用英语逗号“,”隔开" id="${name}"></textarea>
                   </div>
                 </div>`;
        return html;
      }, "");
      listsDom.innerHTML = html;
    },
    // 接口获取商品详情
    getGoodsList(ids, element) {
      console.log("ids", ids);
      var self = this;
      $.ajax({
        url: "http://api.inner:8010/goods/goods_info_by_ids.do",
        type: "GET",
        data: {
          goodsIds: ids
        },
        dataType: "json",
        async: false,
        success: function(response) {
          var goodsList = response.data.goodsList;
          var idsArray = ids.split(",");
          //按照传参ids顺序排列服务器返回的商品列表goodsList
          var arrayResult = idsArray.map(function(id) {
            return goodsList[id];
          });
          console.log("arrayResult>>", arrayResult);
          self.replaceList(element, arrayResult);
        },
        fail: function(status) {
          console.log("status>>", status);
        }
      });
    },
    // 替换商品list
    replaceList(listname, goodlists) {
      function replacer(name, goods) {
        return '"list_' + name + '": ' + JSON.stringify(goods);
      }
      var replaceExp = new RegExp(`"list_${listname}"\\s*:\\s*\\[\\s*\\]`);
      this.config = this.config.replace(
        replaceExp,
        replacer(listname, goodlists)
      );
      console.log("this.config>>", this.config);
    },
    // 创建新的Config文件
    creatConfig() {
      var listNames = this.lists.map(function(list) {
        return list[1];
      });
      for (var i in listNames) {
        var element = listNames[i];
        var vals = document.getElementById(element).value;
        this.getGoodsList(vals, element);
      }
    },
    // 下载新的Config文件
    save() {
      var saveData = (function() {
        var a = document.createElement("a");
        document.body.appendChild(a);
        a.style = "display: none";
        return function(data, fileName) {
          var blob = new Blob([data], {type: "octet/stream"});
          var url = window.URL.createObjectURL(blob);
          a.href = url;
          a.download = fileName;
          a.click();
          window.URL.revokeObjectURL(url);
        };
      })();
      saveData(this.config, "config.json");
    }
  },
  mounted() {
    // 读取本地config文件模板
    var file = document.getElementById("myfile");

    var self = this;
    file.onchange = function(event) {
      var f = this.files[0];
      var reader = new FileReader();

      reader.onload = function(event) {
        self.config = this.result;

        var listExp = /"list_([^"]*)"\s*:\s*\[\s*\]/g;
        var matchLists = [];
        var singleList;
        while ((singleList = listExp.exec(self.config)) !== null) {
          matchLists.push(singleList);
        }

        self.lists = matchLists;

        self.generateListEditors();
      };
      reader.readAsText(f);
    };
  }
};
</script>

<style>
#app {
  font-family: Helvetica, sans-serif;
  /* text-align: center; */
}
h1 {
  text-align: center;
}
.container {
  margin: 20px auto;
  padding: 40px;
  overflow: auto;
  width: 900px;
  height: 750px;
  border: 1px solid #b3c0d1;
  border-radius: 6px;
  box-shadow: 3px 3px 3px #b3c0d1;
}
.choose-company {
  margin-right: 15px;
}
.upload-file {
  margin-left: 250px;
  width: 90px;
}
h6 {
  margin: 15px 0;
}
.btn {
  margin-top: 100px;
  text-align: center;
}
.el-button + .el-button {
  margin-left: 40px;
}
.footer {
  margin: 0 auto;
  padding: 10px 40px 20px;
  width: 900px;
}
</style>
