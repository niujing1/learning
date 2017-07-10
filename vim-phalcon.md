#### vim下的.zep语法高亮支持

1. 下载zephir-parser项目

```
git clone https://github.com/phalcon/php-zephir-parser.git
```

2. 进入目录

```
cd php-zephir-parser
```

3. 编译扩展

```
1) phpize
2) ./configure --with-php-config=/usr/local/bin/php-config
3) make && make install
```

4. 添加扩展

```
第三步不出意外的话、编译成功之后，会给出扩展的路径
编辑php扩展配置文件
vi /usr/local/etx/php/7.1/extension/ext-phalcon-parser.ini

[Zephir Parser]
extention=/usr/local/Cellar/php71/7.1.6_18/lib/php/extensions/no-debug-non-zts-20160303/zephir_parser.so
```

5. 重启fpm、nginx

```
sudo pkill -9 fpm
sudo pkill -9 nginx
sudo nginx
brew services start php71
```

6. 测试

```
php -m | grep Zephir
```

7. 设置vim

```
在zep文件中、vim命令模式下执行
:set syn=phalcon
可以看到语法高亮显示了~~~
```

希望可以有所帮助~~
