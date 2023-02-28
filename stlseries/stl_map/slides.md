---
theme: seriph
background: images/bg.png
class: text-center
highlighter: shiki
lineNumbers: true
info: |
  ## Archibate's STL course series
  Presentation slides for archibate.

  Learn more at [parallel101/course](https://github.com/parallel101/course)
drawings:
  persist: false
transition: none
css: unocss
---

# 小彭老师 STL 课程系列 之 map

让高性能数据结构惠及每一人

---

# 课程简介

😀😀😀

面向学习 C++ 标准库的童鞋。

主要学习 STL 容器。

主要基于 C++17 语言标准。

---

# 课程大纲

✨✨✨

之前几期课程的录播已经上传到比站了[^1]。

1. vector 容器初体验 & 迭代器入门 (BV1qF411T7sd)
2. 你所不知道的 set 容器 & 迭代器分类 (BV1m34y157wb)
3. string，string_view，const char * 的爱恨纠葛 (BV1ja411M7Di) 
4. 万能的 map 容器全家桶及其妙用举例 (本期)
5. 函子 functor 与 lambda 表达式知多少
6. 通过实战案例来学习 STL 算法库
7. C++ 标准输入输出流 & 字符串格式化
8. traits 技术，用户自定义迭代器与算法
9. allocator，内存管理与对象生命周期
10. C++ 异常处理机制的前世今生

[^1]: https://space.bilibili.com/263032155

---

# 课程要求

✅✅✅

课件中所示代码实验推荐环境如下：

| 要求     | ❤❤❤        | 💣💣💣       | 💩💩💩         |
|----------|------------|--------------|----------------|
| 操作系统 | Arch Linux | Ubuntu 20.04 | Wendous 10     |
| 知识储备 | 会一点 C++ | 大学 C 语言  | Java 面向对象  |
| 编译器   | GCC 9 以上 | Clang 12     | VS 2019        |
| 构建系统 | CMake 3.18 | 任意 C++ IDE | 命令行手动编译 |

❤ = 推荐，💣 = 可用，💩 = 困难

---

# 如何使用课件

🥰🥰🥰

本系列课件和源码均公布在：https://github.com/parallel101/course

例如本期的课件位于 `course/stlseries/stl_map/slides.md`。

课件基于 Slidev[^1] 开发，Markdown 格式书写，在浏览器中显示，在本地运行课件需要 Node.js：

- 运行命令 `npm install` 即可自动安装 Slidev
- 运行命令 `npm run dev` 即可运行 Slidev 服务
- 访问 http://localhost:3030 即可看到课件

如果报错找不到 `slidev` 命令，试试看 `export PATH="$PWD/node_modules/.bin:$PATH"`。

如果不想自己配置 Node.js 也可以直接以文本文件格式打开 slides.md 浏览课件。

Slidev 服务运行中对 slides.md 的修改会立刻实时显现在浏览器中。

[^1]: https://sli.dev/

---

# 如何运行案例代码

🥺🥺🥺

CMake 工程位于课件同目录的 `course/stlseries/stl_map/experiment/` 文件夹下。

其中 main.cpp 仅导入运行案例所需的头文件，具体各个案例代码分布在 slides.md 里。

如需测试具体代码，可以把 slides.md 中的案例代码粘贴到 main.cpp 的 main 函数体中进行实验。

附赠的一些实用头文件，同鞋们可以自己的项目里随意运用。

| 文件名 | 功能 |
|-|-|
| main.cpp | 把 slides.md 中你想要实验的代码粘贴到 main 函数中 |
| print.h | 内含 print 函数，支持打印绝大多数 STL 容器，方便调试 |
| cppdemangle.h | 获得类型的名字，以模板传入，详见该文件中的 Usage 介绍 |
| map_get.h | 带默认值的 map 表项查询，稍后课程中会介绍到 |

---

创建一个 map 对象：

```cpp
map<string, int> config;
```

一开始 map 默认是空的，如何插入一些初始数据？

```cpp
config["timeout"] = 985;
config["delay"] = 211;
```

---

C++11 新特性——花括号初始化列表，允许创建 map 时直接指定初始数据：

```cpp
map<string, int> config = { {"timeout", 985}, {"delay", 211} };
```

通常会换行写，一行一个键值对，看起来条理更清晰：

```cpp
map<string, int> config = {
    {"timeout", 985},
    {"delay", 211},
};
```

总结：

```cpp
map<K, V> m = {
    {k1, v1},
    {k2, v2},
    ...,
};
```

---

```cpp
map<string, int> config = {
    {"timeout", 985},
    {"delay", 211},
};
```

等号可以省略（这其实就是 map 的构造函数）：

```cpp
map<string, int> config{
    {"timeout", 985},
    {"delay", 211},
};
```

也可以先构造再赋值给 auto 变量：

```cpp
auto config = map<string, int>{
    {"timeout", 985},
    {"delay", 211},
};
```

都是等价的。

---

作为函数参数时，可以用花括号初始化列表就地构造一个 map 对象：

```cpp
void myfunc(map<string, int> config);

myfunc(map<string, int>{
    {"timeout", 985},
    {"delay", 211},
});
```

由于 `myfunc` 函数具有唯一确定的重载，要构造的参数类型 `map<string, int>` 可以省略不写：

```cpp
myfunc({
    {"timeout", 985},
    {"delay", 211},
});
```

函数这边，通常还会加上 `const &` 修饰避免不必要的拷贝。

```cpp
void myfunc(map<string, int> const &config);
```

---

如何根据键查询相应的值？

很多同学都知道 map 具有 [] 运算符重载，和一些脚本语言一样，直观易懂。

```cpp
config["timeout"] = 985;       // 把 config 中键 timeout 对应值设为 985
auto val = config["timeout"];  // 读取 config 中键 timeout 对应值
print(val);                    // 985
```

但其实用 [] 访问元素是很不安全的，下面我会做实验演示这一点。

---

沉默的 []，无言的危险：当键不存在时，会返回 0 而不会出错！

```cpp
map<string, int> config = {
    {"timeout", 985},
    {"delay", 211},
};
print(config["timeout"]); // 985
print(config["tmeout"]);  // 默默返回 0
```

```
985
0
```

当查询的键值不存在时，[] 会默默创建并返回 0。

这非常危险，例如一个简简单单的拼写错误，就会导致 map 的查询默默返回 0，你还在那里找了半天摸不着头脑，根本没发现错误原来在 map 这里。

---

爱哭爱闹的 at()，反而更讨人喜欢

```cpp
map<string, int> config = {
    {"timeout", 985},
    {"delay", 211},
};
print(config.at("timeout"));  // 985
print(config.at("tmeout"));   // 该键不存在！响亮地出错
```

```
985
terminate called after throwing an instance of 'std::out_of_range'
  what():  map::at
Aborted (core dumped)
```

有经验的老手都明白一个道理：**及时奔溃**比**容忍错误**更有利于调试。即 fail-early, fail-loudly[^1] 原则。

例如 JS 和 Lua 的 [] 访问越界不报错而是返回 undefined / nil，导致实际出错的位置在好几十行之后，无法定位到真正出错的位置，这就是为什么后来发明了错误检查更严格的 TS。

使用 at() 可以帮助你更容易定位到错误，是好事。

[^1]: https://oncodingstyle.blogspot.com/2008/10/fail-early-fail-loudly.html

---

[] 更危险的地方在于，当所查询的键值不存在时：

```cpp
map<string, int> config = {
    {"timeout", 985},
    {"delay", 211},
};
print(config);
print(config["tmeout"]);  // 有副作用！
print(config);
```

```
{"delay": 211, "timeout": 985}
0
{"delay": 211, "timeout": 985, "tmeout": 0}
```

会自动创建那个不存在的键值！

你以为你只是观察了一下 map 里的 "tmeout" 元素，却意外改变了 map 的内容，薛定谔[^1]直呼内行。

[^1]: https://baike.baidu.com/item/%E8%96%9B%E5%AE%9A%E8%B0%94%E7%9A%84%E7%8C%AB/554903

---

[] 这个成员函数没有 const 修饰，因此当 map 修饰为 const 时编译会不通过[^1]：

```cpp
const map<string, int> config = {
    {"timeout", 985},
    {"delay", 211},
};
print(config["timeout"]);  // 编译出错
```

```
/home/bate/Codes/course/stlseries/stl_map/experiment/main.cpp: In function ‘int main()’:
/home/bate/Codes/course/stlseries/stl_map/experiment/main.cpp:10:23: error: passing ‘const
std::map<std::__cxx11::basic_string<char>, int>’ as ‘this’ argument discards qualifiers
[-fpermissive]
   10 | print(config["timeout"]);
```

编译器说 discards qualifiers，意思是 map 有 const 修饰，但是 map::operator[] 没有。

把有 const 修饰的 map 作为 this 指针传入没有 const 修饰的 map::operator[] 函数，是减少了修饰（discards qualifers）。C++ 规定传参时只能增加修饰不能减少修饰：只能从 map &amp; 转换到 const map &amp; 而不能反之，所以对着一个 const map 调用非 const 的成员函数就出错了。

相比之下 at() 就可以在 const 修饰下编译通过，可见 at() 才是读取元素的正道。

[^1]: https://blog.csdn.net/benobug/article/details/104903314

---

既然 [] 这么危险，为什么还要存在呢？

```cpp
map<string, int> config = {
    {"delay", 211},
};
config.at("timeout") = 985;  // 键值不存在，报错！
config["timeout"] = 985;     // 成功创建并写入 985
```

因为当我们写入一个本不存在的键值的时候，恰恰需要他的“自动创建”这一特性。

总结：读取时应该用 at() 更安全，写入时才需要 []。

---

因此，我的建议是：

- 读取元素时，统一用 at()
- 写入元素时，统一用 []

```cpp
auto val = m.at("key");
m["key"] = val;
```

不要用混了哦！

为什么其他语言比如 Python，只有一个 [] 就行了呢？而 C++ 需要两个？

- 因为 Python 会检测 [] 位于等号左侧还是右侧，根据情况分别调用 `__getitem__` 或者 `__setitem__`。
- C++ 编译器没有这个特殊检测，C++ 的 [] 只是返回了个引用，并不知道 [] 函数返回以后，你是拿这个引用写入还是读取。为了保险起见他默认你是写入，所以先帮你创建了元素，返回这个元素的引用，让你写入。
- 而 Python 的引用是不能用 = 覆盖原值的，那样只会让变量指向新的引用，只能用 .func() 引用成员函数或者 += 才能就地修改原变量，这是 Python 这类脚本语言和 C++ 最本质的不同。
- 总而言之，我们用 C++ 的 map 读取元素时，需要显式地用 at() 告诉编译器我是打算读取。

---

[] 也有其他的意义

除了写入元素需要用 [] 以外，还有一些案例中合理运用 [] 会非常的方便。

[] 的效果：当所查询的键值不存在时，会调用默认构造函数创建一个元素。

- 对于 int, float 等数值类型而言，默认值是 0。
- 对于指针（包括智能指针）而言，默认值是 nullptr。
- 对于 string 而言，默认值是空字符串 ""。
- 对于 vector 而言，默认值是空数组 {}。
- 对于自定义类而言，调用你写的默认构造函数，如果没有，则每个成员都取默认值。

---

[] 运用举例：出现次数统计

```cpp
vector<string> input = {"hello", "world", "hello"};
map<string, int> counter;
for (auto const &key: input) {
    counter[key]++;
}
print(counter);
```

```
{"hello": 2, "world": 1}
```

---
layout: two-cols-header
---

::left::

<center>

活用 [] 自动创建 0 元素的特性

</center>

```cpp {3}
map<string, int> counter;
for (auto const &key: input) {
    counter[key]++;
}
```

::right::

<center>

古板的写法

</center>

```cpp {3-7}
map<string, int> counter;
for (auto const &key: input) {
    if (!counter.count(key)) {
        counter[key] = 1;
    } else {
        counter[key] = counter.at(key) + 1;
    }
}
```

---

[] 运用举例：归类

```cpp
vector<string> input = {"happy", "world", "hello", "weak", "strong"};
map<char, vector<string>> categories;
for (auto const &str: input) {
    char key = str[0];
    categories[key].push_back(str);
}
print(categories);
```

```
{'h': {"happy", "hello"}, 'w': {"world", "weak"}, 's': {"strong"}}
```

---
layout: two-cols-header
---

::left::

<center>

活用 [] 自动创建"默认值"元素的特性

</center>

```cpp {4}
map<char, vector<string>> categories;
for (auto const &str: input) {
    char key = str[0];
    categories[key].push_back(str);
}
print(categories);
```

::right::

<center>

古板的写法

</center>

```cpp {4-8}
map<char, vector<string>> categories;
for (auto const &str: input) {
    char key = str[0];
    if (!categories.count(key)) {
        categories[key] = {str};
    } else {
        categories[key].push_back(str);
    }
}
```

---
layout: center
---

![Elegence](https://pica.zhimg.com/50/v2-f2560f634b1e09f81522f29f363827f7_720w.jpg)

---

STL 容器的元素类型都可以通过成员 `value_type` 查询，常用于泛型编程（又称元编程）。

```cpp
set<int>::value_type      // int
vector<int>::value_type   // int
string::value_type        // char
```

在本课程的案例代码中，附赠一份 "cppdemangle.h"，可以实现根据指定的类型查询类型名称并打印出来。

跨平台，支持 MSVC，Clang，GCC 三大编译器，例如：

```cpp
int i;
print(cppdemangle<decltype(std::move(i))>());
print(cppdemangle<std::string>());
print(cppdemangle<std::wstring::value_type>());
```

在我的 GCC 12.2.1 上得到：

```
"int &&"
"std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> >"
"wchar_t"
```

---

问题: map 真正的元素类型究竟是什么？其具有三个成员类型[^1]：

- 元素类型：`value_type`
- 键类型：`key_type`
- 值类型：`mapped_type`

用 cppdemangle 做实验，看看这些成员类型具体是什么吧：

```cpp
map<int, float>::value_type   // pair<const int, float>
map<int, float>::key_type     // int
map<int, float>::mapped_type  // float
```

结论：`map<K, V>` 的元素类型是 `pair<const K, V>` 而不是 `V`。

[^1]: https://blog.csdn.net/janeqi1987/article/details/100049597

---

`pair<const K, V>` ——为什么 K 要加 const？

上期 set 课说过，set 内部采用红黑树数据结构保持有序，这样才能实现在 $O(\log N)$ 时间内高效查找。

键值改变的话会需要重新排序，如果只修改键值而不重新排序，会破坏有序性，导致二分查找结果错误！
所以 set 只有不可变迭代器（const_iterator），不允许修改元素的值。

map 和 set 一样也是红黑树，不同在于：map 只有键 K 的部分会参与排序，V 是个旁观者，随便修改也没关系。

所以 map 有可变迭代器，只是在其 value_type 中给 K 加上了 const 修饰：不允许修改 K，但可以修改 V。

如果你确实需要修改键值，那么请先把这个键删了，然后再以同样的 V 重新插入一遍，保证红黑树的有序。

---

```cpp
iterator find(K const &k);
const_iterator find(K const &k) const;
```

m.find(key) 函数，根据指定的键 key 查找元素[^1]。

- 成功找到，则返回指向找到元素的迭代器
- 找不到，则返回 m.end()

第二个版本的原型作用是：如果 map 本身有 const 修饰，则返回的也是 const 迭代器。

[^1]: https://en.cppreference.com/w/cpp/container/map/find

---

检查过不是 m.end()，以确认成功找到后，就可以通过 * 运算符解引用获取迭代器指向的值：

```cpp
map<string, int> m = {
    {"fuck", 985},
};
auto it = m.find("fuck");  // 寻找 K 为 "fuck" 的元素
if (it != m.end()) {
    auto kv = *it;     // 解引用得到 K-V 对
    print(kv);         // {"fuck", 985}
    print(kv.first);   // "fuck"
    print(kv.second);  // 985
} else {
    print("找不到 fuck！");
}
```

---

检查过不是 m.end()，以确认成功找到后，就可以通过 * 运算符解引用获取迭代器指向的值：

```cpp
map<string, int> m = {
    {"fuck", 985},
};
auto it = m.find("fuck");  // 寻找 K 为 "fuck" 的元素
if (it != m.end()) {
    auto kv = *it;     // 解引用得到 K-V 对
    print(kv);         // {"fuck", 985}
    print(kv.first);   // "fuck"
    print(kv.second);  // 985
} else {
    print("找不到 fuck！");
}
```

---

注意 `*it` 解引用得到的是 K-V 对，需要 `(*it).second` 才能获取单独的 V。

好在 C 语言就有 `->` 运算符作为语法糖，我们可以简写成 `it->second`，与 `(*it).second` 等价。

```cpp
map<string, int> m = {
    {"fuck", 985},
};
auto it = m.find("fuck");   // 寻找 K 为 "fuck" 的元素
if (it != m.end()) {
    print(it->second);      // 迭代器有效，可以直接获得值部分 985
} else {
    print("找不到 fuck！");  // 这个分支里不得用 * 和 -> 运算符解引用 it
}
```

大多数情况下我们查询只需要获取值 V 的部分就行了，直接 `it->second` 就可以了✅

> 注意：find 找不到键时，会返回 `m.end()`，这是个无效迭代器，只作为标识符使用（类比 Python 中的 find 有时会返回 -1）。
>
> 没有确认 `it != m.end()` 前，不可以访问 `it->second`！那相当于解引用一个空指针，会造成 segfault（更专业一点说是 UB）。
>
> 记住，一定要在 `it != m.end()` 的分支里才能访问 `it->second` 哦！你得先检查过饭碗里没有老鼠💩之后，才能安心吃饭！
>
> 如果你想让老妈（标准库）自动帮你检查有没有老鼠💩，那就用会自动报错的 at（类比 Python 中的 index 找不到直接报错）。
>
> 之所以用 find，是因为有时饭碗里出老鼠💩，是计划的一部分！例如当有老鼠💩时你可以改吃别的零食。而 at 这个良心老妈呢？一发现老鼠💩就拖着你去警察局报案，零食（默认值）也不让你吃了。今日行程全部取消，维权（异常处理，找上层 try-catch 块）设为第一要务。

---

带默认值的查询

众所周知，Python 中的 dict 有一个 m.get(key, defl) 的功能，效果是当 key 不存在时，返回 defl 这个默认值代替 m[key]，而 C++ 的 map 却没有，只能用一套组合拳代替：

```cpp
m.count(key) ? m.at(key) : defl
```

但上面这样写是比较低效的，相当于查询了 map 两遍，at 里还额外做了一次多余的异常判断。

正常来说是用通用 find 去找，返回一个迭代器，然后判断是不是 end() 决定要不要采用默认值。

```cpp
auto it = m.find(key);
return it == m.end() ? it->second : defl;
```

> 饭碗里发现了老鼠💩？别急着报警，这也在我的预料之中：启用 B 计划，改吃 defl 这款美味零食即可！
>
> 如果是良心老妈 at，就直接启用 C 计划：![Plan C](images/planc.png) 抛出异常然后奔溃了，虽然这很方便我们程序员调试。

---

由于自带默认值的查询这一功能实在是太常用了，为了把这个操作浓缩到一行，我建议同学们封装成函数放到自己的项目公共头文件（一般是 utils.h 之类的名称）里方便以后使用：

```cpp
template <class M>
typename M::mapped_type map_get
( M const &m
, typename M::key_type const &key
, typename M::mapped_type const &defl
) {
  typename M::const_iterator it = m.find(key);
  if (it != m.end()) {
    return it->second;
  } else {
    return defl;
  }
}
```

```cpp
int val = map_get(config, "timeout", -1);  // 如果配置文件里不指定，则默认 timeout 为 -1
```

---

这样还不够优雅，我们还可以更优雅地运用 C++17 的函数式容器 optional：

```cpp
template <class M>
std::optional<typename M::mapped_type> map_get
( M const &m
, typename M::key_type const &key
) {
  typename M::const_iterator it = m.find(key);
  if (it != m.end()) {
    return it->second;
  } else {
    return std::nullopt;
  }
}
```

当找不到时就返回 nullopt，找到就返回含有值的 optional。

注：本段代码已附在案例代码库的 "map_get.h" 文件中，等课后可以去 GitHub 下载。

---

调用者可以自行运用 optional 的 value_or 函数[^1]指定找不到时采用的默认值：

```cpp
int val = map_get(config, "timeout").value_or(-1);
```

如果要实现 at 同样的找不到就自动报错功能，那就改用 value 函数：

```cpp
int val = map_get(config, "timeout").value();
```

以上是典型的函数式编程范式 (FP)，C++20 还引入了更多这样的玩意[^2]，等有空会专门开节课为大家一一介绍。

```cpp
auto even = [] (int i) { return 0 == i % 2; };
auto square = [] (int i) { return i * i; };
for (int i: std::views::iota(0, 6)
          | std::views::filter(even)
          | std::views::transform(square))
    print(i);  // 0 4 16
```

[^1]: https://en.cppreference.com/w/cpp/utility/optional/value_or
[^2]: https://en.cppreference.com/w/cpp/ranges/filter_view

---

map 的成员 insert 函数原型如下[^1]：

```cpp
pair<iterator, bool> insert(pair<const K, V> const &kv);
pair<iterator, bool> insert(pair<const K, V> &&kv);
```

他的参数类型就是刚刚介绍的 `value_type`，也就是 `pair<const K, V>`。

pair 是一个 STL 中常见的模板类型，`pair<K, V>` 有两个成员变量：

- first：V 类型，表示要插入元素的键
- second：K 类型，表示要插入元素的值

我称之为"键值对"。

[^1]: https://en.cppreference.com/w/cpp/container/map/insert

---

试着用 insert 插入键值对：

```cpp
map<string, int> m;
pair<string, int> p;
p.first = "fuck";  // 键
p.second = 985;    // 值
m.insert(p);  // pair<string, int> 可以隐式转换为 insert 参数所需的 pair<const string, int>
print(m);
```

结果：

```
{"fuck": 985}
```

---

简化 insert

<!-- <v-clicks> -->

1. 直接使用 pair 的构造函数，初始化 first 和 second

```cpp
pair<string, int> p("fuck", 985);
m.insert(p);
```

2. 不用创建一个临时变量，pair 表达式直接作为 insert 函数的参数

```cpp
m.insert(pair<string, int>("fuck", 985));
```

2. 可以用 `std::make_pair` 这个函数，自动帮你推导模板参数类型，省略 `<string, int>`

```cpp
m.insert(make_pair("fuck", 985));  // 虽然会推导为 pair<const char *, int> 但还是能隐式转换为 pair<const string, int>
```

3. 由于 insert 函数原型已知参数类型，可以直接用 C++11 的花括号初始化列表 {...}，无需指定类型

```cpp
m.insert({"fuck", 985});           // ✅
```

<!-- </v-clicks> -->

---
layout: two-cols-header
---

因此，insert 的最佳用法是：

```cpp
map<K, V> m;
m.insert({"key", "val"});
```

insert 插入和 [] 写入的异同：

- 同：当键 K 不存在时，insert 和 [] 都会创建键值对。
- 异：当键 K 已经存在时，insert 不会覆盖，默默离开；而 [] 会覆盖旧的值。

例子：

::left::

```cpp
map<string, string> m;
m.insert({"key", "old"});
m.insert({"key", "new"});  // 插入失败，默默放弃不出错
print(m);
```

```
{"key": "old"}
```

::right::

```cpp
map<string, string> m;
m["key"] = "old";
m["key"] = "new";        // 已经存在？我踏马强行覆盖！
print(m);
```

```
{"key": "new"}
```

---

insert 的返回值是 `pair<iterator, bool>` 类型，<del>STL 的尿性：在需要一次性返回两个值时喜欢用 pair</del>。

这又是一个 pair 类型，其具有两个成员：

- first：iterator 类型，是个迭代器
- second：bool 类型，表示插入成功与否，如果发生键冲突则为 false

其中 first 这个迭代器指向的是：

- 如果插入成功（second 为 true），指向刚刚成功插入的元素位置
- 如果插入失败（second 为 false），说明已经有相同的键 K 存在，发生了键冲突，指向已经存在的那个元素

---

其实 insert 返回的 first 迭代器等价于插入以后再重新用 find 找到刚刚插入的那个键，只是效率更高：

```cpp
auto it = m.insert({k, v}).first;  // 高效，只需遍历一次
```

```cpp
m.insert({k, v});     // 插入完就忘事了
auto it = m.find(k);  // 重新遍历第二次，但结果一样
```

参考 C 编程网[^1]对 insert 返回值的解释：

> 当该方法将新键值对成功添加到容器中时，返回的迭代器指向新添加的键值对；
>
> 反之，如果添加失败，该迭代器指向的是容器中和要添加键值对键相同的那个键值对。

[^1]: http://c.biancheng.net/view/7241.html

---

可以用 insert 返回的 second 判断插入多次是否成功：

```cpp
map<string, string> m;
print(m.insert({"key", "old"}).second);  // true
print(m.insert({"key", "new"}).second);  // false
m.erase("key");     // 把原来的 {"key", "old"} 删了
print(m.insert({"key", "new"}).second);  // true
```

也可以用 structual-binding 语法拆解他返回的 `pair<iterator, bool>`：

```cpp
map<string, int> counter;
auto [it, success] = counter.insert("key", 1);  // 直接用
if (!success) {  // 如果已经存在，则修改其值+1
    it->second = it->second + 1;
} else {  // 如果不存在，则打印以下信息
    print("created a new entry!");
}
```

以上这一长串代码和之前“优雅”的计数 [] 等价：

```cpp
counter["key"]++;
```

---

在 C++17 中，[] 写入有了个更高效的替代品 insert_or_assign[^1]：

```cpp
pair<iterator, bool> insert_or_assign(K const &k, V v);
pair<iterator, bool> insert_or_assign(K &&k, V v);
```

正如他名字的含义，“插入或者写入”：

- 如果 K 不存在则创建（插入）
- 如果 K 已经存在则覆盖（写入）

用法如下：

```cpp
m.insert_or_assign("key", "new");  // 与 insert 不同，他不需要 {...}，他的参数就是两个单独的 K 和 V
```

返回值依旧是 `pair<iterator, bool>`。由于这函数在键冲突时会覆盖，按理说是必定成功了，因此这个 bool 的含义从“是否插入成功”变为“是否创建了元素”，如果是创建的新元素返回true，如果覆盖了旧元素返回false。

[^1]: https://en.cppreference.com/w/cpp/container/map/insert_or_assign

---

看来 insert_or_assign 和 [] 的效果完全相同！都是在键值冲突时覆盖旧值。

既然 [] 已经可以做到同样的效果，为什么还要发明个 insert_or_assign 呢？

insert_or_assign 的优点是**不需要调用默认构造函数**，可以提升性能。

其应用场景有以下三种情况：

- ⏱ 您特别在乎性能
- ❌ 有时 V 类型没有默认构造函数，用 [] 编译器会报错
- 🥵 强迫症发作

否则用 [] 写入也是没问题的。

insert_or_assign 能取代 [] 的岗位仅限于纯写入，之前 `counter[key]++` 这种“优雅”写法依然是需要用 [] 的。

---
layout: two-cols-header
---

创建新键时，insert_or_assign 更高效。

::left::

```cpp
map<string, string> m;
m["key"] = "old";
m["key"] = "new";
print(m);
```

```
{"key": "new"}
```

覆盖旧键时，使用 [] 造成的开销：

- 调用移动赋值函数 `V &operator=(V &&)`

创建新键时，使用 [] 造成的开销：

- 调用默认构造函数 `V()`
- 调用移动赋值函数 `V &operator=(V &&)`

::right::

```cpp
map<string, string> m;
m.insert_or_assign("key", "old");
m.insert_or_assign("key", "new");
print(m);
```

```
{"key": "new"}
```

覆盖旧键时，使用 insert_or_assign 造成的开销：

- 调用移动赋值函数 `V &operator=(V &&)`

创建新键时，使用 insert_or_assign 造成的开销：

- 调用移动构造函数 `V(V &&)`

---

如果你有性能强迫症，并且是 C++17 标准：

- 写入用 insert_or_assign
- 读取用 at

如果没有性能强迫症，或者你的编译器不支持 C++17 标准：

- 写入用 []
- 读取用 at

最后，如果你是还原论者，只需要 find 和 insert 函数就是完备的了，别的函数都不用去记。所有 at、[]、insert_or_assign 之类的操作都可以通过 find 和 insert 的组合拳实现，例如刚刚我们自定义的 map_get。

---

除了刚刚介绍的 `insert(pair<K, V>)` 外，insert 还有一个特殊的版本，用于批量插入一系列元素。

```cpp
template <class InputIt>
void insert(InputIt beg, InputIt end);
```

参数[^1]是两个迭代器 beg 和 end，组成一个区间，之间是你要插入的数据。

该区间可以是任何其他容器的 begin() 和 end() 迭代器——那会把该容器中所有的元素都插入到本 map 中去。

用法举例：把 vector 中的键值对批量插入 map：

```cpp
vector<pair<string, int>> kvs = {
    {"timeout", 985},
    {"delay", 211},
};
map<string, int> config;
config.insert(kvs.begin(), kvs.end());
print(config);  // {"delay": 211, "timeout": 985}
```

[^1]: 轶事：在标准库文档里批量插入版 insert 函数的 beg 和 end 这两个参数名为 first 和 last，但与 pair 的 first 并没有任何关系，只是为了防止变量名和 begin() 和 end() 成员函数发生命名冲突。为了防止同学们与 pair 的 first 混淆所以才改成 beg 和 end 做参数名。

---

C++11 还引入了一个以初始化列表（initializer_list）为参数的版本：

```cpp
void insert(initializer_list<pair<K, V>> ilist);
```

用法和 map 的构造函数一样，还是用花括号列表：

```cpp
map<string, int> m = {  // 初始化时就插入两个元素
    {"answer", 42},
    {"timeout", 7},
};
m.insert({              // 批量再插入两个新元素
    {"timeout", 985},   // "timeout" 发生键冲突，根据 insert 的尿性，不会覆盖
    {"delay", 211},
});
```

```
{"answer": 42, "delay": 211, "timeout": 7}
```

注意看这里还是和 insert({k, v}) 一样的尿性，重复的键 "timeout" 没有被覆盖，依旧了保留原值。

---

小彭老师锐评批量 insert

```cpp
m.insert({
    {"timeout", 985},
    {"delay", 211},
});
```

总之这玩意和分别调用两次 insert 等价，也并不高效多少：

```cpp
m.insert({"timeout", 985});
m.insert({"delay", 211});
```

如果需要覆盖原值的批量插入，还是乖乖用 for 循环调用 [] 或 insert_or_assign 吧。


---

```cpp
iterator insert(const_iterator pos, pair<K, V> const &kv);
```

这又是 insert 函数的一个重载版，增加了 pos 参数提示插入位置，我的评价是[意义不明的存在](https://www.bilibili.com/video/av9609348?p=1)，官方文档称[^1]：

> Inserts value in the position as close as possible to the position just prior to pos.
>
> 把元素（键值对）插入到位于 pos 之前，又离 pos 尽可能近的地方。

然而 map 作为红黑树应该始终保持有序，插入位置可以由 K 唯一确定，为啥还要提示？C 编程网的说法是[^2]：

> （带提示的 insert 版本）中传入的迭代器，仅是给 map 容器提供一个建议，并不一定会被容器采纳。该迭代器表明将新键值对添加到容器中的位置。需要注意的是，新键值对添加到容器中的位置，并不是此迭代器说了算，最终仍取决于该键值对的键的值。

也就是说这玩意还不一定管用，只是提示性质的（和 mmap 函数的 start 参数很像，你可以指定，但只是个提示，指定了不一定有什么软用，具体什么地址还是操作系统说了算，他从返回值里给你的地址才是正确答案）。

估计又是性能强迫症发作了吧，例如已知指向 "key" 的迭代器，想要插入 "kea"，那么指定指向 "key" 的迭代器就会让 find 能更容易定位到 "kea" 要插入的位置？如果你知道准确原因，欢迎在评论区指出。

[^1]: https://en.cppreference.com/w/cpp/container/map/insert
[^2]: http://c.biancheng.net/view/7241.html

---

insert 的究极分奴版：emplace

insert 的宇宙无敌分奴版：emplace_hint

insert 的托马斯黄金大回旋分奴版：try_emplace

insert 的炫彩中二摇摆混沌大魔王分奴版：带插入位置提示的 try_emplace

---

find 和 insert 都学完了，最后，让我们来学习删除元素用的 erase 函数，其原型如下[^1]：

```cpp
iterator erase(K const &key);
iterator erase(iterator pos);
iterator erase(iterator beg, iterator end);
```

[^1]: https://en.cppreference.com/w/cpp/container/map/erase

---

erase 运用举例：删除一个元素

```cpp
map<string, string> msg = {
    {"hello", "world"},
    {"fuck", "rust"},
};
print(msg);
msg.erase("fuck");
print(msg);
```

```
{"fuck": "rust", "hello": "world"}
{"hello": "world"}
```

---

# 增删

| 写法 | 效果 | 版本 | 推荐 |
|------|------|------|------|
| `m.insert(make_pair(key, val))` | 插入但不覆盖 | C++98 | 💩 |
| `m.insert({key, val})` | 插入但不覆盖 | C++11 | ❤ |
| `m.emplace(key, val)` | 插入但不覆盖 | C++11 | 💩 |
| `m.try_emplace(key, valcons...)` | 插入但不覆盖 | C++17 | 💣 |
| `m.insert_or_assign(key, val)` | 插入或覆盖 | C++17 | ❤ |
| `m[key] = val` | 插入或覆盖 | C++98 | 💣 |
| `m.erase(key)` | 删除指定元素 | C++98 | ❤ |

---

# 改查

| 写法 | 效果 | 版本 | 推荐 |
|------|------|------|------|
| `m.at(key)` | 找不到则出错，找到则返回引用 | C++98 | ❤ |
| `m[key]` | 找不到则自动创建`0`值，返回引用 | C++98 | 💣 |
| `myutils::map_get(m, key, defl)` | 找不到则返回默认值 | C++98 | ❤ |
| `m.find(key) == m.end()` | 检查键 `key` 是否存在 | C++98 | 💣 |
| `m.count(key)` | 检查键 `key` 是否存在 | C++98 | ❤ |
| `m.contains(key)` | 检查键 `key` 是否存在 | C++20 | 💩 |

---

# 初始化

| 写法 | 效果 | 版本 | 推荐 |
|------|------|------|------|
| `map<K, V> m = {{k1, v1}, {k2, v2}}` | 初始化为一系列键值对 | C++11 | ❤ |
| `auto m = map<K, V>{{k1, v1}, {k2, v2}}` | 初始化为一系列键值对 | C++11 | 💩 |
| `func({{k1, v1}, {k2, v2}})` | 给函数参数传入一个 map | C++11 | ❤ |
| `m = {{k1, v1}, {k2, v2}}` | 重置为一系列键值对 | C++11 | ❤ |
| `m.assign({{k1, v1}, {k2, v2}})` | 重置为一系列键值对 | C++11 | 💩 |
| `m.clear()` | 清空所有表项 | C++98 | ❤ |
| `m = {}` | 清空所有表项 | C++11 | 💣 |

---

map 的迭代器失效问题

---

一边遍历一边删除部分元素（错误示范）

```cpp
map<string, string> msg = {
    {"hello", "world"},
    {"fucker", "rust"},
    {"fucking", "java"},
    {"good", "job"},
};
for (auto const &[k, v]: msg) {
    if (k.find("fuck") == 0) {
        msg.erase(k);  // 遍历过程中动态删除元素，会导致正在遍历中的迭代器失效，奔溃
    }
}
print(msg);
```

```
Segmentation fault (core dumped)
```

---

一边遍历一边删除部分元素（正解[^1]）

```cpp
map<string, string> msg = {
    {"hello", "world"},
    {"fucker", "rust"},
    {"fucking", "java"},
    {"good", "job"},
};
for (auto it = m.begin(); it != m.end(); ) {  // 没有 ++it
    auto const &[k, v] = *it;
    if (k.find("fuck") == 0) {
        it = msg.erase(it);
    } else {
        ++it;
    }
}
print(msg);
```

```
{"good": "job", "hello": "world"}
```

[^1]: https://stackoverflow.com/questions/8234779/how-to-remove-from-a-map-while-iterating-it

---

批量删除符合条件的元素（C++20[^1]）

```cpp
map<string, string> msg = {
    {"hello", "world"},
    {"fucker", "rust"},
    {"fucking", "java"},
    {"good", "job"},
};
std::erase_if(msg, [&] (auto const &kv) {
    auto &[k, v] = kv;
    return k.starts_with("fuck");
});
print(msg);
```

```
{"good": "job", "hello": "world"}
```

[^1]: https://www.apiref.com/cpp-zh/cpp/container/map/erase_if.html
