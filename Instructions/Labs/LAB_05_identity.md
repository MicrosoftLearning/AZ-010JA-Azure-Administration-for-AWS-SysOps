---
lab:
    title: 'ユーザー、グループ、ポリシーの作成。ログとアラートの監視。'
    module: 'モジュール 5: Azure ID'
---

# ラボ 01: Azure ID

ユーザー、グループ、ポリシーの作成とログとアラートの監視

## 受講者ラボ マニュアル

## シナリオ

このラボでは、Bash インターフェイス上で Cloud Shell の CLI コマンドを使用して Azure Identity を管理します。  Azure ロール ベースのアクセス制御、Azure Policy、およびクエリ エクスプローラを使用した監視の確認を行います。

## 目的

このラボを修了すると、次のことが可能になります:

* Azure ロールベースのアクセス制御を使用してユーザーとグループを作成および構成する
* ソフトウェアのインストールを制限するポリシーを作成する
* Query Explorer を使用して Azure Portal の監視ログとアラートを確認する

## ラボのセットアップ

* **予想時間**: 45 分

## 指示

### 開始する前に

#### セットアップ タスク

1. このコースで使用する Azure アカウントを構成します。
2. **モジュール 1: Azure Administration, Lab: リソース グループの作成** WestRG リソース グループを構成します。

### エクササイズ 1: ユーザー、グループ、ポリシーの作成

このエクササイズの主なタスクは次のとおりです。

1. ユーザーとグループを追加します。
1. ソフトウェアのインストールを制限するポリシーを作成します。

### 演習 1 - タスク 1: ユーザーとグループを追加する

**ユーザーを追加する**

1. ユーザー ドメインの変数を作成します。

> eric@contoso.com のメール アドレスを使用すると、コマンドは 'my_domain=ericcontoso.onmicrosoft.com' になります。
>
> * *必要に応じてインストラクターに相談してください。*

2. ユーザー ドメインの変数を編集します。

```sh
# ユーザー ドメインの変数を作成する
# user@contoso.com =>  usercontoso.onmicrosoft.com

my_domain=<email+service>.onmicrosoft.com
```

**ユーザー アカウントを作成する**

1. ユーザー アカウント名の作成

```sh
my_user_account=AZ010@$my_domain
```

2. 一意の強力なパスワードを作成します (パスワードの編集!)。

```sh
# パスワードを一意に編集する (先頭の "!"またはエラーを削除する)
az ad user create \
    --display-name AZ010Tester \
    --password !sTR0ngP@ssWorD543%* \
    --user-principal-name $my_user_account
```

3. 表示名、パスワード、--user-principal-name を書き留める

**ユーザーとグループの管理**

1. AD ユーザーを一覧表示します。

```sh
az ad user list --output json | jq '.[] | {"userPrincipalName":.userPrincipalName, "objectId":.objectId}'
```

2. 前のステップで作成したユーザーが一覧表示されます。
3. すべてのロール割り当てを一覧表示します。

```sh
az role assignment list --all -o table
```

4. このリストの初期状態に注意してください。
5. リソース グループのロール割り当てを一覧表示します。

```sh
az role assignment list --resource-group WestRG --output json | jq '.[] | {"principalName":.principalName, "roleDefinitionName":.roleDefinitionName, "scope":.scope}'
```

6. 新しいユーザーにロール (「所有者]) を追加します。

```sh
az role assignment create --role "Owner" --assignee $my_user_account --resource-group WestRG

#az role assignment create --role "Owner" --assignee <assignee object id> --resource-group <resource_group>
```

**ユーザーとグループの変更を確認する**

1. リソース グループのロール割り当ての一覧表示を繰り返します (変更に注意)。

```sh
az role assignment list --resource-group WestRG --output json | jq '.[] | {"principalName":.principalName, "roleDefinitionName":.roleDefinitionName, "scope":.scope}'
```

2. 作成したユーザーのロールの割り当てを一覧表示します (変更に注意)。

```sh
az role assignment list --assignee $my_user_account -g WestRG #--output json | jq '.[] | {"principalName":.principalName, "roleDefinitionName":.roleDefinitionName, "scope":.scope}'
```

## タスク 2 – ソフトウェアのインストールを制限するポリシーを作成する

**ソフトウェア制限ポリシーをデプロイする**

> *[Github](https://github.com/Azure/azure-policy/tree/master/samples/built-in-policy/require-sqlserver-version12) のスクリプト下で使用される rules.json と parameters.json を確認する*

1. ポリシー定義を作成します。

```sh
az policy definition create --name 'require-sqlserver-version12' \
    --display-name 'Require SQL Server version 12.0' \
    --description 'This policy ensures all SQL servers use version 12.0.' \
    --rules 'https://raw.githubusercontent.com/Azure/azure-policy/master/samples/built-in-policy/require-sqlserver-version12/azurepolicy.rules.json' \
    --params 'https://raw.githubusercontent.com/Azure/azure-policy/master/samples/built-in-policy/require-sqlserver-version12/azurepolicy.parameters.json' \
    --mode All
```

2. サブスクリプションレベルの範囲。

```sh
az policy assignment create --name SQL12AZ010 \
    --display-name 'Require SQL Server version 12.0 - subscription scope' \
    --scope '/subscriptions/'$subscriptionID \
    --policy 'require-sqlserver-version12'
```

3. ポリシーの割り当てを一覧表示します。

```sh
az policy assignment list
```

4. 新しく作成したポリシーを表示します。

```sh
az policy assignment show --name 'SQL12AZ010'
```

**コンプライアンスの確認**

1. Azure Policy サービス ページに戻ります。
2. 「コンプライアンス] を選択します。コンプライアンス状態フィルタを「すべてのコンプライアンス状態 *(All compliance states)]に設定します。
3. ポリシーのステータスと定義を確認します。
## 演習 2: Query Explorer を使用してログとアラートを監視する

1. ログとアラートの監視

### 演習 2 - タスク 1 – ログとアラートの監視を確認する

**デモンストレーション環境へのアクセス**

1. 新しいブラウザー タブで、[Log Analytics Querying Demonstration](https://portal.loganalytics.io/demo) に移動します。
2. クエリ エクスプローラの使用
    1. 「クエリ エクスプローラ] を選択します (右上)。
    2. 「お気に入り] を展開し、*エラーのあるすべての Syslog レコード*を選択します。
    3. クエリが編集ペインに追加されることに注意してください。クエリの構造に注意してください。
    4. クエリを実行します。返されたレコードを調べます。
    5. インストラクターの追加手順に従います。
    6. 時間があれば、他のお気に入りや保存済みクエリを試しましょう。
