# 云配置接口帮助接口

本接口为采集类应用封装了从云配置服务中加载设备模板的逻辑。帮助用户快速使用云配置服务。

* helper:initialize\(sys\_api, conf, templates\_ext, templates\_dir, templates\_node, devices\_node\)

构造函数。参数:
sys\_api - 应用系统接口 app.sys
conf - 应用配置数据 (table)
templates\_ext - 设备模板文件本地存储的扩展名。默认为csv
templates\_dir - 设备模板文件本地存储的子目录名。 默认为tpl
templates\_node - 应用配置数据中模板列表节点名称。默认为tpls
devices\_node - 应用配置数据中设备列表节点名称。默认为devs

* api:fetch\(async\)

获取所有设备模板数据文件。async为true时将开启异步获取模式。

* api:templates\(\)

获取已经完成获取的模板列表

* api:devices\(\)

获取已经完成设备模板的设备列表

使用代码示例：
``` lua
	local conf = self._conf or {}
	local conf_api = self._sys:conf_api(conf.cnf or 'CNF000000001', 'cnf', 'tpl')

	local config_str, err = conf_api:data(conf.version or 1)
	if not config_str then
		self._log:warning("DLT645 conf loading failure", err)
	end

	local config = cjson.decode(config_str or "") or {}

	config.opt = config.opt or {
		--port = "/dev/ttymxc1",
		port = "/tmp/ttyS10",
		baudrate = 9600
	}

	config.tpls = config.tpls or {
		{ id = "TPL000000001", name = "S1", ver = 1 },
		{ id = "TPL000000001", name = "S2", ver = 1 }
	}

	config.devs = config.devs or {
		{ addr = 992233445566, name = 's1', sn = 'xxx-xx-1', tpl = 'S1' },
		{ addr = 990000000001, name = 's2', sn = 'xxx-xx-2', tpl = 'S2' },
	}
	local helper = conf_helper:new(self._sys, config)
	helper:fetch()

```

获取完成后文件目录：
```
tpl/
├── CNF000000001_1.cnf
├── TPL000000001_1.csv
├── tpl1.csv
└── tpl2.csv
 
```

应用配置数据示例\(CNF000000001\):
``` json
{
	"opt": {
		"port": "/dev/ttymxc1",
		"baudrate": 19200
	},
	"tpls": [{
			"id": "TPL000000001",
			"name": "tpl1",
			"ver": 1
		},
		{
			"id": "TPL000000001",
			"name": "tpl2",
			"ver": 1
		}
	],
	"devs": [{
			"addr": 991122334455,
			"name": "s01",
			"sn": "xxx-xx-xx-1",
			"tpl": "tpl1"
		},
		{
			"addr": 112233445566,
			"name": "s02",
			"sn": "xxx-xx-xx-2",
			"tpl": "tpl2"
		}
	],
	"loop_gap": 3000
}
```

设备模板示例\(TPL000000001\)：
``` csv
COMMENT,name,description,series,,,,
META,S2,Supper Meter Device,v1,,,,
,,,,,,,
COMMENT,name,description,data address,vt,offset,rate,format
INPUT,total,组合有功总电能(kWh),0x00000000,,,,
INPUT,total_positive,正向有功总电能(kWh),0x00010000,,,,
INPUT,total_negative,反向有功总电能(kWh),0x00020000,,,,
INPUT,balance,剩余电量(kWh),0x00900100,,,,
INPUT,overdraft,透支电量(kWh),0x00900101,,,,
INPUT,current_total,当前结算周期组合有功总累计用电量(kWh),0x000B0000,,,,
,,,,,,,
COMMENT,name,description,data address,vt,rate,format,
OUTPUT,xxx,xxx,0x00000000,,,,
```



