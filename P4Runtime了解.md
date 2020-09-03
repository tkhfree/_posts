---
title: P4Runtime了解
toc: true
thumbnail: 'https://pic.downk.cc/item/5eda03efc2a9a83be5241038.jpg'
comments: true
tags:
  - P4
  - SDN
date: 2020-08-27 08:44:05
urlname:
categories:
---

# Protocol Buffer

是一种序列化成二进制的数据缓冲编码协议

### 定义消息类型

```protobuf
syntax = "proto3"

message SearchRequest{
	String query = 1;
	int32 page_num = 2;
	int32 result_per_page = 3;
}
//分配字段类型、字段编号
//消息字段主要由两个规则：singular0次或者1次、repeated重复任意次，默认是repeated

message SearchResponse{
	string result = 1;
}

//可以保留字段，如果直接删除字段，以后用户在对消息类型进行更新时可能会重复使用删除掉的字段号，如果再次加载旧版本就会造成冲突
message foo{
	reserved 2,3,4 to 9;
	reserved "query";
}
```

如何使用.proto文件

```powershell
protoc --proto_path=IMPORT_PATH --cpp_out=DST_DIR --java_out=DST_DIR --python_out=DST_DIR --go_out=DST_DIR --ruby_out=DST_DIR --objc_out=DST_DIR --csharp_out=DST_DIR path/to/file.proto
```

使用protoc编译proto文件，将会以选择的语言输出代码，代码内容主要是根据你编写的message类型获取和设置字段，序列化输出以及解析序列化的输入。

- For **C++**, the compiler generates a `.h` and `.cc` file from each `.proto`, with a class for each message type described in your file.
- For **Java**, the compiler generates a `.java` file with a class for each message type, as well as a special `Builder` classes for creating message class instances.
- **Python** is a little different – the Python compiler generates a module with a static descriptor of each message type in your `.proto`, which is then used with a *metaclass* to create the necessary Python data access class at runtime.
- For **Go**, the compiler generates a `.pb.go` file with a type for each message type in your file.
- For **Ruby**, the compiler generates a `.rb` file with a Ruby module containing your message types.
- For **Objective-C**, the compiler generates a `pbobjc.h` and `pbobjc.m` file from each `.proto`, with a class for each message type described in your file.
- For **C#**, the compiler generates a `.cs` file from each `.proto`, with a class for each message type described in your file.
- For **Dart**, the compiler generates a `.pb.dart` file with a class for each message type in your file.

### 枚举

```protobuf
message SearchRequest{
	string query = 1;
	enum Corpus{
		UNIBERSAL = 0; //第一个值必须为0
		WEB = 1;
		IMAGEs = 2;
	}
	Corpus corpus = 2;
}
```

也可以使用别名，将`allow_alias`选项设置为`true`

```protobuf
message MyMessage1{
	enum Enum1{
		option allow_alias = true;
		UNKnow = 0;
		STATUS = 1;
		RUNNING = 1;
	}
}
```

### 使用其他消息类型

```protobuf
message SearchRequest{
	repeated Result result = 1;
}

message Result{
	string url = 1;
	string title = 2;
	repeated string snippet = 3;
}
```

如果没有定义在同一个.proto文件里，则可以导入

```protobuf
import "myproject/anotherproto.proto";
```

### 嵌套消息类型

```protobuf
message SearchResponse{
	message Result{
		string url = 1;
		string title = 2;
		repeated string snippets = 3;
	}
	repeated Result results = 1;
}
```

在另一个message里使用Result可以

```protobuf
message OtherMessage{
	SearchResponse.Result results = 1;
}
```

### 更新消息类型

- 不要更改现有的字段的编号，不适用的使用reserve保留字段。
- 添加了新的字段也可以使用新生成的代码来解析使用“旧”消息格式通过代码序列话的任何消息。同样，旧的代码也可以解析新代码创建的消息，旧的在解析时只会忽略新字段。

