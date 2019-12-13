# Borgbackup Project for Ansible Tower / AWX

2019.12.13 吉山あきら

## はじめに

このリポジトリは Ansible Tower / AWX で Borgbackup によるバックアップを行うためのプロジェクト(リポジトリ)です。
このプロジェクトを用いてジョブテンプレートを作成する事で、Ansible Tower / AWX を一種のバックアップサーバとして運用する事ができます。

なお、roles/borgbackup/ 下のファイル群は https://github.com/SphericalElephant/ansible-role-borgbackup から派生したものです。
この場をお借りして御礼申し上げます。

## 使い方

以下は Ansible Tower / AWX 上で行います。

1. バックアップサーバ、バックアップクライアントを含むインベントリを新規作成します。
   インベントリには２つのグループ(servers, clients)を作成します。
   servers グループにはバックアップサーバホスト(1台で結構です)を登録し、
   clients グループにはバックアップクライアントホスト群を登録します。
   これらのホストには Ansible Tower / AWX からジョブを管理者権限で実行できる必要があります(パスワード無し sudo 可など)。
2. 1 のホスト群へのアクセス用認証情報を登録します。
3. 本リポジトリをプロジェクトとして登録します。
4. 1～3 を使用するバックアップサーバ用テンプレートを新規作成します。
   * 名前： (ホスト名が判る適切な名前)
   * ジョブタイプ：実行
   * インベントリー：1. で作成したインベントリ名
   * プロジェクト：3. で作成したプロジェクト名
   * PLAYBOOK：server.yml
   * 認証情報：2. で作成した認証情報名
   * 制限：servers
   * オプション：権限昇格の有効化
   * 追加変数：YAML にて以下の要領で記載
        ```
        ---
        borgbackup_install_from_repo: True
        borgbackup_install_from_binary: False
        ```
5. 1～3 を使用するテンプレートを【ホスト毎に】新規作成します。
   * 名前： (ホスト名が判る適切な名前)
   * ジョブタイプ：実行
   * インベントリー：1. で作成したインベントリ名
   * プロジェクト：3. で作成したプロジェクト名
   * PLAYBOOK：client.yml
   * 認証情報：2. で作成した認証情報名
   * 制限：（ホスト名）
   * オプション：権限昇格の有効化
   * 追加変数：YAML にて以下の要領で記載
        ```
        ---
        borgbackup_install_from_repo: True
        borgbackup_install_from_binary: False
        borgbackup_client_backup_server: (バックアップサーバのIPアドレス)
        borgbackup_client_jobs:
          - name: system
            directories:
              - /etc
              - /home
              - /var
            excludes:
              - 're:^/var/lib/apt'
              - 're:^/var/[^/]+\/cache/'
        borgbackup_prune_jobs:
          - name: system
            prune_options: "--keep-daily=7 --keep-weekly=4"
        ```
6. 以下の要領で 4～5 を使用するワークフローを作成します。
   1. 最初 4 のテンプレートのみ実行
   2. 上記ジョブ成功時に 5 の各ジョブを実行
7. 6 のワークフローを実行します。

## ライセンス

Apache License version 2.0
