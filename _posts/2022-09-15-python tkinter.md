---
title: Python Tkinter详解
author: fnoobt
date: 2022-09-15 11:34:00 +0800
categories: [Python,Tkinter]
tags: [Python,Tkinter]
img_path: '/assets/img/commons/python'
---

## 简介

- tkinter
: tkinter(Tk interface)是Python的标准GUl库，支持跨平台的GUl程序开发。tkinter适合小型的GUl程序编写，也特别适合初学者学习GUl编程。

- wxPython
: wxPython是比较流行的GUI库，适合大型应用程序开发，功能强于tkinter,整体设计框架类似于MFC(MicrosoftFoundation Classes微软基础类库).

- PyQT
: Qt是一种开源的GUI库，适合大型GUI程序开发，PyQt是Qt工具包标准的Python实现。我们也可以使用QtDesginer界面设计器快速开发GUl应用程序.

## Tkinter编程

一个标准的用tkinter编写的GUI程序应当有以下几点：

1. tkinter模块的导入
2. 创建一个tkinter控件
3. 对于这个控件，指定master， 即这个控件属于哪一个
4. 告诉 GM(geometry manager) 有一个控件产生了

>注：py3和py2的tkinter模块的导入存在差异（本文下面的代码都是py3形式）

```python
#python3
import tkinter
 
#python2
import Tkinter
```

tkinter模块并不需要额外的导入，python在安装的时候就有了tkinter了，所以不要去pip install tkinter了。

### 窗口框架

tkinter的每个程序都需要一个窗口的框架，其由`导入`+`指定master`+`消息`循环组成

```python
import tkinter
root = tkinter.Tk()#通常习惯将这个变量名设置为root或者window
# 进入消息循环
root.mainloop()
```

## Tkinter 标准属性

标准属性也就是所有控件的共同属性，如大小，字体和颜色等等。

| 属性      | 描述     |
|-----------|----------|
| Dimension | 控件大小 |
| Color     | 控件颜色 |
| Font      | 控件字体 |
| Anchor    | 锚点     |
| Relief    | 控件样式 |
| Bitmap    | 位图     |
| Cursor    | 光标     |

一般来说，`Dimension`, `color`, `font`, `anchor` 和 `cursor`是比较常用的控件，想要做到对于一个简单的GUI程序得心应手，这五个属性是一定要熟练的。（除此之外`command`也很重要）

## Tkinter组件

Tkinter的提供各种控件，如按钮，标签和文本框，可以在一个GUI应用程序中使用。这些控件通常被称为控件或者部件。  
有15种比较常见的Tkinter的部件:

| 控件名称     | 描述                                                              |
|--------------|-------------------------------------------------------------------|
| Button       | 按钮控件；在程序中显示按钮。                                      |
| Canvas       | 画布控件；显示图形元素如线条或文本                                |
| Checkbutton  | 多选框控件；用于在程序中提供多项选择框                            |
| Entry        | 输入控件；用于显示简单的文本内容                                  |
| Frame        | 框架控件；在屏幕上显示一个矩形区域，多用来作为容器                |
| Label        | 标签控件；可以显示文本和位图                                      |
| Listbox      | 列表框控件；在Listbox窗口小部件是用来显示一个字符串列表给用户     |
| Menubutton   | 菜单按钮控件，用于显示菜单项。                                    |
| Menu         | 菜单控件；显示菜单栏,下拉菜单和弹出菜单                           |
| Message      | 消息控件；用来显示多行文本，与label比较类似                       |
| Radiobutton  | 单选按钮控件；显示一个单选的按钮状态                              |
| Scale        | 范围控件；显示一个数值刻度，为输出限定范围的数字区间              |
| Scrollbar    | 滚动条控件，当内容超过可视化区域时使用，如列表框。.               |
| Text         | 文本控件；用于显示多行文本                                        |
| Toplevel     | 容器控件；用来提供一个单独的对话框，和Frame比较类似               |
| Spinbox      | 输入控件；与Entry类似，但是可以指定输入范围值                     |
| PanedWindow  | PanedWindow是一个窗口布局管理的插件，可以包含一个或者多个子控件。 |
| LabelFrame   | labelframe 是一个简单的容器控件。常用与复杂的窗口布局。           |
| tkMessageBox | 用于显示你应用程序的消息框                                        |

其中，最常用的有`Button`，`Canvas`，`Checkbutton`，`Entry`，`Frame`，`Label`。

### Button控件

Button控件用于在 GUI程序中添加按钮，按钮上可以放上文本或图像，按钮可用于监听用户行为。可以与一个 函数关联，当按钮被按下时，会自动调用该函数。

语法：

```python
Button(master, option=value, ... )
```

