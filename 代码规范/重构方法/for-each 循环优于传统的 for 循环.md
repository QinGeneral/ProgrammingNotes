# for-each 循环优于传统的 for 循环

Ref：《Effective Java 第三版》
Day：2021-12-12

for-each 无需处理迭代器、索引，更加简洁、灵活，能预防出错。而且没有性能损失。

能使用 for-each 的地方，尽量使用 for-each。

```java
for (Iterator<Element> i = c.iterator(); i.hasNext();) {
    Element e = i.next();
}

for (int i = 0; i < a.length; i++) {

}

for (Element e : elements) {

}
```