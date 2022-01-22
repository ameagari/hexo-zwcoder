---
title: WordPress实现多域名访问
date: 2022-01-22 10:53:35
tags: wordpress
categories: [wordpress]
---
> 默认情况下，Wordpress会在安装时自动获取当前的域名或者IP，并与当前访问的端口后组装成URL放在设置字段，用户可以按照需求自行修改。当用户想通过不同的域名来访问该Wordpress时，总会被自动跳转到设置中的URL，因此实现多域名访问需要一定的修改。

## 常规修改方案

默认情况下，通过设置->常规界面可以修改当前站点的WordPress地址（URL）和站点地址（URL），wordpress会固定使用这两个URL地址进行访问，如下图所示。

## 使Wordpress支持多域名
网上很容易搜索到解决方法，大部分帖子提供的方法是修改**wp-config.php**文件，写入以下信息：

{% codeblock %}
define('WP_SITEURL', 'http://' . $_SERVER['HTTP_HOST']);
define('WP_HOME', 'http://' . $_SERVER['HTTP_HOST']);
{% endcodeblock %}
原理说起来其实很简单，通过$_SERVER语法获取当前访问的HOST信息，填入对应的环境变量WP_SITEURL和WP_HOME，这时我们通过任意域名访问后可以发现，在设置中对应的配置已经变成我们当前访问的域名，同时变为灰色且无法修改。但这种简单的脚本缺陷显而易见，不支持HTTPS，如果我们想通过https来访问，会指向http域名，导致出错，这里提供一个可以兼容HTTP和HTTPS的脚本：

{% codeblock lang:php%}
$pageURL = 'http';
if ($_SERVER["HTTPS"] == "on") {$pageURL .= "s";}
$pageURL .= "://";
if ($_SERVER["SERVER_PORT"] != "80" and $_SERVER["SERVER_PORT"] != "443") {
    $pageURL .= $_SERVER["SERVER_NAME"].":".$_SERVER["SERVER_PORT"];
} else {
    $pageURL .= $_SERVER["SERVER_NAME"];
}
define('WP_SITEURL', $pageURL);
define('WP_HOME', $pageURL);
{% endcodeblock %}

基本原理就是通过解析url，判断是否为https，如果是则动态添加一个"s"到pageURL变量。

### 最终效果
如下所示，能够完美的支持http，https和端口号配置。
{% asset_img http_result.PNG 若用https连接这里会显示为https:// %}

