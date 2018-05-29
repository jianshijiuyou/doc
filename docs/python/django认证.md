# Django 中的身份认证

Django 带有一个用户认证系统。用于处理用户帐户，组，权限和基于 cookie 的用户会话。本文档的这一部分解释了默认实现如何开箱即用，以及如何扩展和定制它以适应您的项目需求。

## 概览

认证系统由以下部分组成：

* 用户(Users)
* 权限(Permissions)：标志指定用户是否可以执行特定任务。
* 组(Groups)：将标签和权限应用于多个用户的通用方式。
* 一个可配置的密码散列系统
* 表单和查看工具，用于登录用户或限制内容
* 可插入的后端系统

Django 中的认证系统的目标是非常通用，并且不提供 Web 认证系统中常见的一些功能。第三方软件包已经实施了一些常见问题的解决方案：

* 密码强度检查
* 限制登录尝试
* 针对第三方的身份认证（例如，OAuth）

## 安装

身份认证 ( Authentication ) 在 `django.contrib.auth` 中作为 Django contrib 模块捆绑在一起。默认情况下，所需的配置已包含在由 `django-admin startproject` 生成的 `settings.py` 中，这些配置由 `INSTALLED_APPS` setting 中列出的两个项目组成：

1. `'django.contrib.auth'` 包含身份认证 ( authentication ) 框架的核心，以及它的默认模型。
2. `'django.contrib.contenttypes'` 是 Django 内容类型系统，它允许权限与您创建的模型相关联。

以及 `MIDDLEWARE` setting 中的这些项目：

1. `SessionMiddleware` 管理跨请求的会话。
2. `AuthenticationMiddleware` 使用会话将用户与请求相关联。

使用这些设置后，运行命令 `manage.py migrate` 将为 auth 相关模型创建必要的数据库表，并为安装的应用程序中定义的模型创建权限。

# 使用 Django 身份认证系统

本文档解释了 Django 认证系统在其默认配置中的用法。这种配置已经发展到满足最常见的项目需求，处理合理范围广泛的任务，并且仔细实现了密码和权限。对于认证需求与默认不同的项目，Django 支持广泛的扩展和定制认证。

Django 中的认证系统认证和授权在一起，因为这些功能有些耦合。

## User 对象

`User` 对象是认证系统的核心。他们通常代表与您的网站进行互动的人，并用于实现限制访问，注册用户信息，将内容与创作者相关联等。在 Django 的身份验证框架中只存在一类用户，即 `'superusers'` 或 admin `'staff'` 用户是具有特殊属性集的用户对象，而不是不同类的用户对象。

默认用户的主要属性是：

* `username`
* `password`
* `email`
* `first_name`
* `last_name`

### 创建用户

创建用户最直接的方法是使用包含的 `create_user()` 辅助函数：

``` python
>>> from django.contrib.auth.models import User
>>> user = User.objects.create_user('john', 'lennon@thebeatles.com', 'johnpassword')

# At this point, user is a User object that has already been saved
# to the database. You can continue to change its attributes
# if you want to change other fields.
>>> user.last_name = 'Lennon'
>>> user.save()
```

### 创建超级用户

使用 `createsuperuser` 命令创建超级用户：

``` python
$ python manage.py createsuperuser --username=joe --email=joe@example.com
```

系统会提示您输入密码。输入之后，用户将立即被创建。如果您不使用 `--username` 或 `--email` 选项，它会提示您输入这些值。

### 更改密码

Django 不会在用户模型中存储原始（纯文本）密码，而只存储哈希值。因此，请勿尝试直接操作用户的密码属性。这就是创建用户时使用帮助函数的原因。

要更改用户的密码，您有几个选择：

`manage.py changepassword <username>` 提供了一种从命令行更改用户密码的方法。它会提示您更改您必须输入两次的给定用户的密码。如果两者都匹配，则新密码将立即更改。如果您不提供用户，则该命令将尝试更改其用户名与当前系统用户匹配的密码。

您还可以使用 `set_password()` 以编程方式更改密码：

``` python
>>> from django.contrib.auth.models import User
>>> u = User.objects.get(username='john')
>>> u.set_password('new password')
>>> u.save()
```

如果您安装了 Django admin，您还可以在认证系统的管理页面上更改用户的密码。

Django 还提供了可用于允许用户更改自己的密码的视图和表单。

更改用户的密码将注销其所有会话。

### 用户认证

`authenticate(request=None, **credentials)`

使用 `authenticate()` 来验证一组凭据。它将凭据作为关键字参数，默认情况下是 `username` 和 `password`，并针对每个验证后端进行检查，并在凭据对后端有效时返回 `User` 对象。如果凭证对任何后端无效，或者后端引发 `PermissionDenied`，则返回 `None`。例如：

``` python
from django.contrib.auth import authenticate
user = authenticate(username='john', password='secret')
if user is not None:
    # A backend authenticated the credentials
else:
    # No backend authenticated the credentials
```

`request` 是在身份验证后端的 `authenticate()` 方法上传递的可选 `HttpRequest`。

