---
title:		C++ Stream
subtitle:	C++ Stream
summary:	notes of C++ Stream
date:		2021-11-07
lastmod:	2023-08-31
author:		shaopu
draft: 		false

type:		book
---

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