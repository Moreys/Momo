# Log4Cpp相关内容

## 1、安装----使用源码安装

 1.下载log4cpp-1.1.1.tar.gz    
          
 2. 安装：先将log4cpp-1.1.1.tar.gz拖入用户主目录(~),
    然后再执行以下步骤:

      $ tar zxvf log4cpp-1.1.1.tar.gz 

      $ cd ~/log4cpp/
    $ ./configure
    $ make
    $ make check
    $ sudo make install

          这里已经安装成功.

      默认lib库路径是 ： /usr/local/lib/
          
      默认头文件的位置： /usr/local/include/log4cpp        

 3. 使用:
  3.1 编译使用log4cpp库的CPP文件时，要加上库文件，才能顺利的编译通过，如下示例

  $ g++ log4test.cpp -llog4cpp -lpthread

  3.2 运行时，如若提示缺少log4cpp库文件，表示找不到log4cpp的动态库，需要进行以下设置
  以管理员身份登录终端，然后执行以下操作：

  a. $ sudo vim /etc/ld.so.conf

  b. 在打开的文件末尾另起一行添加动态库log4cpp的路径(这里是/usr/local/lib)，然后保存退出；
     执行命令ldconfig使设置生效即可。
  c. $ sudo ldconfig   //更新库文件的缓存信息

 4. log4cpp学习
  http://blog.csdn.net/liuhong135541/article/category/1496383



## 2、简单使用

编译：g++ helloworld.cpp -o helloworld -llog4cpp -lpthread

运行结果：1248337987 ERROR : Hello log4cpp in a Error Message!

注：以上两条日志格式很简陋，要设置合乎心意的日志格式，请参考后续的PatternLayout章节。

相关概念：

　　Log4cpp中的概念继承自log4j，最重要的是Category(种类)、Appender(附加目的地)和Layout(布局)三个概念，此外还有Priority(优先级)和NDC(嵌套的诊断上下文)等。

　　简言之，Category负责向日志中写入信息，Appender负责指定日志的目的地，Layout负责设定日志的格式，Priority被用来指定Category的优先级和日志的优先级， NDC则是一种用来区分不同场景中交替出现的日志的手段。

　　Log4cpp记录日志的原理如下：每个Category都有一个优先级，该优先级可以由setPriority方法设置，或者从其父Category中继承而来。每条日志也有一个优先级，当Category记录该条日志时，若日志优先级高于Category的优先级时，该日志被记录，否则被忽略。系统中默认的优先级等级如下：

```c++
  typedef enum {EMERG  = 0, 
              FATAL  = 0,
                      ALERT  = 100,
                      CRIT   = 200,
                      ERROR  = 300, 
                      WARN   = 400,
                      NOTICE = 500,
                      INFO   = 600,
                      DEBUG  = 700,
                      NOTSET = 800
        } PriorityLevel;
```

　　**注意：取值越小，优先级越高。**例如一个Category的优先级为101，则所有EMERG、FATAL、ALERT日志都可以记录下来，而其他则不能。

　　Category、Appender和Layout三者的关系如下：系统中可以有多个Category，它们都是继承自同一个根，每个Category负责记录自己的日志;每个Category可以添加多个Appender，每个Appender指定了一个日志的目的地，例如文件、字符流或者Windows日志，当Category记录一条日志时，该日志被写入所有附加到此Category的Appender;每个Append都包含一个Layout，该Layout定义了这个Appender上日志的格式。

```c++
#include <log4cpp/Category.hh>
#include <log4cpp/OstreamAppender.hh>
#include <log4cpp/BasicLayout.hh>
#include <log4cpp/Priority.hh>


#include <iostream>

using  namespace std;
/*
 * log4cpp 进行步骤
 *  1、创建一个Appende，并指定其包含的Layout；
 *  2、从系统中得到Category的根，将Appender添加到 Category中；
 *  3、设置Category的优先级
 *  4、记录日记
 *  5、关闭Category
 * */
int main()
{
    using namespace log4cpp;//使用命名空间
    Category & root = Category::getRoot(); //创建一个getroot对象
    

    log4cpp::OstreamAppender *pOstreamAppender = new OstreamAppender("OstreamAppender", &cout);
    //创建一个OstreamApp0ender 对象
    pOstreamAppender->setLayout(new BasicLayout()); //设置setLayout
    
    root.setAppender(pOstreamAppender);
    root.setPriority(Priority::DEBUG);

    root.debug("this is debug message");

    Category::shutdown();
    std::cout << "Hello world" << std::endl;
    return 0;
}

```

