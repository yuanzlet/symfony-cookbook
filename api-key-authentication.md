# 如何使用 API 验证用户

如今，我们常使用 API 去验证一个用户的身份 (例如开发一个 web 服务的时候)。该 API 密钥可以为每个请求提供服务，并且以查询字符串参数的形式或通过 HTTP 头部信息进行传递。

## API 密钥身份验证器

我们应该通过预身份验证机制请求信息来对用户身份进行验证。[SimplePreAuthenticatorInterface](http://api.symfony.com/2.7/Symfony/Component/Security/Core/Authentication/SimplePreAuthenticatorInterface.html) 接口能让您很容易的达到这个目的。

您的实际情况可能会有所不同，但在此示例中，从 apikey 查询参数中读取令牌、 从该值中加载正确的用户名，然后创建一个用户对象:

```
// src/AppBundle/Security/ApiKeyAuthenticator.php
namespace AppBundle\Security;

use Symfony\Component\Security\Core\Authentication\SimplePreAuthenticatorInterface;
use Symfony\Component\Security\Core\Authentication\Token\TokenInterface;
use Symfony\Component\Security\Core\Exception\AuthenticationException;
use Symfony\Component\Security\Core\Authentication\Token\PreAuthenticatedToken;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\Security\Core\User\UserProviderInterface;
use Symfony\Component\Security\Core\Exception\BadCredentialsException;

class ApiKeyAuthenticator implements SimplePreAuthenticatorInterface
{
    public function createToken(Request $request, $providerKey)
    {
        // look for an apikey query parameter
        $apiKey = $request->query->get('apikey');

        // or if you want to use an "apikey" header, then do something like this:
        // $apiKey = $request->headers->get('apikey');

        if (!$apiKey) {
            throw new BadCredentialsException('No API key found');

            // or to just skip api key authentication
            // return null;
        }

        return new PreAuthenticatedToken(
            'anon.',
            $apiKey,
            $providerKey
        );
    }

    public function authenticateToken(TokenInterface $token, UserProviderInterface $userProvider, $providerKey)
    {
        if (!$userProvider instanceof ApiKeyUserProvider) {
            throw new \InvalidArgumentException(
                sprintf(
                    'The user provider must be an instance of ApiKeyUserProvider (%s was given).',
                    get_class($userProvider)
                )
            );
        }

        $apiKey = $token->getCredentials();
        $username = $userProvider->getUsernameForApiKey($apiKey);

        if (!$username) {
            throw new AuthenticationException(
                sprintf('API Key "%s" does not exist.', $apiKey)
            );
        }

        $user = $userProvider->loadUserByUsername($username);

        return new PreAuthenticatedToken(
            $user,
            $apiKey,
            $providerKey,
            $user->getRoles()
        );
    }

    public function supportsToken(TokenInterface $token, $providerKey)
    {
        return $token instanceof PreAuthenticatedToken && $token->getProviderKey() === $providerKey;
    }
}
```

当您已经配置了所以东西，你可以通过将 apikey 参数添加到查询字符串来进行身份验证，如同** http://example.com/admin/foo?apikey=37b51d194a7513e45b56f6524f2d51f2** 。

身份验证过程有几个步骤，不过实现的过程可能会有所不同:

## 1\. createToken方法

在请求周期的早期 Symfony 调用 **createToken()** 方法。在这里您需要做的就是去创建一个包含所有消息的令牌对象，这些消息是来自于您的某个请求，在这个请求中您需要对用户进行身份验证 (例如 apikey 查询参数) 。如果缺少这些信息，则会抛出 [BadCredentialsException](http://api.symfony.com/2.7/Symfony/Component/Security/Core/Exception/BadCredentialsException.html) 异常从而导致身份验证失败。相反，您可能想要跳过身份验证并且不返回信息，所以 Symfony 可以回退到另一种身份验证方法，如果存在这种方法。

## 2\. supportsToken方法

当 Symfony 调用 **createToken()** 之后，它将调用您的类中的 **supportsToken()** 方法 (和任何其它身份验证监听器) 来弄清到底应该谁来处理令牌，这只是一种允许同一种防火墙使用几种身份验证机制的方法(用这种方式，例如您可以首先尝试使用证书或 API 密钥来对用户进行身份验证然后回退到表单登录)。

## 3\. authenticateToken方法
如果 **supportsToken()** 返回 true，Symfony 将会调用 **authenticateToken()** 方法。一个关键部分是 **$userProvider**，这是外部的类，并且可以帮助您加载有关用户的信息。接下来您会了解更多关于 **supportsToken()** 的内容。

在此特定示例中，下面所述的事情将会出现在 **authenticateToken()** 中:

1\. 首先，您可以使用 **$userProvider** 来以某种方式查找 **$apiKey** 对应的 **$username**。

2\. 其次，再次使用 **$userProvider** 为 **$username** 加载或创建用户对象;

3\. 最后，您将创建具有适当的角色并且已经通过身份验证的令牌(即标记具有至少一个角色)，然后在令牌上还附加上用户对象。

最终目标是使用 **$apiKey** 来查找或创建用户对象。您如何去完成它 (例如查询数据库) 以及您的用户对象的确切的类可能会不同。这些差异在您的用户提供程序中最明显。

## 用户提供程序

**$userProvider** 可以是任何用户提供程序 (请参阅[如何创建一个自定义的用户提供程序](http://symfony.com/doc/current/cookbook/security/custom_provider.html))。在此示例中，以某种方式用 **$apiKey** 为用户寻找用户名。这项工作是在 **getUsernameForApiKey()** 方法中完成的，这个方法在这个用例中完全是自定义的 (即这不是 Symfony 核心用户提供程序系统中的一种方法) 。

如下所示便是一个 **$userProvider** :

```
// src/AppBundle/Security/ApiKeyUserProvider.php
namespace AppBundle\Security;

use Symfony\Component\Security\Core\User\UserProviderInterface;
use Symfony\Component\Security\Core\User\User;
use Symfony\Component\Security\Core\User\UserInterface;
use Symfony\Component\Security\Core\Exception\UnsupportedUserException;

class ApiKeyUserProvider implements UserProviderInterface
{
    public function getUsernameForApiKey($apiKey)
    {
        // Look up the username based on the token in the database, via
        // an API call, or do something entirely different
        $username = ...;

        return $username;
    }

    public function loadUserByUsername($username)
    {
        return new User(
            $username,
            null,
            // the roles for the user - you may choose to determine
            // these dynamically somehow based on the user
            array('ROLE_USER')
        );
    }

    public function refreshUser(UserInterface $user)
    {
        // this is used for storing authentication in the session
        // but in this example, the token is sent in each request,
        // so authentication can be stateless. Throwing this exception
        // is proper to make things stateless
        throw new UnsupportedUserException();
    }

    public function supportsClass($class)
    {
        return 'Symfony\Component\Security\Core\User\User' === $class;
    }
}
```

这时，把您的用户提供程序注册成为一种服务：

YAML:

```
# app/config/services.yml
services:
    api_key_user_provider:
        class: AppBundle\Security\ApiKeyUserProvider
```

XML:

```
<!-- app/config/services.xml -->
<?xml version="1.0" ?>
<container xmlns="http://symfony.com/schema/dic/services"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://symfony.com/schema/dic/services
        http://symfony.com/schema/dic/services/services-1.0.xsd">
    <services>
        <!-- ... -->

        <service id="api_key_user_provider"
            class="AppBundle\Security\ApiKeyUserProvider" />
    </services>
</container>
```

PHP:

```
// app/config/services.php

// ...
$container
    ->register('api_key_user_provider', 'AppBundle\Security\ApiKeyUserProvider');
```

> 请阅读特定的文章来了解[如何去创建一个用户自定义提供程序](http://symfony.com/doc/current/cookbook/security/custom_provider.html)。

**getUsernameForApiKey()** 方法中的代码逻辑是由您决定的。您可能用某种方式通过在“令牌”数据表中查找一些信息来把 API 秘钥（如 **37b51d**）转换成一个用户名(例如 **jondoe**)。

上述过程同样适用于 **loadUserByUsername()** 方法。在此示例中，只需创建 Symfony 的核心[用户类](http://api.symfony.com/2.7/Symfony/Component/Security/Core/User/User.html)。如果你不需要在您的用户对象中存储额外的信息 (如用户的**姓**)，这样将会更加合理。不过如果您需要去存储更多的信息，那么您可以创建一个您自己的用户类并且通过查询数据库来填充它，这样将允许您在**用户对象**中添加自定义数据。

最后，就像任何通过 **loadUserByUsername()** 方法返回的用户类一样，我们只需要确保 **supportsClass()** 方法为用户对象返回**正确**的内容。 就如同这个例子中描述的那样，如果您的身份验证是无状态的 (即您期望用户发送的 API 密钥包含了所有的请求，那么您就不用把登录保存到 session 中了 ），那么您只用通过 **refreshUser()** 方法抛出 **UnsupportedUserException** 异常。     

> 如果你想要在 session 中存储身份验证数据，那么并不需要在每个请求中发送秘钥，请参阅[在 session 中存储身份验证](http://symfony.com/doc/current/cookbook/security/api_key_authentication.html#cookbook-security-api-key-session)。

## 身份验证处理失败

当凭据验证失败或者身份验证失败时，为了能让您的 **ApiKeyAuthenticator** 正确的显示 403 http 状态，您应该在您的身份验证器中实现 [AuthenticationFailureHandlerInterface](http://api.symfony.com/2.7/Symfony/Component/Security/Http/Authentication/AuthenticationFailureHandlerInterface.html) 接口。您可以使用该接口中的 **onAuthenticationFailure** 方法去创建一个错误**响应**。

```
// src/AppBundle/Security/ApiKeyAuthenticator.php
namespace AppBundle\Security;

use Symfony\Component\Security\Core\Authentication\SimplePreAuthenticatorInterface;
use Symfony\Component\Security\Core\Exception\AuthenticationException;
use Symfony\Component\Security\Http\Authentication\AuthenticationFailureHandlerInterface;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\HttpFoundation\Request;

class ApiKeyAuthenticator implements SimplePreAuthenticatorInterface, AuthenticationFailureHandlerInterface
{
    // ...

    public function onAuthenticationFailure(Request $request, AuthenticationException $exception)
    {
        return new Response("Authentication Failed.", 403);
    }
}
```

## 配置

当您完成了对 **ApiKeyAuthenticator** 的所有安装，您需要把它注册为一个服务并且在您的安全配置中使用它(例如 **security.yml**)。第一步，先把它注册为一个服务.

YAML:

```
# app/config/config.yml
services:
    # ...

    apikey_authenticator:
        class:  AppBundle\Security\ApiKeyAuthenticator
        public: false
```

XML:

```
<!-- app/config/config.xml -->
<?xml version="1.0" ?>
<container xmlns="http://symfony.com/schema/dic/services"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://symfony.com/schema/dic/services
        http://symfony.com/schema/dic/services/services-1.0.xsd">
    <services>
        <!-- ... -->

        <service id="apikey_authenticator"
            class="AppBundle\Security\ApiKeyAuthenticator"
            public="false" />
    </services>
</container>
```

PHP:

```
// app/config/config.php
use Symfony\Component\DependencyInjection\Definition;
use Symfony\Component\DependencyInjection\Reference;

// ...

$definition = new Definition('AppBundle\Security\ApiKeyAuthenticator');
$definition->setPublic(false);
$container->setDefinition('apikey_authenticator', $definition);
```

第二步，分别使用 **simple_preauth** 和提供程序秘钥在您的安全配置中的防火墙部分中激活它和自定义用户提供程序 (请参阅[如何创建一个自定义的用户提供程序](http://symfony.com/doc/current/cookbook/security/custom_provider.html))：

YAML:

```
# app/config/security.yml
security:
    # ...

    firewalls:
        secured_area:
            pattern: ^/admin
            stateless: true
            simple_preauth:
                authenticator: apikey_authenticator
            provider: api_key_user_provider

    providers:
        api_key_user_provider:
            id: api_key_user_provider
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
        <!-- ... -->

        <firewall name="secured_area"
            pattern="^/admin"
            stateless="true"
            provider="api_key_user_provider"
        >
            <simple-preauth authenticator="apikey_authenticator" />
        </firewall>

        <provider name="api_key_user_provider" id="api_key_user_provider" />
    </config>
</srv:container>
```

PHP:

```
// app/config/security.php

// ..

$container->loadFromExtension('security', array(
    'firewalls' => array(
        'secured_area'       => array(
            'pattern'        => '^/admin',
            'stateless'      => true,
            'simple_preauth' => array(
                'authenticator'  => 'apikey_authenticator',
            ),
            'provider' => 'api_key_user_provider',
        ),
    ),
    'providers' => array(
        'api_key_user_provider'  => array(
            'id' => 'api_key_user_provider',
        ),
    ),
));
```

完成了上述步骤!现在，在每个请求开始的时候，您的 **ApiKeyAuthenticator** 方法都会被调用，然后将进行身份验证过程。

无状态配置参数防止 Symfony 尝试在 session 中存储身份验证信息，但是并不必要，因为对于每个请求客户端都会发送 **apikey**。对每个请求中存储身份验证信息。如果你确实需要在 session 中存储身份验证，请继续阅读下去!

## 在 Session 中存储身份验证

到目前为止，本文已经介绍了有一些身份验证令牌在每个请求中都会被传送。但在某些情况下 (如 OAuth 流程)，令牌可能只被传递给一个请求。在这种情况下，您一定想要对用户进行身份验证并在 session 中存储身份验证，以便用户在每个后续请求中自动登录。

想要实现上述功能，首先您应该在您的防火墙设置中将**无状态**秘钥删除或者把它们设置为 **false**：

YAML:

```
# app/config/security.yml
security:
    # ...

    firewalls:
        secured_area:
            pattern: ^/admin
            stateless: false
            simple_preauth:
                authenticator: apikey_authenticator
            provider: api_key_user_provider

    providers:
        api_key_user_provider:
            id: api_key_user_provider
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
        <!-- ... -->

        <firewall name="secured_area"
            pattern="^/admin"
            stateless="false"
            provider="api_key_user_provider"
        >
            <simple-preauth authenticator="apikey_authenticator" />
        </firewall>

        <provider name="api_key_user_provider" id="api_key_user_provider" />
    </config>
</srv:container>
```

PHP:

```
// app/config/security.php

// ..
$container->loadFromExtension('security', array(
    'firewalls' => array(
        'secured_area'       => array(
            'pattern'        => '^/admin',
            'stateless'      => false,
            'simple_preauth' => array(
                'authenticator'  => 'apikey_authenticator',
            ),
            'provider' => 'api_key_user_provider',
        ),
    ),
    'providers' => array(
        'api_key_user_provider' => array(
            'id' => 'api_key_user_provider',
        ),
    ),
));
```

即使令牌被存储在 session 中，在这种情况下由于某些安全性原因，凭据和API 密钥 (即 **$token->getCredentials()**) 不会被存储在 session 中。如果想要利用 session，请更新 **ApiKeyAuthenticator** 来查看被存储的令牌是否有一个可以使用的有效用户对象：

```
// src/AppBundle/Security/ApiKeyAuthenticator.php
// ...

class ApiKeyAuthenticator implements SimplePreAuthenticatorInterface
{
    // ...
    public function authenticateToken(TokenInterface $token, UserProviderInterface $userProvider, $providerKey)
    {
        if (!$userProvider instanceof ApiKeyUserProvider) {
            throw new \InvalidArgumentException(
                sprintf(
                    'The user provider must be an instance of ApiKeyUserProvider (%s was given).',
                    get_class($userProvider)
                )
            );
        }

        $apiKey = $token->getCredentials();
        $username = $userProvider->getUsernameForApiKey($apiKey);

        // User is the Entity which represents your user
        $user = $token->getUser();
        if ($user instanceof User) {
            return new PreAuthenticatedToken(
                $user,
                $apiKey,
                $providerKey,
                $user->getRoles()
            );
        }

        if (!$username) {
            throw new AuthenticationException(
                sprintf('API Key "%s" does not exist.', $apiKey)
            );
        }

        $user = $userProvider->loadUserByUsername($username);

        return new PreAuthenticatedToken(
            $user,
            $apiKey,
            $providerKey,
            $user->getRoles()
        );
    }
    // ...
}
```

在 session 中存储身份验证信息工作原理是这样的:

1\. 在每个请求结束后，Symfony 将会序列化令牌对象 (由 **authenticateToken()** 返回)，同时也会序列化用户对象 (因为在令牌中设置了它的属性); 

2\. 在下一个请求中令牌将被反序列化并且被反序列化的用户对象将被传送给用户提供程序中的 **refreshUser()** 函数。

第二步是最重要的: Symfony 将会调用 **refreshUser()** 方法并把在 session 周期中序列化的用户对象传递给您。如果您的用户信息存储在数据库中，然后你可能想要重新查询一个新版本的用户信息来确保还没过期。但是，如果不理会您的要求，**refreshUser()** 现在应该返回用户对象:

```
// src/AppBundle/Security/ApiKeyUserProvider.php

// ...
class ApiKeyUserProvider implements UserProviderInterface
{
    // ...

    public function refreshUser(UserInterface $user)
    {
        // $user is the User that you set in the token inside authenticateToken()
        // after it has been deserialized from the session

        // you might use $user to query the database for a fresh user
        // $id = $user->getId();
        // use $id to make a query

        // if you are *not* reading from a database and are just creating
        // a User object (like in this example), you can just return it
        return $user;
    }
}
```

> 您一定也会想要确保您的用户对象被正确的序列化。如果您的用户对象具有私有属性，PHP 将无法序列化它们。在这种情况下，你可能得到一个所有属性都是空的用户对象。有关示例，请参见[如何从数据库中加载安全用户](http://symfony.com/doc/current/cookbook/security/entity_provider.html) (实体提供程序)。

## 仅仅为特定的 URL 进行验证

在本小节中假定您想要获取每次请求的 apikey 身份验证。但在某些情况下 (如 OAuth 流程)，你只需要在用户已经访问到了确定的 URL 的时候获取一次身份验证信息 (例如在 OAuth 中的 URL 重定向 )。

幸运的是，处理这个问题还比较容易: 只用检查使用 **createToken()** 方法创建令牌之前的 URL 是什么即可:

```
// src/AppBundle/Security/ApiKeyAuthenticator.php

// ...
use Symfony\Component\Security\Http\HttpUtils;
use Symfony\Component\HttpFoundation\Request;

class ApiKeyAuthenticator implements SimplePreAuthenticatorInterface
{
    protected $httpUtils;

    public function __construct(HttpUtils $httpUtils)
    {
        $this->httpUtils = $httpUtils;
    }

    public function createToken(Request $request, $providerKey)
    {
        // set the only URL where we should look for auth information
        // and only return the token if we're at that URL
        $targetUrl = '/login/check';
        if (!$this->httpUtils->checkRequestPath($request, $targetUrl)) {
            return;
        }

        // ...
    }
}
```

在这里使用较为便利的 [HttpUtils](http://api.symfony.com/2.7/Symfony/Component/Security/Http/HttpUtils.html) 类来检查当前的 URL 是否与您想要获取的 URL 相匹配。在这种情况下，URL (**/登录/检查**) 已经在类中被硬编码，但是您仍然可以把它作为构造函数的第二个参数。

接下来，只用更新您的服务配置来注入 **security.http_utils** 服务:

YAML:

```
# app/config/config.yml
services:
    # ...

    apikey_authenticator:
        class:     AppBundle\Security\ApiKeyAuthenticator
        arguments: ["@security.http_utils"]
        public:    false
```

XML:

```
<!-- app/config/config.xml -->
<?xml version="1.0" ?>
<container xmlns="http://symfony.com/schema/dic/services"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://symfony.com/schema/dic/services
        http://symfony.com/schema/dic/services/services-1.0.xsd">
    <services>
        <!-- ... -->

        <service id="apikey_authenticator"
            class="AppBundle\Security\ApiKeyAuthenticator"
            public="false"
        >
            <argument type="service" id="security.http_utils" />
        </service>
    </services>
</container>
```

PHP:

```
// app/config/config.php
use Symfony\Component\DependencyInjection\Definition;
use Symfony\Component\DependencyInjection\Reference;

// ...

$definition = new Definition(
    'AppBundle\Security\ApiKeyAuthenticator',
    array(
        new Reference('security.http_utils')
    )
);
$definition->setPublic(false);
$container->setDefinition('apikey_authenticator', $definition);
```

到这里，本章就讲完了，祝您愉快！