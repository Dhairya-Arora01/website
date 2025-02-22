---
title: API概要
content_type: concept
weight: 20
no_list: true
card:
  name: reference
  weight: 50
  title: API概要
---

<!-- overview -->

このセクションでは、Kubernetes APIのリファレンス情報を提供します。

REST APIはKubernetesの基本的な構造です。
すべての操作とコンポーネント間のと通信、および外部ユーザーのコマンドは、REST API呼び出しでありAPIサーバーが処理します。

その結果、Kubernetesプラットフォーム内のすべてのものは、APIオブジェクトとして扱われ、[API](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/)に対応するエントリーがあります。

[Kubernetes APIリファレンス](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/)は、Kubernetesバージョン{{< param "version" >}}のAPI一覧を提供します。

一般的な背景情報を知るには、[The Kubernetes API](/docs/concepts/overview/kubernetes-api/)、
[Controlling Access to the Kubernetes API](/docs/concepts/security/controlling-access/)を読んでください。
それらはKubernetes APIサーバーがクライアントを認証する方法とリクエストを認可する方法を説明します。

## APIバージョニング

JSONとProtobufなどのシリアル化スキーマの変更については同じガイドラインに従います。
以下の説明は、両方のフォーマットをカバーしています。

APIのバージョニングとソフトウェアのバージョニングは間接的に関係しています。
[API and release versioning proposal](https://git.k8s.io/sig-release/release-engineering/versioning.md)は、APIバージョニングとソフトウェアバージョニングの関係を説明しています。

APIのバージョンが異なると、安定性やサポートのレベルも異なります。
各レベルの基準については、[API Changes documentation](https://git.k8s.io/community/contributors/devel/sig-architecture/api_changes.md#alpha-beta-and-stable-versions)で詳しく説明しています。

各レベルの概要は以下の通りです:

- Alpha:
  - バージョン名に「alpha」が含まれています（例：「v1alpha1」）。
  - バグが含まれている可能性があります。
    機能を有効にするとバグが露呈する可能性があります。
    機能がデフォルトで無効になっている可能性があります。
  - ある機能のサポートは、予告なしにいつでも中止される可能性があります。
  - 後にリリースされるソフトウェアで、互換性のない方法で予告なく変更される可能性があります。
  - バグのリスクが高く、長期的なサポートが得られないため、短期間のテストクラスターのみでの使用を推奨します。

- Beta:
  - バージョン名には `beta` が含まれています(例：`v2beta3`)。
  - ソフトウェアは十分にテストされています。
    機能を有効にすることは安全であると考えられています。
    機能はデフォルトで有効になっています。
  - 機能のサポートが打ち切られることはありませんが、詳細は変更される可能性があります。

  - オブジェクトのスキーマやセマンティクスは、その後のベータ版や安定版のリリースで互換性のない方法で変更される可能性があります。
    このような場合には、移行手順が提供されます。
    スキーマの変更に伴い、APIオブジェクトの削除、編集、再作成が必要になる場合があります。
    編集作業は単純ではないかもしれません。
    移行に伴い、その機能に依存しているアプリケーションのダウンタイムが必要になる場合があります。

  - 本番環境での使用は推奨しません。
    後続のリリース は、互換性のない変更を導入する可能性があります。
    独立してアップグレード可能な複数のクラスターがある場合、この制限を緩和できる可能性があります。

  {{< note >}}
ベータ版の機能をお試しいただき、ご意見をお寄せください。
ベータ版の機能が終了した後はこれ以上の変更ができない場合があります。
  {{< /note >}}

- Stable:
  - バージョン名は `vX` であり、`X` は整数である。
  - 安定版の機能は、リリースされたソフトウェアの中で、その後の多くのバージョンに登場します。

## APIグループ

[API groups](https://git.k8s.io/community/contributors/design-proposals/api-machinery/api-group.md)で、KubernetesのAPIを簡単に拡張することができます。
APIグループは、RESTパスとシリアル化されたオブジェクトの`apiVersion`フィールドで指定されます。

KubernetesにはいくつかのAPIグループがあります:

* *core*(*legacy*とも呼ばれる)グループは、RESTパス `/api/v1` にあります。
   コアグループは `apiVersion` フィールドの一部としては指定されません。
   例えば、`apiVersion: v1` のように。
* 名前付きのグループは、RESTパス `/apis/$GROUP_NAME/$VERSION` にあり、以下のように使用します。
   `apiVersion: $GROUP_NAME/$VERSION`を使用します(例：`apiVersion: batch/v1`)。
   サポートされているAPIグループの完全なリストは以下にあります。
   [Kubernetes API reference](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#strong-api-groups-strong-)。

## APIグループの有効化と無効化   {#enabling-or-disabling}

一部のリソースやAPIグループはデフォルトで有効になっています。
APIサーバー上で`--runtime-config`を設定することで、有効にしたり無効にしたりすることができます。
また`runtime-config`フラグには、APIサーバーのランタイム構成を記述したコンマ区切りの`<key>[=<value>]`ペアを指定します。
もし`=<value>`の部分が省略された場合には、`=true`が指定されたものとして扱われます。

例えば:

 - `batch/v1`を無効するには、`--runtime-config=batch/v1=false`を設定する
 - `batch/v2alpha1`を有効するには、`--runtime-config=batch/v2alpha1`を設定する

{{< note >}}
グループやリソースを有効または無効にした場合、
APIサーバーとコントローラーマネージャーを再起動して、`--runtime-config`の変更を反映させる必要があります。
{{< /note >}}

## 永続化

Kubernetesはシリアライズされた状態を、APIリソースとして{{< glossary_tooltip term_id="etcd" >}}に書き込んで保存します。

## {{% heading "whatsnext" %}}

- [API conventions](https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#api-conventions)をもっと知る
- [aggregator](https://github.com/kubernetes/community/blob/master/contributors/design-proposals/api-machinery/aggregated-api-servers.md)の設計ドキュメントを読む
