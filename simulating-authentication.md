# 如何在功能测试中用 Token 模拟认证


功能测试中的认证请求可能会延缓程序组。尤其当使用 **form_login** 的时候，它可能成为问题，因为它需要额外的填写和提交表单的需求。

解决办法之一是在测试环境中像[如何在功能测试中模拟 HTTP 认证](http://symfony.com/doc/current/cookbook/testing/http_authentication.html)中解释的用法一样来设置防火墙使用 **http_basic**。另一个方法是您自己创建一个 token 并将它储存在一个会话中。当您这样做的时候，您必须确认一个适当的 cookie 随着一个请求发送。下面的例子演示了这一技术：

```PHP
// src/AppBundle/Tests/Controller/DefaultControllerTest.php
namespace Appbundle\Tests\Controller;

use Symfony\Bundle\FrameworkBundle\Test\WebTestCase;
use Symfony\Component\BrowserKit\Cookie;
use Symfony\Component\Security\Core\Authentication\Token\UsernamePasswordToken;

class DefaultControllerTest extends WebTestCase
{
    private $client = null;

    public function setUp()
    {
        $this->client = static::createClient();
    }

    public function testSecuredHello()
    {
        $this->logIn();

        $crawler = $this->client->request('GET', '/admin');

        $this->assertTrue($this->client->getResponse()->isSuccessful());
        $this->assertGreaterThan(0, $crawler->filter('html:contains("Admin Dashboard")')->count());
    }

    private function logIn()
    {
        $session = $this->client->getContainer()->get('session');

        $firewall = 'secured_area';
        $token = new UsernamePasswordToken('admin', null, $firewall, array('ROLE_ADMIN'));
        $session->set('_security_'.$firewall, serialize($token));
        $session->save();

        $cookie = new Cookie($session->getName(), $session->getId());
        $this->client->getCookieJar()->set($cookie);
    }
}
```

> 这一技术在[如何在功能测试中模拟 HTTP 认证](http://symfony.com/doc/current/cookbook/testing/http_authentication.html)中解释得更为整齐，是首选方式。
