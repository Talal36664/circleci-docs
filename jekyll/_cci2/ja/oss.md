---
layout: classic-docs
title: "オープンソースプロジェクトの構築"
short-title: "オープンソースプロジェクトの構築"
description: "オープンソースプロジェクトの構築に関するベストプラクティス"
categories:
  - getting-started
order: 1
---

ここでは、以下のセクションに沿って、CircleCI 上でのオープンソースプロジェクトのビルドに関するヒントとベストプラクティスについて説明します。

- 目次 {:toc}

## 概要

{:.no_toc}

オープンソースコミュニティをサポートする目的で、GitHub または Bitbucket 上のパブリックプロジェクトには 3つの無料のビルドコンテナが提供され、コンテナ数は合計 4つになります。 複数のビルドコンテナを使用することで、単一のプルリクエスト (PR) を並列処理で高速に、または複数の PR を一斉にビルドできます。

Linux 上で動作するパブリックプロジェクトの場合、これらの追加コンテナは自動的に有効になります。 追加コンテナを使用しない場合、または CircleCI プロジェクトをパブリックにしない場合には、この設定を変更できます。 プロジェクトの **[Advanced Settings (詳細設定)]** で **[Free and Open Source (無料オープンソース)]** オプションを*オフ*に設定します。

**メモ：**macOS 上のオープンソースプロジェクトをビルドする場合は、これらの追加コンテナを有効にする方法について billing@circleci.com にお問い合わせください。

## セキュリティ

オープンソースは開放型の活動であり、機密情報を「開放」しないように注意が必要です。

