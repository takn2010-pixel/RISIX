RISIX - xv6-riscv 用 POSIX 互換レイヤー
========================================

xv6-riscv に POSIX 風の機能を少しずつ追加していくプロジェクトです。
教育用の最小カーネルを保ちながら、POSIX 互換 API を段階的に実装します。


概要
----

xv6-riscv は意図的に最小限に設計されています。システムコールの数は
POSIX よりはるかに少なく、一般的な API の多くが欠けています。
RISIX は、xv6 の設計とコードスタイルを保ちながら、
それらを一つずつ追加していきます。


追加した機能
------------

システムコール
~~~~~~~~~~~~~~

  API      状態    備考
  -------  ------  --------------------------------------------
  lseek    完了    SEEK_SET、SEEK_CUR、SEEK_END
  dup2     完了    POSIX 互換の複製
  waitpid  完了    WNOHANG と POSIX の status エンコーディング
  pledge   完了    サンドボックス機構（OpenBSD 風）

ファイルシステム
~~~~~~~~~~~~~~~~

  機能                 状態    備考
  -------------------  ------  ---------------------------------
  O_APPEND             完了    >> リダイレクトが正しく追記
  errno（方式B）       部分    Linux 風の負のエラーコード
  struct xv6_dirent    完了    POSIX との衝突を避けるため改名

ユーザ空間ライブラリ
~~~~~~~~~~~~~~~~~~~~

  API                          状態  備考
  ---------------------------  ----  ----------------------
  opendir/readdir/closedir     完了  POSIX 風のディレクトリ走査
  pmkdir                       完了  mkdir(path, mode)
  errno                        完了  グローバル変数 + 定義

シェル
~~~~~~

  機能  状態  備考
  ----  ----  ----------------------------------------
  >     完了  dup2 ベースに書き換え
  >>    完了  O_APPEND を使用
  <     完了  dup2 ベースに書き換え
  |     完了  既存、リダイレクトとの組み合わせも確認済み

サンドボックス
~~~~~~~~~~~~~~

sandbox コマンドは、子プロセスを許可された操作カテゴリに制限します。

  sandbox stdio,rpath cat README
  sandbox stdio cat README
  sandbox 0 echo hello

カテゴリ：stdio、rpath、wpath、proc、mem。


テストプログラム
----------------

  プログラム     目的
  -------------  --------------------------------------------
  lseektests     lseek のテスト
  dup2tests      標準出力リダイレクトによる dup2 のテスト
  waitpidtests   WNOHANG あり・なしの waitpid のテスト
  pmkdirtests    pmkdir のテスト
  errnotests     open 失敗時の errno のテスト
  pls            opendir/readdir を使った POSIX 風 ls


ビルドと実行
------------

  make clean
  make qemu

xv6 シェルで：

  $ lseektests
  $ dup2tests
  $ waitpidtests
  $ pls
  $ sandbox stdio,rpath cat README


POSIX 互換達成度の目安
----------------------

  基準                    達成度
  ----------------------  ------
  システムコール数        約 3%
  POSIX 必須 API          約 15%
  意味論の一致            約 6%

数値は概算です。完全な POSIX 準拠ではなく、
簡単なツールが動く実用的な互換性を目標としています。


設計上の注意
------------

- カーネル側のエラーコードは Linux 方式：
  -ENOENT のような負の値を返す。

- ユーザ空間のラッパーがそれを errno に変換し、-1 を返す。
  これで POSIX の挙動に一致する。

- struct xv6_dirent は xv6 本来のディレクトリエントリ。
  POSIX の struct dirent は user/dirent.h に別途定義。

- pledge は OpenBSD に着想を得た機構で、POSIX ではない。


既知の制限
----------

- errno が設定されるのは open のみ。
  他のシステムコールは -1 を返す。

- errno はグローバル変数で、スレッドローカルではない
  （xv6 にスレッドはない）。

- pledge は一方向性がない：再度呼ぶと制限を緩められる。

- サンドボックス内では exec が常に許可される。

- シェルはクォートを解釈しない
  （"stdio rpath" はパースされない）。

- 矢印キーが argv を壊す（コンソール入力の素朴な実装のため）。


謝辞
----

MIT PDOS による xv6-riscv を基にしています。
オリジナルは https://github.com/mit-pdos/xv6-riscv を参照してください。