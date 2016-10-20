title: SlideMenuStudy
speaker: 王佳
url: localhost:8000

[slide]
# SlideMenu layout

是一种支持侧滑打开快捷菜单的视图.举两个栗子如下

![img-DragLayout][]
![img-AndroidResideMenu][]

[img-DragLayout]: https://raw.githubusercontent.com/BlueMor/DragLayout/master/screenshots/123.gif
[img-AndroidResideMenu]: https://raw.githubusercontent.com/SpecialCyCi/AndroidResideMenu/master/2.gif

[slide]
## 1.1 配置 AndroidResideMenu
EtsyStaggered在maven中心仓库可用，在工程中添加如下依赖即可；
```groovy
repositories {
    mavenCentral()
}
dependencies {
    compile 'com.specyci:residemenu:1.6+'
}
```

[slide]
## 1.2 AndroidResideMenu 的基本使用
### 1.2.1 在Activity onCreate()中初始化 AndroidResideMenu.
```java
// attach to current activity;
resideMenu = new ResideMenu(this);
resideMenu.setBackground(R.drawable.menu_background);
resideMenu.attachToActivity(this);

// create menu items;
String titles[] = { "Home", "Profile", "Calendar", "Settings" };
int icon[] = { R.drawable.icon_home, R.drawable.icon_profile, R.drawable.icon_calendar, R.drawable.icon_settings };

for (int i = 0; i < titles.length; i++){
    ResideMenuItem item = new ResideMenuItem(this, icon[i], titles[i]);
    item.setOnClickListener(this);
    resideMenu.addMenuItem(item,  ResideMenu.DIRECTION_LEFT); // or  ResideMenu.DIRECTION_RIGHT
}
```

[slide]
### 1.2.2 支持手势滑动开启/关闭菜单
** 重载 activity#dispatchTouchEvent() **
```java
@Override
public boolean dispatchTouchEvent(MotionEvent ev) {
    return resideMenu.dispatchTouchEvent(ev);
}
```
** 解决手势冲突 **
当某些控件与滑动手势产生冲突时，可将其添加到ignored view，如viewpager
```java
// add gesture operation's ignored views
FrameLayout ignored_view = (FrameLayout) findViewById(R.id.ignored_view);
resideMenu.addIgnoredView(ignored_view);
```

[slide]
### 1.2.3 监听菜单状态
** 重载 activity#dispatchTouchEvent() **
```java
resideMenu.setMenuListener(menuListener);
private ResideMenu.OnMenuListener menuListener = new ResideMenu.OnMenuListener() {
    @Override
    public void openMenu() {
        Toast.makeText(mContext, "Menu is opened!", Toast.LENGTH_SHORT).show();
    }

    @Override
    public void closeMenu() {
        Toast.makeText(mContext, "Menu is closed!", Toast.LENGTH_SHORT).show();
    }
};
```

[slide]
## 2.1 代码实现分析
```java
@Override
public boolean dispatchTouchEvent(MotionEvent ev) {
    float currentActivityScaleX = ViewHelper.getScaleX(viewActivity);
    if (currentActivityScaleX == 1.0f)
        setScaleDirectionByRawX(ev.getRawX());

    switch (ev.getAction()) {
        case MotionEvent.ACTION_DOWN:
            // 标记开始拖动
            break;

        case MotionEvent.ACTION_MOVE:
            // 更新拖动状态，执行拖动动画
            break;

        case MotionEvent.ACTION_UP:
            // 判断是否打开菜单，执行剩下菜单动画并标记拖动结束
            break;

    }
    lastRawX = ev.getRawX();
    return super.dispatchTouchEvent(ev);
}
```

[slide]
## 2.1.1 case MotionEvent.ACTION_DOWN:
** 标记开始拖动 **
```java
case MotionEvent.ACTION_DOWN:
    // 标记最后一次ActionDown的座标值
    lastActionDownX = ev.getX();  
    lastActionDownY = ev.getY();  
    
    isInIgnoredView = isInIgnoredView(ev) && !isOpened(); // 标记忽略视图
    pressedState = PRESSED_DOWN; // 标记开始拖动
    break;
}
```

[slide]
## 2.1.2 case MotionEvent.ACTION_MOVE:
** 更新拖动状态，执行拖动动画 **
```java
case MotionEvent.ACTION_MOVE:
    if (isInIgnoredView || isInDisableDirection(scaleDirection))
        break; // 判断是否忽略当前拖动
        
    if (pressedState != PRESSED_DOWN && pressedState != PRESSED_MOVE_HORIZONTAL)
        break; // 判断是否处于可以拖动的状态
        
    // 标记当前的位移值
    int xOffset = (int) (ev.getX() - lastActionDownX); 
    int yOffset = (int) (ev.getY() - lastActionDownY); 

    if (pressedState == PRESSED_DOWN) {
        // 判断是否进入拖动状态
    } else if (pressedState == PRESSED_MOVE_HORIZONTAL) {
        // 更新拖动动画状态，返回true
    }

    break;
}
```

[slide]
## 2.1.3 case MotionEvent.ACTION_UP:
** 判断是否打开菜单，执行剩下菜单动画并标记拖动结束 **
```java
case MotionEvent.ACTION_UP:
    // 判断是否
    if (isInIgnoredView) break; // 判断是否忽略当前拖动
    if (pressedState != PRESSED_MOVE_HORIZONTAL) break; // 判断是否处于拖动状态
    pressedState = PRESSED_DONE; // 标记完成拖动
    if (isOpened()) {
        if (currentActivityScaleX > 0.56f)
            // 当原来的菜单状态为打开，且拖动距离足够，则关闭菜单
            closeMenu();
        else
            openMenu(scaleDirection);
    } else {
        if (currentActivityScaleX < 0.94f) {
            // 当原来的菜单状态为关闭，且拖动距离足够，则打开菜单
            openMenu(scaleDirection);
        } else {
            closeMenu();
        }
    }
    break;
}
```

[slide]
# 谢谢！