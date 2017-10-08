列表视图
=============

.. note::

    本文档正在更新过程中。如果你读到了这里你可以帮助做出贡献，**无论你对 Sonata 有无经验** 。检出 
    `issues on GitHub`_ 了解关于怎样参与其中的更多信息。

这篇文档会涵盖在你的系统中查看用来查看对象的列表视图。它将涵盖此列表自身和控制哪些可见的过滤器的配置。

基础配置
-------------------

会影响列表视图的 SonataAdmin 选项：

.. code-block:: yaml

    sonata_admin:
        templates:
            list:                       SonataAdminBundle:CRUD:list.html.twig
            action:                     SonataAdminBundle:CRUD:action.html.twig
            select:                     SonataAdminBundle:CRUD:list__select.html.twig
            list_block:                 SonataAdminBundle:Block:block_admin_list.html.twig
            short_object_description:   SonataAdminBundle:Helper:short-object-description.html.twig
            batch:                      SonataAdminBundle:CRUD:list__batch.html.twig
            inner_list_row:             SonataAdminBundle:CRUD:list_inner_row.html.twig
            base_list_field:            SonataAdminBundle:CRUD:base_list_field.html.twig
            pager_links:                SonataAdminBundle:Pager:links.html.twig
            pager_results:              SonataAdminBundle:Pager:results.html.twig


.. note::

    **TODO**:
    * 一篇关于路由的注意事项，以及怎样关闭它们以关闭相关的操作
    * 添加自定义列

自定义列表页面上显示的字段
-------------------------------------------------

你可以通过 ``configureListFields`` 方法自定义列表上显示的列。
这里是一个示例：

.. code-block:: php

    <?php

    // ...

    public function configureListFields(ListMapper $listMapper)
    {
        $listMapper
            // addIdentifier 允许设定这个列提供一个到数据实体的链接
            // (编辑或显示路由, 取决于你的准入权限)
            ->addIdentifier('name')

            // 你可以直接将字段类型作为第二个参数来设定，以代替作为选项设定
            ->add('isVariation', 'boolean')

            // 如果没设定，会猜测这个类型
            ->add('enabled', null, array(
                'editable' => true
            ))

            // 可编辑的关联字段
            ->add('status', 'choice', array(
                'editable' => true,
                'class' => 'Vendor\ExampleBundle\Entity\ExampleStatus',
                'choices' => array(
                    1 => 'Active',
                    2 => 'Inactive',
                    3 => 'Draft',
                ),
            ))

            // 我们可以根据这个类型添加选项到这个字段
            ->add('price', 'currency', array(
                'currency' => $this->currencyDetector->getCurrency()->getLabel()
            ))

            // 这里我们设定用哪个属性在列表里渲染每个数据实体的翻译标签
            ->add('productCategories', null, array(
                'associated_property' => 'name')
            )

            // 你也可以使用点分隔符来访问一个相关数据实体的特定的属性
            ->add('image.name')

            // 你或许也想设定列表中想要显示的操作
            ->add('_action', null, array(
                'actions' => array(
                    'show' => array(),
                    'edit' => array(),
                    'delete' => array(),
                )
            ))

        ;
    }

选项
^^^^^^^

.. note::

    * ``(m)`` 代表必须的
    * ``(o)`` 代表选项

- ``type`` (m): 定义字段类型 - 为字段描述自身而必填的，但没设定时会试着自动探测类型
- ``template`` (o): 用来渲染字段的模板
- ``label`` (o): 用来做列标题的名称
- ``link_parameters`` (o): 当 ``Admin::generateUrl`` 被调用时添加链接参数到相关的 Admin 类
- ``code`` (o): 检索相关值的方法名(例如，如果你有一个 `array` 类型字段，你会想要比 `[0] => 'Value'` 
  更漂亮的方式显示信息；通常当一个简单的 getter 不够用时)。
  注意：只针对字符串类型(string, text, html)
- ``associated_property`` (o): 用来检索集合元素 "string" 表示的属性路径，或者使用此元素作为参数的闭包并
  返回一个字符串。
- ``identifier`` (o): 如果设定为 true，会在这个值上出现一个链接来编辑这个元素

可得的类型和相关选项
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. note::

    ``(m)`` 意味着这个选项是必填的

