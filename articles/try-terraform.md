---
title: "AWSの無料枠がなくなった今、Terraformを試したい"
emoji: "🕌"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["aws","terraform"]
published: false
---

## あらためてはじめに
TerraformはIaC(Infrastructure as Code)の1つ。
複数のクラウド環境で使えて、CI/CD的にも良いというメリットもあります。
コードとしてインフラ環境を構築できるため、ソースコードと合わせてバージョン管理できてうれしいみたいなこともあるみたいです。

ずっと気になってたんですが、いい加減触ろうと思っていたらEC2の無料枠とかなくなっていたのですが一旦ざっくり理解できるまでやってみようと思いました。

## やってみる
### 書き始めるまで
個人的に一番触ることが多いWebアプリケーションのよくあるEC2・RDS・S3あたりで試してみます。

VSCodeで書くつもりなので拡張機能入れます。
https://marketplace.visualstudio.com/items?itemName=HashiCorp.terraform
