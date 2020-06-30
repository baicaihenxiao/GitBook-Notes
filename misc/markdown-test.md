# markdown test

[http://www.cser.club/md](http://www.cser.club/md)

{% hint style="info" %}
**sss**
{% endhint %}

### \*\*\*\*[**https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/pdf/20200701000601城市地摊财富秘籍.pdf**](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/pdf/20200701000601城市地摊财富秘籍.pdf)\*\*\*\*

### \*\*\*\*

### **Font**

_aaaa italic_

**bbbb bold**

_**italic+bold**_

~~呜呜呜哈哈哈哼哼哼积极 strikethrough~~

```text
*aaaa italic*

**bbbb bold**

***italic+bold***

~~呜呜呜哈哈哈哼哼哼积极 strikethrough~~
```

### **New line & indent**

1. add double spaces at end line add 2 spaces at the end of the previous line. auto indention here.
2. one blank line with indention

   4 spaces at the beginning of this line and the previous blank line to indent

   bb

3. one blank line without indention

   less than 4 spaces not leading to indent

4. sublist leads to auto indent
5. asd
6. multi level indent
   * asd  

     add 2 spaces at the end of previous line.

   * asd

     here more than 2 spaces to indent

```text
1. add double spaces at end line  
add 2 spaces at the end of the previous line.  
auto indention here.

2. one blank line with indention

    4 spaces at the beginning of this line and the previous blank line to indent

    bb

3. one blank line without indention

  less than 4 spaces not leading to indent

4. sublist leads to auto indent
* asd

5. multi level indent
    * asd  
    add 2 spaces at the end of previous line.
    * asd

      here more than 2 spaces to indent
```

### **Space**

全 角

半 角

### **Quote**

> Dorothy followed her through many of the beautiful rooms in her castle.
>
> The Witch bade
>
> > The Wi
>
> **quote h4 https://www.markdownguide.org/basic-syntax/**
>
> * Revenue was off the chart.
> * Profits were higher than ever.
>
>   _Everything_ is going according to **plan**.

```text
> Dorothy followed her through many of the beautiful rooms in her castle.
>
> The Witch bade
>
>> The Wi

> #### **quote h4 <https://www.markdownguide.org/basic-syntax/>**
>
> - Revenue was off the chart.
> - Profits were higher than ever.
>
>  *Everything* is going according to **plan**.
```

### **Image**

* image with fixed width

![](/photos/papers/blockchain/Aggregating%20crowd%20wisdom%20via%20blockchain/system%20architecture.png){:width="300px"}

* image with alt msg

![you seeeeeeeeeeeeeeeeeeeeeeeeeee](/photos/papers/blockchain/Aggregating%20crowd%20wisdom%20via%20blockchain/system%20architecture.png){:width="20%" }

* image with link

[![link to markdownguide.org](/photos/papers/blockchain/Aggregating%20crowd%20wisdom%20via%20blockchain/system%20architecture.png){:width="300px"}](https://www.markdownguide.org/basic-syntax/)

* image aligns to right in html code\(seems not work on github jekyll\)

![](/photos/papers/blockchain/Aggregating%20crowd%20wisdom%20via%20blockchain/system%20architecture.png)

```text
* image with fixed width

![](/photos/papers/blockchain/Aggregating crowd wisdom via blockchain/system architecture.png){:width="300px"}

* image with alt msg

![](/photos/papers/blockchain/Aggregating crowd wisdom via blockchain/system architecture.png "you seeeeeeeeeeeeeeeeeeeeeeeeeee"){:width="20%" }

* image with link

[![](/photos/papers/blockchain/Aggregating crowd wisdom via blockchain/system architecture.png "link to markdownguide.org"){:width="300px"}](https://www.markdownguide.org/basic-syntax/)

* image aligns to right in html code(seems not work on github jekyll)
<img src="/photos/papers/blockchain/Aggregating crowd wisdom via blockchain/system architecture.png" width="20%"  align=right />
```

### **Unordered list**

* First item, yo
* Second item, dawg
* Third item, what what?!

```text
* First item, yo
* Second item, dawg
* Third item, what what?!
```

### **Ordered list**

1. First item, yo
2. Second item, dawg
3. Third item, what what?!

```text
1. First item, yo
2. Second item, dawg
3. Third item, what what?!
```

### **Tables**

| Title 1 | Title 2 | Title 3 | Title 4 |
| :--- | :--- | :--- | :--- |
| lorem | lorem ipsum | lorem ipsum dolor | lorem ipsum dolor sit |

TA \| CC \| ES \| ET Fully trusted \| Fully trusted\| Honest-but-curious \| sss

| TA | CC | ES | ET |
| :--- | :--- | :--- | :--- |
| Fully trusted | Fully trusted | Honest-but-curious | sss |

```text
Title 1               | Title 2               | Title 3               | Title 4
--------------------- | --------------------- | --------------------- | ---------------------
lorem                 | lorem ipsum           | lorem ipsum dolor     | lorem ipsum dolor sit

TA | CC | ES | ET
Fully trusted | Fully trusted| Honest-but-curious | sss

TA | CC | ES | ET
--- | --- | --- | ---
Fully trusted | Fully trusted| Honest-but-curious | sss
```

### **Task Lists**

* [x] Write the press release
* [ ] Update the website
* [ ] Contact the media

```text
- [x] Write the press release
- [ ] Update the website
- [ ] Contact the media
```

### **Code snippet**

code block using \'\'\'html blabla \'\'\' syntax

```markup
<html>
  <head>
  </head>
  <body>
    <p>Hello, WorldHello, WorldHello, WorldHello, WorldHello, WorldHello, WorldHello, WorldHello, WorldHello, WorldHello, WorldHello, World!</p>
  </body>
</html>
```

code block without list using spaces

```text
a blank line + 4 spaces
asdasd
```

* code block with list using spaces

  ```text
  jekyll: a blank line + 6 spaces
    markdown: a blank line + 8 spaces
  ```

inline code `inline`

```text
code block without list using spaces

    a blank line + 4 spaces
    asdasd

* code block with list using spaces

      jekyll: a blank line + 6 spaces
        markdown: a blank line + 8 spaces

inline code `inline`
```

### \*\*\*\*

