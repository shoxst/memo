# Gitの設定

## 前提
### 改行コード
|  OS  |  名称  | 正規表現 | 16進表記 |
| ----         | ----    | ---  | --- |
|  Windows     |  CR+LF  | \r\n | 0x0d0a
|  Mac / Linux  |  LF     | \n   | 0x0a

## config
### 設定ファイルの場所
#### Mac
- local
  - `.git/config`
- global
  - `~/.gitconfig`
- system
  - `/usr/local/etc/gitconfig`
  - `/usr/local/git/etc/gitconfig`

#### Windows
- local
  - `.git\config`
- global
  - `C:\Users\{ユーザー名}\.gitconfig`
- sysytem
  - `C:¥Program Files¥Git¥etc¥gitconfig`
  - `C:¥Program Files¥Git¥mingw64¥etc¥gitconfig`
  - `C:¥Program Files (x86)¥Git¥etc¥gitconfig`
  - `C:¥Program Files (x86)¥Git¥mingw64¥etc¥gitconfig`

### `core.eol` 
作業ディレクトリ上のファイルに使用する改行コード。lf, crlf, nativeの3通りでデフォルトはnative。通常は変更する必要なし。

### `core.autocrlf`
作業ディレクトリとリポジトリの間でCRLFとLFの変換を行うかどうか。よくcommit時に変換と説明されているが、blob objectつくるタイミングならaddするタイミング?
- true
  - 作業ディレクトリ -> リポジトリ で CRLF -> LF、リポジトリ -> 作業ディレクトリ で LF -> CRLFの変換を行う
- input
  - 作業ディレクトリ -> リポジトリ で CRLF -> LFのみ変換を行う
- false(デフォルト)
  - どちらもなし

### `core.safecrlf`
改行コードが混在している場合に変換しないオプション。

### `core.whitespace`
`cr-at-eol`を設定しておくと、改行コードCRLFのファイルを`git diff`したときに`^M`の表示を抑止することができる。

CRをgitは余分な空白として扱ってしまうので、改行コードの一部であると定義している。

### `core.quotepath`
falseにすると、`git status`などのときに日本語のファイル名が文字化けしない。

## .gitattributes
`.git`と同じ階層に`.gitattributes`という名前でファイルを作成する。configはユーザーごとの設定だが、.gitattributesはリポジトリごとの設定なので、リポジトリごとに改行コードなどの扱いを統一できる。

configとの設定の組み合わせによる挙動の違いは、http://ogawa.s18.xrea.com/tdiary/20200828p01.html を参照。

```
*           text=auto
*.txt       text
*.vcproj    text eol=crlf
*.sh        text eol=lf
*.jpg       -text
```
のように記載して、後ろの行に書いた設定によって上書きされる。

```
*           text=auto
```
gitがテキストファイルだと判定したら、text属性をtrueにする。
```
*.txt       text
```
`*.txt`にマッチするファイルは、text属性をtrueにする。
```
*.vcproj    text eol=crlf
*.sh        text eol=lf
```
マッチするファイルをtext属性trueにしつつ、改行コードを指定している。この場合、シェルスクリプトのファイルはユーザーがconfigで`core.autocrlf=true`にしていても、作業ディレクトリのファイルはLFでチェックアウトされる。

## 設定まとめ
リポジトリ内の改行コードはすべてLF、チェックアウト時はWindowsでCRLF、MacでLFという方針のもと、以下のように設定。

### Windows, Mac共通
```
$ git config --global user.name <name>
$ git config --global user.email <email>
$ git config --global core.quotepath false
```
Emailは https://github.com/settings/emails の Keep my email addresses private にチェックを入れて、noreplyを使う。

### Windows
```
$ git config --global core.autocrlf true
$ git config --global core.safecrlf true
$ git config --global core.whitespace cr-at-eol
```

### Mac
```
$ git config --global core.autocrlf input
$ git config --global core.safecrlf true
```
`~/.config/git/ignore`にglobalな.gitignoreを設定
```
.DS_Store
```

### WindowsにおいてもLFでチェックアウトしたい場合
.gitattributesで以下を設定。
```
*.sh    text eol=lf
```

### Shift_JISを扱う場合
vscodeでそのフォルダ全体をshiftjisにするには、`Command + ,` で設定 -> ワークスペース からShift JISを選択。

`git diff`で文字化けしないように、.gitattributesに以下を記述。
```
*       diff=sjis
```
さらに
```
$ git config --local diff.sjis.textconv "iconv -f sjis"
```
これは、Windows, Macどちらでも設定が必要。この設定はglobalでもよい。
