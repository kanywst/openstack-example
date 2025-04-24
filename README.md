# OpenStack

## 目次
- [OpenStackとは](#openstackとは)
- [基本構成](#基本構成)
- [主要コンポーネントの詳細](#主要コンポーネントの詳細)
- [実世界での利用例](#実世界での利用例)
- [Macローカルでの試し方](#macローカルでの試し方)
- [Nova、Cloud-init、Hypervisorの関係](#novacloud-inithypervisorの関係)

## OpenStackとは

OpenStackは、データセンターにおいてコンピューティング、ストレージ、ネットワークリソースを制御するためのオープンソースのクラウドコンピューティングプラットフォームです。Infrastructure as a Service (IaaS) を提供し、ユーザーは仮想マシン、ストレージボリューム、ネットワークなどのリソースをプログラマティックに作成・管理できます。

2010年にRackspaceとNASAによって共同で開発が始まり、現在は世界中の多くの企業や組織によって開発・維持されています。OpenStackは6ヶ月ごとにリリースサイクルを持ち、各リリースはアルファベット順の地名で命名されています（例：Antelope, Bobcat, Caracal）。

## 基本構成

OpenStackは、相互に連携する複数のサービス（コンポーネント）から構成されています。各コンポーネントは特定の機能を担当し、RESTful APIを通じて他のコンポーネントと通信します。

```
+------------------------------------------------------------------+
|                        OpenStack Dashboard (Horizon)              |
+------------------------------------------------------------------+
     |           |           |           |           |           |
+----------+ +----------+ +----------+ +----------+ +----------+ +----------+
|  Nova    | |  Swift   | | Neutron  | | Cinder   | |  Glance  | | Keystone |
| (Compute)| | (Object  | | (Network)| | (Block   | | (Image)  | | (Identity)|
|          | | Storage) | |          | | Storage) | |          | |          |
+----------+ +----------+ +----------+ +----------+ +----------+ +----------+
     |           |           |           |           |           |
+------------------------------------------------------------------+
|                      Supporting Services                          |
| (Heat, Ceilometer, Trove, Manila, Ironic, Magnum, etc.)          |
+------------------------------------------------------------------+
```

### コアコンポーネント

1. **Nova (Compute)**: 仮想マシンのプロビジョニングと管理
2. **Swift (Object Storage)**: スケーラブルな分散オブジェクトストレージ
3. **Cinder (Block Storage)**: ブロックストレージデバイスの管理
4. **Neutron (Networking)**: ネットワークとIPアドレスの管理
5. **Glance (Image)**: 仮想マシンイメージの検索と取得
6. **Keystone (Identity)**: 認証と認可サービス
7. **Horizon (Dashboard)**: ウェブベースの管理インターフェース

### 追加コンポーネント

8. **Heat (Orchestration)**: インフラストラクチャのテンプレートベースのデプロイメント
9. **Ceilometer (Telemetry)**: リソース使用状況の監視と課金
10. **Ironic (Bare Metal)**: 物理サーバーのプロビジョニング
11. **Trove (Database)**: データベースサービス
12. **Manila (Shared File System)**: 共有ファイルシステムサービス
13. **Designate (DNS)**: DNSサービス
14. **Magnum (Container Infrastructure)**: コンテナオーケストレーションエンジンの管理

## 主要コンポーネントの詳細

### Nova (Compute)

Novaは、OpenStackのコンピューティングサービスで、仮想マシンのライフサイクル管理を担当します。KVM、Xen、VMware、Hyper-Vなど様々なハイパーバイザーをサポートしています。

主な機能:
- インスタンス（仮想マシン）の作成・削除・再起動
- インスタンスのサイズ変更
- コンソールアクセスの提供
- スナップショットの作成

### Neutron (Networking)

Neutronは、「ネットワークをサービスとして」提供し、OpenStack環境内のネットワーク接続を管理します。

主な機能:
- 仮想ネットワークの作成
- サブネットの管理
- ルーターの設定
- セキュリティグループの管理
- ロードバランサーの提供

### Cinder (Block Storage)

Cinderは、永続的なブロックストレージデバイスを仮想マシンに提供します。

主な機能:
- ボリュームの作成・削除・アタッチ
- ボリュームのスナップショット
- ボリュームのバックアップと復元

### Swift (Object Storage)

Swiftは、大規模で耐障害性の高いオブジェクトストレージシステムです。ファイル、画像、バックアップなどの非構造化データを保存するのに適しています。

主な機能:
- 大規模なデータの保存と取得
- データの冗長性と高可用性
- コンテナとオブジェクトの管理
- アクセス制御と認証

### Glance (Image)

Glanceは、仮想マシンイメージの検索、登録、取得サービスを提供します。

主な機能:
- 仮想マシンイメージのカタログ管理
- イメージのメタデータ管理
- イメージの検索と取得
- イメージフォーマットの変換

### Keystone (Identity)

Keystoneは、OpenStackの認証と認可サービスを提供します。

主な機能:
- ユーザー認証と認可
- サービスカタログの提供
- ポリシー管理
- トークンベースの認証

### Horizon (Dashboard)

Horizonは、OpenStackの各サービスを管理するためのウェブベースのダッシュボードを提供します。

主な機能:
- インスタンス、ボリューム、ネットワークの管理
- イメージとスナップショットの管理
- ユーザーとプロジェクトの管理
- リソース使用状況の監視

## OpenStack以外の仮想化技術

OpenStackはクラウドインフラストラクチャを構築するための包括的なプラットフォームですが、仮想マシン（VM）を実現するための技術は他にも多く存在します。以下に主要な仮想化技術とクラウドプラットフォームを紹介します：

### ハイパーバイザー技術

1. **VMware vSphere/ESXi**
   - 企業向けの成熟した仮想化プラットフォーム
   - 高度な管理機能と安定性を提供
   - vCenter Serverによる一元管理

2. **Microsoft Hyper-V**
   - Windowsサーバーに統合された仮想化技術
   - Windows環境との高い親和性
   - System Center Virtual Machine Managerによる管理

3. **KVM (Kernel-based Virtual Machine)**
   - Linuxカーネルに組み込まれたオープンソースの仮想化技術
   - OpenStackのデフォルトハイパーバイザーとしても使用
   - libvirtによる管理

4. **Xen/Citrix Hypervisor**
   - オープンソースのType-1ハイパーバイザー
   - AWSの初期インフラストラクチャとして使用
   - パラ仮想化と完全仮想化をサポート

5. **Oracle VM VirtualBox**
   - デスクトップ向けのクロスプラットフォーム仮想化ソフトウェア
   - 開発・テスト環境に適している

### コンテナ技術

1. **Docker**
   - コンテナ化技術の代表格
   - 軽量で高速な仮想化環境
   - アプリケーションとその依存関係をパッケージ化

2. **Kubernetes**
   - コンテナオーケストレーションプラットフォーム
   - コンテナのデプロイ、スケーリング、管理を自動化
   - クラウドネイティブアプリケーション向け

3. **LXC/LXD**
   - Linuxコンテナ技術
   - システムコンテナとして機能
   - VMに近い使用感でコンテナを提供

### その他のクラウドプラットフォーム

1. **CloudStack**
   - Apache財団によるオープンソースのクラウド管理プラットフォーム
   - OpenStackの代替として使用される
   - 使いやすいUIと豊富な機能

2. **oVirt**
   - Red Hatが支援するオープンソースの仮想化管理プラットフォーム
   - KVMベースの仮想化環境を管理
   - Red Hat Virtualizationの上流プロジェクト

3. **Proxmox VE**
   - オープンソースのサーバー仮想化管理プラットフォーム
   - KVMとLXCを統合
   - ウェブベースの管理インターフェース

### 仮想化技術とOpenStackの比較

```
+------------------+------------------------+------------------------+
| 特性             | OpenStack              | 他の仮想化技術         |
+------------------+------------------------+------------------------+
| スケール         | 大規模環境向け         | 小〜大規模まで様々     |
| 複雑さ           | 比較的複雑             | 製品により異なる       |
| 管理オーバーヘッド| 高い（多機能のため）   | 製品により異なる       |
| カスタマイズ性   | 非常に高い             | 製品により制限あり     |
| 導入コスト       | 低（オープンソース）   | 製品により異なる       |
| 運用コスト       | 高い（専門知識が必要） | 製品により異なる       |
| APIの豊富さ      | 非常に豊富             | 製品により異なる       |
| マルチテナント   | ネイティブサポート     | 追加機能として提供     |
+------------------+------------------------+------------------------+
```

## 実世界での利用例

OpenStackは多くの企業や組織で採用されており、以下のような有名なサービスや組織で利用されています：

### 企業での利用

1. **CERN (欧州原子核研究機構)**
   - 世界最大のOpenStackデプロイメントの一つを運用
   - 高エネルギー物理学の研究データ処理に利用

2. **Walmart**
   - eコマースプラットフォームとリテールシステムの基盤として利用
   - プライベートクラウドインフラストラクチャの構築

3. **中国移動 (China Mobile)**
   - 通信インフラストラクチャの管理
   - 大規模なプライベートクラウド環境の構築

4. **Comcast**
   - ケーブルテレビとインターネットサービスのバックエンド
   - コンテンツ配信ネットワークの一部

### パブリッククラウドサービス

1. **AWS (Amazon Web Services)**
   - EC2サービスの一部のリージョンでOpenStackコンポーネントを活用
   - ハイブリッドクラウド連携のためのOpenStack互換APIを提供

2. **IBM Cloud**
   - OpenStackベースのプライベートクラウドソリューションを提供
   - IBMのパブリッククラウドサービスの一部にOpenStackを採用

3. **Rackspace Public Cloud**
   - OpenStackの共同創設者であるRackspaceが提供するパブリッククラウド
   - OpenStackのリファレンス実装として知られる

4. **OVH Public Cloud**
   - ヨーロッパ最大のホスティングプロバイダーによるOpenStackベースのクラウド
   - 大規模なOpenStackデプロイメントを運用

5. **VEXXHOST Cloud**
   - OpenStackベースのパブリッククラウドサービス
   - 最新のOpenStackリリースを迅速に採用

6. **Catalyst Cloud**
   - ニュージーランドのOpenStackベースのクラウドプロバイダー
   - 地域特化型のクラウドサービス

### 学術・研究機関

1. **NeCTAR (オーストラリア研究クラウド)**
   - オーストラリアの研究者向けクラウドインフラストラクチャ

2. **Jetstream (米国)**
   - 米国科学財団が支援する研究用クラウド

## Macローカルでの試し方

MacでOpenStackを試すには、いくつかの方法があります。以下に主な方法を紹介します：

### 1. DevStack

DevStackは、開発環境でOpenStackを素早くセットアップするためのスクリプトのセットです。

```bash
# 必要なパッケージのインストール
brew install git python

# DevStackのクローン
git clone https://opendev.org/openstack/devstack
cd devstack

# local.confの作成
cat > local.conf << EOF
[[local|localrc]]
ADMIN_PASSWORD=secret
DATABASE_PASSWORD=\$ADMIN_PASSWORD
RABBIT_PASSWORD=\$ADMIN_PASSWORD
SERVICE_PASSWORD=\$ADMIN_PASSWORD
HOST_IP=127.0.0.1
EOF

# DevStackのインストール
./stack.sh
```

### 2. Microstack

Microstackは、単一のマシンでOpenStackをすぐに実行できるようにするSnapパッケージです。

```bash
# Snapのインストール（Homebrewを使用）
brew install snapd
sudo snap install microstack --beta

# Microstackの初期化
sudo microstack init --auto

# ステータスの確認
microstack.openstack service list
```

### 3. OpenStack-Ansible All-In-One

OpenStack-AnsibleのAll-In-Oneデプロイメントは、単一のマシンで完全なOpenStack環境を提供します。

```bash
# 必要なパッケージのインストール
brew install ansible git

# OpenStack-Ansibleのクローン
git clone https://opendev.org/openstack/openstack-ansible
cd openstack-ansible

# All-In-Oneのデプロイ
./scripts/bootstrap-aio.sh
./scripts/run-playbooks.sh
```

### 4. Docker + Kolla-Ansible

Kolla-AnsibleはDockerコンテナを使用してOpenStackをデプロイします。

```bash
# 必要なパッケージのインストール
brew install python docker ansible

# Pythonの仮想環境を作成
python -m venv kolla-venv
source kolla-venv/bin/activate

# Kolla-Ansibleのインストール
pip install kolla-ansible

# 設定ディレクトリの作成
mkdir -p /etc/kolla
cp -r kolla-venv/share/kolla-ansible/etc_examples/kolla/* /etc/kolla

# All-In-Oneのデプロイ
kolla-ansible -i all-in-one bootstrap-servers
kolla-ansible -i all-in-one deploy
```

## Nova、Cloud-init、Hypervisor、Metadata APIの関係

OpenStackの仮想マシン管理における主要コンポーネントの関係を理解することは重要です。以下にNova、Cloud-init、Hypervisor、Metadata APIなどの関係を図示します。

### コンポーネント間の関係

```
+--------------------------------------------------------------+
|                     OpenStack Controller                      |
|                                                              |
|  +----------+    +----------+    +----------+    +--------+  |
|  | Keystone |<-->|   Nova   |<-->|  Glance  |<-->| Neutron|  |
|  +----------+    +----------+    +----------+    +--------+  |
|                       |                                      |
+------------------------|--------------------------------------+
                         |
                         v
+--------------------------------------------------------------+
|                      Compute Node                            |
|                                                              |
|  +----------+    +---------------+    +----------------+     |
|  |   Nova   |<-->|  Hypervisor   |<-->| Virtual Machine|     |
|  | Compute  |    | (KVM/Xen/etc) |    |                |     |
|  +----------+    +---------------+    +----------------+     |
|                                              |               |
|                                              v               |
|                                       +-------------+        |
|                                       |  Cloud-Init |        |
|                                       +-------------+        |
|                                              |               |
|                                              v               |
|                                     +----------------+       |
|                                     | cloud-init.service     |
|                                     | cloud-config.service   |
|                                     | cloud-final.service    |
|                                     +----------------+       |
+--------------------------------------------------------------+
```

### 詳細な説明

#### 1. Nova (Compute Service)

Novaは、OpenStackのコンピューティングサービスで、仮想マシンのライフサイクル全体を管理します。

```
+--------------------------------------------------+
|                     Nova                         |
|                                                  |
| +-------------+  +-------------+  +-------------+|
| | nova-api    |  | nova-      |  | nova-       ||
| | (APIサービス) |  | scheduler  |  | conductor   ||
| +-------------+  +-------------+  +-------------+|
|                        |                         |
|                        v                         |
| +-------------+  +-------------+  +-------------+|
| | nova-       |  | nova-       |  | その他の     ||
| | compute     |  | network     |  | サービス     ||
| +-------------+  +-------------+  +-------------+|
+--------------------------------------------------+
```

- **nova-api**: RESTful APIを提供し、外部からのリクエストを処理
- **nova-scheduler**: 仮想マシンをどのコンピュートノードで実行するかを決定
- **nova-conductor**: データベース操作とコンピュートノード間の通信を仲介
- **nova-compute**: ハイパーバイザーと通信して仮想マシンを管理

#### 2. Hypervisor

ハイパーバイザーは、物理ハードウェア上で複数の仮想マシンを実行するためのソフトウェアレイヤーです。Novaは様々なハイパーバイザーをサポートしています：

- **KVM**: Linux向けの完全仮想化ソリューション
- **Xen**: オープンソースの仮想化プラットフォーム
- **VMware vSphere**: 企業向け仮想化プラットフォーム
- **Hyper-V**: Microsoftの仮想化技術
- **QEMU**: ハードウェアエミュレーター

Nova-computeは、libvirtなどのAPIを通じてハイパーバイザーと通信します。

```
+----------------+
| Nova Compute   |
+----------------+
        |
        v
+----------------+
| Libvirt API    |
+----------------+
        |
        v
+----------------+
| Hypervisor     |
| (KVM/Xen/etc)  |
+----------------+
        |
        v
+----------------+
| Virtual Machine|
+----------------+
```

#### 3. Cloud-Init

Cloud-Initは、クラウド環境での仮想マシン初期化のための業界標準ツールです。インスタンス起動時に以下のような初期設定を行います：

- ホスト名の設定
- SSHキーの配置
- ユーザーアカウントの作成
- パッケージのインストール
- カスタムスクリプトの実行

```
+------------------------------------------+
|             Cloud-Init Process           |
|                                          |
| +----------------+                       |
| | Metadata       |                       |
| | Service        |<----+                 |
| +----------------+     |                 |
|        |               |                 |
|        v               |                 |
| +----------------+     |                 |
| | cloud-init     |     |                 |
| | (初期化プロセス) |-----+                 |
| +----------------+                       |
|        |                                 |
|        v                                 |
| +----------------+  +----------------+   |
| | cloud-init.    |  | cloud-config.  |   |
| | service        |->| service        |   |
| +----------------+  +----------------+   |
|                            |             |
|                            v             |
|                     +----------------+   |
|                     | cloud-final.   |   |
|                     | service        |   |
|                     +----------------+   |
+------------------------------------------+
```

#### 4. Cloud-Init サービスの流れ

Cloud-Initは、システム起動時に以下のサービスを順番に実行します：

1. **cloud-init.service**:
   - ネットワーク設定
   - シードファイルの処理
   - データソースの検出と初期化

2. **cloud-config.service**:
   - ユーザーとグループの設定
   - SSHキーの配置
   - マウントポイントの設定
   - パッケージリポジトリの設定

3. **cloud-final.service**:
   - パッケージのインストール
   - ブートスクリプトの実行
   - 最終設定の適用

#### 5. Metadata API

Metadata APIは、仮想マシンインスタンスが自身の設定情報を取得するための重要なサービスです。Cloud-Initなどの初期化ツールは、このAPIを通じてインスタンス固有の設定を取得します。

```
+--------------------------------------------------+
|                 Metadata API                     |
|                                                  |
| +----------------+  +----------------+           |
| | Instance       |  | User           |           |
| | Metadata       |  | Data           |           |
| +----------------+  +----------------+           |
|          |                  |                    |
|          v                  v                    |
| +----------------+  +----------------+           |
| | SSH Keys       |  | Network        |           |
| |                |  | Configuration  |           |
| +----------------+  +----------------+           |
|                                                  |
| +----------------+  +----------------+           |
| | Cloud-Init     |  | Vendor         |           |
| | Directives     |  | Data           |           |
| +----------------+  +----------------+           |
+--------------------------------------------------+
```

主な機能:
- インスタンスのホスト名、IPアドレス、ネットワーク設定の提供
- SSHキーの配布
- ユーザーデータ（カスタムスクリプトなど）の提供
- インスタンスのメタデータ（インスタンスID、フレーバー情報など）の提供

アクセス方法:
- インスタンス内からは通常 `http://169.254.169.254` でアクセス可能
- 例: `curl http://169.254.169.254/openstack/latest/meta_data.json`

#### 6. Nova、Hypervisor、Cloud-Init、Metadata APIの連携

1. ユーザーがNovaにインスタンス作成をリクエスト
2. Nova-schedulerが適切なコンピュートノードを選択
3. Nova-computeがハイパーバイザーに指示して仮想マシンを作成
4. 仮想マシン起動時にCloud-Initが実行され、初期設定を適用
5. Cloud-InitはMetadata APIにアクセスして設定情報を取得
6. Metadata APIはNovaから情報を取得し、インスタンスに提供
7. 設定完了後、インスタンスが利用可能になる

```
+-------------+     +-------------+     +-------------+
| User        |---->| Nova API    |---->| Nova        |
| Request     |     |             |     | Scheduler   |
+-------------+     +-------------+     +-------------+
                                              |
                                              v
+-------------+     +-------------+     +-------------+
| Cloud-Init  |<----| Virtual     |<----| Nova        |
| Process     |     | Machine     |     | Compute     |
+-------------+     +-------------+     +-------------+
      |                    ^                   |
      |                    |                   v
      |             +-------------+     +-------------+
      +------------>| Metadata    |<----| Hypervisor  |
                    | API         |     | (KVM/etc)   |
                    +-------------+     +-------------+
                          ^
                          |
                    +-------------+
                    | Config      |
                    | Drive       |
                    +-------------+
```

この連携により、OpenStackは自動化されたクラウドインフラストラクチャを提供し、仮想マシンのプロビジョニングから初期設定までをシームレスに行うことができます。

## その他の重要なコンポーネントと関係性

### Placement

Placementは、リソース管理と追跡のためのサービスで、Nova-schedulerが適切なコンピュートノードを選択する際に重要な役割を果たします。

```
+------------------+     +------------------+
| Nova Scheduler   |<--->| Placement API    |
+------------------+     +------------------+
                               |
                               v
                         +------------------+
                         | Resource         |
                         | Providers        |
                         +------------------+
```

主な機能:
- リソースプロバイダー（コンピュートノードなど）の管理
- リソースクラス（CPU、メモリ、ディスクなど）の追跡
- リソース使用状況の監視
- リソース割り当ての最適化

### Barbican (Key Management)

Barbicanは、OpenStackの鍵管理サービスで、暗号化キー、証明書、パスワードなどの機密情報を安全に保存・管理します。

主な機能:
- 対称鍵の保存と管理
- 非対称鍵の保存と管理
- X.509証明書の管理
- シークレット（パスワードなど）の安全な保存

### Oslo

Osloは、OpenStackの共通ライブラリ群で、各コンポーネント間で共有される機能を提供します。

主なライブラリ:
- oslo.config: 設定管理
- oslo.messaging: メッセージングサービス
- oslo.db: データベース抽象化
- oslo.log: ロギング
- oslo.policy: ポリシー管理

### RabbitMQ/Message Queue

OpenStackコンポーネント間の通信は、RabbitMQなどのメッセージキューを通じて行われます。

```
+-------------+     +-------------+     +-------------+
| Nova API    |---->| Message     |---->| Nova        |
|             |     | Queue       |     | Compute     |
+-------------+     +-------------+     +-------------+
                          ^
                          |
                    +-------------+
                    | Other       |
                    | Services    |
                    +-------------+
```

主な役割:
- 非同期通信の実現
- サービス間の疎結合
- 負荷分散
- 耐障害性の向上

### Ceph (Storage Backend)

Cephは、OpenStackのストレージバックエンドとして広く使用されています。

```
+-------------+     +-------------+     +-------------+
| Cinder      |---->| Ceph        |<----| Glance      |
| (Block)     |     | Storage     |     | (Image)     |
+-------------+     +-------------+     +-------------+
                          ^
                          |
                    +-------------+
                    | Swift       |
                    | (Object)    |
                    +-------------+
```

主な特徴:
- スケーラブルなストレージ
- ブロック、オブジェクト、ファイルストレージの統合
- 高可用性と耐障害性
- 自己修復機能

### Octavia (Load Balancing)

Octaviaは、OpenStackの高可用性ロードバランシングサービスです。

主な機能:
- L4/L7ロードバランシング
- TLS終端
- ヘルスモニタリング
- 自動スケーリング

### Senlin (Clustering)

Senlinは、OpenStackのクラスタリングサービスで、リソースのグループ化と管理を行います。

主な機能:
- ノードのグループ化
- 自動スケーリング
- ポリシーベースの管理
- ヘルスモニタリング

## まとめ

OpenStackは、クラウドインフラストラクチャを構築・管理するための包括的なオープンソースプラットフォームです。様々なコンポーネントが連携して動作し、仮想マシン、ストレージ、ネットワークなどのリソースをプログラマティックに管理できます。

多くの企業や組織がOpenStackを採用しており、プライベートクラウド、パブリッククラウド、ハイブリッドクラウドなど様々な形態で利用されています。AWS、IBM Cloud、Rackspaceなどの主要パブリッククラウドプロバイダーも、一部のサービスでOpenStackを活用しています。

OpenStack以外にも、VMware vSphere、Microsoft Hyper-V、KVM、Xen、Docker、Kubernetesなど、様々な仮想化技術やクラウドプラットフォームが存在します。これらの技術は、用途や規模に応じて選択されています。

Macローカル環境でもDevStack、Microstack、Kolla-Ansibleなどのツールを使用して、OpenStackを試すことができます。

Nova、Hypervisor、Cloud-Init、Metadata APIなどのコンポーネントが連携することで、仮想マシンのライフサイクル管理と初期設定の自動化を実現しています。また、Placement、Barbican、Oslo、RabbitMQ、Cephなどの追加コンポーネントが、OpenStackの機能を拡張し、より堅牢で柔軟なクラウド環境を提供しています。
