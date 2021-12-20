# Js调用本地exe的方式 - 底云飞 - 博客园
### 1.     使用记事本（或其他文本编辑器）创建一个 myprotocal.reg 文件，并写入以下内容

### Windows Registry Editor Version 5.00

### \[HKEY_CLASSES_ROOT\\Webshell]

### @="URL:Webshell Protocol Handler"

### "URL Protocol"=""

### \[HKEY_CLASSES_ROOT\\Webshell\\DefaultIcon]

### @="C:\\\\Program Files (x86)\\\\Tencent\\\\WeChat\\\\WeChat.exe"

### \[HKEY_CLASSES_ROOT\\Webshell\\shell]

### \[HKEY_CLASSES_ROOT\\Webshell\\shell\\open]

### \[HKEY_CLASSES_ROOT\\Webshell\\shell\\open\\command]

### @="\\"C:\\\\Program Files (x86)\\\\Tencent\\\\WeChat\\\\WeChat.exe\\" \\"%1\\""

### **2.** **修改参数**

### **修改步骤 \*\***1\***\* 中连接名称，如上面的**Webshell，该处为自定义名称，是 JS 调用时的 href，且必须使用全英文字符进行命名，共六处

###  ![](https://img2018.cnblogs.com/blog/811883/201908/811883-20190806140403717-684923065.png)

### 修改步骤 1 中的可执行文件路径，如上面的 "C:\\\\Program Files (x86)\\\\Tencent\\\\WeChat\\\\WeChat.exe"，文件路径需使用绝对路径，并且以\\\\进行分割，共两处

###  ![](https://img2018.cnblogs.com/blog/811883/201908/811883-20190806140414789-23931683.png)

### 3.     执行 myprotocal.reg 文件

### 4.     创建 JS 调用链接进行测试，连接地址为 步骤 1 中所命名的链接名称，后面加://hello，（hello 为传递参数，可任意添加）

###  ![](https://img2018.cnblogs.com/blog/811883/201908/811883-20190806140422970-1198833592.png)

 [https://www.cnblogs.com/diyunfei/p/11308515.html](https://www.cnblogs.com/diyunfei/p/11308515.html)
