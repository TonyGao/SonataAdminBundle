导出操作
=================

这篇文档会涵盖导出操作以及相关配置选项。

基本配置
-------------------

If you have registered the ``SonataExporterBundle`` bundle in the kernel of your application,
you can benefit from a lot of flexibility:

如果你已经在程序的内核注册了 ``SonataExporterBundle``，

* 你可以全局的配置默认的导出器。
* 你也可以添加全局的自定义导出器。
* 你可以配置所有默认的导出器。

查看 `the exporter bundle documentation`_ 了解更多信息。

翻译
~~~~~~~~~~~

默认所有字段名称都是被翻译的。
有一个内部的机制会检查当前的翻译文件中是否有与翻译器的策略标签相匹配的字段，并将字段名称作为备用。


选择要导出的字段
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

默认，所有字段都会被导出。更确切的说，它取决于你正在使用的持久化后端，例如，doctrine ORM 
后端会导出所有字段(不导出关联)。如果你想要为特定的 admin 修改这个行为，你可以覆盖 
``getExportFields()`` 方法:

.. code-block:: php

    <?php

    public function getExportFields()
    {
        return array('givenName', 'familyName', 'contact.phone');
    }

.. note::

    注意你可以使用 `contact.phone` 来访问 `Contact` 数据实体的 `phone` 属性

你也可以通过创建一个实现 ``configureExportFields()`` 方法的 admin 扩展来
修改列表。

.. code-block:: php

    public function configureExportFields(AdminInterface $admin, array $fields)
    {
        unset($fields['updatedAt']);

        return $fields;
    }


为一个特定的 admin 覆盖导出格式
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

在 admin 类中通过定义一个 ``getExportFormats`` 方法可以修改导出格式。

.. code-block:: php

    <?php

    public function getExportFormats()
    {
        return array('pdf', 'html');
    }

自定义用于获取结果的查询
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
如果你想要为一个特定的 admin 自定义用来获取结果查询，你可以覆盖 ``getDataSourceIterator`` 方法:

.. code-block:: php

    <?php
    // src/AppBundle/Admin/PersonAdmin.php

    class PersonAdmin extends AbstractAdmin
    {
        public function getDataSourceIterator()
        {
            $iterator = parent::getDataSourceIterator();
            $iterator->setDateTimeFormat('d/m/Y'); //修改这里来适合你的需求
            return $iterator;
        }
    }

.. note::

    **TODO**:
    * 自定义模板用来渲染输出
    * 发布导出器文档到项目网站并更新链接

.. _`the exporter bundle documentation`: https://github.com/sonata-project/exporter/blob/1.x/docs/reference/symfony.rst