+-----------+----------------+-----------------------------------------------------------------------+
| 类型       | 选项           | 描述                                                                   |
+===========+================+=======================================================================+
| actions   | actions        | 可用操作列表                                                            |
+-----------+----------------+-----------------------------------------------------------------------+
| batch     |                | 渲染一个复选框                                                          |
+-----------+----------------+-----------------------------------------------------------------------+
| select    |                | 渲染一个单选框                                                          |
+-----------+----------------+-----------------------------------------------------------------------+
| array     |                | 显示一个数组                                                            |
+-----------+----------------+-----------------------------------------------------------------------+
| boolean   | ajax_hidden    | Yes/No; ajax_hidden 允许在 AJAX 上下文期间隐藏列表字段。                   |
+           +----------------+-----------------------------------------------------------------------+
|           | editable       | Yes/No; 如果权限允许 editable 允许直接从列表编辑。                         |
+           +----------------+-----------------------------------------------------------------------+
|           | inverse        | Yes/No; 反转背景颜色(假是绿色，真是红色)                                   |
+-----------+----------------+-----------------------------------------------------------------------+
| choice    | choices        | 可能的选项                                                              |
+           +----------------+-----------------------------------------------------------------------+
|           | multiple       | 是多选择选项吗？默认为 false 。                                           |
+           +----------------+-----------------------------------------------------------------------+
|           | delimiter      | 在多个值的情况下，作为分隔符。                                             |
+           +----------------+-----------------------------------------------------------------------+
|           | catalogue      | 翻译域                                                                 |
+           +----------------+-----------------------------------------------------------------------+
|           | class          | 可编辑关联字段的类路径。                                                  |
+-----------+----------------+-----------------------------------------------------------------------+
| currency  | currency (m)   | 一个现金字符串 (比如 EUR 或 USD)。                                        |
+-----------+----------------+-----------------------------------------------------------------------+
| date      | format         | 一种可以被 Twig 的 ``date`` 函数理解的格式。                               |
+-----------+----------------+-----------------------------------------------------------------------+
| datetime  | format         | 一种可以被 Twig 的 ``date`` 函数理解的格式。                               |
+-----------+----------------+-----------------------------------------------------------------------+
| email     | as_string      | 将邮箱作为字符串渲染，而没有任何链接。                                      |
+           +----------------+-----------------------------------------------------------------------+
|           | subject        | 为邮件链接添加标题参数。                                                  |
+           +----------------+-----------------------------------------------------------------------+
|           | body           | 为邮件链接添加内容参数。                                                  |
+-----------+----------------+-----------------------------------------------------------------------+
| percent   |                | 将一个值作为百分比渲染。                                                  |
+-----------+----------------+-----------------------------------------------------------------------+
| string    |                | 渲染一个简单的字符串。                                                    |
+-----------+----------------+-----------------------------------------------------------------------+
| text      |                | 同 'string'                                                           |
+-----------+----------------+-----------------------------------------------------------------------+
| html      |                | 将字符串作为 html 渲染                                                   |
+-----------+----------------+-----------------------------------------------------------------------+
| time      |                | 以 ``H:i:s`` 格式对一个日期时间进行渲染。                                  |
+-----------+----------------+-----------------------------------------------------------------------+
| trans     | catalogue      | 如果已定义，用域 ``catalogue`` 来翻译此值。                                |
+-----------+----------------+-----------------------------------------------------------------------+
| url       | url            | 为显示的值添加一个 url 为 ``url`` 的链接                                  |
+           +----------------+-----------------------------------------------------------------------+
|           | route          | 用于生成 url 的路由                                                     |
+           +                +                                                                       +
|           |   name         | 路由名称                                                               |
+           +                +                                                                       +
|           |   parameters   | 路由参数                                                               |
+           +----------------+-----------------------------------------------------------------------+
|           | hide_protocol  | 隐藏 http:// 或 https:// (默认为: false)                                |
+-----------+----------------+-----------------------------------------------------------------------+

如果你安装了 SonataDoctrineORMAdminBundle, 你可以使用更多的字段类型，详见文档
`SonataDoctrineORMAdminBundle Documentation <https://sonata-project.org/bundles/doctrine-orm-admin/master/doc/reference/list_field_definition.html>`_.

