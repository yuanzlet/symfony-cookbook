# 如何创建自定义认证提供者

> 创建一个人自定义的身份验证系统很难，本章将引导您完成这一进程。但根据您的需求，您可能可以通过一种更简单的， 或者通过集群包来解决这个问题:

> - [如何创建一个自定义表单的密码验证器](http://symfony.com/doc/current/cookbook/security/custom_password_authenticator.html)

> - [如何使用 API 来进行用户身份验证](http://symfony.com/doc/current/cookbook/security/api_key_authentication.html)

> - 如果要通过 OAuth 来使用第三方平台的服务，如谷歌、 Facebook 或者 Twitter 进行身份验证，那么请尝试使用 [HWIOAuthBundle](https://github.com/hwi/HWIOAuthBundle) 集群包。

如果您读过关于[安全](http://symfony.com/doc/current/book/security.html)的那一章，那么您已经了解了在实现安全性的过程中 Symfony 对身份处理和授权的不同处理方式。本章将讨论在身份验证过程中所涉及的核心类以及如何实现一个自定义的身份验证提供程序。因为身份验证和授权是单独的概念，此扩展将会成为未知的用户提供程序，并将与您的应用程序中的用户提供程序一同运行，并且它们可能建立在内存，数据库，或任何其它您选择来存储它们的地方的基础上。

## 满足 WSSE

接下来的一章将演示如何为 WSSE 身份验证创建一个自定义的身份验证提供程序。WSSE 的安全协议提供了几个安全性利益:

1\. 用户名 / 密码加密

2\. 安全的防范再次攻击

3\. 不需要 web 服务器配置

用 WSSE 来保护 SOAP 或 REST 架构的 web 服务非常有效 。

目前有很多关于 [WSSE](http://www.xml.com/pub/a/2003/12/17/dive.html) 的文档，本文不会重点讲解安全协议，而是讲解如何把一个自定义的协议添加到您的 Symfony 应用程序中。WSSE 的基础是:使用请求标头来检查加密凭据，使用时间戳和[随机数](https://en.wikipedia.org/wiki/Cryptographic_nonce)来进行验证，使用密码摘要来为发出请求的用户进行身份验证。

> WSSE 还支持应用程序密钥验证，这对 web 服务非常有用，但是该内容超出了本章的介绍范围。

## 令牌

令牌在 Symfony 安全环境中扮演了一个重要的角色。令牌表示当前请求中的用户身份验证数据。一旦请求通过了身份验证，令牌保留用户数据，并且通过安全环境传送该数据。首先，创建您的令牌类。这将允许您给您的身份验证提供程序传递所有的相关信息。

```
// src/AppBundle/Security/Authentication/Token/WsseUserToken.php
namespace AppBundle\Security\Authentication\Token;

use Symfony\Component\Security\Core\Authentication\Token\AbstractToken;

class WsseUserToken extends AbstractToken
{
    public $created;
    public $digest;
    public $nonce;

    public function __construct(array $roles = array())
    {
        parent::__construct($roles);

        // If the user has roles, consider it authenticated
        $this->setAuthenticated(count($roles) > 0);
    }

    public function getCredentials()
    {
        return '';
    }
}
```

> **WsseUserToken** 类是通过继承安全组件中的 [AbstractToken](http://api.symfony.com/2.7/Symfony/Component/Security/Core/Authentication/Token/AbstractToken.html) 类来实现在每一个类中把 [TokenInterface](http://api.symfony.com/2.7/Symfony/Component/Security/Core/Authentication/Token/TokenInterface.html) 当成令牌来使用，其中 AbstractToken 类提供了基本的令牌功能。

## 监听器

接下来，您需要一个监听器来监听防火墙。监听器的作用是向防火墙发送请求，并且调用身份验证提供程序。监听器必须是 [ListenerInterface](http://api.symfony.com/2.7/Symfony/Component/Security/Http/Firewall/ListenerInterface.html) 的一个实例。如果成功完成上述过程，那么一个安全的监听器应该具有处理 [GetResponseEvent](http://api.symfony.com/2.7/Symfony/Component/HttpKernel/Event/GetResponseEvent.html) 事件的能力，并在令牌存储中设置一个已通过身份验证的令牌，同时在令牌存储器中设置一个身份验证令牌。

```
// src/AppBundle/Security/Firewall/WsseListener.php
namespace AppBundle\Security\Firewall;

use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\HttpKernel\Event\GetResponseEvent;
use Symfony\Component\Security\Core\Authentication\AuthenticationManagerInterface;
use Symfony\Component\Security\Core\Authentication\Token\Storage\TokenStorageInterface;
use Symfony\Component\Security\Core\Exception\AuthenticationException;
use Symfony\Component\Security\Http\Firewall\ListenerInterface;
use AppBundle\Security\Authentication\Token\WsseUserToken;

class WsseListener implements ListenerInterface
{
    protected $tokenStorage;
    protected $authenticationManager;

    public function __construct(TokenStorageInterface $tokenStorage, AuthenticationManagerInterface $authenticationManager)
    {
        $this->tokenStorage = $tokenStorage;
        $this->authenticationManager = $authenticationManager;
    }

    public function handle(GetResponseEvent $event)
    {
        $request = $event->getRequest();

        $wsseRegex = '/UsernameToken Username="([^"]+)", PasswordDigest="([^"]+)", Nonce="([^"]+)", Created="([^"]+)"/';
        if (!$request->headers->has('x-wsse') || 1 !== preg_match($wsseRegex, $request->headers->get('x-wsse'), $matches)) {
            return;
        }

        $token = new WsseUserToken();
        $token->setUser($matches[1]);

        $token->digest   = $matches[2];
        $token->nonce    = $matches[3];
        $token->created  = $matches[4];

        try {
            $authToken = $this->authenticationManager->authenticate($token);
            $this->tokenStorage->setToken($authToken);

            return;
        } catch (AuthenticationException $failed) {
            // ... you might log something here

            // To deny the authentication clear the token. This will redirect to the login page.
            // Make sure to only clear your token, not those of other authentication listeners.
            // $token = $this->tokenStorage->getToken();
            // if ($token instanceof WsseUserToken && $this->providerKey === $token->getProviderKey()) {
            //     $this->tokenStorage->setToken(null);
            // }
            // return;
        }

        // By default deny authorization
        $response = new Response();
        $response->setStatusCode(Response::HTTP_FORBIDDEN);
        $event->setResponse($response);
    }
}
```

这个监听器的作用是为预期的 X-WSSE 头部进行检查，为预期的 WSSE 信息匹配返回值，然后使用这个信息来创建一个令牌，接着在身份验证管理器中传递这个令牌。如果没有提供正确的信息，或身份验证管理器抛出 [AuthenticationException](http://api.symfony.com/2.7/Symfony/Component/Security/Core/Exception/AuthenticationException.html) 异常，那么将会返回 403 页面。

> 在上面的过程中没有使用到 [AbstractAuthenticationListener](http://api.symfony.com/2.7/Symfony/Component/Security/Http/Firewall/AbstractAuthenticationListener.html) 类，它是一个非常有用并且为安全性扩展插件提供了常用功能的基类。其中包括在 session 中维持令牌功能，提供成功 / 失败的处理程序、 登录表单的 URL，以及更多的功能。因为 WSSE 不需要在 session 中 保持身份验证或登录表单，所以在本实例中没有用到它。

> 只有当您想把身份验证程序串接起来的时候，提前的从监听器返回一个值才是有价值的，如果您想要禁止匿名用户访问，并且能够较好地展示 403 错误，则应在返回结果之前设置响应的状态码。

## 身份验证提供程序

身份验证提供程序将会为 **WsseUserToken** 进行验证。即,提供程序将会在 5 分钟之内验证**已经创建好的**标头值是否有效。**Nonce** 是唯一一个能在 5 分钟之内检查出结果的标头值，并且 **PasswordDigest** 标头值与该用户的密码相匹配。

```
// src/AppBundle/Security/Authentication/Provider/WsseProvider.php
namespace AppBundle\Security\Authentication\Provider;

use Symfony\Component\Security\Core\Authentication\Provider\AuthenticationProviderInterface;
use Symfony\Component\Security\Core\User\UserProviderInterface;
use Symfony\Component\Security\Core\Exception\AuthenticationException;
use Symfony\Component\Security\Core\Exception\NonceExpiredException;
use Symfony\Component\Security\Core\Authentication\Token\TokenInterface;
use AppBundle\Security\Authentication\Token\WsseUserToken;
use Symfony\Component\Security\Core\Util\StringUtils;

class WsseProvider implements AuthenticationProviderInterface
{
    private $userProvider;
    private $cacheDir;

    public function __construct(UserProviderInterface $userProvider, $cacheDir)
    {
        $this->userProvider = $userProvider;
        $this->cacheDir     = $cacheDir;
    }

    public function authenticate(TokenInterface $token)
    {
        $user = $this->userProvider->loadUserByUsername($token->getUsername());

        if ($user && $this->validateDigest($token->digest, $token->nonce, $token->created, $user->getPassword())) {
            $authenticatedToken = new WsseUserToken($user->getRoles());
            $authenticatedToken->setUser($user);

            return $authenticatedToken;
        }

        throw new AuthenticationException('The WSSE authentication failed.');
    }

    /**
     * This function is specific to Wsse authentication and is only used to help this example
     *
     * For more information specific to the logic here, see
     * https://github.com/symfony/symfony-docs/pull/3134#issuecomment-27699129
     */
    protected function validateDigest($digest, $nonce, $created, $secret)
    {
        // Check created time is not in the future
        if (strtotime($created) > time()) {
            return false;
        }

        // Expire timestamp after 5 minutes
        if (time() - strtotime($created) > 300) {
            return false;
        }

        // Validate that the nonce is *not* used in the last 5 minutes
        // if it has, this could be a replay attack
        if (file_exists($this->cacheDir.'/'.$nonce) && file_get_contents($this->cacheDir.'/'.$nonce) + 300 > time()) {
            throw new NonceExpiredException('Previously used nonce detected');
        }
        // If cache directory does not exist we create it
        if (!is_dir($this->cacheDir)) {
            mkdir($this->cacheDir, 0777, true);
        }
        file_put_contents($this->cacheDir.'/'.$nonce, time());

        // Validate Secret
        $expected = base64_encode(sha1(base64_decode($nonce).$created.$secret, true));

        return StringUtils::equals($expected, $digest);
    }

    public function supports(TokenInterface $token)
    {
        return $token instanceof WsseUserToken;
    }
}
```

> [AuthenticationProviderInterface](http://api.symfony.com/2.7/Symfony/Component/Security/Core/Authentication/Provider/AuthenticationProviderInterface.html) 需要用户令牌中的一种**身份验证**方法，和一种能够告诉身份验证管理器是否为给定的令牌使用提供程序的**支持**方法。在众多提供程序中，身份验证管理器会根据列表依次移动到每个提供程序。

> 预期的比较和提供的摘要会使用 **StringUtils** 类的 [equals ()](http://api.symfony.com/2.7/Symfony/Component/Security/Core/Util/StringUtils.html#equals()) 方法提供的恒定的时间比较。它的作用是用来减少可能的[定时攻击](https://en.wikipedia.org/wiki/Timing_attack)。

## 工厂模式

您已经创建了一个自定义的令牌，自定义监听器和自定义提供程序。现在您需要把它们联系到一起。问题是您怎么为每个防火墙设定一个独特的提供程序？答案是通过使用一个工厂。工厂是您钩入安全组件的地方，您需要告诉它您的提供程序的名称和任何可用于它的配置选项。首先，您必须创建一个实现 [SecurityFactoryInterface](http://api.symfony.com/2.7/Symfony/Bundle/SecurityBundle/DependencyInjection/Security/Factory/SecurityFactoryInterface.html) 的类。

```
// src/AppBundle/DependencyInjection/Security/Factory/WsseFactory.php
namespace AppBundle\DependencyInjection\Security\Factory;

use Symfony\Component\DependencyInjection\ContainerBuilder;
use Symfony\Component\DependencyInjection\Reference;
use Symfony\Component\DependencyInjection\DefinitionDecorator;
use Symfony\Component\Config\Definition\Builder\NodeDefinition;
use Symfony\Bundle\SecurityBundle\DependencyInjection\Security\Factory\SecurityFactoryInterface;

class WsseFactory implements SecurityFactoryInterface
{
    public function create(ContainerBuilder $container, $id, $config, $userProvider, $defaultEntryPoint)
    {
        $providerId = 'security.authentication.provider.wsse.'.$id;
        $container
            ->setDefinition($providerId, new DefinitionDecorator('wsse.security.authentication.provider'))
            ->replaceArgument(0, new Reference($userProvider))
        ;

        $listenerId = 'security.authentication.listener.wsse.'.$id;
        $listener = $container->setDefinition($listenerId, new DefinitionDecorator('wsse.security.authentication.listener'));

        return array($providerId, $listenerId, $defaultEntryPoint);
    }

    public function getPosition()
    {
        return 'pre_auth';
    }

    public function getKey()
    {
        return 'wsse';
    }

    public function addConfiguration(NodeDefinition $node)
    {
    }
}
```

[SecurityFactoryInterface](http://api.symfony.com/2.7/Symfony/Bundle/SecurityBundle/DependencyInjection/Security/Factory/SecurityFactoryInterface.html) 需要下列方法:

**create** 方法  

这个方法为适当的安全环境把监听器和身份验证提供程序添加到 DI 容器中。

**getPosition** 方法  

这必须是 **pre_auth**、**表单**、**http**，**remember_me** 类型和定义在已经被调用的提供程序中的方法。

**getKey** 方法  

该方法定义了用来引用防火墙配置中的提供程序的配置密钥。

**addConfiguration** 方法  

该方法用于定义您安全配置中的配置密钥下的配置选项。在后面将要介绍设置配置选项。

> 在这个示例中，我们没有用到 [AbstractFactory](http://api.symfony.com/2.7/Symfony/Bundle/SecurityBundle/DependencyInjection/Security/Factory/AbstractFactory.html) 类，它是一个非常有用的基类，并且它为安全工厂提供了常用的功能。当定义不同类型的身份验证提供程序的时候，它可能会比较有用。

既然您已经创建了一个工厂类，在您的安全配置中 wsse 密钥可以当成防火墙来使用。

> 您可能会想知道，"您为什么需要特殊的工厂类，将监听器和提供程序添加到依赖注入容器？"这是一个非常好的问题。原因是，您可以多次使用您的防火墙，来保护您的应用程序的各个部分。正因为如此，每次使用您的防火墙时，在 DI 容器中便会创建一项新的服务 。工厂的作用就是创造这些新的服务。

## 配置

现在可以在过程中来查看身份验证提供程序。为了让它们运行起来，您现在需要做一些事情。第一件事是将上面描述的服务添加到 DI 容器。上边提到的您的工厂类可以参考还不存在的服务 id : **wsse.security.authentication.provider** 和 **wsse.security.authentication.listener**。现在让我们来定义这些服务。

YAML:

```
# src/AppBundle/Resources/config/services.yml
services:
    wsse.security.authentication.provider:
        class: AppBundle\Security\Authentication\Provider\WsseProvider
        arguments: ["", "%kernel.cache_dir%/security/nonces"]

    wsse.security.authentication.listener:
        class: AppBundle\Security\Firewall\WsseListener
        arguments: ["@security.token_storage", "@security.authentication.manager"]
```

XML:

```
<!-- src/AppBundle/Resources/config/services.xml -->
<container xmlns="http://symfony.com/schema/dic/services"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://symfony.com/schema/dic/services http://symfony.com/schema/dic/services/services-1.0.xsd">

    <services>
        <service id="wsse.security.authentication.provider"
            class="AppBundle\Security\Authentication\Provider\WsseProvider" public="false">
            <argument /> <!-- User Provider -->
            <argument>%kernel.cache_dir%/security/nonces</argument>
        </service>

        <service id="wsse.security.authentication.listener"
            class="AppBundle\Security\Firewall\WsseListener" public="false">
            <argument type="service" id="security.token_storage"/>
            <argument type="service" id="security.authentication.manager" />
        </service>
    </services>
</container>
```

PHP:

```
// src/AppBundle/Resources/config/services.php
use Symfony\Component\DependencyInjection\Definition;
use Symfony\Component\DependencyInjection\Reference;

$container->setDefinition('wsse.security.authentication.provider',
    new Definition(
        'AppBundle\Security\Authentication\Provider\WsseProvider', array(
            '',
            '%kernel.cache_dir%/security/nonces',
        )
    )
);

$container->setDefinition('wsse.security.authentication.listener',
    new Definition(
        'AppBundle\Security\Firewall\WsseListener', array(
            new Reference('security.token_storage'),
            new Reference('security.authentication.manager'),
        )
    )
);
```

到现在，您的服务已经定义好了，现在可以把您的包类中的工厂告诉您的安全环境:

```
// src/AppBundle/AppBundle.php
namespace AppBundle;

use AppBundle\DependencyInjection\Security\Factory\WsseFactory;
use Symfony\Component\HttpKernel\Bundle\Bundle;
use Symfony\Component\DependencyInjection\ContainerBuilder;

class AppBundle extends Bundle
{
    public function build(ContainerBuilder $container)
    {
        parent::build($container);

        $extension = $container->getExtension('security');
        $extension->addSecurityListenerFactory(new WsseFactory());
    }
}
```

这样我们就完成了配置！您现在可以在 WSSE 的保护下以定义您的应用程序了。

YAML:

```
security:
    firewalls:
        wsse_secured:
            pattern:   /api/.*
            stateless: true
            wsse:      true
```

XML:

```
<config>
    <firewall name="wsse_secured" pattern="/api/.*">
        <stateless />
        <wsse />
    </firewall>
</config>
```

PHP:

```
$container->loadFromExtension('security', array(
    'firewalls' => array(
        'wsse_secured' => array(
            'pattern' => '/api/.*',
            'stateless'    => true,
            'wsse'    => true,
        ),
    ),
));
```

祝贺您！您已经完成了您的定义安全身份验证提供程序的编写！

## 额外的补充

如何让您的 WSSE 身份验证提供程序更令人兴奋呢？这个答案可能是无解的。那么您为什么不给它添加一些闪烁的光芒？

### 配置

您可以在您的安全配置中的 **wsse** 密钥下添加自定义选项。例如，默认情况下，在终止**已经创建的**标题项之前有 5 分钟的时间。可以通过配置来实现让不同的防火墙有不同的超时限额。

首先，您需要编辑 **WsseFactory** 并且在 **addConfiguration** 方法中定义新的选项。

```
class WsseFactory implements SecurityFactoryInterface
{
    // ...

    public function addConfiguration(NodeDefinition $node)
    {
      $node
        ->children()
        ->scalarNode('lifetime')->defaultValue(300)
        ->end();
    }
}
```

现在，在工厂的**构造**方法中，**$config** 参数将会包含一个**生存期**秘钥，并且设置为 5 分钟 (300 秒)，除非在配置中设置了其他时间。然后向您的身份验证提供程序传递此参数来使用它。

```
class WsseFactory implements SecurityFactoryInterface
{
    public function create(ContainerBuilder $container, $id, $config, $userProvider, $defaultEntryPoint)
    {
        $providerId = 'security.authentication.provider.wsse.'.$id;
        $container
            ->setDefinition($providerId,
              new DefinitionDecorator('wsse.security.authentication.provider'))
            ->replaceArgument(0, new Reference($userProvider))
            ->replaceArgument(2, $config['lifetime']);
        // ...
    }

    // ...
}
```

> 您还需要为 **wsse.security.authentication.provider** 服务配置添加第三个参数，它可以是空白的，但必须在工厂的生存期内填充。**WsseProvider** 类现在还需要接受第三个构造函数参数 - 生存期 - 而它需要使用并不是硬编码的 300 秒。在这里没有展示这两个步骤。

每个 WSSE 请求的生存期现在都是是可配置的，并可以为每个防火墙设置任何可取的值。

YAML:

```
security:
    firewalls:
        wsse_secured:
            pattern:   /api/.*
            stateless: true
            wsse:      { lifetime: 30 }
```

XML:

```
<config>
    <firewall name="wsse_secured"
        pattern="/api/.*"
    >
        <stateless />
        <wsse lifetime="30" />
    </firewall>
</config>
```

PHP:

```
$container->loadFromExtension('security', array(
    'firewalls' => array(
        'wsse_secured' => array(
            'pattern' => '/api/.*',
            'stateless' => true,
            'wsse'    => array(
                'lifetime' => 30,
            ),
        ),
    ),
));
```

接下来就交给您自己配置了!在工厂中可以定义任何相关的配置项并且配置项可以传递到容器中的其他类或被消耗。
