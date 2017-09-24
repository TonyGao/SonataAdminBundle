安装
============

SonataAdminBundle 仅仅是一个 bundle 包，所以你可以在一个项目生命周期的任何时刻来安装它。

1. 下载此 Bundle
----------------------

打开一个终端，进入你的项目目录，执行下边的命令来下载这个 bundle 的最新稳定版：

.. code-block:: bash

    $ composer require sonata-project/admin-bundle

这个命令需要你已经全局安装了 Composer , 就如 Composer 文档的
 `installation chapter`_ 所述。

1.1. 下载一个储存层 Bundle
------------------------------

你现在已经下载了 SonataAdminBundle 。虽然这个包包含了所有功能，但它需要存储层的 
bundle来与数据库通信。在使用 SonataAdminBundle 之前，你得下载这些存储层 Bundle 
的一个。这些官方的存储层 bundle 包括：

* `SonataDoctrineORMAdminBundle`_ (与 Doctrine ORM 整合);
* `SonataDoctrineMongoDBAdminBundle`_ (与 Doctrine MongoDB ODM 整合);
* `SonataPropelAdminBundle`_ (与 Propel 整合);
* `SonataDoctrinePhpcrAdminBundle`_ (与 Doctrine PHPCR ODM 整合).

你可以以 SonataAdminBundle 同样的方式来下载它们。例如，下载 
SonataDoctrineORMAdminBundle，执行下边的命令：

.. code-block:: bash

    $ composer require sonata-project/doctrine-orm-admin-bundle

.. tip::

    不知道选哪个？大多数新用户会选 SonataDoctrineORMAdmin，其与传统的关系型数据库整合
    (MySQL, PostgreSQL 等等)。

步骤2：开始此 Bundle
-------------------------

然后，开启这个 bundle 和其他所依赖的 bundle，如下，在你项目的 app/AppKernel.php 
文件里加入这些行：

.. code-block:: php

    // app/AppKernel.php

    // ...
    class AppKernel extends Kernel
    {
        public function registerBundles()
        {
            $bundles = array(
                // ...

                // 后台管理需要一些在安全 bundle 中定义的 twig 函数 ，如 is_granted 。
                // 如果还没登记就把它登记上。
                new Symfony\Bundle\SecurityBundle\SecurityBundle(),

                // 这里是 SonataAdminBundle 所依赖的其他的 Bundle
                new Sonata\CoreBundle\SonataCoreBundle(),
                new Sonata\BlockBundle\SonataBlockBundle(),
                new Knp\Bundle\MenuBundle\KnpMenuBundle(),

                // 最后，存储层和 SonataAdminBundle
                new Sonata\AdminBundle\SonataAdminBundle(),
            );

            // ...
        }

        // ...
    }

.. note::

    如果一个 bundle 已经在你的 AppKernel.php 里做过登记了，就不要再登记一遍了。

.. note::

    从 2.3 版开始，此 bundle 就自带了 jQuery和一些其他的前端库。要更新版本(不要的)，
    你可以使用 `Bower`_. 为了确保你能得到与你 SonataAdminBundle 的版本相匹配的依赖，
    你可以让 bower 使用本地的 bower 依赖文件，像这样：

    .. code-block:: bash

        $ bower install ./vendor/sonata-project/admin-bundle/bower.json

.. note::

    你必须在 `config.yml` 中开启翻译器服务

    .. code-block:: yaml

        framework:
            translator: { fallbacks: ["%locale%"] }

    详见: http://symfony.com/doc/current/translation.html#configuration

步骤3：配置已安装的 Bundle
---------------------------------------

现在所需的所有 bundle 都下载并登记了，你还得添加一些配置。它的后台管理界面使用 SonataBlockBundle 
将所有东西都放到块中。你得告诉这个 block bundle 这个后台管理块的存在：

.. code-block:: yaml

    # app/config/config.yml
    sonata_block:
        default_contexts: [cms]
        blocks:
            # 开启 SonataAdminBundle 块
            sonata.admin.block.admin_list:
                contexts: [admin]
            # ...

.. note::

    就在此刻，如果你还没完全搞清楚啥是 block ，也不用过分担心。SonataBlockBundle 
    是个非常有用的工具，但并不是意味着你需要理解它来使用后台管理的 bundle 。

步骤4：导入路由配置
------------------------------------

这些 bundle 已经登记好了，也正确配置了。在使用它之前，Symfony 路由需要知道 SonataAdminBundle 
所提供的路由有哪些。你可以通过在路由配置中引入它们来做到：

.. code-block:: yaml

    # app/config/routing.yml
    admin_area:
        resource: "@SonataAdminBundle/Resources/config/routing/sonata_admin.xml"
        prefix: /admin

步骤5：开启”翻译器”服务
---------------------------------------

SonataAdmin 需要翻译器服务来显示所有可能的标签。

.. code-block:: yaml

    # app/config/config.yml
    framework:
        translator: { fallbacks: [en] }

步骤6： 定义路由
---------------------

要进入 SonataAdminBundle 的页面，需要把它的路由添加到程序的主路由文件：

.. configuration-block::

    .. code-block:: yaml

        # app/config/routing.yml

        admin:
            resource: '@SonataAdminBundle/Resources/config/routing/sonata_admin.xml'
            prefix: /admin

        _sonata_admin:
            resource: .
            type: sonata_admin
            prefix: /admin

.. note::

    如果你在使用 XML 或 PHP 来设置你的程序配置，上边的路由配置必须用 routing.xml 或
    routing.php 来配置，这取决于你的格式(如，XML 或 PHP)。

.. note::

    对 ``resource: .`` 设定好奇的人：这是一种不同寻常的语法用法，因为 Symfony 需要一个定义好的资源
    (其指向一个真实的文件)。一旦通过了 Sonata 的负责处理这个路由的 ``AdminPoolLoader`` 的验证过程，
    它只是忽略这个资源设定。

此刻你已经可以进入(空的) admin 仪表盘了，只要访问：
``http://yoursite.local/admin/dashboard``.

步骤7：环境准备
----------------------------------

因为你安装了很多 bundle，清除缓存并安装一遍静态资源是一个好习惯：

.. code-block:: bash

    $ php bin/console cache:clear
    $ php bin/console assets:install

后台管理界面
-------------------

你已经完成了安装过程和相关配置。如果你想访问服务器，你可以现在访问后台管理页面
 http://localhost:8000/admin

.. note::

    这篇教程假设你使用内置的服务器，通过
    ``php bin/console server:start`` (或``server:run``) 命令执行。

.. image:: ../images/getting_started_empty_dashboard.png

如你所见，管理后台控制面板是空的。这是因为还没 bundle 给后台提供管理功能。幸运的是，
你在下一章 :doc:`next chapter <creating_an_admin>` 就会学习怎么用了。

.. _`installation chapter`: https://getcomposer.org/doc/00-intro.md
.. _SonataDoctrineORMAdminBundle: http://sonata-project.org/bundles/doctrine-orm-admin/master/doc/index.html
.. _SonataDoctrineMongoDBAdminBundle: http://sonata-project.org/bundles/mongo-admin/master/doc/index.html
.. _SonataPropelAdminBundle: http://sonata-project.org/bundles/propel-admin/master/doc/index.html
.. _SonataDoctrinePhpcrAdminBundle: http://sonata-project.org/bundles/doctrine-phpcr-admin/master/doc/index.html
.. _Bower: http://bower.io/
