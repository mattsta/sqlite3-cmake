sqlite3 for us
==============

A build wrapper around sqlite3 for CMake projects.

The included sqlite3 is built from the sqlite3 fossil branch `begin-concurrent-pnu-wal2` as:

```bash
./configure \
    --enable-rtree \
    --enable-geopoly \
    --enable-update-limit \
    --enable-fts5 \
    --enable-json1 \
    --disable-load-extension

make sqlite3.c
```
