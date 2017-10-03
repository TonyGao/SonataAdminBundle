仪表盘
=========

仪表盘是主要的页面。默认它陈列了在 ``Admin`` 服务中定义的已映射的模型。这有助于你快速开始使用
``SonataAdminBundle`` ，但仪表盘的优点还远不止这些。

默认的仪表盘在 ``/admin/dashboard``, 其由 ``SonataAdminBundle:Core:dashboard`` 控制器操作
来处理。这个操作的默认视图文件是 ``SonataAdminBundle:Core:dashboard.html.twig``, 但你可以在
你的 ``config.yml`` 里对它进行修改:

.. configuration-block::

    .. code-block:: yaml

        # app/config/config.yml

        sonata_admin:
            templates:
                dashboard: SonataAdminBundle:Core:dashboard.html.twig

.. note::

    此视图像 ``SonataAdminBundle`` 的大多数视图一样，扩展了一个全局的模板文件，其包含了页面的重要部分。
    详细信息请查阅 :doc:`templates` 模板章节。

Blocks
------

仪表盘实际上使用了 ``SonataBlockBundle`` 的 block 来构建的。你可以通过 `SonataBlock documentation page`_
来学习这个 bundle 以及怎样构建你自己的 block 。

``Admin`` 列表 block
------------------------

``Admin`` 列表是一个 ``Block`` ，其从 ``Admin`` 服务的 ``Pool`` 获取信息并将它以漂亮的列表格式
打印到你默认的仪表盘上。``Admin`` 列表是由 ``sonata.admin.block.admin.admin_list`` 服务定义的，
其实现自 ``Block\AdminListBlockService`` 类。然后它使用 ``SonataAdminBundle:Block:block_admin_list.html.twig``
模板文件渲染出来。

闲来看看这些文件。你会发现它的代码特别简单并易于理解，对于你实现自己的 block 也大有裨益。

配置 ``Admin`` 列表
------------------------------

你现在或许注意到了，``Admin`` 列表将 ``Admin`` 映射组合到了一起。有几种方法可以用来配置这些组。

默认管理是有你定义它们的顺序排序的。用 ``sort_admins`` 设置，分组和管理将按相应的带一个回执到 admin id 
的标签进行排序。

使用 ``Admin`` 服务声明
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

当定义你的 ``Admin`` 服务时第一个也是最常用的方法是设定一个分组:

.. configuration-block::

    .. code-block:: xml

        <service id="app.admin.post" class="AppBundle\Admin\PostAdmin">
              <tag name="sonata.admin" manager_type="orm"
                  group="Content"
                  label="Post" />
              <argument />
              <argument>AppBundle\Entity\Post</argument>
              <argument />
          </service>

    .. code-block:: yaml

        services:
            app.admin.post:
                class: AppBundle\Admin\PostAdmin
                tags:
                    - name: sonata.admin
                      manager_type: orm
                      group: "Content"
                      label: "Post"
                arguments:
                    - ~
                    - AppBundle\Entity\Post
                    - ~
                public: true

在这些示例里，注意 ``group`` 标签，表示了这个特定的 ``Admin`` 服务属于 ``Content`` 组。

.. configuration-block::

    .. code-block:: xml

        <service id="app.admin.post" class="AppBundle\Admin\PostAdmin">
              <tag name="sonata.admin" manager_type="orm"
                  group="app.admin.group.content"
                  label="app.admin.model.post" label_catalogue="AppBundle" />
              <argument />
              <argument>AppBundle\Entity\Post</argument>
              <argument />
          </service>

    .. code-block:: yaml

        services:
            app.admin.post:
                class: AppBundle\Admin\PostAdmin
                tags:
                    - name: sonata.admin
                      manager_type: orm
                      group: "app.admin.group.content"
                      label: "app.admin.model.post"
                      label_catalogue: "AppBundle"
                arguments:
                    - ~
                    - AppBundle\Entity\Post
                    - ~

在这个示例里，翻译标签由 ``AppBundle`` 翻译，使用给定的 ``label_catalogue`` 。因此你可以使用上边的例子
来在你的项目中提供多语言支持。

