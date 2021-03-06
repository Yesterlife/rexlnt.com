---
title: AWS IAM
description: Who can use your AWS resources and what resources they can use and in what ways
date: 2019-09-28T02:32:53.402Z
---

AWS IAM（Identity and Access Management）是 AWS 的一项资源管控服务，管控的核心对象是使用者的身份和权限。通过 AWS IAM，AWS 的账户管理者可以做到三层粒度的资源管控：

- 谁可以使用你的 AWS 资源
- 谁可以使用你的哪一种 AWS 资源
- 谁可以对 AWS 资源进行何种操作，比如上传 S3、读取 DynamoDB 等

## 术语

- `user / user group`：注册 AWS 账户之后，系统会默认生成一个 Root 用户，可以用于访问所有的 AWS 服务。由于 Root 用户权限过大，实际开发中会使用权限更小、粒度更细的用户，这些用户可以归类到具体某个用户组，一旦用户组具有了某种权限，则组内所有用户都会具有相同的权限；
- `policy / permission`：一条 IAM 策略由多条权限规则组成，策略规定了对某种 AWS 资源某种操作的许可，即可以许可某种操作，也可以否定某种操作；
- `role`：角色类似于用户，但角色没有用户所拥有的凭证（密码、Access Keys），同时角色可以附加到任意的资源、用户上面，代表用户去执行某类操作；

下面演示的策略定义了用户有权对 us-east-2 区域内的 123456789012 账户中的 Books 表执行任意 Amazon DynamoDB 操作：

```json
{
  "Version": "2012-10-17",
  "Statement": {
    "Effect": "Allow",
    "Action": "dynamodb:*",
    "Resource": "arn:aws:dynamodb:us-east-2:123456789012:table/Books"
  }
}
```

## References

- [AWS IAM](https://docs.aws.amazon.com/zh_cn/IAM/latest/UserGuide/introduction.html)

