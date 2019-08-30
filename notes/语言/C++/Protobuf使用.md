# Protobuf使用

## 目录
* [proto3的更新](#proto3的更新)
* [定义协议格式](#定义协议格式)
* [编译protobuf](#编译protobuf)
* [protobuf_API](#protobuf_API)
  * [枚举和嵌套类](#枚举和嵌套类)
  * [标准消息方法](#标准消息方法)
  * [解析和序列化](#解析和序列化)
* [写一条消息](#写一条消息)
* [阅读消息](#阅读消息)    
* [编译](#编译)    
* [Protobuf扩展](#Protobuf扩展)    
* [优化](#优化)    
* [高级用法](#高级用法)    

### proto3的更新

* 在第一行非空白非注释行，必须写：
```cpp
syntax  = "proto3";
```
* 字段规则移除了 `required`，并把 `optional` 改名为 `singular`;\
在 `proto2` 中 `required` 也是不推荐使用的。`proto3` 直接从语法层面上移除了 `required`规则。其实可以做的更彻底，把所有字段规则描述都撤销，原来的`repeated` 改为在类型或字段名后加一对中括号。这样是不是更简洁？

* `repeated`字段默认采用 `packed` 编码;\
在 `proto2` 中，需要明确使用 `[packed=true]` 来为字段指定比较紧凑的 `packed` 编码方式。

* 移除了`default` 选项;\
在 `proto2` 中，可以使用 `default` 选项为某一字段指定默认值。在 `proto3` 中，字段的默认值只能根据字段类型由系统决定。也就是说，默认值全部是约定好的，而不再提供指定默认值的语法。\
在字段被设置为默认值的时候，该字段不会被序列化。这样可以节省空间，提高效率。
但这样就无法区分某字段是根本没赋值，还是赋值了默认值。这在 `proto3` 中问题不大，但在 `proto2` 中会有问题。\
比如，在更新协议的时候使用 `default` 选项为某个字段指定了一个与原来不同的默认值，旧代码获取到的该字段的值会与新代码不一样。

* 枚举类型的第一个字段必须为 0 ;

* 移除了对分组的支持;\
分组的功能完全可以用消息嵌套的方式来实现，并且更清晰。在 `proto2` 中已经把分组语法标注为『过期』了。这次也算清理垃圾了。

* 移除了对扩展的支持，新增了 `Any` 类型;\
`Any` 类型是用来替代 `proto2` 中的扩展的。目前还在开发中。\
`proto2` 中的扩展特性很像 `Swift` 语言中的扩展。理解起来有点困难，使用起来更是会带来不少混乱。\
相比之下，`proto3` 中新增的 `Any` 类型有点像 `C/C++` 中的 `void*` ，好理解，使用起来逻辑也更清晰。

* 增加了 `JSON` 映射特性;\
语言的活力来自于与时俱进。当前，`JSON` 的流行有其充分的理由。很多『现代化』的语言都内置了对 `JSON` 的支持，比如 `Go`、`PHP` 等。而 `C++` 这种看似包罗万象的学院派语言，因循守旧、故步自封，以致于现出了式微的苗头。

* map支持
```cpp
map<key_type, value_type> map_field = N;
example:
map<string, Project> projects = 3;
```

* 在 proto3 中，纯数字类型的 repeated 字段编码时候默认采用 packed 编码（具体原因见 [Protocol Buffer 编码原理](https://github.com/halfrost/Halfrost-Field/blob/master/contents/Protocol/Protocol-buffers-encode.md#六-protocol-buffer-编码原理) 这一章节）

### 定义协议格式
`.proto`文件中的定义很简单：为要序列化的每个数据结构添加消息，然后为消息中的每个字段指定名称和类型。这是`.proto`定义您的消息的文件`addressbook.proto`。

(好的`.proto`文件命名风格是：`packagename.messagename.proto`)
```Cpp
syntax = "proto3";

package tutorial;

message Person {
  string name = 1;
  int32 id = 2;
  string email = 3;

  enum PhoneType {
    MOBILE = 0;
    HOME = 1;
    WORK = 2;
  }

  message PhoneNumber {
    string number = 1;
    PhoneType type = 2;
  }

  repeated PhoneNumber phones = 4;
}

message AddressBook {
  repeated Person people = 1;
}

```
该`.proto`文件以包声明开头，这有助于防止不同项目之间的命名冲突。在`C++`中，生成的类将放在与包名匹配的命名空间中。

每个元素上的“= 1”，“= 2”标记标识该字段在二进制编码中使用的唯一“标记”。标签号1-15需要少于一个字节来编码而不是更高的数字，因此作为优化，您可以决定将这些标签用于常用或重复的元素，将标签16和更高版本留给不太常用的可选元素。重复字段中的每个元素都需要重新编码标记号，因此重复字段特别适合此优化。

可以指定的最小字段编号为1，最大字段编号为2^29^-1 或 536,870,911。也不能使用数字 19000 到 19999（FieldDescriptor :: kFirstReservedNumber 到 FieldDescriptor :: kLastReservedNumber），因为它们是为 Protocol Buffers实现保留的。

必须使用以下修饰符之一注释每个字段：

* **required(proto3中移除)**：必须提供该字段的值，否则该消息将被视为“未初始化”。如果`libprotobuf`在调试模式下编译，则序列化未初始化的消息将导致断言失败。在优化的构建中，将跳过检查并始终写入消息。但是，解析未初始化的消息将始终失败（通过`false`从解析方法返回）。除此之外，必填字段的行为与可选字段完全相同。

* **optional(proto3中为singular)**：该字段可能已设置，也可能未设置。如果未设置可选字段值，则使用默认值。对于简单类型，您可以指定自己的默认值，就像我们`type`在示例中为电话号码所做的那样。否则，使用系统默认值：数字类型为0，字符串为空字符串，`bools`为`false`。对于嵌入式消息，默认值始终是消息的“默认实例”或“原型”，其中没有设置其字段。调用访问器以获取尚未显式设置的可选（或必需）字段的值始终返回该字段的默认值。

* **repeated(proto3默认采用 packed 编码)**：该字段可以重复任意次数（包括零）。重复值的顺序将保留在协议缓冲区中。将重复字段视为动态大小的数组。

* **proto3 中移除了default选项**:字段的默认值只能根据字段类型由系统决定。也就是说，默认值全部是约定好的，而不再提供指定默认值的语法。在字段被设置为默认值的时候，该字段不会被序列化。这样可以节省空间，提高效率。

### 编译protobuf
现在运行编译器，指定源目录（应用程序的源代码所在的位置 - 如果不提​​供值，则使用当前目录），目标目录（您希望生成的代码在哪里;通常相同`$SRC_DIR`） ，以及你的道路`.proto`。：

```cpp
protoc -I = $ SRC_DIR --cpp_out = $ DST_DIR $ SRC_DIR / addressbook.proto
```

这里都生成到当前目录，输入

```cpp
protoc -I=. --cpp_out=. ./addressbook.proto
protoc --cpp_out=. addressbook.proto // 这种也可以
```

因为您需要`C++`类，所以使用该`--cpp_out`选项 - 为其他支持的语言提供了类似的选项。

这将在指定的目标目录中生成以下文件：

* `addressbook.pb.h`，标头声明您生成的类。
* `addressbook.pb.cc`，其中包含您的类的实现。

### protobuf_API
`addressbook.pb.h`中，可以看到指定的每条消息都有一个类`addressbook.proto`。对于`Person`类，可以看到编译器已为每个字段生成了访问器。 例如，对于名称，`ID`，电子邮件和电话字段，有以下方法：

```cpp
// name
inline bool has_name() const;
inline void clear_name();
inline const ::std::string& name() const;
inline void set_name(const ::std::string& value);
inline void set_name(const char* value);
inline ::std::string* mutable_name();

// id
inline bool has_id() const;
inline void clear_id();
inline int32_t id() const;
inline void set_id(int32_t value);

// email
inline bool has_email() const;
inline void clear_email();
inline const ::std::string& email() const;
inline void set_email(const ::std::string& value);
inline void set_email(const char* value);
inline ::std::string* mutable_email();

// phones
inline int phones_size() const;
inline void clear_phones();
inline const ::google::protobuf::RepeatedPtrField< ::tutorial::Person_PhoneNumber >& phones() const;
inline ::google::protobuf::RepeatedPtrField< ::tutorial::Person_PhoneNumber >* mutable_phones();
inline const ::tutorial::Person_PhoneNumber& phones(int index) const;
inline ::tutorial::Person_PhoneNumber* mutable_phones(int index);
inline ::tutorial::Person_PhoneNumber* add_phones();
 ```

对于字符串 ： 一个`mutable_`让你获得指向字符串的直接指针的`getter`，以及一个额外的`setter`。请注意，`mutable_email()`即使`email`尚未设置，您也可以进行呼叫; 它将自动初始化为空字符串。如果你在这个例子中有一个单数的消息字段，它也有一个`mutable_`方法但不是一个`set_`方法。

重复的字段也有一些特殊的方法 - 如果你看一下`repeated phones`字段的方法，你会发现你可以

* 检查重复的字段的` _size` (换句话说，有多少电话号码与此相关联的 `Person`).

* 使用索引获取指定的电话号码.

* 更新指定索引处的现有电话号码.

* 在邮件中添加另一个电话号码然后可以编辑（重复的标量类型`add_`只允许您传入新值）.

有关协议编译器为任何特定字段定义生成的确切成员的详细信息，请参阅[C ++生成的代码参考](https://developers.google.com/protocol-buffers/docs/reference/cpp-generated)。

#### 枚举和嵌套类
生成的代码包含`PhoneType`与您的`.proto`枚举对应的枚举。您可以参考这个类型`Person::PhoneType`及其作为值的`Person::MOBILE`，`Person::HOME`和`Person::WORK`（实现细节是稍微复杂一点，但你并不需要了解他们使用`ENUM`）。

编译器还为您生成了一个嵌套类`Person::PhoneNumber`。如果查看代码，可以看到实际调用了“真实”类`Person_PhoneNumber`，但是在内部定义的`typedef Person`允许您将其视为嵌套类。唯一不同的情况是，如果你想在另一个文件中转发声明类 - 你不能在`C++`中转发声明嵌套类型，但你可以转发声明`Person_PhoneNumber`。

#### 标准消息方法
每个消息类还包含许多其他方法，可用于检查或操作整个消息，包括：

* **bool IsInitialized() const;** 检查是否已设置所有必填字段。

* **string DebugString() const;** 返回消息的人类可读表示，对调试特别有用。

* **void CopyFrom(const Person& from);** 使用给定消息的值覆盖消息。

* **void Clear();** 清除所有元素回到空状态。

以下部分中描述的这些和`I / O`方法实现了`Message`所有`C ++`协议缓冲区类共享的接口。有关详细信息，请参阅[完整的API文档Message](https://developers.google.com/protocol-buffers/docs/reference/cpp/google.protobuf.message#Message)。

#### 解析和序列化
最后，每个协议缓冲区类都有使用协议缓冲区[二进制格式](https://developers.google.com/protocol-buffers/docs/encoding)编写和读取所选类型消息的方法。这些包括：

* **bool SerializeToString(string* output) const;** 序列化消息并将字节存储在给定的字符串中。请注意，字节是二进制的，而不是文本; 我们只将该 `string` 类用作方便的容器。

* **bool ParseFromString(const string& data);**  解析给定字符串中的消息。

* **bool SerializeToOstream(ostream* output) const;** 将消息写入给定的 `C++ ostream`。

* **bool ParseFromIstream(istream* input);** 解析来自给定 `C++` 的消息 `istream`。

这些只是解析和序列化提供的几个选项。再次，请参阅[MessageAPI参考](https://developers.google.com/protocol-buffers/docs/reference/cpp/google.protobuf.message#Message)以获取完整列表。

### 写一条消息
现在尝试使用协议缓冲类。地址簿应用程序能够做的第一件事是将个人详细信息写入地址簿文件。为此，需要创建并填充协议缓冲区类的实例，然后将它们写入输出流。

这是一个程序，它从文件中读取一个`AddressBook`，根据用户输入在`AddressBook`文件中添加一个新的`Person`，然后再将新文本写回文件。直接调用或引用协议编译器生成的代码的部分将突出显示。

```Cpp
#include <iostream>
#include <fstream>
#include <string>
#include "addressbook.pb.h"
using namespace std;

// This function fills in a Person message based on user input.
void PromptForAddress(tutorial::Person* person) {
  cout << "Enter person ID number: ";
  int id;
  cin >> id;
  person->set_id(id);
  cin.ignore(256, '\n');

  cout << "Enter name: ";
  getline(cin, *person->mutable_name());

  cout << "Enter email address (blank for none): ";
  string email;
  getline(cin, email);
  if (!email.empty()) {
    person->set_email(email);
  }

  while (true) {
    cout << "Enter a phone number (or leave blank to finish): ";
    string number;
    getline(cin, number);
    if (number.empty()) {
      break;
    }

    tutorial::Person::PhoneNumber* phone_number = person->add_phones();
    phone_number->set_number(number);

    cout << "Is this a mobile, home, or work phone? ";
    string type;
    getline(cin, type);
    if (type == "mobile") {
      phone_number->set_type(tutorial::Person::MOBILE);
    } else if (type == "home") {
      phone_number->set_type(tutorial::Person::HOME);
    } else if (type == "work") {
      phone_number->set_type(tutorial::Person::WORK);
    } else {
      cout << "Unknown phone type.  Using default." << endl;
    }
  }
}

// Main function:  Reads the entire address book from a file,
//   adds one person based on user input, then writes it back out to the same
//   file.
int main(int argc, char* argv[]) {
  // Verify that the version of the library that we linked against is
  // compatible with the version of the headers we compiled against.
  GOOGLE_PROTOBUF_VERIFY_VERSION;

  if (argc != 2) {
    cerr << "Usage:  " << argv[0] << " ADDRESS_BOOK_FILE" << endl;
    return -1;
  }

  tutorial::AddressBook address_book;

  {
    // Read the existing address book.
    fstream input(argv[1], ios::in | ios::binary);
    if (!input) {
      cout << argv[1] << ": File not found.  Creating a new file." << endl;
    } else if (!address_book.ParseFromIstream(&input)) {
      cerr << "Failed to parse address book." << endl;
      return -1;
    }
  }

  // Add an address.
  PromptForAddress(address_book.add_people());

  {
    // Write the new address book back to disk.
    fstream output(argv[1], ios::out | ios::trunc | ios::binary);
    if (!address_book.SerializeToOstream(&output)) {
      cerr << "Failed to write address book." << endl;
      return -1;
    }
  }

  // Optional:  Delete all global objects allocated by libprotobuf.
  google::protobuf::ShutdownProtobufLibrary();

  return 0;
}
```

注意`GOOGLE_PROTOBUF_VERIFY_VERSION`宏。在使用`C ++`协议缓冲区库之前执行此宏是一种很好的做法 - 尽管不是绝对必要的。它验证您没有意外链接到与您编译的标头版本不兼容的库版本。如果检测到版本不匹配，程序将中止。请注意，每个`.pb.cc`文件在启动时都会自动调用此宏。

还要注意`ShutdownProtobufLibrary()`程序结束时的调用。所有这一切都是删除协议缓冲区库分配的所有全局对象。对于大多数程序来说这是不必要的，因为该过程无论如何都要退出，操作系统将负责回收其所有内存。但是，如果您使用需要释放每个最后一个对象的内存泄漏检查程序，或者您正在编写可以由单个进程多次加载和卸载的库，那么您可能希望强制协议缓冲区清除所有内容。

### 阅读消息
当然，如果无法从中获取任何信息，那么地址簿就不会有多大用处！此示例读取上面示例创建的文件并打印其中的所有信息。
```cpp
#include <iostream>
#include <fstream>
#include <string>
#include "addressbook.pb.h"
using namespace std;

// Iterates though all people in the AddressBook and prints info about them.
void ListPeople(const tutorial::AddressBook& address_book) {
  for (int i = 0; i < address_book.people_size(); i++) {
    const tutorial::Person& person = address_book.people(i);

    cout << "Person ID: " << person.id() << endl;
    cout << "  Name: " << person.name() << endl;
    if (person.email() !=  "") {
      cout << "  E-mail address: " << person.email() << endl;
    }

    for (int j = 0; j < person.phones_size(); j++) {
      const tutorial::Person::PhoneNumber& phone_number = person.phones(j);

      switch (phone_number.type()) {
        case tutorial::Person::MOBILE:
          cout << "  Mobile phone #: ";
          break;
        case tutorial::Person::HOME:
          cout << "  Home phone #: ";
          break;
        case tutorial::Person::WORK:
          cout << "  Work phone #: ";
          break;
      }
      cout << phone_number.number() << endl;
    }
  }
}

// Main function:  Reads the entire address book from a file and prints all
//   the information inside.
int main(int argc, char* argv[]) {
  // Verify that the version of the library that we linked against is
  // compatible with the version of the headers we compiled against.
  GOOGLE_PROTOBUF_VERIFY_VERSION;

  if (argc != 2) {
    cerr << "Usage:  " << argv[0] << " ADDRESS_BOOK_FILE" << endl;
    return -1;
  }

  tutorial::AddressBook address_book;

  {
    // Read the existing address book.
    fstream input(argv[1], ios::in | ios::binary);
    if (!address_book.ParseFromIstream(&input)) {
      cerr << "Failed to parse address book." << endl;
      return -1;
    }
  }

  ListPeople(address_book);

  // Optional:  Delete all global objects allocated by libprotobuf.
  google::protobuf::ShutdownProtobufLibrary();

  return 0;
}
```

### 编译
    g++ -Wall -std=c++11 write.cpp addressbook.pb.cc -o write `pkg-config --cflags --libs protobuf`
    g++ -Wall -std=c++11 read.cpp addressbook.pb.cc -o read `pkg-config --cflags --libs protobuf`

### Protobuf扩展
在释放使用协议缓冲区的代码之后迟早，您无疑会想要“改进”协议缓冲区的定义。如果你希望你的新缓冲区向后兼容，并且你的旧缓冲区是向前兼容的 - 而且你几乎肯定想要这个 - 那么你需要遵循一些规则。在新版本的协议缓冲区中：

* 不得更改任何现有字段的标记号。

* 不得添加或删除任何必填字段。

* 可以删除可选或重复的字段。

* 可以添加新的可选或重复字段，但必须使用新的标记号（即从未在此协议缓冲区中使用的标记号，甚至不包括已删除的字段）。

（这些规则有[一些例外](https://developers.google.com/protocol-buffers/docs/proto#updating)，但它们很少使用。）

如果您遵循这些规则，旧代码将很乐意阅读新消息并简单地忽略任何新字段。对于旧代码，已删除的可选字段将只具有其默认值，删除的重复字段将为空。新代码也将透明地读取旧消息。但是，请记住旧的消息中不会出现新的可选字段，因此您需要明确检查它们是否已设置`has_`，或者在`.proto`文件中提供合理的默认值`[default = value]`标签号后面。如果未为可选元素指定默认值，则使用特定于类型的默认值：对于字符串，默认值为空字符串。对于布尔值，默认值为`false`。对于数字类型，默认值为零。另请注意，如果添加了新的重复字段，则新代码将无法判断它是否为空（通过新代码）或从未设置（通过旧代码），因为没有`has_`标记。

### 优化
`C ++`协议缓冲区库经过了极大的优化。但是，正确使用可以进一步提高性能。以下是从库中挤出最后一滴速度的一些提示：

**尽可能重用消息对象**。消息尝试保留它们分配用于重用的任何内存，即使它们被清除。因此，如果您连续处理具有相同类型和类似结构的许多消息，则每次重新使用相同的消息对象来加载内存分配器是个好主意。但是，随着时间的推移，对象会变得臃肿，特别是如果您的消息在“形状”上有所不同，或者您偶尔构造的消息比平常大得多。

**您应该通过调用[SpaceUsed](https://developers.google.com/protocol-buffers/docs/reference/cpp/google.protobuf.message#Message.SpaceUsed.details)方法来监视消息对象的大小，并在它们变得太大时删除它们。**
您的系统内存分配器可能没有针对从多个线程分配大量小对象进行良好优化。请尝试使用[Google的tcmalloc](https://github.com/gperftools/gperftools)。

### 高级用法
协议缓冲区的用途不仅仅是简单的访问器和序列化。请务必浏览[C ++ API](https://developers.google.com/protocol-buffers/docs/reference/cpp/)参考，以了解您可以使用它们做些什么。

协议消息类提供的一个关键特性是反射。您可以迭代消息的字段并操纵它们的值，而无需针对任何特定的消息类型编写代码。使用反射的一种非常有用的方法是将协议消息转换为与其他编码（例如`XML`或`JSON`）之间的转换。更高级的反射使用可能是找到两个相同类型的消息之间的差异，或者开发一种“协议消息的正则表达式”，您可以在其中编写与某些消息内容匹配的表达式。如果您运用自己的想象力，可以将协议缓冲区应用于比您最初预期更广泛的问题！

[Message::Reflection](https://developers.google.com/protocol-buffers/docs/reference/cpp/google.protobuf.message#Message.Reflection)界面 提供反射。
