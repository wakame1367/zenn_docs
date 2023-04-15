---
title: "GitHub Organizationの通知先の追加および変更方法"
emoji: "🔔"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["GitHub", "Tech", "メール"]
published: true
---

# はじめに

GitHubの個人アカウントを会社用として運用した場合、PR等のメール通知が個人メールアドレスに送信されてしまう。会社用のメールアドレスに通知が送信されるよう設定したい、その場合の設定方法を以下にまとめる。

# 通知先の変更方法

## GitHubの通知先メールアドレスを追加する

メールアドレスを追加後、追加したメールアドレス先に認証のためのメールが送信される。認証完了後、GitHubの通知先のメールアドレスが変更できるようになる。

- [GitHub - GitHub アカウントへのメールアドレスの追加](https://docs.github.com/ja/account-and-profile/setting-up-and-managing-your-personal-account-on-github/managing-email-preferences/adding-an-email-address-to-your-github-account)

## GitHub Organizationの通知先を変更する

[Github - settings > notifications > custom_routing](https://github.com/settings/notifications/custom_routing)より`Add new route`をクリック。`Pick organization`より自社のOrganization名を選択する。`Select Email`より追加したメールアドレスを選択する。

- [GitHub - Organization ごとにメールの送信先を設定する](https://docs.github.com/ja/account-and-profile/managing-subscriptions-and-notifications-on-github/setting-up-notifications/configuring-notifications#customizing-email-routes-per-organization)

# 参考文献

- [Zenn - GitHub を使うなら通知くらいまともに設定してくれ](https://zenn.dev/siketyan/articles/you-are-not-using-github-correctly)
- [GitHub - GitHub アカウントへのメールアドレスの追加](https://docs.github.com/ja/account-and-profile/setting-up-and-managing-your-personal-account-on-github/managing-email-preferences/adding-an-email-address-to-your-github-account)
- [GitHub - Organization のメール通知の送信先を選択する](https://docs.github.com/ja/account-and-profile/managing-subscriptions-and-notifications-on-github/setting-up-notifications/configuring-notifications#choosing-where-your-organizations-email-notifications-are-sent)
- [GitHub - Organization ごとにメールの送信先を設定する](https://docs.github.com/ja/account-and-profile/managing-subscriptions-and-notifications-on-github/setting-up-notifications/configuring-notifications#customizing-email-routes-per-organization)
