---
lab:
    title: 'Azure 仮想ネットワークとピアリング'
    module: 'モジュール 2: Azure ネットワーク'
---
    
# ラボ 02: Azure 仮想ネットワークとピアリング

## 受講者ラボ マニュアル

## シナリオ

このラボでは、Bash インターフェイス上で Cloud Shell の CLI コマンドを使用して仮想ネットワーク (VNet) を管理します。リソース グループとリージョンにまたがる VNet を作成し、VNet ピアリングを構成します。

## 目的

この実習ラボを完了すると、Azure CLI を使用して次の操作を行うことができるようになります。

* お望みのリソース グループでサブネットを使用して VNet を作成および構成します。
* VNet 間の VNet ピアリングを構成します。

## ラボのセットアップ

* **予想時間**: 30 分

## 指示

### 開始する前に

#### セットアップ タスク

1. **モジュール 1** から構成された EastRG および WestRG リソース グループ: **Azure Administration、Labリソース グループの作成**:

### エクササイズ 1: サブネットを使用して Virtual Network を作成する

このエクササイズの主なタスクは次のとおりです。

1. サブネットを使用して West VNet を作成する。
1. West ネットワークとサブネットが作成されたことを確認する。
1. サブネットを使用して East VNet を作成する
1. East ネットワークとサブネットが作成されたことを確認する。

#### タスク 1 サブネットを使用して West VNet を作成する

1. [**Cloud Shell**] コマンド プロンプトで、次のコマンドを入力して、WestSubNet1 サブネットを使用して WestVNet 仮想ネットワークを作成します。

```sh
az network vnet create \
  --resource-group WestRG \
  --name WestVNet \
  --address-prefix 10.1.0.0/16 \
  --subnet-name WestSubnet1 \
  --subnet-prefix 10.1.0.0/24
```

2. West ネットワークとサブネットが作成されたことを確認する。

```sh
az network vnet list --output table
```

3. West ネットワークとサブネットが作成されたことを確認する。

```sh
az network vnet subnet list --resource-group WestRG --vnet-name WestVNet --output table
```

#### タスク 2 サブネットを使用して East VNet を作成する

1. [**Cloud Shell**] コマンド プロンプトで、次のコマンドを入力して、EastSubNet1 サブネットを使用して EastVNet 仮想ネットワークを作成します。

```sh
az network vnet create \
  --resource-group EastRG \
  --name EastVNet \
  --address-prefix 10.2.0.0/16 \
  --subnet-name EastSubnet1 \
  --subnet-prefix 10.2.0.0/24
```

2. 既存の VNet にサブネットを追加することが可能です。
3. [**Cloud Shell**] コマンド プロンプトで、次のコマンドを入力して、EastVNet EastSubNet2 サブネットを作成します。

```sh
az network vnet subnet create \
  --resource-group EastRG \
  --vnet-name EastVNet \
  --name EastSubnet2 \
  --address-prefix 10.2.1.0/24
```

4. East ネットワーク (EastVNet) が作成されたことを確認する

```sh
az network vnet list --output table
```

5. East ネットワーク サブネットが作成されたことを確認する。

```sh
az network vnet subnet list --resource-group EastRG --vnet-name EastVNet --output table
```

#### タスク 3: ピアリング ネットワーク West to East を作成する

1. `remote-vnet-id` CLI コマンドを使用して、West VNet と East VNet 間のピアリングを作成する
1. [**Cloud Shell**] コマンド プロンプトで、次のコマンドを入力して、変数に EastVNet ID をキャプチャします。

```sh
EastVNetId=$(az network vnet show \
  --resource-group EastRG \
  --name EastVNet \
  --query id --out tsv)

  echo "EastVNetId = " $EastVNetId
```

3. 次のコマンドを入力して、WestVNet から EastVNet にピアする

```sh
az network vnet peering create \
  --name WesttoEastPeering \
  --resource-group WestRG \
  --vnet-name WestVNet \
  --remote-vnet $EastVNetId \
  --allow-vnet-access
```

> *注: Cloud Shell `--remote-vnet-id $EastVNetId` は使用されず、警告がスローされますが、古い CLI シェルで必要になる場合があります。*

4. 次のコマンドを入力してピアリングの状態を確認する

```sh
az network vnet peering list \
  --resource-group WestRG \
  --vnet-name WestVNet \
  --output table
```

#### タスク 4: ピアリング ネットワーク East to West を作成する

1. 変数で WestVNet ID をキャプチャする

```sh
WestVNetId=$(az network vnet show \
  --resource-group WestRG \
  --name WestVNet \
  --query id --out tsv)
  
echo "WestVNetId = " $WestVNetId
```

2. 次のコマンドを入力して、EastVNet から WestVNet にピアする

```sh
az network vnet peering create \
  --name EasttoWestPeering \
  --resource-group EastRG \
  --vnet-name EastVNet \
  --remote-vnet $WestVNetId \
  --allow-vnet-access
```

3. 次のコマンドを入力してピアリングの状態を確認する

```sh
az network vnet peering list \
  --resource-group EastRG \
  --vnet-name EastVNet \
  --output table
```

> **結果**: この実習ラボでは、東西の仮想ネットワークとサブネットを構成し、ネットワーク間のピアリングを作成しました (East から West、West から East)。これらのリソースは、以降のラボでさらに使用されます。
