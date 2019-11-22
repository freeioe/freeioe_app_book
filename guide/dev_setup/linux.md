
---

# Linux原生开发环境

## 获取Skynet源码

[源码地址](https://github.com/srdgame/skynet)

### 编译Skynet

``` bash
git submodule init
git submodule update
make linux
```

## 获取FreeIOE源码

[源码地址](https://github.com/freeioe/freeioe)

### 搭建环境

进入已经编译好的skynet源码目录，将FreeIOE目录连接过来:

``` bash
ln -s ../freeioe ./ioe
```

### 运行FreeIOE

在skynet 目录下，运行：

``` bash
./skynet ioe/config
```


![linux_run](assets/linux_run.png "运行输出")





