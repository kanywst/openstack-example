# Nova

## 目次
- [概要](#概要)
- [アーキテクチャ](#アーキテクチャ)
- [主要コンポーネント](#主要コンポーネント)
- [インスタンスのライフサイクル](#インスタンスのライフサイクル)
- [スケジューリング](#スケジューリング)
- [ハイパーバイザーサポート](#ハイパーバイザーサポート)
- [ネットワーキング](#ネットワーキング)
- [ストレージ統合](#ストレージ統合)
- [APIとエンドポイント](#apiとエンドポイント)
- [設定と最適化](#設定と最適化)
- [高可用性と耐障害性](#高可用性と耐障害性)
- [セキュリティ](#セキュリティ)
- [監視とトラブルシューティング](#監視とトラブルシューティング)

## 概要

Novaは、OpenStackのコンピュートサービスであり、クラウド環境における仮想マシン（インスタンス）のライフサイクル管理を担当します。ユーザーはNovaを通じて、仮想マシンの作成、起動、停止、再起動、削除などの操作を行うことができます。

Novaは、KVM、Xen、VMware、Hyper-Vなど様々なハイパーバイザーをサポートし、異種混合環境での一貫した管理インターフェースを提供します。また、ベアメタルサーバーのプロビジョニングもサポートしています。

## アーキテクチャ

Novaは、分散アーキテクチャを採用しており、複数のコンポーネントが協調して動作します。これにより、高いスケーラビリティと耐障害性を実現しています。

```
+------------------------------------------------------------------+
|                        Nova Architecture                         |
|                                                                  |
|  +----------+    +----------+    +----------+    +----------+    |
|  | nova-api |<-->| nova-    |<-->| nova-    |<-->| nova-    |    |
|  |          |    | scheduler|    | conductor|    | compute  |    |
|  +----------+    +----------+    +----------+    +----------+    |
|       ^               ^               ^               ^          |
|       |               |               |               |          |
|       v               v               v               v          |
|  +----------------------------------------------------------+   |
|  |                      Message Queue                        |   |
|  |                      (RabbitMQ)                           |   |
|  +----------------------------------------------------------+   |
|       ^               ^               ^               ^          |
|       |               |               |               |          |
|       v               v               v               v          |
|  +----------------------------------------------------------+   |
|  |                      Database                             |   |
|  |                      (MySQL/MariaDB)                      |   |
|  +----------------------------------------------------------+   |
|                                                                  |
+------------------------------------------------------------------+
```

### 分散コンポーネントモデル

Novaは、RESTful APIを通じてリクエストを受け付け、内部的にはRabbitMQなどのメッセージキューを使用してコンポーネント間で通信します。状態情報はMySQLなどのリレーショナルデータベースに保存されます。

## 主要コンポーネント

### 1. nova-api

RESTful APIを提供し、外部からのリクエストを処理します。主に以下の機能を担当します：

- APIリクエストの受付と検証
- リクエストの適切なコンポーネントへの転送
- レスポンスの生成と返送

nova-apiは、EC2互換APIとOpenStack独自のAPIの両方をサポートしています。

### 2. nova-scheduler

どのコンピュートノードで仮想マシンを実行するかを決定します。以下の要素を考慮してスケジューリングを行います：

- リソース要件（CPU、メモリ、ディスク）
- アフィニティ/アンチアフィニティルール
- フィルタリングとウェイト付け

nova-schedulerは、Placementサービスと連携して、利用可能なリソースを追跡します。

### 3. nova-conductor

データベース操作とコンピュートノード間の通信を仲介します。主な役割は：

- コンピュートノードからのデータベースアクセスの抽象化
- 複雑な操作のオーケストレーション
- コンピュートノードとの通信の管理

nova-conductorにより、コンピュートノードが直接データベースにアクセスする必要がなくなり、セキュリティが向上します。

### 4. nova-compute

ハイパーバイザーと通信して仮想マシンを管理します。主な機能は：

- 仮想マシンの作成、起動、停止、再起動、削除
- ハイパーバイザーAPIの抽象化
- 仮想マシンの状態監視
- リソース使用状況の報告

nova-computeは、libvirtなどのドライバーを通じて様々なハイパーバイザーと通信します。

### 5. その他のコンポーネント

- **nova-consoleauth**: コンソールプロキシのトークン認証を担当
- **nova-novncproxy**: NoVNCベースのコンソールアクセスを提供
- **nova-spicehtml5proxy**: SPICE HTML5ベースのコンソールアクセスを提供
- **nova-serialproxy**: シリアルコンソールアクセスを提供

## インスタンスのライフサイクル

Novaでは、インスタンス（仮想マシン）は以下のライフサイクルを持ちます：

```
+----------------+
| BUILD          |
| (作成中)        |
+----------------+
        |
        v
+----------------+     +----------------+
| ACTIVE         |---->| STOPPED        |
| (実行中)        |<----| (停止中)        |
+----------------+     +----------------+
    |        ^             |        ^
    v        |             v        |
+----------------+     +----------------+
| PAUSED         |     | SUSPENDED      |
| (一時停止中)    |     | (サスペンド中)  |
+----------------+     +----------------+
        |
        v
+----------------+     +----------------+
| RESCUED        |---->| RESIZED        |
| (レスキュー中)  |<----| (リサイズ中)    |
+----------------+     +----------------+
        |
        v
+----------------+
| ERROR          |
| (エラー)        |
+----------------+
        |
        v
+----------------+
| DELETED        |
| (削除済み)      |
+----------------+
```

### インスタンス作成プロセス

1. **リクエスト受付**: nova-apiがインスタンス作成リクエストを受け取る
2. **スケジューリング**: nova-schedulerが適切なコンピュートノードを選択
3. **ネットワーク準備**: Neutronがネットワークリソースを準備
4. **ボリューム準備**: Cinderが必要なボリュームを準備
5. **イメージ取得**: Glanceから仮想マシンイメージを取得
6. **インスタンス作成**: nova-computeがハイパーバイザーに指示して仮想マシンを作成
7. **初期化**: Cloud-Initによる初期設定の適用
8. **完了通知**: インスタンスの状態をACTIVEに更新

## スケジューリング

### フィルタースケジューラー

Novaのデフォルトスケジューラーは、フィルタースケジューラーです。これは、以下の2段階のプロセスでコンピュートノードを選択します：

1. **フィルタリング**: 要件を満たさないノードを除外
2. **ウェイト付け**: 残ったノードにスコアを付け、最適なノードを選択

### 主要なフィルター

- **ComputeFilter**: コンピュートノードが有効かどうかを確認
- **AvailabilityZoneFilter**: 指定されたアベイラビリティゾーンに基づいてフィルタリング
- **RamFilter**: 十分なRAMがあるかを確認
- **DiskFilter**: 十分なディスク容量があるかを確認
- **CoreFilter**: 十分なCPUコアがあるかを確認
- **ImagePropertiesFilter**: イメージのプロパティに基づいてフィルタリング
- **ServerGroupAntiAffinityFilter**: アンチアフィニティルールに基づいてフィルタリング

### Placementサービスとの連携

最新のNovaは、Placementサービスと連携してリソース管理を行います：

```
+----------------+     +----------------+
| Nova Scheduler |<--->| Placement API  |
+----------------+     +----------------+
                              |
                              v
                       +----------------+
                       | Resource       |
                       | Providers      |
                       +----------------+
```

Placementサービスは、以下の情報を管理します：

- リソースプロバイダー（コンピュートノードなど）
- リソースクラス（CPU、メモリ、ディスクなど）
- リソーストレイト（HW特性、機能など）
- 割り当て（インスタンスに割り当てられたリソース）

## ハイパーバイザーサポート

Novaは、様々なハイパーバイザーをサポートしており、ドライバーを通じて抽象化されたインターフェースを提供します。

### サポートされる主要なハイパーバイザー

1. **KVM (Kernel-based Virtual Machine)**
   - Linuxカーネルに組み込まれた仮想化技術
   - 最も広く使用されるOpenStackのハイパーバイザー
   - libvirtを通じて管理

2. **Xen/XenServer**
   - オープンソースのType-1ハイパーバイザー
   - パラ仮想化と完全仮想化をサポート
   - libvirtまたは直接APIを通じて管理

3. **VMware vSphere**
   - 企業向けの商用ハイパーバイザー
   - VMware APIを通じて管理
   - 既存のVMware環境との統合に適している

4. **Hyper-V**
   - Microsoftの仮想化技術
   - Windows環境との統合に適している
   - Hyper-V APIを通じて管理

5. **Ironic (Bare Metal)**
   - 物理サーバーを直接管理
   - ハイパーバイザーなしで動作
   - 高性能コンピューティングに適している

### ハイパーバイザードライバー

Novaは、各ハイパーバイザーとの通信に以下のドライバーを使用します：

```
+------------------------------------------------------------------+
|                        Nova Compute                              |
|                                                                  |
|  +----------+  +----------+  +----------+  +----------+          |
|  | libvirt  |  | vmware   |  | hyper-v  |  | ironic   |          |
|  | driver   |  | driver   |  | driver   |  | driver   |          |
|  +----------+  +----------+  +----------+  +----------+          |
|       |             |             |             |                |
|       v             v             v             v                |
|  +----------+  +----------+  +----------+  +----------+          |
|  | KVM/QEMU |  | vSphere  |  | Hyper-V  |  | Bare     |          |
|  | Xen      |  |          |  |          |  | Metal    |          |
|  +----------+  +----------+  +----------+  +----------+          |
|                                                                  |
+------------------------------------------------------------------+
```

## ネットワーキング

Novaは、Neutronと連携してインスタンスのネットワーキングを管理します。

### Neutronとの統合

```
+----------------+     +----------------+
| Nova Compute   |<--->| Neutron Server |
+----------------+     +----------------+
        |                      |
        v                      v
+----------------+     +----------------+
| Instance       |<--->| Virtual        |
|                |     | Network        |
+----------------+     +----------------+
```

主な連携ポイント：

1. **インスタンス作成時**:
   - ポートの作成
   - IPアドレスの割り当て
   - セキュリティグループの適用

2. **インスタンス削除時**:
   - ネットワークリソースの解放

3. **インスタンス移行時**:
   - ネットワーク接続の再構成

### ネットワークモデル

Novaは、以下のネットワークモデルをサポートしています：

1. **Provider Networks**: プロバイダーが定義した物理ネットワークに直接接続
2. **Self-Service Networks**: テナントが定義した仮想ネットワーク
3. **Floating IPs**: インスタンスにパブリックIPを動的に割り当て

## ストレージ統合

Novaは、様々なストレージオプションをサポートしています：

### エフェメラルストレージ

インスタンスのライフサイクルに紐づいた一時的なストレージです。インスタンスが削除されると、データも失われます。

### Cinderとの統合（ブロックストレージ）

```
+----------------+     +----------------+
| Nova Compute   |<--->| Cinder Volume  |
+----------------+     +----------------+
        |                      |
        v                      v
+----------------+     +----------------+
| Instance       |<--->| Persistent     |
|                |     | Volume         |
+----------------+     +----------------+
```

永続的なブロックストレージを提供し、以下の操作をサポートします：

- ボリュームのアタッチ/デタッチ
- ブートボリュームからのインスタンス起動
- ボリュームスナップショットの作成

### Glanceとの統合（イメージサービス）

```
+----------------+     +----------------+
| Nova Compute   |<--->| Glance         |
+----------------+     +----------------+
        |                      |
        v                      v
+----------------+     +----------------+
| Instance       |     | Image          |
|                |     | Repository     |
+----------------+     +----------------+
```

仮想マシンイメージの管理を担当し、以下の機能を提供します：

- イメージからのインスタンス起動
- インスタンスからのスナップショット作成
- イメージのキャッシング

## APIとエンドポイント

### Nova API

Nova APIは、RESTful APIを提供し、以下のリソースを管理します：

- Servers (インスタンス)
- Flavors (インスタンスタイプ)
- Images (Glanceと連携)
- Keypairs (SSHキーペア)
- Quotas (リソース制限)
- Server Groups (アフィニティグループ)

### 主要なAPIエンドポイント

```
# インスタンスの一覧取得
GET /v2.1/{tenant_id}/servers

# インスタンスの詳細取得
GET /v2.1/{tenant_id}/servers/{server_id}

# インスタンスの作成
POST /v2.1/{tenant_id}/servers

# インスタンスの削除
DELETE /v2.1/{tenant_id}/servers/{server_id}

# インスタンスのアクション実行（起動、停止、再起動など）
POST /v2.1/{tenant_id}/servers/{server_id}/action
```

### マイクロバージョニング

Nova APIは、マイクロバージョニングをサポートしており、APIの互換性を維持しながら機能を追加できます：

```
# APIバージョンの一覧取得
GET /

# 特定のマイクロバージョンを指定したリクエスト
GET /v2.1/servers
X-OpenStack-Nova-API-Version: 2.79
```

## 設定と最適化

### 主要な設定ファイル

Novaの設定は、主に以下のファイルで管理されます：

- **nova.conf**: メインの設定ファイル
- **policy.json**: アクセス制御ポリシーの定義

### 重要な設定パラメータ

```ini
[DEFAULT]
# 基本設定
debug = False
verbose = True
log_dir = /var/log/nova
state_path = /var/lib/nova

# メッセージキュー設定
transport_url = rabbit://openstack:password@controller:5672

# 認証設定
auth_strategy = keystone

# ネットワーク設定
use_neutron = True
firewall_driver = nova.virt.firewall.NoopFirewallDriver

[api]
# API設定
auth_strategy = keystone

[keystone_authtoken]
# Keystone認証設定
www_authenticate_uri = http://controller:5000/
auth_url = http://controller:5000/
memcached_servers = controller:11211
auth_type = password
project_domain_name = Default
user_domain_name = Default
project_name = service
username = nova
password = password

[vnc]
# VNCコンソール設定
enabled = true
server_listen = $my_ip
server_proxyclient_address = $my_ip

[glance]
# Glance連携設定
api_servers = http://controller:9292

[oslo_concurrency]
# 同時実行設定
lock_path = /var/lib/nova/tmp

[placement]
# Placement API設定
region_name = RegionOne
project_domain_name = Default
project_name = service
auth_type = password
user_domain_name = Default
auth_url = http://controller:5000/v3
username = placement
password = password

[libvirt]
# libvirtドライバー設定
virt_type = kvm
cpu_mode = host-model
```

### パフォーマンス最適化

大規模環境でのNovaのパフォーマンスを最適化するためのベストプラクティス：

1. **コンピュートノードのサイジング**
   - オーバーコミットメント比率の適切な設定
   - CPU、メモリ、ディスクのバランス

2. **データベース最適化**
   - インデックスの最適化
   - 定期的なアーカイブと削除

3. **キャッシング**
   - Memcachedの活用
   - イメージキャッシングの設定

4. **ネットワーク最適化**
   - 専用ネットワークの使用
   - ジャンボフレームの有効化

## 高可用性と耐障害性

### コントローラーノードの高可用性

Novaのコントローラーコンポーネント（nova-api、nova-scheduler、nova-conductor）は、複数のノードで実行することで高可用性を実現できます：

```
+----------------+     +----------------+     +----------------+
| Controller 1   |     | Controller 2   |     | Controller 3   |
|                |     |                |     |                |
| - nova-api     |     | - nova-api     |     | - nova-api     |
| - nova-        |     | - nova-        |     | - nova-        |
|   scheduler    |     |   scheduler    |     |   scheduler    |
| - nova-        |     | - nova-        |     | - nova-        |
|   conductor    |     |   conductor    |     |   conductor    |
+----------------+     +----------------+     +----------------+
         |                     |                     |
         v                     v                     v
+----------------------------------------------------------+
|                      Load Balancer                       |
+----------------------------------------------------------+
         |                     |                     |
         v                     v                     v
+----------------------------------------------------------+
|                      Database Cluster                    |
+----------------------------------------------------------+
         |                     |                     |
         v                     v                     v
+----------------------------------------------------------+
|                      Message Queue Cluster               |
+----------------------------------------------------------+
```

### コンピュートノードの耐障害性

コンピュートノード（nova-compute）の障害に対処するための機能：

1. **インスタンス退避（Evacuation）**
   - 障害が発生したコンピュートノードからインスタンスを別のノードに移動

2. **自動復旧（Auto Recovery）**
   - 障害検出時に自動的にインスタンスを再起動

3. **ライブマイグレーション**
   - 実行中のインスタンスをダウンタイムなしで別のノードに移動

## セキュリティ

### 認証と認可

Novaは、Keystoneと連携して認証と認可を管理します：

1. **認証（Authentication）**
   - ユーザーIDとパスワード、トークンによる認証
   - 多要素認証のサポート

2. **認可（Authorization）**
   - Role-Based Access Control (RBAC)
   - policy.jsonによる詳細なアクセス制御

### インスタンスセキュリティ

1. **セキュリティグループ**
   - インスタンスへのネットワークアクセス制御
   - ステートフルファイアウォールルール

2. **SSH鍵管理**
   - インスタンスへのセキュアなアクセス
   - 公開鍵認証の強制

3. **インスタンス分離**
   - ハイパーバイザーレベルでの分離
   - テナント間のネットワーク分離

### データ保護

1. **暗号化**
   - ボリューム暗号化（Cinderと連携）
   - イメージ暗号化（Glanceと連携）

2. **セキュアな削除**
   - インスタンス削除時のデータ消去オプション

## 監視とトラブルシューティング

### ログ

Novaの各コンポーネントは、詳細なログを生成します：

- **/var/log/nova/nova-api.log**
- **/var/log/nova/nova-scheduler.log**
- **/var/log/nova/nova-conductor.log**
- **/var/log/nova/nova-compute.log**

### 監視指標

Novaの健全性を監視するための重要な指標：

1. **インスタンス状態**
   - 実行中、エラー、ビルド中などの状態分布

2. **リソース使用率**
   - CPU、メモリ、ディスク使用率
   - オーバーコミットメント率

3. **API応答時間**
   - リクエスト処理時間
   - エラーレート

4. **スケジューリング成功率**
   - スケジューリング試行回数
   - スケジューリング失敗の理由

### 一般的な問題と解決策

1. **インスタンス起動失敗**
   - 症状: インスタンスがERROR状態になる
   - 解決策: nova-computeのログを確認、リソース不足やイメージの問題を特定

2. **スケジューリング失敗**
   - 症状: NoValidHost例外
   - 解決策: フィルター条件の確認、リソース不足の解消

3. **ネットワーク接続問題**
   - 症状: インスタンスにネットワーク接続できない
   - 解決策: セキュリティグループ、Neutron設定の確認

4. **パフォーマンス低下**
   - 症状: インスタンスの応答が遅い
   - 解決策: オーバーコミットメント率の調整、リソース競合の解消

### 診断コマンド

Novaの状態を診断するための有用なコマンド：

```bash
# インスタンスの一覧と状態を表示
openstack server list

# インスタンスの詳細情報を表示
openstack server show <server_id>

# コンピュートサービスの状態を確認
openstack compute service list

# ハイパーバイザーの統計情報を表示
openstack hypervisor stats show

# インスタンスの診断情報を取得
nova diagnostics <server_id>

# コンピュートノードのリソース使用状況を表示
openstack resource provider show <compute_node>
```

## まとめ

Novaは、OpenStackのコアコンポーネントとして、クラウド環境における仮想マシンのライフサイクル管理を担当します。分散アーキテクチャと様々なハイパーバイザーのサポートにより、柔軟でスケーラブルなコンピューティングリソースの管理を実現しています。

Novaは、他のOpenStackサービス（Keystone、Glance、Neutron、Cinder、Placement）と密接に連携して動作し、包括的なクラウドインフラストラクチャを提供します。適切に設計・設定・管理されたNova環境は、信頼性、スケーラビリティ、セキュリティを備えたクラウドコンピューティング基盤となります。
