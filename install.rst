ckan 安裝
========================================

1. 安装依赖组件
------------------------
   .. code-block:: bash

      $ sudo apt-get install python-dev postgresql libpq-dev python-pip python-virtualenv git-core jetty8 openjdk-7-jdk

2. Virtual environment 设置 
----------------------------
a. 新增一个 virtual environment (virtualenv) 供 ckan 使用：

   .. code-block:: bash

      $ sudo mkdir -p /usr/lib/ckan/default
      $ sudo chown `whoami` /usr/lib/ckan/default
      $ virtualenv --no-site-packages /usr/lib/ckan/default

b. 进入刚才新增的 virtualenv：

   .. code-block:: bash

      $ . /usr/lib/ckan/default/bin/activate

   .. note::

      要离开 virtualenv，可使用 deactivate 指令。若需要返回 virtualenv，可以再运行一次 . /usr/lib/ckan/default/bin/activate 即可。

3. 安装 ckan 2.0
-----------------
   自 github ckeckout source (这边以 release-2.0 为例）并安装：

   .. code-block:: bash

      (pyenv) $ pip install -e 'git+https://github.com/okfn/ckan.git@ckan-2.0#egg=ckan'

   安装所需 Python 组件：

   .. code-block:: bash

      (pyenv) $ pip install -r /usr/lib/ckan/default/src/ckan/pip-requirements.txt

4. 设定数据库
--------------
a. 新增 ckan 使用之 postgreSQL 用户：

   .. code-block:: bash

      $ sudo -u postgres createuser -S -D -R -P ckan_default

b. 新增 ckan 使用之数据库：

   .. code-block:: bash

      $ sudo -u postgres createdb -O ckan_default ckan_default -E utf-8

5. 建立 ckan 设置文件
--------------------
a. 新增放置 ckan 设置文件的目录：

   .. code-block:: bash

      $ sudo mkdir -p /etc/ckan/default
      $ sudo chown -R `whoami` /etc/ckan/

b. 透过 paster 新增示例设置文件：

   .. important::

      运行任何 paster 指令时，请确认是在 virtualenv 下

   .. code-block:: bash

      (pyenv) $ paster make-config ckan /etc/ckan/default/development.ini

c. 修改前面新增的 development.ini，搜寻下面字串，并将帐号密码与 db 名称依照 4. 所新增的 db 设定：

   .. code-block:: ini

      sqlalchemy.url = postgresql://ckan_default:pass@localhost/ckan_default

   .. note::

      第一个 ckan_default 是使用者名称，pass 请填写 db 密码，最后的 ckan_default 填入 db 名称）

6. 设定 jetty8 与 solr4（w/搜索中文支援）
-----------------------------------------
a. 修改 jetty 设定（位于 /etc/default/jetty8）：

   .. code-block:: ini

      NO_START=0
      JETTY_HOST=127.0.0.1
      JETTY_PORT=8983
      JAVA_OPTIONS="-Dsolr.solr.home=/usr/share/solr $JAVA_OPTIONS" 

b. 安装 solr4：

   至官网 http://lucene.apache.org/solr/ 下载 solr-4.6.1
   
   解压缩下载回来的压缩文件
   
   并复制 ./dist 下的 solr-4.6.1.war 至 jetty webapps 目录（solr 目录请自行建立）：

   .. code-block:: bash

      $ sudo cp solr-4.6.1.war /usr/share/jetty8/webapps/solr/solr.war

   复制以下目录至指定位置：

   复制 ./example/solr 至 /usr/share

   复制 ./contrib 至 /usr/share/solr/bin

   复制 ./dist 至 /usr/share/solr

   修改 solr 目录权限，使 jetty 可以存取：
   
   .. code-block:: bash
   
      $ sudo chown -R jetty:adm /usr/share/solr

   新增 schema symlink：

   .. code-block:: bash

      $ sudo mv /usr/share/solr/collection1/conf/schema.xml /usr/share/solr/collection1/conf/schema.xml.bak
      $ sudo ln -s /usr/lib/ckan/default/src/ckan/ckan/config/solr/schema-2.0.xml /usr/share/solr/collection1/conf/schema.xml

   解压缩 solr-4.6.1.war：
   
   .. code-block:: bash
      
      $ jar -xvf solr.war

   复制 b. 所下载之 solr 压缩文件中之 ./example/lib/ext 下的所有 jar 文件至 /usr/share/jetty8/webapps/solr/WEB-INF/lib

   承上，复制 ./example/resources/log4j.properties 至 /usr/share/jetty8/webapps/solr/WEB-INF/classes

c. 安装 IKAnalyzer：

   下载 IKAnalyzer https://ik-analyzer.googlecode.com/files/IK%20Analyzer%202012FF_hf1.zip 并解压缩

   复制 IKAnalyzer2012FF_fh1.jar 至 /var/lib/jetty8/webapps/solr/WEB-INF/lib
  
   复制 IKAnalyzer.cfg.xml 和 stopword.dic 至 /var/lib/jetty8/webapps/solr/WEB-INF/class

d. 设定 IKAnalyzer：

   修改 schema.xml，fieldType name="text" 区段修改为：

   .. code-block:: xml

      <fieldType name="text" class="solr.TextField">
         <analyzer type="index" class="org.wltea.analyzer.lucene.IKAnalyzer" isMaxWordLength="false"/>
         <analyzer type="query" class="org.wltea.analyzer.lucene.IKAnalyzer" isMaxWordLength="false"/>
         <filter class="solr.SynonymFilterFactory" synonyms="synonyms.txt" ignoreCase="true" expand="true"/>
         <filter class="solr.WordDelimiterFilterFactory" generateWordParts="1" generateNumberParts="1" catenateWords="0" catenateNumbers="0" catenateAll="0" splitOnCaseChange="1"/>
         <filter class="solr.SnowballPorterFilterFactory" language="English" protected="protwords.txt"/>
         <filter class="solr.LowerCaseFilterFactory"/>
         <filter class="solr.ASCIIFoldingFilterFactory"/>
      </fieldType>

   .. note::

       schema.xml 位于 /usr/share/solr/collection1/conf/schema.xml

e. 启动 jetty：

   .. code-block:: bash

      $ sudo service jetty8 start

f. 打开浏览器，前往 http://127.0.0.1:8983/solr ，若能看到画面则代表安装完成


7. 初始化资料库
------------------------
a. 透过 paster 初始化 ckan db：

   .. code-block:: bash

      (pyenv) $ paster db init -c /etc/ckan/default/development.ini

b. 如果一切正常，则会看到此讯息：Initialising DB: SUCCESS

8. 建立 who.ini link
------------------------
   .. code-block:: bash

      $ ln -s /usr/lib/ckan/default/src/ckan/who.ini /etc/ckan/default/who.ini

9. 新增 ckan 系统管理员
------------------------
   透过 paster 新增 ckan 系统管理员：

   .. code-block:: bash

      (pyenv) $ paster sysadmin add admin -c /etc/ckan/default/development.ini

   .. note::

      admin 请代换为您需要的使用者名称，并依照程式提示设定密码

10. 在 development 环境下运行
------------------------------
a. 透过 paster serve 新安装的 ckan instance：

   .. code-block:: bash

      (pyenv) $ paster serve /etc/ckan/default/development.ini

b. 打开浏览器，前往 http://127.0.0.1:5000/ ，至此 ckan 安装完成