.. note::

    在可能的情况下，针对布尔值最好是非负选项，所以如果你没有更好的表达反义的词，
    那可以使用 ``inverse`` 选项。

自定义用于生成列表的查询
-----------------------------------------------

多亏了 ``createQuery`` 方法，你可以自定义列表的查询。

.. code-block:: php

    <?php

    public function createQuery($context = 'list')
    {
        $query = parent::createQuery($context);
        $query->andWhere(
            $query->expr()->eq($query->getRootAliases()[0] . '.my_field', ':my_param')
        );
        $query->setParameter('my_param', 'my_value');
        return $query;
    }


自定义排序顺序
--------------------------

在列表视图中配置默认排序
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

配置默认排序列可以简单的通过覆盖 ``datagridValues`` 数组属性来实现。所有的三个键 ``_page``, 
``_sort_order`` 和 ``_sort_by`` 可以省略。

.. code-block:: php

    <?php
    // src/AppBundle/Admin/PostAdmin.php

    use Sonata\AdminBundle\Admin\AbstractAdmin;

    class PostAdmin extends AbstractAdmin
    {
        // ...

        protected $datagridValues = array(

            // 显示第一页(默认是 1)
            '_page' => 1,

            // 相反的顺序 (默认是 'ASC')
            '_sort_order' => 'DESC',

            // 排序字段的名称(默认是模型的 id 字段，如果有的话)
            '_sort_by' => 'updatedAt',
        );

        // ...
    }

.. note::

    ``_sort_by`` 键可以是 ``mySubModel.mySubSubModel.myField`` 的形式。

.. note::

    **TODO** 怎么通过多字段进行排序(这可能是一个单独的秘诀？)

过滤器
-------

你可以添加过滤器来让用户控制显示哪些数据。

.. code-block:: php

    <?php
    // src/AppBundle/Admin/PostAdmin.php

    use Sonata\AdminBundle\Datagrid\DatagridMapper;

    class ClientAdmin extends AbstractAdmin
    {

        protected function configureDatagridFilters(DatagridMapper $datagridMapper)
        {
            $datagridMapper
                ->add('phone')
                ->add('email')
            ;
        }

        // ...
    }

为了节省空间所有过滤器默认都是隐藏的。用户需要选中你想要使用的过滤器。

想要让过滤器总是可见(甚至当它没开启时)，设定参数 ``show_filter`` 为 ``true`` 。

.. code-block:: php

    <?php

    protected function configureDatagridFilters(DatagridMapper $datagridMapper)
    {
        $datagridMapper
            ->add('phone')
            ->add('email', null, array(
                'show_filter' => true
            ))

            // ...
        ;
    }

默认情况下模板会为一个默认为 ``sonata_type_equal`` 的过滤器生成一个 ``操作符`` 。虽然这个 ``operator_type``
是被自动检测的，它可以被修改甚至被隐藏：

.. code-block:: php

    protected function configureDatagridFilters(DatagridMapper $datagridMapper)
    {
        $datagridMapper
            ->add('foo', null, array(
                'operator_type' => 'sonata_type_boolean'
            ))
            ->add('bar', null, array(
                'operator_type' => 'hidden'
            ))

            // ...
        ;
    }

如果你不需要高级过滤器，或者说所有的 ``operator_type`` 都是隐藏的，你可以通过设定 ``advanced_filter`` 为 false 来关闭它们。
你需要关闭所有高级过滤器来让按钮不可见。

.. code-block:: php

    protected function configureDatagridFilters(DatagridMapper $datagridMapper)
    {
        $datagridMapper
            ->add('bar', null, array(
                'operator_type' => 'hidden',
                'advanced_filter' => false
            ))

            // ...
        ;
    }

默认过滤器
^^^^^^^^^^^^^^^

可以使用 ``configureDefaultFilterValues`` 方法来将默认的过滤器添加到数据列表的值里。一个过滤器有一个 ``value`` 和一个选项的
``type`` 。如果没给定 ``type``, 那么默认被使用的的类型是 ``is equal`` 。

.. code-block:: php

    public function configureDefaultFilterValues(array &$filterValues)
    {
        $filterValues['foo'] = array(
            'type'  => ChoiceFilter::TYPE_CONTAINS,
            'value' => 'bar',
        );
    }

可用的类型是通过类表现的，可以在这里找到：
https://github.com/sonata-project/SonataCoreBundle/tree/master/Form/Type

