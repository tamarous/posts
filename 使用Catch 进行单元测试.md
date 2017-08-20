# 使用Catch 进行单元测试

单元测试，是一个在我耳边常常出现，但是我从来没有实践过的软件开发过程，因为我曾经认为单元测试对于我平时写的玩具代码而言是大材小用，是杀鸡用牛刀。最近在给华为做一个项目的过程中，甲方向我们明确提出了所写代码必须通过单元测试的要求，使得不得不去学习了一下单元测试的方法，这一试，让我感受到了单元测试的好处，现将整个过程记录如下。

简而言之，整个项目是用C++开发的，我负责了链表操作、消息传递这两个模块。甲方为我们提供了文档，而我们要做的就是实现这些函数的功能。在拿到文档后，我花了半天时间实现了这些函数的功能。但是对我编写的函数的正确性，我却不敢打包票保证，这是因为在之前编写代码的过程中，我习惯了通过调试来寻找代码中的 bug，而现在由于我只负责了一个模块，而看不到项目整体的代码，因此就没有办法进行调试了。

单元测试，可以说很好的解决了我所遇到的这个问题。顾名思义，单元测试，测试的就是一个单元，即某一模块、某个类或某个函数的正确性。通过编写单元测试，我们可以检测我们的代码的输出是否符合预期，进而判断代码的正确性。对已经通过单元测试的代码进行修改后，如果仍然能够通过单元测试，那么说明我们新作的修改是没有问题的。

我所使用的单元测试框架是 [Catch](https://github.com/philsquared/Catch)，是一个现代化、轻量级的单元测试工具，非常简单易用。在使用时只需要下载一个头文件[Catch.hpp](https://github.com/philsquared/Catch/releases/download/v1.9.7/catch.hpp)，然后将这个头文件添加到项目目录中。为了快速进行测试，Catch 推荐我们创建一个专门的源文件，然后在这个文件中放入下面两行代码，有兴趣的可以阅读[这里](https://github.com/philsquared/Catch/blob/master/docs/slow-compiles.md)来了解细节。

    // Catch.cpp
    #define CATCH_CONFIG_MAIN
    #include "catch.hpp"

在每一个要测试的模块的**源文件**中，包含头文件`Catch.hpp`，之后便可以编写单元测试用例了。

下面我以一个实际的例子介绍 Catch 的使用方法。我所编写的模块名为 AVFrameList，头文件如下：

    // AVFrameList.h
    
    int create_AVFrameManager(AVFrameManager*& AVFrameMag, int subpicNum);
    int create_AVFrameNode(AVFrameNode*& pNode);
    int get_AVFrameNode(AVFrameNode*& pNode, AVFrameList *pList);
    int add_AVFrameNode(AVFrameList *pList,AVFrameNode *pNode,HANDLE *mutex);
    int update_AVFrameList(AVFrameList *pList,UINT8 used_mask,HANDLE *mutex);
    int delete_AVFrameManager(AVFrameManager*& AVFrameMag, HANDLE *mutex);
    
源文件如下：

    #include "AVFrameManager.h"
    #include "catch.hpp"
    
    // 函数的具体实现
    ...
    ...

单元测试的编写格式如下：

    TEST_CASE("Test Name","Tag Name") {
        SECTION("SECTION 1 Name") {
        
        }
        SECTION("SECTION 2 Name") {
        
        }        
        ...
    }
    
`TEST_CASE`这个宏接受两个参数，其中第一个参数是测试的名称，必须要独特，第二个参数是标签名，可选。大括号中的部分是若干个SECTION，代表所要执行的测试，每个SECTION接受一个字符串参数，表示SECTION的名字。我推荐用每个SECTION的测试内容来给SECTION取名，比如"Create AVFrameManager"，这样会更加直观。

在每个SECTION中，先调用所要测试的函数，然后用REQUIRE()或CHECK()来检测执行结果是否符合预期。如果括号里的表达式返回结果为true，那么表示断言成立，否则表示断言不成立。如果一个SECTION中的每个断言都成立，那么便通过了单元测试。REQUIRE()和CHECK()的区别在于：表达式的返回值如果为false，那么REQUIRE()便会立即停止向下执行，而CHECK()则会跳过当前断言，继续向下执行。
    
    TEST_CASE("AVFrameManager.", "[AVFrameManager]") {
        AVFrameManager *manager = NULL;
        SECTION("create_AVFrameManager") {
            create_AVFrameManager(manager, 0);
            REQUIRE(manager != NULL); 
        }
    }

另外有一点值得注意，那就是运行单元测试时，SECTION与SECTION之间是相互独立的，在某个SECTION中声明定义的变量不能在另外一个SECTION中使用。而这也符合编写单元测试的一个原则：
> 每个单元测试应该足够短小而清晰，并且每个测试之间应该是独立的。

整个模块的单元测试如下：

    TEST_CASE("AVFrameManager.", "[AVFrameManager]") {
        AVFrameManager *manager = NULL;
        SECTION("create_AVFrameManager") {
            create AVFrameManager(manager, 0);
            REQUIRE(manager != NULL);
            REQUIRE(manager->pProjectList != NULL);
            REQUIRE(manager->pSPM != NULL);
            REQUIRE(manager->psourceAList != NULL);
        }
        SECTION("add AVFrameNode") {
            create_AVFrameManager(manager, 0);
            AVFrameNode *node = NULL;
            create_AVFrameNode(node);
            add_AVFrameNode(manager->pProjectList, node, NULL);
            REQUIRE(node != NULL);
        }
        SECTION("get AVFrameNode") {
    
            create_AVFrameManager(manager, 0);
            AVFrameNode *node = NULL;
            create_AVFrameNode(node);
            add_AVFrameNode(manager->pProjectList, node, NULL);
            AVFrameNode *node2 = NULL;
            get_AVFrameNode(node2, manager->pProjectList);
            REQUIRE(node2 != NULL);
        }
        SECTION("delete AVFrameManager") {
            create_AVFrameManager(manager, 0);
            REQUIRE(manager != NULL);
            delete_AVFrameManager(manager, NULL);
            REQUIRE(manager == NULL);
        }
    }

在编写好了单元测试后，便可以运行单元测试。由于Catch.hpp中定义了一个main()函数，因此如果我们的项目中已经有了main()函数，那么需要将我们的main()函数暂时改个名字，等到运行完单元测试后再改回去。当然我觉得理论上来说进行单元测试应该不需要这么麻烦，但是由于我没看到文档上有相关的介绍，所以暂时只能这么做了。

总结：
单元测试的确很有必要，编写起来也不是很复杂，而且有了单元测试我们对于每个模块的实现正确与否实际上是更有信心的，因此在以后的项目中，我会主动编写单元测试，让我所写的代码更加健壮。

