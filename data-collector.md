# 如何创建一个自定义的数据收集器

[Symfony 的分析器](http://symfony.com/doc/current/cookbook/profiler/index.html)将数据采集功能委托给数据收集器。Symfony 在默认上捆绑了一些数据收集器，但是你可以很容易地按照自己的需求创建自定义的收集器。

## 创建一个自定义的数据收集器
创建一个自定义的数据收集器，所需要做的实质工作就是实现数据收集器接口类[DataCollectorInterface](http://api.symfony.com/2.7/Symfony/Component/HttpKernel/DataCollector/DataCollectorInterface.html)：

```

interface DataCollectorInterface
{
    /**
     * Collects data for the given Request and Response.
     *
     * @param Request    $request   A Request instance
     * @param Response   $response  A Response instance
     * @param \Exception $exception An Exception instance
     */
    function collect(Request $request, Response $response, \Exception $exception = null);

    /**
     * Returns the name of the collector.
     *
     * @return string The collector name
     */
    function getName();
}

```


上述的 getName() 方法返回值必须是一个独一无二的名字。这是被用来之后进行信息访问的依据（可以在[功能测试章查看如何使用分析器文档](http://symfony.com/doc/current/cookbook/testing/profiling.html)来获得使用实例）。  

而 collect() 方法负责存储想要被提供给局部属性的数据。


> 为探查序列化数据采集器的情况下，你不应该存储不可序列化的对象（如 PDO 对象），或者你需要提供自己的 serialize() 方法。  
  

大部分的时间，扩展数据采集器和填充 $this->数据属性（它需要序列化 $this->数据属性）是非常方便的：  

```

class MemoryDataCollector extends DataCollector
{
    public function collect(Request $request, Response $response, \Exception $exception = null)
    {
        $this->data = array(
            'memory' => memory_get_peak_usage(true),
        );
    }

    public function getMemory()
    {
        return $this->data['memory'];
    }

    public function getName()
    {
        return 'memory';
    }
}


```  

## 使自定义数据收集器生效

为使一个数据收集器生效，将它添加入你的一个配置表中的常规服务，并把它附属在data_collector之后:  

YAML:  
```       
services:
    data_collector.your_collector_name:
        class: Fully\Qualified\Collector\Class\Name
        tags:
            - { name: data_collector }
           
```   

XML:
``` 
<service id="data_collector.your_collector_name" class="Fully\Qualified\Collector\Class\Name">
    <tag name="data_collector" />
</service>
``` 
  
PHP:
``` 
$container
    ->register('data_collector.your_collector_name', 'Fully\Qualified\Collector\Class\Name')
    ->addTag('data_collector')
;

``` 

## 添加网络分析器模板 
当你想在网络调试工具条或网络分析器上显示你的数据收集器收集的数据，您将需要创建一个 Twig 模板。下面的例子可以帮助你开始：   
```   
{% extends 'WebProfilerBundle:Profiler:layout.html.twig' %}

{% block toolbar %}
    {# This toolbar item may appear along the top or bottom of the screen.#}
    {% set icon %}
    <span class="icon"><img src="data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAABoAAAAcCAQAAADVGmdYAAAAAmJLR0QA/4ePzL8AAAAJcEhZcwAACxMAAAsTAQCanBgAAAAHdElNRQffAxkBCDStonIVAAAAGXRFWHRDb21tZW50AENyZWF0ZWQgd2l0aCBHSU1QV4EOFwAAAHpJREFUOMtj3PWfgXRAuqZd/5nIsIdhVBPFmgqIjCuYOrJsYtz1fxuUOYER2TQID8afwIiQ8YIkI4TzCv5D2AgaWSuExJKMIDbA7EEVhQEWXJ6FKUY4D48m7HYU/EcWZ8JlE6qfMELPDcUJuEMPxvYazYTDWRMjOcUyAEswO+VjeQQaAAAAAElFTkSuQmCC" alt=""/></span>
    <span class="sf-toolbar-status">Example</span>
    {% endset %}

    {% set text %}
    <div class="sf-toolbar-info-piece">
        <b>Quick piece of data</b>
        <span>100 units</span>
    </div>
    <div class="sf-toolbar-info-piece">
        <b>Another quick thing</b>
        <span>300 units</span>
    </div>
    {% endset %}

    {# Set the "link" value to false if you do not have a big "panel"
       section that you want to direct the user to. #}
    {% include '@WebProfiler/Profiler/toolbar_item.html.twig' with { 'link': true } %}

{% endblock %}

{% block head %}
    {# Optional, if you need your own JS or CSS files. #}
    {{ parent() }} {# Use parent() to keep the default styles #}
{% endblock %}

{% block menu %}
    {# This left-hand menu appears when using the full-screen profiler. #}
    <span class="label">
        <span class="icon"><img src="data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAABoAAAAcCAQAAADVGmdYAAAAAmJLR0QA/4ePzL8AAAAJcEhZcwAACxMAAAsTAQCanBgAAAAHdElNRQffAxkBCDStonIVAAAAGXRFWHRDb21tZW50AENyZWF0ZWQgd2l0aCBHSU1QV4EOFwAAAHpJREFUOMtj3PWfgXRAuqZd/5nIsIdhVBPFmgqIjCuYOrJsYtz1fxuUOYER2TQID8afwIiQ8YIkI4TzCv5D2AgaWSuExJKMIDbA7EEVhQEWXJ6FKUY4D48m7HYU/EcWZ8JlE6qfMELPDcUJuEMPxvYazYTDWRMjOcUyAEswO+VjeQQaAAAAAElFTkSuQmCC" alt=""/></span>
        <strong>Example Collector</strong>
    </span>
{% endblock %}

{% block panel %}
    {# Optional, for showing the most details. #}
    <h2>Example</h2>
    <p>
        <em>Major information goes here</em>
    </p>
{% endblock %}


``` 

每个块都是可选的。工具栏块用于网页调试工具，菜单和面板同于将一个面板添加到网络分析器中。

所有块都有访问收集器对象的权限。  


>    内置的模板使用 Base64 编码的图像工具栏：
>    ``` 
>    <img src="data:image/png;base64,..." />  
>    ``` 
>    
>    你可以很容易利用这个小脚本计算出图像的 base64 值：
>    ``` 
>    #!/usr/bin/env php
>    <?php
>    echo base64_encode(file_get_contents($_SERVER['argv'][1]));
>    ``` 


为了使模板生效，在您的配置的 data_collector 标签中添加模板属性。例如，假设你的模板是在一些AcmeDebugBundle 中，那么：  
YAML:  
```       
services:
    data_collector.your_collector_name:
        class: Acme\DebugBundle\Collector\Class\Name
        tags:
            - { name: data_collector, template: "AcmeDebugBundle:Collector:templatename", 
           
```   

XML:
``` 
<service id="data_collector.your_collector_name" class="Acme\DebugBundle\Collector\Class\Name">
    <tag name="data_collector" template="AcmeDebugBundle:Collector:templatename" id="your_collector_name" />
</service>
``` 
  
PHP:
``` 
$container
    ->register('data_collector.your_collector_name', 'Acme\DebugBundle\Collector\Class\Name')
    ->addTag('data_collector', array(
        'template' => 'AcmeDebugBundle:Collector:templatename',
        'id'       => 'your_collector_name',
    ))
;
```   

> 记得确保你的ID 属性与用于 getname() 方法的字符串相同。

这项工作的许可为 Creative Commons Attribution-Share Alike 3.0 Unported [License](http://creativecommons.org/licenses/by-sa/3.0/)。