- リポジトリをパブリックにすると、CircleCI プロジェクトとそのビルドログもパブリックになります。 表示対象として選択する情報に注意してください。
- CircleCI アプリケーション内に設定される環境変数は、一般には公開されず、明示的に有効にされない限り[フォークされた PR](#pass-secrets-to-builds-from-forked-pull-requests) に共有されることもありません。

## オープンソースプロジェクトの機能と設定

以下の機能と設定は、オープンソースプロジェクトにおいて特に便利です。

### プライベート環境変数

{:.no_toc}

多くのプロジェクトでは、API トークン、SSH キー、またはパスワードが必要です。 プライベート環境変数を使用すると、プロジェクトがパブリックの場合でもシークレットを安全に格納できます。 詳細については、「[プロジェクト内で環境変数を設定する]({{ site.baseurl }}/ja/2.0/env-vars/#プロジェクト内で環境変数を設定する)」を参照してください。

### プルリクエストのみをビルドする

{:.no_toc}

CircleCI はデフォルトですべてのブランチのすべてのコミットをビルドします。 この動作は、オープンソースプロジェクトで使用するには活動的すぎることがあり、場合によってはプライベートプロジェクトよりもきわめて多くのコミットが存在することになります。 この設定を変更するには、プロジェクトの **[Advanced Settings (詳細設定)]** に移動して、**[Only build pull requests (プルリクエストのみビルド)]** オプションを*オン*に設定します。

**メモ：**このオプションが有効であっても、CircleCI はプロジェクトのデフォルトのブランチからはすべてのコミットをビルドします。

### フォークされたリポジトリからプルリクエストをビルドする

{:.no_toc}

多くのオープンソースプロジェクトは、フォークされたリポジトリから PR を受け入れます。 これらの PR をビルドすると、手動で変更をレビューする前にバグを捕捉することができるので、効果的な方法と言えます。

CircleCI はデフォルトで、フォークされたリポジトリからの PR をビルドしません。 この設定を変更するには、ユーザープロジェクトの **[Advanced Settings (詳細設定)]** に移動して、**[Build forked pull requests (フォークされたプルリクエストをビルド)]** オプションを*オン*に設定します。

### フォークされたプルリクエストからのビルドにシークレットを渡す

{:.no_toc}

制限を設定していないビルドを親リポジトリ内で実行することは、場合によっては危険です。 プロジェクトにはしばしば機密情報が含まれており、ビルドをトリガーするコードをプッシュするユーザーならだれでも、この情報を自由に入手できます。

オープンソースプロジェクトの場合、CircleCI のデフォルトでは、フォークされた PR からのビルドにシークレットを渡さず、以下の 4種類の設定データを隠します。

- アプリケーションを通して設定される[環境変数](#private-environment-variables)

- [デプロイキーとユーザーキー]({{ site.baseurl }}/ja/2.0/gh-bb-integration/#デプロイキーとユーザーキー)

- ビルド中に任意のホストにアクセスするために [CircleCI に追加した]({{ site.baseurl }}/ja/2.0/add-ssh-key)、パスフレーズのないプライベート SSH キー

- [AWS 権限]({{ site.baseurl }}/ja/2.0/deployment-integrations/#aws)および設定ファイル

**メモ：**シークレットを必要とするオープンソースプロジェクトのフォークされた PR ビルドは、この設定を有効にしない限り CircleCI 上で正しく動作しません。

プロジェクトをフォークし、PR をオープンする任意のユーザーとシークレットを共有しても問題がない場合は、**[Pass secrets to builds from forked pull requests (フォークされたプルリクエストからのビルドにシークレットを渡す)]** オプションを有効にできます。 プロジェクトの **[Advanced Settings (詳細設定)]** で **[Pass secrets to builds from forked pull requests (フォークされたプルリクエストからビルドにシークレットを渡す)]** オプションを*オン*に設定します。

### キャッシュ

キャッシュは、PR の GitHub リポジトリに基づいて分離されます。 CircleCI は、フォーク PR の生成元の GitHub リポジトリ ID を使用してキャッシュを識別します。

- 同じフォークリポジトリからの PR は、キャッシュを共有します (前述のように、これにはマスターリポジトリ内の PR とマスターによるキャッシュの共有が含まれます)。
- それぞれ異なるフォークリポジトリ内にある 2つの PR は、別々のキャッシュを持ちます。

現在、キャッシュの自動入力は行われていません。この最適化がまだ優先順位リストの上位に入っていないためです。

## オープンソースプロジェクトの例

CircleCI 上でビルドされたさまざまな規模のプロジェクトを以下にいくつかご紹介します。

- **[React](https://github.com/facebook/react)** - Facebook の JavaScript ベースの React は、CircleCI (および他の CI ツール) でビルドされています。 
- **[React Native](https://github.com/facebook/react-native/)** - JavaScript と React を使用してネイティブモバイルアプリケーションをビルドします。
- **[Flow](https://github.com/facebook/flow/)** - JavaScript に静的な型指定を追加して、開発者の生産性とコードの品質を向上させます。
- **[Relay](https://github.com/facebook/relay)** - データ駆動型の React アプリケーションをビルドするための JavaScript フレームワーク。 
- **[Vue](https://github.com/vuejs/vue)** - Vue.js は、Web 上で UI をビルドするための漸進的な JavaScript フレームワークであり、段階的に採用できます。
- **[Storybook](https://github.com/storybooks/storybook)** - 対話型 UI コンポーネントの開発とテストを行います (React、React Native、Vue、Angular、Ember)。
- **[Electron](https://github.com/electron/electron)** - JavaScript、HTML、および CSS でクロスプラットフォームのデスクトップアプリケーションをビルドします。
- **[Angular](https://github.com/angular/angular)** - ブラウザーおよびデスクトップ Web アプリケーションをビルドするためのフレームワーク。
- **[Apollo](https://github.com/apollographql)** - GraphQL 用の柔軟なオープンソースツールをビルドしているコミュニティ。
- **[PyTorch](https://github.com/pytorch/pytorch)** - データ操作および機械学習のプラットフォーム。
- **[Calypso](https://github.com/Automattic/wp-calypso)** - WordPress.com を活用するための次世代 Web アプリケーション。
- **[fastlane](https://github.com/fastlane/fastlane)** - Android および iOS 用の自動ビルドツール。
- **[Yarn](https://github.com/yarnpkg/yarn)** - [npm に代わるツール](https://circleci.com/blog/why-are-developers-moving-to-yarn/)。

## 関連項目

{:.no_toc}

「[パブリックリポジトリの例]({{ site.baseurl }}/ja/2.0/example-configs/)」では、パブリックおよびオープンソースのプロジェクトの設定に関する各種のリンクが、CircleCI の機能とプログラミング言語ごとに紹介されています。