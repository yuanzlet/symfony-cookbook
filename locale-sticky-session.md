# 在用户的 Session 中使用局部 "Sticky"

在 Symfony 2.1 之前，本地设置被存储在被称为 **_locale** 的会话的属性中。从 2.1 版本开始，它储存在 Request 里，这意味着在一次用户请求中它不是“粘性的（sticky）”，在本文中，您将学习如何让用户的本地设置“粘性（sticky）”，一旦设定，相同的本地设置将被用于所有后续请求。

## 创建一个 LocaleListener

为了模拟储存在一个会话中的本地设置，您需要创建并注册一个新的事件监听器。为了模拟储存在一个会话中的本地设置，您需要创建并注册一个新的事件监听器。监听器是像这样的东西。通常情况下，**_locale** 被用作一个路由参数来表示本地设置，虽然它并不影响你如何确定一个请求所需的设置。

PHP:

```PHP
// src/AppBundle/EventListener/LocaleListener.php
namespace AppBundle\EventListener;

use Symfony\Component\HttpKernel\Event\GetResponseEvent;
use Symfony\Component\HttpKernel\KernelEvents;
use Symfony\Component\EventDispatcher\EventSubscriberInterface;

class LocaleListener implements EventSubscriberInterface
{
    private $defaultLocale;

    public function __construct($defaultLocale = 'en')
    {
        $this->defaultLocale = $defaultLocale;
    }

    public function onKernelRequest(GetResponseEvent $event)
    {
        $request = $event->getRequest();
        if (!$request->hasPreviousSession()) {
            return;
        }

        // try to see if the locale has been set as a _locale routing parameter
        if ($locale = $request->attributes->get('_locale')) {
            $request->getSession()->set('_locale', $locale);
        } else {
            // if no explicit locale has been set on this request, use one from the session
            $request->setLocale($request->getSession()->get('_locale', $this->defaultLocale));
        }
    }

    public static function getSubscribedEvents()
    {
        return array(
            // must be registered before the default Locale listener
            KernelEvents::REQUEST => array(array('onKernelRequest', 17)),
        );
    }
}
```

然后注册监听器。

YAML:

```YAML
services:
    app.locale_listener:
        class: AppBundle\EventListener\LocaleListener
        arguments: ["%kernel.default_locale%"]
        tags:
            - { name: kernel.event_subscriber }
```

XML:

```XML
<service id="app.locale_listener"
    class="AppBundle\EventListener\LocaleListener">
    <argument>%kernel.default_locale%</argument>

    <tag name="kernel.event_subscriber" />
</service>
```

```PHP
use Symfony\Component\DependencyInjection\Definition;

$container
    ->setDefinition('app.locale_listener', new Definition(
        'AppBundle\EventListener\LocaleListener',
        array('%kernel.default_locale%')
    ))
    ->addTag('kernel.event_subscriber')
;
```

好了！现在通过改变用户设置并查看它在所有请求中都是粘性的（sticky）。记住，想要得到用户设置，使用 **Request::getLocale** 这个方法。

PHP:

```PHP
// from a controller...
use Symfony\Component\HttpFoundation\Request;

public function indexAction(Request $request)
{
    $locale = $request->getLocale();
}
```

## 根据用户的喜好设置本地设置

您可能希望进一步提高该技术，并且以已登录用户的用户实体为依据定义本地设置。然而，由于 **LocaleListener** 比负责处理身份验证和设置具有 **TokenStorage** 的用户的 **FirewallListener** 早调用，您无法访问已登录用户。

假设您已经在 **User** 实体上定义了 **locale** 属性并且您想用此作为特定用户的本地设置。要做到这一点，您可以在登录过程和更新用户会话被重定向到它们的的第一个页面之前，用这个本地设置值将它们挂钩连接（hook into）。

要做到这一点，您需要对 **security.interactive_login** 事件添加一个事件监听器。

PHP:

```PHP
// src/AppBundle/EventListener/UserLocaleListener.php
namespace AppBundle\EventListener;

use Symfony\Component\HttpFoundation\Session\Session;
use Symfony\Component\Security\Http\Event\InteractiveLoginEvent;

/**
 * Stores the locale of the user in the session after the
 * login. This can be used by the LocaleListener afterwards.
 */
class UserLocaleListener
{
    /**
     * @var Session
     */
    private $session;

    public function __construct(Session $session)
    {
        $this->session = $session;
    }

    /**
     * @param InteractiveLoginEvent $event
     */
    public function onInteractiveLogin(InteractiveLoginEvent $event)
    {
        $user = $event->getAuthenticationToken()->getUser();

        if (null !== $user->getLocale()) {
            $this->session->set('_locale', $user->getLocale());
        }
    }
}
```

然后注册监听器。

YAML:

```YAML
# app/config/services.yml
services:
    app.user_locale_listener:
        class: AppBundle\EventListener\UserLocaleListener
        arguments: [@session]
        tags:
            - { name: kernel.event_listener, event: security.interactive_login, method: onInteractiveLogin }
```

XML:

```XML
<!-- app/config/services.xml -->
<?xml version="1.0" encoding="UTF-8" ?>
<container xmlns="http://symfony.com/schema/dic/services"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://symfony.com/schema/dic/services
        http://symfony.com/schema/dic/services/services-1.0.xsd">

    <services>
        <service id="app.user_locale_listener"
            class="AppBundle\EventListener\UserLocaleListener">

            <argument type="service" id="session"/>

            <tag name="kernel.event_listener"
                event="security.interactive_login"
                method="onInteractiveLogin" />
        </service>
    </services>
</container>
```

PHP:

```PHP
// app/config/services.php
$container
    ->register('app.user_locale_listener', 'AppBundle\EventListener\UserLocaleListener')
    ->addArgument('session')
    ->addTag(
        'kernel.event_listener',
        array('event' => 'security.interactive_login', 'method' => 'onInteractiveLogin'
    );
```

> 为了在用户更改语言偏好后立即更新语言，您需要在更新 **User** 实体后更新会话。