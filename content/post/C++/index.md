---
title:		CS 106L
subtitle:	CS 106L
summary:	notes of 106L Standard C++ Programming
date:		2021-11-07
lastmod:	2023-08-31
author:		shaopu
draft: 		false
image:		  
  focal_point: ''
  placement: 2
  preview_only: true
Type:		docs
tags:
    - course
    - C++

categories:
    - CS course notes
    - Programming Language
---



本篇文章是我学习*CS 106L*课程的笔记记录，但本文中大部分内容来自于学习过程中查阅的各种*blog*以及*StackOverflow*和**C++ **的标准文档，作为个人日后查询、补充和深入学习的手册使用。此外，我将2019秋季Stanford的CS 106L作业发布在我的[Github仓库](https://github.com/SongShaopu1998/Stanford-CS-106L).

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202111091826467.png" alt="image-20211109182644300" style="zoom:50%;" />

## stream

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202111091455815.png" alt="image-20211109145516729" style="zoom:50%;" />

### stringstream

以下内容来自官方文档：

> Stream class to operate on strings.
>
> Objects of this class use a *[string buffer](https://www.cplusplus.com/stringbuf)* that contains a sequence of characters. This sequence of characters can be accessed directly as a [string](https://www.cplusplus.com/string) object, using member [str](https://www.cplusplus.com/stringstream::str).
>
> Characters can be inserted and/or extracted from the stream using any operation allowed on both *[input](https://www.cplusplus.com/istream)* and *[output](https://www.cplusplus.com/ostream)* streams.
>
> **stream buffer**: "A stream buffer is an object in charge of performing the reading and writing operations of the *stream* object it is associated with: the stream delegates all such operations to its associated *stream buffer* object, which is **an intermediary between the *stream* and its *controlled input and output sequences***."
>
> <img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202111100150967.png" alt="image-20211110015005910" style="zoom:50%;" />
>
> <img src="https://upload.cppreference.com/mwiki/images/7/75/std-streambuf.svg" alt="std-streambuf.svg"  />

> **All *stream* objects, no matter whether buffered or unbuffered, have an associated *stream buffer***: Some *stream buffer* types may then be set to either use an intermediate *buffer* or not.

`stringstream`类继承自几个`ios`基础类：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202111081327496.png" alt="image-20211108132713194" style="zoom:50%;" />

`stringbuf`继承自`streambuf`:

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202111090021218.png" alt="image-20211109002123175" style="zoom:50%;" />

#### read & write

该类在实际使用中，常常用于分割字符（按空格）。它将`string`存放到一个`string buffer`（继承自`streambuf`）中：

> Internally, its [iostream](https://www.cplusplus.com/iostream::iostream) base constructor is passed a pointer to a [stringbuf](https://www.cplusplus.com/stringbuf) object constructed with str and which as arguments.

我们可以通过参数`stringstream::ate`指定指针位置在末端；如果我们使用的是`istringstream`，那么可以使用`stringstream::bin`的方式以二进制读取。

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202111081336407.png" alt="image-20211108133647355" style="zoom:33%;" />

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202111081337944.png" alt="image-20211108133704904" style="zoom:33%;" />

每当我们使用`stringstream object`向该`string buffer`中读入数据时，指针会不断向后做相应移动，就像这样：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202111081338230.png" alt="image-20211108133834185" style="zoom:33%;" />

我们还可以使用`.str()`方法将存放于`string buffer`中的内容读取成字符串。

#### about ">>"

我们可以指定类型，之后使用`>>`操作符将内容读入指定类型的变量中。在C++官方[文档页面](https://www.cplusplus.com/reference/istream/istream/operator%3E%3E/)上，对这一操作进行了详细的描述，该运算符被多次重载用以接受以下三种参数：

- `arithmetic types`
- `stream buffers`
- `manipulators`

具体的操作可以参见上边的链接，总之就是要借助于继承自`istream`的`sentry`类来辅助处理输入流（接受一个`stream object parameter`），**它首先会检测当前`internal error flags`的状态，如果为`good`，则继续进行，否则不会对流做任何操作，这一点需要注意。**

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202111100144760.png" alt="image-20211110014417695" style="zoom:50%;" />

在运算过程中，程序通过设定`ios_base::iostate`的值来告诉目前`stream`的情况--还提供了相应的函数帮助我们进行判断，这几个常量被称为`internal error state flags`，具体见下表:

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202111081426342.png" alt="image-20211108142645284" style="zoom:50%;" />

我们可以通过调用方法`ios::rdstate() const`来获取当前`internal error state flags`的值，或者使用`basic_ios::setstate()`来修改当前标志位的值。需要注意的是，已经设置了的`bitflag`是不会自动清除的(`sticky!`)，需要通过调用`basic_ios::clear(iostate state = goodbit)`来替换当前状态。

> 需要注意的是，`End-Of-File`并不是什么存在于文件末尾的字符，`EOF`是定义于标准库内的一个宏(`macro`):
>
> ```c++
> #define EOF (-1)
> ```
>
> 当文件或者字符串被读取到末尾时，**读取函数会返回`-1`**，所以我们也说`eof reached`.当文件（字符串）末尾到达后，程序设置`eofbit`，但是需要注意此时`failbit`与`badbit`也可能同时被设置.
>
> > "Reaching the *End-of-File* sets the eofbit. But note that operations that reach the *End-of-File* may also set the failbit if this makes them fail (thus setting both eofbit and failbit)."

那么问题来了，`>>`会在什么时候设置这几个`bitflag`呢？官方[文档](https://www.cplusplus.com/reference/istream/istream/operator%3E%3E/)也给出了详细说明（啥都没发生就是`goodbit`）:

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202111081445541.png" alt="image-20211108144549477" style="zoom:50%;" />

当内置错误标志为`fail`时，我们也可以利用C++库的从`istream object&`到`bool`的隐式转换`ss >> ch`作为`!ss.fail()`的替代，之后的`getline()`函数也可以利用`null pointer->bool`的转换:

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202111102311395.png" alt="image-20211110231100308" style="zoom:50%;" />

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202111102311243.png" alt="image-20211110231116176" style="zoom:50%;" />

在本课的`stringToInteger`函数中，我们见到如下使用方式：

```c++
int stringToInteger(const string& str) {
    istringstream iss(str);
    int result;
    iss >> result;
    cout << "result: " << result << endl;
    if (iss.fail())
        throw domain_error("error1");
    char remain;
    iss >> remain;
    cout << "remain: " << remain << endl;
    if (!iss.fail())
        throw domain_error("error2");
    return result;
}

// case 1:
// result: 5, remain: l
// error2
stringToInetger("5lol");
// case 2:
// result: 0
// error1
stringToInteger("lol");
```

为什么会出现如上的情况？这要考虑到`>>`第一种重载模式，即用来读取算术类型的内置操作：

> "Extracts and parses characters sequentially from the stream to interpret them as the representation of a value of the proper type, which is stored as the value of val.
> Internally, the function accesses the input sequence by first constructing a [sentry](https://www.cplusplus.com/istream::sentry) object (with noskipws set to `false`). Then (if [good](https://www.cplusplus.com/ios::good)), it calls [num_get::get](https://www.cplusplus.com/num_get::get) (using the stream's *[selected locale](https://www.cplusplus.com/ios_base::getloc)*) to perform both the extraction and the parsing operations, adjusting the stream's *[internal state flags](https://www.cplusplus.com/basic_ios::rdstate)* accordingly. Finally, it destroys the [sentry](https://www.cplusplus.com/istream::sentry) object before returning."

注意到此时在内部是使用了`num_get::get`方法对数字进行读取的，如果成功读取，把结果存储在参数`val`中，并利用此函数更新`internal error flags`(利用传递给`get`方法的`ios_base::iostate& err`参数)：

> - "The function stops reading characters from the sequence as soon as one character cannot be part of a valid numerical expression (or end is reached). The next character in the sequence is pointed by the iterator returned by the function."
>
> - "*Return value*: The next character in the sequence right after where the extraction operation ended."
> - <img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202111081615482.png" alt="image-20211108161553419" style="zoom:50%;" />

当尝试使用`int type`读入字符串`"5lol"`时，首先`num_get::get`方法读取数字5，由于下一个`char`不是`part of valid numerical expression`，所以他立即停止读取，此时`val`的数值为`5`. 指针指向`5`的下一个字符`l`.

当尝试使用`int type`读入字符串`"lol"`时，首先`num_get::get`方法读取字符`l`，发现` The sequence did not match the expected format`，所以他立即停止读取，此时`val`的数值为`0`. 指针仍然指向头部，并设置了一个`failbit`，于是当我们返回到`>>`的处理过程中时，就会显示`fail()`了，这也与`>>`产生`failbit`的条件吻合，因为此时相当于没有新的字符被成功提取，同时`sb`此时也是一个`null pointer`.(因为我们用的是`int type`处理)。并且`result`的输出值为`0`，这也是`num_get::get`方法中`val`被存储的值。

> 如果我们把第二次读取的类型换成`double`，并尝试读入字符串“5.2”时，不会出现任何问题，因为在第一次读取`5`之后，序列中余下的`.2`被程序理解为`0.2`，也是我们第二次读取输出的结果。

#### white space separating？

为什么使用`stringstream`之后`istream >> string`读取字符串能够自动以空白字符作为分割？

这是因为`string`类本身也对`extraction operator`进行了重载：

```c++
istream& operator>> (istream& is, string& str);
```

按照*StackOverflow*的说法，该函数的实现类似于**C**中的*scanf %s*，他会持续读取直到遇到空白字符（包括空格、tab等）下一次从空白字符开始读取（但是默认会`skipws`）。在官方[文档](https://www.cplusplus.com/reference/string/string/operator%3E%3E/)中，我们找到了依据：

> Notice that the [istream](https://www.cplusplus.com/istream) extraction operations use whitespaces as separators; Therefore, this operation will only extract what can be considered a word from the stream. To extract entire lines of text, see the [string](https://www.cplusplus.com/string) overload of global function [getline](https://www.cplusplus.com/string:getline).
>
> <img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202111100156520.png" alt="image-20211110015649451" style="zoom:50%;" />

也就是说如果我们使用`>>`将数据读入字符串，那么他会尝试着一个单词一个单词地读取。该过程的实现也是借助于一个由指针`sb`指向的`streambuf object`，每次我们都在向该`streambuf`写入数据，在向`streambuf`写入完毕之后，将`streambuf object`中的内容写给`string`对象。**所有的stream对象，都有一个与之关联的`streambuf object`**.

#### Again--About Whitespaces!

下边来看这样两个例子：

1. ```c++
   string a = " 12 3";
   stringstream iss(a);
   char b, c;
   iss >> std::noskipws >> b >> c;
   cout << "b: " << b << " c: " << c << endl;
   // result: b:  c: 1
   ```

   ```c++
   string a = " 12 3";
   stringstream iss(a);
   char b, c;
   iss >> b >> c;
   cout << "b: " << b << " c: " << c << endl;
   // result: b:1  c: 2
   ```

2. ```c++
   string a = " 12 3";
   stringstream iss(a);
   string b, c;
   iss >> std::noskipws >> b >> c;
   cout << "b: " << b << " c: " << c << endl;
   // result: b:  c: 
   ```

   ```c++
   string a = " 12 3";
   stringstream iss(a);
   string b, c;
   iss >> b >> c;
   cout << "b: " << b << " c: " << c << endl;
   // result: b: 12 c: 3
   ```

这两个例子中唯一的区别就是第一个我们提供了`char`类型，而第二个是`string`类型。首先我们不去关注代码中的`noskipws`，看第二个例子，即`string`的那一个：根据上一节的描述，当`istream >> string`遇到了**空白字符**时，他会停止读取，而恰好，在字符串`a`的最前边存在一个**空白字符**，所以我们想它应该会直接在开头处停止，不会读取到任何东西，所以最后返回的是空字符串。

但是我们发现如果尝试将代码中的`std::noskipws`去掉，那么返回的结果会与之前不同，这又是为什么呢？

首先我们需要了解运算符`>>`重载的第三种类型：`manipulators`。

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202111081624307.png" alt="image-20211108162408233" style="zoom:50%;" />

注意到`skipws/noskipws`这种类型，当`skipws`被设置时，`stream`默认会忽视所有的空白字符，[文档](https://www.cplusplus.com/reference/ios/skipws/)中提到：

> "For standard streams, the skipws flag **is set** on initialization."

也就是说，在默认情况下初始化的结果为设置了`skipws`，所以所有空白字符都会被读取之后跳过，直到我们又发现了一个**非空白字符**。如此一来，上边例子的结果就好理解了。需要注意的是，在C++中，对于所有的`formatted input`（即可以格式化成我们需要的类型的，比如C中的`prinf, scanf`，C++中使用`>>`操作的），默认都是`noskipws=false`，也就是说会**跳过空白字符**；而对于所有的`unformatted input`（比如`getchar(char), getline(string)`）均设置为`noskipws=true`。

> 关于格式化输入与非格式化输入，我是这么理解的，格式化输入意味着我们可以把输入的数据变成我们想要的类型，比如当我们使用`>>`操作符时，我们可以读入整型，也可以读入一个字符串；但是非格式化输入只能读入`rawText type`，好比`geline(string)`我们输入了一个字符串那么接受类型就是一个字符串.

同时，`skipws`与`noskipws`两者是`sticky`的，也就是说我们只有重新设置，才能变回原来的状态。如果想要每一步都可以设置是否保留**空白字符**，我们可以使用`std::ws`，提取**空白字符**，直到遇到**非空白字符**。

> 设置`skipws`等的步骤是在为输入序列创建`sentry`对象时进行的.

除了上述提到的`skipws`这种`manipulator`之外，常见的可以用于`pad output`的还有：`left/right/internal`, `setw`, `setfill`等：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202111091756123.png" alt="image-20211109175615973" style="zoom:50%;" />

这几个`manipulator`在内部调用了内置的一些方法，比如`setfill`是调用了`basic::ios::fill(char_type)`，`left`是设置了名为`adjustfiled`的`flag value`，`setw`则和使用`width(n)`的效果相同，设置了`field width`–所以该函数指定的是输出序列中允许存在的**最少的字符数**。

#### set position

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202111081358479.png" alt="image-20211108135808429" style="zoom:50%;" />

需要注意的是，在`streamoff`中，`n`的值可正可负。常见的使用方法如下：

```c++
fpos = oss.tellp() + streamoff(3);
oss.seekp(pos);
// or
oss.seekp(streamoff(1), stringstream::cur);
```

### cin & cout

由于`cin`实际上是一种`istream object`，有些特性上边一节已经总结的差不多了…这里只总结两点：

1. 为什么明明输入流默认设置`skipws`，但是我们输入一个`\n`时，`console`内仍然会打印出新的一行？

这是因为控制台内的打印是由`consle software`控制的，与`cin`没有任何关系。

2. cin is *[tied](https://www.cplusplus.com/ios::tie)* to the standard output stream [cout](https://www.cplusplus.com/cout) (see [ios::tie](https://www.cplusplus.com/ios::tie)), which indicates that [cout](https://www.cplusplus.com/cout)'s buffer is *flushed* (see [ostream::flush](https://www.cplusplus.com/ostream::flush)) before each i/o operation performed on cin.

> The tied stream is an output stream object which is *[flushed](https://www.cplusplus.com/ostream::flush)* before each i/o operation in this stream object.

我们常用的`std::endl`就是**换行+刷新输出缓冲区**，所以如果我们仅仅使用`cout`而不添加`endl`，字符会首先存到`output buffer`中，**只有当`output buffer`满了或者程序终止时，才会调用`flush`方法**。

那么问题来了，`std::flush`是如何工作的？[这里](https://stackoverflow.com/questions/14105650/how-does-stdflush-work)给出了较为详细的解答。

言简意赅地说，`ostream::flush()`会在内部调用`streambuf::pubsync()`方法，对流缓冲区进行操作；流缓冲区的作用是负责“缓冲”字符并把数据发送给外部目的地–这发生在缓冲区已满或者内部数据**应当**与外部目的地进行**同步**的时候（比如`flush()`）。当需要同步时，缓冲区内部的数据立即发送给外部目的地，根据官方[文档](https://www.cplusplus.com/reference/ostream/ostream/flush/)：

> For *[stream buffer](https://www.cplusplus.com/streambuf)* objects that implement intermediate buffers, this function requests all characters to be written to the controlled sequence.

这意味着，如果我们正在尝试把数据输出到控制台上，那么此操作会立即把缓冲区内目前所有的字符输出输出流中，再显示到控制台界面上，这意味着清空了`stream buffer`。对于文件流效果是一样的，只不过外部目的地换成了**文件**而已。

所以何时需要进行`flush`操作呢？当然是我们接下来有可能需要向外部目的地写入数据的时候，因为如果此时缓冲区内还存有先前的数据，那毫无疑问的会造成影响！

3. 如果我们注意在输入过程中`internal error flag`的变化，则在每次`flush`之后，以及每次`cin`读取完数据后，内置错误标志位都会被设置成`eof`. 因为输入流意识到自己读到了字符串的末尾。需要注意的是，如果在输入过程中，标志位在某一刻被设置成`fail`，那么此后的所有尝试输入的操作均会失效，这一点我们在前边提到过，必须使用`clear()`清空状态才可以继续进行。

#### ignore

函数原型：

```c++
istream& ignore (streamsize n = 1, int delim = EOF);
```

注意到这种特殊的使用方法，可以帮助我们忽略掉当前`stream buffer`中所有的数据，直到**eof**：

```c++ 
con.ignore(numeric_limits<streamsize>::max(), ‘\n’);
```

### getline(string)

该函数常被用来读取一整行的数据，或者按照`delimiters`分割数据；这是一个`unformatted input function`，根据前几节所属的特点，非格式化输入函数默认设置`noskipws`，那么当遇到分割字符时，它的处理方法是：

> If the delimiter is found, it is extracted and discarded (i.e. it is not stored and the next input operation will begin after it).

同样的，该函数也会对`internal error flags`进行设置。

在文件读取中，如果我们读到了文件末尾，那么`getline`会设置一个`eofbit`，但是此时并不会设置`failbit`，而在继续的下一次读取中，因为我们没有读取到任何内容，**根据failbit**的定义，`sentry object`会置一个`failbit`，借助这一特点，我们在实际使用中常把读取循环写作如下形式而非`while(!input.fail())`，进而避免向数据结构中输入最后一次读取的`garbage value`：

```c++
ifstream file(filename);
while(true) {
    string line;
    getline(file, line);
    if (file.fail()) {
        break;
    }
    // process the read data
}
```

> 我们可以通过在`char*(c-string)`后加一个`s`字母的方式，将其转化为`C++ string`

## type deduction

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202111091508689.png" alt="image-20211109150854609" style="zoom:50%;" />

## structure

### pair

在C++中，传统的返回多个值的方法是使用`reference parameters`，但问题在于除非我们查看函数定义，否则引用参数并不明显。在C++11中提供了`pair`或者`tuple`，我们可以使用他们组合多个返回值：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202111091512437.png" alt="image-20211109151222334" style="zoom:50%;" />

```c++
std::pair<std::string, std::string> f(std::pair<std::string, std::string> p)
{
    return {p.second, p.first}; // list-initialization in return statement
}
```

### tuple

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202111091657348.png" alt="image-20211109165717288" style="zoom:50%;" />

### structured binding

在C++17中，与Python类似，我们可以做`unpack`（`structured binding`）：

```c++
auto [min, max] = findPriceRange(dist);
```

如果我们使用结构体作为以上函数的返回值，那么`structured binding`仍然可以使用。

我们还可以对引用使用`structured binding`:

```c++
void transformToDST(vector<Course>& courses) { 
    for (auto& [code, start, end, instructors] : courses) { 
        start++; 
        end++; 
    }
}
```

```c++
void print_map(std::string_view comment, const std::map<std::string, int>& m)
{
    std::cout << comment;
    for (const auto& [key, value] : m) {
        std::cout << key << " = " << value << "; ";
    }
    std::cout << "\n";
}
```

### Aggregate initialization

随着C++标准的不断更新，初始化的方式愈发多样化。`aggregate initialization`是一种`list-initilization`，在**C++20**中也被叫做`direct initialization`. 他应用于`arrays`以及`class type`，比如**结构体**或者**联合**，但是在使用时有一些特殊要求，可参见[文档](https://en.cppreference.com/w/cpp/language/aggregate_initialization)。

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202111111743654.png" alt="image-20211111174324498" style="zoom:50%;" />

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202111091539422.png" alt="image-20211109153853532" style="zoom:50%;" />

> 这里的`course`是一个结构体

需要注意的是，当使用`uniform initialization`（目前我将这两者理解为相同的东西……）时，对象会首先尝试使用`initilizer list constructor`而非普通的构造函数：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202111091548990.png" alt="image-20211109154855889" style="zoom:33%;" />

另一个使用`uniform intialization`的例子是我们将要初始化一个`std::pair`：我们可以选择使用`{}`语法初始化所有部分，或者初始化一部分：

```c++
foo({ {}, {} }); // OK: call default constructor on both parts
foo({ key, {} }); // OK: call defualt constructor on the second part
foo({ key1, key2 }); // OK
```

下边根据文档说一些比较细节的内容：



**当我们尝试使用`aggregate initialization`的方法初始化结构体等`class type`时，在C++11中被做了`不允许在存在默认初始化参数的情况下使用`的限制，这一限制条件在C++14中被取消。**：

```c++
struct A {
  string str;
  int n = 42;
  int m = -1;
};
A{.m=21}  // Initializes str with {}, which calls the default constructor
          // then initializes n with = 42
          // then initializes m with = 21
```

**文档截图中的初始化方案3、4被称为`designated initilizers`，这种初始化方法要保证所有元素的初始化顺序与定义顺序一样，但是我们可以缺省某个参数：**

```c++
struct A { int x; int y; int z; };
A a{.y = 2, .x = 1}; // error; designator order does not match declaration order
A b{.x = 1, .z = 2}; // ok, b.y initialized to 0
```

我们可以使用`designated initializers`来初始化**联合**，但是**union只允许我们提供一个初始化参数成员：**

```c++
union u { int a; const char* b; };
u f = { .b = "asdf" };         // OK, active member of the union is b
u g = { .a = 1, .b = "asdf" }; // Error, only one initializer may be provided
```

详细内容参见官方文档.

## STL

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202111092139795.png" alt="image-20211109213909613" style="zoom:50%;" />

> `vector[i]` causes undefined behavior!

- `Sequence containers`

1. vector
2. deque（双端队列）
3. array
4. list（双向链表）
5. forward_list（单链表）

- `Container Adaptors`

1. stack
2. queue
2. priority_queue

这类容器之所以叫`adaptors`，是因为他们底层的结构实际上是由另一种容器构成的，我们也可以通过更换函数声明中的模板参数: stack和queue的底层是一个deque:

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202111092042210.png" alt="image-20211109204236062" style="zoom:50%;" />

- `Associative containers`

1. map
2. set
3. unordered_map
4. unordered_set
4. multimap/unordered_multimap
4. multiset/unordered_multiset

> 所谓`multimap`与`multiset`，是指的可以存在多个相同的键值（元素）.

需要注意的是，`map`方法`at(i)`和`[i]`的区别：`at`方法如果没有找到该元素，会抛出一个异常，而如果我们使用`[]`，那么在没有找到该元素时，会先进行创建。

### deque & vector

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20220313011821059.png" alt="image-20220313011821059" style="zoom:50%;" />

如图所示的结构使得`deque`的左右插入要比`vector`更快，但是对于元素的随机访问和随机删除操作，要比`vector`慢。

### iterator

迭代器是STL的重要方法，我们所使用的`range for loop`就是使用迭代器在内部工作的。

> 注意`iterator`的`end`指向的是最后一个元素的后一位.

#### map iterator

`map`的迭代器比较特殊，它指向一个`pair`对象，所以我们可以使用`a.first`和`a.second`获取键与值：

```c++
map<int, int> m;
map<int, int>::iterator i = m.begin();
map<int, int>::iterator end = m.end();
while(i != end) {
    cout << (*i).first << (*i).second << endl;
    ++i;
}
```

在C++20之前，如果我们想要查找容器中是否存在某个键，那么需要调用`find`或者`end`方法，而在C++20中，我们只需要调用`contains`即可。

> 如果`find`成功，则`iterator`指向对应的元素，否则指向`end`. `count`方法通过调用`find`实现，所以`find`方法的速度更快.

#### lower_bound & upper_bound

`lower_bound`接受一个值`v`，返回一个`iterator`对象，该对象指向第一个**不小于**元素`v`的位置，如果没有找到，则指向`end`。

`upper_bound`与`lower_bound`的工作方式类似，但是其返回的迭代器对象指向第一个**大于**元素`v`的位置。

#### Iterator Types Introduction

共有五种基本的迭代器类型：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202111100017458.png" alt="image-20211110001729377" style="zoom:50%;" />

上图中的箭头我们可以理解成*继承*关系。对于所有的迭代器类型，他们都具备以下几种基本操作功能：

- Is *[copy-constructible](https://www.cplusplus.com/CopyConstructible)*, [copy-assignable](https://www.cplusplus.com/CopyAssignable) and [destructible](https://www.cplusplus.com/Destructible)
- can be advanced using `++`
- can be derederenced using `*`

1. `Input Iterators`

应用对象：**连续**+**单向**输入，下边来分别解释这两个限制条件的意思：

- **连续**，表示所应用的数据结构不可以是`queue`，`stack`等（不包括`deque`，因为`deque`实际效果上是一个优化了前序插入的`vector`）
- **单向**（`single-pass`），迭代器对应的每个位置只能允许被*经过*一次，这个限制条件之后会详细解释

`input iterators`只能作为右值（`rvalue`）被解引用：

```c++
int val = *itr;
```

这种迭代器的应用场景有之前提到过的`find`以及`count`函数等。这种迭代器类型数据是**只读的**。

2. `output iterators`

它的应用条件与`input iterators`相同，但是它只能作为左值（`lvalue`）被解引用：

```c++
*itr = 12;
```

该迭代器对象的应用场景主要有`copy`以及`output streams`等。它是**只写的**。

3. `forward iterators`

这种迭代器类型与把前两种类型结合起来的效果类似，不同的是，他可以做`multiple pass`. 应用场景主要有`replace`函数以及`forward_list`中。

4. `bidirectional iterators`

承接`forward iterators`，但是可以做`--`操作：

```c++
vector<int>::iterator itr = v.begin();
--itr;
```

应用场景主要有`reverse`函数，`map`, `set`以及`list`等。

5. `random Access iterators`

承接上一种类型，但是支持`+=n`与`-=n`操作，同时可以使用`offset dereference operator ([])`，应用在容器`vector`, `deque`, `string`以及**指针**上。

> Instead of being defined by specific types, each category of iterator is defined by the operations that can be performed on it. This definition means that any type that supports the necessary operations can be used as an iterator -- for example, a pointer supports all of the operations required by [*LegacyRandomAccessIterator*](https://en.cppreference.com/w/cpp/named_req/RandomAccessIterator), so a pointer can be used anywhere a [*LegacyRandomAccessIterator*](https://en.cppreference.com/w/cpp/named_req/RandomAccessIterator) is expected.

关于迭代器类型，补充以下一些内容：

除了上述提到的5种基本类型的迭代器，**C++**后来新增了一种基本迭代器类型：`Contiguous Iterator`，这种迭代器类型在`randomAccess Iterators`的基础上保证了**其中的数据在内存必须连续存储**。

在[**C++20**](https://en.cppreference.com/w/cpp/iterator)中，根据新的关键字`concept`以及`requires`对迭代器类型根据新的系统设计了一套定义，但是基本的类型是相似的；

> 我们可以简单地将这两个关键字的功能理解为：为模板参数制定一些限制，使得在*编译*阶段就能够进行`evaluation`。应用他们的主要优点是可以得到更加清晰地编译器报错。这里不去深究，[这篇文章](https://www.cppstories.com/2021/concepts-intro/)写的较为清楚。

新的迭代器定义基本方法是先使用这两个关键字定义了一种`input_or_output_iterator`类型，之后扩展的每一种类型均满足这种基本迭代器类型的要求，从定义中可以看到，这种基本类型支持两种操作：解引用+递增操作符：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202111101555478.png" alt="image-20211110155516376" style="zoom: 33%;" />

新定义的迭代器类型除了`output_iterator`是直接在`input_or_output_iterator`基础上增加了写入功能之外，其余的迭代器类型均[继承](https://en.cppreference.com/w/cpp/concepts/derived_from)自**C++20**之前的迭代器类型，比如对于`input iterator`:

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202111101600970.png" alt="image-20211110160016902" style="zoom:50%;" />

#### What is single-pass?

什么是`single-pass`？为什么只能`single-pass`？**C++标准库**中提到：

> **For input iterators, a == b does not imply ++a == ++b.** (*Equality does not guarantee the substitution property or referential transparency.*) Algorithms on input iterators should never attempt to pass through the same iterator twice. They should be single pass algorithms. Value type T is not required to be an Assignable type (23.1). These algorithms can be used with istreams as the source of the input data through the istream_iterator class.

[这里](https://stackoverflow.com/questions/5947683/for-input-iterators-why-a-b-does-not-imply-a-b)对该问题进行了细致的讨论。在讨论`input iterators`时，常使用`istream_iterator`作为例子，该迭代器类型从输入流中读取数据，我们可以想到，无论该读取过程是否需要用到`stream buffer`，读取都是发生在迭代器不断向前推进的时候，而输入流某个位置在被读取之后，我们就不能够再次对流中同一个位置的元素进行操作了，也就是说我们**只能用一个迭代器，单向，走一次**：

> `std::istream_iterator` is a single-pass input iterator that reads successive objects of type `T` from the [std::basic_istream](https://en.cppreference.com/w/cpp/io/basic_istream) object for which it was constructed, by calling the appropriate `operator>>`. The actual read operation is performed when the iterator is incremented, not when it is dereferenced. The first object is read when the iterator is constructed. Dereferencing only returns a copy of the most recently read object.
>
> The default-constructed `std::istream_iterator` is known as the *end-of-stream* iterator. When a valid `std::istream_iterator` reaches the end of the underlying stream, it becomes equal to the end-of-stream iterator. Dereferencing or incrementing it further invokes undefined behavior.

所以在标准文档给出的说明中，`++a==++b`为什么此时不能够保证成立呢？这里首先我们要明确以下`++`操作符的作用机理：

##### Increment/decrement operators

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202111101700052.png" alt="image-20211110170018954" style="zoom:50%;" />

根据[官方文档](https://en.cppreference.com/w/cpp/language/operator_incdec)中的解释，**前置**递增操作符的运算过程与`x += 1`完全相同，在对元素递增之后，**返回一个该元素的引用**；而**后置**递增操作符则是先保存一下元素副本，对该元素本身（引用）进行递增之后，**把副本（保存着原值）返回**。

在清楚了`++`操作符的作用机理后，我们把目光转回到`iterator`上。在表达式`++a==++b`上，如果我们已经进行了`++a`的操作，那么根据`istream_iterator`的特性，已经对流内的数据进行了读取，那么此时`iterator b`就找不到流内原位上的数据了，换而言之，`iterator b`**失效了**！

> 如果我们读取的是字符，那么需要用到`stream buffer`，此时使用[std::istreambuf_iterator](https://en.cppreference.com/w/cpp/iterator/istreambuf_iterator)是效率更高的选择，因为他不需要对每一个字符建立一个`sentry object`，程序在开始时先建立一个`sentry object`，之后把所有数据放入缓冲区，再使用迭代器进行读取.

所以，在`count`以及`find`函数中，我们确实只需要走一次就可以获得我们想要的结果，于是只需要使用一个`input iterator`即可。对应着`output iterator`的`ostream_iterator`也是一样的道理。

#### Write our own iterator?

在C++17之前的版本中，如果我们想给自己的类写一个`iterator`，那么一般的方法是继承`std::iterator`的类模板，并在类模板中指定所谓的`iterator_category`：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202111101732699.png" alt="image-20211110173203619" style="zoom:50%;" />

迭代器标签关联着之前所述的迭代器实体。但是自**C++17**起，`std::iterator`的类模板遭到舍弃，[详情参见](https://stackoverflow.com/questions/37031805/preparation-for-stditerator-being-deprecated)。所以现在，我们只能手写迭代器，具体可以看[这里](https://segmentfault.com/a/1190000040879971)。

> 在这里，`iterator_traits`是一个`meta-function`，返回的是根据给入参数决定的迭代器的`type`（通过`::iterator_category`），也即上图中的各种`iterator_tag`，这些`tags`绑定着一些同名迭代器类实体，我们可以利用这些类别构造我们自己的函数，在下方模板一章节的最后一个例子中有提到（来自于*Stanford CS 106L Fa2020 lecture*）。

##### iterator – friend class

在作业*HashMap*中，我们被要求手写一个迭代器类。在这里，我们被要求将`hashmap`与`hashmap_iterator`的定义（声明）放到两个头文件中。所以，而迭代器与容器本身的关系要求他们**互为友元**，也是我们后边提到的`mutual friends class`，因为此处均为模板类，所以我们使用了`Template friends class`的方法在各自类中声明其友元。

需要注意的是，我们应当把该类也声明做友元，因为我们可能会获取它的成员们：

```c++
friend iterator_of_hashmap<HashMap, !Const>;
```

##### iterator – const?

在这里，一个很重要的方法是利用`std::conditional_t<>`来对迭代器内部的数据类型（是否为`const`）进行条件选择–这是因为我们要提供`iterator`与`const-iterator`两个类。而`const_iterator`之所以存在，是因为当我们的容器（`container`）类被置为`const`时，迭代器不应该被允许修改数据：

> 当一个类对象为常量对象时，其中所有数据也均为常量类型的数据。而对于常量数据，只能使用指向常量的指针指向他们，或者使用`const`引用与他们绑定，否则会编译错误。

这就是为什么在代码中我们需要诸如：

```c++
using node_pointer = std::conditional_t<Const, const node*, node*>;
```

##### iterator – basic requirement

```c++
using difference_type = std::ptrdiff_t;
using value_type = std::conditional_t<Const, const _value_type, _value_type>;
using pointer = std::conditional_t<Const, const _value_type*, _value_type*>;
using reference = std::conditional_t<Const, const _value_type&, _value_type&>;
using iterator_category = std::forward_iterator_tag;
```

我们必须添加这些以满足程序对`iterator`类设计的基本要求，同时根据`iterator_category`提供**必要的运算符重载**，需要注意的是其中前置`++`与后置`++`的实现方法（基本是固定套路），参看我的[Github repository](https://github.com/SongShaopu1998/Stanford-CS-106L/blob/main/Assignment3-HashMap/hashmap_iterator.h). 

根据*Mastering the C++17 STL*这本书的设计，我们还提供了一个`conversion operator`，允许迭代器从`iterator`转型到`const_iterator`：

```c++
/** a conversion operator */
operator iterator_of_hashmap<HashMap, true>() const {
    return iterator_of_hashmap<HashMap, true>{_buckets_array, _ptr, curr_bucket};
}
```

##### iterator – private member variables

迭代器的内部数据成员一定会有一个指针，指向容器中具体的，每一个数据的位置（注意：也要通过`std::consitional_t`来选择），其他私有成员根据需要提供，比如在*HashMap*中我们就需要一个`size_t`来告诉我们现在内部指针处于第几个`bucket`中。在设计时，还包含了一个指向容器的数据区的指针，具体可参见后边的`Template mutual friends`的一节。

我们使用一个**private constructor**，这是因为除了在该类及其友元类中，别处不允许**直接初始化**一个迭代器。

## Template–Generic Programming

关于模板，[这里](https://sg-first.gitbooks.io/cpp-template-tutorial/content/T_ji_ben_yu_fa.html)有一篇很好的文章。

### function template

函数模板的基本形式是：

```c++
template<typename T>
T getInteger(const string& prompt, const string& reprompt) {
    while (true) {
        cout << prompt; 
        string line; 
        T result; 
        char extra; 
        if (!getline(cin, line))
            throw domain_error(“[shortened]”);
        istringstream iss(line);
        if (iss >> result && !(iss >> extra)) 
            return result; 
        cout << reprompt << endl; 
    }
}
```

### class template

e.g.

```c++
template<class T, class Container = std::vector<T>>
class Priority_Q {
  // do something  
};
```

### template parameters & template arguments

函数模板的形参列表可以由几下几种方式构成：

1. `non-type template parameter`

什么是非类型模板参数？**非类型模板参数会在编译阶段被预定义并被替换为常量值**作为实参，它可以具备如下类型：

- An integral type
- An enumeration type
- A pointer or reference to a class object
- A pointer or reference to a function
- A pointer or reference to a class member function
- std::nullptr_t
- A floating point type (since C++20)

对于该类型的参数，我们需要注意以下几点：

1.  **The top-level `cv-qualifiers` on the template-parameter are ignored** when determining its type

```c++
// this const will be ignored
template<const int k>
void foo() {
    // do something
}
```

2. When the name of a non-type template parameter is used in an expression within the body of the class template, it is an unmodifiable [prvalue](https://en.cppreference.com/w/cpp/language/value_category) unless its type was an lvalue reference type, or unless its type is a class type (since C++20).

```c++
template <int N>
void f()
{
    // N is a prvalue, so this is invalid! It will produce a compile error
    N = 42;
}
```

2. the expression we give to the template as its argument must can be parsed and substitued during compile time (**non-constant** expressions **cannot** do this, since they could change during runtime)

> 因为模板的匹配是在编译的时候完成的，所以实例化模板的时候所使用的参数，也必须要在编译期就能确定。

```c++
template <int N>
void f()
{
    // do something
}

int main() {
    int a = 1;
    const int b = 0; // constexpr int b = 0;
    // error
    f<a>();
    // ok
    f<b>();
}
```

2. `type template parameter`

这类参数包括使用`typename`与`class`(即分别对应了函数模板与类模板的两种`type-parameter-key`)，在**C++20**中还提出了使用`constrained type template`:

```c++
template<typename... Ts> concept C2 = true; // variadic concept
template<C2... T> struct s3;      // constraint-expression is (C2<T> && ...)
```

3. `template template parameter`

模板模板参数：

```c++
// two type template parameters and one template template parameter:
template<typename K, typename V, template<typename> typename C = my_array>
class Map
{
    C<K> key;
    C<V> value;
};
```

#### instantiation（explicit/implicit）

在调用模板函数时，我们可以显式指定对应的模板参数类型（`explicit instantiation`）,也可以不显式指定模板参数类型（`implicit instantiation`），让模板函数进行**类型推断（`Template Argument Deduction`）**，需要注意的是，如果我们省略`<>`符号，那么`overload resolution`会查找所有的模板类型重载与非模板类型重载，而不是仅仅查找模板类型重载。

> 什么是`overload resolution`?后边有提到.

需要注意的是，如果我们在`explicit instantiation`之前已经做了模板特化（`template specialization`），那么此时的实例化是不起作用的。

#### implicit inference

在模板函数中，我们可能会隐含着一些使用参数的限定条件，比如对于下图所示的函数：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202111131433705.png" alt="image-20211113143322516" style="zoom:50%;" />

其实隐含着这样一些限定参数使用条件：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202111131434734.png" alt="image-20211113143458651" style="zoom:50%;" />

这些限定条件意味着如果我们在调用函数时给出了不恰当的参数，那么该模板函数会报出一些非常凌乱的错误…

在**C++20**中给出了新的关键字`concepts`与`requires`，也就是在之前我们提到过用于重写`iterator`的两个关键字，他们可以帮助我们在编译阶段就检查出参数错误，将报错的位置从函数主体转移到这两个关键字的位置上：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202111131442975.png" alt="image-20211113144241875" style="zoom:50%;" />

既然`concepts`关键字可以方便解决模板编译与运行过程中的报错问题的，那么接下来就针对模板的编译过程进行分析。

### Compile a template function call

接下来所述的概念来自于官方文档以及这两篇[1](https://www.cppstories.com/2016/02/notes-on-c-sfinae/#improved-code)[2](https://www.cppstories.com/2018/03/ifconstexpr/#c20)博客. 根据作者原文给出的例子:

```cpp
struct Bar {
    typedef double internalType;  
};

template <typename T> 
typename T::internalType foo(const T& t) { 
    cout << "foo<T>\n"; 
    return 0; 
}

int main() {
    foo(Bar());
    foo(0); // << error!
}
```

当我们向程序中添加了一个新的非模板函数重载`int foo(int i)`时，程序便不会报错，为什么呢？我们需要了解函数整个的编译过程。

#### name lookup

根据[官方文档](https://en.cppreference.com/w/cpp/language/overload_resolution),为了编译一个函数调用，程序首先会进行`name lookup`，这一步是将程序中出现的名字与引入它的**声明**联系起来，比如对于如下语句，程序正是通过`name lookup`的方式来解析这条语句中出现的各个名字：

```c++
std::cout << std::endl;
```

> 在进行`name lookup`时，我们依赖的是`scope`这个概念，也即名称的**声明**所在地，`scope`有很多[种类](https://en.cppreference.com/w/cpp/language/scope)，`namespace`,`function`,`class`,`block`,`enumeration`,`template parameter`都拥有自己的`scope`.

##### unqualified name lookup & qualified name lookup

在上边的例子中，编译器首先会对名称`std`进行`unqualified name`查找，在头文件`<iostream>`中找到它；之后再对`cout, endl`做`qualified name`的查找，即在命名空间`std`中进行的查找过程。

> `qualified name`即在作用域解析符`::`右侧出现的名称，所以他包括：
>
> - class member
> - namespace member
> - enumerator
>
> 同样地，`unqualified name`即没有在作用域解析符`::`右侧出现的名称

需要注意的是，如果作用域解析符`::`左侧没有东西，那么程序会默认在`global namespace scope`或者由`using`引入的命名空间中进行查找：

```c++
#include <iostream>
int main() {
  struct std{};
  std::cout << "fail\n"; // Error: unqualified lookup for 'std' finds the struct
  ::std::cout << "ok\n"; // OK: ::std finds the namespace std
}
```

##### Argument-dependent lookup (ADL)

[这种](https://en.cppreference.com/w/cpp/language/adl)名称查找方式也属于一种`unqualified name lookup`，叫`Koenig lookup`，用于查找`function-call expressions`中的`unqualified function names`（包括运算符重载时对函数的隐式调用），这种查找方式的存在使得在进行函数调用时，程序不仅仅会查找一般的`unqualified name lookup`的`scope`和命名空间，还会查找**函数参数们所在的命名空间**：

该示例来自于官方文档：

```c++
#include <iostream>
int main()
{
    std::cout << "Test\n"; // There is no operator<< in global namespace, but ADL
                           // examines std namespace because the left argument is in
                           // std and finds std::operator<<(std::ostream&, const char*)
    operator<<(std::cout, "Test\n"); // same, using function call notation
 
    // however,
    std::cout << endl; // Error: 'endl' is not declared in this namespace.
                       // This is not a function call to endl(), so ADL does not apply
 
    endl(std::cout); // OK: this is a function call: ADL examines std namespace
                     // because the argument of endl is in std, and finds std::endl
 
    (endl)(std::cout); // Error: 'endl' is not declared in this namespace.
                       // The sub-expression (endl) is not a function call expression
}
```

#### Template Argument Deduction

在进行完`name lookup`这一步之后，程序会进行`Template Argument Deduction`：

> In order to instantiate a [function template](https://en.cppreference.com/w/cpp/language/function_template), every template argument must be known, but not every template argument has to be specified. When possible, the compiler will deduce the missing template arguments from the function arguments. This occurs when a function call is attempted, when an address of a function template is taken, and in some [other contexts](https://en.cppreference.com/w/cpp/language/template_argument_deduction#Other_contexts).

这一机制以及上一节提到的`ADL`，让我们使用`template operator`成为可能，因为在使用`template operator`时，我们无法显式指定所用的模板参数的类型：

```c++
#include <iostream>
 
int main() 
{
    std::cout << "Hello, world" << std::endl;
    // operator<< is looked up via ADL as std::operator<<,
    // then deduced to operator<<<char, std::char_traits<char>> both times
    // std::endl is deduced to &std::endl<char, std::char_traits<char>>
}
```

需要注意的是，对于函数以及函数模板名称，`name lookup`可以找到多个合理的名称，并交给之后的`overload resolution`过程处理，但是对于其他的名字，比如变量，类，命名空间等，`name lookup`**只允许找到一个名字的声明以保证正常编译**。

#### Template Argument Substitution

现在，我们已经获取到了所使用的模板参数类型，在这一步我们要将所有出现在原本的模板中的参数`T`替换为我们推断出的参数类型。

> When all template arguments have been specified, deduced or obtained from default template arguments, every use of a template parameter in the function parameter list is replaced with the corresponding template arguments.

需要注意的是如果在替换过程中，出现了错误，比如在先前给出的例子中，我们获得了一个`int::interalType`的类型，那么此时，我们需要把这一参数类型（一个替换方案）剔除出`overload set`，这一[操作](https://en.wikipedia.org/wiki/Substitution_failure_is_not_an_error)被称为**SFINAE–*Substitution failure is not an error***.

#### overload resolution

在做完`Template Argument Substitution`这一步后，如果`overload set`中已然没有可用的函数，那么编译失败，但存在这样一种情况：即目前有多个`candicate function`可供选用，那么此时`overload resolution`便会派上用场。

> In order to compile a function call, the compiler must first perform [name lookup](https://en.cppreference.com/w/cpp/language/lookup), which, for functions, may involve [argument-dependent lookup](https://en.cppreference.com/w/cpp/language/adl), and for function templates may be followed by [template argument deduction](https://en.cppreference.com/w/cpp/language/template_argument_deduction). If these steps produce more than one *candidate function*, then *overload resolution* is performed to select the function that will actually be called.

> 这是一种针对于`function call`的通用概念，并非只针对`template`.

当我们试图在模板函数中应用`overload resolution`以选择最佳匹配重载函数时，会使用`partial ordering of overloaded function templates`的[方法](https://en.cppreference.com/w/cpp/language/function_template)来选择最佳匹配。什么是最佳匹配重载函数？

> Informally "A is more specialized than B" means "A accepts fewer types than B".

*StackOverflow*上有一篇很详细的[文章](https://stackoverflow.com/questions/17005985/what-is-the-partial-ordering-procedure-in-template-deduction)描述这一机理。这也是为什么在先前的例子中加入一个匹配的新`function`，就可以编译成功的原因。

#### SFINAE

在上一节的`template argument substituion`中，我们提到了`SFINAE`的内部作用机理，那么我们要如何写出这样的代码或者说这一机理有什么用处呢？

##### Why we need SFINAE?

在这篇[文章](https://www.cppstories.com/2018/03/ifconstexpr/#c20)里，其实解释的很清楚。这里是我的一个总结。

- 首先，对于我们自己的函数模板代码，如果其中存在`if-else`分支，并且我们利用了`compile-time-if`，那么程序很有可能会在编译阶段报错，因为他会编译`if-else`两个分支，之后程序发现自己无法`reject the invalid code (for this case)`：

```cpp
template <typename T>
std::string str(T t) {
    if (std::is_convertible_v<T, std::string>)
        return t;
    else if (std::is_same_v<T, bool>)
        return t ? "true" : "false";
    else
        return std::to_string(t);
}
```

而我们要做的，就是让程序在编译阶段把在当前情况下不正确的分支给剔除，从`overload set`中将其去掉，只编译符合要求的部分，这也对应着`SFINAE`的定义。

##### How to leverage SFINAE?

为了解决这个问题，**C++11/14/17/20**提出了多种方案：

###### [Tag Dispatching](https://www.cppstories.com/2016/02/notes-on-c-sfinae/#improved-code)

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202111132349840.png" alt="image-20211113234948754" style="zoom:50%;" />

###### enable_if

> **std::enable_if** can be used as an additional function argument (not applicable to operator overloads), as a return type (not applicable to constructors and destructors), or as a class template or function template parameter.

[这一语法](https://en.cppreference.com/w/cpp/types/enable_if)自**C++11**提出，但当时的写法是`enable_if`，在**C++14/17**中，提出了`enable_if_t`.

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202111132147605.png" alt="image-20211113214738450" style="zoom:50%;" />

> 模板偏特化

也就是说，当`B==true`时，`enable_if`具备一个`typedef type`，即`T`，否则没有`type`成员，就会发生我们所说的**SFINAE**，于是当我们写成如下代码：

```cpp
template <class T>
typename std::enable_if<std::is_arithmetic<T>::value, T>::type
```

或者参看文档中的`alias`–`enable_if_t`的定义时，如果其不具备成员`type`但是我们却使用了`::type`，那么编译就会出错，并使用`SFINAE`机制处理。在**C++14**及以后的版本中，上述代码可被简化为：

```c++
template <class T>
typename std::enable_if_t<std::is_arithmetic_v<T>, T> 
```

`enable_if`常与`type traits`一起使用：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202111132152195.png" alt="image-20211113215251103" style="zoom:50%;" />

`type traits`在**编译阶段**用以确定模板特征和属性。在**C++14**中也提出了类似于`is_arithmetic_v`的形式（原本的`value`当`T`为`arithmetic type`是返回`true`，反之`false`）

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202111132155450.png" alt="image-20211113215505370" style="zoom:50%;" />

借助于`enable_if_t`，我们可以将本节最初的代码更改为：

```cpp
template <typename T>
enable_if_t<is_convertible_v<T, string>, string> strOld(T t) {
    return t;
}

template <typename T>
enable_if_t<!is_convertible_v<T, string>, string> strOld(T t) {
    return to_string(t);
}
```

> 关于`enable_if`的缺点，*StackOverflow*上的一篇[文章](https://stackoverflow.com/questions/14600201/why-should-i-avoid-stdenable-if-in-function-signatures)进行了探讨。

> **需要注意的是，`enable_if`这里直接充当了函数的返回值类型，因为它本身就返回一个`type`**

`enable_if`除了可以放在返回值的位置上以外，还可以直接使用在模板参数、函数参数中，或者在模板偏特化中出现，它的使用方式十分灵活，具体可以看官方文档：

```c++
// 2. the second template argument is only valid if T is an integral type:
template < class T,
           class = typename std::enable_if<std::is_integral<T>::value>::type>
bool is_even (T i) {return !bool(i%2);}

// the partial specialization of A is enabled via a template parameter
template<class T, class Enable = void>
class A {}; // primary template
 
template<class T>
class A<T, typename std::enable_if<std::is_floating_point<T>::value>::type> {
}; // specialization for floating point types
```

在使用`enable_if`或者`enable_if_t`时，我们也可以省略模板参数中的`T`，因为`T`具备默认值`void`。

###### Notes

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202111132229957.png" alt="image-20211113222941812" style="zoom:50%;" />

> 需要注意的是，这里提到的是`only differ in default template arguments`，如果两个模板的`default template argument`一样但是`template parameters`不一样，那么不会发生编译错误.
>
> 所以如上图所示，对于第一个例子，由于两者只在默认模板参数上有所不同，进而报错。但是第二个例子两者的模板参数是不一样的（虽然两者的默认模板参数都被设置成了`true`，但仍然不会报错）

###### if constexpr

在**C++17**中，我们可以将`if conxtexpr statement`使用在模板中，起到与`enable_if_t`相同的作用：

```c++
template <typename T>
auto get_value(T t) {
    if constexpr (std::is_pointer_v<T>)
        return *t; // deduces return type to int for T = int*
    else
        return t;  // deduces return type to int for T = int
}
```

###### concepts-requires

终于，我们来到了`concepts`。

```cpp
// concept:
template<typename T>
concept Hashable = requires(T a) {
    { std::hash<T>{}(a) } -> std::convertible_to<std::size_t>;
};
```

在这里，我们定义一个`concept`，将一个`requires-expression`赋值给它。`requires`关键字有很多种写法：

- requires clauses

```c++
template<typename T>
void f(T&&) requires Eq<T>; // can appear as the last element of a function declarator
 
template<typename T> requires Addable<T> // or right after a template parameter list
T add(T a, T b) { return a + b; }
```

- requires expressions

```c++
template<typename T>
concept Addable = requires (T x) { x + x; }; // requires-expression
```

- simple requirements

```c++
template<typename T>
concept Addable =
requires (T a, T b) {
    a + b; // "the expression a+b is a valid expression that will compile"
};
```

> 因为所有以关键字`requires`开头的`requirements`都会被解释成`nested requirement`，所以`simple requirements`不会以`requires`开头

- Type requirements

```c++
template<typename T> using Ref = T&;
template<typename T> concept C =
requires {
    typename T::inner; // required nested member name
    typename S<T>;     // required class template specialization
    typename Ref<T>;   // required alias template substitution
};
```

- Compound requirements

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202111222032893.png" alt="image-20211122203241729" style="zoom:50%;" />

- Nested requirements

###### Abbreviated function template

自**C++20**始，提出了`Abbreviated function template`的做法，即使用`placeholder types(auto, concept auto)`出现在函数声明或者函数模板声明中：

```c++
void f1(auto); // same as template<class T> void f(T)
void f2(C1 auto); // same as template<C1 T> void f2(T), if C1 is a concept
void f3(C2 auto...); // same as template<C2... Ts> void f3(Ts...), if C2 is a concept
void f4(const C3 auto*, C4 auto&); // same as template<C3 T, C4 U> void f4(const T*, U&);
template <class T, C U>
void g(T x, U y, C auto z); // same as template<class T, C U, C W> void g(T x, U y, W z);
```

于是对于原博客中给出的例子：

```cpp
// requires:
template <typename T>
requires std::is_floating_point_v<T>
constexpr bool close_enough20(T a, T b) {
   return absolute(a - b) < precision_threshold<T>;
}
constexpr bool close_enough20(auto a, auto b) {
   return a == b;
}
```

根据[官方文档](https://en.cppreference.com/w/cpp/language/function_template#Abbreviated_function_template)，我们利用`constrained auto`将其改写为：



```cpp
constexpr bool close_enough20(std::floating_point auto a,
                              std::floating_point auto b) {
   return absolute(a - b) < precision_threshold<std::common_type_t<decltype(a), decltype(b)>>;
}
constexpr bool close_enough20(std::integral auto a, std::integral auto b) {
   return a == b;
}
```

这里有一篇[文章](https://devblogs.microsoft.com/cppblog/abbreviated-function-templates-and-constrained-auto/)作了较为详细的介绍。

### Template friends

这是一节单独列出来的主题。分为两个部分，**Template class friends**和**Template function friends**，在第二种中尤其讨论**Template friend operator**.

#### Template class friends

在做`hashmap`作业时，我们需要将自己实现的`iterator class`作为友元声明到`container`也即`hashmap`类中，我们有两种方法来实现这一声明操作：

```c++
template<typename T>
class foo{
    template<typename U>
    friend class foo_1;
};
```

> Template parameters cannot shadow each other.

如此一来，不论`T`被实例化为什么类型，任何类型的`foo_1`类都是`foo`的友元。需要注意的是**类模板的声明方式**，我们不可以写出这样的语法：

```c++
template<typename U>
friend class foo_1<U>;
```

或者我们也可以这样声明，这也是`hashmap`中我们可以做的（因为只需要这么做就足够了）：

```c++
template<typename T>
class foo{
    friend class foo_1<T>;
};
```

这种声明方式中，**模板参数推断**只发生在一开始的类模板部分，而在类内部仅仅对`foo_1`按照推断出来的类型`T`做了一次实例化。如若`foo`被实例化为`foo<int>`，那么此时也便只有`foo_1<int>`有资格成为它的友元了。

我们可以在友元处做模板特化（`Template specialization`），需要注意的是，**模板友元**（`Template friends`）不允许我们做**模板偏特化**，但是我们可以做**模板全特化**：

```c++
template<class T> class A {}; // primary
template<class T> class A<T*> {}; // partial,因为此时T*仍然带有泛型T
template<> class A<int> {}; // full, int已经脱离T了，被“完全特化”了，所以模板参数中也不需要填东西
class X {
    template<class T> friend class A<T*>; // error!
    friend class A<int>; // OK
};
```

#### Template function friends

我们常常需要将重载的操作符声明为友元（见下方`Object-Oriented Programming`部分详述），根据[这篇文章](https://stackoverflow.com/questions/4660123/overloading-friend-operator-for-template-class)，我们有三种方式实现这一声明操作：

##### The Extrovert

与模板友元类类似，我们可以这样声明：

```c++
template <typename T>
class Test {
   template <typename U>      // all instantiations of this template are my friends
   friend std::ostream& operator<<( std::ostream&, const Test<U>& );
};
template <typename T>
std::ostream& operator<<( std::ostream& o, const Test<T>& ) {
   // Can access all Test<int>, Test<double>... regardless of what T is
}
```

按照友元声明习惯，将声明放在类内部，定义放在类外部，这种方法类似模板友元类声明的第一种方法，不再详述。

##### The Introverts

我们也可以把友元函数的声明+定义全部放在类内，这允许我们进行如下两种声明模式：

> 这两种声明**不允许**我们将函数定义放在类外，否则编译器无法找到模板函数的定义

```c++
template <typename T>
class Test {
   friend std::ostream& operator<<( std::ostream& o, const Test& t ) {
      // can access the enclosing Test. If T is int, it cannot access Test<double>
   }
};
```

这种方法类似模板友元类声明的第二种模式，不同的是在这里我们必须给函数在类内提供一个定义。

或者，我们也可以做得更加麻烦一些，在[官方文档](https://en.cppreference.com/w/cpp/language/friend)中也提过这种方法：

> or the function template has to be declared as a template before the class body, in which case the friend declaration within `Foo<T>` can refer to the full specialization of `operator<<` for its `T`:

```c++
// Forward declare both templates:
template <typename T> class Test;
template <typename T> std::ostream& operator<<( std::ostream&, const Test<T>& );

// Declare the actual templates:
template <typename T>
class Test {
   // refers to a full specialization for this particular T 
    friend std::ostream& operator<< <> (std::ostream&, const Test&);
    // note: this relies on template argument deduction in declarations
    // can also specify the template argument with operator<< <T>":
    // friend std::ostream& operator<< <T>( std::ostream&, const Test<T>& );
};
// Implement the operator
template <typename T>
std::ostream& operator<<( std::ostream& o, const Test<T>& t ) {
   // Can only access Test<T> for the same T as is instantiating, that is:
   // if T is int, this template cannot access Test<double>, Test<char> ...
}
```

### Template meta-programming

本部分内容来自于`Stanford CS 106L Fa2020`的`guest lecture`，以及[C++ Template Tutorial](https://sg-first.gitbooks.io/cpp-template-tutorial/content/TMP_ji_chu_md.html)和**C++**官方文档。

在学习了一系列基本关于模板的基础知识以及先前关于`iterator`的内容后，对模板元编程（`Template meta-pragramming`）做一下浅显的了解和认识。在这一章节，也会对先前没有提到的模板特化（`Template Specialization`）做一些介绍。

所以什么是元编程？这里直接引用提到的文章中的一段话：

> 元（meta）无论在中文还是英文里，都是个很“抽象（abstract）”的词。因为它的本意就是“抽象”。元编程，也可以说就是“编程的抽象”。用更好理解的说法，元编程意味着你撰写一段程序A，程序A会运行后生成另外一个程序B，程序B才是真正实现功能的程序。那么这个时候程序A可以称作程序B的元程序，撰写程序A的过程，就称之为“元编程”。
>
> 我们的目的，是找出程序之间的相似性，进行“元编程”。而在C++中，元编程的手段，可以是宏，也可以是模板。

#### Template Specialization

我们应当区分模板特化与模板实例化的区别，其实实例化（`template instantiation`也可以被叫做模板实例化的特例（`instantiated/generated specialization`））模板特化又分为`Explicit/full specialization`与`Partial template specialization`(模板偏特化)两种。对于模板特化，首先我们要注意以下几点规则：

1. 在定义模板特例之前应当有模板通例（`generic template/primary template`）在先
2. 模板**偏特化（部分特化）**（`partial template specialization`）只适用于`class template`和`variable template`。
3. 模板特化的匹配规则遵循以下顺序：

> - If only one specialization matches the template arguments, that specialization is used
>
> - If more than one specialization matches, partial order rules are used to determine which specialization is more specialized. The most specialized specialization is used, if it is unique (if it is not unique, the program cannot be compiled)
>
> - If no specializations match, the primary template is used

这也是我们在先前提到过的`Partial ordering`方法，只不过在模板特化这里被使用并明确了。简而言之，**从特例到通例**，从`more specializaed`到`less specialized`。

#### Template Specialization Examples

```c++
// primary template
template <typename T> class TypeToID
{
public:
    static int const ID = -1;
};
// specialization template
template <> class TypeToID<uint8_t>
{
public:
    static int const ID = 0;
};
```

需要注意的是，在模板特化时，如果我们特化的类型不需要依赖模板参数`T`，那我们就不需要也不能在`specialiazed template`中的`<>`位置写任何东西，因为他的位置已经被其后的`int`占领了：

```c++
// use float specialization
template <> class TypeToID<float>
{
  static int const ID = 0xF10A7;
};
// use void* specialization
template <> class TypeToID<void*>
{
  static int const ID = 0x401d;
};
```

但有些时候我们不得不给入一个模板参数`T`来进行模板特化，比如我们要特化所有指针类型：

```c++
template <typename T>                    // 嗯，需要一个T
class TypeToID<T*>                        // 我要对所有的指针类型特化，所以这里就写T*
{
public:
    typedef T         SameAsT;
    static int const ID = 0x80000000;    // 用最高位表示它是一个指针
};
```

在进行模板特化时，我们需要注意的一点是，特化的模板可以把我们提供的类型中的`const`或者`*`给去掉，也就是说，当我们给如下的模板定义分别提供`float*`与`const int`时，其中对应的`T`分别变成了`float`与`int`:

```c++
// float* -> T:float
template <typename T>
class RemovePointer<T*> {
public:
    typedef T Result;
};

// const int -> T:int
template<typename T>
class RemovePointer<const T> {
public:
    using type = T;
};
```

在继续进行我们的主题之前，我们先来对模板中的名称查找问题进行解决，这会帮助我们理解之后的例子。

#### Name Resolution

在一开始给到的文章中，提到了`Name Resolution`这一概念：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202111231726211.png" alt="image-20211123172604046" style="zoom:50%;" />

这里我们需要注意的是，**模板中的名称**可以被分为两个大类：**依赖性名称**与**非依赖性名称**。什么是**依赖性名称**？

> **A dependent name is a name that depends on the type or the value of a template parameter. For example**：

```c++
template<class T> class U : A<T>
{
  typename T::B x;
  void f(A<T>& y)
  {
    *y++;
  }
}; 
```

在这里，`A<T>`，`T::B`，`y`都是**依赖性名称**。

\- 除此之外，如果成员访问运算符(`.`, `->`)依赖于模板参数，则`p->x`中的`x`也是**依赖性名称**；

\- `this->x`在模板定义中出现时，也是**依赖性名称**;

\- `f(A<T>)`中的`f`也是**依赖性名称**.

而他们被程序进行`name lookup`的时机也不相同：`Non-dependent names`会被程序在模板定义的时候查找，可是`dependent names`只有当**模板被实例化**时才被查找到，因为他们都是与模板参数有关的，依赖于具体的模板参数。所以，我们在模板这一节开始时提到的`name lookup`过程会在**模板定义**和**实例化**时各做一次，分别处理`Non-dependent names`与`dependent names`。这一机制被称为**Two phase name lookup(两阶段查找)**。

而正是基于这样的原因，在如下的例子中：

```c++
#include <iostream>
using namespace std;

void f(double) { cout << "Function f(double)" << endl; }

template <class A> struct container{ // point of definition of container
   void member1(){
      // This call is not template dependent, 
      // because it does not make any use of a template parameter.
      // The name is resolved at the point of definition, so f(int) is not visible.
      f(1); 
   }
   void member2(A arg);
};

void f(int) { cout << "Function f(int)" << endl; }

void h(double) { cout << "Function h(double)" << endl; }

template <class A> void container<A>::member2(A arg){ 
   // This call is template dependent, so qualified name lookup only finds
   // names visible at the point of instantiation.
   ::h(arg);  
}

template struct container<int>; // point of instantiation of container<int>

void h(int) { cout << "Function h(int)" << endl; }

int main(void){   
   container<int> test;   
   test.member1();
   test.member2(10);
   return 0;
}
```

在这个例子中，我们为`f`和`h`分别提供了两种类型的重载，并在第26行对模板进行实例化。由于文件编译顺序自上而下，所以在模板定义阶段发生的第一阶段名称查找只能找到在模板定义上方的`f`或者`h`，实例化也同样。如果更改这两个函数定义的位置，那么程序最终会输出不同的结果。

我们得到了如下的结果：

```bash
Function f(double)
Function h(double)
```

正是因为编译器分别在**模板定义**和**模板实例化**时完成了**名称查找**。在之后进行名称解析时，根本不会用到后边定义的东西。

而`name lookup`中提到，名称查找又分为`qualified name look-up`以及`unqualified name look-up`。这样一来，在模板名称定义的概念里，我们有如下四种组合：

|                | 非依赖性名称 | 依赖性名称 |
| -------------- | ------------ | ---------- |
| **非受限名称** | 1            | 2          |
| **受限名称**   | 3            | 4          |

> 基于这种特性，对于一个多文件的项目，我们必须把模板类或者模板函数的声明与实现放在一个文件中，根据[官方文档](https://www.cplusplus.com/doc/oldtutorial/templates/)：
>
> > Because templates are compiled when required, this forces a restriction for multi-file projects: the implementation (definition) of a template class or function must be in the same file as its declaration. That means that we cannot separate the interface in a separate header file, and that we must include both interface and implementation in any file that uses the templates.

##### Why use typename?

正因为这一`Two phase name lookup`的特性，`typename`关键字在模板定义内派上了用场。

```c++
template <typename T> struct X {};

template <typename T> struct Y
{   
    // X[T] is a dependent name on template parameter T
    // which will be lookup during the template instantaition
    // not the template definition period
    typedef X<T> ReboundType;                        
    typedef typename X<T>::MemberType MemberType2;    // 这里的typename是做什么的？
};
```

这里使用`typename`的目的简而言之，就是**C++**标准规定，`T::type`的形式不仅可以是一个类型，还可以是`T`的一个成员变量，**如果编译器能够在模板定义（该形式出现）时就明确知道它的类型，就不需要加`typename`；如果要等到二阶段，即模板实例化时才知道它是否合法（到底是个变量还是个类型），那么我们必须使用`typename`明确指定这是一个类型。**更多的例子参见原文章。

这个例子更清晰一些：

```c++
template<class T> class A
{
  T::x(y);
  typedef char C;
  A::C d;
}
```

根据IBM文档的解释，`T::x`可以被理解成一个**函数调用**，即`T::X()`；或者是`T::x`**类型**用`y`初始化的变量；在这种情况下，**C++编译器解释为函数调用，我们为了让编译器把它理解为`类型`，`typename`派上了用场**。最后的`A::C`这里就成了`ill-formed`，我们也需要使用`typename`来告诉编译器我们想让你如何处理它。

#### Type Computations

与`Computations on Types`相对应的是`Computations on Values`:

```c++
int s = 3; // computations on values
using S = int; // computations on types

int triple = 3 * s; // computations on values
using cl_ref = const S&; // computations on types

int result = foo(triple); // computations on values
using result = std::remove_reference<cl_ref>::type; // computations on types

bool equals = (result = 0); // computations on values
constexpr bool equals = std::is_same<result, const int>::value; // computations on types

if (equals) exit(1); // computations on values
if constexpr (equals) exit(1); // computations on types
```

我们通过将`type`传递给*meta_function*实现这一点。

> A meta-function is a "function" that operates on some **types**/values (parameters) and outputs some **types**/values (return values). Concretely, it is a **struct** that has **public member types/fields** which depend on what the **template types/values** are instantiated with.

比如：

```c++
// a value meta-function: identity
template<int V>
struct identity {
    static const int value = V;
}

// a type meta-function: identity
template<typename T>
struct identity {
    using type = T;
}
```

> 在第一个结构体中使用`static`关键字的作用是，允许我们在不实例化结构体（或者类）的时候使用其成员（他们在定义的时候就存在了），即：

```c++
int val = identity<3>::value;
```

那么这种`meta-function`有什么作用呢？

#### type_traits

我们结合`meta-function`的知识，以及先前我们对模板特例化（`template specialization`）的了解，我们便可以得知**C++**内置的`type traits`库是在做什么事情了，并且为什么我们经常遇到`::type`或者`::value`的写法。这里只需要一个例子，就可以说明问题：

```c++
// primary/generic template
template<typename T>
struct is_pointer {
    static const bool value = false;
}

// specializaed template
template<typename T>
struct is_pointer<T*> {
    static const bool value = true;
}
```

如此一来，根据`partial ordering`的规则，如果我们传入一个指针，那么`is_pointer<int*>::value`应当为`true`，如果不是指针，则无法匹配`specialized template`，会使用通例模板，最终返回`false`。

#### Examples

最后，我们来看一个例子。这里我们想写一个标准库里`iterator`的`distance`函数，大致的结构应该是这样：

```c++
template<typename It>
size_t my_distance(It first, It last) {
    // if it_category == random_access_Iterator
    // then
    // return last - first
    else {
        size_t result = 0;
        while (first != last) {
            ++first;
            ++result;
        }
        return result;
    }
}
```

现在的问题在于我们首先要找到一个办法能拿到迭代器的类型。于是我们想到了我们的`meta functions`和**C++**的`type traits`:

```c++
template<typename It>
size_t my_distance(It first, It last) {
    // notice! typename, lol we just introduced its effect
    using category = typename std::iterator_traits<It>::iterator_category;
    if (std::is_same<category, std::random_access_iterator_tag>::value) {
        return last - first;
    }
    else {
        size_t result = 0;
        while (first != last) {
            ++first;
            ++result;
        }
        return result;
    }
}
```

**但是！**这个例子目前并不完美，他无法编译通过，原因正是我们之前总结的-没有利用**SFINAE**！

在**C++17**中，很简单，我们只需做如下改写：

```c++
template<typename It>
size_t my_distance(It first, It last) {
    // notice! typename, lol we just introduced its effect
    using category = typename std::iterator_traits<It>::iterator_category;
    if constexpr (std::is_same<category, std::random_access_iterator_tag>::value) {
        return last - first;
    }
    else {
        size_t result = 0;
        while (first != last) {
            ++first;
            ++result;
        }
        return result;
    }
}
```

### Lambda Expressions

[1](https://www.cppstories.com/2020/08/lambda-generic.html/)话题的引入来自于该函数中的`predicate`，我们尝试着将它写做一个非固定最大值的版本，即将下边的5替换为一个`limit`变量：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202111140037577.png" alt="image-20211114003742461" style="zoom:50%;" />

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202111140038721.png" alt="image-20211114003814642" style="zoom:50%;" />

> call it by using function pointers

#### Pre-C++11(Functors)

在**C++11**之前，我们使用如下方法，这也是我们所说的**闭包**，之后`lambda`表达式的基础或者说内部形式：

```c++
// by value
class GreaterThan {
public:
    GreaterThan(int limit) : limit(limit) {}
    // note: this should have a const
    bool operator() (int val) const {return val >= limit};
private:
    int limit; /* the value we captured outside*/
}

// by reference
class GreaterThan {
public:
    GreaterThan(int &limit) : limit(limit) {}/* pass by reference */
    // note: this should have a const
    bool operator() (int val) const {return val >= limit};
private:
    int& limit; /* the value we captured outside, by reference*/
}
```

在**C++11**中引入了`lambda expression`:

```cpp
auto func = [capture-clause](parameters) -> return-value {
// body
};

// the simplest lambda:
[]{};
```

> - `return-value` is optional
> - can use `=` to capture all by value (**not** recommend)
> - can use `&` to capture all by reference (**not** recommend)

需要注意的是，在**C++11**中引出了一种返回值的写法：

```c++
auto identifier(argument) -> return type
```

这种写法帮助我们写出如下函数：

```c++
template<typename T1, typename T2>
auto compose(T1 a, T2 b) -> decltype(a + b);
```

而在**C++14**中，我们可以直接省略`->return type`。

> 在**C++11**中规定，如果`lambda`函数体内部出现了除了`return`额外的语句，则`lambda`对象默认返回**void**，所以我们需要`-> return type`指定返回类型.

#### C++14

1. can use `auto` as the parameters to templatize the lambda–**generic lambda**

```CPP
auto p = [](auto a) {cout << a << endl; };
```

1. can capture with an initialiser–`z = x + y`

#### C++17

1. use `constexpr` :

```cpp
constexpr auto Square = [](int n) { return n * n; };
```

2. can capture `*this`

#### C++20

We can pass a template tail:

```cpp
auto ForwardToTestFunc = []<typename ...T>(T&& ...args) {
  return TestFunc(std::forward<T>(args)...);
};
```

同时对于**未捕获值**的`lambda`对象，拥有了**默认构造函数和赋值运算符(不再被声明为`delete`)**.

#### mutable

让我们把注意力移回到**C++11**之前的版本，严格来说，成员函数`operator()`的重载应当设置一个**指向常量对象的常量指针**，即写为：

```cpp
void operator()(...) const {
    // since there is a const,
    // if captured by reference, we can change the values captured
    // if captured by value, we cannot change it.
}
```

所以关键字`mutable`允许我们改变捕获的值：

```cpp
class a {
  
public:
  a(int x1): x(x1) {}
  mutable int x;
  
  bool changeX(int y) const {
    x = y;
  }
};
```

关于关键字`mutable`的用法，其只能作用于非常量和非静态数据成员上（因为静态数据成员不属于类自己）。

#### globals & statics

`lambda`表达式**仅允许捕捉当前函数中定义的具有`automatic storage duration`的变量**，所以我们不能够捕捉`global/static variables`，对于定义在当前函数之外的名字，`lambda`可以直接使用。

##### storage duration

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202111142007243.png" alt="image-20211114200707114" style="zoom:50%;" />

需要注意的是，对于具有`static`生命周期的变量，还包括我们使用`static`关键字定义的变量。**即使我们将`static variable`定义到了函数中，他的生命周期也是直至程序结束，只不过在函数栈退出后，我们无法获取这个定义的变量了.** 它还包括一种特殊情况，就是**字符串字面值(`string literals`)**：

```c
// a string literal
char *str = "ABCDEFG";
// NOT a string literal -- 储存在栈上
char str[] = "ABCDEFG";
```

这三种不同的生命周期的变量，在其后的*CS 61C*课程中我们会了解到，其实是对应着储存在三个不同的存储器区域：`static`,`stack`,`heap`.

#### type of lambda?

需要注意的是，`lambda expression`最前边的`auto`并不是`lambda`的返回值，而是**函数类型**。所以我们是否能够不使用`auto`或者说在模板中为了得到`lambda`的类型使用`decltype`关键字？

在**C++20**之前，答案是不行。从定义上来说，`lambda`表达式的类型是闭包类型（`closure type`），如果我们尝试使用`typeid`关键字获取其类型，则即使是同一个类型的`lambda`对象，每一个`lambda`对象的`typeid`值也是不同的：

> **The type of the *lambda-expression*** (which is also the type of the closure object) **is a unique, unnamed non-union class type** — called the *closure type*.

这是由于`lambda`被编译器翻译为**未命名类的未命名对象**(*C++ Primer P508*)，即使定义一样，编译器也会认为每个匿名类名都不一样。

**该匿名类的默认构造函数被设置为`delete`，并且赋值函数也被禁用，但是我们可以调用其拷贝构造函数(是否含有默认的拷贝构造/移动构造函数通常要视捕获的数据成员类型而定)**：

```c++
auto f = [](int x){ return x; };
decltype(f) g = f;
```

需要注意的是，C++允许使用`decltype`提取已经计算的`lambda`表达式，如上所示；但是不允许提取未计算的`lambda`表达式类型：

```c++
decltype([](int x){ return x; })    // error
```

不过自**C++20**起，**未捕获值的`lambda`表达式拥有了默认构造函数和赋值运算符**。



当**捕获列表为空**时，我们可以将`lambda`表达式赋值给一个拥有相同函数签名的函数指针，**相应的函数指针指向的是lambda表达式内部的一个static成员函数**：

```c++
int(* f)(int, int) = [](int a, int b){ return a + b; };
```

但是当存在捕获值时，便无法进行这种隐式转换，因为捕获对象的实体是以类成员的形式存在的，对象整体上形成一个`functor`。

#### Pass Lambda As Argument?

所以接下来要思考的一个问题是，如何才能把一个`lambda`表达式作为参数传进函数？

根据先前的讨论，当`lambda`捕获值时，我们无法执行到函数指针的隐式转换。同时此时由于我们无法确定`lambda`表达式的具体类型是什么，意味着使用**继承**，即**运行期多态**让`lambda`对象进行动态绑定不可行。那么我们只有使用**模板**，作为一种**编译器多态**，可以将`lambda`表达式传入函数当中。这样做的一个好处是编译器可以在编译期将`lambda`表达式**内联**进调用函数内部，加快运行速度：

```c++
template< class ForwardIt, class T >
constexpr std::pair<ForwardIt,ForwardIt>
              equal_range( ForwardIt first, ForwardIt last,
                           const T& value );
```

##### std::function

而在**C++11**中，提出了`std::function`，它的存在使得几个可调用对象共享同一种调用形式成为可能。

```c++
// normal functions
int add(int i, int j) { return i + j; }
// lambda
auto mod = [](int i, int j) { return i % j; };
// function object class
struct divide {
    int operator()(int denominator, int divisor) {
        return denominator / divisor;
    }
}
```

我们也可以利用`std::function`来把`lambda`表达式传入函数中：

```c++
std::function<double(double, double)> f_mul = [a](double x, double y) { return x * y + a; };

double calculator(double a, double b, std::function<double(double, double)> fn)
{
	return fn(a, b);
}
```

> Class template `std::function` is a general-purpose polymorphic function wrapper. Instances of `std::function` can store, copy, and invoke any `CopyConstructible` `Callable` target – functions, lambda expressions, bind expressions, or other function objects, as well as pointers to member functions and pointers to data members.

### std::bind

除了`lambda expression`之外，我们还可以使用头文件`<functional>`中的`bind`方法来写先前我们所需要的`predicate`函数,根据[官方文档](https://en.cppreference.com/w/cpp/utility/functional/bind)给出的示例，我们可以结合占位符`std::placeholders`来使用该函数：

```cpp
using namespace std::placeholders;  // for _1, _2, _3...

std::cout << "1) argument reordering and pass-by-reference: ";
int n = 7;
// (_1 and _2 are from std::placeholders, and represent future
// arguments that will be passed to f1)
auto f1 = std::bind(f, _2, 42, _1, std::cref(n), n);
n = 10;
f1(1, 2, 1001); // 1 is bound by _1, 2 is bound by _2, 1001 is unused
                // makes a call to f(2, 42, 1, n, 7)
```

在该段示例代码中，`std::cref(n)`的作用是为`n`创建一个`reference_wrapper`的对象，与之有相同作用的还有函数`std::ref(n)`：

>`std::reference_wrapper` is a class template that wraps a reference in a copyable, assignable object. It is frequently used as a mechanism to store references inside standard containers (like [std::vector](https://en.cppreference.com/w/cpp/container/vector "cpp/container/vector")) which cannot normally hold references.

放到上边的例子中，就是让bind函数按照引用传递n。

比如，我们可以使用如下语法：

```c++
std::vector<std::reference_wrapper<int>> v(l.begin(), l.end());
```

如此一来，`v`内所有元素均是`vector l`内元素的引用。

`bind()`函数会给原函数创建一个`call wrapper`，调用`wrapper`函数相当于`invoke f with some of its arguments bound to args`.值得注意的是，`bind`函数声明也是通过**可变函数参数**实现的。

### Variadic template

#### Variadic arguments

[1](https://eli.thegreenplace.net/2014/variadic-templates-in-c/)在**C++11**之前，如果我们想让一个函数接受任意参数，那么我们需要使用`...`的语法配合`va_`系列的宏来使用，[文档](https://en.cppreference.com/w/cpp/utility/variadic)给出了一个例子：

```cpp
#include <iostream>
#include <cstdarg>
 
void simple_printf(const char* fmt...) // C-style "const char* fmt, ..." is also valid
{
    va_list args;
    va_start(args, fmt);
 
    while (*fmt != '\0') {
        if (*fmt == 'd') {
            int i = va_arg(args, int);
            std::cout << i << '\n';
        } else if (*fmt == 'c') {
            // note automatic conversion to integral type
            int c = va_arg(args, int);
            std::cout << static_cast<char>(c) << '\n';
        } else if (*fmt == 'f') {
            double d = va_arg(args, double);
            std::cout << d << '\n';
        }
        ++fmt;
    }
 
    va_end(args);
}
 
int main()
{
    simple_printf("dcff", 3, 'a', 1.999, 42.5); 
}
```

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202111141728044.png" alt="image-20211114172837870" style="zoom:50%;" />

我们首先建立一个`va_list`的对象用于储存余下几个宏需要的信息，之后调用`va_start`，允许程序访问后随具名参数`parm_n`的可变参数：

```cpp
void va_start( std::va_list ap, parm_n );
```

之后每当我们调用一次`va_arg`，就可以获取`va_list`内的下一个参数；最终调用`va_end`终止这一过程。

需要注意的是，在使用`variadic arguments`时，`...`必须跟随在参数列表之后，而不允许被放到参数列表的前边。但是在**C++**中，这样的使用方式是被允许的：

```cpp
int printz(...)
```

这种使用方式在模板重载中被作为`SFINAE`的`fallback overload`使用，因为`...`最不`specialized`，所以在`overload resolution`中具有最低的优先级。

> fallback is a function that does not take any arguments and does not return anything.

#### Parameter pack

上一节中提到的`variadic argument`与`parameter pack`不一样：

> Note: this is different from a function [parameter pack](https://en.cppreference.com/w/cpp/language/parameter_pack) expansion, which is indicated by an ellipsis that is a part of a parameter declarator, rather than an ellipsis that appears after all parameter declarations. Both parameter pack expansion and the "variadic" ellipsis may appear in the declaration of a function template, as in the case of [std::is_function](https://en.cppreference.com/w/cpp/types/is_function).

换而言之：

```cpp
// variadic arguments
int printx(const char* fmt...);
// parameter pack
void foo(const T &t, const Args&... rest);
```

`parameter pack`是构成`variadic template`的基础：

> A template with at least one parameter pack is called a *variadic template*.

共有两种参数包：**模板参数包**与**函数参数包**：

```cpp
template <typename T, typename... Args> // a template parameter packet
void foo(const T &t, const Args&... rest); // a function parameter packet
```

我们如何理解上边的语句？可以举例来想，**模板参数包**中将各种不同的可能出现的类型（`int, double, char* ...`）打包，之后把这个包交给函数，函数对包`Args`做**包扩展（`packet expansion`）**，传入的**每一种类型**都可能存在多个元素，比如`double a, double b...`，于是对每一种类型（即模板参数包中的每一个对应元素）打包、组合成一个名为`rest`的**函数参数包**。

> 需要注意的是，对于`class template`，参数包只能出现在最后一个参数的位置上。但是对于`function template`，参数包可以出现在任意位置上。

#### Packet expansion

在上一节中，提到了包扩展：*pattern…*，**扩展一个包就是将他分解为构成的元素，并对每个元素应用模式`pattern`**:

```cpp
template<class ...Us> void f(Us... pargs) {}
template<class ...Ts> void g(Ts... args) {
    f(&args...); // “&args...” is a pack expansion
                 // “&args” is its pattern
}
g(1, 0.2, "a"); // Ts... args expand to int E1, double E2, const char* E3
                // &args... expands to &E1, &E2, &E3
                // Us... pargs expand to int* E1, double* E2, const char** E3
```

包扩展可以出现在函数调用符`()`内部，根据[官方文档](https://en.cppreference.com/w/cpp/language/parameter_pack#Pack_expansion)：

```cpp
f(&args...); // expands to f(&E1, &E2, &E3)
f(n, ++args...); // expands to f(n, ++E1, ++E2, ++E3);
f(++args..., n); // expands to f(++E1, ++E2, ++E3, n);
f(const_cast<const Args*>(&args)...);
// f(const_cast<const E1*>(&X1), const_cast<const E2*>(&X2), const_cast<const E3*>(&X3))
f(h(args...) + args...); // expands to 
// f(h(E1,E2,E3) + E1, h(E1,E2,E3) + E2, h(E1,E2,E3) + E3)
```

一个包扩展的例子可参见*C++ Primer* P621，需要注意的是，在例中进行函数调用时的包扩展，先将包中第一个参数剥离（`peel off`），对应了`const T &t`,余下的元素构成了新的参数包，并递归调用，直到我们遇到了`base case`，并使用对应了非可变参数函数模板：

```c++
// base case function
template<typename T>
ostream &print(ostream &os, const T &t) {
    return os << t;
}

template<typename T, typename... Args>
// the first packet expansion, using pattern `const Args&`
ostream& print(ostream &os, const T &t, const Args&... rest) {
    os << t << ", ";
    // the second packet expansion, using pattern `rest`
    // then peel off the first element in the expansion list to be `const T &t`
    // recursion with the list with other elements
    return print(os, rest...);
}
```

#### `sizeof...`

我们可以通过使用`sizeof...()`得到包中参数的数目。

#### Performance

摘自：https://eli.thegreenplace.net/2014/variadic-templates-in-c/

> If you're concerned with the performance of code that relies on variadic templates, worry not. As there's no actual recursion involved, all we have is a sequence of function calls pre-generated at compile-time. This sequence is, in practice, fairly short (variadic calls with more than 5-6 arguments are rare). Since modern compilers are aggressively inlining code, it's likely to end up being compiled to machine code that has absolutely no function calls. What you end up with, actually, is not unlike loop unrolling.
>
> **Compared to the C-style variadic functions, this is a marked win, because C-style variadic arguments have to be resolved at runtime. The `va_` macros are literally manipulating the runtime stack.** Therefore, variadic templates are often a performance optimization for variadic functions.

### range

在**C++20**中，提出了`range`的概念，写了一个`range`库，其中包含`range`, `view`, `range adaptor`等……但这里不打算深究，具体详见官方文档以及[这篇文章](https://www.zhihu.com/column/p/86809598)。

## Object-Oriented Programming

### Const

几种不同的`const`含义：

```c++
// 指向整型常量的指针(可修改指针本身，不可修改指针指向的对象)
const int* a;
// or:
int const* a;

// 指向整形的常量指针(不可修改指针本身，可修改指针指向的对象)
int* const a;

// 根据《C++ Primer》，this指针默认为指向非常量对象的常量指针，
// 此操作将this指针设定为指向常量对象的常量指针
void func() const
```

#### Why initialization-list?

加入我们将对象定义为`const`类型，那么在声明之后，对象就不可以再改变了，这意味着一般的构造函数初始化方法会出现问题（先建立对象，再挨个初始化其元素）。而`initialization list`在建立对象的同时立即初始化其所有元素：

```c++
test(int num1, double str1): num(num1), str(str) {};
```

我们也可以直接使用大括号初始化列表（`brace-init-list`），编译器也会为我们选择合适的对应的构造函数（如果需要的话）：

```c++
class foo {
public:
    foo(size_t size): 
        size{size},
        vec{size, nullptr} {}
private:
    vector<int*> vec;
    size_t size;
};
```

### friend

**相同class的各个objects互为友元**：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202111181413064.png" alt="img" style="zoom:50%;" />

#### Mutual friends class

在完成`HashMap`作业的过程中，出现了`Container`与`Iterator`类需要互为友元的问题，这一问题在**模板**的章节中已有提到，但是对于这种特殊的`Mutual friends class`行为并未作出过多的解释。

当`A`, `B`两个类互为友元时，我们称这两个类为*Mutual Friends Classes*。为了解决这个问题，我们首先要知道，当一个类仅有**声明**，而没有**定义**时，类为不完整类型。我们仅能够使用指向该类的指针、引用或者将类作为形参和返回值出现在函数声明中。

##### one file

当这两个类位于同一文件中时：

1. 作类`B`的**前置声明**（`forward declaration`），告知类`A`存在一个名为`B`的类；
2. 定义类`A`，在类`A`中将类`B`声明为友元，对于需要用到类`B`的函数，仅提供**函数声明**，因为此时我们还不知道`B`的具体定义；
3. 定义类`B`，提供完整定义；
4. 将类`A`中需要调用`B`的方法**定义**补充完整

```c++
// Forward Declaration
class B;
 
// Class A
class A {
 
    // Member of class A
    int data;
 
public:
    // Make B as a friend of class A
    friend class B;
 
    // Constructor to initialise member
    // of class A
    A(int d) { data = d; }
 
    // Function to get data of friend
    // class B
    void get_data_B(B objb);
};
 
// Class B
class B {
 
    // Member of class B
    int dataB;
 
public:
    // Making A a friend of class B
    friend class A;
 
    // Constructor to initialise member of
    // class B
    B(int d) { dataB = d; }
 
    // Function to get the data of
    // friend class A
    void get_data_A(A obja)
    {
        cout << "Data of A is: "
             << obja.data;
    }
};
 
// Function for accessing friend class
// B's object
void A::get_data_B(B objb)
{
    cout << "Data of B is: "
         << objb.dataB;
}
```

##### separate file

当类`A`与类`B`分别定义在不同的文件中时，我们需要注意应当避免出现**头文件递归包含**额情况，也就是说我们不可以在`A.h`中包含了`B.h`的同时反过来也同时包含另一个文件。

假定我们现在`B.h`包含了`A.h`，那么在文件`A.h`中，我们应当使用先前提到的策略：做一个类`B`的**前置声明**，并且同样地**不可以在类`A`内部提供类`B`的内部信息**：

- A.h

```c++
/* This is called a "forward declaration".  We use it to tell the compiler that
   the identifier "B" will from now on stand for a class, and this class will be
   defined later.  We will not be able to make any use of "B" before it has been
   defined, but we will at least be able to declare pointers to it. */
class B;

class A
{
    /* We cannot have a field of type "B" here, because it has not yet been
       defined. However, with the forward declaration we have told the compiler
       that "B" is a class, so we can at least have a field which is a pointer 
       to "B". */
    B* pb; 
}
```

- B.h

```c++
#include "A.h"

class B
{
   /* the compiler now knows the size of "A", so we can have a field
      of type "A". */
   A a;
}
```

##### template mutual friends

在*hashmap*的作业里，我们看到了这种用法：两个文件`hashmap.h`与`hashmap_iterator.h`中，前一个文件对后一个文件，即提供了迭代器类的文件做了包含，但是在`hashmap_iterator.h`中我们发现，即使不提供`HashMap`的前置声明，运行也不会报错，这是为什么呢？

这是因为在`hashmap_iterator.h`中，我们对`iterator`做了模板类的定义，在模板参数中包含了`HashMap`类型，这允许我们直接通过作用于运算符`::`来获取（不需要提供其他任何参数，因为在`hashmap_iterator`模板类被**实例化**之后，我们便得到了其模板参数`HashMap`的**具体类型**）：

1. `HashMap`中的类型别名定义
2. `HashMap`的静态成员

于是这里出现一个问题，如果我们要用到`HashMap`中的非静态成员变量、函数怎么办呢？这些内容我们无法直接通过`::`的方法获取，而因为我们是把`container class name`放到了`template parameter`中更是无法直接在此类中给出实例化类名的代码从而通过成员访问运算符获取非静态内容。

我们可以通过如下手段解决此问题：首先把需要使用的容器非静态类成员转化成使用类型别名表示的形式，之后在友元类中使用`::`获取其类型，定义一个该类型的指针，并将指针初始化指向容器成员：

```c++
// friend class, class type foo as template parameter
template<typename foo>
class foo_1 {
    // since foo will be a complete type even if it's a template class,
    // we can directly write 'foo'
    friend foo;
    // the type of vector, defined in the hashmap class
    // since it's a type, we can directly use :: to get it here!
    using bucket_array_type = foo::bucket_array_type;
    
    // thyen, make a pointer!
    bucket_array_type* _buckets_array;
    
    // in iterator class, we shall use private constructor
    // iterator's constructor, we will use it in the 
    // hashmap class
    foo_1(bucket_array_type* _buckets_array):
    	_buckets_array(_buckets_array) {}
    
public:
	// some member functions
};

class foo {
private:
    vector<node*> _buckets_array;
    
    using bucket_array_type = decltype(_buckets_array);
    // foo_1's template specialization of <foo> type
    friend class foo_1<foo>;
    
public:
    foo(/* */);
    foo_1 begin() {
        // call iterator constructor here
        /** give the address of the vector to initialize the
         * pointer pointing to vector<> type in the iterator
         * class */
        return {&_buckets_array};
    }
};


```

经过类似的操作，我们便可以获得指向这些成员类型的**指针**，比如对于原容器中的`vector`，我们可以使用`decltype(vector_name)`的方式获取该容器的**类型**，而对于`vector<>`类型来说，指向它的指针其实就像我们所使用的最最普通的数组一样，是`&array_name`的形式！

我们在构造函数中将这个**指针指向类`foo`中对应的成员**（用`foo`的成员初始化它们），也即`_buckets_array`。这样一来，我们便可以在友元类中对另一个类中的成员进行操作了。

### Operators

基本的用例略过，这里记录课件中几点关键的原则：

#### General rule of thumb

1. Some operators must be implemented as members (
   eg. [], (), ->, =) due to C++ semantics.

> 因为成员函数隐藏的第一个参数一定是`this`，即对象的地址，而使用这些操作符时我们无法在别处提供对象地址.

2. Some must be implemented as non-members (eg. <<,
   if you are writing class for rhs, not lhs).

> 因为第一个参数要是`ostream& os`，所以必不可能为成员函数.

3. If unary operator (eg. ++), implement as member.

4. If binary operator and treats both operands equally (eg.
   both unchanged) implement as non-member (maybe
   friend). Examples: +, <.

> **根据官方文档：**
>
> Binary operators are typically implemented as non-members to maintain symmetry (for example, when adding a complex number and an integer, if `operator+` is a member function of the complex type, then only `complex+integer` would compile, and not `integer+complex`). 

何为`symmetric`?

> 根据**Wikipedia**：
>
> A corresponding property exists for binary relations; a binary relation is said to be **symmetric** if the relation applies **regardless of the order of its operands**.

因为定义为非成员函数，所以需要友元，以下是最习惯的一种重载二元运算符的方式：

```cpp
// friends defined inside class body are inline and are hidden from non-ADL lookup
  friend X operator+(X lhs,        // passing lhs by value helps optimize chained a+b+c
                     const X& rhs) // otherwise, both parameters may be const references
  {
    lhs += rhs; // reuse compound assignment
    return lhs; // return the result by value (uses move constructor)
  }
```

需要注意的是，当我们在类模板中定义友元函数时，我们**还需要为友元函数单独提供一份模板参数，而不能直接利用类模板参数**–因为友元本身并不属于类成员的一部分：

```c++
template<typename K, typename M, typename H=std::hash<K>>
class foo {
    // do something
    template<typename K_, typename M_, typename H_>
    friend std::ostream& operator<<(std::ostream& os, const foo<K_, M_, H_>& foo_1);
};
```

5. If binary operator and not both equally (changes lhs),
   implement as member (allows easy access to lhs private
   members). Examples: +=

```cpp
  X& operator+=(const X& rhs) // compound assignment (does not need to be a member,
  {                           // but often is, to modify the private members)
    /* addition of rhs to *this takes place here */
    return *this; // return the result by reference
  }
```

#### ->

在*HashMap*的作业中，我们需要在`iterator`类中对**成员访问操作符**进行重载。该运算符的重载可能看起来有些奇怪，实际上，运算符重载的结果不是简单的*替换原则*，比如我们可能认为`obj->value`重载替换之后应当被理解为`ptrvalue`（`ptr`是私有指针成员），但实际上该运算符的重载结果为：

```c++
(obj.operator->())->value;
```

`obj`是一个类对象，它使用运算符重载函数`operator->()`之后，我们知道一般是要返回一个**指针**（比如在迭代器的实现中，是利用迭代器内置指针指向对应的容器中的数据的一个指针），之后程序会**在该运算符重载函数返回值上调用指针自己的`->`运算符来获取指向的数据成员**。

#### []

一般来说，我们需要重载`operator []`的两种形式（`const`与非`const`），这是为了解决当我们定义的对象本身即为`const`类型时可能出现的问题：

由于`this`指针默认是一个指向非常量对象的常量指针，如果对象被声明为`const`，则指`this`为指向常量对象的常量指针，只有在我们将成员函数后加上`const`，才可以使得参数中实参类型为`const test*`，否则势必**丢失底层`const`**。（这也是为什么`const`成员函数内只允许调用`const`成员函数的原因）。同时需要注意，该函数可以返回引用，引用前也应当有`const`，确保不能改变对象：

```cpp
const string& vector<string>::operator[](size_t index) const {
    // do something
}
```

#### Principle of Least Astonishment (POLA)

这里只提几点：当我们实现了`+`的重载时，我们最好也要实现类似于`+=`的重载，类推:

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202111172313712.png" alt="image-20211117231351569" style="zoom:50%;" />

同时，我们也要注意能够链式使用重载运算符的计算结果（靠运算符重载函数返回对象或者对象的引用来实现）:

```cpp
ostream& operator<<(ostream& out, const Fraction& f) {
    out << f.num << “/”<< f.denom;
    return os;
}
```

#### Converting constructor & Conversion operator

##### Converting constructor

> 如果~~构造函数**可以只接受一个实参**（包括对其余所有形参提供了默认值的情况）~~，则它实际上定义了由实参类型转换为类类型的隐式转换机制。这种构造函数被称为**转换构造函数**。

仅接受一个实参的规定仅在*C++11*之前适用。在*C++11*之后，`converting constructor`的定义变为：

> 没有由`explicit`关键字修饰的构造函数

> *C++ Primer*P264:
>
> 编译器**只会自动执行一步隐式转换**，也就是说如果我们接受一个`string`类型的参数，那么我们不能直接把`char*`放进去，否则自动转换为`string`后，不会继续隐式转换了.(但我们可以做显式转换):

```c++
class test_explicit {
public:
 test_explicit(string text): text(text) {}
 string combine(const test_explicit& new_text) {
     return text + new_text.text;
 }
private:
 string text;
};
// OK, implicit conversion:
string new_value = "9999";
cout << "Combine value: " << foo_1.combine(new_value) << endl;

// error, only permit one conversion:
cout << "Combine value: " << foo_1.combine("9999") << endl;
```

我们可以通过关键字`explicit`来抑制构造函数定义的隐式转换:

```c++
class test_explicit {
public:
    explicit test_explicit(int i): data(i) {}
    int combine(const test_explicit& new_value) {
        return data + new_value.data;
    }
private:
    int data;
};

// error, try to converting implicitly from int to test_explicit type
string new_value = "9999";
cout << "Combine value: " << foo_1.combine(new_value) << endl;
```

需要多个实参的构造函数不能用于执行隐式转换，所以无需将其定义为`explicit`的。同时，只能在类内声明构造函数时使用`explicit`关键字，在类外部定义时不应重复。

> 注意，当拷贝构造函数被声明为`explicit`时，表明拷贝构造函数不能被隐式调用，所以，如果我们尝试使用`=`进行拷贝初始化（**隐式调用了拷贝构造函数**），也是不被允许的:

```c++
string null_book = "9999";
Sales_data item2 = null_book; //error
```

`explicit`可以避免我们写出二义性的代码。

##### Conversion operator

`explicit`关键字除了可以用在构造函数之前外，还可以用在类型转换运算符前：

```c++
explicit operator double() const {
    // do something
}
```

类型转换运算符是类的一种特殊成员函数，一般形式便如(因为一般不改变原对象的内容，所以声明为`const`)：

```c++
operator type() const;
```

一个可能借此机会发生隐式类型转换的例子（*C++ Primer P516*）是：

```c++
int i = 42;
cin << i;
```

如果这里`cin`的`operator bool() const`没有被声明为`explicit`，那么由于`cin`并没有`<<`重载，编译器会尝试将`cin`转化为一个`bool`，之后将`<<`理解为一个移位运算符！

> 使用了`explicit`后，该规定存在一个例外，**如果表达式被用作条件，则编译器会将显式的类型转换自动应用于它**。`operator bool`一般定义成`explicit`的.

### Special member functions

这些函数会由编译器自动生成一份：

- **Default construction**
- **Copy construction**
- **Copy assignment**
- **Destruction**

#### Most vexing parse

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202111180150966.png" alt="image-20211118015039848" style="zoom:50%;" />

根据*Wikipedia*的解释：

> The **most vexing parse** is a counterintuitive form of syntactic [ambiguity resolution](https://en.wikipedia.org/wiki/Ambiguous_grammar) in the [C++](https://en.wikipedia.org/wiki/C%2B%2B) programming language. In certain situations, the C++ grammar cannot distinguish between the [creation](https://en.wikipedia.org/wiki/Initialization_(programming)) of an object [parameter](https://en.wikipedia.org/wiki/Parameter_(computer_programming)) and [specification of a function's type](https://en.wikipedia.org/wiki/Type_declaration). In those situations, the compiler is required to interpret the line as a function type specification.

简而言之，编译器不能判断这是一个初始化操作还是在调用函数，在这种情况下，他被当作一个函数来处理了。**所以不要这么写！**我们可以使用C++11中的`Uniform initialization`（`a{x}`）来规避歧义。

需要注意的是，上图中在函数结尾返回对象时调用的即是**copy constructor**.

#### shallow copy & deep copy

编译器在默认情况下生成的`copy assignment`就是一种`shallow copy`，而我们需要的往往是`deep copy`，`shallow copy`带来的问题很明显，它并没有完全复制所有的数据，而是用指针指向了源数据存放的位置：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202111180155717.png" alt="image-20211118015546603" style="zoom:50%;" />

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202111180156728.png" alt="image-20211118015632623" style="zoom:50%;" />

#### Copy constructor

```cpp
StringVector::StringVector(const StringVector& other) :
        _logicalSize(other._logicalSize),
        _allocatedSize(other._allocatedSize) {
            
    _elems = new ValueType[_allocatedSize];
    std::copy(other.begin(), other.end(), begin());
}
```

#### Copy assignment

需要注意的是，在`copy assignment`中，我们要考虑到`copy`的对象与源对象是同一个的情况（`self-assignment`）:

```c++
StringVector& StringVector::operator=(const StringVector& rhs) {
    if (this != &rhs) {
        delete [] _elems;
        _logicalSize = rhs._logicalSize;
        _allocatedSize = rhs._allocatedSize;
        _elems = new ValueType[_allocatedSize];
        std::copy(other.begin(), other.end(), begin());
    }
    return *this;
}
```

> 不可以用`*this != rhs`，因为这需要我们重载`!=`操作符.

#### =delete & =default

如果我们的类不需要拷贝/移动操作，我们需要在`public`域中使用`=delete`将其禁用（但`=delete`关键字并非只适用于这几种函数，该关键字意为“弃用”）：

```c++
test(const test&) =delete;
test& operator=(const test&) =delete;
```

如果我们需要默认的行为，可以要求编译器提供生成默认构造函数（一般此时我们还需要其它类型的构造函数）

```c++
test() = default;
```

#### Write our own?

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202111180206534.png" alt="image-20211118020633426" style="zoom:50%;" />

需要注意的是，`stream`对象均不可被复制，因为这样的操作没有意义，而在实现中，他们的复制构造函数也都被声明为`private`。

#### Rule Of Three

> If you explicitly define (or delete) a copy constructor, copy assignment, or destructor, you should define (or delete) all three.

因为当定义了这三种函数的任意一种时，就表明会出现`ownership issues`.

#### Rules of Zero

> If the default operations work, then don’t define your own custom ones.

### Delegating Constructor

**C++11**引入了**委托构造函数**–使用它所属类的其他构造函数执行他自己的初始化过程，或者将自己的职责委托给其他构造函数：

```c++
class Sales_data {
public:
    Sales_data(): Sales_data("", 0, 0) {}
    // other constructors
}
```

### inline & constexpr

内联函数（`inline`），在编译过程中**内联**地在调用点展开，从而消除函数在运行时的开销。inline本义是将所调用函数用自身的函数本体替换之，免受函数调用所招致的额外开销，比宏还要不易出错；但是实际上inline的受编译器的控制，编译器根据执行语境来对inline函数是否做优化，**inline只是对编译器的申请，不是强制命令。**(如果函数体比较大，用inline关键字可能导致编译产生的目标文件过大，导致额外的换页行为，降低CPU高速缓存的命中率，效率有损失；当然如果inline函数本体很小，还可能导致更小的目标文件和更高的CPU SRAM的命中率).

> 一般来说，内联函数用于优化规模较小，流程直接，频繁调用的函数。**定义在类内部的函数是自动`inline`的**。

`constexpr`函数是指能用于常量表达式的函数：它的返回类型、所有的形参类型都是**字面值类型**，而且函数体中除了`using/typedef`等在**运行时不执行任何操作**的语句外，只能有一条`return`语句：

```c++
constexpr int new_sz() {return 42;}
constexpr int foo = new_sz(); // foo is a const expression
```

> 算术类型、引用和指针（初始值必须为`nullptr`或者`0`，或者存储于某个固定地址的对象–一般定义于函数体之外）类型均为字面值类型，自定义类、IO库、`string`类型不是字面值类型，不可以被定义为`constexpr`。

我们允许`constexpr`函数的返回值并非一个常量：当给入函数的实参是常量表达式时，返回值也是常量表达式，反之不然：

```c++
constexpr size_t scale(size_t cnt) { return new_sz() * cnt;}
// OK
int arr[scale(2)];
// ERROR
int i = 2;
int arr[scale(i)];
```

编译器对`constexpr`函数的调用替换为结果值，为了能在编译中随时展开，`constexpr`函数被隐式指定为`inline`函数。

> 正因为`inline`与`constexpr`函数的编译展开特性，仅仅有函数声明是不够的，还需要函数的定义–所以他们的**定义通常直接放在头文件中**。

### Move semantics

#### emplace_back

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202111182220012.png" alt="image-20211118222054939" style="zoom:50%;" />

该函数的参数声明是用了可变参数模板，并对参数包`Args`进行了`pattern=Args&&`的`pack expansion`操作。与`push_back`方法所不同的是，该方法借助**右值引用**，也就是我们接下来要谈的特性，避免了先创建一份数据，再复制的操作，而是直接给定需要的数据元素参数，加入数据队列中：

```c++
std::vector<President> elections;
std::cout << "emplace_back:\n";
auto& ref = elections.emplace_back("Nelson Mandela", "South Africa", 1994);
assert(ref.year == 1994 && "uses a reference to the created object (C++17)");
 
std::vector<President> reElections;
std::cout << "\npush_back:\n";
reElections.push_back(President("Franklin Delano Roosevelt", "the USA", 1936));
```

#### Without copy elision

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202111182246321.png" alt="image-20211118224658231" style="zoom:50%;" />

在上图所示的代码中，需要注意的是，在不进行RVO等优化机制时，`return`这一步会创建一个`temporary object`，这需要我们调用复制构造函数，将变量`names`复制到临时对象中。

#### Basic idea

根据[这篇文章](http://thbecker.net/articles/rvalue_references/section_02.html)，为了提高运行效率，我们尝试使用某种方法避免一些不必要的数据复制过程，比如当给到这样一个拷贝赋值函数：

```c++
X& X::operator=(X const & rhs)
{
  // [...]
  // Make a clone of what rhs.m_pResource refers to.
  // Destruct the resource that m_pResource refers to. 
  // Attach the clone to m_pResource.
  // [...]
}
```

现在，假设`X`被如下使用：

```c++
x foo();
// do something
x = foo();
```

那么相比起在先前的函数体描述中的方法，是否存在一种更为高效的方法让`X`获取当前`foo()`的内容呢？我们想如果可以直接将指向`X`与指向`temporary object`的指针交换一下(即让`X`拿到这个行将销毁的临时对象的控制权，而不是想着再去复制一份给自己用)，不是更为高效嘛？于是我们有了一个期望的函数模板：

```c++
X& X::operator=(<mystery type> rhs)
{
  // [...]
  // swap this->m_pResource and rhs.m_pResource
  // [...]  
}
```

我们需要一种新的`mystery type`，表示传入了一个**右值**（因为打算直接利用临时对象），并且可以让程序在编译阶段识别出到底是应该使用这个新的重载拷贝赋值函数还是先前的传入左值引用的那一个。因为我们此处想要实现的也是一种`copy assignment overload`，所以传入的也要是一种**引用**类型，并且传入的要是一个**右值**，故`rvalue reference(&&)`被引入作为`mystery type`，我们期望该类型具备如下的行为特征：

> rvalues must prefer the mystery type, while lvalues must prefer the ordinary reference.

程序对于这两种成员重载函数（传入左值或者右值）的选择通过`overlaod resolution`实现：

> Rvalue references allow a function to branch at compile time (via overload resolution) on the condition "Am I being called on an lvalue or an rvalue?"

如果说我们传入的是左值，那么程序仍然会按照之前的重载函数运行（包含复制等一系列操作），我们不能偷取左值中存放的内容，因为之后可能还会用到；但如果我们传入的是右值，因为右值表达式行将销毁。

基于上述特性，我们想**把等式右边的内容偷过来，或者说我们想让等式左边的内容的指针指向等式右边的右值的内容，而不是把右值的内容复制到左边一份**。这种操作（想法）被叫做`move semantics`。

完整的来说，我们希望移动赋值函数具有如下定义：

```c++
X& X::operator=(X&& rhs)
{

  // Perform a cleanup that takes care of at least those parts of the
  // destructor that have side effects. Be sure to leave the object
  // in a destructible and assignable state.

  // Move semantics: exchange content between this and rhs
  
  return *this;
}
```

先把等式左边的内容清空，再把等式右边的内容拿过来，这也符合我们对**赋值**的作用定义。并且通过这个函数伪代码可以看出，当我们在等式右边给一个右值之后，就是告诉程序我们打算做`move semantics`了，具体如何去`move`，如何`cleanup`，如何交换控制权（`ownership`），是由我们自己所定义的行为（e.g. 一个类移动构造函数）决定的。

> `std::move`正是通过强制将左值转换为右值的方法，使得外部函数可以调用如上所示的右值重载，使得等式左边的对象获得给入的右对象的实际控制权：

```c++
struct A {
    A(A&& a) {
        this->data = a->data;
        a->data = nullptr;
    }
};
```

> PS: 这也是为什么我们说在进行**移动**后，我们不能对源对象的值做任何假设（在后边有提到）的原因，右对象的控制权（指针）已经被交出去了.

#### lvalues & rvalues

根据[官方文档](https://en.cppreference.com/w/cpp/language/value_category)，每一个`Expression`都具备两个独立属性：

1. `type`(e.g. int, double, class, …)

**需要特别注意的是，有一种特殊的类型为`reference type`**:

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/2023-08-24-105949.png" alt="image-20230824035948138" style="zoom: 33%;" />

比如拿`std::forward`的实现来讲：

```cpp
// forward:
template<class S>
S&& forward(typename remove_reference<S>::type& a) noexcept
{
  return static_cast<S&&>(a);
} 
```

这里的`type`就是`rvalue reference type`。

2. `value category`

如果按照标准定义，共有三种基本的`value category`: `prvalue`, `xvalue`, `lvalue`。在这里，我们做两个粗略的定义：**左值**与**右值**：

##### lvalue

> An **lvalue** is an expression that has a name/identity. In other words, we can find address using address-of operator(&var)

##### rvalue

> An **rvalue** is an expression that does not have a name/identity.
>
> - temporaray values
> - cannot find address using address-of operator (&var)

从直观上来说，左值可以出现在表达式的左侧或者右侧，但是右值一定只能出现在右侧。



右值引用，利用`rvalue`的结果，但本身为`lvalue`.

##### l/rvalue reference

需要注意的是，我们只能够将**左值引用绑定到左值，右值引用绑定到右值**，

```c++
// rvalue reference
// here, v1 + v2 is a temporary object, which is a rvalue
auto&& v4 = v1 + v2;
```

一种特殊情况是：可以将`const lvalue reference`绑定到`rvalue`上：

```c++
const auto& ptr3 = ptr + 5;
```

原因如下：

> Normally, a temporary object lasts only until the end of the full expression in which it appears. However, **C++ deliberately specifies that binding a temporary object to a reference *to const* on the stack lengthens the lifetime of the temporary to the lifetime of the reference itself**, and thus avoids what would otherwise be a common dangling-reference error. In the example above, the temporary returned by f() lives until the closing curly brace. (**Note this only applies to stack-based references. It doesn’t work for references that are members of objects.**)

简而言之，使用`const reference`可以延长临时对象的生命周期，但这仅限于`local const reference`，在如下语句中，是不会产生实际效应的：

```c++
Sandbox(const string& n): member(n) {};
```

这也很好理解，这里的`const reference`对应的`scope`为构造函数本身，当构造函数栈退出后，该`reference object`也随之消亡了。



关于`life time`的问题，与`const auto&`一样，根据[官方文档](https://en.cppreference.com/w/cpp/language/reference_initialization#Lifetime_of_a_temporary)：

> - **Whenever a reference is bound to a temporary object or to a subobject thereof, the lifetime of the temporary object is extended to match the lifetime of the reference!**
> - **Rvalue references can be used to [extend the lifetimes](https://en.cppreference.com/w/cpp/language/reference_initialization#Lifetime_of_a_temporary) of temporary objects (note, lvalue references to const can extend the lifetimes of temporary objects too, but they are not modifiable through them)**

**当我们把一个引用绑定到一个临时对象上时，临时对象的生命周期被延长到引用的生命周期**。

##### l/rvalue reference & l/rvalue

对函数声明`test(T&& a)`的形式产生了一点疑问，这里的`a`是一个`lvalue`而不是`rvalue`。为什么？

首先我们要知道，根据先前的介绍，表达式具备两个独立的特性：

- `type`
- `value category`

在该函数参数中，`type`为`rvalue reference of T`，而我们这里所说的`lvalue`，是指的该表达式的`value category`。那么问题来了，为什么它的`value category`为`lvalue`呢？这是因为**C++**的这条规定：

> Things that are declared as rvalue reference can be lvalues or rvalues. The distinguishing criterion is: *if it has a name*, then it is an lvalue. Otherwise, it is an rvalue.

这里的`T&& a`显然是一个`name variable`，而不是类似于`3`或者`goo()`这种的`unnamed variable`。为什么**C++**要做出这种规定，详见[这篇文章](http://thbecker.net/articles/rvalue_references/section_05.html)。

```c++
void foo(X&& x) {
    X anotherX = x; // calls X(X const & rhs)
}

X&& goo();
X x = goo(); // calls X(X&& rhs) because the thing on the righ side has no name
```

简而言之，如果在代码`X anotherX = x`中的`x`被归类为一个`rvalue`，那么这意味着此式之后，等式右侧作为一个右值，应该是消亡了：

> 1. An object that is an **l-value** is **NOT** disposable（一次性的）, so you can
>    copy from, but definitely **cannot** move from.
> 2. An object that is an **r-value** is disposable, so you **can** either
>    copy or move from.

但是在这里显然我们仍然可以通过`&x`获取关于`x`的信息，`x`本身应该被视为一个`lvalue`。

而当我们试图给函数`foo(X&& x)`传递参数的时候，我们也要保证传入的参数本身是一个右值(这里也是说的参数的`value category`)，因为我们只允许将**右值绑定到右值引用上**。这也是`std::move()`会派上用场的地方:

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202111202240703.png" alt="image-20211120224058588" style="zoom:50%;" />

#### Move Constructor & Move assignment

```c++
StringVector(StringVector&& other);
StringVector& operator=(StringVector&& rhs);
```

在写这两个函数的时候，我们需要注意一点，即之前谈到的函数参数中的`StringVector&& other`的`value category`其实是`lvalue`，这意味着我们可以写出如下的代码：

```c++
Axess& operator=(Axess&& rhs) {
    students = rhs.students;
}
```

我们还是把数据copy了一份……

如何避免这个问题？我们想如果可以把这个**左值强制转化为右值**就好了，之前不是说对一个右值可以窃取他的内容吗？为了`force move semantics`，我们引入方法`std::move()`。

在课上有一个`stringVector`的例子：

```c++
StringVector& StringVector::operator=(const StringVector& rhs) noexcept {
    if (this != &rhs) { // IMPORTANT: prevent self-assignment
        delete [] _elems;
        _logicalSize = rhs._logicalSize;
        _allocatedSize = rhs._allocatedSize;
        _elems = new std::string[_allocatedSize];
        std::copy(rhs.begin(), rhs.end(), begin());
    }
    return *this;
}

StringVector::StringVector(StringVector&& other) noexcept :
    // IMPORTANT: need to move all members, not copy.
		// 事实上，对于内置类型使用move没有意义，这里只是用于演示
    _logicalSize(std::move(other._logicalSize)),
    _allocatedSize(std::move(other._allocatedSize)),
		// elems是一个指向string数组的指针
    _elems(std::move(other._elems)) {
    other._elems = nullptr; // IMPORTANT: set other to valid undetermined state.
}
```

注意到其中的`_elems`成员，我们实际上是在做这样的事情：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20211130190235422.png" style="zoom:50%;" />

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20211130190428067.png" alt="image-20211130190428067" style="zoom:50%;" />

最后随着函数栈的退出，`unnamed return value`也随之消失。

在上述代码中，将`other._elems`这个指针设置为`nullptr`是相当重要的一步，当我们设计**移动构造函数**与**移动赋值运算符**时我们必须保证：

1. **移后源对象处于这样一个状态–销毁他是无害的。特别是，一旦资源完成移动，源对象必须不再指向被移动的资源–这些资源的所有权已经归属于新的对象**;
2. **移后源对象进入一个有效且可析构的状态，但用户不能对其值做任何假设**.

而将指针指向`nullptr`，正是确保其对象进入可析构状态（析构它不会对当前程序产生`side effect`）。否则，销毁移后源对象会释放掉我们刚刚移动的内存。

##### vector's move operation

对于`vector`这种常用的数据结构来说，官方文档给出了其关于`move`操作的注意事项：

- **Move Constructor**

> Move constructor. Constructs the container with the contents of `other` using move semantics. Allocator is obtained by move-construction from the allocator belonging to `other`. After the move, `other` is guaranteed to be [empty()](https://en.cppreference.com/w/cpp/container/vector/empty).

- **Move assignment**

> Move assignment operator. Replaces the contents with those of `other` using move semantics (i.e. the data in `other` is moved from `other` into this container). `other` is in a valid but unspecified state afterwards.

这意味着，当我们有一个`vector<node*>`形式的容器时，在进行`move constructor`后不必将容器内所有指针设置为指向`nullptr`；但是在`move assignment`后，我们最好这么做。

#### move()

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202111191736625.png" alt="image-20211119173654522" style="zoom:50%;" />

首先一个疑惑就是，既然右值引用只能被绑定到右值上，那为什么我们可以将左值`rhs`传入到声明为`move(T&& t)`的参数中呢？这是因为这里的`T&& t`并非`rvalue reference`，而是一种**forwarding references(universal references)**。（下边会介绍）

使用`move`函数可以帮助我们强制执行`move semantics`(`Forcing Move Semantics`)，它将参数转化为一个`rvalue reference`(`type`)的`ravlue`(`value category`)。

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202111191801959.png" alt="image-20211119180156845" style="zoom:50%;" />

```c++
template<class T> 
typename remove_reference<T>::type&&
std::move(T&& a) noexcept
{
  typedef typename remove_reference<T>::type&& RvalRef;
  return static_cast<RvalRef>(a);
} 
```

> **Static_cast:**
>
> <img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/2023-08-24-124625.png" alt="image-20230824054624181" style="zoom:50%;" />
>
> <img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/2023-08-24-112924.png" alt="image-20230824042923526" style="zoom:50%;" />
>
> 也就是说，`static_cast`可以改变expression的`value category`，比如对于如下的例子（相当于简化版的move）：
>
> ```cpp
>   std::vector<int> v0{1,2,3};
>   std::vector<int> v2 = static_cast<std::vector<int>&&>(v0);
>   std::cout << "2) after move, v0.size() = " << v0.size() << '\n';
> ```
>
> 最终的输出结果会是0，因为v0的value category变成了xvalue（暂且理解为rvalue）；于是vector的移动复制函数被调用来创建v2，STL确保了原vector被清空。

`remove_reference`的实现：利用模板偏特化来实现引用的去除。

```cpp
template<class T> struct remove_reference { typedef T type; };
template<class T> struct remove_reference<T&> { typedef T type; };
template<class T> struct remove_reference<T&&> { typedef T type; };
```

通过观察`move`的实现，我们发现写作`static_cast<X&&>(x)`也是可以的，但`std::move`更好（`more expressive`）.

> 如果这里我们直接使用`static_cast`，而非`std::move`，那么我们需要作重载已分别适应传入的`X`经过推断后变为`X&`（原本的`X`为`lvalue`），以及保持`X`（原本的`X`为`rvalue`）的两种情况。而在原函数`std:move`中使用`remove_reference`的原因就是避免因对`static_cast<X&&>`中的`unversal referneces`进行`reference-collapsing`从而最终返回一个左值引用。

根据*C++ Primer*，使用`std::move(rr1)`意味着承诺：除了对`rr1`进行**赋值或者销毁外，不再使用它**。这不也正是我们一开始的初衷吗？“当我们对函数传入一个右值作为参数的时候，因为它行将销毁，所以我们计划窃取它的值，把我们的指针指向他存储的内容所在的位置。”

#### Universal References

根据[这篇文章](https://isocpp.org/blog/2012/11/universal-references-in-c11-scott-meyers)，如果一个类型为`T&&`的变量或者参数，其中`T`为`deduced type`，那么此时该参数或者变量即为`universal reference`. `universal reference`**到底是左值引用还是右值引用完全取决于我们如何初始化它，也就是说既可以绑定到左值上，也可以绑定到右值上**：

> - If the expression initializing a universal reference is an lvalue, the universal reference becomes an lvalue reference.
> - If the expression initializing the universal reference is an rvalue, the universal reference becomes an rvalue reference.

而根据官方文档，`universal reference`主要应用于这两个地方：

> Forwarding references are a special kind of references that preserve the value category of a function argument, making it possible to *forward* it by means of [std::forward](https://en.cppreference.com/w/cpp/utility/forward). Forwarding references are either:
>
> 1. **function parameter of a function template** declared as rvalue reference to cv-unqualified [type template parameter](https://en.cppreference.com/w/cpp/language/template_parameters#Type_template_parameter) of that same function template.
> 2. **auto&&** except when deduced from a brace-enclosed initializer list:

> **cv-unqualified**:
>
> A type is "cv-unqualified" if it doesn't have any cv-qualifiers. A cv-qualifer is either `const` or `volatile`

其实在`auto&&`的这种情况中，与`function template parameters`的情况基本完全相同，也是在做`type deduction`而已。比如在这个例子当中，`auto&&`就是一个`lvalue reference`:

```c++
std::vector<int> v;
...
auto&& val = v[0];
```

因为`v[0]`本身返回的是一个左值引用，而所有的左值引用（`type`）必定都是左值(`value category`)。

##### Notice! Where type decution takes place?

需要注意的是，`universal reference`只能存在于`type deduction`**直接出现**的地方，并且只有当引用声明为`T&&`的形式时才可以，也就是说以下的例子全部为`rvalue reference`:

```c++
template<typename T>
void f(const T&& param); // const T rather than T

template<typename T>
void f(std::vector<T>&& param); // vector<T> rather than T
```

而如下则为`universal refernce`:

```cpp
template<typename MyTemplateParamType>
void f(MyTemplateParamType&& param);  // “&&” means universal reference
```

特别的是，在*Scott Meyers*的[文章](https://isocpp.org/blog/2012/11/universal-references-in-c11-scott-meyers)中，有这么一个例子：

```c++
template <class T, class Allocator = allocator<T> >
class vector {
public:
    ...
    void push_back(T&& x);       // fully specified parameter type ⇒ no type deduction;
    ...                          // && ≡ rvalue reference
      													 // push_back()无法脱离vector<T>而存在
};
```

这就是`vector`的`push_back`方法，在这里`T&&`是一个`rvalue reference`而非`universal reference`，因为类型推断并没有发生在`push_back`函数的声明处，`vector<T>`依赖于`T`（`dependent name`[1](https://en.cppreference.com/w/cpp/language/dependent_name)）。只要`vector`的类型被确定，`push_back`方法便不再需要`type deduction`了。所以`universal reference`只会出现在`type deduction`的位置上。所以在`emplace_back`的声明里：

```c++
template <class T, class Allocator = allocator<T> >
class vector {
public:
    ...
    template <class... Args>
    void emplace_back(Args&&... args); // deduced parameter types ⇒ type deduction;
    ...                                // && ≡ universal references
};
```

这里的`&&`就是一个`universal reference`了。于是我们知道，`move()`参数里的`&&`确实是一个`universal reference`。

##### reference to reference

通常意义上，以下语句是不合法的：

```c++
Widget w1;
Widget& & w2 = w1;
```

但是我们无法在编译过程中避免出现如上所示的情况：为了解决这个问题，**C++**提出了`reference-collapsing`法则。

##### reference-collapsing

**当我们对`universal reference`类型的模板参数做类型推断时**：

```c++
template<typename T>
void f(T&& param);

f(10);
f(x);
```

需要注意的是，根据前边提到的，在遇到万能引用时，`lvalue` `T`(`value category`)会被推断为`T&`（`type`），而`rvalue` `T(`value category`)`则被推断为`T`（`type`）。这意味着，我们一定会在之后遇到`& &&`的类似情况，由于这看起来不太合理，**C++**是用了**引用折叠**（`reference-collapsing`）的方法解决这一问题：

> - **An rvalue reference to an rvalue reference becomes (“collapses into”) an rvalue reference.**
> - **All other references to references (i.e., all combinations involving an lvalue reference) collapse into an lvalue reference.**

需要注意的是，当**一个变量的类型本身就是引用时，情况有所不同：变量类型（`type`）的引用部分会被直接忽视**：

```c++
int x;
int&& r1 = 10;                   // r1’s type is int&&
int& r2 = x;                     // r2’s type is int&

f(r1);
f(r2);
```

所以何时会发生引用折叠呢？除了模板实例化（`template instantiation`）以外，当然还包括`auto`，以及`typedef`和`decltype`:

```c++
// typedef:
template<typename T>
class Widget {
    typedef T& LvalueRefType;
};
// use
Widget<int&> w;
```

#### Perfect Forwarding

什么是`perfect forwarding`? 通俗的讲，就是如果我们通过一个函数将一个参数转发给另一个函数处理，在传递的过程中，参数始终能够保持先前的特征，比如右值始终未为右值，左值始终为左值。[这里](https://stackoverflow.com/questions/3582001/what-are-the-main-purposes-of-using-stdforward-and-which-problems-it-solves)详细讨论了这一主题。为什么我们需要`perfect forwarding`？因为作为设计者，我们必须考虑到多种用户输入的可能性-给入左值、给入右值（比如一个`temporaray object`）、给如的数据带有`const`等等，而所有的这些输入我们希望在经过`factory function`之后能够与`factory function`不存在的情况完全一样。

举个例子，以如下的代码为例，我们尝试要将参数`arg`通过`factory function`传递给`T`的构造函数：

```c++
template<typename T, typename Arg> 
shared_ptr<T> factory(Arg arg)
{ 
  return shared_ptr<T>(new T(arg));
} 

// another example
template <typename A, typename B, typename C>
void f(A& a, B& b, C& c)
{
    E(a, b, c);
}
```

但上述写法显然不能应对`rvalue`参数；我们发现了一个可行方案，主要依靠两个部分：

1. `std::forward()`
2. `reference-collapsing`

```c++
// perfect forwarding
template<typename T, typename Arg> 
shared_ptr<T> factory(Arg&& arg)
{ 
  return shared_ptr<T>(new T(std::forward<Arg>(arg)));
}
```

`std::forward()`的定义如下：

```c++
// forward:
template<class S>
// universal reference only happens in the param list, not at the return value position
S&& forward(typename remove_reference<S>::type& a) noexcept
{
  return static_cast<S&&>(a);
} 

// called on lvalue
X x;
factory<A>(x);
// called on rvalue
X foo();
factory<A>(foo());
```

为什么这种方式可以实现完美转发？我们可以尝试向上述代码中分别传递左值与右值以[验证结果](http://thbecker.net/articles/rvalue_references/section_08.html#footnote_1)。在*Scott Meyer*的一个**talk**中，很形象的写出了实际上`std::forward()`的作用机理：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202111202140959.png" style="zoom:50%;" />

> 上图中的返回值处为右值引用，并非万能引用。所以如果T为左值引用返回param时会报错。

总结，编译器会去调用对应的“工厂函数”，

- 如果经过引用折叠，此类函数的参数的`type`为**rvalue reference**，参数本身的`value category`一定是**lvalue**。在此之后我们使用`std::forward`，并作为返回值，那么返回值的`value category`一定与进入“工厂函数”之前相同 ---- 依靠`std::forward`内的`static_cast<s&&>`将`value category`为左值的传入`forward`的“工厂函数”参数转化为**xvalue**的`value category`来实现。
- 左值同理。

在*HashMap*作业中，我们实现了一个简易版本的`try_emplace`，正是利用了完美转发的特性，保证传入函数的参数被完整的保留到数据类型的构造函数中，代码参见仓库：

```cpp
template<typename K, typename M, typename H>
template<class... Args>
std::pair<typename HashMap<K, M, H>::iterator, bool> HashMap<K, M, H>::try_emplace(K&& key, Args&&... args) {
    /** support Automatic Rehash */
    if (load_factor() > MaxLoadPercentage) {
        rehash(2 * bucket_count() + 1);
    }

    auto it = find(key);
    if (it != end()) {
        return {it, false};
    }
    // if not found
    size_t index = _hash_function(key) % bucket_count();
    // Now, we can move key's contents
    _buckets_array[index] = std::move(new node({std::move(key), M(std::forward<Args>(args)...)}, _buckets_array[index]));

    ++_size;

    return {{&_buckets_array, _buckets_array[index], index}, true};
}
```

关于`try_emplace`方法，这里有几篇博客可供查阅：

1. [关于`emplace`的构造函数调用问题](https://juejin.cn/post/7029372430397210632).
2. [try_emplace的使用](https://zhuanlan.zhihu.com/p/351788009).
2. [关于原位构造](https://hedzr.com/c++/variant/in-place-construction-in-cxx/).

#### noexcept

在函数声明后加`noexcept`有什么好处？

根据[这篇文章](https://www.cnblogs.com/RioTian/p/15115387.html)，首先，它会进行编译器优化：

> 因为在调用 noexcept 函数时不需要记录 exception handler，所以编译器可以生成更高效的二进制码（编译器是否优化不一定，但理论上 noexcept 给了编译器更多优化的机会）。另外编译器在编译一个 `noexcept(false)` 的函数时可能会生成很多冗余的代码，这些代码虽然只在出错的时候执行，但还是会对 Instruction Cache 造成影响，进而影响程序整体的性能

其次，当我们试图使用右值引用优化代码时，如果在移动构造函数和移动赋值函数后**不添加**`noexcept`标识符，则编译器不会进行`move semantics`的操作：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202111202337531.png" alt="image-20211120233753420" style="zoom:50%;" />

#### Rule Of Five

> If you explicitly define (or delete) a copy constructor, copy assignment, move constructor, move assignment, or destructor, you should define (or delete) all five. 
>
> The fact that you defined one of these means one of your members has ownership issues that need to be resolved.

需要注意的是，自**C++11**起，如果没有用户定义的复制构造函数、复制赋值运算符、析构函数，并且生成的移动构造函数（移动赋值运算符）有效，则编译器会隐式生成移动构造函数（移动赋值运算符）。

### Copy elision & RVO

`copy elision`，也叫**复制省略**，复制省略常出现**我们使用临时对象初始化一个新对象时**：

共有两种环境，分别为：

- 在 “return” 语句中，当操作数为与函数返回类型为同一类类型
- 在变量的初始化中，当初始化表达式为与变量类型为同一类类型

同时，可以基本分为两种应用场景：

1. 一个函数以值传递参数，当调用时，我们选择传入一个临时对象作为参数：

```c++
void foo(MyClass param) {
    // do something
}

int main() {
    // pass a temporary object to initialize the param
    foo(MyClass());
}
```

何为复制省略？这里，程序原本需要调用移动构造函数，把临时对象`MyClass()`移动到`param`中，而由于`copy elision`机制的存在，这里就可以直接把生成的`temporary object`传入函数，而不需要多余的移动（或者复制）了。

2. 还有一种情况，是函数返回一个临时对象，这里**C++**引出了重要的编译器优化方法：**RVO**.

需要注意的是，`copy elision`即使在复制/移动操作存在`side effects`时仍然适用。也就是说，根据*Wikipedia*上的这个例子：

```c++
struct C {
  C() {}
  C(const C&) { std::cout << "A copy was made.\n"; }
};
 
C f() {
  return C();
}
 
int main() {
  std::cout << "Hello World!\n";
  C obj = f();
}
```

函数返回的是一个临时对象，所以编译器会进行`copy elision`，这里被叫做`Return Value Optimization(RVO)`。根据编译器的不同设置，它可以选择优化掉一次将`f()`内生成的临时对象赋值给返回值的操作，还可以选择优化掉一次将`f()`返回值赋值给`C obj`的操作，故以下所有输出结果都是合理的：

```c++
Hello World!
A copy was made.
A copy was made.
```

```c++
Hello World!
A copy was made.
```

```c++
Hello World!
```

需要注意的是，这里的例子中`C`的复制构造函数必须是带`const`的参数，因为右值可以绑定到`const`的左值引用，如果没有`const`，则会报错。

需要注意的一点是，在**C++17**之前，**使用临时对象（一个`prvalue`）初始化另一个对象时的`copy elision`优化不是强制性的**。我们可以通过在编译器中使用加上*`-fno-elide-constructors`*来关闭这一选项。而正因为该优化此时并非强制，编译器要求即使优化发生，复制构造函数/移动构造函数也要存在（显式或者隐式（编译器自动生成）），如果我们给这些函数加上一个`=delete`，则不能通过编译。

但自**C++17**起，这种行为被描述为`passing Unmaterialized objects`，它变成了编译器**强制优化**的一种行为，即使此时复制构造函数/移动构造函数不可选，程序仍可正常编译运行。

一种特殊情况是`Named Return Value Optimization(NRVO)`，此时我们返回的是一个`lvalue`，一个具名对象，而不是一个临时对象（`prvalue`）：

```c++
MyClass foo() {
    MyClass obj;
    // do something
    return obj;
}

// amother case: as a parameter
MyClass bar(MyClass obj) {
    // do something
    return obj;
}
```

**NRVO**目前仍被列为**可选优化**，也就是说如果开启**NRVO**，我们仍需保证存在合适的复制构造函数/移动构造函数。

需要注意的是，在上边的代码中，程序在将函数内的局部变量传递给返回值时，会先尝试调用`move constructor`而非`copy constructor`。为什么呢？

根据[官方文档](https://en.cppreference.com/w/cpp/language/return#Notes)，在`overload resolution`的过程中，对于一个位于返回值位置上的表达式，如果具有`Automatic Storage Duration`，那么首先会尝试使用`move constructor`，如果失败，再使用`copy constructor`:

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202111211414317.png" alt="image-20211121141428179" style="zoom:50%;" />

### Polymorphism

在先前学习的**CS 106L**课程中，我们也提到了*多态*这一概念，`templates`与`Derived classes`都是多态：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202111232142717.png" alt="image-20211123214207558" style="zoom:50%;" />

在*CS 106X*的课程中，我们详细讨论了关于从派生类到基类的隐式转换过程。在这一节中我们即将提到动态绑定（`Dynamic Binding`），以及许多其他的继承（`Inheritance`）的注意事项。

#### Inheritance

在我们使用继承语法：

```c++
class A : public B {

}
```

时，`B`所在的位置被称为**派生类列表**。

> 关键字`virtual`只能使用在类内。

一个**纯虚函数**就可以使类成为**抽象基类**，但是抽象基类中除了包含**纯虚函数**外，还可以包含其它的成员函数（虚函数或普通函数）和成员变量。**需要注意的是，我们不能直接实例化`抽象基类`，但是可以实例化它的派生类。**

##### static

如果基类中存在静态成员，则无论从基类派生出多少子类，**静态成员**只存在**唯一的实例**。如果该成员为`private`，则**派生类无权访问**；如果该成员被设置为允许访问，则派生类与基类均可访问。关于**static**关键字在面向对象时的使用，作如下补充：

`static`对象可以分为`non-local static`和`local static`.`non-local static`对象的初始化发生在**加载阶段**，即**main**函数执行前。但C++没有规定多个`non-local static`对象的初始化顺序，尤其是来自多个编译单元的`non-local static`对象，他们的初始化顺序是**随机**的.

而对于`local static`，初始化过程发生在**第一次执行到该语句时**，并且该**局部静态变量只初始化这一次**。

**类内静态成员**属于一种`non-local static`.

1. 类的静态成员独立于任何对象之外，**对象中不包含任何与静态成员有关的数据**。
2. 类内静态成员和静态成员函数均不与对象绑定在一起，故静态成员函数不包含`this`指针，不能声明成**const**的，我们也不能在`static`函数体内使用`this`指针。
3. 成员函数**不需要**作用域运算符即可访问静态成员。
4. 一般来讲，我们只能**在类内声明静态成员；类外初始化、定义静态成员**，但存在一些例外：

- `static const int`成员可以在类内声明+初始化，如果`static const`不是**整型**，则不可以类内初始化;
- `static constexpr`成员必须在类内初始化；若编译时`static constexpr`数据成员可用它的**值**替代（如表示数组个数等），它可以不需要类外定义。若不能替代（如作为参数等），类外必须含有一条定义语句(所以习惯上无论如何都在类外定义一下)

5. 当在类的外部定义静态函数时，不能重复`static`关键字，该关键字只出现在类内的声明语句;

6. **我们可以使用静态数据成员作为成员函数的默认实参，但非静态成员不行**，因为他们的值是对象的一部分，**如果**非静态成员如果在成员函数被调用前没有被初始化，将导致在函数调用时编译器无法得知默认实参的值是多少。所以编译器做了这条规定，将可能出现的错误及时避免在编译期。

   > 虚函数也可以拥有默认实参，但是虚函数的默认实参**由本次调用的静态类型决定**。所以，虚函数的默认实参在基类和派生类中最好一致。

##### final & override

我们可以使用`final`阻止一个类被继承：

```c++
class A final {/* */}
```

还可以使用它来阻止一个函数被后续覆盖（放在函数声明最后，函数体之前）：

```c++
void foo() const final;
```

我们可以使用关键字`override`表示这是一个派生类中的虚函数，显式注明覆盖了基类的虚函数；来方便编译器检查我们是否给错了参数列表（找不到对应的基类中的函数）：

```c++
void foo() const override;
```

如果我们不覆盖基类的某个虚函数，则该虚函数的行为类似于其他普通成员-派生类直接继承其在基类中的版本。

##### Accessible？

首先我们要知道，在使用了继承之后，一个类可以有三种不同的用户：

- 普通用户
- 类的实现者
- 派生类

派生类可以访问基类的`protected`成员，但是普通用户不行。需要注意的是，派生类的成员以及友元只能通过派生类对象访问基类的受保护成员，即**他们只能访问派生类对象中的基类部分的受保护成员，对基类中的受保护成员没有访问特权：** 参见*C++ Primer* P543.

```c++
class Base {
protected:
    int prot_mem;
};

class sneaky : public Base {
    friend void clobber(sneaky&);	// can access sneaky::prot_mem
    friend void clobber(Base&);     // can NOT access Base::prot_mem
};
```

这样做的理由是为了避免我们可以直接通过定义一个类似于`sneaky`形式的派生类直接修改基类中由`Protected`保护的内容–虽然其中的函数并非是`Base`类的友元。

在程序中，存在两个访问说明符，分别位于类的实现中以及继承的接口处。其中，**派生类的成员以及友元能否访问基类的成员只与基类中的访问说明符有关。** 而位于继承接口处的访问说明符可以控制普通用户，即**派生类用户**的访问权限：也就是我们在*CS 106X*中所提到的：继承关系在**类外部**是否可见----如果此时我们使用了**非public**访问说明符，那么派生类用户无法获取基类内容：

```c++
class Base {
public:
    void pub_mem();
protected:
    int prot_mem;
private:
    char priv_mem;
};
struct Pub_Derv : public Base {
    int f() { return prot_mem; }  // OK, can access protected members
    char g() { return priv_mem; } // error, can NOT access Base's private members
};
struct Priv_Derv : private Base {
    int f1() const { return prot_mem; } // ok
}

Pub_Derv d1;
Priv_Derv d2;
d1.pub_mem();		// ok
d2.pub_mem();		// error
```

我们可以这样理解，使用**private/protected inheritance**时**Base**类的成员在**派生类**中是**private/protected**的，所以我们不能使用**派生类用户**访问他们。

> 需要注意的是，我们将**继承自派生类的新类，也视为一种普通用户**（即基类的普通用户）。

>同时，如果我们是以`private`或者`protected`的方式继承的基类，则**用户代码**不能使用派生类到基类的转换。这正是因为派生类用户无法访问基类成员的缘故。只是在这里，**用户代码**与**派生类的派生类**有了待遇的差别，我们使用**公有或者受保护**方式继承基类时，**派生类的派生类成员和友元**可以访问派生类到基类的转换，只有使用**私有**方式继承时不可以使用该转换。

##### virtual dtor

基类通常应当定义一个虚析构函数，如果我们不这样做，则delete一个指向派生类对象的基类指针将产生未定义的行为。Why？这正是因为**动态绑定**所致：

```c++
Quote *itemP = new Quote;	// static type = dynamic type
delete itemP;				// call Quote's dtor
itemP = new Bulk_quote;		// static type != dynamic type
delete itemP;				// call Bulk_quote's dtor
```

由于指针的静态类型和被删除对象的动态类型可能不一致，我们通过将基类的析构函数设置为虚函数来保证**delete基类指针**时运行正确的析构函数版本。

需要注意的是，因为基类始终需要一个虚析构函数，但是该析构函数可能为空，我们无法判断此时究竟是否需要一个拷贝构造函数或者赋值运算符，故*Rule Of Three*此时并不适用。（*C++ Primer* P552）

需要注意的是，在继承关系中的析构函数的调用顺序与构造函数相反：**构造函数是先调用基类构造函数，再调用派生类构造函数；二析构函数是先析构派生类，再析构基类。并且，我们不需要显式调用基类的析构函数，系统会自动调用。**

##### Dynamic Binding

什么是动态绑定？通俗地说，当我们在继承关系中，使用一个**基类的引用或者指针，调用一个虚函数时，将会发生动态绑定。** 为什么叫动态绑定？因为在编译时，程序并不清楚到底是需要基类的版本还是需要派生类的版本，**只有当运行时才能决定**， 所以被叫做动态绑定。从而，与`Dynamic Binding`相对应的就是**静态绑定**，也即在编译时即可确定的绑定关系----成员函数如果没有被声明为虚函数，则解析过程发生在编译时，如果我们参看静态绑定的汇编代码，会发现他只是简单的*call+address*的形式。根据*C++ Primer*：

> 表达式的**静态类型**在编译时已知，它是变量声明时的类型或者表达式生成的类型；而**动态类型**则是**变量或表达式表示的内存中的对象类型**。基类的指针或者引用的静态类型与动态类型可能不一样。

我们通过两个例子体会上面的内容：

```c++
double print_total(const Quote &item) {
    double ret = item.net_price(n);
}
```

在这里，因为`item`的静态类型是基类，但是到底执行的是基类的该函数还是派生类的该函数取决于`print_total`给入的实参。所以这里可能存在`item`的静态类型与动态类型不同的情况。同样地，借由此规定，我们理解了为什么在*CS 106X*中存在如下的情况：

<img src='https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202110312240802.png' alt='IMG_0220' style='zoom:50%'/>

指针`var1`的静态类型为`Snow`，但是动态类型为`Sleet`（内存中的对象类型），这也是为什么对于表达式`var1->method2`，编译时检查`Snow`中是否有`method2`，运行时检查`Sleet`中是否有`method2`的原因。

> 对于对象（即非引用或者指针）来说，是不存在什么类型转换与动态类型的。

同理，我们可以体会到这句话的理由：

> 对于`virtual`关键字，我们需要注意：如果想继承某个函数，我们可以不加该关键字，程序可以正常编译，新的继承函数也可以编译通过，但这会导致程序在运行时不知道去调用新的继承函数（缺少了**动态绑定**的过程）。

关于类型转换，我们还需要注意，**不存在从基类向派生类的隐式类型转换**：

```c++
Quote base;
Bulk_quote* bulkP = &base;    //error, base->derive class
Bulk_quote& bulkRef = base;   //error, base->derive class
```

如果我们确实要做从基类到派生类的类型转换，我们有两种选择：

- `dynamic_cast`:**RTTI(run-time type identification)**

如果我们将基类指针转换为指向派生类的指针，则失败返回0；如果我们将积累引用转换为派生类的引用，则失败抛出**bad_cast**异常：

```c++
if (Derived *dp = dynamic_cast<Derived*>(bp)) {
    // use dp pointing to Derived
}
else {
    // use bp pointing to Base
}

void f(const Base &b) {
    try {
        const Derived &d = dynamic_cast<const Derived&>(b);
    }
    catch (bad_cast) {
        // deal with failure
    }
}
```

- `static_cast`:已知转换安全，强制覆盖掉编译器的检查工作

我们可以使用派生类对象为一个基类对象构造或者赋值，此时只会对派生类对象中的基类部分处理，其余部分被切割掉（`sliced down`）。

##### vptr & vtbl

候捷老师的这张图很清晰的说明了问题：

<img src='https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/IMG_0219.PNG' alt='IMG_0219' style="zoom: 25%;"/>

只要我们在类中包含了虚函数，在哪存的对象模型中就会有且始终仅有一个**虚指针**（`virtual pointer`），而观察虚指针所在的内存可以看到，**派生类中包含有每个基类的子对象，也即派生类中含有基类的部分！**。（所以，每当含有虚函数时，类对象实际占用的内存就要比所有类成员的大小大`4`，因为多了一个虚指针.）

这个虚指针指向**虚表**（`virtual table`），虚表中存放的是所有指向该类对象内部虚函数的指针。当我们使用指针`p`去调用类对象时，动态绑定决定了`p`指针指向的位置（指向哪一个类对象的内存位置），那这个`p`是什么呢？

<img src='https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/IMG_0221.PNG' alt='IMG_0221' style="zoom: 25%;"/>

如上图所示，我们首先使用一个派生类对象调用基类的方法，而在该基类方法内，存在一个虚函数，因为指针`p`其实就是调用成员函数的对象的地址，也就是我们常说的`this`指针，走到虚函数的位置后，程序会根据传入的指针的地址，找到派生类，并执行对应的虚函数，而这一过程在内部就是经由`vptr`与`vtbl`实现的！从而，我们也可以将这一调用过程使用C语言表达出来，即如图所示的形式，首先从虚指针定位虚表，再利用形成的**函数指针**调用传入的对象。

##### Inheritance on Templates

最后，来聊一聊模板类的继承。[*HERE*](https:*//www.cnblogs.com/yyxt/p/5150449.html)

首先我们要知道两个概念：

\- **依赖性基类**

\- **非依赖性基类**

按照我们先前对**非依赖性**的解释，即无需知道模板实参就可以完全确定类型的基类。他应该具备如下的样子：

```c++
template <typename X>
class Base
{
 public:
     int basefield;
     typedef  int  T;
};

template <typename T>
class D2 : public Base<double> // INdependent on template T
{T strange; };
```

**C++规定，派生类查找一个`非受限名称`时，会先在非依赖型基类(Base\<double>)中查找，然后才查找模板参数列表(template\<typename T>)**。所以上例中的`strange`是`Base<double>::T`而非模板定义中的`T`.

另一方面，对于**依赖性基类**：

```c++
template <typename T>
class D2 : public Base<T> // dependent on template T
{T strange; };
```

在**两阶段名称查找**中我们知道，对于模板中的**非依赖型名称**，将会在看到的**模板定义时进行查找**。

```c++
// This is a declaration
template<typename X>
class Base
{
    public:
        int basefield;
        typedef int T;
};

// This is a definition
template <typename T>
class DD : public Base<T>
{
    public:
        void f() { basefield = 0; }     // (1)problem……
};

template<>    // 显式特化
class Base<bool>
{
    public:
        double basefield = 1.0;
};

void g(DD<bool>& d)
{
    d.f();
}
```

**但是C++规定，非依赖型名称不会在依赖型基类中进行查找，所以编译器会在`basefield = 0`处报错**。

那我们可以让这个变量成为**依赖性变量**：

```c++
// 修改方案1
template <typename T>
class DD1 : public Base<T>
{
    public:
        void f() { this->basefield = 0; }     // 查找被延迟了
};

// 修改方案2：利用受限名称来引入依赖性
template <typename T>
class DD2 : public Base<T>
{
    public:
        void f() { Base<T>::basefield = 0; }   
};
```

这样一来，对于**依赖性变量**，编译器就会在**模板实例化**时进行**名称查找**，而**实例化时还有个好处，基类的特化是已知的**。

> 如果是使用这个解决方法，我们需要格外小心，因为如果（原来的）非受限的非依赖型名称是被用于虚函数调用的话，那么这种引入依赖性的限定将会禁止该虚函数调用.

或者我们也可以使用`using`语法以提供在实例化阶段进行名称查找的机制：

```c++
// 修改方案3:重复的限定让代码不雅观，可以在派生类中只引入依赖型基类
template <typename T>
class DD3 : public Base<T>
{
    public:
        using Base<T>::basefield;       // (1)依赖型名称现在位于作用域
        void f() { basefield = 0; }     // 正确
};
```

## RAII & Smart Pointers

### RAII

`RAII`是**C++**的一种重要机制。什么是`RAII`？中文翻译为资源获取即初始化–使用局部对象来管理资源的技术。他还有几个名字，比如**SBRM**（`Scope Based Resource Management`），**CADRE**（`Constructor Acquires, Destructor Releases`）。这样做有什么好处？根据下图所示的例子：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/image-20211130190336216.png" alt="image-20211130190336216" style="zoom:50%;" />

如果我们利用两条单独的语句分配内存和释放内存，如果代码中间出现了异常，那么内存无法被合理释放，便会产生内存泄漏。但如果我们利用一个局部对象来管理和分配内存，那么当局部对象`out of scope`时，便会自动调用他的析构函数来释放内存，进而避免了内存泄漏的问题。

从而，借助于**RAII**机制，我们可以将分配内存的任务交给构造函数，释放内存的任务交给析构函数。也就是说：

> There should never be a ”half-valid” state of the object. Object immediately useable after its creation.
>
> The destructor is always called (even with exceptions), so the resource is always freed.

**RAII**机制在多处都有体现，包括我们经常使用的`new`与`delete`过程，以及：

- 文件的打开与关闭-**C++**内部为我们提供了具备**RAII**机制的文件处理操作：

```c++
// DO NOT WRITE IN THIS WAY:
{
    ifstream input();
    input.open("test.txt");
    // do something
    input.close();
}

// with RAII provided
{
    ifstream input("test.txt");
    // do something
    // no need to close the file anymore!
}
```

- multithreading

```c++
// NO NO NO
void cleanDatabase (mutex& databaseLock, map<int, int>& database) {
    databaseLock.lock();
    // other threads will not modify database
    // modify the database
    // if exception thrown, mutex never unlocked!
    databaseLock.unlock();
}

// use RAII-"lock_guard" instead
void cleanDatabase (mutex& databaseLock, map<int, int>& database) {
    lock_guard<mutex>(databaseLock);
}
class lock_guard {
public:
    lock_guard(mutex& lock) : acquired_lock(lock) {
        acquired_lock.lock()
    }
    ~lock_guard() {
        acquired_lock.unlock();
    }
private:
    mutex& acquired_lock;
}
```

正是基于**RAII**的这种想法，**C++**引入了智能指针（`Smart Pointers`）.在介绍智能指针之前，我们先来了解一下`new`与`delete`关键字是如何工作的。

### new & delete

当我们写出如下的代码时，程序实际上在做什么？

```c++
Complex* pc = new Complex(1, 2);
String* str = new String("Hello");
```

#### new

**先分配内存，再调用构造函数**：

- **分配内存**->使用`operator new`的内置函数

```c++
String* str;
void* mem = operator new(sizeof(String));
```

在这一步，函数`operator new`中会调用`malloc()`以分配内存。

- **转型**->使用`static_cast`

```c++
str = static_cast<String*>(mem);
```

- **使用构造函数**

```c++
str->String::String("Hello");
```

这里明确两个点：

1. 分配内存，分配的是谁的内存？需要注意的是，这一步分配的并不是我们传入的`Hello`字符串的内存，而是类，也就是**对象**本身的内存。
2. 使用构造函数时，我们也会分配内存，在这里，我们分配的是所传入的对象，即C式字符串的内存

所以最终，我们会得到这样的内存分布：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202111211913964.png" alt="image-20211121191351876" style="zoom:50%;" />

`ps`是指向类对象的指针，而每个类对象中有一个私有成员指针用来表示传入的字符串（一个`char`数组）的头指针，指向另一块内存空间。

当然，由于构造函数的行为是由我们自己定义的，如果我们设计的是`Complex`对象，情况有所不同：

```c++
Complex* pc;
pc->Complex::Complex(1, 2);
```

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202111211921334.png" alt="image-20211121192101241" style="zoom:50%;" />

如果我们提供了一个括号包围的初始化器，就可以使用`auto`，但仅当括号中有单一初始化器时才可以这样使用：

```c++
auto p1 = new auto(obj); // OK
auto p2 = new auto{a, b ,c}; // error
```

我们还可以使用`new`分配一个`const`对象：

```c++
const int *pc1 = new const int(1024);
```

#### delete

> Calling a destructor releases the resources owned by the object, but it does not release the memory allocated to the object itself.

可以预想到，`delete`需要具备与`new`相反的执行过程，所以我们**首先要调用析构函数，再释放对象内存**。

> 调用析构函数时，函数的行为也是由我们定义的，他可能会释放之前在使用`new`时为类成员分配的内存，当然，也有可能啥都不需要它干.

比如对于先前的`String`对象：

- **调用析构函数**

```c++
String::~String(str);
```

而在析构函数内部，我们定义了如何释放给入的字符串（`char`数组）的内存：

```c++
~String() {
    delete [] m_data;
}
```

- **释放对象内存->使用内置的`operator delete`函数**

```c++
operator delete(str);
```

在这一步，函数`operator delete`内部会使用`free(str)`释放对象内存。

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202111212016675.png" alt="image-20211121201615585" style="zoom:50%;" />

如上图所示，程序会首先调用析构函数释放掉`Hello`的内存块，之后使用`operator delete`也即`free(str)`释放指针`ps`指向的，也即我们使用`new`时`malloc`给对象分配的内存块。我们发现这里的执行过程确确实实是和`new`时反过来的。

> 如上所述，程序也有可能在调用析构函数时啥也不做，比如当我们面对`Complex`对象：
>
> <img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202111212020355.png" alt="image-20211121202015271" style="zoom:50%;" />
>
> 因为我们没有在类内动态分配内存，所以我们只需要使用`operator delete`释放对象内存，即`pc`指向的内存块即可。

#### array new & array delete?

为什么我们说`array new`一定要搭配`array delete`？首先我们看一下动态分配所得的内存：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202111212102856.png" alt="image-20211121210236744" style="zoom:50%;" />



在使用`new`进行内存分配时，我们观察到除了分配给对象的内存外，我们还需要`Debugger Header`(在调试模式下)，`cookie`（用来表示总共分配给用户的内存块大小，并使用最后一个`byte`记录程序是在获得内存还是释放内存），用于**[内存对齐](https://zhuanlan.zhihu.com/p/30007037)**（在*VC*中要求整块为16的倍数）的`pad`，还有记录储存的`object`数量的一个整数。

那么为什么这种情况下就要使用对应的`array delete`呢？因为如果我们写作`delete p`，那么在`delete`的实际执行过程中，析构函数只能被唤起一次，这意味着我们**无法完全释放之前在类内为数据分配的内存，进而发生内存泄漏**。而使用`delete[] p`，则可以利用内存块内存储的`object`数量的数据，完整的释放内存：

<img src="https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/202111212115390.png" alt="image-20211121211533286" style="zoom:50%;" />

#### overload operator new/operator delete

`new`与`delete`是一个表达式，行为无法重载，但其中调用的`operator new(operator new[])`与`operator delete(operator delete[])`是一个函数，可以被重载。

这两个函数的重载可以作为全局重载，也可以放在`class`内部进行重载。在`name lookup`那一节我们提到过，如果`::`左侧没有东西，那么默认在`globalscope`或者由`using`引入的命名空间中寻找名称，我们可以借助这一点来去区分到底是使用类中的重载`operator new`还是全局的`operator new`:

```c++
// look for members, if no member, use global one
Foo* pf = new Foo;
// force to use the global one
Foo* pf = ::new Foo;
// delete 同理
```

<img src='https://shaopu-blog.oss-cn-beijing.aliyuncs.com/img/IMG_0238.PNG' alt='IMG_0238' style="zoom: 25%;"/>

需要注意的是，即使我们在此时的构造函数中抛出异常后没有调用`operator delete`释放内存，程序也不会因此报错，因为这表明我们放弃处理ctor抛出的异常。在标准库的`basic_string`代码中，即重载了`operator new`与`operator delete`用于扩充`string`的内存申请量.

### Smart Pointers

- `shared_ptr`
- `unique_ptr`
- `weak_ptr`

这三种类型都定义在`<memory>`头文件中。

#### shared_ptr

我们可以检测智能指针是否为空：

```c++
if (p1 && p1->empty()) {
    // do something
}
```

最安全的分配和使用动态内存的方法是调用`make_shared`:

```c++
// we can use auto here
auto p = make_shared<string>(10, '9');
```

需要注意的是，调用`make_shared`时传递的参数必须与类型的某个构造函数相匹配.

##### reference_counting

`shared_ptr`的一大特征是允许多个指针指向同一个对象，为了操作这一特性，每一个`shared_ptr`都有一个关联的计数器，即**引用计数**（`reference counting`）功能。当我们进行拷贝或者赋值操作时，计数器都会作相应的变化：

- 创建一个`shared_ptr`对象，计数器为1；
- 将对象`q`赋值给`p`（`p=q`），则相应地，`p`的计数器`-1`，`q`的计数器`+1`；
- 对于两个`shared_ptr`对象，当我们使用`q`初始化`p`时（`p(q)`），`q`的计数器`+1`，并传递给`p`；
- **当指向某对象的任意一个`shared_ptr`计数器为0时，销毁对象**；

> 基于以上特性，在保证`shared_ptr`无用之后不再保留就很重要了，否则会浪费内存，注意**如果将`shared_ptr`存放在一个容器中，随后重排了容器，从而不再需要某些元素，则应当用`erase`方法删除那些元素**

##### NOTICE!

- 接受指针参数的智能指针构造函数是`explicit`的，这意味着我们不能将一个内置指针形式隐式转化为智能指针，**必须使用直接初始化**：

```c++
shared_ptr<int> p1 = new int(3); // error
shared_ptr<int> p2(new int(3)); // OK

shared_ptr<int> clone(int p) {
    return new int(2); // error
}
```

- **不要混合使用智能指针和普通指针**

如何理解这句话？在*C++ Primer*中给出了清晰的解释：

```c++
// a function
void process(shared_ptr<int> p) {
    // do something
} // then the temp p will be destroied (go out of scope)
```

当我们面对上边的函数时，正确的传参做法是什么？应当是传递一个非临时对象的`shared_ptr`给该函数：

```c++
shared_ptr<int> ptr(new int(3));
process(ptr);
```

需要注意，我们不能给该函数传递一个普通指针！因为无法隐式转换：

```c++
int *ptr = new int(3);
process(ptr);
```

并且，我们不应该给函数传入一个**临时对象**的`shared_ptr`，这样做会导致函数结束后，对应的内存被释放：

```c++
int *x = new int(3);
process(shared_ptr<int> ptr(x));
int j = *x; // oop, x is a dangling pointer
```

- **不要使用`get`初始化另一个智能指针或者为智能指针赋值**

因为这样做我们**会创建两个独立的指向同一块内存区域的`shared_ptr`**:

```c++
shared_ptr<int> p(new int(42));
int *q = p.get();
// undefined:
shared_ptr<int>(q);
```

同时，我们需要保证自己不会释放`q`指针指向的内存，否则后续无法再对原本的智能指针操作。

- `reset`

`reset`会更新计数器，如果需要会释放对象：

```c++
p = new int(1024);
p.reset(new int(3));
```

#### unique_ptr

与`shared_ptr`所不同的是，`unique_ptr`只允许一个该类型指针指向某个对象。正因为此，其**不支持普通的拷贝或者赋值操作，我们只能将其直接初始化**。

但我们可以使用`reset`或者`release`操作转移指针控制权：

```c++
unique_ptr<string> p2(p1.release());
```

`release`返回当前的`unique_ptr`保存的指针并将其置空，并将`p1`的控制权转移给`p2`.

```c++
p3.reset(p2.release());
```

上述代码不仅将`p2`的控制权转移给`p3`，同时将`p3`原本指向的内存清空。

> exception:
>
> 不能拷贝`unique_ptr`的规则有一个例外，我们可以拷贝或者赋值一个行将销毁的`unique_ptr`，即作为函数的返回值处理，见*C++ Primer* P418. 个人猜测是因为编译器的*RVO*与*NRVO*机制导致允许我们这么做。

标准库也提供给我们管理`new`分配数组的`unique_ptr`版本，我们需要在原先的基础上加一个方括号：

```c++
unique_ptr<int[]> up(new int(10));
up.release(); // use delete[]
```

> 此时我们可以使用下标运算符，但是不支持成员访问运算符。

#### weak_ptr

这里不予讨论。

## Function Pointers

这里来讨论一下函数指针，并对关键字`typedef`与`decltype`的用法加以熟悉。关于函数指针，[这里](https://blog.csdn.net/Hipercomer/article/details/8616707?spm=1001.2101.3001.6650.1&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7EOPENSEARCH%7Edefault-1.no_search_link&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7EOPENSEARCH%7Edefault-1.no_search_link)有一篇很清晰的系列。

### Function

#### Function pointer declaration

一个函数：

```c++
bool foo(const string&, cosnt string&);
```

函数指针–一个指向函数的指针：

```c++
bool (*pf)(const string&, const string&);
```

> 需要注意的是，第一个括号是必须的，否则我们相当于**声明了一个函数，其返回值为`bool*`**:
>
> ```c++
> bool *pf(const string&, const string&);

那么我们如何理解函数指针的形式？首先，存在`*`说明`pf`是一个指针，其次它指向的是一个返回类型为`bool`的，参数类型为两个`const string`引用的函数。

#### Use function pointers

在使用函数指针时，`&`与`*`号往往是可选的，编译器会帮助我们自动转换，但为了满足指针调用的*一致性*，我们习惯于加上这两个符号：

```c++
pf = foo;
pf = &foo; // the same

pf("1", "2");
(*pf)("1", "2"); // the same
```

##### As parameters

```c++
void useBigger(bool pf(const string&, const string& ));
void useBigger(bool (*pf)(const string&, const string& )); // the same
```

##### As return values

我们先来思考，如果要返回一个函数，那应该写作什么形式？–>为了确定函数的类型，我们需要：

- 函数返回类型
- 函数参数类型
- 函数参数个数

于是我们可以有如下代码：

```c++
using F = int(int*, int);
F f(int);
```

上述代码表示函数`f`返回的是一个函数类型，该函数类型具备`int`类型的返回值以及`int*, int`的参数类型。

那么如果我们想要一个函数返回一个函数指针，该如何表示呢？根据我们定义函数指针格式的经验，应当这么写：

```c++
int (*) (int*, int);
```

所以，如果现在我们要直接声明一个函数，该函数**返回一个函数指针**，形式是这样：

```c++
int (*foo(int, int)) (int*, int);
```

从内而外的分析，因为`foo`有一个形参列表，所以`foo`是一个函数，前边有`*`所以返回一个指针，该指针具备函数的形式：返回类型为`int`，接受参数为`int*, int`，所以是一个函数指针。故函数`foo`返回一个函数指针。

当然，根据**C++11**提供的语法，我们还可以这样写：

```c++
auto f1(int)->int(*)(int*, int);
```

### Array

#### Decalaration

在声明数组时，我们一定要区分数组指针和指针数组的区别：

```c++
int *p1[10]; //p1 is an array, which has 10 pointers that pointing to int.
int (*p1)[10]; // p1 is a pointer, pointing to an array which has 10 ints.
```

所以如果我们想声明一个返回数组指针的函数，该怎么做？有了之前返回函数指针的经验，这里也就不难理解了：

```c++
int (*func(int i))[10];
```

该函数返回一个指针，指向一个含有十个整数的数组。

> 这里外边的括号必须存在，理由和之前一样，如果没有，将直接返回一个有十个整形指针的数组。

同样地，我们也可以使用**尾置返回类型**：

```c++
auto func(int i) -> int(*)[10];
```

函数指针数组的定义方式，也很好理解了：

```c++
int(*pf[10])(int*, int);
```

### typedef & decltype

#### typedef

##### variable

这里记录几点容易混淆和误判的使用方法：

```c++
typedef double wages; // wages is a type alias of double
typedef wages base, *p; // ???
```

这里，在第二行代码中，我们需要注意，`p`是`double*`的类型别名！

```c++
typedef char *pstring; // the same as above
```

##### array & struct

另外，对于数组的使用：

```c++
typedef int arrs[10];
```

这里的含义是`arrs`是一个`type alias`，其代表有十个`int`元素的数组。(可类推到结构体等类型上)

##### function & function pointers

```c++
typedef bool func(const string&, const string&);
```

这里的`func`是一个函数类型的`type alias`。

```c++
typedef bool (*func)(const string&, const string&)
```

这里的`func`是一个函数指针。

#### decltype

`decltype`的简单用法这里不提，这里首先说一条关键的语法规则：

> `delcltype((variable))`（双层括号）的结果永远是引用，而`decltype(variable)`的结果只有当`variable`的结果本身是引用时才是引用。（是引用意味着我们必须对其初始化.）
>
> 所以，由于`[]`的返回值为一个引用，我们使用`decltype(a[i])`时必须对其赋初始值。

需要注意的是，`decltype`关键字返回的是**类型**，这意味着对于如下代码：

```c++
int foo(int i) {
    // do something
}
decltype(foo(3));
```

我们会得到返回值类型，而对于

```c++
int foo(int i) {
    // do something
}
decltype(foo);
```

我们得到的是**函数类型**！表示出来是：

```c++
int(int);
```

于是我们可以将`typedef`与`decltype`结合来表示函数指针：

```c++
typedef decltype(foo) *ptr;
```

对于数组也是一样，他只会返回数组类型，我们想要指针，则需要一个`*`：

```c++
int odd[] = {1,2,3,4,5};
int even[] = {6,7,8,9,10};
decltype(odd) *arrPtr(int i) {
    return (i % 2) ? &odd : &even;
}
```

在这里，`decltype`返回的是一个指向含有5个整型数据的数组的**类型**，此后我们把他的指针传递回去，即：

```c++
int(*)[5];
```

需要注意的是，在上边的例子中，`odd`与`&odd`是不一样的，如果在上例中不加`&`，我们会得到编译器报错：

```c++
cannot convert ‘int*’ to ‘int (*)[5]’ in return
```

也就是说，虽然数组名也是一个指针，但他是一个`int*`的指针，但`&odd`是指向整个数组的指针，这两者的区别就好比省政府和省会市政府，虽然都在同一个位置，地址是一样的，但是他们的意义完全不同：

```c++
*odd; // 1
*(&odd); // the same as above

&odd + 1; // we will move to the next variable position (go beyond sizeof(odd) length)
odd + 1; // we will get into the next posision in the current odd array (move sizeof(odd[0]) length)
```

数组名严格意义上并不是一个指针，通过对其使用`decltype`可以看出，他是一个`identifier for a variable of type array`。

但是在使用时，数组名可以被隐式的转化成为一个指针。一个[例外](https://stackoverflow.com/questions/1641957/is-an-array-name-a-pointer)就是在使用`sizeof`操作符时：

> Array names in a C program are (in most cases) converted to pointers. One exception is when we use the `sizeof` operator on an array. If `a` was converted to a pointer in this context, `sizeof a` would give the size of a pointer and not of the actual array, which would be rather useless, so in that case `a` means the array itself.

## Multithreading

本部分作为扩展内容，在这里不予列出，个人觉得相比起这个版本的*CS 106L*的多线程内容，*CS 106B Su2020*是更好的入门。
