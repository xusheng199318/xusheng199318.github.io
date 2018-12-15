---
title: 设计原则solid
date: 2018-12-15 22:07:38
tags: 设计模式
---

### 单一职责原则（SRP)

就一个类而言，应该仅有一个引起它变化的原因，一个类应该只做一类事

### 开闭原则（OCP）

软件实体（类、模块、函数等）应该是可以扩展的，但是不可修改

违反OCP原则设计

```java
/**
 * 违反开闭原则设计
 */
public class UnOcp {
    @Test
    public void testUnOcp() {
        Circle circle = new Circle();
        Square square = new Square();
        List list = new ArrayList();
        list.add(circle);
        list.add(square);

        for (Object o : list) {
            //如果多加一种三角形就需要多加一个else if 进行判断
            if (o instanceof Circle) {
                ((Circle) o).drawCircle();
            } else if (o instanceof Square) {
                ((Square) o).drawSquare();
            }
        }
    }

    class Circle {
        ShapeType itsType = ShapeType.circle;
        void drawCircle() {
            System.out.println("draw Circle");
        }
    }

    class Square {
        ShapeType itsType = ShapeType.square;
        void drawSquare() {
            System.out.println("draw Square");
        }
    }

    enum ShapeType{
        circle, square;
    }

}

```

符合OCP

```java
/**
 * 满足开闭原则
 * 如果新增一种形状就新建一个类继承Shape，
 * 无需改动testOcp这个方法
 */
public class Ocp {

    @Test
    public void testOcp() {
        Circle circle = new Circle();
        Square square = new Square();
        List<Shape> shapes = new ArrayList<>();
        shapes.add(circle);
        shapes.add(square);
        for (Shape shape : shapes) {
            shape.draw();
        }
    }

    abstract class Shape {
        public abstract void draw();
    }

    class Circle extends Shape{

        @Override
        public void draw() {
            System.out.println("draw Circle");
        }
    }

    class Square extends Shape {

        @Override
        public void draw() {
            System.out.println("draw Square");
        }
    }

}

```



### Liskov替换原则（LSP）

子类型必须能够替换掉他们的基本类型

违反LSP

```java
public class UnLsp {

    @Test
    public void testUnLsp() {
        testArea(new Rectangle());
        testArea(new Square());
    }

    private void testArea(Rectangle rectangle) {
        rectangle.setHeight(5);
        rectangle.setWidth(4);
        Assert.assertEquals(20, rectangle.getArea());
    }

    class Rectangle {
        private int width;
        private int height;

        public void setWidth(int width) {
            this.width = width;
        }


        public void setHeight(int height) {
            this.height = height;
        }

        public int getArea() {
            return width * height;
        }
    }

    class Square extends Rectangle{

        public void setWidth(int width) {
            super.setWidth(width);
            super.setHeight(width);
        }

        public void setHeight(int height) {
            super.setWidth(height);
            super.setHeight(height);
        }

    }
}

```



### 依赖倒置原则（DIP）

高层模块不应该直接底层模块，二者都应该依赖抽象

抽象不应该依赖于细节，细节应该依赖于抽象

违反DIP

```java
public class UnDip {
    private static final int THERMONTER = 0x86;
    private static final int HEATER = 0x87;
    private static final int ENGAGE = 1;
    private static final int DISENGATE = 0;

    public void regulate(double minTemp, double maxTemp) throws Exception {
        while(true) {
            while(in(THERMONTER) > minTemp) {
                wait(1);
            }
            out(HEATER, ENGAGE);

            while(in(THERMONTER) < maxTemp) {
                wait(1);
            }
            out(HEATER, DISENGATE);
        }
    }

    private int in(int thermonter) {
        return 0;
    }

    private void out(int heater, int engage) {
    }


}
```

符合DIP

```java
/**
 * 依赖倒置原则
 * 高层模块不应该依赖于实现，
 * 应该依赖于抽象
 */
public class Dip {

    public void regulate(Thermometer  thermometer, Heater h, double minTemp, double maxTemp) throws Exception {
        while(true) {
            while(thermometer.read() > minTemp) {
                wait(1);
            }
            h.engage();

            while(thermometer.read() < maxTemp) {
                wait(1);
            }
            h.disEngage();
        }
    }
}

class Heater {
    public void engage() {

    }
    public void disEngage() {

    }
}

class Thermometer {

    public int read() {
        return 0;
    }
}

```

