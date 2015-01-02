ckan 布署
========================================

这边使用 nginx+uwsgi 示范。

1. 新增 production.ini 设置文件
--------------------------------
   .. code-block:: bash

      $ cp /etc/ckan/default/development.ini /etc/ckan/default/production.ini

2. 修改 production.ini
------------------------
   搜寻并修改/新增下列字串：

   .. code-block:: python

      [server:main]
      use = egg:Paste#http
      host = site.domain
      port = 80
      #...
      ckan.site_url = http://site.domain
      #...
      #add the following lines at the bottom
      [uwsgi]
      socket = /tmp/uwsgi.sock
      master = true
      chmod-socket = 666

3. 安装 nginx
----------------
   .. code-block:: bash

      $ sudo apt-get install nginx

4. nginx 服务器设置
----------------------
a. 新增 /etc/nginx/sites-available/ckan 文件，并编辑加入以下设置：

   .. code-block:: php

      proxy_cache_path /tmp/nginx_cache levels=1:2 keys_zone=cache:30m max_size=250m;
      proxy_temp_path /tmp/nginx_proxy 1 2;

      server {
          client_max_body_size 100M;
          location / {
             include uwsgi_params;
             uwsgi_pass unix:///tmp/uwsgi.sock;
             uwsgi_param SCRIPT_NAME '';
          }
      }

b. 建立 alies 至 sites-enabled：

   .. code-block:: bash

      $ sudo ln -s /etc/nginx/sites-available/ckan /etc/nginx/sites-enabled/ckan

5. uwsgi 设置
----------------
   在 virtual env 下安装 uwsgi：

   .. code-block:: bash

      $ . /usr/lib/ckan/default/bin/activate
      (pyenv) $ pip install uwsgi

6. 执行与测试
-------------------------
a. 不要离开 virtual env，执行 nginx 与 uwsgi：

   .. code-block:: bash

      $ sudo service nginx start
      (pyenv) $ uwsgi --ini-paste /etc/ckan/default/production.ini

b. 打开浏览器，前往 http://127.0.0.1/ ，若能看到页面，恭喜您已经完成所有设置！
