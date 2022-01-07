# BillingAlarm-CloudFormation
`BillingAlarm-CloudFormation`は`AWS CloudFormation`を利用したInfrastructure as CodeでAWS利用料金通知機能を構築する。


# Requirement
作成時点では以下の環境にローカル環境よりSSH接続で実行検証済
- インスタンス: AWS EC2
- Amazon Linux 2 (AWS EC2)
- RockyLinux 8.5 (実機)

**`Windows`環境では`.sh`が実行出来ないので注意**


# Installation
1. AWSへアクセスし、CloudFormation実行用IAMユーザーを作成する。
    <br>
    ユーザーポリシーは下記を推奨する。
    <br>
    但しポリシー要件によってAdmin権限が許可されない場合は構築リソースによって最小権限とする事。

```
# ユーザーポリシー
arn:aws:iam::aws:policy/AdministratorAccess
```

<br>

2. `{作成したユーザー}`を選択し`認証情報`タブへ移動`アクセスキーの作成`を行う。
    <br>
    credentialの記載された`.csv`ファイルをダウンロードする。

<br>

3. `.aws`を作成する。

```Bash
$ aws configure
```
**上記を実行すると`AWS Access Key ID`等の入力を求められるので任意に入力をする。**

**ここで入力したものは`\.aws\config`に記述されるが`[default]`以外は削除して良い。**

```
# \.aws\config
[default]
```

<br>

4. `.csv`内に記載されている`Access key ID`及び`Secret access key`を`\.aws\credentials`内に転記する。

```
# \.aws\credentials
[default]
aws_access_key_id = {Access key ID}
aws_secret_access_key = {Secret access key}
region = {任意のリージョン}

# 任意に構築先を追加作成可能
# 構築先AWSアカウント毎に設定を行う
[{プロジェクト名}-{環境名}]
aws_access_key_id = {Access key ID}
aws_secret_access_key = {Secret access key}
region = ap-northeast-1
```

<br>

5. `Bash`で各`.sh`に権限を付与する。

```Bash
$ chmod +x ${ファイル名}
```

<br>


# Usage
1. `\BillingAlarm\{リソース名}\{環境名}-parameters.json`内を設定する。

### 例(cloudwatch):

```json
{
    "Parameters": [
        {
            "ParameterKey": "SystemName",
            "ParameterValue": "BillingAlarm"
        },
        {
            "ParameterKey": "EnvType",
            "ParameterValue": "dev"
        },
        {
            "ParameterKey": "Amount",
            "ParameterValue": "10"
        }
    ]
}
```

<br>

2. `create_stacks.sh` (構築用)の設定を行う。

### 例:

```Bash
# 構築対象リソースのコメントアウトを外す
# また構築対象リソースは複数選択可能である。
# 依存関係に注意する事 (例:cloudwatchを構築する為にはsnsを先に構築しなければならない)

# create_stack sns
create_stack cloudwatch
```

<br>

`create_change_sets.sh` (更新用)
<br>
`delete_stacks.sh` (削除用)
<br>
#### 上記に`.sh`についても`create_stacks.sh`と同様に設定を行う。

#### **`create_stacks.sh`同様、依存関係に注意して操作する事。**

<br>

3. `Bash`で下記コマンドを実行する。

```Bash
# 構築
$ ./create_stacks.sh ${環境名}

# 更新
$ ./create_change_sets.sh ${環境名}

# 削除
$ ./delete_stacks.sh ${環境名}
```

<br>

4. `AWS CloudFormation`へアクセスしリソースが構築されているか確認を行う。

<br>


# Note
- 現在対応している環境名は `dev`(開発用), `stg`(検証用), `prod`(本番用) のみである。
- `create_change_sets.sh` (更新用) についてはコマンド実行後、変更セットの実行が必要である。
    <br>
    参考URL:
    <br>
    https://docs.aws.amazon.com/ja_jp/AWSCloudFormation/latest/UserGuide/using-cfn-updating-stacks-changesets-execute.html

<br>


# Author
- 作成者 : NaoyaOgura