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

#### 3.1. 在新建的 AWS SSO 中，User Access 部分，点击右侧的扩展按钮：