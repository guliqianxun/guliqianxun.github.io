# Pip 使用

在cmd命令行中安装python第三方库

例如：**python -m pip install --user requests**

经常因为下载速度受限或者链接失败而苦恼

换个pip下载源试试吧！！

https://www.jb51.net/article/98401.htm



(1):在windows文件管理器中,输入 **%APPDATA%**

(2):会定位到一个新的目录下，在该目录下新建pip文件夹，然后到pip文件夹里面去新建个pip.ini文件

(3):在新建的pip.ini文件中输入以下内容，搞定

```
[global]
timeout = 6000
index-url = http://pypi.douban.com/simple
trusted-host = pypi.douban.com
```

想换源的话。。

阿里云 http://mirrors.aliyun.com/pypi/simple/ 

中国科技大学 https://pypi.mirrors.ustc.edu.cn/simple/

豆瓣(douban)  个人体验较好 http://pypi.douban.com/simple/

Python官方 https://pypi.python.org/simple/

v2ex http://pypi.v2ex.com/simple/

中国科学院 http://pypi.mirrors.opencas.cn/simple/

清华大学 个人体验较好 https://pypi.tuna.tsinghua.edu.cn/simple/

下面的方法我试过，emmm没有好使，但还是记录一下

也可以直接使用用-i指令指定镜像源下载，不过这样每次下载库都要加上镜像网址，如下。

pip install -i http://mirrors.aliyun.com/pypi/simple/ 想要安装的库

pip install -i http://mirrors.aliyun.com/pypi/simple/ 

pip install -i http://mirrors.aliyun.com/pypi/simple/ 

有时在更新pip时出现错误

然后提示

No module named 'pip'

因为这个错误导致 pip找不到，

可以首先执行  python -m ensurepip  然后执行 python -m pip install --upgrade pip  即可更新完毕。