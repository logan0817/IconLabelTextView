# IconLabelTextView
## android-自定义TextView在文字内容末尾添加图片icon、可以添加间距

## 样式示意图

![样式示意图](https://github.com/logan0817/IconLabelTextView/assets/19951960/fc7bf7f7-732d-4fe5-96db-d16b2622d996)

## 自定义属性 style.xml
```javascript
    <declare-styleable name="IconLabelTextView">
        <attr name="iconSrc" format="reference"/>
        <attr name="iconPaddingStart" format="dimension"/>
        <attr name="iconPaddingTop" format="dimension"/>
        <attr name="iconPaddingBottom" format="dimension"/>
    </declare-styleable>
```

## 自定义View-IconLabelTextView代码片段
```javascript
import android.content.Context
import android.graphics.Canvas
import android.graphics.drawable.Drawable
import android.text.Layout
import android.text.StaticLayout
import android.text.TextPaint
import android.util.AttributeSet
import androidx.appcompat.widget.AppCompatTextView
import com.foreo.common.utils.DensityUtils
import com.xyz.R

class IconLabelTextView : AppCompatTextView {

    private var icon: Drawable? = null
    private var iconPaddingStart: Int = 0
    private var iconPaddingTop: Int = 0
    private var iconPaddingBottom: Int = 0

    constructor(context: Context) : super(context) {
        init(null)
    }

    constructor(context: Context, attrs: AttributeSet?) : super(context, attrs) {
        init(attrs)
    }

    constructor(context: Context, attrs: AttributeSet?, defStyleAttr: Int) : super(
        context,
        attrs,
        defStyleAttr
    ) {
        init(attrs)
    }

    private fun init(attrs: AttributeSet?) {
        val a = context.obtainStyledAttributes(attrs, R.styleable.IconLabelTextView)
        icon = a.getDrawable(R.styleable.IconLabelTextView_iconSrc)
        iconPaddingStart = a.getDimensionPixelSize(R.styleable.IconLabelTextView_iconPaddingStart, 0)
        iconPaddingTop = a.getDimensionPixelSize(R.styleable.IconLabelTextView_iconPaddingTop, 0)
        iconPaddingBottom = a.getDimensionPixelSize(R.styleable.IconLabelTextView_iconPaddingBottom, 0)
        a.recycle()
    }

    override fun onDraw(canvas: Canvas?) {
        super.onDraw(canvas)

        canvas?.let { canvas ->
            val textPaint: TextPaint = paint
            val lineHeight = textPaint.descent() - textPaint.ascent()
            val lineCount = layout.lineCount
            val paddingTop = paddingTop.toFloat()
            val availableWidth = width - paddingLeft - paddingRight - DensityUtils.dp2px(30)

            val layout = layout ?: return@let

            for (i in 0 until lineCount) {
                val lineStart = layout.getLineStart(i)
                val lineEnd = layout.getLineEnd(i)
                val lineText = text.subSequence(lineStart, lineEnd).toString()

                if (i == lineCount - 1 && layout.getLineWidth(i) >= availableWidth) {
                    // last line with maxLines and text filled completely, adjust text to fit icon
                    var adjustedText = lineText
                    var textWidth = textPaint.measureText(lineText)
                    val iconWidth = icon?.intrinsicWidth ?: 0

                    while (textWidth + iconWidth + iconPaddingStart > availableWidth) {
                        adjustedText = adjustedText.substring(0, adjustedText.length - 1)
                        textWidth = textPaint.measureText(adjustedText)
                    }
                    if (!adjustedText.isNullOrBlank() && lineText != adjustedText) {
                        val splitIndex = text.indexOf(adjustedText) + adjustedText.length
                        text?.substring(0, splitIndex)?.let { text = "$it..." }
                        return
                    }

                    if (adjustedText.isNotEmpty()) {
                        val staticLayout = StaticLayout(
                            adjustedText,
                            textPaint,
                            availableWidth.toInt(),
                            Layout.Alignment.ALIGN_NORMAL,
                            1f,
                            0f,
                            false
                        )
                        val iconY =
                            paddingTop + (lineHeight * i) + (lineHeight - (icon?.intrinsicHeight
                                ?: 0)) / 2 + iconPaddingTop - iconPaddingBottom
                        val iconX = paddingLeft + staticLayout.width + iconPaddingStart
                        icon?.setBounds(
                            iconX.toInt(),
                            iconY.toInt(),
                            (iconX + iconWidth).toInt(),
                            (iconY + (icon?.intrinsicHeight ?: 0)).toInt()
                        )
                        icon?.draw(canvas)
                    }
                } else if (i == lineCount - 1 && lineEnd <= text.length) {
                    // last line, add icon
                    icon?.let {
                        val textWidth = textPaint.measureText(lineText)
                        val iconY =
                            paddingTop + (lineHeight * i) + (lineHeight - it.intrinsicHeight) / 2 + iconPaddingTop - iconPaddingBottom
                        val iconX = paddingLeft + textWidth + iconPaddingStart
                        it.setBounds(
                            iconX.toInt(),
                            iconY.toInt(),
                            (iconX + it.intrinsicWidth).toInt(),
                            (iconY + it.intrinsicHeight).toInt()
                        )
                        it.draw(canvas)
                    }
                }
            }
        }
    }

    fun setIcon(icon: Drawable?, paddingStart: Int, paddingTop: Int, paddingBottom: Int) {
        this.icon = icon
        this.iconPaddingStart = paddingStart
        this.iconPaddingTop = paddingTop
        this.iconPaddingBottom = paddingBottom
        invalidate()
    }
}
```

## xml中使用
```javascript
  <com.xyz.IconLabelTextView
	android:id="@+id/productVariation"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:ellipsize="end"
        android:maxLines="2"
        android:textSize="12sp"
        app:iconPaddingBottom="0dp"
        app:iconPaddingStart="5dp"
        app:iconPaddingTop="3dp"
        app:iconSrc="@drawable/ic_arrow_down_cart"
        tools:text="Normal Skin" />
```

## 代码中使用
```javascript
productVariation.setIcon(getDrawable(R.drawable.ic_arrow_down_cart),10,0,0)
```


### CSDN文章地址: [IconLabelTextView文章](https://blog.csdn.net/notwalnut/article/details/137506582?spm=1001.2014.3001.5502)

