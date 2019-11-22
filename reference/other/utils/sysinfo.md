
----

# 系统信息获取接口

用以帮助应用获取系统信息，而不用处理一些硬件差异化逻辑。


### exec
> function sysinfo.exec(cmd)

执行命令行指令(cmd)，并获取其输出信息


### cat_file
> function sysinfo.cat_file(path)

获取指定路径文件(path)内容


### cpu_model
> function sysinfo.cpu_model()

获取当前硬件的CPU型号信息，如:ARMv7 Processor rev 5 (v7l)


### cpu_temperature
> function sysinfo.cpu_temperature()


获取设备CPU的温度信息。单位是摄氏度


### uname
> function sysinfo.uname(arg)

获取Linux系统uname信息，arg的值参考uname指令参数


### meminfo
> function sysinfo.meminfo()

获取设备内存信息，返回数据包含: 总量，使用量，空闲量。 如: {total=250696, used=44396, free=183112}。 单位是KB。


### uptime
> function sysinfo.uptime()

获取系统运行时长, 返回单位为秒的浮点数


### loadavg
> function sysinfo.loadavg()

获取系统运行负载情况，返回的结果包含: lavg_1, lavg_5, lavg_15, nr_running, nr_threads


### network_if
> function sysinfo.network_if(ifname)

获取网卡信息, 包含: hwaddr(MAC地址) ipv4(ip v4地址） ipv6(ip v6地址)


### network
> function sysinfo.network()

获取所有网卡信息，返回以网卡名称为key的table, 值为network_if函数返回的结果


### list_serial
> function sysinfo.list_serial()

枚举系统中串口(匹配规则为 /dev/ttyS* /dev/ttyUSB* /dev/ttyACM*)


### version
> function sysinfo.version()

获取FreeIOE版本


### skynet_version
> function sysinfo.skynet_version()

获取使用的核心软件(skynet)版本


### cpu_arch_short
> function sysinfo.cpu_arch_short()

获取CPU架构段名称。 如: arm, x86_64, mips等


### cpu_arch
> function sysinfo.cpu_arch()

获取CPU架构名称。 如armv7l, arm_cortex-a7_neon-vfpv4


### openwrt_cpu_arch
> function sysinfo.openwrt_cpu_arch()

获取OpenWRT格式的CPU架构信息,如果系统不是OpenWRT则返回空

### os_id
> function sysinfo.os_id()

获取操作系统类型。如: linux, openwrt


### platform
> function sysinfo.platform()

获取操作系统名称版本和CPU架构信息，如: openwrt/18.06/x86_64 openwrt/19.07/arm_cortex-a7_neon_vfpv4


### ioe_sn
> function sysinfo.ioe_sn()

获取设备的FreeIOE序列号


### firmware_version
> function sysinfo.firmware_version()

获取硬件系统固件版本信息(非操作系统版本)

### board_name
> function sysinfo.board_name()

获取硬件名称（仅限OpenWRT系统)。如: tgw303, tlink-x1, Q2040等。 读取的是/tmp/sysinfo/board_name文件


### data_dir
> function sysinfo.data_dir()

获取设备可用以存储数据的路径信息， 如返回/tmp则代表此设备没有存储器可以用来进行数据存储。


### TZ
> function sysinfo.TZ()

获取设备的时区名称。 OpenWRT下是/tmp/TZ文件内容，如CST-8。 普通Linux是的/etc/timezone内容，如Asia/Shanghai。 获取失败时返回UTC


