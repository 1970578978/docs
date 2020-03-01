## ubuntu 16.04 server Selenium环境搭建

#### 一、搭建selenium服务端

1. 安装java环境

   selenium是java编写的，所以运行selenium需要java环境，我们需先在ubuntu中安装java环境，使用apt安装openjdk即可。

```
sudo apt install openjdk-8-jdk
```

> 注：需要java8及其以上的环境

2. 下载selenium

   打开 [官网链接](https://www.seleniumhq.org/download/) https://www.seleniumhq.org/download/找到Download version后面的链接就可以下载。找到下载好的selenium-server-standalone-3.141.59.jar包，使用命令：

   ```
   java -jar ./selenium-server-standalone-3.141.59.jar 
   ```

   就可以启动服务，可以打开http://ip:4444/wd/hub 看到出现页面，就说明正常启动。
   
3. 创建selenium服务：

   在/lib/systemd/system下新建selenium.service,并填入下面内容，注意路径的区别。

````
[Unit]
Description=selenium server
[Service]
type=forking
ExecStart=/usr/bin/java -jar /home/name/java/app/selenium-server-standalone-3.141.59.jar

[Install]
WantedBy=multi-user.target
````

​	修改配置文件后需要重新加载配置

```
sudo systemctl daemon-reload
```

​	运行以下命令就可以启动和关闭服务

```
sudo service selenium start
sudo service selenium stop
```

设置开机启动和关闭开机启动

```
sudo systemctl enable selenium.service
sudo systemctl disable selenium.service
```



#### 二、chrome的安装以及对应驱动的下载

1. 安装chrome

   运行以下命令来下载和安装谷歌浏览器

```
sudo apt-get install libxss1 libappindicator1 libindicator7			#安装依赖包
wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
sudo dpkg -i google-chrome*.deb
sudo apt-get install -f
```

​	安装完成后运行

```
google-chrome-stable -version
```

​	可以查看到谷歌浏览器的版本就证明安装成功了。

2. 安装chrome驱动——ChromeDriver

   打开 [chromeDriver下载地址](https://chromedriver.storage.googleapis.com/index.html) https://chromedriver.storage.googleapis.com/index.html 找到对应自己浏览器版本的chromeDriver包并下载下来。然后执行以下命令：

   ```
    unzip chromedriver_linux64.zip
   
   $ sudo mv chromedriver /usr/bin/chromedriver
   $ sudo chown root:root /usr/bin/chromedriver
   $ sudo chmod +x /usr/bin/chromedriver
   ```

   

#### 三、php代码无头运行

1. 安装php包

   完成以上步骤就已经完成了selenium服务的搭建，接下来我们可以通过composer来安装selenium对应的php包

```
composer require facebook/webdriver
```

​	可以参考[github库](https://github.com/facebook/php-webdriver)

2. 简单案例

```php
<?php
namespace Facebook\WebDriver;
use Facebook\WebDriver\Remote\DesiredCapabilities;
use Facebook\WebDriver\Remote\RemoteWebDriver;
use Facebook\WebDriver\Remote\WebDriverCapabilityType;
use Facebook\WebDriver\Chrome\ChromeOptions;
require_once('vendor/autoload.php');

$options = new ChromeOptions();											//设置浏览器无头启动
$options->addArguments(array(
  'headless',
));

//以手机的模式方式打开
$options->setExperimentalOption("mobileEmulation", array(
  "deviceName" =>"iPhone 6/7/8"
));

$capabilities = DesiredCapabilities::chrome();

$capabilities->setCapability(ChromeOptions::CAPABILITY, $options);

$driver = RemoteWebDriver::create(
    'http://192.168.122.225:4444/wd/hub',
    $capabilities,																		//以无头方式启动
    5000
);
```

​	因为selenium是在运行server版的服务器，所以必须以无头的方式去运行浏览器，不然会因无发启动浏览器而报错。

我们还可以通过代码设置窗口大小

```php
$size = new WebDriverDimension(1280, 900);
$driver->manage()->window()->setSize($size);
```

打开网页

```php
$driver->get("https://www.baidu.com");
```

截图

```php
$driver->takeScreenshot("test.png");
```

> 注：如果截图出现乱码，但是通过`$driver->getTitle()`之类的代码获取到的中文内容没有出现乱码则是字体的原因。安装字体步骤可查看附录。

运行js

```php
$js = <<<js
  var ele = document.getElementsByClassName("commentList");
  if(ele.length == 0){
    return false;
  }
  return true;
js;
$driver->executeScript($js);				//获取到js的返回值
```

查找元素，获取元素位置等等很多操作就不一一举例了。具体方法可以参考[官方文档](https://facebook.github.io/php-webdriver/latest/Facebook/WebDriver/JavaScriptExecutor.html)。



#### 四、附录

##### 1. 安装思源宋体

首先需要下载字体文件，网上有很多思源宋体的下载链接，这里附上[站长素材](http://font.chinaz.com/180517194500.htm)的下载地址。

在存放字体的文件夹中，创建一个文件夹用来专门存放自己的字体。

```
cd /usr/share/fonts/
sudo mkdir myfonts
```

然后把字体移动到刚才创建的目录下。

```
sudo mv  *.otf /usr/share/fonts/myfonts/
```

进入到自己的字体文夹目录运行以下命令就可以安装字体啦。

```shell
cd /usr/share/fonts/myfonts/
sudo mkfontscale
sudo mkfontdir
sudo fc-cache -fv
```

> 注：思源宋体亲测有效。理论上安装中文字体就可以。
>
> 如果提示：mkfontscale: command not found
>
> 运行：sudo apt-get install ttf-mscorefonts-installer
>
> 如果提示：fc-cache: command not found
>
> 运行：sudo apt-get install fontconfig