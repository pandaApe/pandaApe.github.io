title: 说说Gson中的注解
date: 2016-03-25 20:03:08
categories: Java
---

## 源起
今天做一个Android项目发现一个需求，本地的Java Object需要用Gson转成JSON，并保存到服务器,而其中一些只用以在本地数据库中作用的Field，比如数据库主键id，更新时间LastUpdateDate,并不想变成JSON。 当然你可能会说用Java 语言关键字transient啊，可是使用transient修饰后的变量是不会被序列化和反序列化的，也就是说这些变量，会在传输中丢失，那么Activity之间传值就会出现问题。
<!--more-->
其实Gson中提供了**@Expose**注解来解决这个问题，除此之外还有三种，归纳起来共分三类，今天我们一起研究研究它们各自的作用。

## Control Exposure   
**@Expose** 注解提供了一个更好的解决文章开头问题方案，

``` java
import com.google.gson.annotations.Expose;
import java.util.Date;
public class People {

    @Expose(serialize = false, deserialize = false)
    private String objectId;

    @Expose
    private String name;

    @Expose(serialize = true, deserialize = false)
    private String sex;

    @Expose(serialize = false, deserialize = true)
    private int age;

    private Date updateDate;

    //Setter and Getter Methods removed for brevity

     @Override
    public String toString() {

        StringBuffer sb = new StringBuffer();
        sb.append("objectId=").append(getObjectId()).append("\n");
        sb.append("name=").append(getName()).append("\n");
        sb.append("sex=").append(getSex()).append("\n");
        sb.append("age=").append(getAge()).append("\n");
        sb.append("updateDate=").append(getUpdateDate()).append("\n");

        return sb.toString();
    }
}
```

**@Expose** 注解有两个element: **serialize** 、**deserialize**，分别控制属性的可序列化和可反序列化，参照上面代码的例子，我新增加一个表帮助理解。

|		Field 			   |	Serialise	  |  deserialize  |
	|:------------------------:|:----------------:|:-------------:|
	| @Expose(serialize = false, deserialize = false)|false|false|
	| @Expose|true|true
	| @Expose(serialize = true, deserialize = false)|true|false|
    | @Expose(serialize = false, deserialize = true)|false|true|
    |不写|false|false|
   
只给属性加上@Expose注解还是不够的，另外需要在得到Gson实例时做一项设置：

```java
    final GsonBuilder builder = new GsonBuilder();
    builder.excludeFieldsWithoutExposeAnnotation();
    final Gson gson = builder.create();
```

>第二行代码如果没有写，和没加@Expose注解时一样的效果。

### @Expose注解示例
```java
  		People people = new People();
        people.setObjectId("1");
        people.setAge(18);
        people.setName("XiaoMing");
        people.setSex("男");
        people.setUpdateDate(new Date());

        GsonBuilder gsonBuilder = new GsonBuilder();
        gsonBuilder.excludeFieldsWithoutExposeAnnotation();

        Gson gson = gsonBuilder.create();
        String jsonStr = new Gson().toJson(people);

        People p = gson.fromJson(jsonStr, People.class);

         String logStr = "Java 实例：\n" + people + "\n不加注解时的JSON:\n" + jsonStr + "\n\n加注解后的JSON:\n" + gson.toJson(people) + "\n\n加注解后进行反序列化:\n" + p;
        Log.d(MainActivity.class.getSimpleName() + "--> ", logStr);
        tvResults.setText(logStr);
```
控制台打印出Log:
```
Java 实例：
objectId=1
name=XiaoMing
sex=男
age=18
updateDate=Sat Mar 26 19:03:55 GMT+08:00 2016

不加注解时的JSON:
{"age":18,"name":"XiaoMing","objectId":"1","sex":"男","updateDate":"Mar 26, 2016 7:03:55 PM"}

加注解后的JSON:
{"name":"XiaoMing","sex":"男"}

加注解后进行反序列化:
objectId=null
name=XiaoMing
sex=null
age=18
updateDate=null

```
objectId, sex的@Expose注解的deserialize为false, updateDate并没有加注解，根据上文的表格，它的serialize和deserialize都为false，所以在反序列化时被忽略，最后值为null.

