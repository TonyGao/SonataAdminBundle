2. 开始使用 SonataAdminBundle
======================================

如果你遵循了安装过程，SonataAdminBundle 应该已经安装好了，但还不可用。在使用之前你得配置你的模型。这里
有一个让你 SonataAdminBundle 快速工作并为你程序的模型创建后台管理界面的勾选清单：

* 步骤 1：创建一个 Admin 类
* 步骤 2： 创建一个 Admin 服务
* 步骤 3： 配置

1.1 创建一个 Admin 类
---------------------

SonataAdminBundle 通过图形界面帮助你管理你的数据，可以让你创建，更新或搜索你的模型的实例。这些操作
需要通过配置 Admin 类来配置。

Admin 类会呈现你模型对每个管理操作的映射。其中，你可以决定哪些字段会显示到清单里，哪些用于过滤，或者
哪些在创建或编辑表单里显示出来。

创建一个 Admin 类最简单的方法是扩展 ``Sonata\AdminBundle\Admin\AbstractAdmin`` 类。

假设你的 ``AppBundle`` 有一个 ``Post`` 数据实体。这里有一个基本的 Admin 类的例子：

.. code-block:: php

    <?php
    // src/AppBundle/Admin/PostAdmin.php

    namespace AppBundle\Admin;

    use Sonata\AdminBundle\Admin\AbstractAdmin;
    use Sonata\AdminBundle\Show\ShowMapper;
    use Sonata\AdminBundle\Form\FormMapper;
    use Sonata\AdminBundle\Datagrid\ListMapper;
    use Sonata\AdminBundle\Datagrid\DatagridMapper;

    class PostAdmin extends AbstractAdmin
    {
        // 在创建/编辑表单中显示的字段
        protected function configureFormFields(FormMapper $formMapper)
        {
            $formMapper
                ->add('title', 'text', array(
                    'label' => 'Post Title'
                ))
                ->add('author', 'entity', array(
                    'class' => 'AppBundle\Entity\User'
                ))

                // 如果没有类型被设定，SonataAdminBundle 会尝试猜测它
                ->add('body')

                // ...
           ;
        }

        // 显示在过滤器表单中的字段
        protected function configureDatagridFilters(DatagridMapper $datagridMapper)
        {
           $datagridMapper
                ->add('title')
                ->add('author')
           ;
        }

        // 在清单中显示的字段
        protected function configureListFields(ListMapper $listMapper)
        {
            $listMapper
                ->addIdentifier('title')
                ->add('slug')
                ->add('author')
           ;
        }

        // 在显示操作中显示的字段
        protected function configureShowFields(ShowMapper $showMapper)
        {
            $showMapper
               ->add('title')
               ->add('slug')
               ->add('author')
           ;
        }
    }

实现这四个函数是创建一个 Admin 类的第一步。也有其他选项，可以让你更深入的定义你模型的显示和处理。
这些会在本手册的高级章节涉及到。

1.3 创建一个 Admin 服务
-----------------------

现在你已经创建了你的 Admin 类，你需要为它创建一个服务。这个服务需要有 ``sonata.admin`` 标签，这是让
 SonataAdminBundle 知道这个特定服务器表示一个 Admin 类的方法：

在 ``src/AppBundle/Resources/config/`` 目录里创建一个新的 ``admin.xml`` 或 ``admin.yml`` 文件：

.. configuration-block::

    .. code-block:: xml

        <!-- src/AppBundle/Resources/config/admin.xml -->

        <service id="app.admin.post" class="AppBundle\Admin\PostAdmin">
            <tag name="sonata.admin" manager_type="orm" group="Content" label="Post" />
            <argument />
            <argument>AppBundle\Entity\Post</argument>
            <argument />
            <call method="setTranslationDomain">
                <argument>AppBundle</argument>
            </call>
        </service>

    .. code-block:: yaml

        # src/AppBundle/Resources/config/admin.yml

        services:
            app.admin.post:
                class: AppBundle\Admin\PostAdmin
                tags:
                    - { name: sonata.admin, manager_type: orm, group: "Content", label: "Post" }
                arguments:
                    - ~
                    - AppBundle\Entity\Post
                    - ~
                calls:
                    - [ setTranslationDomain, [AppBundle]]
                public: true

上边的例子假设你在使用 ``SonataDoctrineORMAdminBundle`` 。如果你使用 ``SonataDoctrineMongoDBAdminBundle`` , 
``SonataPropelAdminBundle`` 或 ``SonataDoctrinePhpcrAdminBundle`` ，分别设置 ``manager_type`` 选项为 
``doctrine_mongodb`` , ``propel`` 或 ``doctrine_phpcr`` 。

