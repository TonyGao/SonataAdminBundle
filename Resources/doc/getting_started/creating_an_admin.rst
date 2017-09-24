创建一个后台管理
=================

你现在可以通过前一章 :doc:`the previous
chapter <installation>` 提到的管理后台界面工作起来了， 你得学习告知 
SonataAdmin 怎么管理你的模型。

步骤 0：创建一个模型
----------------------

因为教程其余部分的需要，你需要一些模型。在本教程，将用到两个非常简单的
 ``Post`` 和 ``Tag`` 数据实体。通过这些命令来生成它们：

.. code-block:: bash

    $ php bin/console doctrine:generate:entity --entity="AppBundle:Category" --fields="name:string(255)" --no-interaction
    $ php bin/console doctrine:generate:entity --entity="AppBundle:BlogPost" --fields="title:string(255) body:text draft:boolean" --no-interaction

此后，你将需要稍微修改一下这些数据实体：

.. code-block:: php

    // src/AppBundle/Entity/BlogPost.php

    // ...
    class BlogPost
    {
        // ...

        /**
         * @ORM\ManyToOne(targetEntity="Category", inversedBy="blogPosts")
         */
        private $category;

        public function setCategory(Category $category)
        {
            $this->category = $category;
        }

        public function getCategory()
        {
            return $this->category;
        }

        // ...
    }

设置默认值为 ``false``.

.. code-block:: php

    // src/AppBundle/Entity/BlogPost.php

    // ...
    class BlogPost
    {
        // ...

        /**
         * @var bool
         *
         * @ORM\Column(name="draft", type="boolean")
         */
        private $draft = false;

        // ...
    }

.. code-block:: php


    // src/AppBundle/Entity/Category.php

    // ...
    use Doctrine\Common\Collections\ArrayCollection;
    // ...

    class Category
    {
        // ...

        /**
        * @ORM\OneToMany(targetEntity="BlogPost", mappedBy="category")
        */
        private $blogPosts;

        public function __construct()
        {
            $this->blogPosts = new ArrayCollection();
        }

        public function getBlogPosts()
        {
            return $this->blogPosts;
        }

        // ...
    }

此后，创建这些数据实体的数据表结构：

.. code-block:: bash

    $ php bin/console doctrine:schema:create

.. note::

    这篇文章假设你有基本的 Doctrine 2 ORM 知识，并且你已经正确安装了一个数据库。

步骤 1：创建一个管理后台类
-----------------------------

SonataAdminBundle 将帮助你通过图形界面来管理你的数据，你可以创建、更新或搜索你的模型实例。
这个 bundle 依赖于管理后台类，来让它知道将管理哪个模型，以及这些操作将是什么样子。

这个 Admin 类决定了哪些字段将被显示到列表里，哪些字段将被用来做筛选，以及创建的表单看起来是
什么样子。每个模型都有它自己的 Admin 类。

知道了这些，我们来创建一个 ``Category`` 数据实体的 ``Admin`` 类吧。 最简单的方法是扩展自
 ``Sonata\AdminBundle\Admin\AbstractAdmin``.

.. code-block:: php

    // src/AppBundle/Admin/CategoryAdmin.php
    namespace AppBundle\Admin;

    use Sonata\AdminBundle\Admin\AbstractAdmin;
    use Sonata\AdminBundle\Datagrid\ListMapper;
    use Sonata\AdminBundle\Datagrid\DatagridMapper;
    use Sonata\AdminBundle\Form\FormMapper;

    class CategoryAdmin extends AbstractAdmin
    {
        protected function configureFormFields(FormMapper $formMapper)
        {
            $formMapper->add('name', 'text');
        }

        protected function configureDatagridFilters(DatagridMapper $datagridMapper)
        {
            $datagridMapper->add('name');
        }

        protected function configureListFields(ListMapper $listMapper)
        {
            $listMapper->addIdentifier('name');
        }
    }

那么，这些代码做了什么呢？

* **11-14 行**: 这些行配置了哪些字段将在编辑和创建操作时显示。 ``FormMapper`` 
的表现类似于 Symfony 表单组件的 ``FormBuilder``;
* **16-19 行**: 这个方法配置了过滤器，用来过滤和排序模型的列表;
* **Line 21-24**: 这里设定了哪些字段用来在所有模型都列出来后用于显示它们(
 ``addIdentifier()`` 方法意味着这个字段将链接到这个特定模型的查看/编辑页面 )。

这是 Admin 类最基本的例子。你可以用 Admin 类做更多的配置。这些将在其他更高级的文章里涵盖。

步骤3：登记 Admin 类
--------------------------------

你现在已经创建了一个 Admin 类，但现在还没方法让 SonataAdminBundle 知道这个 Admin 
类的存在。要想告诉 SonataAdminBundle 这个 Admin 类的存在，你得创建一个服务，
并给它打上 ``sonata.admin`` 的标签:

.. code-block:: yaml

    # app/config/services.yml

    services:
        # ...
        admin.category:
            class: AppBundle\Admin\CategoryAdmin
            arguments: [~, AppBundle\Entity\Category, ~]
            tags:
                - { name: sonata.admin, manager_type: orm, label: Category }
            public: true

基础的 Admin 类的构造器就有很多参数。SonataAdminBundle 提供了一个编译器参数，它用来为你
正确的配置参数。你可以经常通过标签属性来修改东西。这段代码展示了要程序跑起来的最短代码。

步骤 4: 登记 SonataAdmin 自定义路由
------------------------------------------

SonataAdminBundle 在运行是为 Admin 类生成路由。要加载这个路由，你得确保 SonataAdminBundle 的
路由加载器运行起来了： 

.. code-block:: yaml

    # app/config/routing.yml

    # ...
    _sonata_admin:
        resource: .
        type: sonata_admin
        prefix: /admin

显示 Category 的管理界面
---------------------------------

现在已经创建了 Category 的管理类了，你大概想要看看后台管理界面里它是啥样子的。
好，我们通过访问 http://localhost:8000/admin 来看看

.. image:: ../images/getting_started_category_dashboard.png

不要拘束，随便看看，添加一些类型，如 “Symfony” 和 “Sonata Project”。
在下一章，我们将创建 ``BlogPost`` 数据实体的管理后台，并学习这个类的更多知识。

.. tip::

    如果你没看到这个漂亮的标签，但却看到一些类似”link_add“的东西，你得看看你是不是
    开启了 `translator`_.

.. _`translator`: http://symfony.com/doc/current/book/translation.html#configuration
