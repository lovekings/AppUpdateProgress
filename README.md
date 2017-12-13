# AppUpdateProgress

<!-- Baidu Button BEGIN -->

<div id="article_content" class="article_content">

<p><span style="color:rgb(153,153,153); font-family:&quot;Microsoft YaHei&quot;,Arial; font-size:14px"><br>
</span></p>
<p><span style="font-family:&quot;Microsoft YaHei&quot;,Arial"><span style="font-size:14px"><span style="color:rgb(51,51,51); font-family:Arial; line-height:26px"><strong>前言</strong></span><span style="color:rgb(51,51,51); font-family:Arial; line-height:26px">：</span></span><span style="font-size:14px">现在一般的Android软件都是需要不断更新的，当你打开某个app的时候，如果有新的版本，它会提示你有新版本需要更新。当有更新时，会弹出一个提示框，点击下载，则在通知来创建一个数字进度条进行下载，下载成功后才到安装界面。</span></span></p>
<p><span style="font-family:&quot;Microsoft YaHei&quot;,Arial; font-size:14px"><br>
</span></p>
<p><span style="font-family:&quot;Microsoft YaHei&quot;,Arial"><span style="color:rgb(51,51,51); font-family:&quot;Microsoft YaHei&quot;,Arial"><strong><span style="font-size:14px">效果：</span></strong></span><br>
</span></p>
<p style="text-align:center"><span style="font-family:&quot;Microsoft YaHei&quot;,Arial; font-size:14px">
<img src="http://wx2.sinaimg.cn/large/006pg0W4gy1feiqx5fhogg30b60hhtnz.gif" alt="具体看源码"/><br>
</span></p>
<p><span style="font-family:&quot;Microsoft YaHei&quot;,Arial; font-size:14px"><br>
</span></p>
<p><span style="font-family:&quot;Microsoft YaHei&quot;,Arial"><span style="color:rgb(51,51,51); font-family:&quot;Microsoft YaHei&quot;,Arial"><strong><span style="font-size:14px">开发环境：</span><span style="font-size:14px">AndroidStudio2.2.1&#43;gradle-2.14.1</span></strong></span><br>
</span></p>
<p><span style="font-family:&quot;Microsoft YaHei&quot;,Arial"><span style="color:rgb(51,51,51); font-family:&quot;Microsoft YaHei&quot;,Arial"><strong><span style="font-size:14px"><br>
</span></strong></span></span></p>
<p><span style="font-family:&quot;Microsoft YaHei&quot;,Arial"><span style="color:rgb(51,51,51); font-family:&quot;Microsoft YaHei&quot;,Arial"><strong><span style="color:rgb(51,51,51); font-family:Arial"><span style="font-size:14px">涉及知识：</span></span></strong></span></span></p>
<p><strong style="color:rgb(51,51,51); font-family:&quot;Microsoft YaHei&quot;,Arial"><span style="font-size:14px; font-family:Arial"><span style="white-space:pre"></span>1.Handler机制</span></strong></p>
<p><span style="font-family:&quot;Microsoft YaHei&quot;,Arial"><span style="color:rgb(51,51,51); font-family:&quot;Microsoft YaHei&quot;,Arial"><strong><span style="font-size:14px; color:rgb(51,51,51); font-family:Arial"><span style="white-space:pre"></span>2.自定义控件&#43;Canvas绘画</span></strong></span></span></p>
<p><span style="font-family:&quot;Microsoft YaHei&quot;,Arial; font-size:14px"><span style="white-space:pre"></span><strong>3.自定义dialog</strong></span></p>
<p><span style="font-family:&quot;Microsoft YaHei&quot;,Arial; font-size:14px"><strong><br>
</strong></span></p>
<p><span style="font-family:&quot;Microsoft YaHei&quot;,Arial"><span style="color:rgb(51,51,51); font-family:&quot;Microsoft YaHei&quot;,Arial"><strong><span style="font-size:14px">部分代码：</span></strong></span><br>
</span></p>
<p><span style="font-family:&quot;Microsoft YaHei&quot;,Arial"><span style="color:rgb(51,51,51); font-family:&quot;Microsoft YaHei&quot;,Arial"><strong><span style="font-size:14px"><br>
</span></strong></span></span></p>
<p><span style="font-family:&quot;Microsoft YaHei&quot;,Arial"><span style="color:rgb(51,51,51); font-family:&quot;Microsoft YaHei&quot;,Arial"><strong><span style="font-size:14px"></span></strong></span></span><pre name="code" class="java">public class NumberProgressBar extends View {


    /**
     * 右侧未完成进度条的颜色
     */
    private int paintStartColor = 0xffe5e5e5;

    /**
     * Contxt
     */
    private Context context;

    /**
     * 主线程传过来进程 0 - 100
     */
    private int progress;

    /**
     * 得到自定义视图的宽度
     */
    private int viewWidth;


    private RectF pieOval;

    private RectF pieOvalIn;

    /**
     * 得到自定义视图的Y轴中心点
     */
    private int viewCenterY;

    /**
     * 已完成的画笔
     */
    private Paint paintInit = new Paint();


    /**
     * 未完成进度条画笔的属性
     */
    private Paint paintStart = new Paint();

    /**
     * 大圆的画笔
     */
    private Paint paintEndBig = new Paint();

    /**
     * 小圆的画笔
     */
    private Paint paintSmall = new Paint();


    /**
     * 画中间的百分比文字的画笔
     */
    private Paint paintText = new Paint();

    /**
     * 要画的文字的宽度
     */
    private int textWidth;

    /**
     * 画文字时底部的坐标
     */
    private float textBottomY;

    private int smallR;//小圆的半径
    private int bigR;//大圆半径
    private float radius;
    private int jR;//气泡矩形

    /**
     * 文字总共移动的长度（即从0%到100%文字左侧移动的长度）
     */
//    private int totalMovedLength;
    public NumberProgressBar(Context context, AttributeSet attrs) {
        super(context, attrs);
        this.context = context;
        // 构造器中初始化数据
        smallR = dip2px(context, 4);//小圆半径
        bigR = dip2px(context, 8);//大圆半径
        radius = dip2px(context, 10) / 2;//进度条高度
        jR = dip2px(context, 6);//矩形

        initData();
    }

    /**
     * 初始化数据
     */
    private void initData() {

        // 未完成进度条画笔的属性
        paintStart.setColor(paintStartColor);
        paintStart.setStrokeWidth(dip2px(context, 1));
        paintStart.setDither(true);
        paintStart.setAntiAlias(true);
        paintStart.setStyle(Paint.Style.FILL);

        // 已完成进度条画笔的属性
        paintInit.setColor(context.getResources().getColor(R.color.blue));
        paintInit.setStrokeWidth(dip2px(context, 1));
        paintInit.setAntiAlias(true);
        paintInit.setDither(true);
        paintInit.setStyle(Paint.Style.FILL);


        // 小圆画笔
        paintSmall.setColor(Color.WHITE);
        paintSmall.setAntiAlias(true);
        paintSmall.setStyle(Paint.Style.FILL);

        // 大圆画笔
        paintEndBig.setColor(context.getResources().getColor(R.color.blue));
        paintEndBig.setAntiAlias(true);
        paintEndBig.setStyle(Paint.Style.FILL);


        // 百分比文字画笔的属性
        int paintTextSizePx = sp2px(context, 11);  //设置百分比文字的尺寸
        paintText.setColor(context.getResources().getColor(R.color.blue));
        paintText.setTextSize(paintTextSizePx);
        paintText.setAntiAlias(true);
        paintText.setTypeface(Typeface.DEFAULT_BOLD);

    }

    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        super.onMeasure(widthMeasureSpec, heightMeasureSpec);
    }


    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);

        //得到float型进度
        float progressFloat = progress / 100.0f;
        int viewHeight = getMeasuredHeight();//得到控件的高度

        viewWidth = getMeasuredWidth() - 4 * jR;

        viewCenterY = viewHeight - bigR;

        float currentMovedLen = viewWidth * progressFloat + 2 * jR;

        String str = progress + &quot;%&quot;;

        Rect bounds = new Rect();
        paintText.getTextBounds(str, 0, str.length(), bounds);
        textWidth = bounds.width();
        textBottomY = bounds.height();