## 3、Log4Cpp---Layout布局

首先回顾一下HelloWorld的日志格式，它使用了最简单的BasicLayout：

首先回顾一下HelloWorld的日志格式，它使用了最简单的BasicLayout：

1248337987 ERROR  : Hello log4cpp in a Error Message!
1248337987 WARN : Hello log4cpp in a Warning Message!
　　上面的日志格式还可以，但显然不是许多程序员心中理想的格式，许多人理想的格式应该是这样的：

2009-07-24 15:59:55,703: INFO infoCategory : system is running
2009-07-24 15:59:55,703: WARN infoCategory : system has a warning
2009-07-24 15:59:55,703: ERROR infoCategory : system has a error, can't find a file
2009-07-24 15:59:55,718: FATAL infoCategory : system has a fatal error, must be shutdown
2009-07-24 15:59:55,718: INFO infoCategory : system shutdown, you can find some information in system log

   要获得上面的格式，必须使用比BasicLayout复杂的PatternLayout，而且要花一个小时来熟悉一下PatternLayout的格式定义方式，如果你认为值得的话。

PatternLayout

　　在介绍PatternLayout以前，首先来看看log4cpp中所有的Layout子类(Layout本身是个虚类)，一共三个：BasicLayout、PatternLayout和SimpleLayout，其中SimapleLayout并不建议使用，而BaiscLayout过于简单，因此如果程序员不自己扩展Layout的话，就只能使用PatternLayout了，值得庆幸的是，PatternLayout还是比较好用的。

PatternLayout使用setConversionPattern函数来设置日志的输出格式。该函数的声明如下：

```c++
void log4cpp::PatternLayout::setConversionPattern  (  const std::string &  conversionPattern   )  throw (ConfigureFailure) [virtual]
```

其中参数类型为std::string，类似于C语言中的printf，使用格式化字符串来描述输出格式，其具体含义如下：
 %c category；
%d 日期；日期可以进一步的设置格式，用花括号包围，例如%d{%H:%M:%S,%l} 或者 %d{%d %m %Y %H:%M:%S,%l}。如果不设置具体日期格式，则如下默认格式被使用“Wed Jan 02 02:03:55 1980”。日期的格式符号与ANSI C函数strftime中的一致。但增加了一个格式符号%l，表示毫秒，占三个十进制位。
%m 消息；
%n 换行符，会根据平台的不同而不同，但对于用户透明；
%p 优先级；
%r 自从layout被创建后的毫秒数；
%R 从1970年1月1日0时开始到目前为止的秒数；
%u 进程开始到目前为止的时钟周期数；
%x NDC。

   因此，要得到上述的理想格式，可以将setConversionPattern的参数设置为“%d: %p %c %x: %m%n”，其具体含义是“时间: 优先级 Category NDC: 消息 换行”。使用PatternLayout的例子程序如下，项目名称是LayoutExam：

```c++
#include <iostream>
#include <log4cpp/Category.hh>
#include <log4cpp/OstreamAppender.hh>
#include <log4cpp/Priority.hh>
#include <log4cpp/PatternLayout.hh>
using namespace std;

int main(int argc, char* argv[])
{
    log4cpp::OstreamAppender* osAppender = new log4cpp::OstreamAppender("osAppender", &cout);
    
    log4cpp::PatternLayout* pLayout = new log4cpp::PatternLayout();
    pLayout->setConversionPattern("%d: %p %c %x: %m%n");
    osAppender->setLayout(pLayout);
        
    log4cpp::Category& root = log4cpp::Category::getRoot();
    log4cpp::Category& infoCategory = root.getInstance("infoCategory");
    infoCategory.addAppender(osAppender);
    infoCategory.setPriority(log4cpp::Priority::INFO);

    infoCategory.info("system is running");
    infoCategory.warn("system has a warning");
    infoCategory.error("system has a error, can't find a file");
    infoCategory.fatal("system has a fatal error,must be shutdown");
    infoCategory.info("system shutdown,you can find some information in system log");

    log4cpp::Category::shutdown();
    
    return 0;
}
```

其运行结果即如下所示：

```c++
2009-07-24 15:59:55,703: INFO infoCategory : system is running
2009-07-24 15:59:55,703: WARN infoCategory : system has a warning
2009-07-24 15:59:55,703: ERROR infoCategory : system has a error, can't find a file
2009-07-24 15:59:55,718: FATAL infoCategory : system has a fatal error, must be shutdown
2009-07-24 15:59:55,718: INFO infoCategory : system shutdown, you can find some information in system log
```

