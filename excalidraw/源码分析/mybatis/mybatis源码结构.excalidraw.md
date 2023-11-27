---

excalidraw-plugin: parsed
tags: [excalidraw]

---
==⚠  Switch to EXCALIDRAW VIEW in the MORE OPTIONS menu of this document. ⚠==

# Text Elements
包含所有 Mapper 接口中用到的注解，如常用的 @Param、@Update、@Select、@Delete 等 ^ULz08yvM

使用 JDK 动态代理的方式生成 Mapper 接口的具体实例并进行注册管理 ^cKfJixiD

包含所有用于封装 Configuration 全局配置对象的各种构建器，如：
XMLConfigBuilder 用于解析 mybatis-config.xml 核心配置文件；
XMLMapperBuilder 用于解析 Mapper.xml 映射配置文件；
XMLStatementBuilder 用于解析 insert、delete、update、select 等标签；
MapperAnnotationBuilder 用于解析 Mapper 接口中方法上的各种注解； ^xfC00qDL

缓存模块的具体实现，包含各种缓存装饰器 ^1WoTY7tP

数据源和数据源连接池的具体实现 ^4LWIfd7m

核心组件：执行器，用于实现参数映射、SQL解析、SQL执行、结果集映射等功能 ^0opHVzVQ

资源加载模块，主要用于读取 mybatis-config.xml 和 mapper.xml 等配置文件 ^jL31LRHT

日志模块，实现与多种日志框架的对接以及利用动态代理的方式增强所有 JDBC 操作，
实现在 Debug 模式下能够打印出日志信息 ^DnaJfO6x

映射配置文件中各部分（Mapper、参数、结果集、SQL）所对应封装的实体类 ^izQSqOxn

解析器模块，其中的 GenericTokenParser 解析器用于解析 ${}、#{} 占位符，
XPathParser 解析器用于解析 XML 配置文件， PropertyParser 解析器用于解析 property 属性值 ^XHQ8vjym

插件模块，通过动态代理和责任链的方式实现拦截器链 ^50OzdsKP

反射工具箱，简化反射操作 ^tEsWdF6c

负责动态生成 SQL，解析 SQL 中的动态标签，形成 SQL 模板，然后再处理 SQL 模板中的占位符，
根据运行时用户传入的实参填充占位符，得到真正可执行的 SQL 语句 ^qiSDtBts

门面模式，与用户打交道，其中最为重要的 SqlSession 接口包含常见的获取 Mapper、CRUD、控制事务等功能 ^6pBvULBM

封装 JDBC 的事务管理，包括开启连接、事务的提交、回滚以及连接的关闭等功能 ^8IjNkOi4

类型处理器，包含所有常见的数据库类型与 Java 类型进行转换的处理器，
如果想自定义类型处理器，需要实现 TypeHandler 接口或继承自 BaseTypeHandler 类 ^XmnTMRKd


# Embedded files
172a96eb014e04595cc85140840e1861c7dd8ab4: [[../../../attachments/Pasted Image 20231123115608_338.png]]

