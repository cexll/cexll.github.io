---
title: PHP下载限速断点续传
subtitle: ""
date: 2021-06-24 13:40:34.605
lastmod: 2021-06-24 13:40:34.605
draft: true
author: "cexll"
authorLink: ""
description: ""

tags: [
    php,断点续传,限速
]
categories: [
    php
]

hiddenFromHomePage: false
hiddenFromSearch: false

featuredImage: ""
featuredImagePreview: ""

toc:
  enable: true
math:
  enable: false
lightgallery: false
license: ""
---

<!--more-->






> 参考 http://php20.cn

## 文件下载限速

这里直接代码演示
```php
<?php

$filePath = './test.zip';//文件
$fp=fopen($filePath,"r");
 
//取得文件大小
$fileSize=filesize($filePath);
 
header("Content-type:application/octet-stream");//设定header头为下载
header("Accept-Ranges:bytes");
header("Accept-Length:".$fileSize);//响应大小
header("Content-Disposition: attachment; filename=testName");//文件名
ob_end_clean();//缓冲区结束
ob_implicit_flush();//强制每当有输出的时候,即刻把输出发送到浏览器
header('X-Accel-Buffering: no'); // 不缓冲数据
$buffer=1024;
$bufferCount=0;
 
while(!feof($fp)&&$fileSize-$bufferCount>0){//循环读取文件数据
    $data=fread($fp,$buffer);
    $bufferCount+=$buffer;
    echo $data;//输出文件
    sleep(1);
}
 
fclose($fp);
```

## 文件断点续传

```php
<?php

$filePath = './2.txt';//文件
$fp=fopen($filePath,"r");

//取得文件大小
$fileSize=filesize($filePath);
$buffer=5000;
$bufferCount=0;
header("Content-type:application/octet-stream");//设定header头为下载
header("Content-Disposition: attachment; filename=2.txt");//文件名
if (!empty($_SERVER['HTTP_RANGE'])){
    //切割字符串
    $range = explode('-',substr($_SERVER['HTTP_RANGE'],6));
    fseek($fp,$range[0]);//移动文件指针到range上
    header('HTTP/1.1 206 Partial Content');
    header("Content-Range: bytes $range[0]-$fileSize/$fileSize");
    header("content-length:".$fileSize-$range[0]);
}else{
    header("Accept-Length:".$fileSize);//响应大小
}
 
ob_end_clean();//缓冲区结束
ob_implicit_flush();//强制每当有输出的时候,即刻把输出发送到浏览器
header('X-Accel-Buffering: no'); // 不缓冲数据
while(!feof($fp)&&$fileSize-$bufferCount>0){//循环读取文件数据
    $data=fread($fp,$buffer);
    $bufferCount+=$buffer;
    echo $data;//输出文件
    sleep(1);
}
fclose($fp);

```

## 多线程下载

```php
<?php
 
$filePath = '127.0.0.1';
//查看文件大小
$ch = curl_init();
 
$headerData = [
    "Range: bytes=0-1"
];
curl_setopt($ch, CURLOPT_HTTPHEADER, $headerData);
//curl_setopt($ch, CURLOPT_CUSTOMREQUEST, "HEAD");
curl_setopt($ch, CURLOPT_URL, $filePath);
curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1); // return don't print
curl_setopt($ch, CURLOPT_TIMEOUT, 0);
curl_setopt($ch, CURLOPT_USERAGENT, 'Mozilla/4.0 (compatible; MSIE 5.01; Windows NT 5.0)');
curl_setopt($ch, CURLOPT_FOLLOWLOCATION, 1); // 302 redirect
curl_setopt($ch, CURLOPT_MAXREDIRS, 7);
curl_setopt($ch, CURLOPT_HEADER, true);//需要获取header头
curl_setopt($ch, CURLOPT_NOBODY, 1);    //不需要body,只需要获取header头的文件大小
$sContent = curl_exec($ch);
// 获得响应结果里的：头大小
$headerSize = curl_getinfo($ch, CURLINFO_HEADER_SIZE);//获取header头大小
// 根据头大小去获取头信息内容
$header = substr($sContent, 0, $headerSize);//获取真实的header头
curl_close($ch);
$headerArr = explode("\r\n", $header);
foreach ($headerArr as $item) {
    $value = explode(':', $item);
    if ($value[0] == 'Content-Range') {//通过分段,获取到文件大小
        $fileSize = explode('/',$value[1])[1];//文件大小
        break;
    }
}
var_dump($fileSize);
//开启多线程下载
$mh = curl_multi_init();
$count = 5;//n个线程
$handle = [];//n线程数组
$data = [];//数据分段数组
$fileData = ceil($fileSize / $count);
for ($i = 0; $i < $count; $i++) {
    $ch = curl_init();
 
    //判断是否读取数量大于剩余数量
    if ($fileData > ($fileSize-($i * $fileData))) {
        $headerData = [
            "Range:bytes=" . $i * $fileData . "-" . ($fileSize)
        ];
    }else{
        $headerData = [
            "Range:bytes=" . $i * $fileData . "-" .(($i+1)*$fileData)
        ];
    }
    echo PHP_EOL;
    curl_setopt($ch, CURLOPT_URL, $filePath);
    curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1); // return don't print
    curl_setopt($ch, CURLOPT_TIMEOUT, 0);
    curl_setopt($ch, CURLOPT_USERAGENT, 'Mozilla/4.0 (compatible; MSIE 5.01; Windows NT 5.0)');
    curl_setopt($ch, CURLOPT_FOLLOWLOCATION, 1); // 302 redirect
    curl_setopt($ch, CURLOPT_HTTPHEADER, $headerData);
    curl_setopt($ch, CURLOPT_MAXREDIRS, 7);
    curl_multi_add_handle($mh, $ch); // 把 curl resource 放进 multi curl handler 里
    $handle[$i] = $ch;
}
$active = null;
 
do {
    //同时执行多线程,直到全部完成或超时
    $mrc = curl_multi_exec($mh, $active);
} while ($active);
 
for ($i = 0; $i < $count; $i++) {
    $data[$i] = curl_multi_getcontent($handle[$i]);
    curl_multi_remove_handle($mh, $handle[$i]);
}
curl_multi_close($mh);
$file = implode('',$data);//组合成一个文件
$arr = explode('x',$file);
var_dump($data);
var_dump(count($arr));
var_dump($arr[count($arr)-2]);
//测试文件是否正确
```
