# ssh连接打开多终端

```shell
sudo apt install screen
```

1、创建一个新的终端

```
screen -S XX
```

例如：创建一个名字为"core"的终端

```
screen -S core
```

输入后，会跳转到新的终端，可以在新的终端执行相关命令


2、  返回到原始的终端(退出当前screen，但保持其运行状态)

按Ctrl+a，然后再按d，就回到了最原始的终端界面。

3、查看所有终端  screen -ls

```
robot@robot-desktop:~$ screen -ls
There is a screen on:
        7527.core       (2021年08月15日 22时01分00秒)   (Detached)
1 Socket in /run/screen/S-robot.
```

其中，7525是当前终端的ID，core是screen的名称

4、重新进入已经存在的终端

可通过命名的screen名或者screen的id来进入对应的终端界面：

```
screen -r XX (XX可以是ID或者名字)
```

5、删除终端

```
screen -S XX -X quit (XX可以是ID或者名字)
```

原文链接：https://blog.csdn.net/wwwlyj123321/article/details/119720660