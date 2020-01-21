# ラボ 05 解答キー

## 指示

1. 以下の CLI スクリプトをメモ帳などのエディタにコピーする
   1. `# ----実行する前にこれらの値を編集する----` と名付けられたロケーション セクション
   1. 環境を表す値を編集すると、**エラー**が発生します
   1. ファイルをローカルに保存する
1. Azure ポータルにログインし、bash Cloud Shell を開きます。
1. 依存関係が整っているかどうかを確認します (スクリプトのコメントの上部を参照)。
1. ローカル ファイルから CLI スクリプトをコピーし、Bash Cloud Shell にスクリプトを貼り付けます。
1. 一部のタスクは 1 分以上かかる場合があります。スクリプトが完了するのを待ち、出力を確認します。
1. エラーが発生した場合は講師と話しましょう

> 受講生は、ラボ 01 で説明されているように、既定のサブスクリプションを設定する必要があります。
>
> ユーザー ドメインの変数を作成します。
> アカウントのメール アドレスが eric@contoso.com の場合、コマンドは `my_domain=ericcontoso.onmicrosoft.com` になります。
>
> * *必要に応じてインストラクターに相談してください。*

```sh
# AZ-010 LAB5 Solution
# ---------------
# DEPENDENCY: LAB1-solution in place (WestRG created)
# ---------------
# 以下の値を指定します。
#   subscriptionID
#   my_domain
# Azure CLI Bashで以下のコマンドを実行する前に

# ----------開始----------

# +++++++++++++++++++++++++++++++++++++++++++++++++++++
# ---------実行する前にこれらの値を編集する-------------
subscriptionID=[**subscription ID to use for labs**]
# ユーザー ドメインの変数を作成する
my_domain=[**UsernameEmaildomain.onmicrosoft.com**]
# 一意の AD ユーザー パスワードを作成する
password_ad_user=[**sTR0ngP@ssWorD543%**]
# +++++++++++++++++++++++++++++++++++++++++++++++++++++

# ----主なスクリプト----
# ----デフォルトのサブスクリプションを設定する----
az account set --subscription $subscriptionID

#----ユーザー アカウント名 (ユーザー プリンシパル名) の作成----
my_user_account=AZ010@$my_domain

#----ユーザー アカウントの作成----
az ad user create \
    --display-name AZ010Tester \
    --password $password_ad_user \
    --user-principal-name $my_user_account

#----AD ユーザーの一覧表示----
az ad user list --output json | jq '.[] | {"userPrincipalName":.userPrincipalName, "objectId":.objectId}'
# ====================================================
#----上記の出力に一覧表示されている新しい AD ユーザーに注意してください----
# ====================================================

#----新しいユーザーにロール ('"所有者"') を追加する----
az role assignment create --role "Owner" --assignee $my_user_account --resource-group WestRG

#----作成したユーザーのロールの割り当てを一覧表示する (変更に注意)----
az role assignment list --assignee $my_user_account -g WestRG

#----ソフトウェアのインストールを制限するポリシーを作成する----
#----ポリシー定義を作成します。
az policy definition create --name 'require-sqlserver-version12' \
    --display-name 'Require SQL Server version 12.0' \
    --description 'This policy ensures all SQL servers use version 12.0.'\
    --rules 'https://raw.githubusercontent.com/Azure/azure-policy/master/samples/built-in-policy/require-sqlserver-version12/azurepolicy.rules.json' \
    --params 'https://raw.githubusercontent.com/Azure/azure-policy/master/samples/built-in-policy/require-sqlserver-version12/azurepolicy.parameters.json' \
    --mode All

#----サブスクリプションレベルの範囲----
az policy assignment create --name SQL12AZ010 \
    --display-name 'Require SQL Server version 12.0 - subscription scope' \
    --scope '/subscriptions/'$subscriptionID \
    --policy 'require-sqlserver-version12'

#----新しく作成したポリシーを表示する----
az policy assignment show --name 'SQL12AZ010'

# ++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
#----エクササイズ 2: Query Explorer を使用してログとアラートを監視する-----
# ******************************************************************
# ---------------------------手動ステップ---------------------------
#----ログ分析クエリのデモンストレーションに移動する----
#---------------https://portal.loganalytics.io/demo ----------------
#-----LAB05 のエクササイズ 2 の指示に従って Query Explorerを使用する-----

```

> **エクササイズ 2 - タスク 1 – ログとアラートの監視を確認する**
> デモンストレーション環境へのアクセス**
>
> 1. 新しいブラウザー タブで、[Log Analytics Querying Demonstration](https://portal.loganalytics.io/demo) に移動します。
> 2. クエリ エクスプローラの使用
>     1. 「クエリ エクスプローラ] を選択します (右上)。
>     2. 「お気に入り] を展開し、*エラーのあるすべての Syslog レコード*を選択します。
>     3. クエリが編集ペインに追加されることに注意してください。クエリの構造に注意してください。
>     4. クエリを実行します。返されたレコードを調べます。
>     5. インストラクターの追加手順に従います。
>     6. 時間があれば、他のお気に入りや保存済みクエリを試しましょう。