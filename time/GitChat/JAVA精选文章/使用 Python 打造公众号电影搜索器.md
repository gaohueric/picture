### 内容提要

内容提要：

- Uwsig 和 Gunicorn 哪个更好，最大的区别是什么？
- Supervisor 开机启动项目具体要如何配置？
- 文章中提到 Ubuntu 的定时爬取，那 Centos7 的定时爬取有什么推荐？
- 请问 Windows 下能实现吗，配置有些什么区别？
- 有推荐的开源或能分享出来的爬虫系统吗，有 Scrapy 框架的吗？
- 学习 Docker 中想用这种方式部署，你对这方面有什么嘱咐，适不适合生产环境？
- 微信朋友圈或者微信公众号里面的内容爬取，有什么比较好的方法吗？
- 存储能用 MongoDB 吗？
- 爬虫爬取数据的过程中如何避免爬虫被封？
- 其实网上好多服务比如心书，八爪鱼这些能生成自己的盆友圈所有文章，那他们是怎么实现的，你是否可以研究分享下？
- 有没有讲的比较好的 Django 教程博客推荐？

------

**问：Uwsig 和 Gunicorn 哪个更好，最大的区别是什么？**

**答：** Uwsgi 和 Gunicorn 在 Python Web 项目部署上都有很好的效果。具体选择哪一个还是要看个人的熟悉程度。相对来说，Gunicorn 配置简单、运行方便，但是其只适用于在 Unix 类操作系统上部署 Python 应用，而 Uwsgi 则配置丰富，且支持多种操作系统和多种编程语言。

------

**问：Supervisor 开机启动项目具体要如何配置？**

**答：**这个属于操作系统层面的，Supervisor 开机启动和其他的软件或服务开机启动一样，在操作系统里面进行配置即可，比如在 Ubuntu 上可以通过在/etc/rc.local 文件中添加服务的启动命令。Supervisor 启动之后，Supervisor 下的应用项目都会自动启动。

------

**问：文章中提到 Ubuntu 的定时爬取，那 Centos7 的定时爬取有什么推荐？**

**答：**文章中提到的定时采集使用的是 Crontab，这个功能在 Linux 上基本上是可用的，所以在 Centos 上也可以使用 Crontab 进行定时采集任务操作。

------

**问：请问 Windows 下能实现吗，配置有些什么区别？**

**答：** 如果说是 Windows 下 Django 的部署，那么可以使用 Apache+Uwsgi 的方案；如果指的是定时采集任务，Windows 下有图形化界面操作的定时任务设置。在计算机管理里面可以进行配置。

------

**问：有推荐的开源或能分享出来的爬虫系统吗，有 Scrapy 框架的吗？**

**答：**个人使用爬虫框架得少，多是自己构造 HTTP 请求、数据解析、数据存储和并发处理等。个人建议，先了解爬虫每个部分的机制、原理和过程，再上手使用爬虫框架，这样更加容易理解框架的运行。

------

**问：学习 Docker 中想用这种方式部署，你对这方面有什么嘱咐，适不适合生产环境？**

**答：** Docker 容器适合于批量化部署和上线，利用镜像市场的别人写好的配置镜像，可以比自己搭建服务器更加快捷和省事，不用操心各种依赖和组件和服务的问题；但是从个人少得可怜的 Doker 使用经验来看，没有直接操作服务器进行后期管理顺手。

------

**问：微信朋友圈或者微信公众号里面的内容爬取，有什么比较好的方法吗？**

**答：**微信朋友圈的内容采集，没有太好的方法，个人了解在安卓机器上通过 Xposed 框架可以进行部分的采集。微信公众号文章采集的话，可以通过搜狗提供的微信搜索功能对某个公众号进行采集。

------

**问：存储能用 MongoDB 吗？**

**答：**可以使用 MongoDB 数据库。其实在数据采集过程中，因为数据多是不规整的数据，非关系型的数据库更加方便于爬虫数据的存储。如果是在 Django 中使用 MongoDB，也有第三方的模块提供 Django 与 MongoDB 之间的衔接。

------

**问：爬虫爬取数据的过程中如何避免爬虫被封？**

**答：**避免数据采集的时候爬虫被封，可以从以下方面进行优化：增加请求之间的时间间隔；使用代理 IP；使用完善的 Header 请求头。

------

**问：其实网上好多服务比如心书，八爪鱼这些能生成自己的盆友圈所有文章，那他们是怎么实现的，你是否可以研究分享下？**

**答：**这种情况也是添加了个人微信为好友，且好友能够看到所有朋友圈信息才能获取到朋友圈的信息。同时也不能判断其是通过工具还是人工进行的内容提取。

------

**问：有没有讲的比较好的 Django 教程博客推荐？**

**答：** Django 的教程个人首推 Django 的官方文档，讲解得非常详细和易懂。其次推荐自强学堂的Django 教程。



### 文章实录

查看本场Chat

今天，我们来介绍一下如何使用 Python 构建一个个人微信公众号的电影搜索功能。本篇文章将会涉及到：

- Python 网络爬虫的编写；
- Django Web 项目的创建和部署；
- Django 模型的基本使用；
- 微信公众号开发者模式的接入和使用；

实践完成本篇文章中的所有内容，需要大家拥有以下资源：

- 一台 Linux 服务器（本文使用 Ubuntu 14）；
- 一个已经备案的域名；
- 一个个人微信公众号；

如果没有以备案域名和个人微信公众号，可以申请微信公众平台测试账号进行使用，其流程大同小异，申请地址为：

> <http://mp.weixin.qq.com/debug/cgi-bin/sandbox?t=sandbox/login>

下面我们开始进入正文。

### 1. 创建和部署 Django 应用

在这个个人公众号电影搜索器中，一个可靠的 Web 服务是连接电影资源数据和对接微信公众号的关键，电影资源爬取采集之后需要存储到数据库中，微信公众号消息的接收和回复也依赖于 Web 应用提供的服务。所以我们首先需要创建一个 Web 应用，在 Python 中 Web 框架有很多，在此我们选择功能全面且强大的 Django 1.10。没有安装 Django 的同学使用 pip 命令对 Django 模块进行安装：

