# Git 複数アカウント運用
git アカウントをプライベート/仕事、GitHub/GitLab で使い分けるための設定。

## ssh key
`~/.ssh`に以下のように鍵を作成しておく。
```
- id_rsa_github_private
- id_rsa_github_private.pub
- id_rsa_gitlab_private
- id_rsa_gitlab_private.pub
```
`~/.ssh/config`に以下のように設定
```
Host gitlab.com.private
    HostName gitlab.com
    User git
    Port 22
    IdentityFile ~/.ssh/id_rsa_gitlab_private
    TCPKeepAlive yes
    IdentitiesOnly yes

Host github.com
    HostName github.com
    User git
    Port 22
    IdentityFile ~/.ssh/id_rsa_github_private   
    TCPKeepAlive yes
    IdentitiesOnly yes
```
checkout するときは、
```
git clone git@gitlab.com.private:test-group-sh1/test.git
git clone git@github.com:shoxst/memo.git
```
のようにすると、remote originが
```
remote.origin.url=git@gitlab.com.private:test-group-sh1/test.git
```
のようになる。

## ユーザ名、メールアドレス
`~/.gitconfig`に以下のように設定。
```
[includeIf "gitdir:~/src/gitlab_private/"]
    path = ~/src/gitlab_private/.gitconfig

[includeIf "gitdir:~/src/github_private/"]
    path = ~/src/github_private/.gitconfig
```
それぞれの`gitconfig`で`user.name`と`user.email`を指定
```
[user]
    name = xxx
    email = xxx@users.noleply.gitlab.com
```