- master: 按钮的父容器。
- options: 可选项，即该按钮的可设置的属性。这些选项可以用键 = 值的形式设置，并以逗号分隔。

实例：

```python
import tkinter as tk #导入模块
 
root = tk.Tk() 
root.title("python萌新花花")
root.geometry("114x514") #欸嘿
 
b = 0
 
def addOne(): #一个方法，每次按button就给b+1
    global b
    b += 1
    print(b)
 
a = tk.Button(root, text = "看这里", command = addOne).pack() #button，这里面的command就是调用前面的addOne方法
 
root.mainloop() #不能少的东西
```

效果：

![thinter_button](thinter_button.png)

### Canvas控件

Python Tkinter 画布（Canvas）组件和 html5 中的画布一样，都是用来绘图的。可以将图形，文本，小部件或框架放置在画布上。

语法：

```python
Canvas(master, option=value, ... )
```

- master: 按钮的父容器。
- options: 可选项，即该按钮的可设置的属性。这些选项可以用键 = 值的形式设置，并以逗号分隔。

canvas 支持的一些特殊的标准选项：

```python
#arc：创建扇形（不知道为啥，但是基本每个有画图功能的库都有扇形）
places = 10, 50, 240, 210
arc = canvas.create_arc(places, start=0, extent=150, fill="blue")
 
#image：创建图像
name = PhotoImage(file = "sunshine.gif")
image = canvas.create_image(50, 50, anchor=NE, image=name)
 
#line：创建线条
line = canvas.create_line(x0, y0, x1, y1, ..., xn, yn, options)
 
#oval: 创建一个圆（或许可以是个椭圆）
oval = canvas.create_oval(x0, y0, x1, y1, options)
 
#polygon：创建一个多边形（至少有三个顶点）
 
poly = canvas.create_polygon(x0, y0, x1, y1,...xn, yn, options)
```

实例：

```python
import tkinter as tk  # 创建一个矩形，指定画布的颜色为白色

root = tk.Tk() 
 
cv = tk.Canvas(root,bg = 'red') #创建一个Canvas，设置其背景色为红色
cv.create_rectangle(10,10,110,110) # 创建一个矩形，坐标为(10,10,110,110)
cv.pack()
 
root.mainloop()# 为明显起见，将背景色设置为红色，用以区别 root
```

效果：

![tkinter_canvas](tkinter_canvas.png)

### Checkbutton控件

Python Tkinter 复选框用来选取需要的选项，它前面有个小正方形的方块，如果选中则有一个对号，也可以再次点击以取消该对号来取消选中。 

语法：

```python
Checkbutton(master, option=value, ... )
```

- master: 按钮的父容器。
- options: 可选项，即该按钮的可设置的属性。这些选项可以用键 = 值的形式设置，并以逗号分隔。

实例：

```python
import tkinter as tk

root = tk.Tk()
root.title("python萌新花花")
 
var1 = tk.IntVar() #定义一个整型变量，可以被放进控件中，可以在程序中更改值
var2 = tk.IntVar()
c1 = tk.Checkbutton(root, text = "花花最棒了", variable = var1,
                 onvalue = 1, offvalue = 0, height=5,
                 width = 20)
c2 = tk.Checkbutton(root, text = "花花最厉害", variable = var2,
                 onvalue = 1, offvalue = 0, height=5,
                 width = 20)
c1.pack()
c2.pack()
root.mainloop()
```

效果：

![tkinter_checkbutton](tkinter_checkbutton.png)

### Entry控件

Python Tkinter 文本框用来让用户 输入 一行 文本字符串。（一般搭配StringVar而不是前面的IntVar）  
但是如果需要输入多行文本，可以使用Text组件；如果需要显示一行或多行文本且不允许用户修改，可以使用Label组件。

语法：

```python
Entry( master, option, ... )
```

- master: 按钮的父容器。
- options: 可选项，即该按钮的可设置的属性。这些选项可以用键 = 值的形式设置，并以逗号分隔。

实例：

```python
import tkinter as tk
 
root = tk.Tk()
root.title("python萌新花花")
 
L1 = tk.Label(root, text="看过来")
L1.pack(side = tk.LEFT)
E1 = tk.Entry(root, bd =5)
E1.pack(side = tk.RIGHT)
 
root.mainloop()
```

效果：

![tkinter_entry](tkinter_entry.png)

### Label控件 （最最最最常用）
Python Tkinter 标签控件（Label）指定的窗口中显示的文本和图像。  
标签控件（Label）指定的窗口中显示的文本和图像。  
你如果需要显示一行或多行文本且不允许用户修改，你可以使用Label控件。

语法：

```python
Label ( master, option, ... )
```

- master: 框架的父容器。
- options: 可选项，即该标签的可设置的属性。这些选项可以用键-值的形式设置，并以逗号分隔。

