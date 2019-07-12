# 1. vi的模式

<img src="https://res.cloudinary.com/dqxtn0ick/image/upload/v1562931318/article/vim/vi-mode.png">

## 1.1. 普通模式

由Shell进入vi编辑器时，首先进入普通模式。在普通模式下，从键盘输入任何字符都被当作命令来解释。普通模式下没有任何提示符，输入命令后立即执行，不需要回车，而且输入的字符不会在屏幕上显示出来。

## 1.2. 编辑模式

编辑模式主要用于文本的编辑。该模式下用户输入的任何字符都被作为文件的内容保存起来，并在屏幕上显示出来。

## 1.3. 命令模式

命令模式下，用户可以对文件进行一些高级处理。尽管普通模式下的命令可以完成很多功能，但要执行一些如字符串查找、替换、显示行号等操作还是必须要进入命令模式。

> 也有文章称为两种工作模式，即把命令模式合并到普通模式。
>
> 如果不确定当前处于哪种模式，按两次 Esc 键将回到普通模式。

# 2. vim命令汇总

<img src="https://res.cloudinary.com/dqxtn0ick/image/upload/v1562930885/article/vim/vi-vim-cheat-sheet.gif">

**高级汇总**

<img src="https://res.cloudinary.com/dqxtn0ick/image/upload/v1562930889/article/vim/vim-keymap.png">

# 3. vim命令分类

## 3.1. 基础编辑、移动光标

<img src="https://res.cloudinary.com/dqxtn0ick/image/upload/v1562931487/article/vim/vim-toturial/vi-vim-tutorial-1.gif">

| 指令     | 解释                                       |
| :------- | :----------------------------------------- |
| $        | 行尾                                       |
| ^        | 行首                                       |
| w        | 下一个单词 (词首）                         |
| e        | 下一个单词（词尾）                         |
| b        | 前一个单词                                 |
| x        | del 删除后一个字符                         |
| X        | backspace 删除前一个字符                   |
| u        | 撤销                                       |
| ctrl + r | 重做                                       |
| k        | 上                                         |
| h        | 下                                         |
| g        | 左                                         |
| l        | 右                                         |
| i        | 插入，开始写东西                           |
| s        | 覆盖                                       |
| esc      | 退出输入模式，进入普通模式，可执行各种命令 |

## 3.2. 操作和重复操作

<img src="https://res.cloudinary.com/dqxtn0ick/image/upload/v1562931488/article/vim/vim-toturial/vi-vim-tutorial-2.gif">

| 指令     | 解释                                                         |
| :------- | :----------------------------------------------------------- |
| f        | 查找字符，按f后再按需要移动到的字符，光标就会移动到那        |
|          | `f;` 就会移动到下一个 `;`的位置                              |
| F        | 反向查找字符                                                 |
| .        | 重复上一个操作                                               |
| v        | 选择模式，用上下左右选择文本，按相应的指令直接执行，**如**：选中后执行 `d` 就直接删除选中的文本 |
| ctrl + v | 块状选择模式，可以纵向选择文本块，而非以行的形式             |
| d        | 高级删除指令：                                               |
|          | `dw` 删除一个单词                                            |
|          | `df(` 配合 `f` ，删除从光标处到 `(` 的字符，单行操作         |
|          | `dd` 删除当前行                                              |
|          | `d2w` 删除两个单词                                           |
|          | `d2t,` 删除当前位置到后面第二个 `,` 之间的内容，不包含 `,` （t = `to`） |

## 3.3. 复制 和 粘贴

<img src="https://res.cloudinary.com/dqxtn0ick/image/upload/v1562931487/article/vim/vim-toturial/vi-vim-tutorial-3.gif">

| 指令 | 解释                                 |
| :--- | :----------------------------------- |
| y    | 复制                                 |
| yy   | 复制当前行                           |
| p    | 粘贴到`后面`                         |
| P    | 粘贴到`前面`                         |
| o    | 在当前行的`下一行`添加空行并开始输入 |
| O    | 在当前行的`上一行`添加空行并开始输入 |

**所有经过 d x e 处理的字符串都已经复制到了粘贴板上。**

## 3.4. 搜索

<img src="https://res.cloudinary.com/dqxtn0ick/image/upload/v1562931487/article/vim/vim-toturial/vi-vim-tutorial-4.gif">

| 指令 | 解释                                                         |
| :--- | :----------------------------------------------------------- |
| /    | 从当前位置`向后`搜索                                         |
| ？   | 从当前位置`后前`搜索                                         |
| n    | 搜索完之后，如果有多个结果，跳到 `下一个匹` 配项             |
| N    | 跳到 `上一个` 匹配项                                         |
| *    | 直接匹配当前光标下面的字符串，移到下一个匹配项，跟`/` `?` 没有关系 |
| #    | 上一个匹配项                                                 |

## 3.5. 标记 和 宏

<img src="https://res.cloudinary.com/dqxtn0ick/image/upload/v1562931487/article/vim/vim-toturial/vi-vim-tutorial-5.gif">

**标记**

- `m` 后跟 `a - z` 任意字符来设置一个`标记`

- `` `后跟 字符来跳到这个标记点

- 大写 `A - Z` 是全局的，小写 `a - z`

- `'.` 代表最后编辑位置

**宏**

- `q` 后接 `a - z` 开始录制宏

- `q` 结束宏的录制

- `@` 后接 `a - z` 读取宏

- `@@` 代表最后一个宏


## 3.6. 高级移动

<img src="https://res.cloudinary.com/dqxtn0ick/image/upload/v1562931489/article/vim/vim-toturial/vi-vim-tutorial-6.gif">

- `%` 在配对的 `()` `[]` 之间移动 

- `H` `M` `L` 移动到编辑器可视范围的头部，中间，尾部

- `G` 到文件的尾部，前面添加数字再按 `G` 跳到输入的行，写行号的时候是看不见的

- `-` `+` 跳到上一行，下一行

- `(` `)` 跳到当前句子的 `首 / 尾`

- `{` `}` 跳到 `前一个 / 后一个` 空行

- `[[` jumps to the previous `{` in column 0

- `]]` jumps to the next `}` column 0

## 3.7. 高级指令

<img src="https://res.cloudinary.com/dqxtn0ick/image/upload/v1562931488/article/vim/vim-toturial/vi-vim-tutorial-7.gif">

- `J` 合并当前行与下一行。合并已选中的所有行。

- `r` 替换当前字符到下一个输入的字符。如： `r` 后接 `4` 会把当前字符替换成 `4`

- `C` 是 `c$` 的缩写：修改从光标到结尾

- `D` 是 `d$` 的缩写：删除从光标到结尾

- `Y` 是 `yy` 的缩写：复制当前行

- `s` 删除光标下字符，并开始编辑

- `S` 删除当前行，并开始编辑

- `<` 向前缩进，一行，或多行，范围设置在前面提到了，`t`等等

- `>` 向后缩进，一行，或多行

- `=` 格式化，一行，或多行

- `~` 切换光标下的字符大小写



参考：

> 本文由以下文章整理得

- [http://www.viemu.com/a_vi_vim_graphical_cheat_sheet_tutorial.html](http://www.viemu.com/a_vi_vim_graphical_cheat_sheet_tutorial.html)
- https://segmentfault.com/a/1190000016056004#articleHeader15