## 4、Log4cpp---Appender

笔者认为Appender是log4cpp中最精彩的一个部分。我仔细阅读了大部分Appender的源代码并对设计者感到非常敬仰。

   Log4cpp中所有可直接使用的Appender列表如下：

```c++
 log4cpp::IdsaAppender              // 发送到IDS或者
log4cpp::FileAppender              // 输出到文件
log4cpp::RollingFileAppender       // 输出到回卷文件，即当文件到达某个大小后回卷
log4cpp::OstreamAppender           // 输出到一个ostream类
log4cpp::RemoteSyslogAppender      // 输出到远程syslog服务器
log4cpp::StringQueueAppender       // 内存队列
log4cpp::SyslogAppender            // 本地syslog
log4cpp::Win32DebugAppender        // 发送到缺省系统调试器
log4cpp::NTEventLogAppender        // 发送到win 事件日志
```

​    其中SyslogAppender和RemoteSyslogAppender需要与Syslog配合使用，因此这里不介绍。顺便提一句，Syslog是类Unix系统的一个核心服务，用来提供日志服务，在Windows系统中并没有直接提供支持，当然可以用相关工具提供Windows系统中的syslog服务。

　　IdsaAppender的功能是将日志写入Idsa服务，这里也不介绍。因此主要介绍以下Appender：

```c++
 log4cpp::FileAppender             // 输出到文件
log4cpp::RollingFileAppender      // 输出到回卷文件，即当文件到达某个大小后回卷
log4cpp::OstreamAppender          // 输出到一个ostream类
log4cpp::StringQueueAppender      // 内存队列
log4cpp::Win32DebugAppender       // 发送到缺省系统调试器
log4cpp::NTEventLogAppender       // 发送到win 事件日志
```

(1)OstreamAppender

　　在我刚刚学习C/C++编程时，一位老师告诉我，如果没有好用的调试工具，就在代码中加入printf语句，将调试信息打印出来(当时在linux下面，确实没有什么易用的c++调试工具)。现在有了OstreamAppender，一切都好办了，它可以将日志记入一个流，如果该流恰好是cout，则会在标准控制台上输出。比printf优越的是，除了输出消息外，还可以轻松的输出时间、时钟数、优先级等大量有用信息。

　　OstreamAppender的使用非常简单，在前面的HelloWorld程序中已经见过，创建一个OstreamAppender的具体方法如下：

log4cpp::OstreamAppender* osAppender = new log4cpp::OstreamAppender("osAppender", &cout);

   第一个参数指定OstreamAppender的名称，第二个参数指定它关联的流的指针。

(2)StringQueueAppender

　　后来一位高手又告诉我“在调试多线程程序时，不能随意使用printf”。因为printf导致IO中断，会使得本线程挂起，其花费的时间比一条普通指令多数千倍，若多个线程同时运行，则严重干扰了线程间的运行方式。所以调试多线程程序时，最好是将所有调试信息按顺序记入内存中，程序结束时依次打印出来。为此当时我们还写了一个小工具，没想到时隔多年，我碰上了StringQueueAppender。

　　我很怀疑StringQueueAppender被设计出来就是用于记录多线程程序或者实时程序的日志，虽然log4cpp的文档中并没有明确指出这一点。StringQueueAppender的功能是将日志记录到一个字符串队列中，该字符串队列使用了STL中的两个容器，即字符串容器std::string和队列容器std::queue，具体如下：

std::queue<std::string> _queue;

   _queue变量是StringQueueAppender类中用于具体存储日志的内存队列。StringQueueAppender的使用方法与OstreamAppender类似，其创建函数只接收一个参数“名称”，记录完成后需要程序员自己从队列中取出每条日志，例子程序StringQueueAppenderExam如下：