```
pip install django==1.10

```

#### 1.1 创建一个 Django 项目

我们的服务器上当前目录如下图所示：

![img](https://upload-images.jianshu.io/upload_images/38544-24be1cb601852c31.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在此目录下使用 Django 的 django-admin 创建一个 Django 项目：

```
django-admin startproject wxmovie

```

![img](https://upload-images.jianshu.io/upload_images/38544-4c343cface9e5747.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这样，我们的文件夹目录下多了一个名为 wxmovie 的文件夹，里面包含一个 manage.py 文件以及一个 wxmovie 文件夹：

![img](https://upload-images.jianshu.io/upload_images/38544-18a551d8363e3fdd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 1.2 创建一个 Django 应用

Django 项目创建好之后，我们进入到项目路径下，使用其 manage.py 文件继续创建一个 Django 应用：

```
python3 manage.py startapp movie

```

![img](https://upload-images.jianshu.io/upload_images/38544-4e48f8136a30a110.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这时候 wxmovie 项目下就已经多出了一个名为 movie 的文件夹，里面是 movie 应用的所有文件：

![img](https://upload-images.jianshu.io/upload_images/38544-95ac54070a1e79f0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 1.3 配置 Django 项目

在 Django 项目——wxmovie 和它的应用 movie 创建好之后，我们来对这个项目进行一些基本的配置。打开 /wxmovie/wxmovie/ 目录下的 settings.py 文件。

将应用 movie 添加到 wxmovie 项目中：

```
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'movie',
]

```

在项目根路径下创建名为 template 的文件夹作为 Django 模板目录，并将此路径添加到 TEMPLATES 中：

```
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [
            os.path.join(os.path.split(os.path.dirname(__file__))[0],'template'),
        ],
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
            ],
        },
    },
]

```

修改项目的数据库配置为 MySQL：

```
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': '****',
        'USER': '****',
        'PASSWORD': '***',
        'HOST': '',
        'PORT': '3306',
    }
}

```

修改项目的时区的语言配置：

```
LANGUAGE_CODE = 'zh-hans'

TIME_ZONE = 'Asia/Shanghai'

```

然后因为我们使用的 Python3，所以需要在 /wxmovie/wxmovie/ 目录下的`\__init__.py`文件中添加以下代码，使得我们在项目中能够使用 MySQL（pymysql 模块需要先行安装）：

```
import pymysql
pymysql.install_as_MySQLdb()

```

这样，我们的项目的基本配置就已经修改完成了。

#### 1.4 创建电影资源数据模型

在这里，我们创建一个电影资源的最小模型，因为我们接下来要采集的电影资源数据是以百度网盘的资源为主，所以其中主要包含以下字段：

- 电影资源的名称
- 电影资源的链接
- 电影资源的密码

在 /wxmovie/movie/ 目录下的 models.py 文件中，定义一个名为 Movie 的模型类：

```
class Movie(models.Model):
    title = models.CharField(verbose_name='资源标题',max_length=100,null=True,blank=True)
    link = models.CharField(verbose_name='资源链接',max_length=255,null=True,blank=True)
    passwd = models.CharField(verbose_name='资源密码',max_length=50,null=True,blank=True)
    insert_date = models.DateTimeField(verbose_name='写入时间',auto_now_add=True)

    def __str__(self):
        return self.title

    class Meta:
        verbose_name = '电影资源'
        verbose_name_plural = verbose_name
        ordering = ['-insert_date']

```

完成定义之后，我们在终端界面对刚刚定义的模型进行迁移的生成和执行。

创建模型迁移：

```
python3 manage.py makemigrations movie

```

执行模型的迁移：

```
python3 manage.py migrate

```

在命令行中执行完这两条命令，我们的模型对应的数据库就完成了创建：

![img](https://upload-images.jianshu.io/upload_images/38544-dd685c11e70cd290.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在模型创建好之后，我们就可以编写网络爬虫采集电影资源数据并将数据存储到项目中的数据库了。

#### 1.5 创建项目超级用户

为了便捷的使用 Django 提供的功能强大的 admin 后台，我们在此创建一个超级用户用户后台的登陆。

在命令行终端使用如下命令进入项目目录：

```
cd ./wxmovie

```

使用 manage.py 工具的 createsuperuser 命令创建超级用户：

```
python3 manage.py createsuperuser

```

然后输入用户名和密码就完成了超级用户的创建：

![img](https://upload-images.jianshu.io/upload_images/38544-b6b566cfd8bced4d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 1.6 部署 Django 项目

在完成上述步骤之后，我们可以先将 wxmovie 这个 Django 项目部署到服务器上，以便后续的操作。在此我们使用的是 nginx+uwsgi 的方式进行部署。同时使用 supervisor（Python2 环境下）进行进程守护。

本小节需要使用到的模块和软件包有：

- nginx：可通过 apt-get 命令进行安装
- uwsgi：可通过 pip3 命令进行安装
- supervisor：可通过 pip 命令进行安装（Python2 环境下）

##### **1.6.1 配置 uwsgi**

在 wxmovie 项目目录下新建一个名为 wxmovie_uwsgi.ini 的配置文件，打开并将以下内容写入进去：

```
# wxmovie_uwsgi.ini file
[uwsgi]

# Django-related settings

socket = :8004

# the base directory (full path)
chdir           = /home/ubuntu/wxmovie

# Django s wsgi file
module          = wxmovie.wsgi

# process-related settings
# master
master          = true

# maximum number of worker processes
processes       = 1
threads = 1

# ... with appropriate permissions - may be needed
# chmod-socket    = 664
# clear environment on exit
vacuum          = true
python-autoreload = 1

```

同时再新建一个名为 uwsgi_params 的文本文件，将以下内容写入进去：

```
uwsgi_param  QUERY_STRING       $query_string;
uwsgi_param  REQUEST_METHOD     $request_method;
uwsgi_param  CONTENT_TYPE       $content_type;
uwsgi_param  CONTENT_LENGTH     $content_length;

uwsgi_param  REQUEST_URI        $request_uri;
uwsgi_param  PATH_INFO          $document_uri;
uwsgi_param  DOCUMENT_ROOT      $document_root;
uwsgi_param  SERVER_PROTOCOL    $server_protocol;
uwsgi_param  REQUEST_SCHEME     $scheme;
uwsgi_param  HTTPS              $https if_not_empty;

uwsgi_param  REMOTE_ADDR        $remote_addr;
uwsgi_param  REMOTE_PORT        $remote_port;
uwsgi_param  SERVER_PORT        $server_port;
uwsgi_param  SERVER_NAME        $server_name;

```

现在，在我们的项目根路径下，结构如下图所示：

![img](https://upload-images.jianshu.io/upload_images/38544-afa0e2ab8f6c6da8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##### **1.6.2 配置 Nginx**

配置完 uwsgi 后，我们继续配置 Nginx。首先在 wxmovie 项目根路径下创建一个名为 wxmovie_nginx.conf 的配置文件，将以下内容写入其中：

```
 server {
    listen         80; 
    server_name    wxmovie.huabandata.com;
    charset UTF-8;
    access_log      /var/log/nginx/myweb_access.log;
    error_log       /var/log/nginx/myweb_error.log;

    client_max_body_size 75M;

    location / { 
        include /home/ubuntu/wxmovie/uwsgi_params;
        uwsgi_pass 127.0.0.1:8004;
        uwsgi_read_timeout 60;
    }   
    location /static {
        expires 30d;
        autoindex on; 
        add_header Cache-Control private;
        alias /home/ubuntu/wxmovie/static;
     }
     location /media  {
        alias /home/ubuntu/wxmovie/media;
    }
 }

```

接着为 wxmovie_nginx.conf 创建一个软链接连接到 /etc/nginx/sites-enables/ 目录：

```
sudo ln -s /home/ubuntu/wxmovie/wxmovie_nginx.conf /etc/nginx/sites-enabled/

```

执行完命令之后，/etc/nginx/sites-enabled/ 目录下将会多出一个软链接文件：

![img](https://upload-images.jianshu.io/upload_images/38544-19b7d378afc0bf09.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##### **1.6.3 使用 supervisor 管理 uwsgi 进程**

在完成 nginx 和 uwsgi 的配置后，我们的网站就可以运行起来了，但是为了方便进行监控和管理，我们使用 supervisor 对 uwsgi 的进程进行守护。

supervisor 安装完成之后，默认是没有配置文件的，但是我们也不需要向 uwsgi 一样自己创建配置文件。使用 `echo_supervisord_conf` 命令就可以生成一个 supervisor 的配置文件：

```
echo_supervisord_conf > /home/ubuntu/supervisord.conf

```

supervisor 配置文件中必须包含一个或多个 program 的配置信息，以便 supervisor 知道应该启动和控制哪些程序，但是默认生成的 supervisor 配置文件中并没有 program 的内容，我们需要按需自行添加。

打开 supervisord.conf 文件，在文件的最后添加以下语句并保存：

```
[program:wxmovie]
command=uwsgi --ini /home/ubuntu/wxmovie/wxmovie_uwsgi.ini
directory=/home/ubuntu/wxmovie
startsecs=0
stopwaitsecs=0
autostart=true
autorestart=true

```

其中：

- 第一行表示设置一个名为 wxmovie 的 program，我们可以通过这个名称来控制这个 program；
- 第二行表示程序运行的命令，在这里我们设置的是 uwsgi 通过 uwsgi 配置文件启动 django 项目的命令；
- 第三行表示程序的目录；
- 第四行表示程序在启动后需要保持的秒数，设置为 0 表示不需要保持；
- 第五行表示在指示程序停止运行后，需要等待的秒数，设置为 0 表示不需要等待；
- 第六行表示当 supervisord 启动时，该程序自动启动；
- 第七行表示当 superisord 为运行状态时，程序自动重启；

在编写好 supervisord.cof 配置文件后，我们就可以通过 supervisor 来管理通过 uwsgi 启动的 Django 项目了。

首先，通过 supervisord 命令启动 supervisor：

```
sudo supervisord

```

然后再通过 supervisorctl 调用配置文件启动程序：

```
sudo supervisorctl -c /home/ubuntu/supervisord.conf

```

控制台显示如下图所示的界面，说明启动成功：

![img](https://upload-images.jianshu.io/upload_images/38544-b2fa712f5dbfb4bf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 1.7 解析域名

在完成项目的启动之后，我们还暂时不能访问我们的网站，因为我们使用了子域名 wxmovie.huabandata.com 对网站进行了绑定，现在需要在域名解析管理中添加 wxmovie 的解析。

打开域名解析页面（各家服务大同小异，这里使用的是腾讯云的域名管理解析），添加一条A记录的解析：

![img](https://upload-images.jianshu.io/upload_images/38544-54402a31c5269b00.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

现在访问 wxmovie.huabandata.com，就已经可以访问了：

![img](https://upload-images.jianshu.io/upload_images/38544-71faff858d6a50a8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

访问后台页面：![img](https://upload-images.jianshu.io/upload_images/38544-ebd40b190209df8f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

登陆进去：

![img](https://upload-images.jianshu.io/upload_images/38544-bbd6480e35fb5b5c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

发现我们创建的模型没有添加进去，我们在 /wxmovie/movie/admin.py 文件中进行添加，将以下代码写入其中：

```
from django.contrib import admin
from .models import Movie
# Register your models here.
class MovieAdmin(admin.ModelAdmin):
    list_display = ['title','link','passwd']

admin.site.register(Movie,MovieAdmin)

```

刷新一下后台页面，可以发现电影资源模型已经添加到了后台中：

![img](https://upload-images.jianshu.io/upload_images/38544-9cdb2f4b320d957c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这样，我们的 Django 项目就已经部署好了，接下来，可以编写网络爬虫将采集的电影资源存储到电影资源模型中。

### 2. 编写电影资源爬虫

#### 2.1 分析和采集页面数据

我们所需要采集的电影资源来自于一个百度网盘分享站点—— http://www.friok.com/，其中提供了电影资源的下载：

![img](https://upload-images.jianshu.io/upload_images/38544-7920cb943aeff3b0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

每个电影都有一个 id 作为唯一标识：

![img](https://upload-images.jianshu.io/upload_images/38544-36425aaf1d4fcc62.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

下载页面亦是通过这个 id 进行索引，同时下载页面提供了百度网盘的链接和提取密码：

![img](https://upload-images.jianshu.io/upload_images/38544-2a610b20f8a70ebf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

所以我们通过编写一个简单的爬虫就可以将电影资源的数据采集下来。在 /wxmovie/movie/ 目录下新建一个名为 utils.py 的文件，将以下代码写入进去：

```
# coding:utf-8
'''
    采集百度网盘电影资源
    @作者：州的先生
'''

import requests
from bs4 import BeautifulSoup

header = {
    'Accept':'text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8',
    'Accept-Encoding':'gzip, deflate, sdch',
    'Accept-Language':'zh-CN,zh;q=0.8',
    'User-Agent':'Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/53.0.2785.143 Safari/537.36',
    'Connection':'keep-alive',
    'Host':'www.friok.com',
    'Referer':'http://www.friok.com/'
}

#获取电影在线资源
def get_movie_from_friok(page = 1):
    host = 'http://www.friok.com/category/dyzy/page/{0}/'.format(page)
    try:
        wb = requests.get(host,headers=header).content
        soup = BeautifulSoup(wb,'html5lib')
    except BaseException as e:
        print("访问出错：",e)
    #获取电影编号
    nums_list = []
    movie_nums = soup.select("h2 > a")
    for m in movie_nums:
        m = m.get("href")[21:-5]
        nums_list.append(m)
    #从电影下载页中获取百度网盘链接
    video_list = []
    for n in nums_list:
        down_url = 'http://www.friok.com/download.php?id='+n
        try:
            down_data = requests.get(down_url,headers=header).content
        except BaseException as e:
            continue
        dsoup = BeautifulSoup(down_data,'html5lib')
        name = dsoup.title.get_text()[:-4]
        link = dsoup.select_one('div.list > a').get('href')
        passwd = dsoup.select("div.desc > p")[1].get_text().split('：')[-1]
        print(name,link,passwd)

if __name__ == '__main__':
    get_movie_from_friok()

```

在这个函数中，我们首先引入了所需要的模块——requests 和 BeautifulSoup：

```
import requests
from bs4 import BeautifulSoup

```

然后定义了一个请求头：

```
header = {
    'Accept':'text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8',
    'Accept-Encoding':'gzip, deflate, sdch',
    'Accept-Language':'zh-CN,zh;q=0.8',
    'User-Agent':'Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/53.0.2785.143 Safari/537.36',
    'Connection':'keep-alive',
    'Host':'www.friok.com',
    'Referer':'http://www.friok.com/'
}

```

接着，我们定义了一个名为`get_movie_from_friok`的函数，在这个函数中，我们首先请求 friok 网站的电影列表页面，提取出页面中每一个电影的 id 并放入一个列表中，接着按电影 id 去请求电影资源的下载页面，从中提取出电影资源的名称、资源链接和资源提取密码，最后打印出来：

```
#获取电影在线资源
def get_movie_from_friok(page = 1):
    host = 'http://www.friok.com/category/dyzy/page/{0}/'.format(page)
    try:
        wb = requests.get(host,headers=header).content
        soup = BeautifulSoup(wb,'html5lib')
    except BaseException as e:
        print("访问出错：",e)
    #获取电影编号
    nums_list = []
    movie_nums = soup.select("h2 > a")
    for m in movie_nums:
        m = m.get("href")[21:-5]
        nums_list.append(m)
    #从电影下载页中获取百度网盘链接
    video_list = []
    for n in nums_list:
        down_url = 'http://www.friok.com/download.php?id='+n
        try:
            down_data = requests.get(down_url,headers=header).content
        except BaseException as e:
            continue
        dsoup = BeautifulSoup(down_data,'html5lib')
        name = dsoup.title.get_text()[:-4]
        link = dsoup.select_one('div.list > a').get('href')
        passwd = dsoup.select("div.desc > p")[1].get_text().split('：')[-1]
        print(name,link,passwd)

```

如果我们运行这个文件，会得到类似于下图所示的结果：

![img](https://upload-images.jianshu.io/upload_images/38544-d0cdd2e1103e2304.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 2.2 使用 Django 模型存储数据

这样的采集和提取结果我们还是能够接受的，但是直接将数据打印出来肯定是不行的，我们修改一下代码，让数据直接保存到数据库中，修改后的代码如下所示：

```
# coding:utf-8
'''
    采集百度网盘电影资源
    @作者：州的先生
'''

import requests
from bs4 import BeautifulSoup
import sys
import os
from django.core.wsgi import get_wsgi_application
sys.path.extend(['/home/ubuntu/wxmovie',])
os.environ.setdefault("DJANGO_SETTINGS_MODULE","wxmovie.settings")
application = get_wsgi_application()
import django
django.setup()
from movie.models import *

header = {
    'Accept':'text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8',
    'Accept-Encoding':'gzip, deflate, sdch',
    'Accept-Language':'zh-CN,zh;q=0.8',
    'User-Agent':'Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/53.0.2785.143 Safari/537.36',
    'Cookie':'yunsuo_session_verify=773fce65ac8baaa9b2fc339f5898ee53; Hm_lvt_37eca9226885a06726365a4e1ad490cd=1526944142,1527424628,1527604106; Hm_lpvt_37eca9226885a06726365a4e1ad490cd=1527634741',
    'Connection':'keep-alive',
    'Host':'www.friok.com',
    'Referer':'http://www.friok.com/'
}

#获取电影在线资源
def get_movie_from_friok(page = 1):
    host = 'http://www.friok.com/category/dyzy/page/{0}/'.format(page)
    try:
        wb = requests.get(host,params=header).content
        soup = BeautifulSoup(wb,'html5lib')
    except BaseException as e:
        print("访问出错：",e)
    #获取电影编号
    nums_list = []
    movie_nums = soup.select("h2 > a")
    for m in movie_nums:
        m = m.get("href")[21:-5]
        nums_list.append(m)
    #从电影下载页中获取百度网盘链接
    video_list = []
    for n in nums_list:
        down_url = 'http://www.friok.com/download.php?id='+n
        try:
            down_data = requests.get(down_url,headers=header).content
        except BaseException as e:
            continue
        dsoup = BeautifulSoup(down_data,'html5lib')
        name = dsoup.title.get_text()[:-4]
        try:
            link = dsoup.select_one('div.list > a').get('href')
            passwd = dsoup.select("div.desc > p")[1].get_text().split('：')[-1]
            result = Movie.objects.get_or_create(
                title = name,
                link = link,
                passwd = passwd
            )
            print('[*]完成数据写入：',name,link,passwd)
        except Exception as e:
            print('[-]数据写入错误',e)

if __name__ == '__main__':
    for i in range(1,139):
        print('当前第{}页'.format(i))
        get_movie_from_friok(page=i)

```

接着，我们在终端界面运行这个 Python 文件，提取出来的电影资源数据会直接写到数据库中：

![img](https://upload-images.jianshu.io/upload_images/38544-26a39ee785c38093.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

程序运行完成，我们在 admin 后台中可以发现，采集下来的电影资源数据已经存在于我们的数据库中：

![img](https://upload-images.jianshu.io/upload_images/38544-ec38e3a6d2132e43.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 2.3 定时运行爬虫

电影资源的数据刚刚已经采集完了，但是如果有更新怎么办，我们当然可以每天寻找各种电影资源，然后手动在 admin 后台中进行添加：

![img](https://upload-images.jianshu.io/upload_images/38544-7c9e13e7e7de6d61.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

但是这样太过于繁琐，我们可以在服务器上设置一个定时任务，让爬虫脚本每天定时运行。

因为我们使用的是 Ubuntu 的服务器，其系统自带了 crontab 定时任务，我们可以直接使用它。现在我们使用其来设置我们的爬虫的定时任务，对 crontab 使用不了解的同学可以参见：<http://linuxtools-rst.readthedocs.io/zh_CN/latest/tool/crontab.html>，很详细很基础。

在终端界面输入命令 `crontab -e` 对当前用户的 crontab 文件进行编辑，在文件内容最后添加如下代码：

```
30 16 * * * python3 /home/ubuntu/wxmovie/movie/utils.py >> /home/ubuntu/wxmovie/movie/cron.log

```

![img](https://upload-images.jianshu.io/upload_images/38544-094d469592199fb6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这句 crontab 表示每天的 16 点 30 分执行 /home/ubuntu/wxmovie/movie/utils.py 文件，并将日志写入到 /home/ubuntu/wxmovie/movie/cron.log 文件中。

我们保存并退出。然后使用 `crontab -l` 查看一下我们当前的 crontab 定时任务：

![img](https://upload-images.jianshu.io/upload_images/38544-7ca00e8b5a7d1122.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这样，我们的定时采集任务也搞定了，不用再每天手动更新数据。下面，我们开始将其接入到微信公众平台上。

### 3. 接入微信公众号开发模式

**特别提示：**本节涉及较多的微信开发中的知识和规范，微信公众平台开发发展至今说明文档已经很完善，如果有不懂的，基本上都可以从微信开发文档中找到相应的说明和解释，[链接地址](https://mp.weixin.qq.com/wiki?t=resource/res_main&id=mp1445241432)。本节仅对涉及到的概念点进行说明。

首先，我们得有一个微信公众号，在这里，我使用的是自己的微信公众号：州的先生。

#### 3.1 开启开发者模式

在默认情况下，微信公众号的开发者模式是关闭的。我们先来开启开发者模式。在微信公众号的后台页面点击左边侧栏的【开发】下的【基本配置】：

![img](https://upload-images.jianshu.io/upload_images/38544-f4151a6b834541d3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

会出现如下的提示：

![img](https://upload-images.jianshu.io/upload_images/38544-31af06864a2524dd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我们勾选同意并点击【成为开发者】按钮。点击之后，会出现基本的配置信息：

![img](https://upload-images.jianshu.io/upload_images/38544-5f2d1722fc266430.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 3.2 配置开发信息

接着我们对开发信息和服务器配置信息进行填写和配置，点击【修改配置】配置服务器信息：

![img](https://upload-images.jianshu.io/upload_images/38544-65acb34322641fa4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

其中，url 为与微信公众平台进行通信的接口地址，在此我们设置为 http://wxmovie.huabandata.com/wechat/；token 用来验证；为了便于演示消息加密方式我们直接选择明文模式。配置好之后点击【提交】按钮会提示我们 token 验证失败：

![img](https://upload-images.jianshu.io/upload_images/38544-bfaa97a93c24e00e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

为什么会这样呢？因为我们的服务器并没有做任何相应的配置。查看[微信公众平台开发文档](https://mp.weixin.qq.com/wiki?t=resource/res_main&id=mp1421135319)可知，微信公众平台开发的接入有三个步骤：

1. 填写服务器配置
2. 验证服务器地址的有效性
3. 根据微信公众平台的接口文档实现业务逻辑。

在上面，我们只完成了第一步填写服务器的配置信息，并对服务器地址的有效性进行验证。接下来我们对消息进行验证。

#### 3.3 验证服务器地址的有效性

根据微信公众平台开发文档的介绍：

![img](https://upload-images.jianshu.io/upload_images/38544-4a5bf2ce4e544fc0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我们需要对微信公众平台发送的get请求按照一定规则进行响应。下面我们在 Django 项目中创建一个视图函数对微信公众平台的请求进行响应，打开 /wxmovie/movie/views.py 文件，在其中写入以下代码：

```
from django.shortcuts import render,HttpResponse
from django.views.decorators.csrf import csrf_exempt
import hashlib

# 验证服务器地址
TOKEN = '配置页面设置的token'

# 微信服务器验证
def checkSignatrue(request):
    signature = request.GET.get("signature")
    timestamp = request.GET.get("timestamp")
    nonce = request.GET.get("nonce")
    echoStr = request.GET.get("echostr")
    #token为微信验证TOKEN
    token = TOKEN
    tmpList = [token,timestamp,nonce]
    tmpList.sort()
    tmpStr = "%s%s%s"%tuple(tmpList)
    tmpStr = hashlib.sha1(tmpStr.encode('utf-8')).hexdigest()
    if tmpStr == signature:
        return echoStr
    else:
        return None

# 处理微信请求
@csrf_exempt
def handleRequest(request):
    if request.method == "GET":
        response = HttpResponse(checkSignatrue(request),content_type='text/plain')
        return response
    # 接收和响应数据
    elif request.method == "POST":
        pass

```

在这里，我们创建了两个视图函数，其中 checkSignatrue() 用于解析获取微信平台发送来的请求，对 tokene 、时间戳 timestamp、随机数 nonce、随机字符串 echostr 按照微信公众平台要求的方式进行拼接和加密，并将加密的结果与微信加密签名 signature 进行对比，如果相等，那么确认请求来源于微信。

因为微信验证请求使用的是 GET 请求，而普通用户向公众号发送消息时，微信平台会以 POST 请求的形式发送 XML 数据到我们填写的服务器 URL 上，所以函数 handleRequest() 用来对请求进行识别和分类。

创建好视图函数之后，我们设置一个路由对视图函数进行映射，打开 /wxmovie/wxmovie/urls.py 文件，在 urlpatterns 列表中添加一条 URL 映射记录。添加后的 urls.py 文件的内容如下代码所示：

```
from django.conf.urls import url
from django.contrib import admin
from movie.views import handleRequest

urlpatterns = [
    url(r'^admin/', admin.site.urls),
    url(r'^wechat/',handleRequest),
]

```

再次回到微信公众号页面，重新点击提交按钮，就会提示我们已经验证成功了：

![img](https://upload-images.jianshu.io/upload_images/38544-538c655df439e544.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这样我们就完成了微信公众平台开发模式的接入：

![img](https://upload-images.jianshu.io/upload_images/38544-47f87c094b4a51d1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

但是现在我们还不能启用，因为我们的服务器并没有对用户发送的消息进行任何处理。现在我们来对普通用户发送的文本消息进行接收并进行处理。

#### 3.4 处理用户发送的文本消息

##### **3.4.1 接收用户发送的消息**

根据微信开发文档关于[消息管理](https://mp.weixin.qq.com/wiki?t=resource/res_main&id=mp1421140453)的说明，微信服务器通过 POST 请求将消息的 XML 数据包发送到配置的服务器 URL 上，同时文本消息的XML结构为：

```
<xml>  
<ToUserName>< ![CDATA[toUser] ]></ToUserName>  
<FromUserName>< ![CDATA[fromUser] ]></FromUserName>  
<CreateTime>1348831860</CreateTime>  
<MsgType>< ![CDATA[text] ]></MsgType>  
<Content>< ![CDATA[this is a test] ]></Content>  
<MsgId>1234567890123456</MsgId>  
</xml>

```

![img](https://upload-images.jianshu.io/upload_images/38544-93ad72ad5d3ad811.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

从中我们可以提取出消息的发送者、消息的类型和内容。 因为微信服务器发送而来的是 XML 格式的数据包，我们可以使用 Python 的内置 xml 处理模块——xml 对微信服务器发送而来的 XML 数据包进行解析，从中提取出我们需要的数据来。

##### **3.4.2 回复用户消息**

被动回复消息需要我们自己的服务器在收到微信服务器发送 POST 请求后 5 秒内进行响应和回复，看看微信官方文档中的说明：

![img](https://upload-images.jianshu.io/upload_images/38544-4931105b5852f912.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

同时微信提供了 6 种消息类型的回复，分别是：

- 回复文本消息；
- 回复图片消息；
- 回复语言消息；
- 回复视频消息；
- 回复音乐消息；
- 回复图文消息；

所有的回复消息类型如同我们接收的消息一样，都是 XML 结构的数据。在此出于实用性的考虑，我们使用图文消息作为有电影资源搜索结果的回复消息类型，文本消息作为没有电影资源搜索结果的回复消息类型。来看看这两种消息类型的 XML 结构：

**文本消息**

```
<xml> 
<ToUserName>< ![CDATA[toUser] ]></ToUserName> 
<FromUserName>< ![CDATA[fromUser] ]></FromUserName> 
<CreateTime>12345678</CreateTime> 
<MsgType>< ![CDATA[text] ]></MsgType> 
<Content>< ![CDATA[你好] ]></Content> 
</xml>

```

其中：

- ToUserName 表示接收方帐号（收到的 OpenID）
- FromUserName 表示开发者微信号
- CreateTime 表示消息创建时间 （整型）
- MsgType 表示 text
- Content 表示回复的消息内容

**图文消息**

```
<xml>
<ToUserName>< ![CDATA[toUser] ]></ToUserName>
<FromUserName>< ![CDATA[fromUser] ]></FromUserName>
<CreateTime>12345678</CreateTime>
<MsgType>< ![CDATA[image] ]></MsgType>
<Image><MediaId>< ![CDATA[media_id] ]></MediaId></Image>
</xml>

```

其中：

- ToUserName 表示接收方帐号（收到的 OpenID）
- FromUserName 表示开发者微信号
- CreateTime 表示消息创建时间 （整型）
- MsgType 表示 news
- ArticleCount 表示图文消息个数，限制为 8 条以内
- Articles 表示多条图文消息信息，默认第一个 item 为大图，注意，如果图文数超过 8，则将会无响应
- Title 表示图文消息标题
- Description 表示图文消息描述
- PicUrl 表示图片链接，支持 JPG、PNG 格式，较好的效果为大图 `360*200`，小图 `200*200`
- Url 表示点击图文消息跳转链接

基于此，我们可以构造 XML 形式的搜索结果作为响应返回给用户。

##### **3.4.3 在视图函数中实现用户消息的接收和回复**

通过前面的介绍，我们知道的用户发送的消息通过 XML 形式的数据包传递给我们，我们回复给用户的消息也需要以 XML 的形式返回给微信服务器。现在我们就扩展视图函数 handleRequest() 的 POST 请求部分，接收用户发来的消息，并将消息内容解析出来，进行判断。

首先来看看完整的代码（/wxmovie/movie/views.py）：

```
from django.shortcuts import render,HttpResponse
from django.views.decorators.csrf import csrf_exempt
from django.utils.encoding import smart_str
import hashlib
import xml.etree.ElementTree as ET
import time
from movie.models import Movie
'''
    微信公众号模块
'''

# 验证服务器地址
TOKEN = '填写的token'

# 微信服务器验证
def checkSignatrue(request):
    signature = request.GET.get("signature")
    timestamp = request.GET.get("timestamp")
    nonce = request.GET.get("nonce")
    echoStr = request.GET.get("echostr")
    #token为微信验证TOKEN
    token = TOKEN
    tmpList = [token,timestamp,nonce]
    tmpList.sort()
    tmpStr = "%s%s%s"%tuple(tmpList)
    tmpStr = hashlib.sha1(tmpStr.encode('utf-8')).hexdigest()
    if tmpStr == signature:
        return echoStr
    else:
        return None

# 处理微信请求
@csrf_exempt
def handleRequest(request):
    if request.method == "GET":
        response = HttpResponse(checkSignatrue(request),content_type='text/plain')
        return response
    # 接收和响应数据
    elif request.method == "POST":
        # 获取POST数据
        requestMsg = smart_str(request.body)
        # 从POST数据中读取信息
        msg = ET.fromstring(requestMsg)
        toUsername = msg.find("ToUserName").text
        fromUsername = msg.find("FromUserName").text
        msgType = msg.find("MsgType").text
        # 判断消息类型
        if msgType == 'text':
            text = msg.find("Content").text
            try:
                response = textResponse(request, fromUsername, toUsername, text)
                return response
            except:
                return HttpResponse('success')
        else:
            return HttpResponse('success')
    else:
        return None


# 接收文本消息的响应
def textResponse(request, fromUser, toUser, text):
    if text[:2] == '电影':
        data = Movie.objects.filter(title__icontains=str(text[2:]))
        count = data.count()
        if count == 0:
            content = {
                "fromUser": fromUser,
                "toUser": toUser,
                "date": str(int(time.time()))
            }
            xmlMsg = render(request, 'nomovieResponse.xml',
                            context=content, content_type='application/xml')
            return xmlMsg
        else:
            if count > 8:
                result = data[:8]
                num = 8
            else:
                result = data
                num = count
            content = {
                "fromUser": fromUser,
                "toUser": toUser,
                "date": str(int(time.time())),
                "num": num,
                "result": result
            }
            xmlMsg = render(request, 'movieResponse.xml',
                            context=content, content_type='application/xml')
            return xmlMsg
    else:
        return HttpResponse('success')

```

对比之前 views.py 文件中的内容，我们首先引入了很多个模块：

```
from django.utils.encoding import smart_str
import xml.etree.ElementTree as ET
import time
from movie.models import Movie

```

其中：

- smart_str 模块用于智能地解码字符串；
- xml.etree.ElementTree 模块用于解析XML文档树；
- time 模块用于时间转换；
- Movie 模型用于数据查询；

然后在请求为 POST 的时候，使用 smart_str 函数转换请求的主体内容：

```
elif request.method == "POST":
        # 获取POST数据
        requestMsg = smart_str(request.body)

```

再对转换后的请求主体内容进行 XML 格式化：

```
# 从POST数据中读取信息
msg = ET.fromstring(requestMsg)

```

接着从 XML 格式化的请求内容中提取出开发者的微信号、用户的账号、消息类型：

```
toUsername = msg.find("ToUserName").text
fromUsername = msg.find("FromUserName").text
msgType = msg.find("MsgType").text

```

如果消息类型为文本消息，那么我们解析出消息的内容来，然后将请求，开发者账号、用户账号、消息内容作为参数传递给另一个自定义的函数 textResponse()：

```
        # 判断消息类型
        if msgType == 'text':
            text = msg.find("Content").text
            try:
                response = textResponse(request, fromUsername, toUsername, text)
                return response
            except:
                return HttpResponse('success')
        else:
            return HttpResponse('success')

```

在 textResponse() 中，我们对用户消息内容的前两个字符进行判断（因为使用的是个人正在使用的公众号作为演示，为了不与其他消息相冲突，在此设置电影搜索需要在搜索词前加上“电影”二字，比如搜索“星球大战”，那么发送的消息就应该为“电影星球大战”），如果前两个字为“电影”，那么将其后的内容作为搜索词放入Movie模型中进行搜索查询。接着统计查询结果的数量，如果数量为0，那么我们对名为 nomovieResponse.xml 的 XML 文件进行渲染并返回响应；否则，我们提取出查询结果数量和查询结果，对名为 movieResponse.xml 的XML文件进行渲染并返回响应：

```
if text[:2] == '电影':
        data = Movie.objects.filter(title__icontains=str(text[2:]))
        count = data.count()
        if count == 0:
            content = {
                "fromUser": fromUser,
                "toUser": toUser,
                "date": str(int(time.time()))
            }
            xmlMsg = render(request, 'nomovieResponse.xml',
                            context=content, content_type='application/xml')
            return xmlMsg
        else:
            if count > 8:
                result = data[:8]
                num = 8
            else:
                result = data
                num = count
            content = {
                "fromUser": fromUser,
                "toUser": toUser,
                "date": str(int(time.time())),
                "num": num,
                "result": result
            }
            xmlMsg = render(request, 'movieResponse.xml',
                            context=content, content_type='application/xml')
            return xmlMsg
    else:
        return HttpResponse('success')

```

上面，我们提到了两个 XML 文件，这两个文件都需要在 /wxmovie/template/ 路径下进行创建，现在我们来创建它们：

![img](https://upload-images.jianshu.io/upload_images/38544-c3233743266a6534.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

首先是 nomovieResponse.xml，这个 xml 文件用于回复文本消息，我们按照微信规定的文本消息 xml 数据包结构编写这个 xml 文件，将以下内容写入 nomovieResponse.xml 文件中：

```
<xml>
<ToUserName><![CDATA[{{ fromUser }}]]></ToUserName>
<FromUserName><![CDATA[{{ toUser }}]]></FromUserName>
<CreateTime>{{ date }}</CreateTime>
<MsgType><![CDATA[text]]></MsgType>
<Content><![CDATA[
    对不起，暂时没有查询到相关电影资源^ ^!——州小秘
    ]]></Content>
</xml>

```

写入的内容与微信官方规定的 xml 数据包结构是一致的，只不过我们通过 Django 的模板语法渲染了发送账号、接收者账号、创建时间。如果搜索结果为 0，那么就会返回这个 xml 数据包给微信服务器，用户则会接收到内容为“对不起，暂时没有查询到相关电影资源^ ^!——州小秘”的文本消息。接下来来看 movieResponse.xml。

movieResponse.xml 用于回复图文消息，图文消息的 xml 数据结构比文本消息的数据结构的设置项要多一些，但是也并不复杂：

```
<xml>
<ToUserName><![CDATA[{{ fromUser }}]]></ToUserName>
<FromUserName><![CDATA[{{ toUser }}]]></FromUserName>
<CreateTime>{{ date }}</CreateTime>
<MsgType><![CDATA[news]]></MsgType>
<ArticleCount>{{ num }}</ArticleCount>
<Articles>
    {% for item in result %}
<item>
<Title><![CDATA[{{item.passwd}}-{{item.title}}]]></Title>
<Description><![CDATA[{{ item.title }}]]></Description>
<PicUrl><![CDATA[http://www.iyaoxin.com/static/img/wechatmoviebg.jpg]]></PicUrl>
<Url><![CDATA[{{ item.link }}]]></Url>
</item>
    {% endfor %}
</Articles>
</xml>

```

这个 xml 的内容与微信规定的图文消息xml数据包的结构也没有差别，除了消息类型是肯定与文本消息不同之外，我们首先设置了图文消息的条数：

```
<ArticleCount>{{ num }}</ArticleCount>

```

然后用 Django 模板的 for 循环语法遍历电影搜索结果：

- 在标题 title 中显示电影资源名称和提取密码；
- 在描述 Description 中显示电影资源名称；
- 在图片 PicUrl 中显示一个网络图片；
- 在图文消息的链接地址 Url 中显示电影资源的链接：

```
<Articles>
    {% for item in result %}
<item>
<Title><![CDATA[{{item.passwd}}-{{item.title}}]]></Title>
<Description><![CDATA[{{ item.title }}]]></Description>
<PicUrl><![CDATA[http://www.iyaoxin.com/static/img/wechatmoviebg.jpg]]></PicUrl>
<Url><![CDATA[{{ item.link }}]]></Url>
</item>
    {% endfor %}
</Articles>

```

这样两个响应 XML 文档创建好了，我们的这个微信公众号电影搜索器也基本完成。

细心的同学应该会发现，我们在主逻辑之外的代码中都使用 HttpResponse() 返回了一个“success”字符串。因为根据微信的规定：

> 假如服务器无法保证在五秒内处理并回复，必须做出下述回复，这样微信服务器才不会对此作任何处理，并且不会发起重试（这种情况下，可以使用客服消息接口进行异步回复），否则，将出现严重的错误提示。详见下面说明：
>
> 1. 直接回复 success（推荐方式）
> 2. 直接回复空串（指字节长度为 0 的空字符串，而不是 XML 结构体中 content 字段的内容为空）
>
> 一旦遇到以下情况，微信都会在公众号会话中，向用户下发系统提示“该公众号暂时无法提供服务，请稍后再试”：
>
> 1. 开发者在 5 秒内未回复任何内容
> 2. 开发者回复了异常数据，比如JSON数据等

所以我们在其他的逻辑下都返回了“success”这个字符串，比如当消息类型不是 text 的时候、消息的前两个字符串不是“电影”的时候。

### 4. 测试实际效果

#### 4.1 测试效果

完成了上述所有的步骤，我们的公众号电影资源搜索器就完成了从 Django Web 网站的创建和部署、电影资源的采集、对接微信公众平台这三个方面。现在我们来测试一下这个电影资源搜索器的实际效果。

没有关注我的公众号的同学可以先关注：**州的先生**：

![img](https://upload-images.jianshu.io/upload_images/38544-4533cdfb63824e74.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我们进入微信公众号，这里使用的是微信的电脑版：

![img](https://upload-images.jianshu.io/upload_images/38544-b0d7a64e12ceff72.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

先随便发送一些消息：

![img](https://upload-images.jianshu.io/upload_images/38544-ac37d1cdb806320e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

并没有消息的回复，而且因为我们设置响应了“success”的字符串，也不会出现如下图这样的错误提示信息：

![img](https://upload-images.jianshu.io/upload_images/38544-37f073cb3ae8ce21.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

接着我们使用“电影”关键词进行电影资源搜索：

![img](https://upload-images.jianshu.io/upload_images/38544-29929b13bfe1af7d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

搜索今年春节巨火的红海行动，结果返回了一条图文消息，其中 5gg6 为提取密码。我们点击详情，会直接跳转到百度云的链接：

![img](https://upload-images.jianshu.io/upload_images/38544-26772748bc24c76e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

输入验证码，出现了百度网盘的资源：

![img](https://upload-images.jianshu.io/upload_images/38544-b0277fe9e973d0c8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

不过这个资源竟然是种子文件，很是尴尬。

我们再来搜索一个电影，上个月上映的《狂暴巨兽》：

![img](https://upload-images.jianshu.io/upload_images/38544-26d9f772332dfc31.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

因为我们采集的电影资源里面没有这个电影，所以返回了一个文本消息，提示没有资源。

这就是我们这个微信公众号电影资源搜索器的全部结构和内容了。

#### 4.2 思考与扩展

在这个项目中，我们还有很多可以优化的地方，比如：

- 如何检测电影资源链接失效；
- 如何展示不同类型的电影资源；

大家可以思考一下，看如何优化这个电影资源搜索器。

同时在本次构建的项目中，其主要分为了三个部分：

- 爬虫：获取数据；
- Django 应用：提供数据以及与微信公众平台进行交互的载体；
- 微信公众号：作为信息输入输出的载体。

基于此，其实我们可以将我们的电影资源搜索器改造成很多个应用。比如：

- 一个天气预报查询；
- 一个淘宝优惠券的查询；
- 一个聊天机器人；
- 一个图片 ORC 识别器；
- 一个语音识别器；
- 一个语言翻译器；

等等

### 