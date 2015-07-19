# 如何编程访问分析器数据

大多数时候，分析器信息的访问和分析是基于 Web 的可视化的。当然，你也可以利用分析器服务提供的方法以编程方式检索分析信息。  

当回复对象是可用的，使用 [loadProfileFromResponse() 方法](http://api.symfony.com/2.7/Symfony/Component/HttpKernel/Profiler/Profiler.html#loadProfileFromResponse())获得其相关分析器权限： 
```
// ... $profiler is the 'profiler' service
$profile = $profiler->loadProfileFromResponse($response); 

```  

当分析器存储了关于请求的数据时，它还将为之绑定一个令牌；这个令牌在响应的 X-Debug-Token HTTP头中是可用的。使用此令牌，你可以利用 [loadProfile()](http://api.symfony.com/2.7/Symfony/Component/HttpKernel/Profiler/Profiler.html#loadProfile()) 方法访问任何过去的响应：  
```
$token = $response->headers->get('X-Debug-Token');
$profile = $container->get('profiler')->loadProfile($token);
```

> 当分析器启用而 Web 调试工具栏没有启用的话，使用您的浏览器的开发者工具获得的 X-Debug-Token HTTP 头部的值来检查页面。    

分析器服务也提供了 [find()](http://api.symfony.com/2.7/Symfony/Component/HttpKernel/Profiler/Profiler.html#find()) 方法来查看一些基于标准的令牌： 
```
// get the latest 10 tokens
$tokens = $container->get('profiler')->find('', '', 10, '', '');

// get the latest 10 tokens for all URL containing /admin/
$tokens = $container->get('profiler')->find('', '/admin/', 10, '', '');

// get the latest 10 tokens for local requests
$tokens = $container->get('profiler')->find('127.0.0.1', '', 10, '', '');

// get the latest 10 tokens for requests that happened between 2 and 4 days ago
$tokens = $container->get('profiler')
    ->find('', '', 10, '4 days ago', '2 days ago');
```
最后，如果你想在一个与生成信息的机器不同的机器上操纵分析数据的话，使用分析器：导出和分析器：导入命令： 
```
# on the production machine
$ php app/console profiler:export > profile.data

# on the development machine
$ php app/console profiler:import /path/to/profile.data

# you can also pipe from the STDIN
$ cat /path/to/profile.data | php app/console profiler:import
```

这项工作的许可为 Creative Commons Attribution-Share Alike 3.0 Unported [License](http://creativecommons.org/licenses/by-sa/3.0/)