### Any消息类型

```protobuf
import "google/protobuf/any.proto

message Errorstatus{
	string message = 1;
	repeat google.protobuf.Any details = 2;
}
```

### Oneof消息类型

类似于case，当消息包含多个字段但是最多设置一个字段

```protobuf
message SampleMessage{
	oneof test_oneof{
		string name = 1;
		SubMessage sub_message = 9;
	}
}
```

### Maps

protocol buffers定义了一种关联映射

```protobuf
map<key_type, value_type> map_field = N;
```

key_type可以是integral和string类型，enum不能作为key，vaue_type可以是任何除了map的类型。

```protobuf
map<string, Project> projects = 3;
```

- map不能设置repeated字段
- 映射顺序是未定义的
- 从.proto文件生成的代码，映射是按照key值排列
- 不允许有重复的key值
- 当value值为空时，序列化后的值依据编译的语言而定

#### 向后兼容

```protobuf
message MapFieldEntry {
  key_type key = 1;
  value_type value = 2;
}

repeated MapFieldEntry map_field = N;
```

### package

加上包声明，可以在不同的包定义相同的消息类型

```protobuf
package foo.bar;
message Open{
	string name = 1;
}
//就可以使用完整名称引用
message Foo{
	foo.bar.Open open = 1; 
}
```

- In **C++** the generated classes are wrapped inside a C++ namespace. For example, `Open` would be in the namespace `foo::bar`.
- In **Java**, the package is used as the Java package, unless you explicitly provide an `option java_package` in your `.proto` file.
- In **Python**, the package directive is ignored, since Python modules are organized according to their location in the file system.
- In **Go**, the package is used as the Go package name, unless you explicitly provide an `option go_package` in your `.proto` file.
- In **Ruby**, the generated classes are wrapped inside nested Ruby namespaces, converted to the required Ruby capitalization style (first letter capitalized; if the first character is not a letter, `PB_` is prepended). For example, `Open` would be in the namespace `Foo::Bar`.
- In **C#** the package is used as the namespace after converting to PascalCase, unless you explicitly provide an `option csharp_namespace` in your `.proto` file. For example, `Open` would be in the namespace `Foo.Bar`.

### Defining Services

如果想在RPC里调用消息类型，需要定义服务接口，编译器编译的时候就会产生service interface 和stubs

```protobuf
service ServiceRPC{
	rpc Search(SearchRequest) returns (SearchResponse);
}
```

gRPC与protocol buffers都是谷歌开发的系统，因此可以使用一个gRPC编译器直接产生一个RPC调用代码。

### JSON Mapping

Proto3支持JSON编码

# tutorials for python

对于serialize和retrieve数据来说

- python有内置的picking方法，但是不够方便，与其他语言进行共享也不够完善
- 自己设计一种编码方式
- 使用XML，使用方便，应用广泛，但是XML需要巨大的性能消耗

使用Protocol buffers定义数据结构，然后使用编译器自动产生类，这个类依据有效的二进制格式实现.proto文件进行编码和解析（encoding and parser）。这个类提供getters和setters，并负责将.proto文件作为一个单元进行读写的细节操作，因此我们只需要关心暴露出来的接口。

### 写.proto文件

```protobuf
syntax = "proto2";

package tutorial;

message Person {
  required string name = 1;
  required int32 id = 2;
  optional string email = 3;

  enum PhoneType {
    MOBILE = 0;
    HOME = 1;
    WORK = 2;
  }

  message PhoneNumber {
    required string number = 1;
    optional PhoneType type = 2 [default = HOME];
  }

  repeated PhoneNumber phones = 4;
}

message AddressBook {
  repeated Person people = 1;
}
```

### 编译

```
protoc -I=$SRC_DIR --python_out=$DST_DIR $SRC_DIR/addressbook.proto
```

### The Protocol Buffer API

