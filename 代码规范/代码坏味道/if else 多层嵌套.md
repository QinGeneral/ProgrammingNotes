# if else 多层嵌套

Ref：《重构》《代码之丑》
Day：2021-12-12

## 特征

if else 多层嵌套，逻辑比较复杂。

## 重构方法

- [以卫语句取代嵌套条件表达式](../重构方法/以卫语句取代嵌套条件表达式.md)

## 示例代码

```java
// 重构前
private void distributeEpub(final Epub epub) {
  if (epub.isValid()) {
    boolean registered = this.registerIsbn(epub);
    if (registered) {
      this.sendEpub(epub);
    }
  }
}

// 重构后
private void distributeEpub(final Epub epub) {
  if (!epub.isValid()) {
    return;
  }
  
  boolean registered = this.registerIsbn(epub);
  if (!registered) {
    return;
  }
  
  this.sendEpub(epub);
}
```