类似 ``equal`` 和 ``boolean`` 的类型使用常量来赋值给这些 ``类型`` 的选项一个 ``正整型`` 来当做它的 ``值``:

.. code-block:: php

    <?php
    // SonataCoreBundle/Form/Type/EqualType.php

    namespace Sonata\CoreBundle\Form\Type;

    class EqualType extends AbstractType
    {
        const TYPE_IS_EQUAL = 1;
        const TYPE_IS_NOT_EQUAL = 2;
    }

然后这个正整数在列表操作 URL 被传入，如： 
**/admin/user/user/list?filter[enabled][type]=1&filter[enabled][value]=1**

这是一个为一个 ``boolean`` 类型使用这些常量的例子：

.. code-block:: php

    use Sonata\UserBundle\Admin\Model\UserAdmin as SonataUserAdmin;
    use Sonata\CoreBundle\Form\Type\EqualType;
    use Sonata\CoreBundle\Form\Type\BooleanType;

    class UserAdmin extends SonataUserAdmin
    {
        protected $datagridValues = array(
            'enabled' => array(
                'type'  => EqualType::TYPE_IS_EQUAL, // => 1
                'value' => BooleanType::TYPE_YES     // => 1
            )
        );
    }

请注意在一个 ``boolean`` 类型上设置一个 ``false`` 作为值是不能工作的，因为这个类型预期是用正整型 ``2`` 做为值，
如类常量中定义的那样：

.. code-block:: php

    <?php
    // SonataCoreBundle/Form/Type/BooleanType.php

    namespace Sonata\CoreBundle\Form\Type;

    class BooleanType extends AbstractType
    {
        const TYPE_YES = 1;
        const TYPE_NO = 2;
    }

默认的过滤器也可以通过覆盖 ``getFilterParameters`` 方法来添加到数据列表中。

.. code-block:: php

    use Sonata\CoreBundle\Form\Type\EqualType;
    use Sonata\CoreBundle\Form\Type\BooleanType;

    class UserAdmin extends SonataUserAdmin
    {
        public function getFilterParameters()
        {
            $this->datagridValues = array_merge(array(
                    'enabled' => array (
                        'type'  => EqualType::TYPE_IS_EQUAL,
                        'value' => BooleanType::TYPE_YES
                    )
                ), $this->datagridValues);

            return parent::getFilterParameters();
        }
    }

这个实现对于当你需要动态创建过滤器时很有用。

.. code-block:: php

    class PostAdmin extends SonataUserAdmin
    {
        public function getFilterParameters()
        {
            // 假设安全上下文已经注入了
            if (!$this->securityContext->isGranted('ROLE_ADMIN')) {
                $user = $this->securityContext->getToken()->getUser();

                $this->datagridValues = array_merge(array(
                        'author' => array (
                            'type'  => EqualType::TYPE_IS_EQUAL,
                            'value' => $user->getId()
                        )
                    ), $this->datagridValues);
            }

            return parent::getFilterParameters();
        }
    }

请注意这不是一个安全的实现，在发帖间隐藏彼此。这只是一个需要设置过滤器的例子。

回调过滤器
^^^^^^^^^^^^^^^

如果你已经安装了 **SonataDoctrineORMAdminBundle** ，你可以使用 ``doctrine_orm_callback`` 过滤器类型，
比如，用于创建一个全文本过滤器：

.. code-block:: php

    use Sonata\UserBundle\Admin\Model\UserAdmin as SonataUserAdmin;
    use Sonata\AdminBundle\Datagrid\DatagridMapper;

    class UserAdmin extends SonataUserAdmin
    {
        protected function configureDatagridFilters(DatagridMapper $datagridMapper)
        {
            $datagridMapper
                ->add('full_text', CallbackFilter::class, array(
                    'callback' => array($this, 'getFullTextFilter'),
                    'field_type' => 'text'
                ))

                // ...
            ;
        }

        public function getFullTextFilter($queryBuilder, $alias, $field, $value)
        {
            if (!$value['value']) {
                return;
            }

            // 使用 `andWhere` 而不是 `where` 来防止覆盖现有的 `where` 条件
            $queryBuilder->andWhere($queryBuilder->expr()->orX(
                $queryBuilder->expr()->like($alias.'.username', $queryBuilder->expr()->literal('%' . $value['value'] . '%')),
                $queryBuilder->expr()->like($alias.'.firstName', $queryBuilder->expr()->literal('%' . $value['value'] . '%')),
                $queryBuilder->expr()->like($alias.'.lastName', $queryBuilder->expr()->literal('%' . $value['value'] . '%'))
            ));

            return true;
        }
    }

