

## Maven

```groovy
./gradlew clean build bintrayUpload -PbintrayUser=BINTRAY_USERNAME -PbintrayKey=BINTRAY_KEY -PdryRun=false
```



## Gradle

```bash
chmod +x gradlew
```



## Mac OS X

- 复制文件

  ```bash
  cp -r <form-file/dir> <to-dir>
  ```

- 检测网络质量

  ```bash
  networkQuality
  ```
  
- 删除文件夹
  ```
  rm -r <dir>
  ```
  
  



## Homebrew

- 安装

  ```bash
  /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
  ```

- 卸载

  ```ruby
  $ cd `brew --prefix`
  $ rm -rf Cellar
  $ brew prune
  $ rm `git ls-files`
  $ rm -r Library/Homebrew Library/Aliases Library/Formula Library/Contributions
  $ rm -rf .git
  $ rm -rf ~/Library/Caches/Homebrew
  ```

  或

  ```bash
  /bin/bash -c "$(curl -fsSL https://cdn.jsdelivr.net/gh/ineo6/homebrew-install/uninstall.sh)"
  ```

- 安装任意包

  ```ruby
  $ brew install <packageName>
  ```

  示例：安装node

  ```ruby
  $ brew install node
  ```

- 卸载任意包

  ```ruby
  $ brew uninstall <packageName>
  ```

  示例：卸载git

  ```ruby
  $ brew uninstall git
  ```

- 查询可用包

  ```ruby
  $ brew search <packageName>
  ```

- 查看已安装包列表

  ```cpp
  $ brew list
  ```

- 查看任意包信息

  ```ruby
  $ brew info <packageName>
  ```

- 更新Homebrew

  ```ruby
  $ brew update
  ```

- 查看Homebrew版本

  ```ruby
  $ brew -v
  ```

- Homebrew帮助信息

  ```ruby
  $ brew -h
  ```



## Android

显示并控制通过USB或TCP/IP连接的Android设备

```bash
$ scrcpy
```



## VS Code

### 工作区快捷键