```c++
 #include <iostream>
#include <log4cpp/Category.hh>
#include <log4cpp/OstreamAppender.hh>
#include <log4cpp/BasicLayout.hh>
#include <log4cpp/Priority.hh>
#include <log4cpp/StringQueueAppender.hh>
using namespace std;

int main(int argc, char* argv[])
{
     log4cpp::StringQueueAppender* strQAppender = new log4cpp::StringQueueAppender("strQAppender");
    strQAppender->setLayout(new log4cpp::BasicLayout());
    
    log4cpp::Category& root = log4cpp::Category::getRoot();
    root.addAppender(strQAppender);
    root.setPriority(log4cpp::Priority::DEBUG);
    
    root.error("Hello log4cpp in a Error Message!");
    root.warn("Hello log4cpp in a Warning Message!");
    
    cout<<"Get message from Memory Queue!"<<endl;
    cout<<"-------------------------------------------"<<endl;
    queue<string>& myStrQ = strQAppender->getQueue();
    while(!myStrQ.empty())
    {
        cout<<myStrQ.front();
        myStrQ.pop();
    }

    log4cpp::Category::shutdown();    
    return 0;
}
```

(3) FileAppender和RollingFileAppender

​    FileAppender和RollingFileAppender是log4cpp中最常用的两个Appender，其功能是将日志写入文件中。它们之间唯一的区别就是前者会一直在文件中记录日志(直到操作系统承受不了为止)，而后者会在文件长度到达指定值时循环记录日志，文件长度不会超过指定值(默认的指定值是10M byte)。

　　FileAppender的创建函数如下：

​          

​    一般仅使用前两个参数，即“名称”和“日志文件名”。第三个参数指示是否在日志文件后继续记入日志，还是清空原日志文件再记录。第四个参数说明文件的打开方式。

　　RollingFileAppender的创建函数如下：

```c++
 /**
           Constructs a FileAppender.
           @param name the name of the Appender.
           @param fileName the name of the file to which the Appender has 
           to log.
           @param append whether the Appender has to truncate the file or
           just append to it if it already exists. Defaults to 'true'.
           @param mode file mode to open the logfile with. Defaults to 00644.
        **/  
        FileAppender(const std::string& name, const std::string& fileName,
                     bool append = true, mode_t mode = 00644);
```

​     它与FileAppender的创建函数很类似，但是多了两个参数：maxFileSize指出了回滚文件的最大值;maxBackupIndex指出了回滚文件所用的备份文件的最大个数。所谓备份文件，是用来保存回滚文件中因为空间不足未能记录的日志，备份文件的大小仅比回滚文件的最大值大1kb。所以如果maxBackupIndex取值为3，则回滚文件(假设其名称是rollwxb.log，大小为100kb)会有三个备份文件，其名称分别是rollwxb.log.1，rollwxb.log.2和rollwxb.log.3，大小为101kb。另外要注意：如果maxBackupIndex取值为0或者小于0，则回滚文件功能会失效，其表现如同FileAppender一样，不会有大小的限制。这也许是一个bug。

　　例子程序FileAppenderExam如下：

```c++
 #include <iostream>
#include <log4cpp/Category.hh>
#include <log4cpp/Appender.hh>
#include <log4cpp/FileAppender.hh>
#include <log4cpp/Priority.hh>
#include <log4cpp/PatternLayout.hh>
#include <log4cpp/RollingFileAppender.hh>
using namespace std;

int main(int argc, char* argv[])
{
    log4cpp::PatternLayout* pLayout1 = new log4cpp::PatternLayout();
    pLayout1->setConversionPattern("%d: %p %c %x: %m%n");

    log4cpp::PatternLayout* pLayout2 = new log4cpp::PatternLayout();
    pLayout2->setConversionPattern("%d: %p %c %x: %m%n");
    
    log4cpp::Appender* fileAppender = new log4cpp::FileAppender("fileAppender","wxb.log");
    fileAppender->setLayout(pLayout1);

    log4cpp::RollingFileAppender* rollfileAppender = new log4cpp::RollingFileAppender(
        "rollfileAppender","rollwxb.log",5*1024,1);
    rollfileAppender->setLayout(pLayout2);
    
    log4cpp::Category& root = log4cpp::Category::getRoot().getInstance("RootName");
    root.addAppender(fileAppender);
    root.addAppender(rollfileAppender);
    root.setPriority(log4cpp::Priority::DEBUG);

    for (int i = 0; i < 100; i++)
    {
        string strError;
        ostringstream oss;
        oss<<i<<":Root Error Message!";
        strError = oss.str();
        root.error(strError);
    }
    
    log4cpp::Category::shutdown();
    return 0;
}
```

​    程序运行后会产生两个日志文件wxb.log和rollwxb.log，以及一个备份文件rollwxb.log.1。wxb.log的大小为7kb，记录了所有100条日志;rollwxb.log大小为2kb，记录了最新的22条日志;rollwxb.log.1大小为6kb，记录了旧的78条日志。

