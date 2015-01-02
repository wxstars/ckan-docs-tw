Linked Data and RDF
===================

资源描述架构（Resource Description Framework）及 RDF Schema 系由 W3C 制定，用来解决资源描述问题的规范，利用阶层式的概念及属性描述数据的 metadata。

ckan 自 1.7 版后开始内建支持 RDF 格式输出，使用非常容易。

使用方法
--------

以下两种方式均可获得特定数据集的 RDF 格式描述：

方法一
^^^^^^

.. code-block:: bash
   
   curl -L -H "Accept: application/rdf+xml" http://thedatahub.org/dataset/gold-prices

方法二
^^^^^^

.. code-block:: bash

   curl -L http://thedatahub.org/dataset/gold-prices.rdf

Schema Mapping
--------------

ckan 数据集的所有栏位都可以自 RDF 取得，其对应请参见官方说明： `Schema Mapping <http://docs.ckan.org/en/ckan-2.0.2/linked-data-and-rdf.html#schema-mapping>`_
