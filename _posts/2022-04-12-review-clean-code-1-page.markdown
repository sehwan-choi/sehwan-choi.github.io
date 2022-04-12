---
layout: post
title:  "Review Clean Code 1Page Title - Clean Code"
subtitle:   "Review Clean Code 1Page Title - Clean Code"
date:   2022-04-12 00:47:27 +0900
categories: review
tags: review cleancode
comments: true
---

클린코드란 무엇일까? <br>
이 책에서 말하는 클린코드란 다음과 같다.<br>
중복을 없애라. <br>
한 기능만 수행하라. <br>
제대로 표현하라. <br>
작게 추상화 하라.<br>
깨끗한 코드는 잘 쓴 문장처럼 읽여야 하며, 코드는 추측이 아닌 사실에 기반해야하고 반드시 필요한 내용만 담아야 한다.<br>
깨끗한 코드는 주의 깊게 짠 코드다. 누군가 시간을 들여 깔끔하고 단정하게 정리한 코드다.<br>

```java
public class ImageConverter {

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

위 코드를 보자, 정말 의식의 흐름대로 코드가 짜여져있다. 마치 깨진 창문과 같다.창문이 깨진 건물은 누구도 상관하지 않는다는 인상을 풍긴다.<br>
그래서 사람들도 관심을 끊고, 창문이 더 깨져도 상관하지 않는다. 결국은 더이상 건들이지도, 건들수도 없는 코드가 되어버린다.<br>
위 코드는 중복, 한 기능만 수행, 제대로 표현, 작게 추상화를 모두 어겼다고 판단된다. (필자의 생각)<br>
여기 저기 중복된 코드와 하나의 함수에서 여러가지의 일을 하고있으며, convert라는 추상화의 단계가 너무 크다고 생각된다.(이를테면 resize, save 정도가 알맞을 것 같다.)<br>
또한, sPath, tPath는 무엇이며, position은 무엇을 의미하는지 모르기 때문에 제대로 표현하지 못했다고 할 수 있을 것이다.<br>
이제 하나하나 고쳐나가 보자<br><br>

1. 중복 제거

// 중복 제거 코드

2. 한 기능만 수행 / 작게 추상화

// 한 기능만 수행(작게 추상화)하도록 쪼개기

3. 제대로 표현

// 변수명 수정