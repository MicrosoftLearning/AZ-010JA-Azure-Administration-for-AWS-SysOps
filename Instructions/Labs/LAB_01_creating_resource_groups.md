---
lab:
    title: 'リソース グループの作成'
    module: 'モジュール 1: Azure 管理'
---

# ラボ 01: リソース グループの作成

Azure Cloud Shell CLI の概要

## 受講者ラボ マニュアル

## シナリオ

このラボでは、Azure Portal のコースで使用するサブスクリプションの設定と、Azure CLI コマンドを使用する Azure Cloud Shell の設定について説明します。  講師は、特定のコース環境に基づいて Azure サブスクリプションを構成するために必要な手順を提供します。あなたは、2 つのリソース グループを構成して、Azure Cloud Shell で最初の CLI コマンドを実行します。

## 目的

このラボを修了すると、次のことが可能になります:

* Azure Portal を使用してサブスクリプションを設定します (コース インストラクターが必要に応じて手順を提供します)
* Azure CLI コマンドを使用してリソース グループを作成する

## ラボのセットアップ

* **予想時間**: 20 分

## 指示

### 開始する前に

#### セットアップ タスク

1. Azure portal を使用してこのコースで使用する Azure アカウントを構成する手順については、コース インストラクターの指示に従ってください。

## エクササイズ 1: Cloud Shell を使用して Azure CLI を開始し、2 つのリソース グループを作成する

このエクササイズの主なタスクは次のとおりです。

1. Azure Cloud Shell を起動する
1. ラボで使用する既定のサブスクリプションを設定する
1. CLI コマンドを使用して WestRG リソース グループを作成する
1. CLI コマンドを使用して EastRG リソース グループを作成する

### エクササイズ 1 - タスク 1: Cloud Shell を開く

**Azure Cloud Shell でサブスクリプションを設定する**

1. ポータルの上部にある [**Cloud Shell**] アイコンをクリックして、[クラウド シェル] ペインを開きます。

1. Cloud Shell インターフェイスで、**Bash** を選択します。

1. [**Cloud Shell**] コマンド プロンプトで、次のコマンドを入力し、**Enter** キーを押して、ポータル サインインに使用するアカウントに関連付けられているすべてのサブスクリプションを一覧表示します。

```sh
az account list --output table
```

1. サブスクリプションの一覧を確認し、ラベルが "default `true`" とラベル付けされている場合
1. 目的のサブスクリプションに既定値が設定されていない場合は、既定のサブスクリプションをリセットします。
1. **Cloud Shell** コマンド プロンプトで、**お好きなサブスクリプション ID** で次のコマンドを入力し、**Enter** キーを押して既定のサブスクリプションを設定します。

```sh
# subscription ID またはサブスクリプション名とサブスクリプション値を置き換え
az account set --subscription [1111a1a1-22bb-3c33-d44d-e5e555ee5eee]
az account list --output table
```

4. サブスクリプションの一覧を確認し、適切なサブスクリプションが "default `true`" とラベル付けされていることを確認します。

### タスク 2: CLI を使用して WestRG リソース グループを作成する

1. [**Cloud Shell**] コマンド プロンプトで、次のコマンドを入力して、westus リージョンに WestRG リソースグループを作成します。

```sh
az group create --location westus --name WestRG --output table
```

2. **Cloud Shell** コマンド プロンプトで、次のコマンドを入力して、westus リージョンで利用可能なリソース グループを一覧表示します。

```sh
az group list --output table
```

3. 新規作成された WestRG が一覧表示されていることを確認します。

### タスク 3: CLI を使用して EastRG リソース グループを作成する

1. [**Cloud Shell**] コマンド プロンプトで、次のコマンドを入力して、eastus リージョンに EastRG リソースグループを作成します。

```sh
az group create --location eastus --name EastRG --output table
```

2. **Cloud Shell** コマンド プロンプトで、次のコマンドを入力して、eastus リージョンで利用可能なリソース グループを一覧表示します。

```sh
az group list --output table
```

3. 新しく作成された EastRG が一覧表示されていることを確認します。

> **結果**: このラボでは、既定の Azure サブスクリプションを構成し、次のラボでさらに使用する 2 つのリソース グループを作成しました。
