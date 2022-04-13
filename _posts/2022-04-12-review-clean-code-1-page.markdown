---
layout: post
title:  "Review Clean Code 1Page Title - Clean Code"
subtitle:   "Review Clean Code 1Page Title - Clean Code"
date:   2022-04-12 00:47:27 +0900
categories: review
tags: review cleancode
comments: true
---

## 클린코드란 무엇일까? <br>
## 이 책에서 말하는 클린코드란 다음과 같다.<br>
### 중복을 없애라. <br>
### 한 기능만 수행하라. <br>
### 제대로 표현하라. <br>
### 작게 추상화 하라.<br>
## 깨끗한 코드는 잘 쓴 문장처럼 읽여야 하며, 코드는 추측이 아닌 사실에 기반해야하고 반드시 필요한 내용만 담아야 한다.<br>
## 깨끗한 코드는 주의 깊게 짠 코드다. 누군가 시간을 들여 깔끔하고 단정하게 정리한 코드다. <br>

<br>

```java
public class ImageClass {

    public BufferedImage convert(String sPath, String tPath, String format, int width, int height, String position) {
        Image image;
        int imageWidth;
        int imageHeight;
        double ratio;
        int w;
        int h;

        try{
            image = ImageIO.read(new File(sPath));

            imageWidth = image.getWidth(null);
            imageHeight = image.getHeight(null);

            if(position.equals("W")){

                ratio = (double)width/(double)imageWidth;
                w = (int)(imageWidth * ratio);
                h = (int)(imageHeight * ratio);

            }else if(position.equals("H")){

                ratio = (double)height/(double)imageHeight;
                w = (int)(imageWidth * ratio);
                h = (int)(imageHeight * ratio);

            }else{

                w = width;
                h = height;
            }

            Image resizeImage = image.getScaledInstance(w, h, Image.SCALE_SMOOTH);

            BufferedImage newImage = new BufferedImage(w, h, BufferedImage.TYPE_INT_RGB);
            Graphics g = newImage.getGraphics();
            g.drawImage(resizeImage, 0, 0, null);
            g.dispose();
            ImageIO.write(newImage, format, new File(tPath));

            return newImage;

        }catch (Exception e){

            e.printStackTrace();

        }

        return null;
    }

    public BufferedImage rotate(String sPath, String tPath, String format, int rotate) throws Exception {

        File orgFile = new File(sPath);
        BufferedImage oldImage = ImageIO.read(orgFile);

        BufferedImage newImage = null;

        if(180 == rotate) {
            newImage = new BufferedImage(oldImage.getWidth(),oldImage.getHeight(), oldImage.getType());
        }
        else {
            newImage = new BufferedImage(oldImage.getHeight(),oldImage.getWidth(), oldImage.getType());
        }

        Graphics2D graphics = (Graphics2D) newImage.getGraphics();

        graphics.rotate(Math.toRadians(rotate), newImage.getWidth() / 2, newImage.getHeight() / 2);

        if(180 != rotate) {
            graphics.translate((newImage.getWidth() - oldImage.getWidth()) / 2, (newImage.getHeight() - oldImage.getHeight()) / 2);
        }

        graphics.drawImage(oldImage, 0, 0, oldImage.getWidth(), oldImage.getHeight(), null);

        ImageIO.write(newImage, format, new FileOutputStream(new File(tPath)));

        return newImage;
    }
}
```

### 위 코드를 보자, 정말 의식의 흐름대로 코드가 짜여져있다. 마치 깨진 창문과 같다.창문이 깨진 건물은 누구도 상관하지 않는다는 인상을 풍긴다.<br>
### 그래서 사람들도 관심을 끊고, 창문이 더 깨져도 상관하지 않는다. 결국은 더이상 건들이지도, 건들수도 없는 코드가 되어버린다.<br>
### 위 코드는 중복, 한 기능만 수행, 제대로 표현, 작게 추상화를 모두 어겼다고 판단된다. (필자의 생각)<br>
### 여기 저기 중복된 코드와 하나의 함수에서 여러가지의 일을 하고있으며, convert라는 추상화의 단계가 너무 크다고 생각된다.(이를테면 resize, save 정도가 알맞을 것 같다.)<br>
### 또한, sPath, tPath는 무엇이며, position은 무엇을 의미하는지 모르기 때문에 제대로 표현하지 못했다고 할 수 있을 것이다.<br>
### 이제 하나하나 고쳐나가 보자<br><br>

