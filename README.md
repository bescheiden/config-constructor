# config-constructor
根据config.json文件模板生成带新数据的json文件。（适用于电商）

# config更新器分享
标签： json vue ElementUI
## 背景及需求
### 1. 背景
公司商城每次活动页都需要运营同事手动配置页面所有的config.json文件，大型活动涉及几千个商品，每个商品又有多个属性值，运营同事需要从后台管理系统查找每个商品，并把商品属性替换到前端同事根据当次活动页面结构配好的config.json数据结构中。其中，很容易发生人为错误，例如：将原config文件修改成错误的json格式、商品属性不对应等等。导致项目无法运行，增加工作量。
### 2. 需求
产品给的需求图如下：
![file-list](https://raw.githubusercontent.com/bescheiden/Markdown-Photos/master/photos/active_config_create.png)
1. 可选择公司系统；
 2. 选择前端同事配好的config.json文件，将数据结构模板读入config更新器；
 3. config更新器将自动识别需要替换的商品列表list，并在页面渲染出来；
 4. 运营同事在指定模块写入对应商品id，并用**英文**“,”隔开；
 5. 写好相应商品id后，点击“生成（新）”，将生成带有商品所有属性的新config.json文件；
 6. 点击“下载（新）”，将新生成的config.json文件下载到本地；
## 具体实现及原理
### 1. 读取本地config文件模板

```javaScript
// 读取本地config文件模板
var file = document.getElementById("myfile");

var self = this;
file.onchange = function(event) {
  var f = this.files[0];
  var reader = new FileReader();

  reader.onload = function(event) {
    self.config = this.result;
  };
  reader.readAsText(f);
};
```

### 2. 查找 config.json 中的所有需要替换的 list 名字
 RegExp rules：要求需要替换的模块list命名为：**"list_xxxx": []** （以 *"list_* + *匹配不是双引号的任意字符重复0到多次* + *"* + *匹配0个或多个空格* + *:* + *匹配0个或多个空格* + *[* + *匹配0个或多个空格* + *]*）
 即：每个需要替换的模块置空，**"list_模块名字": []**
![file-list](https://raw.githubusercontent.com/bescheiden/Markdown-Photos/master/photos/json_file_old.png)

```javaScript
//用正则表达式匹配每个模块list的名字
var listExp = /"list_([^"]*)"\s*:\s*\[\s*\]/g;
var matchLists = [];
var singleList;
while ((singleList = listExp.exec(self.config)) !== null) {
  matchLists.push(singleList);
}

self.lists = matchLists;

self.generateListEditors();
```

### 3. 生成list编辑框

```javaScript
// 用每个模块list的名字生成list编辑框
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
                    <textarea rows="8" cols="120" placeholder="填写商品id,用英语逗号“,”隔开" id="${name}">
                    </textarea>
                </div>
             </div>`;
    return html;
  }, "");
  listsDom.innerHTML = html;
},
```

效果如下：
 ![file-list](https://raw.githubusercontent.com/bescheiden/Markdown-Photos/master/photos/list_boxs.png)
### 4. 向服务器请求商品列表

```javaScript
// 接口获取商品详情
getGoodsList(ids, element) {
  var self = this;
  $.ajax({
    url: "http://api.xxxx.inner:8011/goods/goods_info_by_ids.do",
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
      self.replaceList(element, arrayResult);
    },
    fail: function(status) {
      console.log("status>>", status);
    }
  });
},
```

![file-list](https://raw.githubusercontent.com/bescheiden/Markdown-Photos/master/photos/send_lists.png)
### 5. 用服务器返回的商品列表替换原有的空列表

```javaScript
replaceList(listname, goodlists) {
  function replacer(name, goods) {
    return '"list_' + name + '": ' + JSON.stringify(goods);
  }
  var replaceExp = new RegExp(`"list_${listname}"\\s*:\\s*\\[\\s*\\]`);
  this.config = this.config.replace(replaceExp,replacer(listname, goodlists));
},
```

![file-list](https://raw.githubusercontent.com/bescheiden/Markdown-Photos/master/photos/get_goodslist.png)
### 6. 创建新的Config文件

```javaScript
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
```

### 7. 下载新的config.json文件

```javaScript
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
```

 ![file-list](https://raw.githubusercontent.com/bescheiden/Markdown-Photos/master/photos/download.png)
 
## 分享感想
>通过与他人代码分享，发现了自己的许多不足，比如很多没有注意到的细节问题、技术缺陷等等。由于自己项目实践经验匮乏，所以在考虑问题上还不够全面、透彻，通过分享会，大家会详细指出代码所存在的问题及需要改进的地方，有助于个人编码能力的提升。同时，在实现某些功能上，各位同事和前辈提出了新的思路，让我的编码思维得到了拓展，也认识到自己在前端开发学习和前进的方向。

#### [源码见github](https://github.com/bescheiden)
