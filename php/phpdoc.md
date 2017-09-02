### phpDocumentor 安装使用

#### 下载安装

```
有两种方法安装phpDocument，一种是通过pear安装，另外一种是通过源码安装。

1. pear安装

pear channel-discover pear.phpdoc.org
pear install phpdoc/phpDocumentor


2. composer安装

{
    "require-dev": {
        "phpdocumentor/phpdocumentor": "2.*"
    }
}

```

#### 支持的常用标签

```
@abstract 申明变量/类/方法
@param定义函数或者方法的参数信息
@return定义函数或者方法的返回信息
@package定义归属的包的信息
@final指明这是一个最终的类、方法、属性，禁止派生、修改。
@todo指明应该改进或没有实现的地方
@var定义说明变量/属性。
@version定义版本信息
@copyright指明版权信息
@author 函数作者的名字和邮箱地址
@deprecate指明不推荐或者是废弃的信息
@const指明常量
@exclude指明当前的注释将不进行分析，不出现在文挡中
@throws指明此函数可能抛出的错误异常,极其发生的情况
@static 指明变量、类、函数是静态的
@ingore 用于在文档中忽略指定的关键字
```

#### 常用参数

```
--target (-t) 设置生成的doc存放的目录
--filename (-f) 设置要生成doc的文件
--directory (-d) 设置生成doc文件的目录
--extensions (-e) 解析的文件类型、默认php php3 phtml
--ignore (-i)  要忽略的文件或者目录
--template  使用的模板(xml、zend、clean、checkstyle、old-ocean
            responsive、responsive-twig、abstract、new-black)
```

#### 使用示例：

```
phpdoc run -d <SOURCE_DIRECTORY> -t <TARGET_DIRECTORY>
```

#### notice 

```
如果点击view source、却显示not found
可以修改 src/phpDocumentor/Plugin/Twig/Writer/Twig.php line:296 注释掉对filepart的编码转化
```
