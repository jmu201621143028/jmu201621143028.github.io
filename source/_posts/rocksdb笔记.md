---
title: rocksdb笔记
date: 2023-03-09 09:58:44
tags:
---
## 下载源码
```shell
git clone https://github.com/facebook/rocksdb.git
```
## 编译 & 安装
```shell
make
make install
```
## 测试
```cpp
#include <assert.h>
#include "rocksdb/db.h"

#include <string>
#include <iostream>

int main() {
    rocksdb::DB* db;
    rocksdb::Options options;
    options.create_if_missing = true;
    rocksdb::Status status =
        rocksdb::DB::Open(options, "./db_dir", &db);
    
    status = db->Put(rocksdb::WriteOptions(), "hello", "world");   //写
    if (status.ok()) {
        std::string k("hello");
        std::string v;
        db->Get(rocksdb::ReadOptions(), k, &v);  //读
        std::cout << v << std::endl;
    }
    assert(status.ok());
}
```
## 编译 & 链接
尝试：
```shell
g++ test.cc
```
报错：
```
/usr/local/include/rocksdb/wide_columns.h:51:21: error: ‘make_from_tuple’ is not a member of ‘std’; did you mean ‘make_tuple’?
   51 |         value_(std::make_from_tuple<Slice>(std::forward<VTuple>(value_tuple))) {
      |                     ^~~~~~~~~~~~~~~
      |                     make_tuple
```
解决方法：

```shell
g++ test.cc -std=c++17
```
报错:
```shell
undefined reference to `rocksdb::DB::Open(rocksdb::Options const&, std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> > const&, rocksdb::DB**)'
```
解决方法:
```shell
g++ test.cc -std=c++17 -lrocksdb
```
报错:
```shell
undefined reference to `pthread_mutex_trylock'
```
解决方法:
```shell
g++ test.cc -std=c++17 -lrocksdb -lpthread
```
报错:
```shell
undefined reference to `dlopen'
```
解决方法:
```shell
g++ test.cc -std=c++17 -lrocksdb -lpthread -ldl
```
ok, 成功啦。啰啰嗦嗦地写了这么多，是为了加强自己对编译链接的理解。

## 遍历rocksdb
```cpp
#include <cassert>
#include <rocksdb/db.h>
#include <iostream>
using namespace std;
int main(int argc, char** argv) {
    rocksdb::DB* db;  rocksdb::Options options;  
    options.create_if_missing = true;  //打开数据库  
    rocksdb::Status status = rocksdb::DB::Open(options, "./omap", &db);  
    assert(status.ok());  //插入数据  
    // db->Put(rocksdb::WriteOptions(), "key1", "value1");  
    // db->Put(rocksdb::WriteOptions(), "key2", "value2");  
    // db->Put(rocksdb::WriteOptions(), "key3", "value3");  
    // db->Put(rocksdb::WriteOptions(), "key4", "value4");  
    // db->Put(rocksdb::WriteOptions(), "key5", "value5");    
    rocksdb::Iterator* it = db->NewIterator(rocksdb::ReadOptions());  //使用迭代器获取所有key和value  
    for (it->SeekToFirst(); it->Valid(); it->Next()) { 
        cout << "key is: " << it->key().ToString() << endl;
        cout << "value is: " << hex << it->value().ToString() << endl;
        cout << "++++++++++++++++++++" << endl;
    }
    assert(it->status().ok());   
    delete it;   //关闭数据库  
    status= db->Close();  
    delete db;
}

```