---
title: Infuser 轻量级的依赖注入
date: 2017-11-15 10:43:00
tags:
- library
- 依赖注入
---
在日常工作中，使用一个类的时候难免会把其他类的对象作为自己内部的变量，由此可见依赖注入的高出现率。在很早的时候就有[Dagger](https://github.com/google/dagger)开源项目，用过的人都知道它功能的全面性，但也正是由于功能全，在依赖的情况较多时，编译会变的非常慢。在只想创建具有空参或者基本类型参数的类的对象时，有没有更好的轮子呢，答案是肯定的，这时候就可以使用Infuser。
### 功能
- 支持空参构造器
- 支持 `String`以及基本类型数组作为参数，包括`int` `long` `float` `double` `char`
<!-- more -->

### 实现
* 通过`Infuser.bind(this)`绑定当前对象，该方法具有`@NonNull`和`@UiThread`注解，因此返回值Binder对象不能为空，并且方法是在主线程中执行的。
```java
@NonNull
@UiThread
public static Binder bind(Object object) {
    return createBinder(object);
}
```
* 构建返回值`Binder`，即要查找的自动生成的类的对象，统一实现了Binder接口。
```java
private static Binder createBinder(Object object) {
    Class<?> clazz = object.getClass();
    Constructor<? extends Binder> constructor = findBinderConstructorForClass(clazz);
    if (constructor == null)
        return Binder.BINDER_EMPTY;
    try {
        return constructor.newInstance(object);
    } catch (Exception e) {
        e.printStackTrace();
        throw new RuntimeException("Unable to create new instance " + constructor, e);
    }
}
```
* 创建依赖的对象，获取该对象的`Class`对象，通过`Class`对象获取`Constructor`对象，最后创建实例。
```java
public class MainActivity_ConstructorBinder implements Binder {

    @UiThread
    public MainActivity_ConstructorBinder(MainActivity object) {
        try {
        	Class<?> singerClass = Class.forName("com.lxt.simple.Singer");
        	Constructor<?> constructor = singerClass.getConstructor(double.class,double.class,double.class);
        	object.singer = (com.lxt.simple.Singer)(constructor.newInstance(20000.0,222222.0,250000.0));
        } catch (Exception e){
           	e.printStackTrace();
        }
    }
}
```
### 链接
在[Github](https://github.com/lxt352/Infuser)上使用。