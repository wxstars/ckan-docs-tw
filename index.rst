.. ckan-2.0_ins documentation master file, created by
   sphinx-quickstart on Sun Jul 21 11:15:04 2013.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

ckan 2.0 安裝使用教学
========================================

ckan 是著名的开放数据门户平台（Data Portal Platform），
由 Open data 界大名鼎鼎的 Open Knowledge Foundation（OKF）支持发展。

他的功能非常多，除了 data repository 外，还支持 visualize、search、tag、revision、share、organization…，
更有许多的 plugins 可以强化其功能。

使用 ckan 最有名的案例，系英国政府开放数据平台 data.gov.uk。

ckan 使用以 Python 为基础的 Pylons 网页框架开发，template 使用 jinja（神社）2，多国语言支援采用 Babel 系统，数据库使用 PostgreSQL，ORM 是 Pylons 推荐的 SQLAlchemy，搜索功能则使用 Apache Solr 搭建，同时搭配 jetty 作为 servlet container。

这篇教学文档将说明如何自 \*nix OS 从无到有安装一个 ckan 2.0 系统。环境为 Linux Mint 14 (based on Ubuntu 12.10)，大致按照官方文件 `Install from Source <http://docs.ckan.org/en/ckan-2.0/install-from-source.html>`_ 的方式。其他相关的使用心得也会一并公布于此。


.. toctree::
   :maxdepth: 2

   install
   deployment
   ckanext-spatial
   ckanext-harvest
   rdf

Indices and tables
==================

* :ref:`genindex`
* :ref:`modindex`
* :ref:`search`

