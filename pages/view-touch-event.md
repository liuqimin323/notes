# View Touch Event

API related to Touch Event:

```java
public boolean dispatchTouchEvent(MotionEvent ev);
public boolean onInterceptTouchEvent(MotionEvent ev);
public boolean onTouchEvent(MotionEvent ev);
```

## Dispatch Touch Event
Touch Event is passed from top to bottom. First **Activity** starts dispatch touch event (but it can't intercept touch event), then it will pass down to root view (**DecorView**). DecorView includes notification bar, title, contents. ViewGroup first receive the touch event. In `ViewGroup.dispatchTouchEvent` returns true, the ViewGroup will handle touch event in `ViewGroup.onTouchEvent` by itself. Else if `ViewGroup.dispatchTouchEvent` returns false, it will pass down to `ViewGroup.interceptTouchEvent`. If ViewGroup returns true in `ViewGroup.interceptTouchEvent`, ViewGroup will handle touch event by itself. Otherwise the event will pass down to child view to `View.dispatchTouchEvent`. If the bottom view doesn't handle touch event (`View.onTouchEvent` returns false), it will pass back touch event to parent view group.

**Note:** the bottom child view doesn't have `onInterceptTouchEvent`. As it is already the bottom child view, there is no way it can pass down touch event.

![image](http://qny.ivanfan.site/TouchEvent_1.jpg)

![image](http://qny.ivanfan.site/TouchEvent_2.jpg)


## OnTouch Event
OnTouch event is triggered before onTouchEvent.

View.dispatchTouchEvent(MotionEvent):
```java
    //noinspection SimplifiableIfStatement
    ListenerInfo li = mListenerInfo;
    if (li != null && li.mOnTouchListener != null
            && (mViewFlags & ENABLED_MASK) == ENABLED
            && li.mOnTouchListener.onTouch(this, event)) {
        result = true;
    }

    if (!result && onTouchEvent(event)) {
        result = true;
    }
```
Basically, if a view has OnTouchListener and `onTouch` returns true, it won't execute `View.onTouchEvent`

**Note:** OnTouch requires 1. The View has onTouchListener; 2. View is enabled. If it requreds touch event, make onTouch returns false, and overrides onTouchEvent.

## OnTouch and OnClick
OnTouch is executed befoe onClick. OnClick is only executed under `Action_UP` in `onTouchEvent()`. So if `OnTouch` returns true, it won't execute `onTouchEvent`, then it won't execute `onClick` event.


> Author: IvanFan.
> Link: [http://ivanfan.site/2016/04/21/TouchEvent/](http://ivanfan.site/2016/04/21/TouchEv.nt/)