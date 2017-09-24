3. 表单视图
=============

在 :doc:`the previous chapter <creating_an_admin>` 你已经看到冰山一角了。 
但还有好多需要讨论的东西！在接下来的章节，你会为更加复杂的 ``BlogPost`` 模型
创建一个 Admin 类。 就是说，你将学习怎么让这个事更漂亮的完成。

3.1 启动 Admin 类
-----------------------------

基本的类定义和 ``CategoryAdmin`` 类似：

.. code-block:: php

    // src/AppBundle/Admin/BlogPostAdmin.php
    namespace AppBundle\Admin;

    use Sonata\AdminBundle\Admin\AbstractAdmin;
    use Sonata\AdminBundle\Datagrid\ListMapper;
    use Sonata\AdminBundle\Form\FormMapper;

    class BlogPostAdmin extends AbstractAdmin
    {
        protected function configureFormFields(FormMapper $formMapper)
        {
            // ... configure $formMapper
        }

        protected function configureListFields(ListMapper $listMapper)
        {
            // ... configure $listMapper
        }
    }

服务定义也是一样的：

.. code-block:: yaml

    # app/config/services.yml

    services:
        # ...
        admin.blog_post:
            class: AppBundle\Admin\BlogPostAdmin
            arguments: [~, AppBundle\Entity\BlogPost, ~]
            tags:
                - { name: sonata.admin, manager_type: orm, label: Blog post }
            public: true

3.2 配置表单映射器
---------------------------

如果你已经了解过了 `Symfony 表单组件`_, ``FormMapper`` 跟它非常类似。

你可以使用 ``add()`` 方法添加字段到表单里。第一参数是这个字段值要映射到的属性的名称，
第二个参数是字段的类型( 查阅 `字段类型参考手册`_ ) 第三个参数是用来自定义表单类型的
额外选项。表单组件只强制要求第一个参数，因为它可有类型猜测器来猜测它的类型。

``BlogPost`` 模型有 4 个属性： ``id``, ``title``, ``body``,
``category`` 。`id` 是由数据库自动生成的。这就是说表单视图只需要3个字段：
title, body 和 category 。

title 和 body 字段只是简单的 “text” 和 “textarea” 类型， 你可以把它们直接添加上：

.. code-block:: php

    // src/AppBundle/Admin/BlogPostAdmin.php

    // ...
    protected function configureFormFields(FormMapper $formMapper)
    {
        $formMapper
            ->add('title', 'text')
            ->add('body', 'textarea')
        ;
    }

然而，category 字段会涉及到另一个模型。你怎么解决它呢？

3.3 添加涉及其他模型的字段
-----------------------------------------

对于添加涉及其他模型的字段你有很多不同的选择。最基础的选择是使用 DoctrineBundle 提供的
 `数据实体字段类型`_ 。这会将可得的数据作为选项来渲染一个下拉选框。

.. code-block:: php

    // src/AppBundle/Admin/BlogPostAdmin.php

    // ...
    protected function configureFormFields(FormMapper $formMapper)
    {
        $formMapper
            // ...
            ->add('category', 'entity', array(
                'class' => 'AppBundle\Entity\Category',
                'property' => 'name',
            ))
        ;
    }
.. note::

    `property`_ 选项在 Symfony 2.7 以上的版本不被支持。你应该使用 `choice_label`_ 来代替。

因为每个 blog 都会有一个类型，所以它渲染为一个选择下拉框：

.. image:: ../images/getting_started_entity_type.png

当一个管理员要创建一个新的类型时，他需要进入类型的管理页面，然后创建一个新的类型。

3.3.1 使用 Sonata 模型类型
~~~~~~~~~~~~~~~~~~~~~~~~~~~

要让事情变得容易些，你可以使用:ref:`sonata_type_model field type <field-types-model>`
(字段类型)。这个字段类型仍然会作为一个下拉选框渲染出来，但它会包含一个创建按钮来打开一个对话框来
管理相关的模型：

