搜索
======

在导航菜单上方后台自带了一个基本的全局搜索。搜索遍历 admin 类来寻找 ``global_search`` 设置为 true 的过滤器。如果你使用
``SonataDoctrineORMBundle`` 那么所有文本过滤器都将默认设定为 ``true`` 。

自定义
-------------

主要的操作是用 ``SonataAdminBundle:Core:search.html.twig`` 模板。每个搜索都被一个 ``block`` 处理，这个块的模板是 
``SonataAdminBundle:Block:block_search_result.html.twig`` 。

默认的模板值可以在配置部分进行配置

.. configuration-block::

    .. code-block:: yaml

        # app/config/config.yml

        sonata_admin:
            templates:
                # 其他配置选项
                search:              SonataAdminBundle:Core:search.html.twig
                search_result_block: SonataAdminBundle:Block:block_search_result.html.twig
                
你也需要在 sonata 块配置里对这个块进行配置

.. configuration-block::

    .. code-block:: yaml

        # app/config/config.yml

        sonata_block:
            blocks:
                sonata.admin.block.search_result:
                contexts: [admin]

你也可以在定义 admin 时每个 admin 都配置块模板：

.. configuration-block::

    .. code-block:: xml

        <service id="app.admin.post" class="AppBundle\Admin\PostAdmin">
              <tag name="sonata.admin" manager_type="orm" group="Content" label="Post" />
              <argument />
              <argument>AppBundle\Entity\Post</argument>
              <argument />
              <call method="setTemplate">
                  <argument>search_result_block</argument>
                  <argument>SonataPostBundle:Block:block_search_result.html.twig</argument>
              </call>
          </service>

配置默认搜索结果操作
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

一般来说，搜索结果会生成一个链接到一个项目的编辑操作，或正用来做显示操作，如果编辑路由被禁用或您没有所需的权限。
您可以通过覆盖 ``searchResultActions`` 属性来修改此行为。定义的操作列表会被一直检查，直到路由有所需的权限
得到满足。如果没有路由满足条件，那么这个子项将显示为一个文本。

.. code-block:: php

    <?php
    // src/AppBundle/Admin/PersonAdmin.php

    class PersonAdmin extends AbstractAdmin
    {
        // ...

        protected $searchResultActions = array('edit', 'show');

        // ...
    }

性能
-----------

如果你有大量的数据实体，现在的实现性能消耗巨大，因为结果查询是通过 ``LIKE %query% OR LIKE %query%`` 实现的 ...

.. note::

    我们正在做一版通过异步 JavaScript 的方案来从数据库更好的加载数据。
