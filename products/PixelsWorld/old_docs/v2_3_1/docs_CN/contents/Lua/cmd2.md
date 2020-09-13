#pw.cmd(cmd代码)
---

**说明：**

- 运行CMD代码。

**参数：**

- pw.cmd(cmdCode). 
- cmdCode : 字符串

**返回：**

- 第一个返回值：如果命令正常结束，返回true， 否则返回**0** 。
- 第二个返回值如下： 
- **"exit"**：命令正常结束， 则返回命令**退出状态**的数字。
- **"signal"**：命令被信号终止，则返回终止命令的**信号**数字。

**示例：**

```lua:cmd.lua
pw.cmd("echo Hello PixelsWorld! & Pause ")
```
> 注意： **请勿**使用Mac OS上的 ```read```命令。 它会阻止渲染线程，并且**永远不会**再次渲染。如果被阻止，请删除```read```命令，保存项目，强制关闭AE再重新打开。 
