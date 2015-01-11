---
layout: post
title: C语言和抽象思维(二)
tags: [c]
---

上一次我们说到C语言结合抽象思维完成一个非所见即所得的编辑器， 并且我们已经定义了这个编辑器应有的行为， 基本上抽象也已经完成。这一节讲的更多是实现上的事情。光有设计思路是不够的， 到最后我们得作出一点什么东西才行。

## 数组实现

字符串缓冲区有什么特点呢？首先我们需要记录光标位置， 其次要能对字符进行增删， 很自然的我们可以想到用数组来进行表示。数组表示可以轻易的记录当前光标的位置， 只需要记录下标值就可以。并且缓冲区中的字符十一个有序的同类序列， 这和数组的表示相吻合。但由于C语言中为数组申请空间时必须知道数组大小， 所以我们需要一个值记录现在已经使用了多少个字符。于是我们把结构体`bufferCDT`定义如下：

```c
#define MaxBuffer 100

struct bufferCDT {
  char text[MaxBuffer];
  int length; // 目前已经使用的长度
  int cursor; // 光标的位置
};
```

接下来就只需要把对字符串进行操作的几个函数实现就可以了， 但因为我们需要把函数和数据分离， 也就是说， 可以同时并发调用这个函数， 但各个不同调用函数的人之间数据不会相互影响。所以我们的函数应该定义为这样子：` void InsertCharacter(bufferADT buffer, char ch)` 每次都把bufferADT的实体传进去， 那么函数进行操作的时候就会有单独的一块空间， 多次调用相同函数不会相互影响。

我想， 在当前光标下删除字符、插入字符你一定可以自己动手完成的！什么？你不确定？那好， 我给你一个参考思路：对于删除字符， 首先要检查当前光标位置是否在有效范围之内，如果是， 那么直接把光标之后的所有字符向前移动一位，然后`buffer->cursor--;`; 对于插入字符， 需要先检查目前有效范围是不是超过最大范围， 如果没有，那么把光标后的所有字符向后移动一位，然后把字符插入进字符串， 最后`buffer->length++; buffer->cursor++;`。

光标移动？那更简单了， 我相信你可以的！

`DisplayBuffer`的实现：

```c
void DisplayBuffer(bufferADT buffer)
{
  int i;

  for(i = 0; i < buffer->length; i++) {
    printf(" %c", buffer->text[i]);
  }
  printf(" \n");
  for(i = 0; i < buffer->cursor; i++) {
    printf("  ");
  }
  printf("^\n");
}
```

## 栈实现

栈？编辑器？反正我一开始是没想到可以用栈表示。但思路其实很简单：分别用两个栈， 一个表示光标之前的字符， 一个表示光标之后的字符。原来是这样！

> 这让我想到火影忍者里的一个小段子：鸣人在修炼风遁螺旋手里剑的时候非常努力，但由于需要大量查克拉，并且很多分身在同时修炼，九尾的查克拉很容易溢出。后来下雨了，鸣人累得趴下了，向卡卡西抱怨，风遁螺旋手里剑就像走路的时候，一边要看左边，同时还要看右边，这怎么做的到啊！卡卡西说， 哦， 这很简单啊， 于是就使用了一个影分身，一个负责看左边，一个负责看右边。这里也是一样的， 一开始我在想，用栈怎么表示缓冲区？同时记录一个索引位置吗？这样子很不方便啊！翻到这一页的时候才发现，可以用两个栈，一个表示光标前，一个表示光标后。。。

数据结构该怎么定义呢？ 如上面所说：用两个栈！

```c
struct bufferCDT {
  stackADT before;
  stackADT after;
};
```

用栈其实很方便， 完成移动光标的操作只需要一个Push， 一个Pop就可以了。完成删除只需要Pop并丢弃该字符、插入只需要Push就可以。

别看我，[我可没有代码](#资料)， 你一定可以自己写出来的！

## 总结

我们看了两种实现， 该总结一下了， 不知道大家有没有发现， 我们用了两种表现方式， 但是代码的接口却完全没动！这就是抽象的好处。 抽象可以让逻辑和实现分开， 只要实现提供能完成功能的函数， 实现随便改， 而逻辑动都不要动！感觉到了吗？为了验证我们的总结， 我们再说一种实现 ---- 链表实现。

## 链表实现

链表有什么好处呢？首先， 只要内存扛得住， 编辑器缓冲区可以无限长！其次，相比栈和数组表示， 把光标移动到缓冲区的首部和尾部时消耗特别小， 再者， 打字出错是经常发生地事情， 如果在缓冲区内插入完数据后发现，在最前面漏了一个字符！ 如果我们用的是字符表示的话， 电脑会说“jerk!你想累死我是吧！”， 因为数组需要把一大堆字符全部往后移动， 然后才能插入！ 栈表示？ 电脑也会骂你的！ 栈也需要一大把的Pop和Push！ 

*链表实现实际上也很方便！但是有一个小坑， 如果你认真自己思考的话， 很快你就会发现的， 当然，你发现以后要解决那就更简单了。链表表示由于需要画大量的图，Linux下我也一直没找到一个顺手的画图工具， 就先不写了！容我偷偷懒*

> 链表表示你就当作是习题吧。习题二：用双向链表表示一下！

## 资料

我把代码打了个包， 你可以下载:

[Click me !](/public/c-abstractions.tar.gz)