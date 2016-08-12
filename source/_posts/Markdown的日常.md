title: Markdown的日常
date: 2016-03-15 13:04:13
tags:
categories:
- tools
---
# 一级标题 Markdown 的日常
#
## 二级标题 基本使用
##

### 三级标题 列表使用
###
- 列表的使用

	- 无序列表的使用使用 * 或者 -
	- 有序列表可以使用阿拉伯数字
		1. 列表1
		2. 列表2
		3. 列表3
		4. 列表4
> 引用别处的句子可以使用 > 符号

<!-- more -->

注意符号和文本之间的空格,任何符号换行后需要多打一个空行才是普通文本


### 斜体粗体 ###
- 一个 ❤ 号包含文本是*斜体*
- 两个 ❤ 号包含文本是 **粗体**

### 代码框 ###

- 采用`code`包围的方法


`
	public static void main(){
	}
`



- 采用table缩进的方法


	public static void main(){

	}

- 高亮代码 ```

```python
class SomeClass:
    pass

if __name__ == '__main__':
    # A comment
    print 'hello world'
```

```c++
#include <iostream>
using namespace std;
int main()
{
    /**
    程序说明
        cout 会默认删除结果的0 但是 cout.setf 会覆盖掉这种方法
    */
    cout.setf(ios_base::fixed, ios_base::floatfield); //fixed-point
    float tub = 10.0 / 3.0 ;
    double mint = 10.0 / 3.0;
    const float million = 1.0e6;
    cout << "tub = " << tub;
    cout << ", million tubs = " << million * tub;
    cout << ",\n and ten million tub = " << 10 * million * tub;

    cout << "mint = " << mint;
    cout << ", million mint = " << million * mint <<  endl;
    cin.get();
    return 0;
}

```

```java
	public class UserDao{

		private int age;
		private String name;

	}
```




### 绘制表格

|     项目        | 价格   |  数量  |
| -----   | -----:  | :----:  |
| 计算机     | \$1600 |   5     |
| 手机      |   \$12   |   12   |
| 管线      |    \$1    |  234  |

其实绘制表格在Atom下直接输入 table再按一下table就可以出来模板了，意外发现。

### 分割线 ###
- 三个♥号
***

### 图片和连接###
- 插入链接和图片的语法区别在于一个！
- 插入链接
	- [my bolgger](http://coffee1993.github.io/)
	- 插入图片
	 ![mypicture](http://cdn.sspai.com/attachment/thumbnail/2014/04/15/f96c892fc63933ab186235f7c910753b10f77_mw_800_wm_1_wmp_3.jpg)


### 图片描述
![markdown](http://7xrw2w.com1.z0.glb.clouddn.com/markdown.png)

***

![markdown2](http://7xrw2w.com1.z0.glb.clouddn.com/markdown2.png)