实例：

```python
import tkinter as tk
 
root = tk.Tk()
root.title("python萌新花花")
 
L1 = tk.Label(root, text="花花最可爱啦")
L1.pack()
 
root.mainloop()
```

效果：

![tkinter_label](tkinter_label.png)


### Tkinter 练习：
制作一个GUI程序，可以让使用者输入账号和密码。密码以` * `的 形式显示。在使用者点下确定后，程序会和提前存在txt中的密码和账号匹配。如果不存在该账号或者密码错误，提示并且询问是否需要创建新账户。如果是，将新的账号和密码存在text中。

代码：

```python
import tkinter as tk
import tkinter.messagebox
 
lan = "english"
getData = []
 
def signUp(usrName, usrPwd):
    with open('data.txt','a') as usrFile:
        usrFile.writelines("{}\n{}\n".format(usrName, usrPwd))
 
def compare():
    name = var_usr_name.get()
    pwd = var_usr_pwd.get()
    usrName, usrPwd = name,pwd
    getData = removeN()
    g = 0
    while(g<(len(getData)-1)):
        if usrName == getData[g] and g%2 == 0 and usrPwd == getData[g+1]:
            tk.messagebox.showinfo(title='login', message='Welcome, {}. You have successfully logged in.'.format(usrName))
            break
        g = g + 1
    if g == len(getData)-1:
        tk.messagebox.showwarning(title = "error",message = "The user name and password are incorrect.")
        
 
def removeN():
    asd = " "
    c = ""
    bd = []
    de = 0
    fg = "\n"
    with open('data.txt','r') as usrFile:
        asd = usrFile.readlines()
        for j in range(len(asd)):
            er = asd[j]
            pos = er.index(fg)
            cex = er[0:pos]
            bd.append(cex)
            de = 0
            cex = ""
        return bd
 
def language():
    global lan
    answer = False
    if lan == "english":
        answer = tk.messagebox.askquestion(title='language changing', message='Do you want to switch to Spanish?')
        if answer == "yes":
            lan = "spanish"
            d_name.set("elija el lenguaje")
            e_name.set("el nombre")
            f_name.set("cifra")
        else:
            pass
    elif lan == "spanish":
        answer = tk.messagebox.askquestion(title='language changing', message='Do you want to switch to English?')
        if answer == "yes":
            lan = "english"
            d_name.set("select language")
            e_name.set("user name")
            f_name.set("password")
        else:
            pass
    
 
window = tk.Tk()
window.title('Fake PowerSchool Help Students!')
window.geometry('500x500')
z = tk.Label(window,anchor = 'w',bg='white',
                    justify = 'center', width=500, height=500)
z.place(x = 0, y = 0)
a = tk.Label(window,anchor = 'w',text="PowerSchool SIS",
                    fg = 'white', bg='darkblue', font=('TimesNewRoman', 30),
                    justify = 'center', width=20, height=1)
a.place(x = 50, y = 100)
 
b = tk.Label(window,anchor='nw',text="Student and Parent Sign In",
             fg='black',bg='white',font=('TimesNewRoman',12),
             justify='center',width=50,height=20)
b.place(x = 50, y = 150)
 
c = tk.Button(window, text='language', width=15,
              height=2, command=language)
c.place(x = 270,y = 200)
 
d_name = tk.StringVar()
d_name.set("select language")
d = tk.Label(window,anchor='nw',textvariable=d_name,
             fg='black',bg='white',font=('TimesNewRoman',12),
             justify='center',width=20,height=2)
d.place(x = 50, y = 210)
 
e_name = tk.StringVar()
e_name.set("user name")
e = tk.Label(window,anchor='nw',textvariable=e_name,
             fg='black',bg='white',font=('TimesNewRoman',12),
             justify='center',width=50,height=1)
e.place(x = 50, y = 290)
 
f_name = tk.StringVar()
f_name.set("password")
f = tk.Label(window,anchor='nw',textvariable=f_name,
             fg='black',bg='white',font=('TimesNewRoman',12),
             justify='center',width=50,height=1)
f.place(x = 50, y = 350)
 
g = tk.Button(window, text='login', width=4,
              height=1, command=compare)
g.place(x = 330,y = 410)
 
var_usr_name = tk.StringVar()
entry_usr_name = tk.Entry(window, textvariable=var_usr_name)
entry_usr_name.place(x=270, y=290)
var_usr_pwd = tk.StringVar()
entry_usr_pwd = tk.Entry(window, textvariable=var_usr_pwd, show='*')
entry_usr_pwd.place(x=270, y=350)
 
window.mainloop()
```

效果：

![tkinter_practice](tkinter_practice.png)

****

[原文出处](https://blog.csdn.net/weixin_56177871/article/details/124256679)  