.. note::

    你可以在任何情况下使用参数(如，``%app_admin.group_post%``) 作为组名。

使用 ``config.yml``
^^^^^^^^^^^^^^^^^^^^^^^^

你也可以在你的 ``config.yml`` 文件里配置 ``Admin`` 列表。这个配置方法覆盖在 Admin 
服务中的任何设置声明。

.. configuration-block::

    .. code-block:: yaml

        # app/config/config.yml

        sonata_admin:
            dashboard:
                groups:
                    app.admin.group.content:
                        label: app.admin.group.content
                        label_catalogue: AppBundle
                        items:
                            - app.admin.post

                    app.admin.group.blog:
                        items: ~
                        item_adds:
                            - sonata.admin.page
                        roles: [ ROLE_ONE, ROLE_TWO ]

                    app.admin.group.misc: ~

.. note::

    这是一个学术的、完全的配置示例。在实际情况中，通常不需要使用全部所示的选项。对任何设定使用默认值
    只要不填写或使用 ``~`` 作为选项值即可。

这个配置设定 ``app.admin.group.content`` 分组使用 ``app.admin.group.content`` 翻译标签，
其使用 ``AppBundle`` 翻译目录进行翻译(和我们之前在服务定义的示例中声明了同样的翻译标签和翻译配置)。

它还指出 ``app.admin.group.content`` 分组仅包含了 ``app.admin.post`` ``Admin`` 映射，
也就是说任何其他属于这个分组的 ``Admin`` 服务声明将不会在这里显示。

第二，我们声明一个 ``app.admin.group.blog`` 分组拥有全部默认选项(如，在 ``Admin`` 服务声明中设定的那样)，
添加一个 *额外的* ``sonata.admin.page`` 映射，最初不属于这个分组。 

我们也在这里使用 ``roles`` 选项，这意味着只有具有 ``ROLE_ONE`` 或 ``ROLE_TWO`` 权限的用户才能能够看到此组，
而不是默认设置允许每个人都看到给定的分组。始终拥有 ``ROLE_SUPER_ADMIN`` 的用户始终能够看到被此配置选项隐藏的
组所不能看到的内容。

第三个分组, ``app.admin.group.misc``, 被用来设定一个只使用默认值的分组，在服务声明中声明默认值。

添加更多 Block
------------------

就像我们以前说的，仪表盘默认自带一个 ``Admin`` 列表的块, 但你你可以为它创建和添加更多的块。

.. figure:: ../images/dashboard.png
   :align: center
   :alt: Dashboard
   :width: 500

在这张截图中，除了左边的默认 ``Admin`` 列表块之外，我们还在右侧添加了一个文本块和 RSS 订阅块。
此方案的配置为：

.. configuration-block::

    .. code-block:: yaml

        # app/config/config.yml

        sonata_admin:
            dashboard:
                blocks:
                    -
                        position: left
                        type: sonata.admin.block.admin_list
                    -
                        position: right
                        type: sonata.block.service.text
                        settings:
                            content: >
                                <h2>Welcome to the Sonata Admin</h2>
                                <p>This is a <code>sonata.block.service.text</code> from the Block
                                Bundle, you can create and add new block in these area by configuring
                                the <code>sonata_admin</code> section.</p> <br /> For instance, here
                                a RSS feed parser (<code>sonata.block.service.rss</code>):
                    -
                        position: right
                        type: sonata.block.service.rss
                        roles: [POST_READER]
                        settings:
                            title: Sonata Project's Feeds
                            url: https://sonata-project.org/blog/archive.rss

.. note::

    块可以接受/需要额外传入配置来更好的工作。参考相关的文档/实现来了解关于每个块选项和要求
    的更多信息。

    你也可以配置 ``roles`` 部分来配置可以查看此块的用户。

为不同的仪表盘分组显示两个 ``Admin`` 列表块
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

同一个块可以有多个实例，并且在每个实例使用不同的配置的仪表盘中多次显示。一个特定的例子是
``Admin`` 列表块，其可以配置为适合这种场景。