```python
import addressbook_pb2
person = addressbook_pb2.Person()
person.id = 1234
person.name = "John Doe"
person.email = "jdoe@example.com"
phone = person.phones.add()
phone.number = "555-4321"
phone.type = addressbook_pb2.Person.HOME
```

#### 枚举

枚举的被扩展成一组常量

#### 标准消息方法

每个消息类还包含许多其他方法，可用于检查或操作整个消息，包括：

- `IsInitialized()`：检查是否已设置所有必填字段。
- `__str__()`：返回消息的可读形式，对于调试特别有用。（通常以`str(message)`或调用`print message`。）
- `CopyFrom(other_msg)`：使用给定消息的值覆盖消息。
- `Clear()`：将所有元素清除为空状态。

#### 解析和序列化

- `SerializeToString()`：序列化消息并以字符串形式返回。
- `ParseFromString(data)`：解析给定字符串中的消息

### 写一个消息

现在写一个地址进入地址薄文件。需要创建并填充protocol buffer类的实例，然后将他们写入输出流。

这个python程序是读AddressBook，加入一个Person，把AddressBook放回文件。

```python
#! /usr/bin/python

import addressbook_pb2
import sys

# This function fills in a Person message based on user input.
def PromptForAddress(person):
  person.id = int(raw_input("Enter person ID number: "))
  person.name = raw_input("Enter name: ")

  email = raw_input("Enter email address (blank for none): ")
#  if email != "":
#    person.email = email

  while True:
    number = raw_input("Enter a phone number (or leave blank to finish): ")
    if number == "":
      break

    phone_number = person.phones.add()
    phone_number.number = number

    type = raw_input("Is this a mobile, home, or work phone? ")
    if type == "mobile":
      phone_number.phonetype = addressbook_pb2.Person.PhoneType.Mobile
    elif type == "home":
      phone_number.phonetype = addressbook_pb2.Person.PhoneType.Home
    elif type == "work":
      phone_number.phonetype = addressbook_pb2.Person.PhoneType.Work
    else:
      print "Unknown phone type; leaving as default value."

# Main procedure:  Reads the entire address book from a file,
#   adds one person based on user input, then writes it back out to the same
#   file.
if len(sys.argv) != 2:
  print "Usage:", sys.argv[0], "ADDRESS_BOOK_FILE"
  sys.exit(-1)

address_book = addressbook_pb2.AddressBook()

# Read the existing address book.
try:
  f = open(sys.argv[1], "rb")
  address_book.ParseFromString(f.read())
  f.close()
except IOError:
  print sys.argv[1] + ": Could not open file.  Creating a new one."

# Add an address.
PromptForAddress(address_book.person.add())

# Write the new address book back to disk.
f = open(sys.argv[1], "wb")
f.write(address_book.SerializeToString())
f.close()
```

### 读一个消息

```python
#! /usr/bin/python

import addressbook_pb2
import sys

# Iterates though all people in the AddressBook and prints info about them.
def ListPeople(address_book):
  for person in address_book.person:
    print "Person ID:", person.id
    print "  Name:", person.name
#    if person.HasField('email'):
#      print "  E-mail address:", person.email

    for phone_number in person.phones:
      if phone_number.phonetype == addressbook_pb2.Person.PhoneType.Mobile:
        print "  Mobile phone #: ",
      elif phone_number.phonetype == addressbook_pb2.Person.PhoneType.Home:
        print "  Home phone #: ",
      elif phone_number.phonetype == addressbook_pb2.Person.PhoneType.Work:
        print "  Work phone #: ",
      print phone_number.number

# Main procedure:  Reads the entire address book from a file and prints all
#   the information inside.
if len(sys.argv) != 2:
  print "Usage:", sys.argv[0], "ADDRESS_BOOK_FILE"
  sys.exit(-1)

address_book = addressbook_pb2.AddressBook()

# Read the existing address book.
f = open(sys.argv[1], "rb")
address_book.ParseFromString(f.read())
f.close()

ListPeople(address_book)
```

