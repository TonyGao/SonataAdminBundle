4. 清单视图
=============

你已经给了管理员一个漂亮的界面来创建和编辑博客和类型。但如果没有所有博客数据的清单列表对于
修改来说还不够 。因此，本章将教你更多关于清单视图的东西。


如果你已经创建一个一些博客，并访问 http://localhost:8000/admin/app/blogpost/list, 
你会看到一个空的列表页面。这不是因为没有内容，而是因为你还没配置你的 Admin 的清单视图。
因为 Sonata 不知道哪些字段应该被显示，因此它只显示空行。

4.1 配置清单映射器
---------------------------

处理上边的问题很简单：只需要将你想要显示在清单页面的字段添加到清单视图就可以了：

.. code-block:: php

    // src/AppBundle/Admin/BlogPostAdmin.php
    namespace AppBundle\Admin;

    // ...
    class BlogPostAdmin extends AbstractAdmin
    {
        // ...

        protected function configureListFields(ListMapper $listMapper)
        {
            $listMapper
                ->add('title')
                ->add('draft')
            ;
        }
    }

再次进入博客管理的清单视图，你会看到可得的博客了：

.. image:: ../images/getting_started_basic_list_view.png

你可以看到 Sonata 已经猜出了一个正确的字段类型，并以一种友好的方式将它呈现出来。比如布尔
类型的 draft 字段会呈现为红色的”no”块，表示 ``false``.

太酷了！但一个管理员怎么能从这个页面进入一个博客的编辑页面呢？并没有类似链接这样的东西出现。
这是对的，你需要告诉 Sonata 哪个字段你想要作为一个链接使用。

4.1.1 定义一个 id 字段
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

包含一个进入编辑页面的链接的字段被称为 id 字段。将 title 字段链接到编辑页面很合理，所以
你可以添加它作为一个 id 字段。通过使用  ``ListMapper#addIdentifier()``
代替 ``ListMapper#add()`` 来实现：

.. code-block:: php

    // src/AppBundle/Admin/BlogPostAdmin.php
    namespace AppBundle\Admin;

    // ...
    class BlogPostAdmin extends AbstractAdmin
    {
        // ...

        protected function configureListFields(ListMapper $listMapper)
        {
            $listMapper
                ->addIdentifier('title')
                ->add('draft')
            ;
        }
    }

保存后，你就可以看到 title 字段有了你在寻找的链接。

4.1.2 显示其他模型
~~~~~~~~~~~~~~~~~~~~~~~

现在你或许想要 Category 也被包含到清单里。要实现，你需要关联它。你不能添加 ``category`` 字段
到这个清单映射器，因为它会试着将数据实体显示为一个字符串。如你在前面一章学的，也不建议你添加
 ``__toString()`` 到这个数据实体。

幸运的是，有个简单的方法来关联其他模型，就是通过使用点符号。用这个符号，你可以设定哪个字段你想要
显示。例如， ``category.name`` 将显示这个类型的 ``name`` 属性。

.. code-block:: php

    // src/AppBundle/Admin/BlogPostAdmin.php
    namespace AppBundle\Admin;

    // ...
    class BlogPostAdmin extends AbstractAdmin
    {
        // ...

        protected function configureListFields(ListMapper $listMapper)
        {
            $listMapper
                ->addIdentifier('title')
                ->add('category.name')
                ->add('draft')
            ;
        }
    }

4.2 添加过滤器/搜索选项
----------------------------

假设你有一个非常成功的博客，它有很多博客数量。有时你想要找到一篇博客来编辑它如同大海捞针。出于
用户体验的考虑，Sonata为此提供了一个解决方案！

它通过给你开放数据列表过滤器的配置来解决，在``Admin#configureDatagridFilters()`` 
方法里配置。例如，如果你想允许管理员通过 title 来搜索博客( 以及通过字母顺序来排序列表 )，
你应该这么写：

.. code-block:: php

    // src/AppBundle/Admin/BlogPostAdmin.php
    namespace AppBundle\Admin;

    use Sonata\AdminBundle\Datagrid\DatagridMapper;

    // ...
    class BlogPostAdmin extends AbstractAdmin
    {
        protected function configureDatagridFilters(DatagridMapper $datagridMapper)
        {
            $datagridMapper->add('title');
        }
    }

这会在这个 block 的左侧添加一个小的 block 来显示一个 title 字段的搜索输入框。

4.2.1 通过类型过滤
~~~~~~~~~~~~~~~~~~~~~

通过其他模型的属性来过滤稍微麻烦一点。add的部分需要添加5个参数的：

.. code-block:: php

    public function add(
        $name,

        // filter
        $type = null,
        array $filterOptions = array(),

        // field
        $fieldType = null,
        $fieldOptions = null
    )

如你所见，你可以自定义用于过滤器的类型和用于显示搜索字段的类型。你可以依赖于 Sonata 的类型
猜测机制来选出正确的字段类型。然而，你仍然需要使用类型的 ``name`` 属性来配置搜索字段：

.. code-block:: php

    // src/AppBundle/Admin/BlogPostAdmin.php
    namespace AppBundle\Admin;

    use Sonata\AdminBundle\Datagrid\DatagridMapper;

    // ...
    class BlogPostAdmin extends AbstractAdmin
    {
        protected function configureDatagridFilters(DatagridMapper $datagridMapper)
        {
            $datagridMapper
                ->add('title')
                ->add('category', null, array(), 'entity', array(
                    'class'    => 'AppBundle\Entity\Category',
                    'choice_label' => 'name', // In Symfony2: 'property' => 'name'
                ))
            ;
        }
    }

用这段代码，一个下拉框将会显示出来，它包含了所有可得的类型。这让通过类型进行过滤变得简单了。

.. image:: ../images/getting_started_filter_category.png

4.3 总结
--------

此刻，你已经学习了怎么让查找和编辑博客变得更轻松。你已经学习了怎么创建一个漂亮的清单视图以及
怎么添加选项来搜索，排序和过滤这个清单。

也许这个过程有些困难，但想想如果这些都自己写的难度！因为你现在已经有一定基础了，你可以开始
阅读这份文档的其他章节了，如：

* :doc:`Customizing the Dashboard <../reference/dashboard>`
* :doc:`Configuring the Security system <../reference/security>`
* :doc:`Adding export functionality <../reference/action_export>`
* :doc:`Adding a preview page <../reference/preview_mode>`