.. configuration-block::

    .. code-block:: yaml

        # app/config/config.yml

        sonata_admin:
            dashboard:
                blocks:

                    # 显示两个仪表盘块
                    -
                        position: left
                        type: sonata.admin.block.admin_list
                        settings:
                            groups: [sonata_page1, sonata_page2]
                    -
                        position: right
                        type: sonata.admin.block.admin_list
                        settings:
                            groups: [sonata_page3]

                groups:
                    sonata_page1:
                        items:
                            - sonata.page.admin.myitem1

                    sonata_page2:
                        items:
                            - sonata.page.admin.myitem2
                            - sonata.page.admin.myitem3

                    sonata_page3:
                        items:
                            - sonata.page.admin.myitem4

在这个例子里，你的仪表盘酱油两个 ``admin_list`` 块，每个块都仅包含各自配置的分组。

.. _`SonataBlock documentation page`:  https://sonata-project.org/bundles/block/master/doc/index.html


统计块
~~~~~~~~~~~~~~~

一个统计块可以用于显示一个带有颜色的简单计数器，font awesome 图标和文本。计数器与某个管理内容的过滤器相关

.. configuration-block::

    .. code-block:: yaml

        sonata_admin:
            dashboard:
                blocks:
                    -
                        class:    col-lg-3 col-xs-6          # bootstrap 的响应代码
                        position: top                        # 仪表盘的区域
                        type:     sonata.admin.block.stats   # block id
                        settings:
                            code:  sonata.page.admin.page    # 管理内容代码 - 服务 id
                            icon:  fa-magic                  # font awesome 图标
                            text:  Edited Pages
                            color: bg-yellow                 # 颜色: bg-green, bg-red and bg-aqua
                            filters:                         # 过滤器值
                                edited: { value: 1 }

仪表盘布局
~~~~~~~~~~~~~~~~

现在支持的位置包括：

* top
* left
* center
* right
* bottom

布局包括:

.. code-block:: bash

    TOP     TOP     TOP

     LEFT CENTER RIGHT
     LEFT CENTER RIGHT
     LEFT CENTER RIGHT

    BOTTOM BOTTOM BOTTOM

在 ``top`` 和 ``bottom`` 的位置，你也可以设定一个选项 ``class`` 来设定这个块的宽度。

.. configuration-block::

    .. code-block:: yaml

        # app/config/config.yml

        sonata_admin:
            dashboard:
                blocks:

                    # 在顶部以 col-md-6 的 css 类来显示仪表盘块
                    -
                        position: top
                        class: col-md-6
                        type: sonata.admin.block.admin_list

配置仪表盘中每个子项可用的操作
---------------------------------------------------------------------

默认，在仪表盘每个子项都有一个 ``列表`` 和一个 ``创建`` 选项。如果你创建了一个自定义操作，
想在仪表盘和其他两个一块显示出来，你可以通过覆盖你 admin 类的 ``getDashboardActions()``
方法来实现：

.. code-block:: php

    <?php
    // src/AppBundle/Admin/PostAdmin.php

    class PostAdmin extends AbstractAdmin
    {
        // ...

        public function getDashboardActions()
        {
            $actions = parent::getDashboardActions();

            $actions['import'] = array(
                'label'              => 'Import',
                'url'                => $this->generateUrl('import'),
                'icon'               => 'import',
                'translation_domain' => 'SonataAdminBundle', // 可选的
                'template'           => 'SonataAdminBundle:CRUD:dashboard__action.html.twig', // 可选的
            );

            return $actions;
        }

    }

你也可以通过 unset 它来在仪表盘隐藏一个操作：

.. code-block:: php

    <?php
    // src/AppBundle/Admin/PostAdmin.php

    class PostAdmin extends AbstractAdmin
    {
        // ...

        public function getDashboardActions()
        {
            $actions = parent::getDashboardActions();

            unset($actions['list']);

            return $actions;
        }

    }

如果你这么做了，你需要注意这个操作仅仅是隐藏了。通过直接访问它的网址它还是可以使用的，除非你
使用适合的安全措施来防止访问(如，ACL 或者基于角色的)。
