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