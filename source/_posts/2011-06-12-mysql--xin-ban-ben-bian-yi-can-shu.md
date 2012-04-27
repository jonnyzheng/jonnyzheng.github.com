--- 
categories: 
  - Mysql
comments: true
layout: post
published: true
status: publish
tags: 
  - Mysql
title: "Mysql 新版本编译参数"
type: post
---
新版本的mysql 使用了CMAKE作为编译工具，不论这种方法的优缺点，先上编译参数：

cmake . \
-DCMAKE_BUILD_TYPE:STRING=Release \
-DCMAKE_INSTALL_PREFIX:PATH=/usr/local/mysql \
-DDEFAULT_CHARSET:STRING=utf8 \
-DDEFAULT_COLLATION:STRING=utf8_general_ci \
-DWITH_INNOBASE_STORAGE_ENGINE=1 \
-DMYSQL_UNIX_ADDR=/tmp/mysqld.sock \
-DCOMMUNITY_BUILD:BOOL=ON \
-DENABLED_PROFILING:BOOL=ON \
-DENABLE_DEBUG_SYNC:BOOL=OFF \
-DINSTALL_LAYOUT:STRING=STANDALONE \
-DMYSQL_MAINTAINER_MODE:BOOL=OFF \
-DWITH_EMBEDDED_SERVER:BOOL=ON \
-DWITH_EXTRA_CHARSETS:STRING=all \
-DWITH_SSL:STRING=bundled \
-DWITH_UNIT_TESTS:BOOL=OFF \
-DWITH_ZLIB:STRING=bundled \
-LH



具体可参考 http://forge.mysql.com/wiki/CMake

CMake和 configure的命令行参数比较 http://forge.mysql.com/wiki/CMake#Introduction
