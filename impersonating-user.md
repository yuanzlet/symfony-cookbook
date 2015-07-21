# 如何冒充一个用户

有时,无需登出和登入就能切换账户是很有用的（例如，当你调试或尝试理解别的用户的一个你无法复制的错误时）。这可以通过激活 **switch_user** 防火墙监听器来很容易地做到：

YAML:
```
# app/config/security.yml
security:
    firewalls:
        main:
            # ...
            switch_user: true
```

XML:
```
<!-- app/config/security.xml -->
<?xml version="1.0" encoding="UTF-8"?>
<srv:container xmlns="http://symfony.com/schema/dic/security"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:srv="http://symfony.com/schema/dic/services"
    xsi:schemaLocation="http://symfony.com/schema/dic/services
        http://symfony.com/schema/dic/services/services-1.0.xsd">
    <config>
        <firewall>
            <!-- ... -->
            <switch-user />
        </firewall>
    </config>
</srv:container>
```

PHP:
```
// app/config/security.php
$container->loadFromExtension('security', array(
    'firewalls' => array(
        'main'=> array(
            // ...
            'switch_user' => true
        ),
    ),
));
```

切换到另一个用户，只需添加一个带有 **the_switch_user** 参数和用户名为当前URL的查询字符串的值：

```
http://example.com/somewhere?_switch_user=thomas
```

切换回原来的用户，使用特殊的 **_exit** 用户名：

```
http://example.com/somewhere?_switch_user=_exit
```

在仿冒中，为用户提供一个特殊的角色，被称为 **ROLE_PREVIOUS_ADMIN**。在一个模板中，例如，这个角色可以用来显示退出仿冒的链接：

TWIG:
```
{% if is_granted('ROLE_PREVIOUS_ADMIN') %}
    <a href="{{ path('homepage', {'_switch_user': '_exit'}) }}">Exit impersonation</a>
{% endif %}
```

PHP:
```
<?php if ($view['security']->isGranted('ROLE_PREVIOUS_ADMIN')): ?>
    <a
        href="<?php echo $view['router']->generate('homepage', array(
            '_switch_user' => '_exit',
        ) ?>"
    >
        Exit impersonation
    </a>
<?php endif ?>
```

在某些情况下，你可能需要得到代表仿冒者而不是被仿冒用户的对象。使用以下代码来遍历用户的角色，直到你找到一个 **SwitchUserRole** 对象：

```
use Symfony\Component\Security\Core\Role\SwitchUserRole;

$authChecker = $this->get('security.authorization_checker');
$tokenStorage = $this->get('security.token_storage');

if ($authChecker->isGranted('ROLE_PREVIOUS_ADMIN')) {
    foreach ($tokenStorage->getToken()->getRoles() as $role) {
        if ($role instanceof SwitchUserRole) {
            $impersonatingUser = $role->getSource()->getUser();
            break;
        }
    }
}
```

当然，这个功能需要向一个小的用户群提供。默认情况下，有 **ROLE_ALLOWED_TO_SWITCH** 角色的用户的访问是被限制的。这个角色的名字可以通过**角色**设置进行修改。对于额外的安全性，您还可以通过**参数**设置更改查询参数名称：

YAML:
```
# app/config/security.yml
security:
    firewalls:
        main:
            # ...
            switch_user: { role: ROLE_ADMIN, parameter: _want_to_be_this_user }
```

XML:
```
<!-- app/config/security.xml -->
<?xml version="1.0" encoding="UTF-8"?>
<srv:container xmlns="http://symfony.com/schema/dic/security"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:srv="http://symfony.com/schema/dic/services"
    xsi:schemaLocation="http://symfony.com/schema/dic/services
        http://symfony.com/schema/dic/services/services-1.0.xsd">
    <config>
        <firewall>
            <!-- ... -->
            <switch-user role="ROLE_ADMIN" parameter="_want_to_be_this_user" />
        </firewall>
    </config>
</srv:container>
```

PHP:
```
// app/config/security.php
$container->loadFromExtension('security', array(
    'firewalls' => array(
        'main'=> array(
            // ...
            'switch_user' => array(
                'role' => 'ROLE_ADMIN',
                'parameter' => '_want_to_be_this_user',
            ),
        ),
    ),
));
```

## 事件
防火墙的 **security.switch_user** 事件在仿冒完成后处理。 [SwitchUserEvent](http://api.symfony.com/2.7/Symfony/Component/Security/Http/Event/SwitchUserEvent.html) 被传递给监听器，并且你可以用它来获得你仿冒的用户。

关于 [Making the Locale "Sticky" during a User's Session](http://symfony.com/doc/current/cookbook/session/locale_sticky_session.html) 的清单文本当你仿冒用户时在本地不进行修正。下面的代码示例将显示如何更改粘滞区域设置：


YAML：
```
# app/config/services.yml
services:
    app.switch_user_listener:
        class: AppBundle\EventListener\SwitchUserListener
        tags:
            - { name: kernel.event_listener, event: security.switch_user, method: onSwitchUser }
```

XML:
```
<!-- app/config/services.xml -->
<service id="app.switch_user_listener" class="AppBundle\EventListener\SwitchUserListener">
    <tag name="kernel.event_listener" event="security.switch_user" method="onSwitchUser" />
</service>
```

PHP:
```
// app/config/services.php
$container
    ->register('app.switch_user_listener', 'AppBundle\EventListener\SwitchUserListener')
    ->addTag('kernel.event_listener', array('event' => 'security.switch_user', 'method' => 'onSwitchUser'))
;
```

> 监听器实现假设你的 **User** 实体有 **getLocale()** 的方法。`

```
// src/AppBundle/EventListener/SwitchUserListener.pnp
namespace AppBundle\EventListener;

use Symfony\Component\Security\Http\Event\SwitchUserEvent;

class SwitchUserListener
{
    public function onSwitchUser(SwitchUserEvent $event)
    {
        $event->getRequest()->getSession()->set(
            '_locale',
            $event->getTargetUser()->getLocale()
        );
    }
}
```
