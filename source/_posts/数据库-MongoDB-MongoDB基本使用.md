---
title: MongoDB基本使用
tags:
  - MongoDB
categories:
  - 数据库
  - MongoDB
date: 2019-10-13 19:25:28
---
![](https://docs.mongodb.com/images/mongodb-logo.png)
<!--more-->

# 服务器相关
## 基础概念
在mongodb中基本的概念是文档、集合、数据库
- 数据库
一个mongodb中可以建立多个数据库。
- 集合
集合就是 MongoDB 文档组 集合处于数据库中 文档处于集合中
- 文档
文档是一组键值(key-value)对(即 BSON)
文档对应数据记录的行

## 安装驱动到Centos

**升级cmake**
```shell
wget https://cmake.org/files/v3.6/cmake-3.6.2.tar.gz    
tar xvf cmake-3.6.2.tar.gz && cd cmake-3.6.2/
./bootstrap

gmake
gmake install（需要在su命令下执行，或者直接使用root账户安装）
/usr/local/bin/cmake --version

yum remove cmake -y
ln -s /usr/local/bin/cmake /usr/bin/
cmake --version
```
**安装**

[官方文档](https://mongoc.org/libmongoc/current/installing.html)
## 安装驱动到Centos
```
yum install gcc-c++

yum install clang

# 安装boost
yum -y install gcc-c++ python-devel bzip2-devel zlib-devel
tar zxvf 安装包
sudo ./bootstrap.sh --prefix=/usr/local/boost
sudo ./b2 install
进入 目录下的tools/build
sudo ./bootstrap.sh
sudo ./b2 install --prefix=/usr/local/boost

wget https://cmake.org/files/v3.5/cmake-3.5.2.tar.gz
tar -zxvf cmake-3.5.2.tar.gz
cd cmake-3.5.2
./bootstrap
gmake
gmake install

# 删除原来cmake版本，建立软连接，测试
yum remove cmake -y
ln -s /usr/local/bin/cmake /usr/bin/
cmake --version

# 安装pip
yum install python-pip
pip install --upgrade pip

# 安装py3
https://segmentfault.com/a/1190000015628625
```

## 安装到Ubuntu 18.04
[整理自官方文档](https://docs.mongodb.com/manual/administration/install-on-linux/)

- 导入公共秘钥, 另你的系统"信任"下面的包
```shell
wget -qO - https://www.mongodb.org/static/pgp/server-4.2.asc | sudo apt-key add -
```
这一步需要显示OK
- 导入第三方MongoDB源
```shell
echo "deb [ arch=amd64 ] https://repo.mongodb.org/apt/ubuntu bionic/mongodb-org/4.2 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-4.2.list
```
- 导入源之后更新
```shell
sudo apt-get update
```
- 安装MongoDB稳定版本包(我安装的时候稳定版本为4.2)
```shell
sudo apt-get install -y mongodb-org=4.2.0 mongodb-org-server=4.2.0 mongodb-org-shell=4.2.0 mongodb-org-mongos=4.2.0 mongodb-org-tools=4.2.0
```
- 禁用apt-get自动更新MongoDB
```shell
echo "mongodb-org hold" | sudo dpkg --set-selections
echo "mongodb-org-server hold" | sudo dpkg --set-selections
echo "mongodb-org-shell hold" | sudo dpkg --set-selections
echo "mongodb-org-mongos hold" | sudo dpkg --set-selections
echo "mongodb-org-tools hold" | sudo dpkg --set-selections
```
## 基础shell操作
**启动 关闭 重启**
`sudo service mongod start(stop\restart)`

*启动成功标志*
日志文件`/var/log/mongodb/mongod.log`中显示`waiting for connections on port 27017`

**进入交互式命令行**
`mongo`

**指定端口运行 默认27017**
`mongo --port 28015`

**非安装机通过shell连接安装机**
[直连](https://docs.mongodb.com/manual/mongo/#mongodb-instance-on-a-remote-host)
[带登录验证](https://docs.mongodb.com/manual/mongo/#mongodb-instance-with-authentication)

---

**显示你正在使用的数据库**
`db`
**切换数据库**
`use <database>`

### CRUD

**C**

*插入一个文档*
`db.collection.insertOne()` 
这里的collection代指的一个集合, 集合的名字可以自定义如果集合并不存在你的数据库中 则自动创建这个集合.
```shell
db.inventory.insertOne(
   { item: "canvas", qty: 100, tags: ["cotton"]}
)
```
---
*插入多个文档*
`db.inventory.insertMany()`
```shell
db.inventory.insertMany([
   { item: "journal", qty: 25, tags: ["blank", "red"]},
   { item: "mat", qty: 85, tags: ["gray"]},
   { item: "mousepad", qty: 25, tags: ["gel", "blue"]}
])
```
---
*插入任意项的文档*
`db.collection.insert()`

---
*_id项*
每一个文档都有自己独一的id 如果插入文档的时候没有指定则会生成随机的`_id`
```shell
{ "_id" : ObjectId("5da325520e80dfd9258a29be"), "x" : 1 }
```
可以通过在插入的时候指定`"_id": xxxx`来自定义id

[额外的方法](https://docs.mongodb.com/manual/reference/insert-methods/#additional-methods-for-inserts)


**R**
`db.collection.find({ item: "canvas" })`
[拦截器-](https://docs.mongodb.com/manual/core/document/#document-query-filter)在collection中查找含有`item: "canvas"`的文档, 并打印
如果不含`{ item: "canvas" }`则代表打印所有的文档

`db.collection.find({status: {$in: [ "A", "D" ]}})` 
`status: {$in: [ "A", "D" ]}`意为A, D不在此status中

`db.inventory.find( { status: "A", qty: { $lt: 30 } } )`
`qty: { $lt: 30 }`意为此qty的值少于30

`db.inventory.find( { $or: [ { status: "A" }, { qty: { $lt: 30 } } ] } )`
`{$or: [{ status: "A" }, { qty: { $lt: 30 }}]}`符合二者之一


[2019年10月13日21:58:24 - 开头](https://docs.mongodb.com/manual/tutorial/query-embedded-documents/)

# C++ Driver
[照着官方文档安装了下](https://mongocxx.org/mongocxx-v3/installation/)
期间遇到了两个坑一个是我没有安装Git导致提示无法克隆
还有就是最后阶段我没安装pkg-config 导致的无法找到头文件

接下来看一些简单的教程
## CRUD
```c++
#include <cstdint>
#include <iostream>
#include <vector>
#include <bsoncxx/json.hpp>
#include <mongocxx/client.hpp>
#include <mongocxx/stdx.hpp>
#include <mongocxx/uri.hpp>
#include <bsoncxx/builder/stream/helpers.hpp>
#include <mongocxx/instance.hpp>
#include <bsoncxx/builder/stream/document.hpp>

using bsoncxx::builder::stream::close_array;
using bsoncxx::builder::stream::close_document;
using bsoncxx::builder::stream::document;
using bsoncxx::builder::stream::finalize;
using bsoncxx::builder::stream::open_array;
using bsoncxx::builder::stream::open_document;


void insert(mongocxx::collection coll_test)
{
	auto builder = document{};

	bsoncxx::document::value doc_value = builder
			<< "name" << "Lsmg"
			<< "type" << "Lemon"
			<< "count" << 1
			<< "versions" << open_array
			<< "v3.2" << "v3.0" << "v2.6"
			<< close_array
			<< "info" << open_document
			<< "x" << 203
			<< "y" << 102
			<< close_document
			<< finalize;

	bsoncxx::document::view view_test = doc_value.view();

	// 打印插入内容制定key的value
	std::string name = view_test["name"].get_utf8().value.to_string();
	std::cout << name << std::endl;

	// 获取插入ID
	auto result = coll_test.insert_one(view_test);
	std::string id1 = result->inserted_id().get_oid().value.to_string();
	std::cout << id1 << std::endl;

}

void insert_many(mongocxx::collection coll_test)
{
	// 插入多个
	std::vector<bsoncxx::document::value> documents;
	documents.reserve(10);
	for (int i = 0; i < 10; ++i)
	{
		documents.push_back(document{} << "i" << i << finalize);
	}
	auto result = coll_test.insert_many(documents);

	// 获取多个的返回ID
	auto id_maps = result->inserted_ids();
	for (auto iter : id_maps)
	{
		std::cout << iter.second.get_oid().value.to_string() << std::endl;
	}

}

void find(mongocxx::collection coll_test)
{
	// 查询
	mongocxx::cursor cursor = coll_test.find({});
	for (auto doc : cursor)
	{
		std::cout << bsoncxx::to_json(doc) << "\n";
	}
}


void update_one(mongocxx::collection coll_test)
{
	coll_test.update_one(document{} << "i" << 9 << finalize,
			document{} << "$set" << open_document <<
			"i" << 110 << close_document << finalize);
}

void update_many(mongocxx::collection coll_test)
{

	coll_test.update_many(document{} << "i" << open_document <<
					           "$lt" << 8 << close_document << finalize,
			document{} << "$inc" << open_document <<
					           "i" << 100 << close_document << finalize);
}

void delete_one(mongocxx::collection coll_test)
{
	coll_test.delete_one(document{} << "i" << 110 << finalize);
}

void delete_many(mongocxx::collection coll_test)
{
	// 删除存在 i： 0 的文档
	coll_test.delete_many(document{} << "i" << 0 << finalize);

	/*
	 * (>) 大于 - $gt
	 * (<) 小于 - $lt
	 * (>=) 大于等于 - $gte
	 * (<= ) 小于等于 - $lte
	 * */

	// 删除 i大于3小于8的文档
	coll_test.delete_many(document{} << "i" << open_document <<
	                                 "$gt" << 0 <<
	                                 "$lt" << 200 <<
	                                 close_document << finalize);
}

void create_index(mongocxx::collection coll_test)
{

	// 1 代表生序 -1代表降序
	auto index_specification = document{} << "i" << -1 << finalize;

	// move的本质就是帮助编译器选择重载函数, 告诉编译器"请尽量把此参数当做右值来处理"
	coll_test.create_index(std::move(index_specification));
}

int main()
{
	mongocxx::instance instance{};
	mongocxx::client client{mongocxx::uri{}};

	mongocxx::database db = client["mydb"];
	mongocxx::collection coll_test = db["test"];

	create_index(coll_test);

	find(coll_test);


	exit(0);
}
```
## 线程安全相关
不能同时对一个client的多个线程同时操作.
同时只能有一个线程对一个client操作, 包括对`mongocxx::client`的`mongocxx::client_session`, `mongocxx::database, mongocxx::collection`, and `mongocxx::cursor`同样如此

简单解决可以通过用同一个`mongocxx::uri`生成多个`mongocxx::client`来解决这个问题

最好的解决方法用`mongocxx::uri`生成一个`mongocxx::pool` 通过`mongocxx::pool`的`acquire()`来获取操作的client
官方demo如下
```c++
mongocxx::instance instance{};
mongocxx::pool pool{mongocxx::uri{}};

auto threadfunc = [](mongocxx::client& client, std::string dbname) {
  auto col = client[dbname]["col"].insert_one({});
};

// Great! Using the pool allows the clients to be synchronized while sharing only one
// background monitoring thread.
std::thread t1 ([&]() {
  auto c = pool.acquire();
  threadfunc(*c, "db1");
  threadfunc(*c, "db2");
});

std::thread t2 ([&]() {
  auto c = pool.acquire();
  threadfunc(*c, "db2");
  threadfunc(*c, "db1");
});

t1.join();
t2.join();
```

关于fork的安全性
`Neither a mongocxx::client or a mongocxx::pool can be safely copied when forking. Because of this, any client or pool must be created after forking, not before.`

官方建议使用`mongocxx::pool`而不是`mongocxx::client` 即使你的应用只有一个线程
`mongocxx::client` 每60S会检查一次自己监控的cluster, 使用前者会有专门的线程来进行检查而且是10S一次

`mongocxx::pool`可以用于多线程中, 也能用来创建客户端. 然而每个`mongocxx::client`只能被一个线程使用`最大默认为100 最小默认为0`

## BSON
**Document Builders**
一共提供了三个接口来创建文档
官网上提到`这三个接口会得到同样的结果, 选择哪个完全是由于美学`

*“One-off” builder functions*
```c++
using bsoncxx::builder::basic::kvp;
// { "hello": "world" }
bsoncxx::document::value document = bsoncxx::builder::basic::make_document(kvp("hello", "world"));
```

*Basic builder*
```c++
using bsoncxx::builder::basic::kvp;
// { "hello" : "world" }
bsoncxx::builder::basic::document basic_builder{};
basic_builder.append(kvp("hello", "world"));
bsoncxx::document::value document = basic_builder.extract();

// basic::document builds a BSON document.
auto doc = builder::basic::document{};
// 通过使用kvp(k, v) 来 append 键值对 to a document 
using bsoncxx::builder::basic::kvp;

doc.append(k, v); // 插入一般的键值对
// 对于k一般为string
// 对于v则包含多种情况 可以使用 bsoncxx::types命名空间里的如下类型
struct b_eod;
struct b_double; // types::b_double{3.14159}
struct b_utf8;
struct b_document;
struct b_array;
struct b_binary;
struct b_undefined;
struct b_oid;
struct b_bool; // types::b_bool{false}
struct b_date; 
struct b_null;
struct b_regex;
struct b_dbpointer;
struct b_code;
struct b_symbol;
struct b_codewscope;
struct b_int32;
struct b_timestamp;
struct b_int64;
struct b_decimal128;
struct b_minkey;
struct b_maxkey;

// 插入k v (v是数组)
arr.append("hello");
arr.append(false, types::b_bool{true}, types::b_double{1.234});

// We can get a view of the resulting bson by calling view()
auto v = doc.view();

// Use 'v' so we don't get compiler warnings.
return v.empty() ? EXIT_FAILURE : EXIT_SUCCESS;
```

*Stream builder*
我现在最喜欢的方法, 不过官方不建议这样, 因为需要保持这个流不被重新初始化, 这样在跨行生成的时候有困难.
官方建议使用上面的两个方法
```c++
// { "hello" : "world" }
using bsoncxx::builder::stream;
bsoncxx::document::value document = stream::document{} << "hello" << "world" << stream::finalize;

// 此外还有
using builder::stream::open_document; // {
using builder::stream::close_document;// }
using builder::stream::open_array; // [
using builder::stream::close_array;// ]
```

**Building arrays**
```c++
// [ 1, 2, 3 ]

const auto elements = {1, 2, 3};
auto array_builder = bsoncxx::builder::basic::array{};

for (const auto& element : elements) {
    array_builder.append(element);
}
```
通过使用lambda来构建 k, v (v是数组)
```c++
// { "foo" : [ 1, 2, 3 ] }

using bsoncxx::builder::basic::kvp;
using bsoncxx::builder::basic::sub_array;

const auto elements = {1, 2, 3};
auto doc = bsoncxx::builder::basic::document{};
doc.append(kvp("foo", [&elements](sub_array child) {
    for (const auto& element : elements) {
        child.append(element);
    }
}));
```

通过使用流来生成
```c++
// { "subdocs" : [ { "key" : 1 }, { "key" : 2 }, { "key" : 3 } ], "another_key" : 42 }
using namespace bsoncxx;

builder::stream::document builder{};

auto in_array = builder << "subdocs" << builder::stream::open_array;
for (auto&& e : {1, 2, 3}) {
    in_array = in_array << builder::stream::open_document << "key" << e
                        << builder::stream::close_document;
}
auto after_array = in_array << builder::stream::close_array;

after_array << "another_key" << 42;

document::value doc = after_array << builder::stream::finalize;

std::cout << to_json(doc) << std::endl;
```