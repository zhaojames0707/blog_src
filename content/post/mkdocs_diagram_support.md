+++
author = "zhaojames0707"
comments = true
date = "2016-06-21T18:18:00+08:00"
draft = false
image = ""
share = true
tags = ["python", "mkdocs"]
title = "在 MkDocs 中使用时序图及流程图"
discription = "foo"

+++

MkDocs 是非常方便的文档展示工具，但是原生不支持将 markdown 文档中的 sequence 及 flow 代码转换为时序图/流程图。
通过以下简单的3步，即可在不改变 markdown 文档的情况下，将时序图和流程图绘制出来。

<!--more-->>

#### 一、安装 PyMdown Extensions:

PyMdown Extension 为 Python Markdown 提供了丰富的拓展，其中的 superfences 模块能支持 sequence/flow 图。
在终端执行：

```
pip install pymdown-extensions
```

安装完成后，在 MkDocs 项目的 mkdocs.yml 中添加：

```
markdown_extensions:
    - pymdownx.superfences
```

#### 二、引入 JavaScript 库

绘制流程图需要以下两个库：

* [raphael.js](https://bramp.github.io/js-sequence-diagrams/js/raphael-min.js)
* [flowchart.js](http://flowchart.js.org/flowchart-latest.js)

绘制时序图需要以下三个库：

* [raphael.js](https://bramp.github.io/js-sequence-diagrams/js/raphael-min.js)
* [underscore.js](https://bramp.github.io/js-sequence-diagrams/js/underscore-min.js)
* [sequence-diagram.js](https://bramp.github.io/js-sequence-diagrams/js/sequence-diagram-min.js)

引入方法很简单，添加到 mkdocs.yml 的 extra_javascript 属性即可，例子如下：

```
extra_javascript:
    - https://bramp.github.io/js-sequence-diagrams/js/raphael-min.js
    - https://bramp.github.io/js-sequence-diagrams/js/underscore-min.js
    - https://bramp.github.io/js-sequence-diagrams/js/sequence-diagram-min.js
    - http://flowchart.js.org/flowchart-latest.js
```

也可以将 js 下载到本地，放到项目文件夹的 docs/js 文件夹下，引入例子：
```
extra_javascript:
	- js/raphael-min.js
```

#### 三、引入三个 JavaScript 文件

引入 js 库以后，还需要三个 js 文件。它们的作用是定位到目标 HTML 元素，调用对应的 JS 库，最终绘制成所需要的图。

* uml-converter.js

```javascript
(function (win, doc) {
  win.convertUML = function(className, converter, settings) {
    var charts = doc.querySelectorAll("pre." + className),
        arr = [],
        i, j, maxItem, diagaram, text, curNode;

    // Is there a settings object?
    if (settings === void 0) {
        settings = {};
    }

    // Make sure we are dealing with an array
    for(i = 0, maxItem = charts.length; i < maxItem; i++) arr.push(charts[i])

    // Find the UML source element and get the text
    for (i = 0, maxItem = arr.length; i < maxItem; i++) {
        childEl = arr[i].firstChild;
        parentEl = childEl.parentNode;
        text = "";
        for (j = 0; j < childEl.childNodes.length; j++) {
            curNode = childEl.childNodes[j];
            whitespace = /^\s*$/;
            if (curNode.nodeName === "#text" && !(whitespace.test(curNode.nodeValue))) {
                text = curNode.nodeValue;
                break;
            }
        }

        // Do UML conversion and replace source
        el = doc.createElement('div');
        el.className = className;
        parentEl.parentNode.insertBefore(el, parentEl);
        parentEl.parentNode.removeChild(parentEl);
        diagram = converter.parse(text);
        diagram.drawSVG(el, settings);
    }
  }
})(window, document)
```

* flow-loader.js

```javascript
(function (doc) {
  function onReady(fn) {
    if (doc.addEventListener) {
      doc.addEventListener('DOMContentLoaded', fn);
    } else {
      doc.attachEvent('onreadystatechange', function() {
        if (doc.readyState === 'interactive')
          fn();
      });
    }
  }

  onReady(function(){convertUML('uml-flowchart', flowchart);});
})(document)
```

* sequence-loader.js

```javascript
(function (doc) {
  function onReady(fn) {
    if (doc.addEventListener) {
      doc.addEventListener('DOMContentLoaded', fn);
    } else {
      doc.attachEvent('onreadystatechange', function() {
        if (doc.readyState === 'interactive')
          fn();
      });
    }
  }

  onReady(function(){convertUML('uml-sequence-diagram', Diagram, {theme: 'simple'});});
})(document)
```

将以上三个 js 保存至项目文件夹的 docs/js 文件夹内，并在 mkdocs.yml 中引入。引入方法同步骤二。
至此已经大功告成，此后使用 mkdocs serve 则能将 markdown 文档中的：

	```flow
	xxx
	```
以及

	```sequence
	xxx
	```

自动渲染成流程图和时序图。
