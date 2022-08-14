# VSCodeで Java WebApp を開発する
Tomcat + Maven の WebアプリケーションをVScodeで開発できるようになるための環境構築

## Javaインストール
- インストール先
  - https://docs.aws.amazon.com/ja_jp/corretto/latest/corretto-11-ug/downloads-list.html
  - jdk11以上でないと、VSCode側が対応していない
- 環境変数の設定
  - JAVA_HOME
    - C:\Program Files\Amazon Corretto\jdk11.0.16_8
  - PATH
    - C:\Program Files\Amazon Corretto\jdk11.0.16_8\bin

## Mavenインストール
- インストール先
  - https://maven.apache.org/download.cgi
- 環境変数の設定
  - PATH
    - C:\Program Files\apache-maven-3.8.6\bin
- VSCodeの設定
  - Maven › Executable: Path
    - C:\Program Files\apache-maven-3.8.6\bin\mvn

## Tomcatインストール
- インストール先
  - https://tomcat.apache.org/download-10.cgi

## VSCode の拡張機能
- Extention Pack for Java
- Community Server Connector
  - エクスプローラー > Community Server Connector > Create New Server