### @Expose注解总结
*Gson提供的@Expose注解，可以我开发人员很方便控制Model中的属性是否支持 序列化和反序列化。
*注意要使@Expose注解起作用，需要通过GsonBuilder进行创建，并且GsonBuilder实例要设置为excludeFieldsWithoutExposeAnnotation。
*属性没有用@Expose修饰，Gson无论在序列化还是反序列化时都会将其忽略。

## Version

Gson 提供 **@Since**和**@Until** 注解，进行Model类序列化和反序列化的版本控制。**@Since**和**@Until** 是两个相反的意义,他们都有一个double型参数。 Show my code:
```java
import com.google.gson.annotations.Since;
import com.google.gson.annotations.Until;
public class Student {

    @Until(0.3)
    private String stNo;

    @Since(0.4)
    private String studentNo;

    private String name;
    
    private int score;

    @Override
    public String toString() {
        StringBuffer sb = new StringBuffer();
        sb.append("stNo=").append(getStNo()).append(",\n");
        sb.append("studentNo=").append(getStudentNo()).append(",\n");
        sb.append("name=").append(getName()).append(",\n");
        sb.append("score=").append(getScore()).append(",\n");
        return sb.toString();
    }

    //Setter and Getter Methods removed for brevity

}

```
> @Until(0.3) GsonBuilder指定的版本小于0.3(不包含)，Gson才会解析这个属性，大于则忽略。
> @Since(0.4) GsonBuilder指定的版本大于0.4(包含)，Gson都会解析这个属性，小于则忽略。

### Version示例
```java

        Student student = new Student();
        student.setName("HanMeiMei");
        student.setScore(60);
        student.setStNo("001");
        student.setStudentNo("001");

        GsonBuilder gsonBuilder = new GsonBuilder();
        String versionStr;
        if (switchFlag > 0) {
            gsonBuilder.setVersion(0.2);
            versionStr = "0.2";
            ((Button)view).setText("Version 0.4");
        } else {
            versionStr = "0.4";
            gsonBuilder.setVersion(0.4);
            ((Button)view).setText("Version 0.2");
        }

        switchFlag *= -1;

        Gson gson = gsonBuilder.create();

        String jsonStr = "Student实例的各属性值：\n" + student + "\n不加注解的JSON:\n" + new Gson().toJson(student) + "\n\n" + versionStr + " 版本JSON:\n" + gson.toJson(student);

        tvResults.setText(jsonStr);

```
版本0.2 的JSON:
```json
{"name":"HanMeiMei","score":60,"stNo":"001"}
```
版本0.4 的JSON:
```json
{"name":"HanMeiMei","score":60,"studentNo":"001"}
```
version为0.2时 studentNo被忽略
version为0.4时 stNo被忽略

### Version 总结
* 需要GsonBuilder的实例指定版本号**@Until**和**@Since**才会起作用。
* @Until(0.3) 并不包含0.3

## SerializedName
**@SerializedName**注解使用比较多的场景是当后端返回JSON中的名称与我们定义的Model类不一致时的情况。
比如 下面的示例：
```java
import com.google.gson.annotations.SerializedName;
public class Car {

    @SerializedName("brandName")
    private String brand;

    private String engineModel;

    @Override
    public String toString() {
        return "name="+getName()+"\nengineModel="+getEngineModel();
    }
    //Setter and getter methods removed for brevity
}
```
对于Car这个类中的表示品牌名字属性，本地字义为 brand , 后端字义为brandName，对于这种情况，使用**@SerializedName**注解可以实现本地和后端兼容。
**@SerializedName**的一个字符串参数，为序列化为JSON后的属性名称。

### SerializedName示例
```java
   		Car car = new Car();
        car.setBrand("BMW");
        car.setEngineModel("bt7320xe");

        String jsonStr = "Car实例中各属性的值：\n" + car + "\n\n加注解后JSON:\n" + new Gson().toJson(car);
        tvResults.setText(jsonStr);
```
Log

```log
Car实例中各属性的值：
brand=BMW
engineModel=bt7320xe

加注解后JSON:
{"brandName":"BMW","engineModel":"bt7320xe"}
```
## 参考链接：
[Demo 地址](https://github.com/pandaApe/GsonAnnotationDemo)
[Gson Annotations Example](http://www.javacreed.com/gson-annotations-example/)
[Gson GitHub](https://github.com/google/gson)


