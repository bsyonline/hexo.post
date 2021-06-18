---
title: BlackBerry 开发：自定义图片背景的 button
date: 2011-09-23 22:25:09
tags:
 - BlackBerry
category: 
 - Java
thumbnail: 
author: bsyonline
lede: "没有摘要"
---


```java
/**
 * 自定义图片背景的button
 *
 * @author Rolex
 * @date 2011-9-23
 */
public class BitmapBackgroundButtonField extends Field {      
    private String _label;
    private int _labelHeight;
    private int _labelWidth;
    private Font _font;

    private Bitmap _currentBitmap;
    private Bitmap _onBitmap;
    private Bitmap _offBitmap;

    /**
     * Constructor.
     * @param text The text to be displayed on the button
     * @param style Combination of field style bits to specify display attributes
     */
    public BitmapBackgroundButtonField(String text, long style,Bitmap bitmap1, Bitmap bitmap2)
    {
        super(style);

        _font = getFont();
        _label = text;
        _labelHeight = _font.getHeight();
        _labelWidth = _font.getAdvance(_label);
        _onBitmap = bitmap1;
        _offBitmap = bitmap2;
        _currentBitmap = _offBitmap;
    }

    /**
     * @return The text on the button
     */
    String getText()
    {
        return _label;
    }

    /**
     * Field implementation.
     * @see net.rim.device.api.ui.Field#getPreferredHeight()
     */
    public int getPreferredHeight()
    {
        return 90;
    }

    /**
     * Field implementation.
     * @see net.rim.device.api.ui.Field#getPreferredWidth()
     */
    public int getPreferredWidth()
    {
        return 70;
    }

    /**
     * Field implementation.  Changes the picture when focus is gained.
     * @see net.rim.device.api.ui.Field#onFocus(int)
     */
    protected void onFocus(int direction)
    {
      _currentBitmap = _onBitmap;
        invalidate();
    }

    /**
     * Field implementation.  Changes picture back when focus is lost.
     * @see net.rim.device.api.ui.Field#onUnfocus()
     */
    protected void onUnfocus()
    {
      _currentBitmap = _offBitmap;
        invalidate();
    }

    /**
     * Field implementation.
     * @see net.rim.device.api.ui.Field#drawFocus(Graphics, boolean)
     */
    protected void drawFocus(Graphics graphics, boolean on)
    {
    }

    /**
     * Field implementation.
     * @see net.rim.device.api.ui.Field#layout(int, int)
     */
    protected void layout(int width, int height)
    {
        setExtent(Math.min( width, getPreferredWidth()),
        Math.min( height, getPreferredHeight()));
    }

    /**
     * Field implementation.
     * @see net.rim.device.api.ui.Field#paint(Graphics)
     */
    protected void paint(Graphics graphics)
    {      
        // First draw the background colour and picture
        graphics.setColor(Color.WHITE);
        graphics.fillRect(0, 0, getWidth(), getHeight());
        graphics.drawBitmap(0, 0, getWidth(), getHeight(), _currentBitmap, 0, 0);

        // Then draw the text
        graphics.setColor(Color.BLACK);
        graphics.setFont(_font.derive(Font.BOLD, 13));

//        graphics.drawText(_label, 4, 70,
//            (int)( getStyle() & DrawStyle.ELLIPSIS | DrawStyle.HALIGN_MASK ),
//            getWidth() - 6 );
        graphics.drawText(_label, 0, _currentBitmap.getHeight() + 4, graphics.HCENTER);
    }

    /**
     * Overridden so that the Event Dispatch thread can catch this event
     * instead of having it be caught here..
     * @see net.rim.device.api.ui.Field#navigationClick(int, int)
     */
    protected boolean navigationClick(int status, int time)
    {
        fieldChangeNotify(1);
        return true;
    }

}
```
