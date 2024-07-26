---
title: BlackBerry 开发：自定义输入框
date: 2012-04-05 15:53:17
tags:
 - BlackBerry
category: 
 - Java
thumbnail: 
author: bsyonline
lede: "没有摘要"
---


```JAVA
public InputBoxField(int boxWigth,int boxHeight,String boxText){
    super(Manager.NO_VERTICAL_SCROLL);//禁用垂直滚动
    this._boxHeight = boxHeight;
    this._boxWigth = boxWigth;
    this._boxText = boxText;
		_vfm = new VerticalFieldManager(Manager.NO_VERTICAL_SCROLL);
		//EditField.NO_NEWLINE    设置输入框不可换行
    _ef = new EditField("","",10,EditField.NO_NEWLINE){
				//创建一个EditedField，并设置其中字符的显示颜色。
		    public void paint(Graphics g){
		        //getManager().invalidate();
		        g.setColor(Color.WHITE);
		        super.paint(g);
		    }
    };
    _ef.setBackground(BackgroundFactory.createSolidBackground(Color.BLACK));//设置EditedField的背景颜色
    _vfm.add(_ef);
    _vfm.setPadding((_boxHeight-_ef.getFont().getHeight())/2, 10, 10, 5);//设置输入区域和外边框的间距
    add(_vfm);
}
```
然后我们需要覆写sublayout方法，告诉布局管理器该组件的大小
```JAVA
protected void sublayout(int maxWidth, int maxHeight) {
    if(_boxWigth == 0){
            _boxWigth = maxWidth;
    }
    if(_boxHeight == 0){
            _boxHeight = maxHeight;
    }
    //super.sublayout(maxWidth, maxHeight);
    setPositionChild(_vfm, 2, 0);
    layoutChild(_vfm, _boxWigth, _boxHeight);
    setExtent(_boxWigth, _boxHeight);
    }
```
覆写paint方法，在原有的EditedField的四周换上我们想要的边框并处理焦点事件。
```Java
protected void paint(Graphics g) {
    super.paint(g);
    int oldColor = g.getColor();
    if(this.isFocus()){
            if(_ef.getText().equals(_boxText)){
                    _ef.setText("");
            }
            g.setColor(Color.BLUE);
            for(int i =0;i<4;i++){

                    g.drawRoundRect(i,i,getWidth()-i*2,getHeight()-i*2,15,15);
            }       

    }else{
            if(_ef.getText().equals("")){
                    _ef.setText(_boxText);
            }

            g.setColor(Color.BLUE);
            g.drawRoundRect(3, 3, getWidth()-6, getHeight()-6,15,15);                }
    g.setColor(oldColor);
}
```
接下来覆写onFocus和onUnfocus方法
```java
protected void onFocus(int direction) {
	  super.onFocus(direction);
	  _ef.setCursorPosition(_ef.getTextLength());
	  invalidate();
}

protected void onUnfocus() {
    super.onUnfocus();
    invalidate();
}
```
