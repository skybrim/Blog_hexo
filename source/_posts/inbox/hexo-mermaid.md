---
title: Hexo 实现 Mermaid 流程图
date: 2018/11/11
tags: [hexo, inbox]
comments: true
---

hexo-filter-mermaid-diagrams,这个插件可以帮助 Hexo 实现各种 mermaid 流程图。
<!--more-->

## 安装

* 进入Blog目录

    yarn 安装

    ```bash
    yarn add hexo-filter-mermaid-diagrams
    ```

    npm 安装

    ```bash
    npm install hexo-filter-mermaid-diagrams
    ```

* Step2 Edit Config

    打开站点配置文件

    ```bash
    # mermaid chart
    mermaid: ## mermaid url https://github.com/knsv/mermaid
        enable: true  # default true
        version: "7.1.2" # default v7.1.2
        options:  # find more api options from https://github.com/knsv/mermaid/blob/master/src/mermaidAPI.js
            #startOnload: true  // default true
    ```

    **注意: 如果你想使用 'Class diagram' , 请编辑 **站点配置文件** 文件, 设置 external_link: false。hexo的bug.**

* Step3 include mermaid.js in pug or ejs

    找到主题里面的页脚文件，也即 themes/主题/layout/_partials， after_footer.pug , after-footer.ejs 或者 .swig

    after_footer.pug

    ```bash
    if theme.mermaid.enable == true
        script(type='text/javascript', id='maid-script' mermaidoptioins=theme.mermaid.options src='https://unpkg.com/mermaid@'+ theme.mermaid.version + '/dist/mermaid.min.js' + '?v=' + theme.version)
        script.
            if (window.mermaid) {
                var options = JSON.parse(document.getElementById('maid-script').getAttribute('mermaidoptioins'));
                mermaid.initialize(options);
            }
    ```

    after-footer.ejs

    ```bash
    <% if (theme.mermaid.enable) { %>
    <script src='https://unpkg.com/mermaid@<%= theme.mermaid.version %>/dist/mermaid.min.js'></script>
    <script>
    if (window.mermaid) {
        mermaid.initialize({theme: 'forest'});
        }
    </script>
    <% } %>
    ```

    .swig

    ```bash
    {% if theme.mermaid.enable %}
        <script src='https://unpkg.com/mermaid@{{ theme.mermaid.version }}/dist/mermaid.min.js'></script>
        <script>
        if (window.mermaid) {
            mermaid.initialize({{ JSON.stringify(theme.mermaid.options) }});
        }
        </script>
    {% endif %}
    ```

## mermaid 演示

### Flowchart

```bash
graph TD;
A-->B;
A-->C;
B-->D;
C-->D;
```

![1](https://cdn.jsdelivr.net/gh/skybrim/AllImages@dev/flow.png)

### Sequence diagram

```bash
sequenceDiagram
participant Alice
participant Bob
Alice->>John: Hello John, how are you?
loop Healthcheck
John->>John: Fight against hypochondria
end
Note right of John: Rational thoughts <br/>prevail...
John-->>Alice: Great!
John->>Bob: How about you?
Bob-->>John: Jolly good!
```

![2](https://cdn.jsdelivr.net/gh/skybrim/AllImages@dev/sequence.png)

### Gantt diagram

```bash
gantt
dateFormat  YYYY-MM-DD
title Adding GANTT diagram to mermaid

section A section
Completed task            :done,    des1, 2014-01-06,2014-01-08
Active task               :active,  des2, 2014-01-09, 3d
Future task               :         des3, after des2, 5d
Future task2               :         des4, after des3, 5d
```

![3](https://cdn.jsdelivr.net/gh/skybrim/AllImages@dev/gantt.png)

### Class diagram - :exclamation: experimental

```bash
classDiagram
Class01 <|-- AveryLongClass : Cool
Class03 *-- Class04
Class05 o-- Class06
Class07 .. Class08
Class09 --> C2 : Where am i?
Class09 --* C3
Class09 --|> Class07
Class07 : equals()
Class07 : Object[] elementData
Class01 : size()
Class01 : int chimp
Class01 : int gorilla
Class08 <--> C2: Cool label
```

![4](https://cdn.jsdelivr.net/gh/skybrim/AllImages@dev/class.png)

### Git graph - :exclamation: experimental

```bash
gitGraph:
options
{
"nodeSpacing": 150,
"nodeRadius": 10
}
end
commit
branch newbranch
checkout newbranch
commit
commit
checkout master
commit
commit
merge newbranch
```

![5](https://cdn.jsdelivr.net/gh/skybrim/AllImages@dev/git.png)
