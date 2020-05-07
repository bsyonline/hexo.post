---
title: BlackBerry 开发：圆角矩形布局
date: 2012-04-07 15:53:17
tags:
 - BlackBerry
category: 
 - Java
thumbnail: 
author: bsyonline
lede: "没有摘要"
---


```JAVA
/**
 * 圆角的布局，内部元素遵守垂直布局的规则，可设置标题和图片,传空则不设置
 *
 * @author Rolex
 * @date 2011-9-23
 */
public class RoundRectFieldManager extends VerticalFieldManager {
    private static final int CORNER_RADIUS = 18;
    private static final int PANDDING = 5;
    private static final int MARGIN = 10;
    public RoundRectFieldManager(Bitmap bitmap, String title) {
        super(Manager.FIELD_HCENTER);
        // 横向放四个图片按钮
//          setPadding(0, 5, 0, 0); // OS 5.0未公开的API
        setMargin(MARGIN, MARGIN, MARGIN, MARGIN); // OS 5.0未公开的API
        if(title != null && !"".equals(title)){
            HorizontalFieldManager h = new HorizontalFieldManager(){
                protected void paintBackground(Graphics g) {

                    int oldColor = g.getColor();
                    try {
                        g.setColor(Color.RED);
                        g.fillRoundRect(0, 0, Display.getWidth() - MARGIN * 2, getHeight() + (PANDDING *2 + CORNER_RADIUS), CORNER_RADIUS, CORNER_RADIUS);// 画实心的圆角的矩形
                        g.setColor(Color.WHITE);
                        g.drawRoundRect(0, 0, Display.getWidth() - MARGIN * 2, getHeight() + (PANDDING *2 + CORNER_RADIUS), CORNER_RADIUS, CORNER_RADIUS); // 换个颜色，画圆角的边框
                    } finally {
                        g.setColor(oldColor);
                    }
                }
            };
            h.setPadding(PANDDING, PANDDING, PANDDING, PANDDING); // OS 5.0未公开的API
            h.add(new BitmapField(bitmap));
            h.add(new RichTextField(title,NON_FOCUSABLE ){
                protected void layout(int width, int height) {
                    width = Display.getWidth()- MARGIN * 2;
                    super.layout(width, height);
                }
            });
            add(h);
        }
    }

    protected void paintBackground(Graphics g) {
        int oldColor = g.getColor();
        try {
            g.setColor(Color.WHITE);
            g.fillRoundRect(0, 0, getWidth(), getHeight(), 20, 20);// 画实心的圆角的矩形
            g.setColor(Color.WHITE);
            g.drawRoundRect(0, 0, getWidth(), getHeight(), 20, 20); // 换个颜色，画圆角的边框
        } finally {
            g.setColor(oldColor);
        }
    }
}
```
