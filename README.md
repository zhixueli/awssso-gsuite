# AWS SSO 与Google G-Suite 账号通过 SAML2.0 方式集成实现联合身份认证登录 AWS Console

## 简介

本文介绍使用 G-Suite (Google Workspaces) 作为 AWS 的身份提供商 (IdP)。通过 SAML2.0 将 AWS SSO 与 G-Suite 集成，允许用户使用 G-Suite 账号登录访问 AWS Organizations 管理的 AWS 账户的控制台。

## 方案架构以及流程 

![alt text](https://github.com/zhixueli/awssso-gsuite/blob/main/images/G-Suite-AWS-SSO-Figure-1.png?raw=true)

AWS SSO 基于 SAML 2.0 协议对 G-Suite 用户进行身份验证，方案登录流程如下：

* 通过浏览器方案 AWS SSO 登录页面
* 用户将被重定向到 G-Suite 登录页面，并使用 G-Suite 账户登录
* 如果登录成功，会创建 SAML 断言响应并发送到 AWS SSO，SAML响应包含用户身份和认证信息
* AWS SSO 收到响应后，会判断用户对 AWS SSO 的访问权限，并显示可访问的 AWS 账户
* 用户选择要访问的账户并通过提供的链接访问 AWS 管理控制台

对于访问 AWS 账号的用户和权限管理，不需要在 AWS IAM 中操作，只需要在 AWS SSO 中利用 permission sets 和 groups 统一管理即可。 Permission sets 是定义用户访问 AWS 资源权限的集合，可使用 AWS 托管的 IAM Policy，也可以自定义 Policy。对于 AWS SSO 用户的创建和配置以及与G-Suite的集成，只需要在 AWS SSO 中创建新用户，并配置与 G-Suite 账户的 Email 地址的映射关系即可。
在 AWS SSO 上配置用户和相应的权限策略之后，系统会在给定的 AWS 账户中自动创建相应的 IAM role，并给 role 分配给定的权限。授权的 G-Suite 用户在登录成功后，会通过 assume role 的方式得到相应 AWS 账号的资源访问权限。

## 配置步骤

### 0. 前提条件

在进行 AWS SSO 配置之前，需要满足一些前提条件，详情请参考：https://docs.aws.amazon.com/singlesignon/latest/userguide/prereqs.html

* 需要预先配置 AWS Organizations，启用 All Features，并使用 AWS Organizations 主账号登陆 AWS 控制台来配置 AWS SSO
* 需要具有 Google Admin Console 的管理员权限的 G-Suite 用户进行操作

### 1. AWS SSO 初始化配置

#### 1.1. 访问 https://console.aws.amazon.com/singlesignon，进入 AWS SSO 首页，启用 AWS SSO

![alt text](https://github.com/zhixueli/awssso-gsuite/blob/main/images/G-Suite-AWS-SSO-Figure-2.png?raw=true)

#### 1.2. AWS SSO 启用之后，在 AWS SSO Dashboard 页面，选择 Choose your identity source

![alt text](https://github.com/zhixueli/awssso-gsuite/blob/main/images/G-Suite-AWS-SSO-Figure-3.png?raw=true)

#### 1.3. 在 Settings 页面，选择 identity source 旁边的 Change

![alt text](https://github.com/zhixueli/awssso-gsuite/blob/main/images/G-Suite-AWS-SSO-Figure-4.png?raw=true)

#### 1.4. 默认情况下，AWS SSO 使用自己的目录作为Idp。 要将 G-Suite 作为Idp，需要切换到外部Idp

![alt text](https://github.com/zhixueli/awssso-gsuite/blob/main/images/G-Suite-AWS-SSO-Figure-5.png?raw=true)

#### 1.5. 选择外部Idp会显示一些额外的信息，选择 Show individual metadata values，所显示的信息会在接下来的步骤中使用到

![alt text](https://github.com/zhixueli/awssso-gsuite/blob/main/images/G-Suite-AWS-SSO-Figure-6.png?raw=true)

### 2. 在 Google WorkSpaces 上配置 SAML 应用

#### 2.1. 访问 https://admin.google.com/，进入 Google Admin Console，左侧菜单栏进入 Apps -> Web and mobile apps，选择 Add app -> Add custom SAML app

![alt text](https://github.com/zhixueli/awssso-gsuite/blob/main/images/G-Suite-AWS-SSO-Figure-7.png?raw=true)

#### 2.2. 在 App details 页面，填入一个合适的 App 的名字，选择 Continue

#### 2.3. 在 Google identity provider details 页面，选择 Option 1：Download Metadata，将会下载文件名为 GoogleIDPMetadata.xml 的文件到本地，这个文件会在后续步骤配置 AWS SSO 时用到

![alt text](https://github.com/zhixueli/awssso-gsuite/blob/main/images/G-Suite-AWS-SSO-Figure-8.png?raw=true)

#### 2.4. 在 Service provider details 页面，使用在步骤 1.5 中 AWS SSO 界面上的 individual metadata values，填入如下对应的信息：

* ACS URL：填入 AWS SSO ACS URL
* Entity ID：填入 AWS SSO Issue URL
* Start URL：留空
* Name ID format：选择 EMAIL
* Name ID： 选择 Basic Information > Primary email

![alt text](https://github.com/zhixueli/awssso-gsuite/blob/main/images/G-Suite-AWS-SSO-Figure-9.png?raw=true)

#### 2.5. 在 Attribute Mapping 页面，默认配置即可，直接选择 Finish

#### 2.6. 在新建的 App 页面，User Access 部分，点击右侧的扩展按钮：

![alt text](https://github.com/zhixueli/awssso-gsuite/blob/main/images/G-Suite-AWS-SSO-Figure-10.png?raw=true)

#### 2.7. 在 Service Status 部分，选择 On for everyone 并选择 Save。至此，已经完成了在 Google WorkSpaces 上的初步配置

![alt text](https://github.com/zhixueli/awssso-gsuite/blob/main/images/G-Suite-AWS-SSO-Figure-11.png?raw=true)

### 3. AWS SSO 后续配置

#### 3.1. 在步骤 1.5 中配置 AWS SSO 外部 Idp 的页面，选择 Browse，上传在步骤 2.3 中下载的 Metadata 文件，然后选择 Next：Review

![alt text](https://github.com/zhixueli/awssso-gsuite/blob/main/images/G-Suite-AWS-SSO-Figure-12.png?raw=true)

#### 3.2. 在 Review and Confirm 页面，在最下方文本框中，填入 Confirm，然后选择 Change identity source 完成配置

![alt text](https://github.com/zhixueli/awssso-gsuite/blob/main/images/G-Suite-AWS-SSO-Figure-13.png?raw=true)

#### 3.3. 接下来页面中会提示 AWS SSO 配置已完成，选择 Return to settings 进行用户和权限相关配置

![alt text](https://github.com/zhixueli/awssso-gsuite/blob/main/images/G-Suite-AWS-SSO-Figure-14.png?raw=true)

### 4. AWS SSO 用户和权限配置

#### 4.1. 在 AWS SSO 页面，左侧菜单栏选择 Users，然后选择 Add user 来添加 SSO 用户

![alt text](https://github.com/zhixueli/awssso-gsuite/blob/main/images/G-Suite-AWS-SSO-Figure-15.png?raw=true)

#### 4.2. 在 User details 页面，Username 和 Email address 请填入相应的 Google Workspaces 用户 Email，然后选择Next：Groups

![alt text](https://github.com/zhixueli/awssso-gsuite/blob/main/images/G-Suite-AWS-SSO-Figure-16.png?raw=true)

#### 4.3. 在接下来的步骤中，可以把 user 加入到某一个已有的 user groups 或者新建一个 user groups 并加入其中。User groups 是具有相同 AWS 资源访问权限（Permission Sets）的一组用户集合，比如数据库管理员，超级管理员等等，可以为需要相同权限的用户建立 User groups，之后新增的需要相同权限的用户可直接加入 User groups，无需单独为用户分配权限。如果需要单独分配权限，请略过 Add user to groups 这一步，直接选择 Add user

![alt text](https://github.com/zhixueli/awssso-gsuite/blob/main/images/G-Suite-AWS-SSO-Figure-17.png?raw=true)

#### 4.4. 在 AWS Accounts 页面，会列出 AWS Organization 下的所有 AWS 账户，需要为新加的用户选择可以访问哪些 AWS 账户，选择完成后，点击 Assign users

![alt text](https://github.com/zhixueli/awssso-gsuite/blob/main/images/G-Suite-AWS-SSO-Figure-18.png?raw=true)

#### 4.5. 在 Select users or groups 页面，选择需要访问在步骤 4.4 中指定的 AWS 账号的用户或组，然后选择 Next: Permission sets 来分配权限

![alt text](https://github.com/zhixueli/awssso-gsuite/blob/main/images/G-Suite-AWS-SSO-Figure-19.png?raw=true)

#### 4.6. 在 Select permission sets 页面，选择已有或者通过 Create new permission set 创建一个新的权限集。Permission set 是一组管理员定义的策略，AWS SSO 使用这些策略来确定用户访问给定 AWS 账户的有效权限。 Permission set 可以包含 AWS 托管策略或存储在 AWS SSO 中的自定义策略，与 AWS IAM Policy 一致。Permission set 存储在 AWS SSO 中，仅用于 AWS 账户。 Permission set 最终在给定的 AWS 账户中创建 IAM 角色并赋予角色给定权限，通过信任关系策略允许用户通过 AWS SSO 承担这个角色。

![alt text](https://github.com/zhixueli/awssso-gsuite/blob/main/images/G-Suite-AWS-SSO-Figure-20.png?raw=true)

#### 4.7. 在 Create new permission set 页面，可以选择已有策略，或者新建一个自定义策略，与创建 IAM Policy类似。选择创建自定义策略过程中，可以选择用户 SSO 用户登录 AWS Console 有效时间，默认为一小时。本例中选择 Use an existing job function policy 来选择已有策略 PowerUserAccess

![alt text](https://github.com/zhixueli/awssso-gsuite/blob/main/images/G-Suite-AWS-SSO-Figure-21.png?raw=true)

#### 4.8. 最后选择 Finish 完成配置，如果添加新用户，请重复步骤 4.1-4.7，添加之前需要确认添加用户的 Email 在 Google Workspaces 已存在相应账号

![alt text](https://github.com/zhixueli/awssso-gsuite/blob/main/images/G-Suite-AWS-SSO-Figure-22.png?raw=true)

### 5. 测试 AWS SSO 用户登录

#### 5.1. 在 AWS SSO 页面，左侧菜单栏选择 Settings，点击 User portal URL 访问登录地址

![alt text](https://github.com/zhixueli/awssso-gsuite/blob/main/images/G-Suite-AWS-SSO-Figure-23.png?raw=true)

#### 5.2. 浏览器将跳转至 Google 账号登录页面进行认证，输入在 AWS SSO 中已配置的 Google Workspaces 账号信息认证通过后，将会跳转回 AWS SSO 页面，本页面将会列出前序步骤中已配置授权的 AWS 账号以及用户信息和访问 AWS Console 的链接地址，点击即可跳转至相应账号的 AWS Console

![alt text](https://github.com/zhixueli/awssso-gsuite/blob/main/images/G-Suite-AWS-SSO-Figure-24.png?raw=true)