| Mac 快捷键           | Win 快捷键               | 作用                                          | 备注                 |
| -------------------- | ------------------------ | --------------------------------------------- | -------------------- |
| **Cmd + Shift + P**  | **Ctrl + Shift + P**，F1 | 显示命令面板                                  |                      |
| **Cmd + B**          | **Ctrl + B**             | 显示/隐藏侧边栏                               | 很实用               |
| `Cmd + \`            | `Ctrl + \`               | **创建多个编辑器**                            | 【重要】抄代码利器   |
| **Cmd + 1、2**       | **Ctrl + 1、2**          | 聚焦到第 1、第 2 个编辑器                     | 同上重要             |
| **Cmd + +、Cmd + -** | **ctrl + +、ctrl + -**   | 将工作区放大/缩小（包括代码字体、左侧导航栏） | 在投影仪场景经常用到 |
| Cmd + J              | Ctrl + J                 | 显示/隐藏控制台                               |                      |
| **Cmd + Shift + N**  | **Ctrl + Shift + N**     | 重新开一个软件的窗口                          | 很常用               |
| Cmd + Shift + W      | Ctrl + Shift + W         | 关闭软件的当前窗口                            |                      |
| Cmd + N              | Ctrl + N                 | 新建文件                                      |                      |
| Cmd + W              | Ctrl + W                 | 关闭当前文件                                  |                      |

### 跳转操作

| Mac 快捷键                    | Win 快捷键             | 作用                                 | 备注               |
| ----------------------------- | ---------------------- | ------------------------------------ | ------------------ |
| Cmd + `                       | 没有                   | 在同一个软件的**多个工作区**之间切换 | 使用很频繁         |
| **Cmd + Option + 左右方向键** | Ctrl + Pagedown/Pageup | 在已经打开的**多个文件**之间进行切换 | 非常实用           |
| Ctrl + Tab                    | Ctrl + Tab             | 在已经打开的多个文件之间进行跳转     | 不如上面的快捷键快 |
| Cmd + Shift + O               | Ctrl + shift + O       | 在当前文件的各种**方法之间**进行跳转 |                    |
| Ctrl + G                      | Ctrl + G               | 跳转到指定行                         |                    |
| `Cmd+Shift+\`                 | `Ctrl+Shift+\`         | 跳转到匹配的括号                     |                    |

### 移动光标

| Mac 快捷键              | Win 快捷键            | 作用                       | 备注       |
| ----------------------- | --------------------- | -------------------------- | ---------- |
| 方向键                  | 方向键                | 在**单个字符**之间移动光标 | 大家都知道 |
| **option + 左右方向键** | **Ctrl + 左右方向键** | 在**单词**之间移动光标     | 很常用     |
| **Cmd + 左右方向键**    | **Fn + 左右方向键**   | 在**整行**之间移动光标     | 很常用     |
| Cmd + ←                 | Fn + ←（或 Win + ←）  | 将光标定位到当前行的最左侧 | 很常用     |
| Cmd + →                 | Fn + →（或 Win + →）  | 将光标定位到当前行的最右侧 | 很常用     |
| Cmd + ↑                 | Ctrl + Home           | 将光标定位到文章的第一行   |            |
| Cmd + ↓                 | Ctrl + End            | 将光标定位到文章的最后一行 |            |
| Cmd + Shift + \         |                       | 在**代码块**之间移动光标   |            |

### 编辑操作

| Mac 快捷键             | Win 快捷键          | 作用                                 | 备注                                   |
| ---------------------- | ------------------- | ------------------------------------ | -------------------------------------- |
| Cmd + C                | Ctrl + C            | 复制                                 |                                        |
| Cmd + X                | Ctrl + X            | 剪切                                 |                                        |
| Cmd + C                | Ctrl + V            | 粘贴                                 |                                        |
| **Cmd + Enter**        | **Ctrl + Enter**    | 在当前行的下方新增一行，然后跳至该行 | 即使光标不在行尾，也能快速向下插入一行 |
| Cmd+Shift+Enter        | Ctrl+Shift+Enter    | 在当前行的上方新增一行，然后跳至该行 | 即使光标不在行尾，也能快速向上插入一行 |
| **Option + ↑**         | **Alt + ↑**         | 将代码向上移动                       | 很常用                                 |
| **Option + ↓**         | **Alt + ↓**         | 将代码向下移动                       | 很常用                                 |
| Option + Shift + ↑     | Alt + Shift + ↑     | 将代码向上复制一行                   |                                        |
| **Option + Shift + ↓** | **Alt + Shift + ↓** | 将代码向下复制一行                   | 写重复代码的利器                       |

### 多光标编辑

| Mac 快捷键                    | Win 快捷键                 | 作用                                 | 备注 |
| ----------------------------- | -------------------------- | ------------------------------------ | ---- |
| **Cmd + Option + 上下键**     | **Ctrl + Alt + 上下键**    | 在连续的多列上，同时出现光标         |      |
| **Option + 鼠标点击任意位置** | **Alt + 鼠标点击任意位置** | 在任意位置，同时出现光标             |      |
| Option + Shift + 鼠标拖动     | Alt + Shift + 鼠标拖动     | 在选中区域的每一行末尾，出现光标     |      |
| Cmd + Shift + L               | Ctrl + Shift + L           | 在选中文本的所有相同内容处，出现光标 |      |

### 删除操作

| Mac 快捷键             | Win 快捷键           | 作用                   | 备注                                      |
| ---------------------- | -------------------- | ---------------------- | ----------------------------------------- |
| Cmd + shift + K        | Ctrl + Shift + K     | 删除整行               | 「Cmd + X」的作用是剪切，但也可以删除整行 |
| **option + Backspace** | **Ctrl + Backspace** | 删除光标之前的一个单词 | 英文有效，很常用                          |
| option + delete        | Ctrl + delete        | 删除光标之后的一个单词 |                                           |
| **Cmd + Backspace**    |                      | 删除光标之前的整行内容 | 很常用                                    |
| Cmd + delete           |                      | 删除光标之后的整行内容 |                                           |

### 编程语言相关

| Mac 快捷键             | Win 快捷键      | 作用                         | 备注                             |
| ---------------------- | --------------- | ---------------------------- | -------------------------------- |
| Cmd + /                | Ctrl + /        | 添加单行注释                 | 很常用                           |
| **Option + Shift + F** | Alt + shift + F | 代码格式化                   | 很常用                           |
| F2                     | F2              | 以重构的方式进行**重命名**   | 改代码备                         |
| Ctrl + J               |                 | 将多行代码合并为一行         | Win 用户可在命令面板搜索”合并行“ |
| Cmd +                  |                 |                              |                                  |
| Cmd + U                | Ctrl + U        | 将光标的移动回退到上一个位置 | 撤销光标的移动和选择             |

### 搜索相关

| Mac 快捷键          | Win 快捷键          | 作用                                       | 备注   |
| ------------------- | ------------------- | ------------------------------------------ | ------ |
| **Cmd + Shift + F** | **Ctrl + Shift +F** | 全局搜索代码                               | 很常用 |
| **Cmd + P**         | **Ctrl + P**        | 在当前的项目工程里，**全局**搜索文件名     |        |
| Cmd + F             | Ctrl + F            | 在当前文件中搜索代码，光标在搜索框里       |        |
| **Cmd + G**         | **F3**              | 在当前文件中搜索代码，光标仍停留在编辑器里 | 很巧妙 |

