# 重复的 switch

Ref：《重构》《代码之丑》
Day：2021-12-12

## 特征

重复代码的一种特例。

同一种 switch 分支，存在于多个地方。

## 引起的问题

每次添加一个分支，就需要找到所有 switch 代码。

## 重构方法

- [以多态取代条件表达式](../重构方法/以多态取代条件表达式.md)

## 示例代码

```java
// 重构前
public double getBookPrice(final User user, final Book book) {
  double price = book.getPrice();
  switch (user.getLevel()) {
    case UserLevel.SILVER:
      return price * 0.9;
    case UserLevel.GOLD: 
      return price * 0.8;
    case UserLevel.PLATINUM:
      return price * 0.75;
    default:
      return price;
  }
}


public double getEpubPrice(final User user, final Epub epub) {
  double price = epub.getPrice();
  switch (user.getLevel()) {
    case UserLevel.SILVER:
      return price * 0.95;
    case UserLevel.GOLD: 
      return price * 0.85;
    case UserLevel.PLATINUM:
      return price * 0.8;
    default:
      return price;
  }
}

// 重构后
interface UserLevel {
  double getBookPrice(Book book);
  double getEpubPrice(Epub epub);
}


class RegularUserLevel implements UserLevel {
  public double getBookPrice(final Book book) {
    return book.getPrice();
  }
  
  public double getEpubPrice(final Epub epub) {
    return epub.getPrice();
}


class GoldUserLevel implements UserLevel {
  public double getBookPrice(final Book book) {
    return book.getPrice() * 0.8;
  }
  
  public double getEpubPrice(final Epub epub) {
    return epub.getPrice() * 0.85;
  }
}


class SilverUserLevel implements UserLevel {
  public double getBookPrice(final Book book) {
    return book.getPrice() * 0.9;
  }
  
  public double getEpubPrice(final Epub epub) {
    return epub.getPrice() * 0.85;
  }
}


class PlatinumUserLevel implements UserLevel {
  public double getBookPrice(final Book book) {
    return book.getPrice() * 0.75;
  }
  
  public double getEpubPrice(final Epub epub) {
    return epub.getPrice() * 0.8; 
	}
}
```

