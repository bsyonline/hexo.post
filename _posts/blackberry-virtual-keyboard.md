---
title: BlackBerry 开发：虚拟键盘
date: 2012-05-02 15:53:17
tags:
 - BlackBerry
category: 
 - Java
thumbnail: 
author: bsyonline
lede: "没有摘要"
---


```JAVA
import java.io.InputStream;

import javax.microedition.media.Player;
import javax.microedition.media.control.VolumeControl;

import net.rim.device.api.system.Application;
import net.rim.device.api.system.Display;
import net.rim.device.api.system.KeyListener;
import net.rim.device.api.ui.Color;
import net.rim.device.api.ui.Field;
import net.rim.device.api.ui.FieldChangeListener;
import net.rim.device.api.ui.FocusChangeListener;
import net.rim.device.api.ui.Graphics;
import net.rim.device.api.ui.Keypad;
import net.rim.device.api.ui.Manager;
import net.rim.device.api.ui.VirtualKeyboard;
import net.rim.device.api.ui.component.EditField;
import net.rim.device.api.ui.component.LabelField;
import net.rim.device.api.ui.component.PasswordEditField;
import net.rim.device.api.ui.container.HorizontalFieldManager;
import net.rim.device.api.ui.container.PopupScreen;
import net.rim.device.api.ui.container.VerticalFieldManager;

public class TestScreen extends PopupScreen implements FieldChangeListener,KeyListener {


   PasswordEditField passwd;//输入框
   VerticalFieldManager vfm;//密码键盘的布局
   HorizontalFieldManager buttonHfm;//按钮的水平布局
   VerticalFieldManager numVfm;//数字键盘的垂直布局
   VerticalFieldManager charVfm;//字母键盘垂直布局
   VerticalFieldManager bcharVfm;//字母键盘垂直布局
   VerticalFieldManager signVfm;//符号键盘的水平布局
   CustomButtonField button1;
   CustomButtonField button2;
   CustomButtonField button3;
   VirtualKeyboard virtKbd;//虚拟键盘
   CustomButtonField cbf;

   EditField _edit;
   EditField _edit1;

   public TestScreen(EditField edit,EditField edit1) {
          super(new VerticalFieldManager()
          {
          }
          );
          _edit = edit;
          _edit1 = edit1;
          virtKbd = this.getVirtualKeyboard();//获得当前屏幕的虚拟键盘

          vfm = new VerticalFieldManager();
          LabelField label = new LabelField("密码键盘");
          passwd = new PasswordEditField();
          passwd.setFocusListener(new FocusChangeListener() {
               public void focusChanged(Field field, int eventType) {
                       if(eventType == FOCUS_GAINED){
                            //virtKbd.setVisibility(VirtualKeyboard.HIDE_FORCE);
                       }
                  }
        });
          buttonHfm = new HorizontalFieldManager();
          button1 = new CustomButtonField("数字", CustomButtonField.BUTTON, Field.FOCUSABLE);
          button1.setChangeListener(this);
          button2 = new CustomButtonField("字母", CustomButtonField.BUTTON, Field.FOCUSABLE);
          button2.setChangeListener(this);
          button3 = new CustomButtonField("符号", CustomButtonField.BUTTON, Field.FOCUSABLE);
          button3.setChangeListener(this);
          buttonHfm.add(button1);
          buttonHfm.add(button2);
          buttonHfm.add(button3);
          numVfm = new VerticalFieldManager(Manager.FIELD_HCENTER);
          charVfm = new VerticalFieldManager(Manager.FIELD_HCENTER);
          bcharVfm = new VerticalFieldManager(Manager.FIELD_HCENTER);
          signVfm = new VerticalFieldManager(Manager.FIELD_HCENTER);

          String num[] = KeyboardUtil.randomArray(new String[] { "1", "2", "3", "4", "5", "6", "7", "8",
                       "9", "0", "确定", "←" });
          HorizontalFieldManager h1 = new HorizontalFieldManager();
          HorizontalFieldManager h2 = new HorizontalFieldManager();
          HorizontalFieldManager h3 = new HorizontalFieldManager();
          cbf = new CustomButtonField(num[0], CustomButtonField.SIGN, Field.FOCUSABLE);
          cbf.setChangeListener(this);
          h1.add(cbf);
          cbf = new CustomButtonField(num[1], CustomButtonField.SIGN, Field.FOCUSABLE);
          cbf.setChangeListener(this);
          h1.add(cbf);
          cbf = new CustomButtonField(num[2], CustomButtonField.SIGN, Field.FOCUSABLE);
          cbf.setChangeListener(this);
          h1.add(cbf);
          cbf = new CustomButtonField(num[3], CustomButtonField.SIGN, Field.FOCUSABLE);
          cbf.setChangeListener(this);
          h1.add(cbf);
          cbf = new CustomButtonField(num[4], CustomButtonField.SIGN, Field.FOCUSABLE);
          cbf.setChangeListener(this);
          h2.add(cbf);
          cbf = new CustomButtonField(num[5], CustomButtonField.SIGN, Field.FOCUSABLE);
          cbf.setChangeListener(this);
          h2.add(cbf);
          cbf = new CustomButtonField(num[6], CustomButtonField.SIGN, Field.FOCUSABLE);
          cbf.setChangeListener(this);
          h2.add(cbf);
          cbf = new CustomButtonField(num[7], CustomButtonField.SIGN, Field.FOCUSABLE);
          cbf.setChangeListener(this);
          h2.add(cbf);
          cbf = new CustomButtonField(num[8], CustomButtonField.SIGN, Field.FOCUSABLE);
          cbf.setChangeListener(this);
          h3.add(cbf);
          cbf = new CustomButtonField(num[9], CustomButtonField.SIGN, Field.FOCUSABLE);
          cbf.setChangeListener(this);
          h3.add(cbf);
          cbf = new CustomButtonField(num[10], CustomButtonField.SIGN, Field.FOCUSABLE);
          cbf.setChangeListener(this);
          h3.add(cbf);
          cbf = new CustomButtonField(num[11], CustomButtonField.SIGN, Field.FOCUSABLE);
         cbf.setChangeListener(this);
         h3.add(cbf);
          numVfm.add(h1);
          numVfm.add(h2);
          numVfm.add(h3);
         //字母键盘
         HorizontalFieldManager hfm = new HorizontalFieldManager(Manager.FIELD_HCENTER);

           // Rectangular button
         cbf = new CustomButtonField("q", CustomButtonField.RECTANGLE, Field.FOCUSABLE);
         cbf.setChangeListener(this);
           hfm.add(cbf);
           cbf = new CustomButtonField("w", CustomButtonField.RECTANGLE, Field.FOCUSABLE);
           cbf.setChangeListener(this);
           hfm.add(cbf);
           cbf = new CustomButtonField("e", CustomButtonField.RECTANGLE, Field.FOCUSABLE);
           cbf.setChangeListener(this);
           hfm.add(cbf);
           cbf = new CustomButtonField("r", CustomButtonField.RECTANGLE, Field.FOCUSABLE);
           cbf.setChangeListener(this);
           hfm.add(cbf);
           cbf = new CustomButtonField("t", CustomButtonField.RECTANGLE, Field.FOCUSABLE);
           cbf.setChangeListener(this);
           hfm.add(cbf);
           cbf = new CustomButtonField("y", CustomButtonField.RECTANGLE, Field.FOCUSABLE);
           cbf.setChangeListener(this);
           hfm.add(cbf);
           cbf = new CustomButtonField("u", CustomButtonField.RECTANGLE, Field.FOCUSABLE);
           cbf.setChangeListener(this);
           hfm.add(cbf);
           cbf = new CustomButtonField("i", CustomButtonField.RECTANGLE, Field.FOCUSABLE);
           cbf.setChangeListener(this);
           hfm.add(cbf);
           cbf = new CustomButtonField("o", CustomButtonField.RECTANGLE, Field.FOCUSABLE);
           cbf.setChangeListener(this);
           hfm.add(cbf);
           cbf = new CustomButtonField("p", CustomButtonField.RECTANGLE, Field.FOCUSABLE);
           cbf.setChangeListener(this);
           hfm.add(cbf);

           HorizontalFieldManager hfm1 = new HorizontalFieldManager(Manager.FIELD_HCENTER);

           // Rectangular button
           cbf = new CustomButtonField("a", CustomButtonField.RECTANGLE, Field.FOCUSABLE);
           cbf.setChangeListener(this);
           hfm1.add(cbf);
           cbf = new CustomButtonField("s", CustomButtonField.RECTANGLE, Field.FOCUSABLE);
           cbf.setChangeListener(this);
           hfm1.add(cbf);
           cbf = new CustomButtonField("d", CustomButtonField.RECTANGLE, Field.FOCUSABLE);
           cbf.setChangeListener(this);
           hfm1.add(cbf);
           cbf = new CustomButtonField("f", CustomButtonField.RECTANGLE, Field.FOCUSABLE);
           cbf.setChangeListener(this);
           hfm1.add(cbf);
           cbf = new CustomButtonField("g", CustomButtonField.RECTANGLE, Field.FOCUSABLE);
           cbf.setChangeListener(this);
           hfm1.add(cbf);
           cbf = new CustomButtonField("h", CustomButtonField.RECTANGLE, Field.FOCUSABLE);
           cbf.setChangeListener(this);
           hfm1.add(cbf);
           cbf = new CustomButtonField("j", CustomButtonField.RECTANGLE, Field.FOCUSABLE);
           cbf.setChangeListener(this);
           hfm1.add(cbf);
           cbf = new CustomButtonField("k", CustomButtonField.RECTANGLE, Field.FOCUSABLE);
           cbf.setChangeListener(this);
           hfm1.add(cbf);
           cbf = new CustomButtonField("l", CustomButtonField.RECTANGLE, Field.FOCUSABLE);
           cbf.setChangeListener(this);
           hfm1.add(cbf);
           cbf = new CustomButtonField("←", CustomButtonField.RECTANGLE, Field.FOCUSABLE);
           cbf.setChangeListener(this);
           hfm1.add(cbf);

           HorizontalFieldManager hfm2 = new HorizontalFieldManager(Manager.FIELD_HCENTER);
           cbf = new CustomButtonField("↑", CustomButtonField.RECTANGLE, Field.FOCUSABLE);
           cbf.setChangeListener(this);
           hfm2.add(cbf);
           // Rectangular button
           cbf = new CustomButtonField("z", CustomButtonField.RECTANGLE, Field.FOCUSABLE);
           cbf.setChangeListener(this);
           hfm2.add(cbf);
           cbf = new CustomButtonField("x", CustomButtonField.RECTANGLE, Field.FOCUSABLE);
           cbf.setChangeListener(this);
           hfm2.add(cbf);
           cbf = new CustomButtonField("d", CustomButtonField.RECTANGLE, Field.FOCUSABLE);
           cbf.setChangeListener(this);
           hfm2.add(cbf);
           cbf = new CustomButtonField("v", CustomButtonField.RECTANGLE, Field.FOCUSABLE);
           cbf.setChangeListener(this);
           hfm2.add(cbf);
           cbf = new CustomButtonField("b", CustomButtonField.RECTANGLE, Field.FOCUSABLE);
           cbf.setChangeListener(this);
           hfm2.add(cbf);
           cbf = new CustomButtonField("n", CustomButtonField.RECTANGLE, Field.FOCUSABLE);
           cbf.setChangeListener(this);
           hfm2.add(cbf);
           cbf = new CustomButtonField("m", CustomButtonField.RECTANGLE, Field.FOCUSABLE);
           cbf.setChangeListener(this);
           hfm2.add(cbf);
           cbf = new CustomButtonField("确定", CustomButtonField.RECTANGLE, Field.FOCUSABLE);
           cbf.setChangeListener(this);
           hfm2.add(cbf);

           charVfm.add(hfm);
           charVfm.add(hfm1);
           charVfm.add(hfm2);



           /***********************************/
           //大写字母键盘
              HorizontalFieldManager bhfm = new HorizontalFieldManager(Manager.FIELD_HCENTER);

                // Rectangular button
              cbf = new CustomButtonField("Q", CustomButtonField.BIG, Field.FOCUSABLE);
              cbf.setChangeListener(this);
                bhfm.add(cbf);
                cbf = new CustomButtonField("W", CustomButtonField.BIG, Field.FOCUSABLE);
                cbf.setChangeListener(this);
                bhfm.add(cbf);
                cbf = new CustomButtonField("E", CustomButtonField.BIG, Field.FOCUSABLE);
                cbf.setChangeListener(this);
                bhfm.add(cbf);
                cbf = new CustomButtonField("R", CustomButtonField.BIG, Field.FOCUSABLE);
                cbf.setChangeListener(this);
                bhfm.add(cbf);
                cbf = new CustomButtonField("T", CustomButtonField.BIG, Field.FOCUSABLE);
                cbf.setChangeListener(this);
                bhfm.add(cbf);
                cbf = new CustomButtonField("Y", CustomButtonField.BIG, Field.FOCUSABLE);
                cbf.setChangeListener(this);
                bhfm.add(cbf);
                cbf = new CustomButtonField("U", CustomButtonField.BIG, Field.FOCUSABLE);
                cbf.setChangeListener(this);
                bhfm.add(cbf);
                cbf = new CustomButtonField("I", CustomButtonField.BIG, Field.FOCUSABLE);
                cbf.setChangeListener(this);
                bhfm.add(cbf);
                cbf = new CustomButtonField("O", CustomButtonField.BIG, Field.FOCUSABLE);
                cbf.setChangeListener(this);
                bhfm.add(cbf);
                cbf = new CustomButtonField("P", CustomButtonField.BIG, Field.FOCUSABLE);
                cbf.setChangeListener(this);
                bhfm.add(cbf);

                HorizontalFieldManager bhfm1 = new HorizontalFieldManager(Manager.FIELD_HCENTER);

                // Rectangular button
                cbf = new CustomButtonField("A", CustomButtonField.BIG, Field.FOCUSABLE);
                cbf.setChangeListener(this);
                bhfm1.add(cbf);
                cbf = new CustomButtonField("S", CustomButtonField.BIG, Field.FOCUSABLE);
                cbf.setChangeListener(this);
                bhfm1.add(cbf);
                cbf = new CustomButtonField("D", CustomButtonField.BIG, Field.FOCUSABLE);
                cbf.setChangeListener(this);
                bhfm1.add(cbf);
                cbf = new CustomButtonField("F", CustomButtonField.BIG, Field.FOCUSABLE);
                cbf.setChangeListener(this);
                bhfm1.add(cbf);
                cbf = new CustomButtonField("G", CustomButtonField.BIG, Field.FOCUSABLE);
                cbf.setChangeListener(this);
                bhfm1.add(cbf);
                cbf = new CustomButtonField("H", CustomButtonField.BIG, Field.FOCUSABLE);
                cbf.setChangeListener(this);
                bhfm1.add(cbf);
                cbf = new CustomButtonField("J", CustomButtonField.BIG, Field.FOCUSABLE);
                cbf.setChangeListener(this);
                bhfm1.add(cbf);
                cbf = new CustomButtonField("K", CustomButtonField.BIG, Field.FOCUSABLE);
                cbf.setChangeListener(this);
                bhfm1.add(cbf);
                cbf = new CustomButtonField("L", CustomButtonField.BIG, Field.FOCUSABLE);
                cbf.setChangeListener(this);
                bhfm1.add(cbf);
                cbf = new CustomButtonField("←", CustomButtonField.BIG, Field.FOCUSABLE);
                cbf.setChangeListener(this);
                bhfm1.add(cbf);

                HorizontalFieldManager bhfm2 = new HorizontalFieldManager(Manager.FIELD_HCENTER);
                cbf = new CustomButtonField("↓", CustomButtonField.BIG, Field.FOCUSABLE);
                cbf.setChangeListener(this);
                bhfm2.add(cbf);
                // Rectangular button
                cbf = new CustomButtonField("Z", CustomButtonField.BIG, Field.FOCUSABLE);
                cbf.setChangeListener(this);
                bhfm2.add(cbf);
                cbf = new CustomButtonField("X", CustomButtonField.BIG, Field.FOCUSABLE);
                cbf.setChangeListener(this);
                bhfm2.add(cbf);
                cbf = new CustomButtonField("C", CustomButtonField.BIG, Field.FOCUSABLE);
                cbf.setChangeListener(this);
                bhfm2.add(cbf);
                cbf = new CustomButtonField("V", CustomButtonField.BIG, Field.FOCUSABLE);
                cbf.setChangeListener(this);
                bhfm2.add(cbf);
                cbf = new CustomButtonField("B", CustomButtonField.BIG, Field.FOCUSABLE);
                cbf.setChangeListener(this);
                bhfm2.add(cbf);
                cbf = new CustomButtonField("N", CustomButtonField.BIG, Field.FOCUSABLE);
                cbf.setChangeListener(this);
                bhfm2.add(cbf);
                cbf = new CustomButtonField("M", CustomButtonField.BIG, Field.FOCUSABLE);
                cbf.setChangeListener(this);
                bhfm2.add(cbf);
                cbf = new CustomButtonField("确定", CustomButtonField.BIG, Field.FOCUSABLE);
                cbf.setChangeListener(this);
                bhfm2.add(cbf);

                bcharVfm.add(bhfm);
                bcharVfm.add(bhfm1);
                bcharVfm.add(bhfm2);
           /***********************************/
         //符号键盘
         String sign[] = new String[] { "!", "@", "#", "$", "%", "^", "&", "←",
                       "(", ")", "*", "确定" };
         HorizontalFieldManager sh1 = new HorizontalFieldManager();
         HorizontalFieldManager sh2 = new HorizontalFieldManager();
         HorizontalFieldManager sh3 = new HorizontalFieldManager();
          cbf = new CustomButtonField(sign[0], CustomButtonField.SIGN, Field.FOCUSABLE);
          cbf.setChangeListener(this);
          sh1.add(cbf);
          cbf = new CustomButtonField(sign[1], CustomButtonField.SIGN, Field.FOCUSABLE);
          cbf.setChangeListener(this);
          sh1.add(cbf);
          cbf = new CustomButtonField(sign[2], CustomButtonField.SIGN, Field.FOCUSABLE);
          cbf.setChangeListener(this);
          sh1.add(cbf);
          cbf = new CustomButtonField(sign[3], CustomButtonField.SIGN, Field.FOCUSABLE);
          cbf.setChangeListener(this);
          sh1.add(cbf);
          cbf = new CustomButtonField(sign[4], CustomButtonField.SIGN, Field.FOCUSABLE);
          cbf.setChangeListener(this);
          sh2.add(cbf);
          cbf = new CustomButtonField(sign[5], CustomButtonField.SIGN, Field.FOCUSABLE);
          cbf.setChangeListener(this);
          sh2.add(cbf);
          cbf = new CustomButtonField(sign[6], CustomButtonField.SIGN, Field.FOCUSABLE);
          cbf.setChangeListener(this);
          sh2.add(cbf);
          cbf = new CustomButtonField(sign[7], CustomButtonField.SIGN, Field.FOCUSABLE);
          cbf.setChangeListener(this);
          sh2.add(cbf);
          cbf = new CustomButtonField(sign[8], CustomButtonField.SIGN, Field.FOCUSABLE);
          cbf.setChangeListener(this);
          sh3.add(cbf);
          cbf = new CustomButtonField(sign[9], CustomButtonField.SIGN, Field.FOCUSABLE);
          cbf.setChangeListener(this);
          sh3.add(cbf);
          cbf = new CustomButtonField(sign[10], CustomButtonField.SIGN, Field.FOCUSABLE);
          cbf.setChangeListener(this);
          sh3.add(cbf);
          cbf = new CustomButtonField(sign[11], CustomButtonField.SIGN, Field.FOCUSABLE);
         cbf.setChangeListener(this);
         sh3.add(cbf);
         signVfm.add(sh1);
         signVfm.add(sh2);
         signVfm.add(sh3);


          vfm.add(label);
          vfm.add(passwd);
          vfm.add(buttonHfm);
          vfm.add(numVfm);
          add(vfm);
          Application.getApplication().addKeyListener(this);           
   }

   protected void paintBackground(Graphics graphics) {
        graphics.setColor(Color.WHITE);
   }

   int index = 0;//标识显示的键盘
   public void fieldChanged(Field field, int context) {
        play();
        if(field instanceof CustomButtonField){
             //virtKbd.setVisibility(VirtualKeyboard.HIDE_FORCE);
             if("数字".equals(((CustomButtonField)field).getText())){
                  if(index == 1){
                       vfm.replace(charVfm, numVfm);
                  }else if(index == 2){
                       vfm.replace(signVfm, numVfm);
                  }else if(index == 3){
                       vfm.replace(bcharVfm, numVfm);
                  }
                  index = 0;
             }else if("字母".equals(((CustomButtonField)field).getText())){
                  if(index == 0){
                       vfm.replace(numVfm, charVfm);
                  }else if(index == 2){
                       vfm.replace(signVfm, charVfm);
                  }else if(index == 3){
                       vfm.replace(bcharVfm, charVfm);
                  }
                  index = 1;
             }else if("符号".equals(((CustomButtonField)field).getText())){
                  if(index == 0){
                       vfm.replace(numVfm, signVfm);
                  }else if(index == 1){
                       vfm.replace(charVfm, signVfm);
                  }else if(index == 3){
                       vfm.replace(bcharVfm, signVfm);
                  }
                  index = 2;
             }else if("←".equals(((CustomButtonField)field).getText())){
                  if(passwd.getText().length()>0){
                       passwd.setText(passwd.getText().substring(0, passwd.getText().length()-1));
                  }
             }else if("确定".equals(((CustomButtonField)field).getText())){
                  _edit.setText(passwd.getText());
                  _edit1.setFocus();
                  this.close();
             }else if("↑".equals(((CustomButtonField)field).getText())){
                  vfm.replace(charVfm, bcharVfm);
                  index = 3;
             }else if("↓".equals(((CustomButtonField)field).getText())){
                  vfm.replace(bcharVfm, charVfm);
                  index = 1;
             }else{
                  passwd.setText(passwd.getText() + ((CustomButtonField)field).getText());
             }
             passwd.setFocus();//设置点击按钮失去焦点
        }
   }

   public void play(){
        // create an instance of the player from the InputStream
        Class clazz;
        try {
             clazz = this.getClass();
            InputStream is = clazz.getResourceAsStream("notify.wav");
            //create an instance of the player from the InputStream
            Player player = javax.microedition.media.Manager.createPlayer(is, "audio/mpeg");

             player.realize();
             player.prefetch();
             VolumeControl volumeControl = (VolumeControl) player.getControl("VolumeControl");
             volumeControl.setLevel(100);
             // start the player
             player.start();
        } catch (Exception e) {
             // TODO Auto-generated catch block
             e.printStackTrace();
        }
   }

   protected void sublayout(int width, int height) {
        layoutDelegate(Display.getWidth()-10, 210);
        setPosition(-15, 265);
        setExtent(Display.getWidth()-10, 180);
   }


	 public boolean keyChar( char key, int status, int time ) {
       if(key == Keypad.KEY_ESCAPE){
            _edit1.setFocus();
             this.close();
             return true;
       }

      return false;
  }

  public boolean keyDown(int keycode, int time)
  {
       if(keycode == Keypad.KEY_ESCAPE){
            _edit1.setFocus();
             this.close();
             return true;
       }
      return false;
  }

  public boolean keyRepeat(int keycode, int time)
  {
      return false;
  }

  public boolean keyStatus(int keycode, int time)
  {
      return false;
  }

  public boolean keyUp(int keycode, int time)
  {
      return false;
  }   

}

/**
* CustomButtonField.java
*
* Copyright � 1998-2010 Research In Motion Ltd.
*
* Note: For the sake of simplicity, this sample application may not leverage
* resource bundles and resource strings.  However, it is STRONGLY recommended
* that application developers make use of the localization features available
* within the BlackBerry development platform to ensure a seamless application
* experience across a variety of languages and geographies.  For more information
* on localizing your application, please refer to the BlackBerry Java Development
* Environment Development Guide associated with this release.
*/

import net.rim.device.api.ui.*;
import net.rim.device.api.system.*;

/**
* CustomButtonField - class which creates button fields of various shapes.
* Demonstrates how to create custom ui fields.
*/
public class CustomButtonField extends Field implements DrawStyle {
   static final int RECTANGLE = 1;
   static final int SIGN = 2;
   static final int BIG = 3;
   static final int BUTTON = 4;

   private String _label;
   private int _shape;
   private Font _font;
   private int _labelHeight;
   private int _labelWidth;

   /**
   * Constructs a button with specified label, and default style and shape.
   */
   public CustomButtonField(String label) {
      this(label, RECTANGLE, 0);
   }

   /**
   * Constructs a button with specified label and shape, and default style.
   */
   public CustomButtonField(String label, int shape) {
      this(label, shape, Field.FIELD_BOTTOM);
   }

   /**
   * Constructs a button with specified label and style, and default shape.
   */
   public CustomButtonField(String label, long style) {
      this(label, RECTANGLE, style);
   }

   /**
   * Constructs a button with specified label, shape, and style.
   */
   public CustomButtonField(String label, int shape, long style) {
      super(style);

      _label = label;
      _shape = shape;
      _font = getFont();
      _labelHeight = _font.getHeight();
      _labelWidth = _font.getAdvance(_label);
      setMargin(2, 3, 2, 3);
      // setPadding(2,0,-2,0);
   }

   /**
   * @return The text on the button
   */
   String getText() {
        return _label;
   }

   /**
   * Field implementation.
   *
   * @see net.rim.device.api.ui.Field#getPreferredWidth()
   */
   public int getPreferredWidth() {
      if (_shape == SIGN) {
         return 83;
      }
      if (_shape == BIG) {
         return _labelWidth + 20;
      }
      if (_shape == BUTTON) {
         return _labelWidth + 78;
      }
      return _labelWidth + 22;
   }

   /**
   * Field implementation.
   *
   * @see net.rim.device.api.ui.Field#getPreferredHeight()
   */
   public int getPreferredHeight() {
      if (_shape == SIGN) {
         return 30;
      }
      return _labelHeight + 10;
   }

   /**
   * Field implementation.
   *
   * @see net.rim.device.api.ui.Field#drawFocus(Graphics, boolean)
   */
   protected void drawFocus(Graphics graphics, boolean on) {
      graphics.invert(1, 1, getWidth() - 2, getHeight() - 2);
   }

   /**
   * Field implementation.
   *
   * @see net.rim.device.api.ui.Field#layout(int, int)
   */
   protected void layout(int width, int height) {
      // Update the cached font - in case it has been changed.
      _font = getFont();
      _labelHeight = _font.getHeight();
      _labelWidth = _font.getAdvance(_label);

      // Calc width.
      width = Math.min(width, getPreferredWidth());

      // Calc height.
      height = Math.min(height, getPreferredHeight());

      // Set dimensions.
      setExtent(width, height);
   }

   /**
   * Field implementation.
   *
   * @see net.rim.device.api.ui.Field#paint(Graphics)
   */
   protected void paint(Graphics graphics) {
      int textX, textY, textWidth;
      int w = getWidth();
      int h = getHeight();

      graphics.drawRect(0, 0, w, h);

      textX = 4;
      textY = 2;
      textWidth = w - 6;

      graphics.drawText(_label, textX, textY, (int) (getStyle()
                & DrawStyle.ELLIPSIS | DrawStyle.HALIGN_MASK), textWidth);
   }

   /**
   * Overridden so that the Event Dispatch thread can catch this event instead
   * of having it be caught here.
   *
   * @see net.rim.device.api.ui.Field#navigationClick(int, int)
   */
   protected boolean navigationClick(int status, int time) {
      fieldChangeNotify(0);
      return true;
   }

}
```
