创建和编辑对象
============================

.. note::

    本文档正在更新过程中。如果你读到了这里你可以帮助做出贡献，**无论你对 Sonata 有无经验** 。检出 
    `issues on GitHub`_ 了解关于怎样参与其中的更多信息。

这篇文档会涵盖在你的系统中查看用来查看对象的列表视图。它将涵盖此列表自身和控制哪些可见的过滤器的配置。


基本配置
-------------------

对创建和编辑视图起作用的 SonataAdmin 选项:

.. code-block:: yaml

    sonata_admin:
        options:
            html5_validate:    true     # 开启或关闭 html5 表单验证
            confirm_exit:      true     # 开启或关闭在导航到其他地方之前的一个确认
            use_select2:       true     # 开启或关闭使用  Select2 jQuery 库
            use_icheck:        true     # 开启或关闭使用 iCheck 库
            use_bootlint:      false    # 开启或关闭使用 Bootlint
            use_stickyforms:   true     # 开启或关闭浮动按钮
            form_type:         standard # 也可以是 'horizontal' (横向)

        templates:
            edit:              SonataAdminBundle:CRUD:edit.html.twig
            tab_menu_template: SonataAdminBundle:Core:tab_menu_template.html.twig


有关可选库的更多信息:

- Select2: https://github.com/select2/select2
- iCheck: http://icheck.fronteed.com/
- Bootlint: https://github.com/twbs/bootlint#in-the-browser


.. note::

    **TODO**:
    * 当添加字段时可用的选项，包括自定义模板

路由
~~~~~~

你可以通过在你的 admin 里删除相应的路由关闭创建或编辑数据实体。关于路由的详细信息，查阅 :doc:`routing` 。

.. code-block:: php

    <?php
    // src/AppBundle/Admin/PersonAdmin.php

    class PersonAdmin extends AbstractAdmin
    {
        // ...

        protected function configureRoutes(RouteCollection $collection): void
        {
            /* 
             * 删除编辑的路由会关闭编辑数据实体。它将使用 '显示' 视图作为列表视图里的唯一标识列的默认链接。
             */
            $collection->remove('edit');

            /*
             * 删除创建的路由将关闭创建新的数据实体。它会在列表视图中删除 '添加新的' 按钮。
             */
            $collection->remove('create');
        }

        // ...
    }

添加表单字段
------------------

用 configureFormFields 方法你可以定义哪些字段应该在编辑或创建数据实体时呈现。
每个字段应该添加到一个特定的表单分组里。而表单分组可选地添加到一个标签页。查看
`FormGroup options`_ 了解更多关于配置表单分组的信息。

使用 FormMapper add 方法，你可以添加表单字段。add 方法有4个参数:

- ``name``: 你数据实体的名称。
- ``type``: 要显示的字段的类型; 默认的 ``null`` 是让 Sonata 来决定使用哪个类型。
  查阅 :doc:`Field Types <field_types>` 来了解更多关于可用类型的信息。
- ``options``: 用于字段的表单选项。可能每个类型都不同。查阅 :doc:`Field Types <field_types>`
  了解更多可用选项的信息。
- ``fieldDescriptionOptions``: 字段描述选项。这里的选项会传入到字段模板中。查阅 
  :ref:`Form Types, FieldDescription options <form_types_fielddescription_options>`
  了解更多。

.. note::

    在 ``name`` 中输入的属性应该在您的数据实体中通过 getter/setter 或 public 准入而可用。


.. code-block:: php

    <?php
    // src/AppBundle/Admin/PersonAdmin.php

    class PersonAdmin extends AbstractAdmin
    {
        // ...

        protected function configureFormFields(FormMapper $formMapper): void
        {
            $formMapper
                ->tab('General') // 标签调用是可选的
                    ->with('Addresses')
                        ->add('title') // 添加一个字段并让 Sonata 决定该用哪个类型
                        ->add('streetname', TextType::class) // 添加一个文本字段
                        ->add('housenumber', NumberType::class) // 添加一个数字字段
                        ->add('housenumberAddition', TextType::class, ['required' => false]) // 添加一个不是必需的文本字段
                    ->end() // 结束表单分组
                ->end() // 结束标签
            ;
        }

        // ...
    }


表单分组选项
~~~~~~~~~~~~~~~~~

当添加一个表单分组到你的编辑/创建表单时，你可能要为分组本身设定一些选项。

- ``collapsed``: 此刻还不使用
- ``class``: 在 admin 中表单分组的css类; 默认这个值会设定为 ``col-md-12`` 。
- ``fields``: 在你表单分组里的字段 (按说你不应该覆盖这个，除非你知道你在干什么)。
- ``box_class``: 在 admin 里分组容器的 css 类; 默认这个值会设定为 ``box box-primary`` 。
- ``description``: 在表单分组顶部显示的文本。
- ``translation_domain``: 表单分组标题的翻译域(默认会使用 Admin 的翻译域)。

要设定选项，如下这样做：

.. code-block:: php

    <?php
    // src/AppBundle/Admin/PersonAdmin.php

    class PersonAdmin extends AbstractAdmin
    {
        // ...

        public function configureFormFields(FormMapper $formMapper): void
        {
            $formMapper
                ->tab('General') // 标签调用是可选的
                    ->with('Addresses', [
                        'class'       => 'col-md-8',
                        'box_class'   => 'box box-solid box-danger',
                        'description' => 'Lorem ipsum',
                        // ...
                    ])
                        ->add('title')
                        // ...
                    ->end()
                ->end()
            ;
        }

        // ...
    }

对于分组的自定义 box_class 你可以这么做，这里是个例子:

.. figure:: ../images/box_class.png
   :align: center
   :alt: Box Class
   :width: 500

嵌入其他 admin
----------------------

.. note::

    **TODO**:
    * 怎么一个 admin 里嵌入另一个(1:1, 1:M, M:M)
    * 怎么从嵌入的 admin 代码进入正确的对象

仅自定义操作中的一个
-----------------------------------

.. note::

    **TODO**:
    * 怎么创建仅出现在一个创建/编辑的设定/字段
    * 以及需要管理任何控制器变动

.. _`issues on GitHub`: https://github.com/sonata-project/SonataAdminBundle/issues/1519