```java
public class ImageClass {
    private Image image;
    private ImageConverter imageConverter;

    private ImageClass() {}

    private ImageClass(Image image) {
        this.image = image;
        this.imageConverter = new JpgImageConverter();
    }

    public static ImageClass createImageClass(String filePath) {
        try {
            Image image = ImageIO.read(new File(filePath));
            return new ImageClass(image);
        } catch (Exception e) {
            e.printStackTrace();
            return null;
        }
    }

    public void setImageConverter(ImageConverter imageConverter) {
        this.imageConverter = imageConverter;
    }

    public boolean loadImage(String filePath) {
        try {
            image = ImageIO.read(new File(filePath));
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }

        return true;
    }

    public void save(String targetPath, String format, int option) throws Exception {
        BufferedImage newImage = imageConverter.getBufferedImage(image, option);
        ImageIO.write(newImage, format, new File(targetPath));
    }
    
    public void resize(int resizeWidth, int resizeHeight, int option) {
        imageConverter.resize(image,resizeWidth, resizeHeight, option);
    }

    public void convert(int convertWidth, int convertHeight, String mainPosition, int option) throws Exception {
        imageConverter.convert(image, convertWidth, convertHeight, mainPosition, option);
    }

    public void rotate(int rotate) {
        imageConverter.rotate(image, rotate);
    }
}

public interface ImageConverter {

    BufferedImage getBufferedImage(Image image, int option);

    void resize(Image image, int resizeWidth, int resizeHeight, int option);

    void convert(Image image, int convertWidth, int convertHeight, String mainPosition, int option);

    void rotate(Image image, int rotate);
}

public class JpgImageConverter implements ImageConverter{

    public void resize(Image image, int resizeWidth, int resizeHeight, int option) {
        image.getScaledInstance(resizeWidth, resizeHeight, option);
    }

    @Override
    public void convert(Image image, int convertWidth, int convertHeight, String mainPosition, int option) {
        double ratio;
        int oriWidth = image.getWidth(null);
        int oriHeight = image.getHeight(null);

        if(mainPosition.equals("W")){
            ratio = (double)convertWidth/(double)oriWidth;
        }else {
            ratio = (double)convertHeight/(double)oriHeight;
        }
        int w = (int)(oriWidth * ratio);
        int h = (int)(oriHeight * ratio);
        resize(image, w, h, option);
    }

    public BufferedImage getBufferedImage(Image image, int option) {
        BufferedImage newImage = new BufferedImage(image.getWidth(null), image.getHeight(null), option);
        Graphics g = newImage.getGraphics();
        g.drawImage(image, 0, 0, null);
        g.dispose();
        return newImage;
    }

    public void rotate(Image image, int rotate) {
        Graphics2D graphics = (Graphics2D) image.getGraphics();
        graphics.rotate(Math.toRadians(rotate), image.getWidth(null) / 2, image.getHeight(null) / 2);
    }
}
```

### 위 코드는 ImageClass에서는 이미지 로드와 저장의 역할을 맡고, ImageConverter에서는 각각의 포맷에 따라서 resize, convert, rotate하는 방법이 다르다고 가정하고 만들었다.<br>
### ImageConvert는 JpgImageConvert, PngImageConvert가 있을 수 있으며 각각의 포맷에 따라서 갈아낄수 있도록 설계 했다.<br>

1. 중복 제거 <br>
    - ImageIO.resize(), ImageIO.write() 와 같이 중복되는 코드는 공통 메소드로 분리했다.
<br>

2. 한 기능만 수행 / 작게 추상화 <br>
    - ImageClass는 이미지 로드, 저장의 역할을 하고, ImageConverter는 이미지를 조작하는 resize, convert, rotate하는 역할로 분담하였다.
    - 기존의 ImageClass클래스는 자신의 역할에 맞는 일을 하고있지 않을뿐 더러 각각의 메소드가 이미지를 불러오고 변환하고 리사이즈하고 화면에 그리고 저장까지 많은 기능을 담당 하고있었다. 이것을 convert -> save, resize, convert로 분리하여 작게 추상화 하였다.
<br>

3. 제대로 표현
    - 기존의 변수 이름의 경우 sPath, tPath와 같이 한번 봐서는 알 수 없는 변수명이 었지만, filePath, targetPath와 같이 좀 더 알기 쉬운 표현을 적용 하였다.