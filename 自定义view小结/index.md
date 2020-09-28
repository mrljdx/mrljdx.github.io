# 自定义View小结


View基类呈现一个100x100像素的空白正方形，要改变控件的大小并呈现一个不同的界面则需要分别对onMesure和onDraw方法进行重写。
** onMesure**方法中，新的视图将会计算一系列给定的边界条件下占据的高度和宽度。
** onDraw**方法用于在画布上进行绘图。
<!--more-->
## 实例代码
下面代码中定义了一个MyView继承自View:
```java
public class MyView extends View {

  // 代码中创建View时调用的构造函数
  public MyView(Context context) {
    super(context);
  }

  // 根据xml中的定义的资源创建View是调用的构造函数，定义默认样式
  public MyView (Context context, AttributeSet ats, int defaultStyle) {
    super(context, ats, defaultStyle );
  }

  //同上，无定义样式
  public MyView (Context context, AttributeSet attrs) {
    super(context, attrs);
  }

//  @Override
//  protected void onMeasure(int wMeasureSpec, int hMeasureSpec) {
//    int measuredHeight = measureHeight(hMeasureSpec);
//    int measuredWidth = measureWidth(wMeasureSpec);
//
//    // 必须调用setMeasuredDimension这个方法
//    // 否则当要显示视图时会报一个运行时的错误
//    setMeasuredDimension(measuredHeight, measuredWidth);
//  }
//	
//  private int measureHeight(int measureSpec) {
//    int specMode = MeasureSpec.getMode(measureSpec);
//    int specSize = MeasureSpec.getSize(measureSpec);
//
//     // [ ... 计算View的高度 ... ]
//
//     return specSize;
//  }
//
//  private int measureWidth(int measureSpec) {
//    int specMode = MeasureSpec.getMode(measureSpec);
//    int specSize = MeasureSpec.getSize(measureSpec);
//
//     // [ ... 计算View的宽度 ... ]
//
//     return specSize;
//  }
//
//  @Override
//  protected void onDraw(Canvas canvas) {
//    // [ ... 绘制的自己定义的视图 ... ]
//  }
  
  /**
   * Listing 4-16: Drawing a custom View
   */
  @Override
  protected void onDraw(Canvas canvas) {
    // 根据最后一次调用onMeasure方法获得的大小.
    int height = getMeasuredHeight();
    int width = getMeasuredWidth();

    // 获取视图中心点
    int px = width/2;
    int py = height/2;

    // 创建一个新的画刷
    // 注意: 考虑性能的话，应该在视图初始化调用的构造函数
    // 中初始化Paint
    Paint mTextPaint = new Paint(Paint.ANTI_ALIAS_FLAG);
    mTextPaint.setColor(Color.WHITE);

    //定义要显示的文本
    String displayText = "Hello World!";

    // 计算文本的宽度
    float textWidth = mTextPaint.measureText(displayText);

    //在视图中心绘制文本
    canvas.drawText(displayText, px-textWidth/2, py, mTextPaint);
  }
  
  /**
   * Listing 4-17: 常见视图绘制的实现
   */
  @Override
  protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
    int measuredHeight = measureHeight(heightMeasureSpec);
    int measuredWidth = measureWidth(widthMeasureSpec);

    setMeasuredDimension(measuredHeight, measuredWidth);
  }

  private int measureHeight(int measureSpec) {
    int specMode = MeasureSpec.getMode(measureSpec);
    int specSize = MeasureSpec.getSize(measureSpec);

    //  如果没有限定大小的话，默认为500
    int result = 500;

    if (specMode == MeasureSpec.AT_MOST) {
      // Calculate the ideal size of your
      // control within this maximum size.
      // If your control fills the available
      // space return the outer bound.
      result = specSize;
    } else if (specMode == MeasureSpec.EXACTLY) {
      // If your control can fit within these bounds return that value.
      result = specSize;
    }
    return result;
  }

  private int measureWidth(int measureSpec) {
    int specMode = MeasureSpec.getMode(measureSpec);
    int specSize = MeasureSpec.getSize(measureSpec);

    //  Default size if no limits are specified.
    int result = 500;

    if (specMode == MeasureSpec.AT_MOST) {
      // Calculate the ideal size of your control
      // within this maximum size.
      // If your control fills the available space
      // return the outer bound.
      result = specSize;
    } else if (specMode == MeasureSpec.EXACTLY) {
      // If your control can fit within these bounds return that value.
      result = specSize;
    }
    return result;
  }
  
  /**
   * Listing 4-18: 视图的输入事件
   */
  @Override
  public boolean onKeyDown(int keyCode, KeyEvent keyEvent) {
    // 如果onKeyDown这个事件被处理了则返回true
    return true;
  }

  @Override
  public boolean onKeyUp(int keyCode, KeyEvent keyEvent) {
    // 如果onKeyUp事件被处理了则返回true
    return true;
  }

  @Override
  public boolean onTrackballEvent(MotionEvent event ) {
    // 获取当前事件表示的动作
    int actionPerformed = event.getAction();
    //如果当前事件被处理则返回true
    return true;
  }

  @Override
  public boolean onTouchEvent(MotionEvent event) {
    // 获取当前事件表示的动作
    int actionPerformed = event.getAction();
    // 返回true，如果当前事件被处理的话
    return true;
  }
}
```
## 小结
从上面的实例可以看出，自定义View的基本步骤：
1. 自定义View类继承自View;
2. 重写**onMesure**方法，新的视图将会计算一系列给定的边界条件下占据的高度和宽度。(注：必须调用``setMeasuredDimension(measuredHeight, measuredWidth)``这个方法，否则当要显示视图时会报一个运行时的错误
3. 重写**onDraw**方法用于在画布上进行绘图，绘制你想展示的界面。
4. 常用的类：**Paint**（画笔） 和 ``onDraw``方法中的参数**Canvas canvas** （画布）。
