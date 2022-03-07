## 运行服务器相关
运行django服务器的时候，突然远程连接断掉了，django服务器突然关闭不了，下面将展示如何关闭

- **重新启动服务器**显示端口已被占用

`python3 manage.py runserver 0.0.0.0:8000`

- 现在我们要**查找是哪个进程占用了8000端口**，然后kill掉

`netstat -atunp`//查看进程端口号及运行的程序

- **根据pid杀死**指定进程：

`kill 123456#pid号`

---

## id:socket访问工作流程



---

## ~/ACAPP/game项目文件
### 1.static

- `**static**`**文件夹下共有**`**css, js, image, audio**`四个文件夹。

直接向用户返回文件，文件内容直接是网页浏览到的内容（需要先将URL`/static`与本地`/static`路径join在一起）

- `**css文件管理**`：一般一个工程只有一个css文件夹就足够了。
- `**js文件管理**`：一个工程会有多个`.js`文件，为了避免每次写新的`.js`文件时import每个`html`文件，考虑将所有`.js`文件存放在`src`文件夹下，并将其所有`.js`源文件通过创建一个脚本整合到`dist`文件夹中。

`~/ACAPP/scripts$ ./compress_game_js.sh`
```cpp
#! /bin/bash

JS_PATH=/home/acs/ACAPP/game/static/js/
JS_PATH_DIST=${JS_PATH}dist/
JS_PATH_SRC=${JS_PATH}src/

find $JS_PATH_SRC -type f -name '*.js' | sort | xargs cat > ${JS_PATH_DIST}game.js
```

- `**html文件管理**`

在`templates`文件夹下创建的`multiends`文件夹，用来存放不同终端下的`html`文件，并返回给`view`
### 2.views
存放各种函数，等待用户访问。

- 在`views`文件夹下创建了三个模块的文件夹：`menu, playground, settings`，为了能够被`import`，我们在这三个文件夹下创建`__init__.py`文件
#### (`__init__.py`:

- `__init__.py `文件的作用是将文件夹变为一个Python模块, Python 中的每个模块的包中，都有`__init__.py `文件
- 我们在**导入一个包时，实际上是导入了它的**`**__init__.py文件**`。这样我们可以在`__init__.py`文件中批量导入我们所需要的模块，而不再需要一个一个的导入。

[__init__.py详解](https://www.cnblogs.com/Lands-ljk/p/5880483.html))
​


- 为了在返回`html`文件到用户浏览器，我们需要创建一个`index.py`文件，返回`templates`中`multiends`文件夹下的`html`文件。



### 3.urls
服务器端跟据`url`格式，来选择调用什么函数。

- **实现路由规则**
```cpp
                                     /-- "" -- index
                                    / -- "menu/" -- menu.index
             / "" --> "game.urls" --> 
            /                       \ -- "playground/" -- playround.index
id:scoket ->                         \-- "settings/" -- settings.index
            \
             \ "/admin" -- 到达管理员页面
```

- 修改**路由起点**文件：`~/ACAPP/app/urls.py`
```cpp
from django.contrib import admin                                                 
from django.urls import path, include

urlpatterns =[
     path('', include('game.urls.index')),
     path('admin/', admin.site.urls),
]
```

- 再找到我们的**第二分支**`~/ACAPP/game/urls/index.py`
```cpp
from django.urls import path, include
from game.views.index import index
   
   urlpatterns = [
       path("", index, name = "index"),
       path("menu/", include("game.urls.menu.index")),
       path("playground/", include("game.urls.playground.index")),
       path("settings/", include("game.urls.settings.index")),
   ]  
```
至此，我们修改完了我们期望的**路由规则**
### 4.medels
存放项目数据结构，如数据库里的`table`，可以理解为各种`class`
### 5.__pycache__文件夹
[__pycache__文件夹详解](https://blog.csdn.net/yitiaodashu/article/details/79023987)

---

## 创建菜单`menu`界面
### 1.搭建菜单`menu`框架

- 在`~/ACAPP/game/templates/multiends/web.html`里创建一个带有**id**的**div**。给名字（id）的目的是我们以后可用`js`来控制它， 比如说移动它或改变它的一些性质等等。
```cpp
<body style="margin: 0">
      <div id="ac_game_wzc"></div>                                                 
      <script type="module">
          $(document).ready(function(){
              let ac_game = new AcGame("ac_game_wzc");
          });
      console.log("hello world");
      </script>
 </body>
```

- 在`~/ACAPP/game/static/js/src/z_base.js`文件中，`AcGame` 开始执行构造函数，并捕获到`html`的标签**id**，然后利用 `AcGameMenu`类创建对象 `menu`，并将整个对象作为参数下传
```cpp
class AcGame {
     constructor(id) {
         this.id = id;
         this.$ac_game = $('#' + id);
         this.menu = new AcGameMenu(this);//实现菜单界面
         this.playground = new AcGamePlayground(this);//实现游戏界面
         this.start();
     }
    start {
    }
}
```

- 在`~/ACAPP/game/static/js/src/menu && playground/z_base.js`文件中，分别调用了其构造函数，将其中的`$menu`**html**代码加到`$ac_game`即捕获到的**html代码**之后
- 设置菜单`menu`的内容，显示单人模式，多人模式，设置三个按钮（此处仅展示单人按钮代码）
```cpp
class AcGameMenu {
     constructor(root) {
         this.root = root;
         this.$menu = 
$(`
 <div class="ac_game_menu">
     <div class = "ac_game_menu_field">
         <div class = "ac_game_menu_field_item ac_game_menu_field_item_single_mode">
             单人模式
         </div>
     </div> 
  </div>                                                                           
`);
         this.root.$ac_game.append(this.$menu);
         this.$single_mode = this.$menu.find('.ac_game_menu_field_item_single_mode');
```
### 2.添加单人模式监听函数，实现打开游戏界面功能

- 点击单人模式按钮，触发`click`事件，触发执行监听函数
- 关闭`menu`界面，打开`playground`界面

因此，我们先创建一个`payground`界面：
```cpp
class AcGamePlayground {
    constructor(root) {
        this.root = root;
        this.$playground = $(`<div>游戏界面</div>`);

        this.hide();
        this.root.$ac_game.append(this.$playground);

        this.start();
    }
    
    start() {

    }
    show() {    //打开 playground 界面
        this.$playground.show();
    }
    hide() {    //关闭 playground 界面
        this.$playground.hide();
    }

}
```
并在`~/js/src/zbase.js`主`js`文件下调用`AcGamePlayground`类创建`Playground`对象
```cpp
this.playground = new AcGamePlayground(this);//实现游戏界面
```
最后实现关闭`menu`界面，打开`playground`界面
`~/ACAPP/game/static/js/src/menu$`
```cpp
        add_listening_events() {
                     let outer = this;                                                                                                                                    
                     this.$single_mode.click(function(){
                         outer.hide();
                         outer.root.playground.show();
                     });
                     this.$multi_mode.click(function(){
                     });
                     this.$settings.click(function(){
                     });
                 }
```
结束。

---

### Django路由

* 我们在前端访问网页的路径是由**`/urls`**提供的，我们要知道，一个文件有许多不同的模块，比如`settings`，`menu`， `playground` 等。
* 因此`django`提供了层级路由，上层文件下的`index.py`可以调用下层目录的`index.py` 中的`path`和`include`。十分方便



