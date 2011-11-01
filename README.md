BinSpec是什么？
==========

BinSpec是Binary Specification的缩写，以下简称BS。它是一个专门用于数据结构的描述型语言。  

最初创造BS的目的是纯粹的用于描述TCP/IP通讯中的协议格式，但是你同样可以用它来描述二进制序列化存储的格式。 

下面的示例可以让你对BinSpec有一个感性的认识。  

假设我们在制作一个基于TCP/IP的在线游戏，客户端和服务器之间的通讯通过头部的两个字节（一个字节的功能ID和一个字节的接口ID）来识别收到的消息类型。  

如果这个游戏有一个登录接口，客户端将账户和登录票据通过接口传递给服务端，服务端验证并返回结果。用BinSpec来描述大概会是这样：  

    // 玩家相关的接口
    pkg player = 1
    {
      // 可能的几种登录结果
      enum login_state
      { 
        INIT    = 0,  // 首次登录，需要初始化数据
        FAILED  = 1,  // 登录失败
        SUCCEED = 2   // 登录成功
      }

      // 登录请求
      def login_params = 0
      {
        user_name  : string,   // 账户
        hash_code  : string,   // 登录票据
        login_time : int       // 登录时间
      }

      // 登录返回结果
      def login_result = 1
      {
        state : enum<login_state> // 登录结果
      }
    }

player是玩家相关接口的集合，login\_params是登录参数，login\_result是登录返回结果，当服务端收到头部信息(1, 0)时便知道收到一个登录请求，当客户端收到(1, 1)头信息时便知道收到登录请求的结果。  


BinSpec的价值在哪里？
==========

首先我们从实际使用价值来看。

很多项目都有自己的一套通讯协议描述方式，有规格化的有非规格化的。规格化的好处是它易于制造工具，例如代码生成工具或调试工具，这样便可以在通讯层省掉很多机械性的劳动又避免出错。

规格化最常见的方式是使用XML按一定格式对数据结构进行描述，XML固然很好，但是它不是为数据结构描述这个目的而生的，这使得用它描述数据结构会让描述文档变得冗余又不好阅读（假设你有时间做一个可视化的编辑工具这也许就不是问题）  

BS的使用价值在于：

1. 它足够简单不会对开发人员造成额外的学习负担
2. 比起XML等非专门性语言，它不会有冗余的结构，这使得文档易于阅读和维护
3. 它提供一个简单方便的机制让你可以快速的开发自己的代码生成工具

其次，退一万步讲即便BS不会被广泛，但它至少也是一次不错的编程实践，它的代码和设计思路也有一定的价值 :)


BinSpec怎样使用？
==========

BS自身只提供了一套规范和一个用于解析BS文档的程序，它不对使用场景做过多的假设，它只做一些最基本的约束，如何运用它完全取决于你的创造力。 

不管如何使用，首先你需要将你数据结构（例如通讯协议）用BS语法描述出来，这个过程就像在设计C语言的结构体，你需要做的就是从BS提供的数据类型中选择合适的组合到一起。  

接着，你还需要用PHP编写一套对应自己项目需求的代码生成模版，才能真正发挥BS的威力，例如你可能会需要一个模版用于生成ActionScript的客户端代码和Erlang的服务端代码。  

BS生成PHP，但它生成的部分仅限于对数据结构的描述，并不负责使用这些数据结构，你可以将BS理解为一个“生成代码生成器的生成器”。

BS通过解析文档，将数据结构描述转换成PHP可以理解的数据格式（一个名为bs\_get\_doc的函数，它返回一组有层级结构的对象，这些对象就是BS文档的解析结果）。  

例如，你将项目的通讯协议写成BS文档，你的模版通过调用bs\_get\_doc这个函数就可以得到通讯协议的所有信息，接着就可以利用这些信息生成客户端代码、服务端代码。  

甚至你乐意的话可以直接就利用这些信息启动一个Socket服务器。  

典型的BS命令行执行方式会像这样：

    ./bs -c protocals.bs template.php | php


如何开始？
==========

首先你需要先在电脑上编译好BS，你需要准备好一个有gcc和make的环境，然后简单的make一下BS就被编译到bin目录里了。

然后你可以试着运行它，像这样：  

    ./bin/bs

这时候它会输出一些帮助信息。  

接着你可以试着让它解析项目自带的测试文档并输出PHP，像这样：

    ./bin/bs -c test.bs

这时候你应该会看到满屏的PHP代码，试着阅读一下，我相信这时候你就大概知道BS是如何运作的了。  

再接着你可以试试让它与项目自带的模版文件一起运行看看效果，像这样：

    ./bin/bs -c test.bs test.php | php

这时候你应该可以看到PHP将文档的内容用变量打印的方式输出出来，打开test.php看看并开始试着制作自己的代码生成器吧。


数据类型
==========

整数
----------

int8,  byte  - 1字节，有符号整数  
int16, short - 2字节，有符号整数  
int32, int   - 4字节，有符号整数  
int64, long  - 8字节，有符号整数  

uint8,  ubyte  - 1字节，无符号整数  
uint16, ushort - 2字节，无符号整数
uint32, uint   - 4字节，无符号整数
uint64, ulong  - 8字节，无符号整数

字符串
----------

字符串头部始终包含一个无符号整数类型的长度信息，空字符串的长度为0。  

string8, string  - 长度信息占用1个字节  
string16         - 长度信息占用2个字节  
string32         - 长度信息占用4个字节  
string64         - 长度信息占用8个字节  

列表
----------

列表头部包含一个无符号整数类型的列表长度信息，空列表的长度为0。  

list8, list  - 长度信息占用1个字节  
list16       - 长度信息占用2个字节  
list32       - 长度信息占用4个字节  
list64       - 长度信息占用8个字节  

列表后面必须紧跟列表元素的描述信息。  

示例 1:

    items : list
    {
        id : int,
        name : string
    }

枚举
----------

与C语言不一样，枚举声明必须有值和值类型  

示例 2:

    enum color : byte
    {
        red = 1, green = 2, blue = 3
    }

文档格式
==========

包和定义
----------

示例 3：

    pkg player = 1
    {
      enum login_state
      { 
        INIT    = 0,
        FAILED  = 1,
        SUCCEED = 2
      }

      def login_params = 0
      {
        user_name  : string,
        hash_code  : string,
        login_time : int
      }

      def login_result = 1
      {
        state : enum<login_state>
      }
    }

包可以嵌套定义，目的是更灵活的组织代码，比如登录接口有上下行数据，
按示例3那样分开声明没有问题，但是显得分散不利于阅读。可以尝试像示例4一样组织代码。

示例 4:

    pkg player = 1
    {
      pkg login
      {
        enum state
        { 
          INIT    = 0, 
          FAILED  = 1,
          SUCCEED = 2
        }

        def in = 0
        {
          user_name  : string,
          hash_code  : string,
          login_time : int
        }

        def out = 1
        {
          state : enum<state>
        }
      }
    }

具体的组织方式应该根据自身项目特点决定，并没有绝对的好坏。但是建议每个项目
只按一种规则书写BS代码，这是从工程化角度出发。

注释
----------

BS支持单行注释和多行注释，单行注释使用 // 起始，到行尾结束

多行注释使用 /* 起始，*/ 结束，多行注释不支持嵌套

