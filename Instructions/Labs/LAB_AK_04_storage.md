# ラボ 04 解答キー

## 指示

1. 以下の CLI スクリプトをメモ帳などのエディタにコピーする
1. `# ----実行する前にこれらの値を編集する----` と名付けられたロケーション セクション
1. 環境を表す値を編集し、ファイルをローカルに保存します。
1. Azure ポータルにログインし、bash Cloud Shell を開きます。
1. 依存関係が整っているかどうかを確認します (スクリプトのコメントの上部を参照)。
1. ローカル ファイルから CLI スクリプトをコピーし、Bash Cloud Shell にスクリプトを貼り付けます。
1. 一部のタスクは 1 分以上かかる場合があります。スクリプトが完了するのを待ち、出力を確認します。
1. エラーが発生した場合は講師と話しましょう

> 注: 受講生は、ラボ 01 で説明されているように、既定のサブスクリプションを設定する必要があります。

```sh
## AZ-010 LAB4 Solution
# ---------------
# 依存関係: 
#   LAB1 - ソリューションを設定済み (WestRG & EastRG 作成済み)
#   LAB2 - ソリューションを設定済み (VNets, SubNets & Peering 作成済み)
#   LAB3 - ソリューションを設定済み (EastDebianVM 作成済み)
# ---------------
# 警告: これらのステップは 1 分以上かかります
# エラーが発生した場合は、講師に相談してください

# ----------開始----------

# ----実行する前にこれらの値を編集する----
subscriptionID=[**subscription ID to use for labs**]

# ----主なスクリプト----
# ----デフォルトのサブスクリプションを設定する----
az account set --subscription $subscriptionID
# ---デフォルトのサブスクリプションに注意する---
az account list --output table

# ----WestRGの作成----
az group create --location westus --name WestRG --output table

# ----EastRGの作成----
az group create --location eastus --name EastRG --output table

# ----リソースグループの一覧表示----
az group list -o table
```
