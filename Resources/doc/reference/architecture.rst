4. 结构
============

``SonataAdminBundle`` 的架构主要受 Django Admin 项目的启发，它真的是一个伟大的项目。
详见 Django 项目网站 `Django Project Website`_.

如果你跟着 :doc:`getting_started` 的页面照做了，你应该已经有一个 ``Admin`` 类和一个 ``Admin`` 服务了。
在本章，我们会更深入的讨论它的工作原理。

4.1 Admin 类
---------------

``Admin`` 类映射一个特定的模型到一个由 ``SonataAdminBundle`` 提供的富 CRUD 界面。换言之，使用你的 
Admin 类，你可以配置 SonataAdminBundle 在每个 CRUD 操作中为每个相关的模型显示哪些字段。
现在你会看到``开始``页面里有三个操作：清单、过滤器和表单( 用于创建/编辑 )。然而，一个完整的配置过的 Admin 类
可以定义更多的操作：

=============       =========================================================================
操作                 描述
=============       =========================================================================
list 清单            在清单表格中显示的字段
filter 过滤器         用于过滤清单内容的字段
form 表单            用于创建/编辑数据实体的字段
show 显示            用于显示数据实体的字段
批量操作              可以用于一组数据实体执行的操作( 如，批量删除 )
=============       =========================================================================


``Sonata\AdminBundle\Admin\AbstractAdmin`` 类是提供的一个映射你数据模型的简单方法，只要扩展它就可以了。
然而，任何 ``Sonata\AdminBundle\Admin\AdminInterface`` 的实现都可以被用来定义一个 ``Admin`` 服务。每个 
``Admin`` 服务，下边的依赖需求都会自动由此 Bundle 注入：

=========================       =========================================================================
类                              描述
=========================       =========================================================================
ConfigurationPool               配置池，所有 Admin 类实例存储的地方
ModelManager                    是一个服务，用于处理涉及到你持久化层的特定代码( 持久化层，如 Doctrine ORM ) 
FormContractor                  使用 Symfony 的 ``FormBuilder`` 构建表单的编辑/创建的视图
ShowBuilder                     构建显示字段
ListBuilder                     构建清单的字段
DatagridBuilder                 构建过滤器的字段
Request                         收到的 http 请求
RouteBuilder                    允许你为新的操作添加添加路由，并且为默认操作删除路由
RouterGenerator                 生成不同的 URL
SecurityHandler                 为模型实例和操作处理权限
Validator                       处理模型验证
Translator                      生成翻译
LabelTranslatorStrategy         当生成标签是所使用的策略
MenuFactory                     生成侧边菜单，取决于当前的操作
=========================       =========================================================================

.. note::

    这些依赖关系的每一个都用于一个特定的任务，如上所述。如果你想学习更多它们的用法，就查阅文档的各自的
    章节。大多数情况，你不需要担心它们的底层实现。


所有这些依赖的默认值都可以通过声明你自己的 ``Admin`` 服务来覆盖。这是通过调用一个 ``call`` 来匹配 ``setter``
来完成的：

.. configuration-block::

    .. code-block:: xml

        <service id="app.admin.post" class="AppBundle\Admin\PostAdmin">
              <tag name="sonata.admin" manager_type="orm" group="Content" label="Post" />
              <argument />
              <argument>AppBundle\Entity\Post</argument>
              <argument />
              <call method="setLabelTranslatorStrategy">
                  <argument type="service" id="sonata.admin.label.strategy.underscore" />
              </call>
          </service>

    .. code-block:: yaml

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
                    - [ setLabelTranslatorStrategy, ["@sonata.admin.label.strategy.underscore"]]
                public: true

这里，我们声明同样的 ``Admin`` 服务，如 :doc:`getting_started` 章节中，但使用了不同的标签翻译器策略，
以替换默认的策略。注意 ``sonata.admin.label.strategy.underscore`` 是由 ``SonataAdminBundle`` 提供的服务，
但您可以轻易的使用自己的服务。

4.2 CRUDController
--------------

``CRUDController`` 包含操作模型实例的操作，如创建、列表、编辑或删除。它使用 ``Admin`` 类来确定其行为，比如在编辑表单中
显示哪些字段，或怎么构建列表视图。在 ``CRUDController`` 内部，你可以通过 ``$admin`` 变量进入 ``Admin`` 类实例。

.. note::

    `CRUD`_ 是 “Create, Read, Update 和 Delete” 的首字母缩写

``CRUDController`` 与其他 Symfony 控制器没什么不同，这意味着你可以使用所有常用选项，如通过依赖注入容器(DIC)来
获得服务。

如果你决定扩展 ``CRUDController`` 来添加新的操作或修改已有操作的行为，这将非常有用。当声明 ``Admin`` 服务时，将第三个参数
用于设定使用哪个控制器。例如设置控制器为 ``AppBundle:PostAdmin``：

