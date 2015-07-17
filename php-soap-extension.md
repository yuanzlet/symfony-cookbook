# 如何在一个 Symfony 控制器中创建一个 SOAP 的 Web 服务

我们可以使用几个简单的工具把控制器设置为一个 SOAP 服务器。当然，您必须安装  [PHP SOAP](http://php.net/manual/en/book.soap.php "PHP SOAP") 扩展。介于目前 PHP SOAP 扩展目前不能生成 WSDL，您必须重头开始创建一个 WSDL 或者使用第三方的生成器。

> 这里有几个通过 PHP 实现的 SOAP 服务器。比如 [Zend SOAP](http://framework.zend.com/manual/current/en/modules/zend.soap.server.html "Zend SOAP") 和 [NuSOAP ](http://sourceforge.net/projects/nusoap/ "NuSOAP")。虽然在这些示例中只使用了 PHP SOAP 扩展，不过这个想法仍然应该适用于其他的实现。

SOAP 通过把 PHP 对象的方法公开给一个外部的实体（即使用 SOAP 服务的人 ）。首先，创建一个名为 **-HelloService-**  的类去表示您将要在 SOAP 中公开的功能。在这种情况下，SOAP 服务将会允许客户端去调用一个名为 **hello** 的方法用来发送一封电子邮件：

```
// src/Acme/SoapBundle/Services/HelloService.php
namespace Acme\SoapBundle\Services;

class HelloService
{
    private $mailer;

    public function __construct(\Swift_Mailer $mailer)
    {
        $this->mailer = $mailer;
    }

    public function hello($name)
    {

        $message = \Swift_Message::newInstance()
                                ->setTo('me@example.com')
                                ->setSubject('Hello Service')
                                ->setBody($name . ' says hi!');

        $this->mailer->send($message);

        return 'Hello, '.$name;
    }
}
```

接下来，您可以试图让 Symfony 能够创建一个该类的实例。因为该类需要发送邮件，所以在设计该类的时候就应该让它接受一个 **Swift_Mailer** 实例。您可以使用服务控制器来配置 Symfony 来构造一个 HelloService 对象：

YAML:

```
# app/config/services.yml
services:
    hello_service:
        class: Acme\SoapBundle\Services\HelloService
        arguments: ["@mailer"]
```

XML:

```
<!-- app/config/services.xml -->
<services>
    <service id="hello_service" class="Acme\SoapBundle\Services\HelloService">
        <argument type="service" id="mailer"/>
    </service>
</services>
```

PHP:

```
// app/config/services.php
$container
    ->register('hello_service', 'Acme\SoapBundle\Services\HelloService')
    ->addArgument(new Reference('mailer'));
```

下面是一个能够处理 SOAP 请求的控制器的示例。如果 **indexAction()** 可以通过路由或者 **soap** 协议访问，那么 WDSL文档就可以通过 **soap** 或 **wsdl** 协议检索。

```
namespace Acme\SoapBundle\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\Controller;
use Symfony\Component\HttpFoundation\Response;

class HelloServiceController extends Controller
{
    public function indexAction()
    {
        $server = new \SoapServer('/path/to/hello.wsdl');
        $server->setObject($this->get('hello_service'));

        $response = new Response();
        $response->headers->set('Content-Type', 'text/xml; charset=ISO-8859-1');

        ob_start();
        $server->handle();
        $response->setContent(ob_get_clean());

        return $response;
    }
}
```

请留意对  **ob_start()** 和 **ob_get_clean()** 方法的调用。这些方法控制着某些输出缓冲 [output buffering](http://php.net/manual/en/book.outcontrol.php "output buffering") ，这些缓冲允许您去接受 **$server->handle()** 的输出响应。这是非常有必要的，因为 Symfony 期望您的控制器返回一个带有把其输出当成它的内容的响应对象。并且，您必须把头部的 ”Content-Type“ 属性设置为 “text/xml”,这同样也是客户端所期望的。所以，您可以使用 **ob_start()**  来开始对 STDOUT 的缓冲，并且使用 **ob_get_clean()** 来把输出响应转存到响应的内容并且清理缓冲区。最后，就可以准备返回响应了。

下面是一个通过使用 [NuSOAP](http://sourceforge.net/projects/nusoap/ "NuSOAPg") 来调用服务的例子。假定在这个例子中，上诉控制器中的 **indexAction** 是通过路由或者 **soap** 协议访问的：

```
$client = new \Soapclient('http://example.com/app.php/soap?wsdl', true);

$result = $client->call('hello', array('name' => 'Scott'));
```

下面是一个 WSDL 的例子：

```
<?xml version="1.0" encoding="ISO-8859-1"?>
<definitions xmlns:SOAP-ENV="http://schemas.xmlsoap.org/soap/envelope/"
    xmlns:xsd="http://www.w3.org/2001/XMLSchema"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:SOAP-ENC="http://schemas.xmlsoap.org/soap/encoding/"
    xmlns:tns="urn:arnleadservicewsdl"
    xmlns:soap="http://schemas.xmlsoap.org/wsdl/soap/"
    xmlns:wsdl="http://schemas.xmlsoap.org/wsdl/"
    xmlns="http://schemas.xmlsoap.org/wsdl/"
    targetNamespace="urn:helloservicewsdl">

    <types>
        <xsd:schema targetNamespace="urn:hellowsdl">
            <xsd:import namespace="http://schemas.xmlsoap.org/soap/encoding/" />
            <xsd:import namespace="http://schemas.xmlsoap.org/wsdl/" />
        </xsd:schema>
    </types>

    <message name="helloRequest">
        <part name="name" type="xsd:string" />
    </message>

    <message name="helloResponse">
        <part name="return" type="xsd:string" />
    </message>

    <portType name="hellowsdlPortType">
        <operation name="hello">
            <documentation>Hello World</documentation>
            <input message="tns:helloRequest"/>
            <output message="tns:helloResponse"/>
        </operation>
    </portType>

    <binding name="hellowsdlBinding" type="tns:hellowsdlPortType">
        <soap:binding style="rpc" transport="http://schemas.xmlsoap.org/soap/http"/>
        <operation name="hello">
            <soap:operation soapAction="urn:arnleadservicewsdl#hello" style="rpc"/>

            <input>
                <soap:body use="encoded" namespace="urn:hellowsdl"
                    encodingStyle="http://schemas.xmlsoap.org/soap/encoding/"/>
            </input>

            <output>
                <soap:body use="encoded" namespace="urn:hellowsdl"
                    encodingStyle="http://schemas.xmlsoap.org/soap/encoding/"/>
            </output>
        </operation>
    </binding>

    <service name="hellowsdl">
        <port name="hellowsdlPort" binding="tns:hellowsdlBinding">
            <soap:address location="http://example.com/app.php/soap" />
        </port>
    </service>
</definitions>
```
