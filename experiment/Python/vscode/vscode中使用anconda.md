由于十分中意 vscode 的编译环境，但是又想使用 anconda 的虚拟环境控制，这里介绍了一下如何配置在 vscode 中使用 anconda 配置好的虚拟环境编译代码。

### 首先
确保 ** vscode**,  vscode插件**python**, **anconda** 都已经安装完毕可以正确执行

vscode 的 Terminal有多种选择，在windows下一般使用powershell作为终端。
这里需要使用管理员打开控制台  
输入： `conda init powershell`  

如果出现
```
无法加载文件C:\XXX\WindowsPowerShell\profile.ps1，因为在此系统上禁止运行脚本
```
的提示，

请执行

```bash
get-ExecutionPolicy
```

若回复 **Restricted**，表示状态是禁止的。

执行命令：

```bash
set-ExecutionPolicy RemoteSigned
```

将出现如下几个选项，输入 **Y** 并回车，设置完毕。

![](Resources\shellpolicy.png)



之后重启电脑，在vscode中输入`conda activate base`检测是否可以正确打开虚拟环境。

在正确打开虚拟环境之后，更换python编译器为虚拟环境的编译器，就可以了