%%
# Drawing
```json
{
	"type": "excalidraw",
	"version": 2,
	"source": "https://github.com/zsviczian/obsidian-excalidraw-plugin/releases/tag/2.0.1",
	"elements": [
		{
			"type": "image",
			"version": 81,
			"versionNonce": 1008776126,
			"isDeleted": false,
			"id": "puVAhfAw30m3QsfF29de6",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -3473.723546272069,
			"y": -2007.485242920523,
			"strokeColor": "transparent",
			"backgroundColor": "transparent",
			"width": 468.2637362637363,
			"height": 804,
			"seed": 331491847,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1696924646816,
			"link": null,
			"locked": false,
			"status": "saved",
			"fileId": "172a96eb014e04595cc85140840e1861c7dd8ab4",
			"scale": [
				1,
				1
			]
		},
		{
			"type": "text",
			"version": 293,
			"versionNonce": 440659746,
			"isDeleted": false,
			"id": "ULz08yvM",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -2977.415237105107,
			"y": -1896.2792323443696,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 680.1597290039062,
			"height": 20,
			"seed": 1224253577,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [
				{
					"id": "ElAtS5IyYWrvPNeEXY-nv",
					"type": "arrow"
				}
			],
			"updated": 1696924646816,
			"link": null,
			"locked": false,
			"customData": {
				"legacyTextWrap": true
			},
			"fontSize": 16,
			"fontFamily": 4,
			"text": "包含所有 Mapper 接口中用到的注解，如常用的 @Param、@Update、@Select、@Delete 等",
			"rawText": "包含所有 Mapper 接口中用到的注解，如常用的 @Param、@Update、@Select、@Delete 等",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "包含所有 Mapper 接口中用到的注解，如常用的 @Param、@Update、@Select、@Delete 等",
			"lineHeight": 1.25,
			"baseline": 14
		},
		{
			"type": "arrow",
			"version": 150,
			"versionNonce": 261065726,
			"isDeleted": false,
			"id": "ElAtS5IyYWrvPNeEXY-nv",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -3168.8065684256458,
			"y": -1785.7182205621175,
			"strokeColor": "#e03131",
			"backgroundColor": "transparent",
			"width": 181.10048999429864,
			"height": 103.84687165957894,
			"seed": 683222023,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1696924646816,
			"link": null,
			"locked": false,
			"startBinding": null,
			"endBinding": {
				"elementId": "ULz08yvM",
				"focus": 0.9956344140790897,
				"gap": 10.290841326240297
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					181.10048999429864,
					-103.84687165957894
				]
			]
		},
		{
			"type": "text",
			"version": 237,
			"versionNonce": 469613282,
			"isDeleted": false,
			"id": "cKfJixiD",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -2977.415237105107,
			"y": -1858.6122781110435,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 518.7679443359375,
			"height": 20,
			"seed": 717551815,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [
				{
					"id": "0KZ1bEgf0M-12VN9z1hHO",
					"type": "arrow"
				}
			],
			"updated": 1696924646816,
			"link": null,
			"locked": false,
			"customData": {
				"legacyTextWrap": true
			},
			"fontSize": 16,
			"fontFamily": 4,
			"text": "使用 JDK 动态代理的方式生成 Mapper 接口的具体实例并进行注册管理",
			"rawText": "使用 JDK 动态代理的方式生成 Mapper 接口的具体实例并进行注册管理",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "使用 JDK 动态代理的方式生成 Mapper 接口的具体实例并进行注册管理",
			"lineHeight": 1.25,
			"baseline": 14
		},
		{
			"type": "arrow",
			"version": 179,
			"versionNonce": 2012830782,
			"isDeleted": false,
			"id": "0KZ1bEgf0M-12VN9z1hHO",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -3198.537260669613,
			"y": -1755.0584441855262,
			"strokeColor": "#e03131",
			"backgroundColor": "transparent",
			"width": 212.0582855538646,
			"height": 99.50974949055012,
			"seed": 55621351,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1696924646816,
			"link": null,
			"locked": false,
			"startBinding": null,
			"endBinding": {
				"elementId": "cKfJixiD",
				"focus": 1.0016571709770927,
				"gap": 9.063738010641373
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					212.0582855538646,
					-99.50974949055012
				]
			]
		},
		{
			"type": "text",
			"version": 613,
			"versionNonce": 604507810,
			"isDeleted": false,
			"id": "xfC00qDL",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -2977.415237105107,
			"y": -1827.7630206983008,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 562.1597900390625,
			"height": 100,
			"seed": 1022305777,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [
				{
					"id": "R-Zkz9tozx3Z6LPvtBqds",
					"type": "arrow"
				}
			],
			"updated": 1696924646817,
			"link": null,
			"locked": false,
			"customData": {
				"legacyTextWrap": true
			},
			"fontSize": 16,
			"fontFamily": 4,
			"text": "包含所有用于封装 Configuration 全局配置对象的各种构建器，如：\nXMLConfigBuilder 用于解析 mybatis-config.xml 核心配置文件；\nXMLMapperBuilder 用于解析 Mapper.xml 映射配置文件；\nXMLStatementBuilder 用于解析 insert、delete、update、select 等标签；\nMapperAnnotationBuilder 用于解析 Mapper 接口中方法上的各种注解；",
			"rawText": "包含所有用于封装 Configuration 全局配置对象的各种构建器，如：\nXMLConfigBuilder 用于解析 mybatis-config.xml 核心配置文件；\nXMLMapperBuilder 用于解析 Mapper.xml 映射配置文件；\nXMLStatementBuilder 用于解析 insert、delete、update、select 等标签；\nMapperAnnotationBuilder 用于解析 Mapper 接口中方法上的各种注解；",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "包含所有用于封装 Configuration 全局配置对象的各种构建器，如：\nXMLConfigBuilder 用于解析 mybatis-config.xml 核心配置文件；\nXMLMapperBuilder 用于解析 Mapper.xml 映射配置文件；\nXMLStatementBuilder 用于解析 insert、delete、update、select 等标签；\nMapperAnnotationBuilder 用于解析 Mapper 接口中方法上的各种注解；",
			"lineHeight": 1.25,
			"baseline": 94
		},
		{
			"type": "arrow",
			"version": 99,
			"versionNonce": 188687486,
			"isDeleted": false,
			"id": "R-Zkz9tozx3Z6LPvtBqds",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -3209.1232111330323,
			"y": -1736.2630206983008,
			"strokeColor": "#e03131",
			"backgroundColor": "transparent",
			"width": 218,
			"height": 43.328713412159004,
			"seed": 135314769,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1696924646817,
			"link": null,
			"locked": false,
			"startBinding": null,
			"endBinding": {
				"elementId": "xfC00qDL",
				"focus": 0.5537617850394658,
				"gap": 13.707974027925502
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					218,
					-43.328713412159004
				]
			]
		},
		{
			"type": "text",
			"version": 140,
			"versionNonce": 655485538,
			"isDeleted": false,
			"id": "1WoTY7tP",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -2977.415237105107,
			"y": -1717.2630206983008,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 304,
			"height": 20,
			"seed": 568756977,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [
				{
					"id": "nKLhWe430NWxFxp7Aa8DI",
					"type": "arrow"
				}
			],
			"updated": 1696924646817,
			"link": null,
			"locked": false,
			"customData": {
				"legacyTextWrap": true
			},
			"fontSize": 16,
			"fontFamily": 4,
			"text": "缓存模块的具体实现，包含各种缓存装饰器",
			"rawText": "缓存模块的具体实现，包含各种缓存装饰器",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "缓存模块的具体实现，包含各种缓存装饰器",
			"lineHeight": 1.25,
			"baseline": 14
		},
		{
			"type": "arrow",
			"version": 64,
			"versionNonce": 2065818814,
			"isDeleted": false,
			"id": "nKLhWe430NWxFxp7Aa8DI",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -3218.1232111330323,
			"y": -1710.2630206983008,
			"strokeColor": "#e03131",
			"backgroundColor": "transparent",
			"width": 229,
			"height": 1,
			"seed": 16858591,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1696924646817,
			"link": null,
			"locked": false,
			"startBinding": null,
			"endBinding": {
				"elementId": "1WoTY7tP",
				"focus": 0.44214085750529303,
				"gap": 11.707974027925502
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					229,
					-1
				]
			]
		},
		{
			"type": "text",
			"version": 111,
			"versionNonce": 736775714,
			"isDeleted": false,
			"id": "4LWIfd7m",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -2977.415237105107,
			"y": -1679.2630206983008,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 240,
			"height": 20,
			"seed": 1827227519,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [
				{
					"id": "4-fM9sIdzOiqFohxNdl40",
					"type": "arrow"
				}
			],
			"updated": 1696924646817,
			"link": null,
			"locked": false,
			"customData": {
				"legacyTextWrap": true
			},
			"fontSize": 16,
			"fontFamily": 4,
			"text": "数据源和数据源连接池的具体实现",
			"rawText": "数据源和数据源连接池的具体实现",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "数据源和数据源连接池的具体实现",
			"lineHeight": 1.25,
			"baseline": 14
		},
		{
			"type": "arrow",
			"version": 92,
			"versionNonce": 423795966,
			"isDeleted": false,
			"id": "4-fM9sIdzOiqFohxNdl40",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -3181.1232111330323,
			"y": -1665.2630206983008,
			"strokeColor": "#e03131",
			"backgroundColor": "transparent",
			"width": 192,
			"height": 5.000000000000227,
			"seed": 553055199,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1696924646817,
			"link": null,
			"locked": false,
			"startBinding": null,
			"endBinding": {
				"elementId": "4LWIfd7m",
				"focus": 0.3375158214839792,
				"gap": 11.707974027925502
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					192,
					-5.000000000000227
				]
			]
		},
		{
			"type": "text",
			"version": 164,
			"versionNonce": 125843938,
			"isDeleted": false,
			"id": "0opHVzVQ",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -2977.415237105107,
			"y": -1637.2630206983008,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 573.1839599609375,
			"height": 20,
			"seed": 1267891167,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [
				{
					"id": "9CkDFZDOLVdBwiBLKSIF7",
					"type": "arrow"
				}
			],
			"updated": 1696924646817,
			"link": null,
			"locked": false,
			"customData": {
				"legacyTextWrap": true
			},
			"fontSize": 16,
			"fontFamily": 4,
			"text": "核心组件：执行器，用于实现参数映射、SQL解析、SQL执行、结果集映射等功能",
			"rawText": "核心组件：执行器，用于实现参数映射、SQL解析、SQL执行、结果集映射等功能",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "核心组件：执行器，用于实现参数映射、SQL解析、SQL执行、结果集映射等功能",
			"lineHeight": 1.25,
			"baseline": 14
		},
		{
			"type": "arrow",
			"version": 94,
			"versionNonce": 1528974654,
			"isDeleted": false,
			"id": "9CkDFZDOLVdBwiBLKSIF7",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -3197.1232111330323,
			"y": -1618.2630206983008,
			"strokeColor": "#e03131",
			"backgroundColor": "transparent",
			"width": 211,
			"height": 10.211976375746872,
			"seed": 1558945727,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1696924646817,
			"link": null,
			"locked": false,
			"startBinding": null,
			"endBinding": {
				"elementId": "0opHVzVQ",
				"focus": 0.6451177450348572,
				"gap": 8.707974027925502
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					211,
					-10.211976375746872
				]
			]
		},
		{
			"type": "text",
			"version": 181,
			"versionNonce": 417508770,
			"isDeleted": false,
			"id": "jL31LRHT",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -2977.415237105107,
			"y": -1606.2630206983008,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 551.5357666015625,
			"height": 20,
			"seed": 296508095,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [
				{
					"id": "qqOir4uEYdb6Uqnl6rA-J",
					"type": "arrow"
				}
			],
			"updated": 1696924646817,
			"link": null,
			"locked": false,
			"customData": {
				"legacyTextWrap": true
			},
			"fontSize": 16,
			"fontFamily": 4,
			"text": "资源加载模块，主要用于读取 mybatis-config.xml 和 mapper.xml 等配置文件",
			"rawText": "资源加载模块，主要用于读取 mybatis-config.xml 和 mapper.xml 等配置文件",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "资源加载模块，主要用于读取 mybatis-config.xml 和 mapper.xml 等配置文件",
			"lineHeight": 1.25,
			"baseline": 14
		},
		{
			"type": "arrow",
			"version": 110,
			"versionNonce": 1579283838,
			"isDeleted": false,
			"id": "qqOir4uEYdb6Uqnl6rA-J",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -3237.1232111330323,
			"y": -1591.2630206983008,
			"strokeColor": "#e03131",
			"backgroundColor": "transparent",
			"width": 248,
			"height": 0.2714034178752627,
			"seed": 1081812575,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1696924646817,
			"link": null,
			"locked": false,
			"startBinding": null,
			"endBinding": {
				"elementId": "jL31LRHT",
				"focus": -0.43008286393929124,
				"gap": 11.707974027925502
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					248,
					-0.2714034178752627
				]
			]
		},
		{
			"type": "text",
			"version": 319,
			"versionNonce": 88123746,
			"isDeleted": false,
			"id": "DnaJfO6x",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -2977.415237105107,
			"y": -1565.2630206983008,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 620.5599365234375,
			"height": 40,
			"seed": 885158015,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [
				{
					"id": "hIyHoctbVSrx7qutihKjY",
					"type": "arrow"
				}
			],
			"updated": 1696924646817,
			"link": null,
			"locked": false,
			"customData": {
				"legacyTextWrap": true
			},
			"fontSize": 16,
			"fontFamily": 4,
			"text": "日志模块，实现与多种日志框架的对接以及利用动态代理的方式增强所有 JDBC 操作，\n实现在 Debug 模式下能够打印出日志信息",
			"rawText": "日志模块，实现与多种日志框架的对接以及利用动态代理的方式增强所有 JDBC 操作，\n实现在 Debug 模式下能够打印出日志信息",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "日志模块，实现与多种日志框架的对接以及利用动态代理的方式增强所有 JDBC 操作，\n实现在 Debug 模式下能够打印出日志信息",
			"lineHeight": 1.25,
			"baseline": 34
		},
		{
			"type": "arrow",
			"version": 119,
			"versionNonce": 1803720126,
			"isDeleted": false,
			"id": "hIyHoctbVSrx7qutihKjY",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -3205.1232111330323,
			"y": -1521.2630206983008,
			"strokeColor": "#e03131",
			"backgroundColor": "transparent",
			"width": 218,
			"height": 22.38109947209159,
			"seed": 222996831,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1696924646817,
			"link": null,
			"locked": false,
			"startBinding": null,
			"endBinding": {
				"elementId": "DnaJfO6x",
				"focus": 0.5959492107076322,
				"gap": 9.707974027925502
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					218,
					-22.38109947209159
				]
			]
		},
		{
			"type": "text",
			"version": 191,
			"versionNonce": 1944661282,
			"isDeleted": false,
			"id": "izQSqOxn",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -2977.415237105107,
			"y": -1507.2630206983008,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 548.9279174804688,
			"height": 20,
			"seed": 91234847,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [
				{
					"id": "cG1aOUxhTc6BQ-td8CpdE",
					"type": "arrow"
				}
			],
			"updated": 1696924646818,
			"link": null,
			"locked": false,
			"customData": {
				"legacyTextWrap": true
			},
			"fontSize": 16,
			"fontFamily": 4,
			"text": "映射配置文件中各部分（Mapper、参数、结果集、SQL）所对应封装的实体类",
			"rawText": "映射配置文件中各部分（Mapper、参数、结果集、SQL）所对应封装的实体类",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "映射配置文件中各部分（Mapper、参数、结果集、SQL）所对应封装的实体类",
			"lineHeight": 1.25,
			"baseline": 14
		},
		{
			"type": "arrow",
			"version": 86,
			"versionNonce": 2087005694,
			"isDeleted": false,
			"id": "cG1aOUxhTc6BQ-td8CpdE",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -3203.1232111330323,
			"y": -1495.2630206983008,
			"strokeColor": "#e03131",
			"backgroundColor": "transparent",
			"width": 218,
			"height": 2.18256103259705,
			"seed": 596662303,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1696924646818,
			"link": null,
			"locked": false,
			"startBinding": null,
			"endBinding": {
				"elementId": "izQSqOxn",
				"focus": 0.23146194803982673,
				"gap": 7.707974027925502
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					218,
					-2.18256103259705
				]
			]
		},
		{
			"type": "text",
			"version": 269,
			"versionNonce": 1246443746,
			"isDeleted": false,
			"id": "XHQ8vjym",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -2977.415237105107,
			"y": -1476.2630206983008,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 722.1437377929688,
			"height": 40,
			"seed": 1903414385,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [
				{
					"id": "sCRkDoyEQ0WpKBynbmjbn",
					"type": "arrow"
				}
			],
			"updated": 1696924646818,
			"link": null,
			"locked": false,
			"customData": {
				"legacyTextWrap": true
			},
			"fontSize": 16,
			"fontFamily": 4,
			"text": "解析器模块，其中的 GenericTokenParser 解析器用于解析 ${}、#{} 占位符，\nXPathParser 解析器用于解析 XML 配置文件， PropertyParser 解析器用于解析 property 属性值",
			"rawText": "解析器模块，其中的 GenericTokenParser 解析器用于解析 ${}、#{} 占位符，\nXPathParser 解析器用于解析 XML 配置文件， PropertyParser 解析器用于解析 property 属性值",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "解析器模块，其中的 GenericTokenParser 解析器用于解析 ${}、#{} 占位符，\nXPathParser 解析器用于解析 XML 配置文件， PropertyParser 解析器用于解析 property 属性值",
			"lineHeight": 1.25,
			"baseline": 34
		},
		{
			"type": "arrow",
			"version": 110,
			"versionNonce": 1036697150,
			"isDeleted": false,
			"id": "sCRkDoyEQ0WpKBynbmjbn",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -3202.1232111330323,
			"y": -1474.2630206983008,
			"strokeColor": "#e03131",
			"backgroundColor": "transparent",
			"width": 217,
			"height": 11.817153863153862,
			"seed": 1262594463,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1696924646818,
			"link": null,
			"locked": false,
			"startBinding": null,
			"endBinding": {
				"elementId": "XHQ8vjym",
				"focus": -0.3261681304745422,
				"gap": 7.707974027925502
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					217,
					11.817153863153862
				]
			]
		},
		{
			"type": "text",
			"version": 242,
			"versionNonce": 1744161954,
			"isDeleted": false,
			"id": "50OzdsKP",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -2977.415237105107,
			"y": -1426.2630206983008,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 384,
			"height": 20,
			"seed": 1748054897,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [
				{
					"id": "EhMKwo8BOuHsh_O6br3tz",
					"type": "arrow"
				}
			],
			"updated": 1696924646818,
			"link": null,
			"locked": false,
			"customData": {
				"legacyTextWrap": true
			},
			"fontSize": 16,
			"fontFamily": 4,
			"text": "插件模块，通过动态代理和责任链的方式实现拦截器链",
			"rawText": "插件模块，通过动态代理和责任链的方式实现拦截器链",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "插件模块，通过动态代理和责任链的方式实现拦截器链",
			"lineHeight": 1.25,
			"baseline": 14
		},
		{
			"type": "arrow",
			"version": 147,
			"versionNonce": 345261694,
			"isDeleted": false,
			"id": "EhMKwo8BOuHsh_O6br3tz",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -3211.1232111330323,
			"y": -1448.2630206983008,
			"strokeColor": "#e03131",
			"backgroundColor": "transparent",
			"width": 223,
			"height": 26.6439802893226,
			"seed": 414581023,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1696924646818,
			"link": null,
			"locked": false,
			"startBinding": null,
			"endBinding": {
				"elementId": "50OzdsKP",
				"focus": -0.5726597621080912,
				"gap": 10.707974027925502
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					223,
					26.6439802893226
				]
			]
		},
		{
			"type": "text",
			"version": 109,
			"versionNonce": 1654122594,
			"isDeleted": false,
			"id": "tEsWdF6c",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -2979.415237105107,
			"y": -1396.2630206983008,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 192,
			"height": 20,
			"seed": 1221564209,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [
				{
					"id": "pvMgLtGH-5jDOFOluT33f",
					"type": "arrow"
				}
			],
			"updated": 1696924646818,
			"link": null,
			"locked": false,
			"customData": {
				"legacyTextWrap": true
			},
			"fontSize": 16,
			"fontFamily": 4,
			"text": "反射工具箱，简化反射操作",
			"rawText": "反射工具箱，简化反射操作",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "反射工具箱，简化反射操作",
			"lineHeight": 1.25,
			"baseline": 14
		},
		{
			"type": "arrow",
			"version": 182,
			"versionNonce": 562218686,
			"isDeleted": false,
			"id": "pvMgLtGH-5jDOFOluT33f",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -3181.1232111330323,
			"y": -1425.2630206983008,
			"strokeColor": "#e03131",
			"backgroundColor": "transparent",
			"width": 195.14099382227414,
			"height": 27.63190404652869,
			"seed": 628248799,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1696924646818,
			"link": null,
			"locked": false,
			"startBinding": null,
			"endBinding": {
				"elementId": "tEsWdF6c",
				"focus": -0.1337379858463106,
				"gap": 6.707974027925502
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					195.14099382227414,
					27.63190404652869
				]
			]
		},
		{
			"type": "text",
			"version": 504,
			"versionNonce": 1107332130,
			"isDeleted": false,
			"id": "qiSDtBts",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -2977.415237105107,
			"y": -1364.2630206983008,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 722.3679809570312,
			"height": 40,
			"seed": 249461087,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [
				{
					"id": "IkKJ8L87t9w9K2fesl40n",
					"type": "arrow"
				}
			],
			"updated": 1696924646818,
			"link": null,
			"locked": false,
			"customData": {
				"legacyTextWrap": true
			},
			"fontSize": 16,
			"fontFamily": 4,
			"text": "负责动态生成 SQL，解析 SQL 中的动态标签，形成 SQL 模板，然后再处理 SQL 模板中的占位符，\n根据运行时用户传入的实参填充占位符，得到真正可执行的 SQL 语句",
			"rawText": "负责动态生成 SQL，解析 SQL 中的动态标签，形成 SQL 模板，然后再处理 SQL 模板中的占位符，\n根据运行时用户传入的实参填充占位符，得到真正可执行的 SQL 语句",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "负责动态生成 SQL，解析 SQL 中的动态标签，形成 SQL 模板，然后再处理 SQL 模板中的占位符，\n根据运行时用户传入的实参填充占位符，得到真正可执行的 SQL 语句",
			"lineHeight": 1.25,
			"baseline": 34
		},
		{
			"type": "arrow",
			"version": 107,
			"versionNonce": 1161248510,
			"isDeleted": false,
			"id": "IkKJ8L87t9w9K2fesl40n",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -3187.1232111330323,
			"y": -1404.2630206983008,
			"strokeColor": "#e03131",
			"backgroundColor": "transparent",
			"width": 201,
			"height": 53.58552213982534,
			"seed": 939719839,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1696924646818,
			"link": null,
			"locked": false,
			"startBinding": null,
			"endBinding": {
				"elementId": "qiSDtBts",
				"focus": -0.7837422710398445,
				"gap": 8.707974027925502
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					201,
					53.58552213982534
				]
			]
		},
		{
			"type": "text",
			"version": 193,
			"versionNonce": 1008646114,
			"isDeleted": false,
			"id": "6pBvULBM",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -2977.415237105107,
			"y": -1311.2630206983008,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 793.7598266601562,
			"height": 20,
			"seed": 1163703121,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [
				{
					"id": "3rPYQoT7R8w32q_AMtqPs",
					"type": "arrow"
				}
			],
			"updated": 1696924646819,
			"link": null,
			"locked": false,
			"customData": {
				"legacyTextWrap": true
			},
			"fontSize": 16,
			"fontFamily": 4,
			"text": "门面模式，与用户打交道，其中最为重要的 SqlSession 接口包含常见的获取 Mapper、CRUD、控制事务等功能",
			"rawText": "门面模式，与用户打交道，其中最为重要的 SqlSession 接口包含常见的获取 Mapper、CRUD、控制事务等功能",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "门面模式，与用户打交道，其中最为重要的 SqlSession 接口包含常见的获取 Mapper、CRUD、控制事务等功能",
			"lineHeight": 1.25,
			"baseline": 14
		},
		{
			"type": "arrow",
			"version": 80,
			"versionNonce": 607301438,
			"isDeleted": false,
			"id": "3rPYQoT7R8w32q_AMtqPs",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -3208.1232111330323,
			"y": -1379.2630206983008,
			"strokeColor": "#e03131",
			"backgroundColor": "transparent",
			"width": 232,
			"height": 75.22984808787942,
			"seed": 811811583,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1696924646819,
			"link": null,
			"locked": false,
			"startBinding": null,
			"endBinding": {
				"elementId": "6pBvULBM",
				"focus": -0.9240460881758262,
				"gap": 8
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					232,
					75.22984808787942
				]
			]
		},
		{
			"type": "text",
			"version": 289,
			"versionNonce": 1895918498,
			"isDeleted": false,
			"id": "8IjNkOi4",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -2977.415237105107,
			"y": -1282.2630206983008,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 588.5599365234375,
			"height": 20,
			"seed": 215236095,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [
				{
					"id": "D8Bq1iU7LUgzN3U1RjPqG",
					"type": "arrow"
				}
			],
			"updated": 1696924646819,
			"link": null,
			"locked": false,
			"customData": {
				"legacyTextWrap": true
			},
			"fontSize": 16,
			"fontFamily": 4,
			"text": "封装 JDBC 的事务管理，包括开启连接、事务的提交、回滚以及连接的关闭等功能",
			"rawText": "封装 JDBC 的事务管理，包括开启连接、事务的提交、回滚以及连接的关闭等功能",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "封装 JDBC 的事务管理，包括开启连接、事务的提交、回滚以及连接的关闭等功能",
			"lineHeight": 1.25,
			"baseline": 14
		},
		{
			"type": "arrow",
			"version": 143,
			"versionNonce": 718673790,
			"isDeleted": false,
			"id": "D8Bq1iU7LUgzN3U1RjPqG",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -3182.1232111330323,
			"y": -1353.2630206983008,
			"strokeColor": "#e03131",
			"backgroundColor": "transparent",
			"width": 196,
			"height": 77.44831369756776,
			"seed": 1361694993,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1696924646819,
			"link": null,
			"locked": false,
			"startBinding": null,
			"endBinding": {
				"elementId": "8IjNkOi4",
				"focus": -0.9178926518794784,
				"gap": 8.707974027925502
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					196,
					77.44831369756776
				]
			]
		},
		{
			"type": "text",
			"version": 365,
			"versionNonce": 651667298,
			"isDeleted": false,
			"id": "XmnTMRKd",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -2977.415237105107,
			"y": -1249.2630206983008,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 625.56787109375,
			"height": 40,
			"seed": 823295775,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [
				{
					"id": "vevC4_jkyEKenLhceejfu",
					"type": "arrow"
				}
			],
			"updated": 1696924646819,
			"link": null,
			"locked": false,
			"customData": {
				"legacyTextWrap": true
			},
			"fontSize": 16,
			"fontFamily": 4,
			"text": "类型处理器，包含所有常见的数据库类型与 Java 类型进行转换的处理器，\n如果想自定义类型处理器，需要实现 TypeHandler 接口或继承自 BaseTypeHandler 类",
			"rawText": "类型处理器，包含所有常见的数据库类型与 Java 类型进行转换的处理器，\n如果想自定义类型处理器，需要实现 TypeHandler 接口或继承自 BaseTypeHandler 类",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "类型处理器，包含所有常见的数据库类型与 Java 类型进行转换的处理器，\n如果想自定义类型处理器，需要实现 TypeHandler 接口或继承自 BaseTypeHandler 类",
			"lineHeight": 1.25,
			"baseline": 34
		},
		{
			"type": "arrow",
			"version": 92,
			"versionNonce": 291665854,
			"isDeleted": false,
			"id": "vevC4_jkyEKenLhceejfu",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -3233.1232111330323,
			"y": -1338.2630206983008,
			"strokeColor": "#e03131",
			"backgroundColor": "transparent",
			"width": 249,
			"height": 97,
			"seed": 882838655,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1696924646819,
			"link": null,
			"locked": false,
			"startBinding": null,
			"endBinding": {
				"elementId": "XmnTMRKd",
				"focus": -0.7819243761657891,
				"gap": 6.707974027925502
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					249,
					97
				]
			]
		}
	],
	"appState": {
		"theme": "light",
		"viewBackgroundColor": "#ffffff",
		"currentItemStrokeColor": "#1e1e1e",
		"currentItemBackgroundColor": "transparent",
		"currentItemFillStyle": "solid",
		"currentItemStrokeWidth": 2,
		"currentItemStrokeStyle": "solid",
		"currentItemRoughness": 1,
		"currentItemOpacity": 100,
		"currentItemFontFamily": 1,
		"currentItemFontSize": 20,
		"currentItemTextAlign": "left",
		"currentItemStartArrowhead": null,
		"currentItemEndArrowhead": "arrow",
		"scrollX": 3525.911700580732,
		"scrollY": 2135.216145698301,
		"zoom": {
			"value": 0.9
		},
		"currentItemRoundness": "round",
		"gridSize": null,
		"gridColor": {
			"Bold": "#C9C9C9FF",
			"Regular": "#EDEDEDFF"
		},
		"currentStrokeOptions": null,
		"previousGridSize": null,
		"frameRendering": {
			"enabled": true,
			"clip": true,
			"name": true,
			"outline": true
		}
	},
	"files": {}
}
```
%%