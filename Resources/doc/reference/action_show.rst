显示操作
===============

.. note::

    本文档正在更新过程中。如果你读到了这里你可以帮助做出贡献，**无论你对 Sonata 有无经验** 。检出 
    `issues on GitHub`_ 了解关于怎样参与其中的更多信息。

这篇文档将涵盖显示操作和相关的配置选项。

基础配置
-------------------

.. note::

    **TODO**:
    * 关于路由的一个注意事项，以及怎样关闭它们来关闭禁用相关的操作
    * 关于删除操作触发的生命周期事件的一个注意事项
    * 当添加一般字段时的可用选项，包括自定义模板
    * 使用点号分隔符定位子模型字段
    * (注意， 如果这跟表单文档类似，可以合并它)

分组选项
~~~~~~~~~~~~~

当添加一个分组到你的显示页面，你可能会为分组自身设定一些选项。

- ``collapsed``: 此刻还没用
- ``class``: 在 admin 里你分组的 css 类; 默认，这个值设定为 ``col-md-12`` 。
- ``fields``: 你分组里的字段(你不应该覆盖这个，除非你知道你在做什么)。
- ``box_class``: 在 admin 里你分组容器的 css 类; 默认，这个值设定为 ``box box-primary``.
- ``description``: 待完善
- ``translation_domain``: 待完善

要设定选项，像下边这么做:

.. code-block:: php

    <?php
    // src/AppBundle/Admin/PersonAdmin.php
    
    use Sonata\AdminBundle\Admin\AbstractAdmin;
    use Sonata\AdminBundle\Show\ShowMapper;

    class PersonAdmin extends AbstractAdmin
    {
        public function configureShowFields(ShowMapper $showMapper)
        {
            $showMapper
                ->tab('General') // 标签调用是可选的
                    ->with('Addresses', array(
                        'class'       => 'col-md-8',
                        'box_class'   => 'box box-solid box-danger',
                        'description' => 'Lorem ipsum',
                    ))
                        ->add('title')
                        // ...
                    ->end()
                ->end()
            ;
        }
    }

当扩展一个已存在的 Admin，你可能想要删除一些字段，分组或标签页。这里是一个实现的例子:

.. code-block:: php

    <?php
    // src/AppBundle/Admin/PersonAdmin.php
    
    use Sonata\AdminBundle\Show\ShowMapper;

    class PersonAdmin extends ParentAdmin
    {
        public function configureShowFields(ShowMapper $showMapper)
        {
            parent::configureShowFields($showMapper);

            // 仅删除一个字段
            $showMapper->remove('field_to_remove');

            // 从 "default" 标签删除一个分组
            $showMapper->removeGroup('GroupToRemove1');

            // 从特定标签删除一个分组
            $showMapper->removeGroup('GroupToRemove2', 'Tab2');

            // 从一个特定的标签删除一个分组，如果删空了，也把标签页删除
            $showMapper->removeGroup('GroupToRemove3', 'Tab3', true);
        }
    }

自定义用于在 Admin 类里显示对象的查询
--------------------------------------------------------------------------

创建一个 showAction 几乎和一个表单一样，就像我们在初始化安装时做的那样。

它其实更容易些，因为我们只关心显示信息。

松口气吧，困难的部分已经完成了。

下边是一个 ShowAction 的例子

.. code-block:: php

    <?php
    // src/AppBundle/Admin/PostAdmin.php

    use Sonata\AdminBundle\Show\ShowMapper;

    class ClientAdmin extends AbstractAdmin
    {
        protected function configureShowFields(ShowMapper $showMapper)
        {
            // 这里我们设定 ShowMapper 变量的字段，
            // $showMapper (其实它可以起任何名称)
            $showMapper

                 // 默认选项仅仅是将值作为文本显示(布尔值会是 1 或 0)
                ->add('name')
                ->add('phone')
                ->add('email')

                 // 布尔值选项其实非常酷
                 // true 显示一个选定的标识以及一个 'yes' 标签
                 // false 显示一个选定标识以及一个 'no' 标签
                ->add('dateCafe', 'boolean')
                ->add('datePub', 'boolean')
                ->add('dateClub', 'boolean')
            ;

        }
    }

.. tip::
    要自定义一个显示字段显示的标签，你可以使用 ``label`` 选项:

    .. code-block:: php
    
        $showMapper->add('name', null, array('label' => 'UserName'));

    设定这个选项为 ``false`` 将标签置空。

设定一个自定义显示模板(非常有用)
===============================================

首先你需要在 app/config/config/yml 里定义它:

.. configuration-block::

    .. code-block:: yaml

        sonata_admin:
            title:      Acme
            title_logo: img/logo_small.png
            templates:
                show:       AppBundle:Admin:Display_Client.html.twig

一旦定义好了，Sonata Admin 会在以下位置寻找它:

``src/AppBundle/Resources/views/Admin/Display_Client.html.twig``

现在你已经告诉 Sonata Admin 去哪里寻找模板，现在放一个在那里吧。

推荐你拷贝一个默认模板过去作为基础。

这确保你可以更新 Sonata Admin 并保证你的辛勤付出。

原版的模板可以在以下位置找到:

``vendor/sonata-project/admin-bundle/Resources/views/CRUD/base_show.html.twig``

现在你已经有一个默认模板了，检查一下确保它可以工作。

就这些，现在去写代码吧。

.. _`issues on GitHub`: https://github.com/sonata-project/SonataAdminBundle/issues/1519