> 这是验证一组凭据的低级别方法;例如，它由 `RemoteUserMiddleware` 使用。除非你正在编写你自己的认证系统，否则你可能不会使用它。


!> 后面内容如果使用 DRF 框架，基本用不到了，所以省略咯～

# django.contrib.auth 模块

## User 模型

### 字段

`username`

必须。150 个字符或更少。用户名可能包含字母数字，`_`，`@`，`+`，。和 `-` 字符。如果您使用的 MySQL，请指定 `max_length=191`，因为默认情况下，MySQL 只能创建 191 个字符的唯一索引。

`first_name`

可选 (`blank=True`)。 30 个字符或更少。

`last_name`

可选 (`blank=True`)。 150 个字符或更少。

`email`

可选 (`blank=True`)。邮箱地址。

`password`

必须。密码的散列和元数据。（Django 不存储原始密码。）原始密码可以是任意长的，并且可以包含任何字符。

`groups`

多对多关系 `Group`

`user_permissions`

多对多关系 `Permission`

`is_staff`

Bollean 类型。指定此用户是否可以访问 admin 站点。

`is_active`

Bollean 类型。指定是否应将此用户帐户视为活动用户。我们建议您将此标志设置为 `False` 而不是删除帐户;这样，如果您的应用程序对用户有任何外键，外键不会中断。

`is_superuser`

Bollean 类型。指定该用户具有所有权限而不明确分配它们。

`last_login`

用户上次登录的日期时间。

`date_joined`

指定帐户何时创建的日期时间。在创建帐户时默认设置为当前日期/时间。

### 属性

`is_authenticated`

只读属性始终为 `True`（与 `AnonymousUser.is_authenticated` 相反，它始终为 `False`）。这是判断用户是否已通过身份验证的一种方式。这并不意味着任何权限，也不会检查用户是否处于活动状态或是否有有效的会话。

`is_anonymous`

只读属性始终为 `False`。这是区分 `User` 和 `AnonymousUser` 对象的一种方式。通常，您应该更喜欢使用 `is_authenticated` 属性。

### 方法

`get_username()`

返回用户的用户名。由于 `User` 模型可以被换出，您应该使用此方法而不是直接引用 `username` 属性。

`get_full_name()`

返回 `first_name` 加上 `last_name`，之间有一个空格。

`get_short_name()`

返回 `first_name`。

`set_password(raw_password)`

将用户的密码设置为给定的原始字符串，注意密码散列。不保存用户对象。

当 `raw_password` 为 `None` 时，密码将被设置为不可用的密码，就像使用 `set_unusable_password()` 一样。

`check_password(raw_password)`

如果给定的原始字符串是用户的正确密码，则返回 `True`。（在进行比较时，会将密码哈希处理。）

`set_unusable_password()`

标记用户没有设置密码。这与为密码输入空白字符串不同。该用户的 `check_password()` 将永远不会返回 `True`。不保存 `User` 对象。

如果您的应用程序的身份验证是针对现有的外部源（例如 LDAP 目录）进行的，您可能需要使用此功能。

`has_usable_password()`

如果为此用户调用了 `set_unusable_password()`，则返回 `False`。

`email_user(subject, message, from_email=None, **kwargs)`

向用户发送电子邮件。

### Manager 方法

`class models.UserManager`

`User` 模型有一个自定义管理器，它具有以下辅助方法（除了由 `BaseUserManager` 提供的方法外）：

`create_user(username, email=None, password=None, **extra_fields)`

创建，保存并返回 `User`。

`username` 和 `password` 设置为给定。 `email` 的域部分将自动转换为小写，并且返回的 `User` 对象将 `is_active` 设置为 `True`。

如果未提供密码，则将调用 `set_unusable_password()`。

`extra_fields` 关键字参数传递给 `User` 的 `__init__` 方法，以允许在自定义用户模型上设置任意字段。

`create_superuser(username, email, password, **extra_fields)`

与 `create_user()` 相同，但将 `is_staff` 和 `is_superuser` 设置为 `True`。

## 实用函数

`get_user(request)`

返回与给定请求的会话关联的用户模型实例。

它检查存储在会话中的认证后端是否存在于 `AUTHENTICATION_BACKENDS` 中。如果存在，它使用后端的 `get_user()` 方法来检索用户模型实例，然后通过调用用户模型的 `get_session_auth_hash()` 方法来验证会话。

如果存储在会话中的认证后端不再位于 `AUTHENTICATION_BACKENDS` 中，如果用户未由后端的 `get_user()` 方法返回，或者如果会话认证哈希未验证，则返回 `AnonymousUser` 的实例。

# 自定义 Django 中的认证机制

## 指定身份认证后端

在幕后，Django 维护着一个 “身份认证后端” 列表，用于检查身份认证。当有人调用 `django.contrib.auth.authenticate()` 时，Django 尝试在所有身份验证后端进行身份验证。如果第一个验证方法失败，Django 会尝试第二个验证方法，依此类推，直到尝试完所有后端。