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

Here, we declare the same ``Admin`` service as in the :doc:`getting_started` chapter, but using a
different label translator strategy, replacing the default one. Notice that
``sonata.admin.label.strategy.underscore`` is a service provided by ``SonataAdminBundle``,
but you could just as easily use a service of your own.

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

When extending ``CRUDController``, remember that the ``Admin`` class already has
a set of automatically injected dependencies that are useful when implementing several
scenarios. Refer to the existing ``CRUDController`` actions for examples of how to get
the best out of them.

当扩展 ``CRUDController`` 时，

In your overloaded CRUDController you can overload also these methods to limit
the number of duplicated code from SonataAdmin:
* ``preCreate``: called from ``createAction``
* ``preEdit``: called from ``editAction``
* ``preDelete``: called from ``deleteAction``
* ``preShow``: called from ``showAction``
* ``preList``: called from ``listAction``

These methods are called after checking the access rights and after retrieving the object
from database. You can use them if you need to redirect user to some other page under certain conditions.

Fields Definition
-----------------

Your ``Admin`` class defines which of your model's fields will be available in each
action defined in your ``CRUDController``. So, for each action, a list of field mappings
is generated. These lists are implemented using the ``FieldDescriptionCollection`` class
which stores instances of ``FieldDescriptionInterface``. Picking up on our previous
``PostAdmin`` class example:

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
        // Fields to be shown on create/edit forms
        protected function configureFormFields(FormMapper $formMapper)
        {
            $formMapper
                ->add('title', 'text', array(
                    'label' => 'Post Title'
                ))
                ->add('author', 'entity', array(
                    'class' => 'AppBundle\Entity\User'
                ))

                // if no type is specified, SonataAdminBundle tries to guess it
                ->add('body')

                // ...
            ;
        }

        // Fields to be shown on filter forms
        protected function configureDatagridFilters(DatagridMapper $datagridMapper)
        {
            $datagridMapper
                ->add('title')
                ->add('author')
            ;
        }

        // Fields to be shown on lists
        protected function configureListFields(ListMapper $listMapper)
        {
            $listMapper
                ->addIdentifier('title')
                ->add('slug')
                ->add('author')
            ;
        }

        // Fields to be shown on show action
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

Internally, the provided ``Admin`` class will use these three functions to create three
``FieldDescriptionCollection`` instances:

* ``$formFieldDescriptions``, containing three ``FieldDescriptionInterface`` instances
  for title, author and body
* ``$filterFieldDescriptions``, containing two ``FieldDescriptionInterface`` instances
  for title and author
* ``$listFieldDescriptions``, containing three ``FieldDescriptionInterface`` instances
  for title, slug and author
* ``$showFieldDescriptions``, containing four ``FieldDescriptionInterface`` instances
  for id, title, slug and author

The actual ``FieldDescription`` implementation is provided by the storage abstraction
bundle that you choose during the installation process, based on the
``BaseFieldDescription`` abstract class provided by ``SonataAdminBundle``.

Each ``FieldDescription`` contains various details about a field mapping. Some of
them are independent of the action in which they are used, like ``name`` or ``type``,
while others are used only in specific actions. More information can be found in the
``BaseFieldDescription`` class file.

In most scenarios, you will not actually need to handle the ``FieldDescription`` yourself.
However, it is important that you know it exists and how it is used, as it sits at the
core of ``SonataAdminBundle``.

Templates
---------

Like most actions, ``CRUDController`` actions use view files to render their output.
``SonataAdminBundle`` provides ready to use views as well as ways to easily customize them.

The current implementation uses ``Twig`` as the template engine. All templates
are located in the ``Resources/views`` directory of the bundle.

There are two base templates, one of these is ultimately used in every action:

* ``SonataAdminBundle::standard_layout.html.twig``
* ``SonataAdminBundle::ajax_layout.html.twig``

Like the names say, one if for standard calls, the other one for AJAX.

The subfolders include Twig files for specific sections of ``SonataAdminBundle``:

Block:
  ``SonataBlockBundle`` block views. By default there is only one, which
  displays all the mapped classes on the dashboard
Button:
  Buttons such as ``Add new`` or ``Delete`` that you can see across several
  CRUD actions
CRUD:
  Base views for every CRUD action, plus several field views for each field type
Core:
  Dashboard view, together with deprecated and stub twig files.
Form:
  Views related to form rendering
Helper:
  A view providing a short object description, as part of a specific form field
  type provided by ``SonataAdminBundle``
Pager:
  Pagination related view files

These will be discussed in greater detail in the specific :doc:`templates` section, where
you will also find instructions on how to configure ``SonataAdminBundle`` to use your templates
instead of the default ones.

Managing ``Admin`` Service
--------------------------

Your ``Admin`` service definitions are parsed when Symfony is loaded, and handled by
the ``Pool`` class. This class, available as the ``sonata.admin.pool`` service from the
DIC, handles the ``Admin`` classes, lazy-loading them on demand (to reduce overhead)
and matching each of them to a group. It is also responsible for handling the top level
template files, administration panel title and logo.

Create child admins
-------------------

Let us say you have a ``PlaylistAdmin`` and a ``VideoAdmin``. You can optionally declare the ``VideoAdmin``
to be a child of the ``PlaylistAdmin``. This will create new routes like, for example, ``/playlist/{id}/video/list``,
where the videos will automatically be filtered by post.

To do this, you first need to call the ``addChild`` method in your ``PlaylistAdmin`` service configuration:

.. configuration-block::

    .. code-block:: xml

        <!-- app/config/config.xml -->
        <service id="sonata.admin.playlist" class="AppBundle\Admin\PlaylistAdmin">
            <!-- ... -->

            <call method="addChild">
                <argument type="service" id="sonata.admin.video" />
            </call>
        </service>

Then, you have to set the VideoAdmin ``parentAssociationMapping`` attribute to ``playlist`` :

.. code-block:: php

    <?php

    namespace AppBundle\Admin;

    // ...

    class VideoAdmin extends AbstractAdmin
    {
        protected $parentAssociationMapping = 'playlist';

        // OR

        public function getParentAssociationMapping()
        {
            return 'playlist';
        }
    }

To display the ``VideoAdmin`` extend the menu in your ``PlaylistAdmin`` class:

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

It also possible to set a dot-separated value, like ``post.author``, if your parent and child admins are not directly related.

Be wary that being a child admin is optional, which means that regular routes
will be created regardless of whether you actually need them or not. To get rid
of them, you may override the ``configureRoutes`` method::

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

                // This is the route configuration as a child
                $collection->clearExcept(['show', 'edit']);

                return;
            }

            // This is the route configuration as a parent
            $collection->clear();

        }
    }

.. _`Django Project Website`: http://www.djangoproject.com/
.. _`CRUD`: http://en.wikipedia.org/wiki/CRUD
