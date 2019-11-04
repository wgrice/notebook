## final基础应用

- final 修饰的变量地址值不能改变。
- final 修饰的方法不能被重写。
- final 修饰的类不能被继承。

## 并发编程中 final 可以禁止特定的重排序

- final 保证先写入对象的 final 变量，后调用该对象引用。
- final 保证先读对象的引用，后读该对象的final变量。
- final 保证先写入对象的 final 变量的成员变量，后调用该对象引用。

