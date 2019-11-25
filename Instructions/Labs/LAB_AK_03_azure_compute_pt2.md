# ラボ 03 pt 2 解答キー

## 指示

1. 以下の CLI スクリプトをメモ帳などのエディタにコピーする
1. `# ----EDIT THESE VALUES Before Running----` と名付けられたロケーション セクション
1. 環境を表す値を編集し、ファイルをローカルに保存します。
1. Azure ポータルにログインし、bash Cloud Shell を開きます。
1. ローカル ファイルから CLI スクリプトをコピーし、Bash Cloud Shell にスクリプトを貼り付けます。
1. 一部のタスクは 1 分以上かかる場合があります。スクリプトが完了するのを待ち、出力を確認します。
1. エラーが発生した場合は講師と話しましょう

> 注: 受講生は、ラボ 01 で説明されているように、既定のサブスクリプションを設定する必要があります。

```sh
# AZ-010 LAB3 Solution
# ---------------
# 依存関係: 
#   LAB1 - ソリューションを設定済み (WestRG & EastRG 作成済み)
# ---------------
# 警告: これらのステップは 1 分以上かかります
# エラーが発生した場合は、講師に相談してください

# ----------開始----------

# ----主なスクリプト----

# ----エクササイズ 2: Ubuntu スケール セットの作成とテスト----
# ----Ubuntu スケール セットを作成する----

# ----スケール セット リソース グループを作成する----
az group create --name EastScaleRG --location eastus

# ----スケール セットを作成する----
az vmss create \
  --resource-group EastScaleRG \
  --name EastUbuntuServers \
  --image UbuntuLTS \
  --upgrade-policy-mode automatic \
  --instance-count 2 \
  --admin-username azuser \
  --generate-ssh-keys

# ----自動スケーリング プロファイルの定義----
az monitor autoscale create \
  --resource-group EastScaleRG \
  --resource EastUbuntuServers \
  --resource-type Microsoft.Compute/virtualMachineScaleSets \
  --name autoscale \
  --min-count 1 \
  --max-count 3 \
  --count 1

# ----自動スケーリング アウト ルールを作成する----
az monitor autoscale rule create \
  --resource-group EastScaleRG \
  --autoscale-name autoscale \
  --condition "Percentage CPU > 50 avg 2m" \
  --scale out 1

# ----自動スケーリング イン ルールを作成する----
az monitor autoscale rule create \
  --resource-group EastScaleRG \
  --autoscale-name autoscale \
  --condition "Percentage CPU < 30 avg 1m" \
  --scale in 1

# --------Ubuntu スケール セットをテストする (オプション タスク)--------

# ----次の出力を使用して、スケール セット Ubuntu サーバー セッションの SSH を接続する----
az vmss list-instance-connection-info \
--resource-group EastScaleRG \
--name EastUbuntuServers

# *********************************************************************************
# ----上記の出力で、SSH を使用して Ubuntu スケール セット インスタンス 0 に接続する----
# ----LAB03 EXERCISE2 TASK2 に移動し、手順を完了します。
# ----	* ストレスのインストール/実行
# ----	* ビューのスケーリング
# *********************************************************************************

# ----CLI コマンドの下で実行してスケール セット デモをクリーンアップします----
# az グループの削除 --name EastScaleRG --yes --no-wait
```

> Ubuntu スケール セットをテストする最後のオプション タスクを実行します。
> **SSH を使用して Ubuntu スケール セット インスタンス 0 に接続する上記の最終的な接続情報出力をキャプチャします。**
> 次に、LAB03 EXERCISE2 TASK2 に移動し、手順を完了します。
>
> * ストレスのインストール/実行
> * ビューのスケーリング
> * スケール セットのクリーンアップ
>
