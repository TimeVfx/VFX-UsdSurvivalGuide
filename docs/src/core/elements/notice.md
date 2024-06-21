# Notices/Event Listeners
Usd's event system is exposed via the [Notice](https://openusd.org/dev/api/group__group__tf___notification.html) class. It can be used to subscribe to different stage events or custom events to get notified about changes.

[ Usd 的事件系统通过Notice 类公开. 它可用于订阅不同的 stage 事件或自定义事件以获取有关更改的通知]

A common case for using it with Python is sending update notifications to UIs.

[ 将其与 Python 一起使用的常见情况是向 UI 发送更新通知]

## TL;DR - Notice/Event Listeners In-A-Nutshell [概述]
- The event system is uni-directional: The listeners subscribe to senders, but can't send information back to the senders. The senders are also not aware of the listeners, senders just send the event and the event system distributes it to the listeners.

    [ 事件系统是单向的：监听器订阅发送者，但不能向发送者发送信息.发送者也不了解监听器，它们只是发送事件，事件系统会将事件分发给监听器]
- The listeners are called synchronously in a random order (per thread where the listener was created), so make sure your listeners action is fast or forwards its execution into a separate thread.

    [ 监听器是按随机顺序同步调用的（按监听器创建的线程），因此请确保监听器的操作足够快，或者将其执行转发到单独的线程中]

## What should I use it for? <a name="usage"></a>

[  我应该用它做什么？]

~~~admonish tip
In production, the mose common use case you'll use the notification system for is changes in the stage. You can use these notifications to track user interaction and to trigger UI refreshes.

[ 在生产中您将使用通知系统的最常见用例是 stage 中的更改, 您可以使用这些通知来跟踪用户交互并触发 UI 刷新]
~~~

## Resources [资源]
- [Usd Notice API Docs](https://openusd.org/dev/api/class_usd_notice.html)
- [Tf Notification API Docs](https://openusd.org/dev/api/group__group__tf___notification.html)

## Notice code examples

#### Register/Revoke notice

[ 注册/撤销通知]

~~~admonish info title=""
```python
{{#include ../../../../code/core/elements.py:noticeRegisterRevoke}}
```
~~~

#### Overview of built-in standard notices for stage change events

[ stage 变更事件的内置标准通知概述]

~~~admonish info title=""
```python
{{#include ../../../../code/core/elements.py:noticeCommon}}
```
~~~

If we run this on a simple example stage, we get the following results:

[ 如果我们在一个简单的示例阶段运行它，我们会得到以下结果]

~~~admonish info title=""
```python
{{#include ../../../../code/core/elements.py:noticeCommonApplied}}
```
~~~

#### Plugin Register Notice
The plugin system sends a notice whenever a new plugin was registered.

[ 插件系统会在每次注册新插件时发送一个通知]

~~~admonish info title=""
```python
{{#include ../../../../code/core/elements.py:noticePlugins}}
```
~~~

#### Setup a custom notice:

[ 设置自定义通知]

~~~admonish info title=""
```python
{{#include ../../../../code/core/elements.py:noticeCustom}}
```
~~~