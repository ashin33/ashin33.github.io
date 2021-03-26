---
layout: ZipArchive
title: php使用ZipArchive类,压缩文件
date: 2021-03-25 22:51:34
tags: [2021,代码]
categories: [代码, bug]
---

>今天工作中,需要从服务器打包个人的签约pdf文件回本地,从数据库拉出个人的身份证号码及对应的签约文件名,最后要实现将签约文件以身份证号命名,于是写了个小脚本拉一下;

>保存身份证及文件名的对应关系文件如下
![zipArchive_1](https://i.loli.net/2021/03/26/uY82Z6AxREbzmtU.png)


### 文件打包成zip

```php
//身份证及文件名一一对应的条件存储文件
$txt_file = './condition.txt';
//最后打包的zip包名
$zip_name = './res.zip';
//签约文件存储地址
$dir = '/Volumes/PHP/tax-manage/public/downloads/';

$zip = new ZipArchive();
/*
 * open方法第一个参数是压缩或解压的文件
 * open第二个参数,用ZipArchive::CREATE,若文件已存在,会继续添加,用ZipArchive::OVERWIRITE则会覆盖之前的zip包
 * 传ZipArchive::CREATE时,若zip包不存在会自动创建
 * 传ZipArchive::OVERWIRITE时,若zip包不存在,$zip->open会返回数字9,后续addFile会报错Invalid or uninitialized Zip object 
 * */
 if(file_exists($zip_name)){
    unlink($zip_name);
}
if ($zip->open($zip_name, ZipArchive::CREATE) == TRUE) {
    //打开条件文件,读取身份证及签约文件名
    $file_handler = fopen($txt_file, 'r');//以可读方式打开文件
    //fgets读取行
    while ($data = fgets($file_handler)) {
        $arr = explode(',', $data);
        $file_name = trim($arr[0]).'.pdf';//用身份证重命名
        $path = $dir.trim($arr[1]);//一定要trim,不然取出来的数据会有换行符
        //添加文件到zip包中,第二个参数传入用于重命名
        $zip->addFile($path.$arr[1], $file_name);
    }
    $zip->close();
    fclose($file_handler);
    echo 'finish';
}
```

##### 记一个bug，心酸

>第一次写完后,`fgets()`方法取出一行数据后,用`explode()`方法分割数据,得到的文件路径是带了换行符`\n`的,但是这茬给忽略了,导致所有数据只有最后一行处理成功,因为没有换行;太菜了,哈哈哈哈;还以为是后面的覆盖了前面的文件呢,跟旁边的小伙伴想了半天还,最后还是小伙伴发现了这个,给自己一巴掌;(痛苦面具);特此记录一次,再忘记剁手(捂脸)

> 如果ZipArchive类的addFile方法返回false,不要怀疑就是文件不存在或者路径不对,不要质疑(捂脸)

### 解压缩zip包

```php
$zip = new ZipArchive();
//解压,open方法不需要传第二个参数
if ($zip->open('res.zip') === TRUE)
{
    //将res.zip的内容解压到res文件夹中
    $zip->extractTo('res');
    $zip->close();//关闭处理的zip文件
}
```