.. code-block:: php

    // src/AppBundle/Admin/BlogPostAdmin.php

    // ...
    protected function configureFormFields(FormMapper $formMapper)
    {
        $formMapper
            // ...
            ->add('category', 'sonata_type_model', array(
                'class' => 'AppBundle\Entity\Category',
                'property' => 'name',
            ))
        ;
    }

.. image:: ../images/getting_started_sonata_model_type.png

3.4 使用分组
------------

当前，所有的东西都放到一个 block 里。表单仅支持树状字段，现在它还可用，但很快它会变成难以维护。
要解决这个问题，表单映射器支持将字段进行分组。

例如，title 和  body 字段可以属于 Content 分组，而 category 字段属于 Meta 数据分组。
用 ``with()`` 方法来实现：

.. code-block:: php

    // src/AppBundle/Admin/BlogPostAdmin.php

    // ...
    protected function configureFormFields(FormMapper $formMapper)
    {
        $formMapper
            ->with('Content')
                ->add('title', 'text')
                ->add('body', 'textarea')
            ->end()

            ->with('Meta data')
                ->add('category', 'sonata_type_model', array(
                    'class' => 'AppBundle\Entity\Category',
                    'property' => 'name',
                ))
            ->end()
        ;
    }

第一个参数是这个分组的名称或标签名，第二个参数是一个选项的数组。例如，你可以传一个 HTML class 
到分组里来修改它们的样式：

.. code-block:: php

    // src/AppBundle/Admin/BlogPostAdmin.php

    // ...
    protected function configureFormFields(FormMapper $formMapper)
    {
        $formMapper
            ->with('Content', array('class' => 'col-md-9'))
                // ...
            ->end()
            ->with('Meta data', array('class' => 'col-md-3')
                // ...
            ->end()
        ;
    }

这会让编辑页面变得更漂亮：

.. image:: ../images/getting_started_post_edit_grid.png

3.4.1 使用标签
~~~~~~~~~~

如果你想用更多的选项，你可以用 ``tab()`` 方法来划分多个标签：

.. code-block:: php

    $formMapper
        ->tab('Post')
            ->with('Content', ...)
                // ...
            ->end()
            // ...
        ->end()

        ->tab('Publish Options')
            // ...
        ->end()
    ;

3.5 创建一篇博文
--------------------

现在你要为这个 ``BlogPost`` 模型的漂亮表单视图做个结束动作了。现在是时候创建一个博客来
测试一下了。

在按了 "Create" 按钮之后，你可能会看到一个绿色的消息，类似：
*子项 "AppBundle\Entity\BlogPost:00000000192ba93c000000001b786396" 已经创建成功。*

虽然 SonataAdminBundle 给管理员做了非常友好的创建提示，但是类名和哈希却并不好读。这是 
SonataAdminBundle 里的对象的默认字符串表达。你可以通过在 Admin 类里定义一个 ``toString()`` 
方法来修改它。这个会接要转换为字符串的对象作为第一个参数：

.. note::

    没有下划线前缀！ ``toString()`` 是正确的！

.. code-block:: php

    // src/AppBundle/Admin/BlogPostAdmin.php

    // ...
    use AppBundle\Entity\BlogPost;

    class BlogPostAdmin extends AbstractAdmin
    {
        // ...

        public function toString($object)
        {
            return $object instanceof BlogPost
                ? $object->getTitle()
                : 'Blog Post'; // shown in the breadcrumb on the create view
        }
    }

3.6 总结
--------

在这个教程里，你已经第一次领略了 SonataAdminBundle 的最棒的特性：可以逐一自定义一切事物。
从创建一个简单的表单开始，一直到实现一个漂亮的管理编辑页面结束。

在下一章， :doc:`next chapter <the_list_view>`, 你会对清单和数据列表操作一探究竟。

.. _`Symfony 表单组件`: http://symfony.com/doc/current/book/forms.html
.. _`字段类型参考手册`: http://symfony.com/doc/current/reference/forms/types.html
.. _`数据实体字段类型`: http://symfony.com/doc/current/reference/forms/types/entity.html
.. _`choice_label`: http://symfony.com/doc/current/reference/forms/types/entity.html#choice-label
.. _`property`: http://symfony.com/doc/2.6/reference/forms/types/entity.html#property
