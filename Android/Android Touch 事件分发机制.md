# Android Touch 事件分发机制

一个点击事件产生后，会先传递给 Activity，再传递给 Window，然后传递给顶级 View。顶级 View 收到事件后，会按照以下伪代码逻辑去分发事件。
1. onInterceptTouchEvent 判断是否拦截事件，如果拦截，则交给 onTouchEvent 处理；
2. 否则交给子 View 处理，子 View 处理的逻辑和父 View 一致；
3. Touch 事件分为三类，DOWN、MOVE、UP；
4. 正常情况下，一个事件序列只能被一个 View 拦截消耗，

```java
public boolean dispatchTouchEvent(MotionEvent ev) {
    boolean consume = false;
    if (onInterceptTouchEvent(ev)) {
        consume = onTouchEvent(ev);
    } else {
        consume = child.dispatchTouchEvent(ev);
    }

    return consume;
}
```

## 滑动冲突

滑动冲突分为三类：
1. 左右 + 上下滑动冲突；
2. 上下 + 上下 或 左右 + 左右滑动冲突；
3. 上下 + 左右 + 上下 或 左右 + 上下 + 左右滑动冲突。

解决办法：
1. 外部拦截法：如果父 View 需要此事件，就拦截事件，否则不拦截事件；
2. 内部拦截法：子 View 拦截事件，如果子 View 不需要此事件，就把事件交给父 View 处理。
   ```java
   // 子 view 需要通过 requestDisallowInterceptTouchEvent 方法来控制是否交给父 view 处理
   public boolean dispatchTouchEvent(MotionEvent event) {
      int x = (int) event.getX();
      int y = (int) event.getY();

      switch (event.getAction()) {
      case MotionEvent.ACTION_DOWN: {
                parent.requestDisallowInterceptTouchEvent(true);
                break;
          }
          case MotionEvent.ACTION_MOVE: {
              int deltaX = x - mLastX;
              int deltaY = y - mLastY;
              if (父容器需要此类点击事件)) {
                  parent.requestDisallowInterceptTouchEvent(false);
              }
              break;
          }
          case MotionEvent.ACTION_UP: {
              break;
          }
          default:
              break;
          }

          mLastX = x;
          mLastY = y;
          return super.dispatchTouchEvent(event);
      }
   }

   // 父 view 不能拦截 DOWN 事件，因为一旦拦截，其他事件无法传递给子 view
    public boolean onInterceptTouchEvent(MotionEvent event) {
        int action = event.getAction();
        if (action == MotionEvent.ACTION_DOWN) {
            return false;
        } else {
            return true;
        }
    }
   ```