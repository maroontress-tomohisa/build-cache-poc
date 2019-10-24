永続テーブルT1を作成。

> | DEP_ID (Integer) | String |
> | :--- | :--- |
> | _整数のID_ | _モジュール名=ID_, _モジュール名=ID_, ... |

イメージのビルドを開始してから終了するまでの間だけ生存するテーブルT2を作成。

> | Path | Module | SHA1 ID |
> | :--- | :--- | :--- |
> | _ファイルのパス_ | _モジュール名_ | _そのモジュールのリビジョン_ |

イメージのビルドを開始してから終了するまでの間だけ生存するテーブルT3を作成。
これはモジュールのDAGを表す。

> | String | String |
> | :--- | :--- |
> | _依存するモジュール名=ID_ | _依存されるモジュール名=ID_ |

モジュール`zzz`（SHA1 ID: `0ad9c1`）をビルドしようとする。
まずチェックアウトする。

`gcc -M *.c`で取得できるヘッダファイルの依存関係を求める。
依存する全インクルードファイルのパス、およびテーブルT2から
依存するモジュールとSHA1 IDを取得する。
ここでは、例えば以下のようになったとする:

```plaintext
zzz (0ad9c1) -> aaa (b88841)
zzz (0ad9c1) -> bbb (0d8a7b)
```

これをT3に登録。T3には`aaa`と`bbb`の依存関係も既に登録されている:

```plaintext
aaa (b88841) -> ccc (...)
bbb (0d8a7b) -> ddd (...)
```

`zzz`に関する依存関係をトポロジカルソートで並べる。
`aaa=b88841, bbb=0d8a7b, ccc=..., ddd=..., ...`は
`zzz`が直接または間接的に依存する全モジュールになる。

`DEP_ID = F(aaa=b88841, bbb=0d8a7b, ...)`として、
`DEP_ID`がテーブルに存在して、
かつ`zzz-0ad9c1-DEP_ID.tar.bz2`が存在すれば、
それをダウンロードして展開する（ビルドはスキップされた）。

ビルドのスキップに失敗したら、実際にビルドする。
`make install`でインストールされるファイルの一覧を求める。

```plaintext
$DESTDIR/usr/local/include/xxx.h
$DESTDIR/usr/local/include/yyy.h
...
$DESTDIR/usr/local/lib/libzzz.a
```

このリストの全てのエントリをテーブルT2に登録する。

| Path | Module | SHA1 ID |
| :--- | :--- | :--- |
| `$DESTDIR/usr/local/include/xxx.h` | `zzz` | `0ad9c1` |
| `$DESTDIR/usr/local/include/yyy.h` | `zzz` | `0ad9c1` |
| ... | | |
| `$DESTDIR/usr/local/lib/libzzz.a` | `zzz` | `0ad9c1` |

`zzz-0ad9c1-DEP_ID.tar.bz2`を作成して、アップロードする。
`DEP_ID = F(aaa=b88841, bbb=0d8a7b, ...)`をテーブルT1に登録する。

イメージのビルドが終了したら、テーブルT2は捨てる。

---

T1は実際には次のようにテーブルを分割しないとダメそう。

> | MOD_ID (Integer) | String |
> | :--- | :--- |
> | _整数のID_ | _モジュール名=ID_ |

> | DEP_ID (Integer) | String |
> | :--- | :--- |
> | _整数のID_ | _MOD_ID1_, _MOD_ID2_, ... |

---