这个 Admin 服务的基本配置十分的简单。它创建了一个基于你之前设定的类的服务实例，并接受三个参数：

    1. Admin 服务的代码( 默认是服务的名称 )
    2. Admin 类所映射的模型( 必须的 )
    3. 用来管理后台操作的控制器( 默认是 ``SonataAdminBundle:CRUDController()`` )

通常你只需要设定第二个参数，因为第一个和第三个的默认值在大多数情况下都可以工作。

``setTranslationDomain`` 的调用让你选取一个翻译作用域，其用于后台管理页面的标签被翻译时。如果你没调用 
``setTranslationDomain`` ，SonataAdmin 使用 ``messages`` 作为翻译作用域。详见 Symfony 翻译页面
`Symfony translations page`_.

现在你有一个 admin 服务的配置文件了。你只需要让 Symfony 来加载它。有两个方法实现：

2.3.1 让你的 bundle 加载它
^^^^^^^^^^^^^^^^^^^^^^^^

在你的 bundle 的扩展文件里，如文档所述 `Symfony cookbook`_ ，使用 load() 方法

针对 ``admin.xml`` 的:

.. code-block:: php

    <?php
    // src/AppBundle/DependencyInjection/AppExtension.php

    namespace AppBundle\DependencyInjection;

    use Symfony\Component\HttpKernel\DependencyInjection\Extension;
    use Symfony\Component\DependencyInjection\ContainerBuilder;
    use Symfony\Component\DependencyInjection\Loader;
    use Symfony\Component\Config\FileLocator;

    class AppExtension extends Extension
    {
        public function load(array $configs, ContainerBuilder $container) {
            // ...
            $loader = new Loader\XmlFileLoader($container, new FileLocator(__DIR__.'/../Resources/config'));
            // ...
            $loader->load('admin.xml');
        }
    }

针对 ``admin.yml`` 的:

.. code-block:: php

    <?php
    // src/AppBundle/DependencyInjection/AppExtension.php

    namespace AppBundle\DependencyInjection;

    use Symfony\Component\HttpKernel\DependencyInjection\Extension;
    use Symfony\Component\DependencyInjection\ContainerBuilder;
    use Symfony\Component\DependencyInjection\Loader;
    use Symfony\Component\Config\FileLocator;

    class AppExtension extends Extension
    {
        public function load(array $configs, ContainerBuilder $container)
        {
            // ...
            $loader = new Loader\YamlFileLoader($container, new FileLocator(__DIR__.'/../Resources/config'));
            // ...
            $loader->load('admin.yml');
        }
    }

2.3.2 在主文件  config.yml 导入它
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

我们建议以扩展形式来加载此文件，但还有另外一个方法。

我们可以将你的新配置文件在主文件 ``config.yml`` 里引入进来 ( 请确保你使用了正确的文件扩展名 )：

.. configuration-block::

    .. code-block:: yaml

        # app/config/config.yml

        imports:

            # for xml
            - { resource: "@AppBundle/Resources/config/admin.xml" }

            # for yaml
            - { resource: "@AppBundle/Resources/config/admin.yml" }

2.4 配置
-------------

基于此，你有了针对你模型的基本的管理操作。如果你再次访问 ``http://yousite.local/admin/dashborad`` ，
你应该就可以看到你映射的模型的面板了。你可以开始创建，清列，编辑和删除实例了。

你或许想要将你项目的名称和 logo 放到上边栏。

将你的 logo 文件放到 ``/src/AppBndle/Resources/public/images/facy_acme_logo.png``

安装资源:

.. code-block:: bash

    $ php app/console assets:install

现在你可以修改项目的主文件 config.yml 文件：

.. configuration-block::

    .. code-block:: yaml

        # app/config/config.yml

        sonata_admin:
            title:      Acme
            title_logo: bundles/app/images/fancy_acme_logo.png

2.5 下一步 - 安全
---------------------

你或许已经注意到了，你只需要输入网址就可以访问你的仪表盘和数据了。默认，SonataAdminBundle 为了
高度的灵活性不做任何用户管理。然而，大多数情况你的程序其实需要这个特性的。Sonata 项目包含一个
 ``SonataUserBundle`` ，其整合了流行的 ``FOSUserBundle`` 。请参考文档的安全 :doc:`security` 
 部分来了解更多信息。

恭喜你！你已经开始使用 SonataAdminBundle 了。你现在可以映射额外的模型或探索高级功能了。下边的
每个部分都会针对这个 bundle 特定的功能或部分，深入详述 SonataAdminBundle 哪些可配置和哪些可实现。

.. _`Symfony cookbook`: http://symfony.com/doc/master/cookbook/bundles/extension.html#using-the-load-method
.. _`Symfony translations page`: http://symfony.com/doc/current/book/translation.html#using-message-domains
