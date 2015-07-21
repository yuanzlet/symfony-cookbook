# 如何在功能测试中测试一封电子邮件被发送

由于 SwiftmailerBundle 的缘故，用 Symfony 发送电子邮件是相当简单的，是利用 [Swift Mailer](http://swiftmailer.org/) 库的能力。

要功能测试电子邮件是否发送，甚至判断电子邮件主题，内容或者其他标题，您可以使用 [Symfony 分析器](http://symfony.com/doc/current/cookbook/profiler/index.html)。

开始先用一个简单的控制器动作发送一封电子邮件：

```
public function sendEmailAction($name)
{
    $message = \Swift_Message::newInstance()
        ->setSubject('Hello Email')
        ->setFrom('send@example.com')
        ->setTo('recipient@example.com')
        ->setBody('You should see me from the profiler!')
    ;

    $this->get('mailer')->send($message);

    return $this->render(...);
}
```

不要忘记按照在[如何在功能测试中使用编译器](http://symfony.com/doc/current/cookbook/testing/profiling.html)解释的那样启动分析器。

在您的功能测试中，在分析器中使用 **swiftmailer** 收集器来获取关于发送在之前请求上的消息的信息：

```
// src/AppBundle/Tests/Controller/MailControllerTest.php
use Symfony\Bundle\FrameworkBundle\Test\WebTestCase;

class MailControllerTest extends WebTestCase
{
    public function testMailIsSentAndContentIsOk()
    {
        $client = static::createClient();

        // Enable the profiler for the next request (it does nothing if the profiler is not available)
        $client->enableProfiler();

        $crawler = $client->request('POST', '/path/to/above/action');

        $mailCollector = $client->getProfile()->getCollector('swiftmailer');

        // Check that an email was sent
        $this->assertEquals(1, $mailCollector->getMessageCount());

        $collectedMessages = $mailCollector->getMessages();
        $message = $collectedMessages[0];

        // Asserting email data
        $this->assertInstanceOf('Swift_Message', $message);
        $this->assertEquals('Hello Email', $message->getSubject());
        $this->assertEquals('send@example.com', key($message->getFrom()));
        $this->assertEquals('recipient@example.com', key($message->getTo()));
        $this->assertEquals(
            'You should see me from the profiler!',
            $message->getBody()
        );
    }
}
```