你也可以获得用于帮助修改你的条件的操作符类型过滤器类型:

.. code-block:: php

    use Sonata\CoreBundle\Form\Type\EqualType;

    class UserAdmin extends SonataUserAdmin
    {
        public function getFullTextFilter($queryBuilder, $alias, $field, $value)
        {
            if (!$value['value']) {
                return;
            }

            $operator = $value['type'] == EqualType::TYPE_IS_EQUAL ? '=' : '!=';

            $queryBuilder
                ->andWhere($alias.'.username '.$operator.' :username')
                ->setParameter('username', $value['value'])
            ;

            return true;
        }
    }

.. note::

    **TODO**:
    * 基本过滤器配置和选项
    * 使用点分隔符号来定位子模型字段
    * 高级过滤器选项(global_search)

可视化配置
--------------------

你可以配置列表视图以自定义渲染，而不用父爱盖整个模板。你可以: 

- `header_style`: 自定义头部样式(宽度，颜色，背景，对齐...)
- `header_class`: 自定义头部的 class 
- `collapse`: 允许使用 "阅读更多" 链接来折叠长文本字段
- `row_align`: 自定义渲染内部单元格的对齐方式
- `label_icon`: 在标签之前添加一个图标

.. code-block:: php

    <?php

    public function configureListFields(ListMapper $list)
    {
        $list
            ->add('id', null, array(
                'header_style' => 'width: 5%; text-align: center',
                'row_align' => 'center'
            ))
            ->add('name', 'text', array(
                'header_style' => 'width: 35%'
            )
            ->add('description', 'text', array(
                'header_style' => 'width: 35%',
                'collapse' => true
            )
            ->add('upvotes', null, array(
                'label_icon' => 'fa fa-thumbs-o-up'
            )
            ->add('actions', null, array(
                'header_class' => 'customActions',
                'row_align' => 'right'
            )

            // ...
        ;
    }

如果你要自定义 `折叠` 选项，你也可以给定一个数组来覆盖默认的参数。

.. code-block:: php

            // ...
            ->add('description', 'text', array(
                'header_style' => 'width: 35%',
                'collapse' => array(
                    'height' => 40, // height in px
                    'read_more' => '我想要看全部描述', // "阅读更多"链接的内容
                    'read_less' => '这个文本太长了，缩短长度' // "阅读更少"链接的内容
                )
            )
            // ...

如果你只想显示 `label_icon`:

.. code-block:: php

            // ...
            ->add('upvotes', null, array(
                'label' => false,
                'label_icon' => 'fa fa-thumbs-o-up'
            )
            // ...

.. _`issues on GitHub`: https://github.com/sonata-project/SonataAdminBundle/issues/1519

马赛克视图按钮
------------------

你可以显示/隐藏马赛克视图按钮

.. code-block:: yaml

    sonata_admin:
        # 要在所有屏幕隐藏马赛克视图按钮，使用 `false`
        show_mosaic_button:   true

你可以使用 admin 服务配置来显示/隐藏马赛克视图按钮。你需要在你的 admin 服务里添加 ``show_mosaic_button`` 选项:

.. code-block:: yaml

    sonata_admin.admin.post:
        class: Sonata\AdminBundle\Admin\PostAdmin
        arguments: [~, Sonata\AdminBundle\Entity\Post, ~]
        tags:
            - { name: sonata.admin, manager_type: orm, group: admin, label: Post, show_mosaic_button: true }

    sonata_admin.admin.news:
        class: Sonata\AdminBundle\Admin\NewsAdmin
        arguments: [~, Sonata\AdminBundle\Entity\News, ~]
        tags:
            - { name: sonata.admin, manager_type: orm, group: admin, label: News, show_mosaic_button: false }

复选框范围选择
------------------------

.. tip::

    你可以通过单击第一个来选中/取消一个复选框范围，然后第二个通过 shift + 单击。
