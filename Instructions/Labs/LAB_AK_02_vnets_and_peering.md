# ラボ 02 解答キー

## 指示

1. Azure ポータルにログインし、bash Cloud Shell を開きます。
1. 以下の CLI スクリプトをコピーする
1. Bash Cloud Shell にスクリプトを貼り付けます。
1. 一部のタスクは 1 分以上かかる場合があります。スクリプトが完了するのを待ち、出力を確認します。
1. エラーが発生した場合は講師と話しましょう

> 注: 受講生は、ラボ 01 で説明されているように、既定のサブスクリプションを設定する必要があります。

```sh
# AZ-010 LAB2 Solution
# ---------------
# 依存関係: LAB1 - ソリューションを設定済み (WestRG & EastRG 作成済み)
# ---------------
# 警告: これらのステップは 1 分以上かかります
# エラーが発生した場合は、講師に相談してください

# ----------開始----------

# ----WestVNet と WestSubNet1 を作成する----
az network vnet create \
  --resource-group WestRG \
  --name WestVNet \
  --address-prefix 10.1.0.0/16 \
  --subnet-name WestSubnet1 \
  --subnet-prefix 10.1.0.0/24

# ----WestVNet と WestSubNet1 が作成されたことを確認する----
az network vnet subnet list --resource-group WestRG \
--vnet-name WestVNet --output table

# ----EastVNet と EastSubNet1 を作成する----
az network vnet create \
  --resource-group EastRG \
  --name EastVNet \
  --address-prefix 10.2.0.0/16 \
  --subnet-name EastSubnet1 \
  --subnet-prefix 10.2.0.0/24

# ----EastVNet 上で EastSubNet2 を作成する----
az network vnet subnet create \
  --resource-group EastRG \
  --vnet-name EastVNet \
  --name EastSubnet2 \
  --address-prefix 10.2.1.0/24

# ----EastVNet が作成されたことを確認する----
az network vnet list --output table

# ----EastVNet SubNets が作成されたことを確認する----
az network vnet subnet list --resource-group EastRG --vnet-name EastVNet --output table

# ------West から East へのネットワークピアリングの開始------
# ----変数でEastVNet IDをキャプチャする----
EastVNetId=$(az network vnet show \
  --resource-group EastRG \
  --name EastVNet \
  --query id --out tsv)

  echo "EastVNetId = " $EastVNetId

# ----Peer West to East----
az network vnet peering create \
  --name WesttoEastPeering \
  --resource-group WestRG \
  --vnet-name WestVNet \
  --remote-vnet $EastVNetId \
  --allow-vnet-access

# ----ピアリングの状態を確認する----
az network vnet peering list \
  --resource-group WestRG \
  --vnet-name WestVNet \
  --output table

# ------West から East へのネットワークピアリングの開始------
# ----変数の WestVNet ID を取得----
WestVNetId=$(az network vnet show \
  --resource-group WestRG \
  --name WestVNet \
  --query id --out tsv)
  
echo "WestVNetId = " $WestVNetId

# ----Peer East to West----
az network vnet peering create \
  --name EasttoWestPeering \
  --resource-group EastRG \
  --vnet-name EastVNet \
  --remote-vnet $WestVNetId \
  --allow-vnet-access

# ----ピアリングの状態を確認する----
az network vnet peering list \
  --resource-group EastRG \
  --vnet-name EastVNet \
  --output table
```