(4) Win32DebugAppender

​    Win32DebugAppender是一个用于调试的Appender，其功能是向Windows的调试器中写入日志，目前支持MSVC和Borland中的调试器。创建Win32DebugAppender仅需要一个参数“名称”，其使用非常简单，下面是例子代码DebugAppenderExam：

```c++
 #include <iostream>
#include <log4cpp/Category.hh>
#include <log4cpp/Appender.hh>
#include <log4cpp/Win32DebugAppender.hh>
#include <log4cpp/Priority.hh>
#include <log4cpp/PatternLayout.hh>
using namespace std;

int main(int argc, char* argv[])
{
    log4cpp::PatternLayout* pLayout1 = new log4cpp::PatternLayout();
    pLayout1->setConversionPattern("%d: %p %c %x: %m%n");
    
    log4cpp::Appender* debugAppender = new log4cpp::Win32DebugAppender("debugAppender");
    debugAppender->setLayout(pLayout1);
    
    log4cpp::Category& root = log4cpp::Category::getRoot().getInstance("RootName");
    root.addAppender(debugAppender);
    root.setPriority(log4cpp::Priority::DEBUG);
    
    root.error("Root Error Message!");
    root.warn("Root Warning Message!");
    
    log4cpp::Category::shutdown();
    return 0;
}
```

　在VC6中调试该代码会得到如下图所示的调试信息，注意最下方的两行调试信息：

