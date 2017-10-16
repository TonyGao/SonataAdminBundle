保存钩子
============

当一个 SonataAdmin 被提交处理时，会有一些事件被调用。一个是在任何持久化层交互之前，而另一个是在之后。
在编辑和创建操作的提交和验证之间也有 ``preValidate`` 事件被调用。此事件的名称如下：

- 创建的对象 : ``preValidate($object)`` / ``prePersist($object)`` / ``postPersist($object)``
- 编辑的对象 : ``preValidate($object)`` / ``preUpdate($object)`` / ``postUpdate($object)``
- 删除的对象 : ``preRemove($object)`` / ``postRemove($object)``

值得注意的是无论何时 Admin 成功提交了都会调用 update 事件，无论是否存在实际的持久化层事件。这与在 DoctrineORM 
或其他一些持久化层里使用 ``preUpdate`` 和 ``postUpdate`` 事件不同。

例如: 如果你提交一个编辑表单而没对表单的值做任何修改，那么数据库里不会做任何修改，那么 DoctrineORM 就不会
触发这个 **Entity** 类自己的 ``preUpdate`` 和 ``postUpdate`` 事件。然而，**Admin** 类的 ``preUpdate``
和 ``postUpdate`` 方法会被调用，这可以方便你使用。

.. note::

    当嵌套一个 Admin 到另一个里，例如使用 ``sonata_type_admin`` 字段类型，子管理的钩子是**不会**触发的。


使用 FOS/UserBundle 的例子
------------------------------------

``FOSUserBundle`` 为你的 Symfony 项目提供用户验证特性，并且兼容 Doctrine ORM, Doctrine ODM
和 Propel. 查阅 `FOSUserBundle on GitHub`_ 了解更多信息。

用户管理系统需要在用户密码或用户名更新时执行特定的调用。这就是 Admin bundle 能够用来解决这个问题的
的原因，通过 ``preUpdate`` 埋一个钩子。

.. code-block:: php

    <?php
    namespace FOS\UserBundle\Admin\Entity;

    use Sonata\AdminBundle\Admin\AbstractAdmin;
    use FOS\UserBundle\Model\UserManagerInterface;

    class UserAdmin extends AbstractAdmin
    {
        protected function configureFormFields(FormMapper $formMapper)
        {
            $formMapper
                ->with('General')
                    ->add('username')
                    ->add('email')
                    ->add('plainPassword', 'text')
                ->end()
                ->with('Groups')
                    ->add('groups', 'sonata_type_model', array('required' => false))
                ->end()
                ->with('Management')
                    ->add('roles', 'sonata_security_roles', array( 'multiple' => true))
                    ->add('locked', null, array('required' => false))
                    ->add('expired', null, array('required' => false))
                    ->add('enabled', null, array('required' => false))
                    ->add('credentialsExpired', null, array('required' => false))
                ->end()
            ;
        }

        public function preUpdate($user)
        {
            $this->getUserManager()->updateCanonicalFields($user);
            $this->getUserManager()->updatePassword($user);
        }

        public function setUserManager(UserManagerInterface $userManager)
        {
            $this->userManager = $userManager;
        }

        /**
         * @return UserManagerInterface
         */
        public function getUserManager()
        {
            return $this->userManager;
        }
    }

这个服务声明是将 ``UserManager`` 注入到 Admin 类。

.. configuration-block::

    .. code-block:: xml

        <service id="fos.user.admin.user" class="%fos.user.admin.user.class%">
            <tag name="sonata.admin" manager_type="orm" group="fos_user" />
            <argument />
            <argument>%fos.user.admin.user.entity%</argument>
            <argument />

            <call method="setUserManager">
                <argument type="service" id="fos_user.user_manager" />
            </call>
        </service>


在控制器里挂载钩子
-------------------------

你也许已经注意到了 **Admin** 里钩子不允许你与删除过程进行交互: 你不能取消它。要实现这个，
你应该知道还有一种在控制器里的操作挂载钩子的方式。

如果你通过继承 ``CRUDController`` 来自定义一个控制器，你可以重定义下边的方法：

- new object : ``preCreate($object)``
- edited object : ``preEdit($object)``
- deleted object : ``preDelete($object)``
- show object : ``preShow($object)``
- list objects : ``preList($object)``

如果这些方法返回一个 **Response** (响应)，那么此进程被中断，并由控制器返回此响应(如果返回 null, 该进程会继续)。
你可以通过 ``redirectTo($object)`` 方法轻易的实现到对象显示页面的从定向。

.. note::

    用例：您可能需要禁止删除特定的子项。你可以简单的看看 ``preDelete($object)`` 方法。


.. _FOSUserBundle on GitHub: https://github.com/FriendsOfSymfony/FOSUserBundle/
