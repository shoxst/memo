# Gitの設定

## 前提
### 改行コード
|  OS  |  名称  | 正規表現 | 16進表記 |
| ----         | ----    | ---  | --- |
|  Windows     |  CRLF  | \r\n | 0x0d0a
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
作業ディレクトリとリポジトリの間でCRLFとLFの変換を行うかどうか。よくcommit時に変換と説明されるが、変換が行われるのはblob objectがつくられるタイミングなので、addするとき。
- true
  - 作業ディレクトリ -> リポジトリ で CRLF -> LF、リポジトリ -> 作業ディレクトリ で LF -> CRLFの変換を行う
- input
  - 作業ディレクトリ -> リポジトリ で CRLF -> LFのみ変換を行う
- false(デフォルト)
  - どちらもなし

### `core.safecrlf`
CRLF -> LFの不可逆的な変換を拒否・警告できるオプション。例えば、`coreautocrlf=input`のときに変換を行おうとしているときや、改行コードが混在しているファイルに対して変換するとき。後者はバイナリファイルに対する誤った変換を防止できる。
- true
  - 不可逆的な変換に対して、コマンドを拒否する。
- warn
  - 不可逆的な変換に対して、警告を出してコマンドを実行する。
- false
  - 不可逆的な変換に対して、コマンドを実行する。

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
$ git config --global alias.co checkout
$ git config --global alias.st status
$ git config --global alias.ci commit
$ git config --global alias.br branch
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

## 検証
### `core.autocrlf = true`の場合
```
$ git config core.autocrlf
true
$ git config core.safecrlf
true
```
のときに、改行コードCRLFの`a.txt`
```
hello

```
を`git add`すると、ハッシュ値
```
ce013625030ba8dba906f756967f9e9ca394464a
```
のblob objectが作成される。中を見ると
```
b'blob 6\x00hello\n'
```
なので、LFに変換されていることがわかる。

### `core.autocrlf = input`の場合
```
$ git config core.autocrlf
input
$ git config core.safecrlf
true
```
のときに、同じファイルを`git add`すると、
```
$ git add a.txt
fatal: CRLF would be replaced by LF in a.txt
```
となり失敗する。これは、`core.autocrlf = input`はリポジトリから取り出すときの変換を行わないので、`core.safecrlf`の設定によって不可逆的な変換が拒否されているため。
```
$ git config core.safecrlf false
```
にするとaddされ、ハッシュ値
```
ce013625030ba8dba906f756967f9e9ca394464a
```
のblob objectが作成、すなわちLFに変換されている。

### `core.autocrlf = false`の場合
```
$ git config core.autocrlf
false
```
のときは、`git add`すると上の2つとはハッシュ値が変わり
```
ef0493b275aa2080237f676d2ef6559246f56636
```
となる。中を見ると
```
b'blob 7\x00hello\r\n'
```
となり、改行コードCRLFのまま保存されていることがわかる。
