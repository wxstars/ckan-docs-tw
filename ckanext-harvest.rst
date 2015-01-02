ckanext-harvest
===============

ckanext-harvest 是一个 ckan 的扩展（extension），提供一可自定义界面（interface），以抓取其他网站（或服务）之 metadata，并汇入为 ckan 数据集。

harvest 的运行大致可分为三步骤（同时也是设计 harvesting interface 的主要结构）:

* gather: 取得 harvest source 的 id, 数量等基本信息。
* fetch: 取得 source 中每个 object（物件，或称数据集）之 metadata。
* import: 将上一阶段取得的 metadata 转换并建立为 ckan package（数据集）。

外挂主要功能简介与使用
----------------------

新增 harvest source
^^^^^^^^^^^^^^^^^^^^

使用浏览器开启 SITE_URL/harvest，选取右上之 "Add Harvest source"，依照画面输入 source 网址及选取 source 类别。

执行 harvest 工作（手动）
^^^^^^^^^^^^^^^^^^^^^^^^^

a. 进入 virtualenv，执行 gather 与 fetch handler：

   .. code-block:: bash

      (pyenv) $ paster --plugin=ckanext-harvest harvester gather_consumer -c /etc/ckan/default/production.ini
      (pyenv) $ paster --plugin=ckanext-harvest harvester fetch_consumer -c /etc/ckan/default/production.ini

   .. note::

      请勿关闭这两个 handler

b. 使用浏览器开启 SITE_URL/harvest，进入刚才建立的 harvest source，选择右上的“管理者”按钮，在接下来的页面选取 ``Reharvest`` ，将此 harvest 工作送入排程。

c. 最后进入 virtualenv，执行 run handler：

   .. code-block:: bash

      (pyenv) $ paster --plugin=ckanext-harvest harvester run -c /etc/ckan/default/production.ini


   即会立即开始执行刚才加入的工作排程。

   .. note::

      手动执行时 harvest 工作并不会自行停止，因为上述 paster harvester run 指令同时也用来确认 harvest 工作是否完成。因此若您确定 harvest 工作已经完成（或已发生错误），可以再次执行 run 指令，即可透过下述 d. 的方式检视此次工作的结果

执行 harvest 工作（自动）
^^^^^^^^^^^^^^^^^^^^^^^^^

在 production 环境时，我们会希望系统可以每隔一段时间自动进行 harvesting，此时可以使用 Supervisor 与 cron 来达到目的：

* `Supervisor <http://supervisord.org/>`_ : 一套任务管理工具，可以在背景执行指定之工作，我们用它来在背景执行 harvest 的 ``gather_consumer`` 与 ``fetch_consumer`` 两个常驻工作。
* cron: unix/linux 系统工具，可以定时执行之工作，我们用它来定时执行 harvest ``run`` 工作。

a. 首先我们要安装 Supervisor：

   .. code-block:: bash

      $ sudo apt-get install supervisor

   您可以透过以下指令确定 Supervisor 是否正在执行：

   .. code-block:: bash

      $ ps aux | grep supervisord

   若 Supervisor 正在执行，则会看到类似以下的输出：

   .. code-block:: bash

      root      9224  0.0  0.3  56420 12204 ?        Ss   15:52   0:00 /usr/bin/python /usr/bin/supervisord

b. Supervisor 的设置文件位于 /etc/supervisor/conf.d 目录下，我们新增一个新的设置文件，命名为 ckan_harvesting.conf，内容如下：

   .. code-block:: python

      ; ===============================
      ; ckan harvester
      ; ===============================

      [program:ckan_gather_consumer]

      command=/usr/lib/ckan/default/bin/paster --plugin=ckanext-harvest harvester gather_consumer -c /etc/ckan/default/production.ini

      ; user that owns virtual environment.
      user=okfn

      numprocs=1
      stdout_logfile=/var/log/ckan/default/gather_consumer.log
      stderr_logfile=/var/log/ckan/default/gather_consumer.log
      autostart=true
      autorestart=true
      startsecs=10

      [program:ckan_fetch_consumer]

      command=/usr/lib/ckan/default/bin/paster --plugin=ckanext-harvest harvester fetch_consumer -c /etc/ckan/default/production.ini

      ; user that owns virtual environment.
      user=okfn

      numprocs=1
      stdout_logfile=/var/log/ckan/default/fetch_consumer.log
      stderr_logfile=/var/log/ckan/default/fetch_consumer.log
      autostart=true
      autorestart=true
      startsecs=10

   其中 ``user=okfn`` 请代换成 python virtual environment 的拥有者， ``/var/log/ckan/default`` 目录请自行新增，拥有者同样为 virtualenv 拥有者