.. configuration-block::

    .. code-block:: xml

        <service id="app.admin.post" class="AppBundle\Admin\PostAdmin">
            <tag name="sonata.admin" manager_type="orm" group="Content" label="Post" />
            <argument />
            <argument>AppBundle\Entity\Post</argument>
            <argument>AppBundle:PostAdmin</argument>
            <call method="setTranslationDomain">
                <argument>AppBundle</argument>
            </call>
        </service>

    .. code-block:: yaml

        services:
            app.admin.post:
                class: AppBundle\Admin\PostAdmin
                tags:
                    - { name: sonata.admin, manager_type: orm, group: "Content", label: "Post" }
                arguments:
                    - ~
                    - AppBundle\Entity\Post
                    - AppBundle:PostAdmin
                calls:
                    - [ setTranslationDomain, [AppBundle]]
                public: true

当扩展 ``CRUDController`` 时，请记住 ``Admin`` 类已经有了很多自动注入的依赖，在很多情况下它们都
很有用。以当前的 ``CRUDController`` 操作为例怎么能最大限度的利用它们。

如果你在重载 CRUDController ，你可以通过 SonataAdmin 重载这些方法来限定重复代码的数量：
* ``preCreate``: 从 ``createAction`` 调用
* ``preEdit``: 从 ``editAction`` 盗用
* ``preDelete``: 从 ``deleteAction`` 调用
* ``preShow``: 从 ``showAction`` 调用
* ``preList``: 从 ``listAction`` 调用

在检查准入权限以及从数据库获取对象之后调用这些方法。如果您需要在某些条件下将用户重定向到某个其他页面，则可以使用它们。

字段定义
-----------------

Your ``Admin`` class defines which of your model's fields will be available in each
action defined in your ``CRUDController``. So, for each action, a list of field mappings
is generated. These lists are implemented using the ``FieldDescriptionCollection`` class
which stores instances of ``FieldDescriptionInterface``. Picking up on our previous
``PostAdmin`` class example:

你的 ``Admin`` 类定义了在你 ``CRUDController`` 中定义的每个操作中哪些模型字段可用。所以，针对每个操作，
都会生成一个字段映射列表。这些列表使用 ``FieldDescriptionCollection`` 类实现，其存储了 
``FieldDescriptionInterface`` 的实例。以我们之前的 ``PostAdmin`` 类为例：

.. code-block:: php

    <?php
    // src/AppBundle/Admin/PostAdmin.php

    namespace AppBundle\Admin;

    use Sonata\AdminBundle\Admin\AbstractAdmin;
    use Sonata\AdminBundle\Datagrid\ListMapper;
    use Sonata\AdminBundle\Datagrid\DatagridMapper;
    use Sonata\AdminBundle\Form\FormMapper;
    use Sonata\AdminBundle\Show\ShowMapper;

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

                // 如果没设定类型，SonataAdminBundle 会猜测它
                ->add('body')

                // ...
            ;
        }

        // 在过滤器表单里显示的字段
        protected function configureDatagridFilters(DatagridMapper $datagridMapper)
        {
            $datagridMapper
                ->add('title')
                ->add('author')
            ;
        }

        // 在列表中显示的字段
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
                ->add('id')
                ->add('title')
                ->add('slug')
                ->add('author')
            ;
        }
    }

在内部，提供的 ``Admin`` 类将使用这三个函数来创建三个 ``FieldDescriptionCollection`` 实例：

* ``$formFieldDescriptions``, 包含三个 ``FieldDescriptionInterface`` 实例, title, author 和 body
* ``$filterFieldDescriptions``, 包含两个 ``FieldDescriptionInterface`` 实例, title 和 author 
* ``$listFieldDescriptions``, 包含三个 ``FieldDescriptionInterface`` 实例, title, slug 和 author
* ``$showFieldDescriptions``, 包含四个 ``FieldDescriptionInterface`` 实例, id, title, slug 和 author

实际的 ``FieldDescription`` 实现由按安装过程中你选取的存储抽象 bundle 提供, 基于 ``BaseFieldDescription``
 的抽象类由 ``SonataAdminBundle`` 提供。

每个 ``FieldDescription`` 包含关于一个字段映射的各种细节。它们中的一些是独立于它们使用的操作之外的，
如 ``name`` 或 ``type`` , 而其他的仅用于特定的操作。更多信息可以在 ``BaseFieldDescription`` 类
文件中找到。

在大多情况下，你都不需要自己实际处理 ``FieldDescription`` 。然而，你还是得知道它的存在以及怎么用它，因为它
对 ``SonataAdminBundle`` 至关重要。

模板
---------

就像大多数操作，``CRUDController`` 操作使用视图文件来渲染它们的输出。``SonataAdminBundle`` 提供了
开箱即用的视图以及轻松自定义它们的方法。

当前的实现使用了 ``Twig`` 作为模板引擎。所有的模板都位于此 bundle 的 ``Resources/views`` 目录下。

有两个基础模板，其中一个最终用于每个操作：

* ``SonataAdminBundle::standard_layout.html.twig``
* ``SonataAdminBundle::ajax_layout.html.twig``