/**
 * 1：绘画的文本
 * 2.距离x的位移
 * 3.距离Y的位移
 * 4.画笔对象
 */
        canvas.drawText(str, currentMovedLen - textWidth / 2,
                viewCenterY - smallR / 2 - bigR / 2 - 2 * jR + textBottomY / 2,
                paintText);//文字


        //圆角矩形初始的
        canvas.drawRoundRect(new RectF(2 * jR, viewCenterY - radius, currentMovedLen,
                        viewCenterY + radius),
                radius, radius, paintInit);

        //圆角矩形--进行中
        canvas.drawRoundRect(new RectF(currentMovedLen, viewCenterY - radius, viewWidth + 2 * jR,
                viewCenterY + radius), radius, radius, paintStart);

        pieOval = new RectF(currentMovedLen - bigR, viewCenterY - bigR,
        currentMovedLen + bigR, viewCenterY + bigR);

        pieOvalIn = new RectF(currentMovedLen - smallR, viewCenterY - smallR, 
        currentMovedLen + smallR, viewCenterY + smallR);

        //大圆
        canvas.drawArc(pieOval, 0, 360, true, paintEndBig);

        //小圆
        canvas.drawArc(pieOvalIn, 0, 360, true, paintSmall);
    }

    /**
     * @param progress 外部传进来的当前进度
     */
    public void setProgress(int progress) {
        this.progress = progress;
        invalidate();
    }

    public static int dip2px(Context ctx, float dp) {
        float density = ctx.getResources().getDisplayMetrics().density;
        int px = (int) (dp * density + 0.5f);
        return px;
    }

    public static int sp2px(Context context, float spValue) {
        return (int) TypedValue.applyDimension(TypedValue.COMPLEX_UNIT_SP,
        spValue, context.getResources().getDisplayMetrics());
    }
}</pre><br>
<br><br>
</p>
   
</div>



<!-- Baidu Button END -->
