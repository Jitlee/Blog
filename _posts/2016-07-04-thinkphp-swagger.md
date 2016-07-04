---
layout: post
title: ThinkPHP API接口文档 Swagger UI 整合工具
description: 基于TP5.0开发的简单版SwaggerUI 扫描器，够用就好
categories: [ThinkPHP]
image: /assets/media/swagger.png
geometry: 
tags: [PHP,Swagger UI,API]
---

> 公司采用的使用 SpringMVC + Swagger 框架搭建的一套API服务框架，手机客户端能够很方便的调用API服务。
Swagger的优点我就不细说了，下面我要说的如何将SwaggerUI整合到ThinkPHP 5.0中去。

#### ThinkPHP5下载载地址
[http://www.thinkphp.cn/down.html](http://www.thinkphp.cn/down.html)

#### ThinkPHP5完全开发手册
[http://www.kancloud.cn/manual/thinkphp5](http://www.kancloud.cn/manual/thinkphp5)

#### SwaggerUI
[http://swagger.io/swagger-ui/](http://swagger.io/swagger-ui/)

#### 本文demo源码下载
[https://github.com/Jitlee/thinkphp-swagger](https://github.com/Jitlee/thinkphp-swagger)

### 1. 基于ThinkPHP搭建一个简单手机登陆后台服务

#### 1.1. 配置利用ThinkPHP5.0 自动创建一个API模块

修改ThinkPHP应用根目录的build.php文件

{% highlight php %}

return [
    // 生成运行时目录
    '__dir__'  => ['runtime/cache', 'runtime/log', 'runtime/temp', 'runtime/template'],
    // 生成应用公共文件
    '__file__' => ['common.php', 'config.php', 'database.php'],

    // 定义api模块的自动生成 （按照实际定义的文件名生成）
    'api'     => [
        '__file__'   => ['common.php'],
        '__dir__'    => ['controller'],
        'controller' => ['Login'],
        'model'      => [],
        'view'       => [],
    ],
    // 其他更多的模块定义
];

{% endhighlight %}

命令行自动生成
在应用的更目录下执行，启动控制台，并执行如下命令：

{% highlight bash %}

>php think build --config build.php

{% endhighlight %}

如果看到输出Successed，则表示自动生成成功。

### 1.2. 修改刚刚生成的api模块下的config.php文件，默认输出都为json类型

{% highlight php %}

<?php
//配置文件
return [
	'default_return_type' => 'json', // 修改控制器默认输出json对象
	'url_param_type' => 1, // URL参数方式 0 按名称成对解析 1 按顺序解析
];

{% endhighlight %}

### 1.3. <span id = "anchor">在刚刚自动生成的/application/api/controoler/Passport.php文件下添加3个API方法</span>

{% highlight php %}

<?php
namespace app\api\controller;

/**
 * swagger: 登录相关
 */
class Passport
{
	/**
	 * post: 发送验证码
	 * path: sendVerify/{phone}/{deviceType}
	 * method: sendVerify
	 * param: phone - {string} 手机号
	 * param: deviceType - {int} = [0|1|2|3|4] 设备类型(0: android手机, 1: ios手机, 2: android平板, 3: ios平板, 4: pc)
	 */
	public function sendVerify($phone, $deviceType) {
		return [
			'code'		=> 200,
			'message'	=> '发送验证码',
			'data'		=> [
				'phone'			=> $phone,
				'deviceType'		=> $deviceType
			]
		];
	}
	
	/**
	 * post: 登陆
	 * path: login
	 * method: login
	 * param: phone - {string} 手机号
	 * param: password - {string} 密码
	 * param: deviceType - {int} = [0|1|2|3|4] 设备类型(0: android手机, 1: ios手机, 2: android平板, 3: ios平板, 4: pc)
	 * param: verifyCode - {string} = 0 验证码
	 */
	public function login($phone, $password, $deviceType, $verifyCode = '0') {
		return [
			'code'		=> 200,
			'message'	=> '登陆成功',
			'data'		=> [
				'phone'			=> $phone,
				'password'		=> $password,
				'deviceType'		=> $deviceType,
				'verifyCode'		=> $verifyCode
			]
		];
	}
	
	/**
	 * get: 获取配置
	 * path: profile
	 * method: profile
	 * param: keys - {string[]} 需要获取配置的Key值数组
	 */
	public function profile($keys) {
		return [
			'code'		=> 200,
			'message'	=> '获取成功',
			'data'		=> $keys
		];
	}
}

{% endhighlight %}

### 2. 下载SwaggerUI

从SwaggerUI的官网下载最新版的源码，直接拷贝dist目录到ThinkPHP应用到public目录下，并命名为swagger。

接下来修改swagger目录下到index.html中到swagger.json路径为本地路径（默认指向http://petstore.swagger.io/v2/swagger.json）

{% highlight php %}

//var url = window.location.search.match(/url=([^&]+)/);
//if (url && url.length > 1) {
//	url = decodeURIComponent(url[1]);
//} else {
//	url = "http://petstore.swagger.io/v2/swagger.json";
//}

var url = window.location.href.replace(window.location.hash, "").replace(/[^/]+$/, "swagger.json");

{% endhighlight %}

### 3. 整合ThinkPHP和SwaggerUI

整合的关键是，将[步骤1.3](#anchor)中注释转换成swagger.json文件，提供给Swagger UI生成API文档

在ThinkPHP应用的Public目录下创建api.php文件：

{% highlight php %}

<?php

// +----------------------------------------------------------------------
// | ThinkPHP SWAGGER [ 够用就好 ]
// +----------------------------------------------------------------------
// | Copyright (c) 2016 http://jitlee.com All rights reserved.
// +----------------------------------------------------------------------
// | Licensed ( http://jitlee.com/licenses/LICENSE-1.0 )
// +----------------------------------------------------------------------
// | Author: Jitlee.Wan <www.wpj@163.com>
// +----------------------------------------------------------------------

// 定义应用目录
define('APP_PATH', __DIR__ . '/../application');

$tags = array(); // Tags对象
$paths = array(); // Path数组

$module_dir = opendir(APP_PATH);
while(($module_name = readdir($module_dir)) !== false) {
	$module_path = APP_PATH . DIRECTORY_SEPARATOR . $module_name;    //构建子目录路径
	if(is_dir($module_path)) {
		$module = strtolower($module_name);
		
		$module_child_dir = opendir($module_path);
		while(($module_child_name = readdir($module_child_dir)) !== false) {
			$module_child_path = $module_path . DIRECTORY_SEPARATOR . $module_child_name;    //构建子目录路径
			if(is_dir($module_child_path) && $module_child_name == 'controller') {
				$controller_dir = opendir($module_child_path);
				while(($controller_file = readdir($controller_dir)) !== false) {
					$controller_path = $module_child_path . DIRECTORY_SEPARATOR . $controller_file;    //构建子目录路径
					$controller_name = strtolower(basename($controller_path, '.php'));
					$contents = file_get_contents($controller_path);
					if(preg_match_all('/swagger:\s*([^\n]+)/i', $contents, $swagger_matches)) {
						
						// 添加tag
						$found_tag = false;
						foreach($tags as $tag) {
							if($tag['name'] == $controller_name) {
								$found_tag = true;
								break;
							}
						}
						if(!$found_tag) {
							array_push($tags, array(
								'name'			=> $controller_name,
								'description'	=> $swagger_matches[1][0]
							));
						}
						
						// 添加path
						if(preg_match_all('/\/\*((?!\*\/).)+\*\//s', $contents, $func_matches)) {
							$length = count($func_matches[0]);
							if($length > 1) {
								for($i = 1; $i < $length; $i++) {
									$func_array = array();
									
									// 解析每个方法
									$func_contents = $func_matches[0][$i];
									
									// 方法说明
									if(!preg_match_all('/(get|post|delete)\s*:\s*([^\n]+)/i', $func_contents, $matches)) {
										break;
									}
									$method = $matches[1][0];
									$summary = $matches[2][0];
									
									// 路径
									if(!preg_match_all('/path\s*:\s*([^\n]+)/i', $func_contents, $matches)) {
										break;
									}
									$path = $matches[1][0];
									
									// 方法名称
									$operations = explode('/', $path);
									$operationId = $operations[0];
									if(preg_match_all('/method\s*:\s*([^\n]+)/i', $func_contents, $matches)) {
										$operationId = $matches[1][0];
									}
									
									$paths[$path] = array();
									$parameters = array();
									$func = array(
										'tags'			=> [$controller_name],
										'summary'		=> $summary,
										'description'	=> '',
										'operationId'	=> $operationId,
										'produces'		=> ['application/json']
									);
									
									// 参数
									$pattern = '/param\s*:\s*(?<name>\w+)\s*-\s*\{(?<type>\w+(?<array>\[\])?)\}\s*(=\s*((\[(?<enum>[^]]+)\])|(?<default>[^\s]+))\s*)?(?<summary>[^*]+)/i';
									if(preg_match_all($pattern, $func_contents, $matches)) {
										$names = $matches['name']; 		// 参数名称
										$types = $matches['type']; 		// 参数类型
										$enums = $matches['enum']; 		// 参数枚举
										$defaults = $matches['default']; // 默认值
										$summarys = $matches['summary']; // 参数说明
										$arrays = $matches['array']; // 参数说明
										
										$params_count = count($names);
										for($j = 0; $j < $params_count; $j++) {
											$in = $method == 'get' ? 'query' : 'formData';
											if(strpos($path, '{'.$names[$j].'}') !== false) {
												$in = 'path';
											}
											
											$parameter = array(
												'name'			=> $names[$j],
												'in'				=> $in,
												'required'		=> true,
												'description'	=> $summarys[$j]
											);
											
											if($defaults[$j] !== '') {
												$parameter['required'] = false;
												$parameter['defaultValue'] = $defaults[$j];	
											}
											
											$type = str_replace('[]', '', $types[$j]);
											if($type == 'int') {
												$type = 'integer';
											}
											
											if($arrays[$j] != '') { // 是否数据参数
												$parameter['type'] = 'array';
												$parameter['items'] = array(
													'type'	=> str_replace('[]', '', $type)
												);
												$parameter['collectionFormat'] = 'brackets'; // url带中括号
//												$parameter['collectionFormat'] = 'multi'; // url不带中括号
											} else if($enums[$j] != '') { // 是否枚举参数
												$enum = explode('|', $enums[$j]);
												$parameter['type'] = $type;
												$parameter['enum'] = $enum;
											} else {
												$parameter['type'] = $type;
											}
											array_push($parameters, $parameter);
										}
									}
									$func['parameters'] = $parameters;
									// 生成api访问路径
									$paths['/'.$module.'/'.$controller_name.'/'.$path][$method] = $func;
								}
							}
						}
					}
				}
				closedir($controller_dir);
			}
		}
		closedir($module_child_dir);
	}
}
closedir($module_dir);

$swagger = array(
	'swagger'	=> '2.0',
	'info'		=> array(
		'description'	=> 'APP 后台服务',
		'version'		=> '1.0.0',
		'title'			=> '［我的APP］Swagger',
		'termsOfService'=> 'http://www.ritacc.cn/',
		'contact'		=> array(
			'email'		=> 'www.wpj@163.com'
		),
		'license'		=> array(
			'name'		=> 'Apache 2.0',
			'url'		=> 'http://www.apache.org/licenses/LICENSE-2.0.html'
		)
	),
	'host'		=> $_SERVER['SERVER_NAME'] . ':' . $_SERVER['SERVER_PORT'],
	'basePath'	=> '',
	'tags'		=> $tags,
	'schemes'	=> [
		'http'
	],
	'paths'		=> $paths,
	'securityDefinitions'=> array(
		
	),
	'definitions'=> array(
	),
	'externalDocs'=> array(
		'description'	=> 'Find out more about Swagger',
		'url'			=> 'http://swagger.io'
	)
);

$jsonFile = fopen("swagger/swagger.json", "w") or die("Unable to open file!");
fwrite($jsonFile, json_encode($swagger));
fclose($jsonFile);

// 跳转到Swagger UI
$url = '/swagger/index.html';
Header('HTTP/1.1 303 See Other'); 
Header("Location: $url"); 
exit;

{% endhighlight %}

### 3. 启动应用，打开浏览器验证

在ThinkPHP应用的public文件下运行

{% highlight bash %}

>php -S localhost:8888 router.php

{% endhighlight %}

在浏览器中输入：[http://localhost:8888/api.php](http://localhost:8888/api.php)

<img src="{{ site.BASE_PATH }}{{page.image}}" width="85%"/>