![Appender](http://image20.it168.com/201104_500x375/550/ecd757cdd9089938.jpg)


(5) NTEventLogAppender



   该Appender可以将日志发送到windows的日志，在运行程序后可以打开windows的计算机管理?系统工具?事件查看器?应用程序，可以看到下图，注意图中第一行和第二行的两个日志。

![Appender](http://image20.it168.com/201104_500x375/550/bbe6fbe8f01b8b74.jpg) 

　　例子程序NTAppenderExam如下：

```c++
 #include <iostream>
#include <log4cpp/Category.hh>
#include <log4cpp/Appender.hh>
#include <log4cpp/NTEventLogAppender.hh>
#include <log4cpp/Priority.hh>
#include <log4cpp/PatternLayout.hh>
using namespace std;

int main(int argc, char* argv[])
{
    log4cpp::PatternLayout* pLayout1 = new log4cpp::PatternLayout();
    pLayout1->setConversionPattern("%d: %p %c %x: %m%n");
    
    log4cpp::Appender* ntAppender = new log4cpp::NTEventLogAppender("debugAppender","wxb_ntlog");
    ntAppender->setLayout(pLayout1);
    
    log4cpp::Category& root = log4cpp::Category::getRoot().getInstance("RootName");
    root.addAppender(ntAppender);
    root.setPriority(log4cpp::Priority::DEBUG);
    
    root.error("Root Error Message!");
    root.warn("Root Warning Message!");
    
    log4cpp::Category::shutdown();
    return 0;
}
```

## 5、Log4cpp---Category

- Log4cpp有一个总可用并实例化好的Category，即根Category。使用log4cpp::Category::getRoot()可以得到根Category。在大多数情况下，一个应用程序只需要一个日志种类(Category)，但是有时也会用到多个Category，此时可以使用根Category的getInstance方法来得到子Category。不同的子Category用于不同的场合。一个简单的例子CategoryExam如下所示：

```c++
#include <iostream>
#include <log4cpp/Category.hh>
#include <log4cpp/OstreamAppender.hh>
#include <log4cpp/FileAppender.hh>
#include <log4cpp/BasicLayout.hh>
#include <log4cpp/Priority.hh>
using namespace std;

int main(int argc, char* argv[])
{
    log4cpp::OstreamAppender* osAppender1 =newlog4cpp::OstreamAppender("osAppender1", &cout);
    osAppender1->setLayout(new log4cpp::BasicLayout());

     log4cpp::OstreamAppender* osAppender2 = new log4cpp::OstreamAppender("osAppender2", &cout);
    osAppender2->setLayout(new log4cpp::BasicLayout());

    log4cpp::Category& root = log4cpp::Category::getRoot();
    root.setPriority(log4cpp::Priority::DEBUG);
    
    log4cpp::Category& sub1 = root.getInstance("sub1");
    sub1.addAppender(osAppender1);
    sub1.setPriority(log4cpp::Priority::DEBUG);
    sub1.error("sub error");

    log4cpp::Category& sub2 = root.getInstance("sub2");
    sub2.addAppender(osAppender2);
    sub2.setPriority(101);
    sub2.warn("sub2 warning");
    sub2.fatal("sub2 fatal");
    sub2.alert("sub2 alert");
    sub2.crit("sub2 crit");

    log4cpp::Category::shutdown();    
    return 0;
}
```

## 6、Log4Cpp---配置文件的使用

- 另一个非常优秀的特征就是通过读取配置文件，确定category、appender、layout等对象。也是我们非常推荐的使用方式，可以灵活地通过配置文件定义所有地对象及其属性，不用重新编码，动态更改日志记录的策略。

- Log4cpp主要提供了 log4cpp::PropertyConfigurator 和log4cpp::SimpleConfigurator两种机制（文件格式），但 log4cpp::SimpleConfigurator将来不再支持了，而且格式非常简单，这里就不多说明，自己看源码吧。

  配置文件的格式和log4j的配置文件一样，是标准的java属性文件格式。下面是附带的例子配置文件：

```c++


# 定义了3个category sub1, sub2, sub3
# 其中sub2和sub3设置了additivity属性为false;sub1的additivity属性默认为true
rootCategory=DEBUG, rootAppender

category.sub1=,A1

category.sub2=INFO, A2
additivity.sub2=false

category.sub3=ERROR, A3
additivity.sub3=false

# 定义rootAppender类型和layout属性，这里使用了BasicLayout
appender.rootAppender=org.apache.log4cpp.ConsoleAppender
appender.rootAppender.layout=org.apache.log4cpp.

#定义A1的属性，这里使用了SimpleLayout

appender.A1=org.apache.log4cpp.FileAppender
appender.A1.fileName=./log/A1.log
appender.A1.layout=org.apache.log4cpp.SimpleLayout
#定义A2的属性，这里使用了PatternLayout

appender.A2=org.apache.log4cpp.ConsoleAppender
appender.A2.layout=org.apache.log4cpp.PatternLayout
appender.A2.layout.ConversionPattern=The message '%m' at time %d%n
#定义A3的属性
appender.A3=org.apache.log4cpp.RollingFileAppender
appender.A3.fileName=./log/A3.log
appender.A3.maxFileSize=50
appender.A3.maxBackupIndex=3
appender.A3.backupPattern=%Y-%m-%d
appender.A3.layout=org.apache.log4cpp.PatternLayout
appender.A3.layout.ConversionPattern=%d{%Y-%m-%d %H:%M:%S} [%p]: [%c] %m%n

带配置文件的程序举例 ：
#include "log4cpp/Category.hh"
#include "log4cpp/PropertyConfigurator.hh"
int main(int argc, char* argv[]) {
    // 1 读取解析配置文件
    // 读取出错, 完全可以忽略，可以定义一个缺省策略或者使用系统缺省策略
    // BasicLayout输出所有优先级日志到ConsoleAppender
    try
    {
        log4cpp::PropertyConfigurator::configure("./log4cpp.conf");
    }
    catch (log4cpp::ConfigureFailure& f)
    {
        std::cout << "Configure Problem " << f.what() << std::endl;
        return -1;
    }

    //2    实例化category对象
    //    这些对象即使配置文件没有定义也可以使用，不过其属性继承其父category
    //    通常使用引用可能不太方便，可以使用指针，以后做指针使用
    log4cpp::Category& root = log4cpp::Category::getRoot();
    log4cpp::Category& sub1 = log4cpp::Category::getInstance(std::string("sub1"));
    log4cpp::Category& sub2 = log4cpp::Category::getInstance(std::string("sub2"));
    log4cpp::Category& sub3 = log4cpp::Category::getInstance(std::string("sub3"));
    log4cpp::Category& sub4 = log4cpp::Category::getInstance(std::string("sub4"));

    //    正常使用这些category对象进行日志处理。
    root.fatal("root's log");

    //    sub1 has appender A1 and rootappender. since the additivity property is set true by default
    sub1.info("sub1's log");

    //    sub2 has appender A2 appender. since the additivity property is set to false
    sub2.alert("sub2's log");

    //    sub3 only has A3 appender. since the additivity property is set to false
    sub3.debug("sub3's log");
    sub3.alert("sub3's log");

    //    sub4 can not be found in the config file, so the root category's appender and layout are used
    sub4.warn("sub4's log");

    return 0;
}
```



