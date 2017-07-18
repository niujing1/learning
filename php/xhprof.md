#### php71 xhprof安装使用

1. 下载安装

```
cd ~/build
git clone https://github.com/yaoguais/phpng-xhprof.git
cd phpng-xhprof/
phpize
./configure --with-php-config=/usr/local/bin/php-config
make && make install
```

2. 添加配置文件

```
vi /usr/local/etc/php/7.1/conf.d/ext-xhprof.conf

[xhprof]
extension=/usr/local/Cellar/php71/7.1.6_18/lib/php/extensions/no-debug-non-zts-20160303/phpng_xhprof.so

Tips: 扩展的位置可以使用 echo $(php-config --extension-dir)来查看
在编译扩展完成后，也会提示扩展位置
```

3. 配置一个web目录

```
#错误日志和访问日志路径
error_log /usr/local/var/log/nginx/xhprof-error.log;
access_log /usr/local/var/log/nginx/xhprof-access.log;

server {
    listen 80;
    server_name xhprof.cc;

    location / {
       #xhrof项目所在目录
        root          /Users/nj/www/php/xhprof;
        index         index.php index.html index.htm;
    }

    location ~ \.php$ {
         root           /Users/nj/www/php/xhprof;
         fastcgi_pass   127.0.0.1:9000; 
         fastcgi_index  index.php;
        #fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
         fastcgi_param  SCRIPT_FILENAME $document_root$fastcgi_script_name;
         include        fastcgi_params;
    }
}

仅供参考~~~
```

4. 如果是php7及以上的版本，还需额外做一些事。。。

```
它不包含xhprof_lib.php和xhprof_runs.php文件，而这两个文件是xhprof运行的必须文件，所以呢、找一个5.6版本的借用下：

cd ~/build
git clone https://github.com/phacility/xhprof.git
文件在xhprof_lib/utils下，也可以mv|cp到任意你喜欢的地方、下边用到的时候可以找到就OK
```

5. 重启服务

```
1）php
brew services restart php71
2）nginx
sudo nginx -s reload
Mac下的重启、请使用自己的对应方式操作
```

6. 测试扩展

```
1）扩展测试
php -m | grep xhprof
或者
php -i | grep xhprof
2）如果看到xhprof的信息，说明安装成功
```

7. 配置host

```
sudo vi /etc/hosts

添加 127.0.0.1 xhprof.cc到自己的host文件
```

8. 测试文件

```
//这里是cd到自己配置文件对应的目录、index.php也可以是任意文件
cd ~/www/php/xhprof/
vi index.php

<?php
if (isset($_REQUEST['xhprof']) && $_REQUEST['xhprof'] = 1) {
    xhprof_enable(XHPROF_FLAGS_MEMORY | XHPROF_FLAGS_CPU);
    register_shutdown_function(function() {
        $xhprof_data = xhprof_disable();
        if (function_exists('fastcgi_finish_request')) {
            fastcgi_finish_request();
        }
// 注意这里 xhprof_lib|runs.php的位置是自己文件的实际位置，注意修改
        require_once "/Users/nj/build/xhprof/xhprof_lib/utils/xhprof_lib.php";
        require_once "/Users/nj/build/xhprof/xhprof_lib/utils/xhprof_runs.php";
        $xhprof_runs = new XHProfRuns_Default();
        $run_id = $xhprof_runs->save_run($xhprof_data, 'test');
        //直接输出方便查看，实际应用最好写到log文件中
        echo 'http://xhprof.cc/index.php?run=' . $run_id . '&source=test';
        //file_put_contents('/Users/nj/www/php/xhprof/xhprof.txt', 'http://xhprof.cc/index.php?run=' . $run_id . '&source=test'."\n");
    });
}

```

9. 访问测试文件

```
xhprof.cc/index.php?xhprof=1 #xhprof=1代表使用xhprof

看到输出内容：
http://xhprof.cc/index.php?run=596b2e0276752&source=test
证明安装成功、enjoy it ~~~
```

10. 是不是想要图形界面？

```
记得上边clone了xhprof 5.6的文件吧 ？借用了xhprof_runs.php 和 xhprof_lib.php

现在 可以在xhprof下新建文件夹lib
cp -R ~/build/xhprof/xhprof_lib lib/ 非必须
cp -R ~/build/xhprof/xhprof_html ./
修改nginx的配置文件，将root定位到xhprof_html文件夹下
root           /Users/nj/www/php/xhprof/xhprof_html;
重新启动nginx

vi /Users/nj/www/php/xhprof/xhprof_html

修改xhprof_html/index.php xhprof_lib文件夹的位置：(你自己的lib位置)
$GLOBALS['XHPROF_LIB_ROOT'] = dirname(dirname(__FILE__)) . "/lib/xhprof_lib";

访问xhprof.cc/xhprof.php?xhprof=1
打开xhprof.cc/index.php
可以看到xhprof的文件列表、点击进去，可以查看图形界面的详情
```

![xhprof](/Users/nj/Pictures/pic-work/xhprof.png)


^(*￣(oo)￣)^ ## 使用顺利

