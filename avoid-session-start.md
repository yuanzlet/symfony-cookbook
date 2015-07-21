# 避免匿名用户开始 Session 会话

会议（Sessions）会在当您读、写，甚至仅仅是检查会话中的数据是否存在的时候自动启动。这意味着如果您需要避免为某些用户创建会话 cookie 是很难得：您必须**完全**避免访问会话。

例如，在这种情况下的一个常见的问题是检查存储在会话中的 flash 消息。以下代码将保证会话**始终**是开始的（started）。

```
{% for flashMessage in app.session.flashbag.get('notice') %}
    <div class="flash-notice">
        {{ flashMessage }}
    </div>
{% endfor %}
```

即使用户没有登录，即使您没有创建任何 flash 信息，只要调用 **flashbag** 的 **get()**（或是 **has()**）方法就可以启动一个会话。这可能会损伤您的应用程序性能，因为所有的用户都会收到一个会话 cookie。为了避免之一行为，添加一个 check 在尝试访问 flash 消息之前。

```
{% if app.request.hasPreviousSession %}
    {% for flashMessage in app.session.flashbag.get('notice') %}
        <div class="flash-notice">
            {{ flashMessage }}
        </div>
    {% endfor %}
{% endif %}
```