c. 接着启动 Supervisor，请依序输入以下指令：

   .. code-block:: bash

      $ sudo supervisorctl reread
      $ sudo supervisorctl add ckan_gather_consumer
      $ sudo supervisorctl add ckan_fetch_consumer
      $ sudo supervisorctl start ckan_gather_consumer
      $ sudo supervisorctl start ckan_fetch_consumer

   您可以透过以下指令确定工作是否正在执行：

   .. code-block:: bash

      $ sudo supervisorctl status

   若 Supervisor 正在执行，则会看到类似以下的输出：

   .. code-block:: bash

      ckan_fetch_consumer              RUNNING    pid 6983, uptime 0:22:06
      ckan_gather_consumer             RUNNING    pid 6968, uptime 0:22:45

d. 最后我们要建立定时执行 ``run`` 排程，执行下列指令打开排程设置文件：

   .. code-block:: bash

      $ sudo crontab -e -u okfn

   ``okfn`` 请代换为 virtualenv 拥有者

e. 进行排程设置，请加入以下文字于 crontab 设置中：

   # m  h  dom mon dow   command

   \*/15 *  *   *   *     /usr/lib/ckan/default/bin/paster --plugin=ckanext-harvest harvester run -c /etc/ckan/default/production.ini

确认 harvest 工作的执行状况
^^^^^^^^^^^^^^^^^^^^^^^^^^^

我们可以在网页介面，harvest source 的“管理者”页面确认 harvest 工作的执行状况，包括错误、新增、更新、完成的数据集数目，如下图所示：

   .. image:: harvest-job-status.png

撰写自定义 harvesting interface
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

如先前所述，ckanext-harvest 提供可以自行定义的 interface，因此您可以为某个网站，或某种数据来源，特别制作 harvester。

* 本人撰写之 `中研院调查研究中心（SRDA）数据库 <https://srda.sinica.edu.tw/>`_ harvester： `SRDAHarvester <https://github.com/u10313335/ckanext-harvest/blob/master/ckanext/harvest/harvesters/srdaharvester.py>`_
* ckan 官方提供之 csv harvester 范例： `DataLondonGovUkHarvester <https://github.com/okfn/ckanext-pdeu/blob/master/ckanext/pdeu/harvesters/london.py>`_

系统需求
--------
* Python (2 or 3) 安装于 virtualenv
* ckan
* RabbitMQ 或 Redis

安装
----
a. 安装 RabbitMQ 或 Redis（两者则一安装即可，以下用 Redis 作示范）：

   .. code-block:: bash

      $ sudo apt-get install rabbitmq-server
      $ sudo apt-get install redis-server

b. 安装 ckanext-harvest 组件：

   .. code-block:: bash

      (pyenv) $ pip install -e git+https://github.com/okfn/ckanext-harvest.git@release-v2.0#egg=ckanext-harvest

   .. note::

      release-v2.0 请自行依 ckan 版本替换之

c. 安装其他需要的 Python 组件：

   .. code-block:: bash

      (pyenv) $ pip install -r pip-requirements.txt

d. 修改 ckan 设置文件，修改 ckan.plugins，加入：

   .. code-block:: python

      ckan.plugins = harvest ckan_harvester
      ...
      ckan.harvest.mq.type = redis

e. 初始化 harvest 数据库：

   .. code-block:: bash

      (pyenv) $ paster --plugin=ckanext-harvest harvester initdb -c /etc/ckan/default/production.ini
