### C++指针和引用的区别

~~~c++
#include <iostream>
#include <cstring>
using namespace std;


struct Books
{
   char  title[50];
   char  author[50];
   char  subject[100];
   int   book_id;
};

void testStruct(Books);
void testStructPoint(Books*);
void testStructReference(Books&);

void testStruct(Books b) {
    strcpy(b.title, "fdf");
    cout << &b << endl;
}
void testStructPoint(Books* b) {
    Books book;
    book.book_id = 2;
    *b = book;
};

void testStructReference(Books& b) {
    Books book;
    book.book_id = 3;
    b = book;
}


int main()
{   Books book1;
    book1.book_id = 1;
    cout << &book1 << endl;
    strcpy(book1.title, "C++ 教程");
    // 直接传递Books会把book1的值copy给形参
    testStruct(book1);
    cout << &book1 << endl;
    // 传递book地址 testStructPoint修改book地址指向的内存为一个新的book对象
    testStructPoint(&book1);
    cout << &book1 << endl;  
    cout << book1.book_id << endl;
    Books& b = book1;
    // 传递引用通过引用直接修改修改book地址指向的内存为一个新的book对象
    testStructReference(b);
    cout << &book1 << endl;
    cout << book1.book_id << endl;
}
// 输出
/*
0x7fffffffdc60
0x7fffffffdb80
0x7fffffffdc60
0x7fffffffdc60
2
0x7fffffffdc60
3
*/

~~~

通过以上三个例子可以看出`指针`和`引用`的区别

* 指针指向一块内存地址， 通过解引用符号`*`可以访问地址保存的对象。
* 引用就是变量的一个别名, 通过引用可以直接修改变量本身， 我的理解引用就是指针加解引用`*`。

### JavaScript参数值传递

在`红宝书`中有说到`JavaScript`的参数传递都是值传递，但是令人比较疑惑的是对象如果对象也是值传递的话为什么修改对象的属性，会影响到原始对象呢？这就有点像引用传递但是另外一点如果直接修改
对象本身替换为一个新对象， 又不会影响到外部对象这就又和引用传递不一致了。

~~~js
let o = { a: 1 }
function test(o) {
  o.a = 2 // 影响原始对象
  o = { b: 3 } // 不会影响到原始对象
}

test(o)
~~~

这里不妨这样理解对于`JavaScript`对象参数传递实际上传递的是一个`指针`，在函数内部如果我们想要修改对象属性通过解引用`*`去修改， 如果是直接修改对象本身则是直接修改`指针`指向另一个内存地址。