---
title: BlackBerry 开发：九宫格
date: 2012-05-21 15:53:17
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
 *
 * @author Rolex
 * @date 2011-9-21
 */
public class HomeScreen extends MainScreen{

    	public HomeScreen() {
          HorizontalBitmapFieldSet set1 = new HorizontalBitmapFieldSet();
          BitmapBackgroundButtonField bitmap1 = new BitmapBackgroundButtonField("登录",Field.FOCUSABLE,Bitmap.getBitmapResource("com/unionpay/mobilepay/mpos/image/home/icon.png"),Bitmap.getBitmapResource("com/unionpay/mobilepay/mpos/image/home/icon_login.png"));
          bitmap1.setChangeListener(new FieldChangeListener() {

                public void fieldChanged(Field field, int context) {
                      pushScreen(new com.unionpay.mobilepay.mpos.profile.TabScreen(null));

                }
          });
          BitmapBackgroundButtonField bitmap2 = new BitmapBackgroundButtonField("注册",Field.FOCUSABLE,Bitmap.getBitmapResource("com/unionpay/mobilepay/mpos/image/home/icon.png"),Bitmap.getBitmapResource("com/unionpay/mobilepay/mpos/image/home/icon_register.png"));
          bitmap2.setChangeListener(new FieldChangeListener() {

                public void fieldChanged(Field field, int context) {
                      pushScreen(new com.unionpay.mobilepay.mpos.profile.TabScreen(null));

                }
          });
          BitmapBackgroundButtonField bitmap3 = new BitmapBackgroundButtonField("帮助",Field.FOCUSABLE,Bitmap.getBitmapResource("com/unionpay/mobilepay/mpos/image/home/icon.png"),Bitmap.getBitmapResource("com/unionpay/mobilepay/mpos/image/home/icon_help.png"));
          bitmap3.setChangeListener(new FieldChangeListener() {

                public void fieldChanged(Field field, int context) {
                      Dialog.alert("help");

                }
          });
          set1.add(bitmap1);
          set1.add(bitmap2);
          set1.add(bitmap3);
          HorizontalBitmapFieldSet set2 = new HorizontalBitmapFieldSet();
          BitmapBackgroundButtonField bitmap4 = new BitmapBackgroundButtonField("交易管理",Field.FOCUSABLE,Bitmap.getBitmapResource("com/unionpay/mobilepay/mpos/image/home/icon.png"),Bitmap.getBitmapResource("com/unionpay/mobilepay/mpos/image/home/icon_transaction.png"));
          bitmap4.setChangeListener(new FieldChangeListener() {

                public void fieldChanged(Field field, int context) {
                      pushScreen(new TradePay());

                }
          });
          BitmapBackgroundButtonField bitmap5 = new BitmapBackgroundButtonField("银行卡管理",Field.FOCUSABLE,Bitmap.getBitmapResource("com/unionpay/mobilepay/mpos/image/home/icon.png"),Bitmap.getBitmapResource("com/unionpay/mobilepay/mpos/image/home/icon_bankcard.png"));
          bitmap5.setChangeListener(new FieldChangeListener() {

                public void fieldChanged(Field field, int context) {
                      pushScreen(new CardManager());

                }
          });
          BitmapBackgroundButtonField bitmap6 = new BitmapBackgroundButtonField("用户管理",Field.FOCUSABLE,Bitmap.getBitmapResource("com/unionpay/mobilepay/mpos/image/home/icon.png"),Bitmap.getBitmapResource("com/unionpay/mobilepay/mpos/image/home/icon_profile.png"));
          bitmap6.setChangeListener(new FieldChangeListener() {

                public void fieldChanged(Field field, int context) {
                      pushScreen(new TabScreenIn(null));

                }
          });
          set2.add(bitmap4);
          set2.add(bitmap5);
          set2.add(bitmap6);
          HorizontalBitmapFieldSet set3 = new HorizontalBitmapFieldSet();
          BitmapBackgroundButtonField bitmap7 = new BitmapBackgroundButtonField("关于",Field.FOCUSABLE,Bitmap.getBitmapResource("com/unionpay/mobilepay/mpos/image/home/icon.png"),Bitmap.getBitmapResource("com/unionpay/mobilepay/mpos/image/home/icon_about.png"));
          bitmap7.setChangeListener(new FieldChangeListener() {

                public void fieldChanged(Field field, int context) {
                      Dialog.alert("login");

                }
          });
          BitmapBackgroundButtonField bitmap8 = new BitmapBackgroundButtonField("设置",Field.FOCUSABLE,Bitmap.getBitmapResource("com/unionpay/mobilepay/mpos/image/home/icon.png"),Bitmap.getBitmapResource("com/unionpay/mobilepay/mpos/image/home/icon_settings.png"));
          bitmap8.setChangeListener(new FieldChangeListener() {

                public void fieldChanged(Field field, int context) {
                      Dialog.alert("reg");

                }
          });
          BitmapBackgroundButtonField bitmap9 = new BitmapBackgroundButtonField("更多",Field.FOCUSABLE,Bitmap.getBitmapResource("com/unionpay/mobilepay/mpos/image/home/icon.png"),Bitmap.getBitmapResource("com/unionpay/mobilepay/mpos/image/home/icon_more.png"));
          bitmap9.setChangeListener(new FieldChangeListener() {

                public void fieldChanged(Field field, int context) {
                      Dialog.alert("help");

                }
          });
          set3.add(bitmap7);
          set3.add(bitmap8);
          set3.add(bitmap9);
          add(set1);
          add(set2);
          add(set3);

    	}

    	public static void pushScreen(MainScreen screen){
          UiApplication.getUiApplication().pushScreen(screen);
    	}
}
```
