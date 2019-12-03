> 读到了老姚的正则表达式文章，对于正则表达式有了一个新的学习。在正则这块很多时候自己都是现用现查，很多地方都是囫囵吞枣。阅读老姚的新著，重新学习```JS```的正则表达式。分几个部分写一下读后笔记吧。

先贴一下老姚文章的[地址](https://juejin.im/post/5965943ff265da6c30653879)。一七年的文章现在才看到，汗😓

文章共分七个章节介绍正则，我们这里主要做一下总结。

##### 1. 首先，正则是匹配模式，要么匹配字符，要么匹配位置。

日常使用到最多的就是字符的匹配，比如说手机号的校验，用户昵称的校验等，很少会用到位置的匹配。但是其实我们常用的```^```、```$```就是匹配位置的锚字符。关于具体的内容稍后在总结，重要的是这个宗旨要有。

##### 2. 常用的字符匹配模式

正则表达式的强大就在于他的模糊匹配。文章把这块分成两个部分

* 横向模糊匹配

这个是指字符串长度的模糊，最常用的如```{m,n}```指连续出现最少m次，最多n次。比如常用的手机号匹配```/^1\d{10}$/```，指以 1 开头，后面出现十个数字的字符串。这里的```{10}```是```{10,10}```的简写，意思是只能出现十次。

* 纵向模糊匹配

这部分是指某个字符可能是不确定的，比如```[abc]```指这个字符可以是 'a'、'b' 或者 'c'。比如限制用户输入的内容只能是数字或者字母的时候，可以使用```/^[0-9a-zA-Z]{5,10}$/```。


##### 3. 字符组的简写和量词的简写

* 字符组的简写


这里我们把```[abc]```这种形式称为字符组，如```/^[0-9a-zA-Z]{5,10}$/```这样的是简写的形式，```-```表示范围。除此之外还有排除字符组，如```[^abc]```表示这个字符不能是 'a'、'b' 或者 'c'。另外还有常见的简写形式。

> ```\d```就是```[0-9]```。表示是一位数字。记忆方式：其英文是digit（数字）。  
```\D```就是```[^0-9]```。表示除数字外的任意字符。  
```\w```就是```[0-9a-zA-Z_]```。表示数字、大小写字母和下划线。记忆方式：w是word的简写，也称单词字符。  
```\W```是```[^0-9a-zA-Z_]```。非单词字符。  
```\s```是```[\t\v\n\r\f]```。表示空白符，包括空格、水平制表符、垂直制表符、换行符、回车符、换页符。记忆方式：s是space character的首字母。  
```\S```是```[^\t\v\n\r\f]```。 非空白符。  
```.```就是```[^\n\r\u2028\u2029]```。通配符，表示几乎任意字符。换行符、回车符、行分隔符和段分隔符除外。记忆方式：想想省略号...中的每个点，都可以理解成占位符，表示任何类似的东西。

* 量词的简写

```{m,}``` 表示至少出现m次。  
```{m}``` 等价于```{m,m}```，表示出现m次。  
```?``` 等价于```{0,1}```，表示出现或者不出现。记忆方式：问号的意思表示，有吗？  
```+``` 等价于```{1,}```，表示出现至少一次。记忆方式：加号是追加的意思，得先有一个，然后才考虑追加。  
```*``` 等价于```{0,}```，表示出现任意次，有可能不出现。记忆方式：看看天上的星星，可能一颗没有，可能零散有几颗，可能数也数不过来。