如名字一样，其中一个是为标准调用准备的，另一个是为 Ajax 的。

子文件夹包含了 ``SonataAdminBundle`` 的特定部分的 Twig 文件：

Block:
  ``SonataBlockBundle`` block 视图。默认只有一个，其在控制面板显示所有已映射的类
Button:
  按钮，如 ``添加`` 或 ``删除`` 这些跨多个 CRUD 操作的 
CRUD:
  每个 CRUD 操作的基础视图，为每个字段类型添加几个字段视图
Core:
  控制面板视图，结合已放弃和残余的 twig 文件。
  Dashboard view, together with deprecated and stub twig files.
Form:
  设计表单渲染的视图
Helper:
  提供一个简短的对象描述的视图，作为 ``SonataAdminBundle`` 提供的一个特定表单字段的一部分
Pager:
  分页相关的视图文件

这些都会在 :doc:`templates` 模板文档详细描述，你还会找到配置 ``SonataAdminBundle`` 来使用自定义模板的方法。

管理 ``Admin`` 服务
--------------------------

你的 ``Admin`` 服务定义会在 Symofony 加载时被解析，并被 ``Pool`` 类处理。这个类可以从依赖注入容器的 
``sonata.admin.pool`` 服务获取，来处理 ``Admin`` 类，在后台懒加载它们(为了减少负载)，并把它们匹配到
各自的分组。它也可以负责处理顶级模板文件，管理员面板标题和logo 。

创建子管理
-------------------

假设你有一个 ``PlaylistAdmin`` 和一个 ``VideoAdmin`` 。你可以选择声明 ``VideoAdmin`` 是 ``PlaylistAdmin`` 的子管理。
这将创建新的路由，如 ``/playlist/{id}/video/list``, 那么这些视频将自动由发帖来过滤。

要做到这点，你首先需要调用 ``PlaylistAdmin`` 服务配置里的这个 ``addChild`` 方法

.. configuration-block::

    .. code-block:: xml

        <!-- app/config/config.xml -->
        <service id="sonata.admin.playlist" class="AppBundle\Admin\PlaylistAdmin">
            <!-- ... -->

            <call method="addChild">
                <argument type="service" id="sonata.admin.video" />
            </call>
        </service>

然后，你需要设定 VideoAdmin ``parentAssociationMapping`` 属性为 ``playlist`` :

.. code-block:: php

    <?php

    namespace AppBundle\Admin;

    // ...

    class VideoAdmin extends AbstractAdmin
    {
        protected $parentAssociationMapping = 'playlist';

        // 或

        public function getParentAssociationMapping()
        {
            return 'playlist';
        }
    }

要显示 ``VideoAdmin`` 就在你的 ``PlaylistAdmin`` 类里扩展菜单

.. code-block:: php

    <?php

    namespace AppBundle\Admin;

    use Knp\Menu\ItemInterface as MenuItemInterface;
    use Sonata\AdminBundle\Admin\AbstractAdmin;
    use Sonata\AdminBundle\Admin\AdminInterface;

    class PlaylistAdmin extends AbstractAdmin
    {
        // ...

        protected function configureSideMenu(MenuItemInterface $menu, $action, AdminInterface $childAdmin = null)
        {
            if (!$childAdmin && !in_array($action, array('edit', 'show'))) {
                return;
            }

            $admin = $this->isChild() ? $this->getParent() : $this;
            $id = $admin->getRequest()->get('id');

            $menu->addChild('View Playlist', array('uri' => $admin->generateUrl('show', array('id' => $id))));

            if ($this->isGranted('EDIT')) {
                $menu->addChild('Edit Playlist', array('uri' => $admin->generateUrl('edit', array('id' => $id))));
            }

            if ($this->isGranted('LIST')) {
                $menu->addChild('Manage Videos', array(
                    'uri' => $admin->generateUrl('sonata.admin.video.list', array('id' => $id))
                ));
            }
        }
    }

如果你的父与子管理不是直接相关的，也可以设定点分隔值，如 ``post.author`` 。

小心子管理只是一个选项，这意味着常规的路由将被创建，无论你真的是否需要它。要摆脱它，你需要
覆盖 ``configureRoutes`` 方法::

    <?php
    
    namespace AppBundle\Admin;

    use Sonata\AdminBundle\Admin\AbstractAdmin;
    use Sonata\AdminBundle\Route\RouteCollection;

    class VideoAdmin extends AbstractAdmin
    {
        protected $parentAssociationMapping = 'playlist';

        protected function configureRoutes(RouteCollection $collection)
        {
            if ($this->isChild()) {

                // 这是作为子管理的路由配置
                $collection->clearExcept(['show', 'edit']);

                return;
            }

            // 这是作为父管理的路由配置
            $collection->clear();

        }
    }

.. _`Django Project Website`: http://www.djangoproject.com/
.. _`CRUD`: http://en.wikipedia.org/wiki/CRUD
