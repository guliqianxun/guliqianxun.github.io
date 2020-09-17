# 配置jupyter

安装jupyter的工作环境往往不是我们希望使用的

### 修改工作环境

打开Windows的cmd，在cmd中输入`jupyter notebook --generate-config`如下图：

![](Recourse\查找出事路径.png)

在对应文件夹下打开

![](Recourse\配置文件位置.png)

```
## The directory to use for notebooks and kernels.
c.NotebookApp.notebook_dir = 'F:\jupyter'
```

修改路径即可

# 参考连接





[jupyter官方文档]([https://nbviewer.jupyter.org/github/ipython/ipython/blob/3.x/examples/Notebook/Notebook%20Basics.ipynb](https://nbviewer.jupyter.org/github/ipython/ipython/blob/3.x/examples/Notebook/Notebook Basics.ipynb))

