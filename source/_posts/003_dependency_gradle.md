---
title: Gradle的配置文件
date: 2017-11-15 10:47:01
tags:
- gradle
- groovy
---
个人习惯不同，有时候也是衡量一个人能力好坏的关键，所以每次在接触到关于某个知识点的不同认知后，我都会拿来和自己的想法进行对比，把好的拿进来、把不好的去掉，以此进步。知识总是不断更新，在以前的工作中，习惯把build.gradle的某些项目地址版本写的另一份gradle文件里面。
```groovy
ext {
    sourceCompatibility = JavaVersion.VERSION_1_7
    targetCompatibility = JavaVersion.VERSION_1_7

    versions = [
            infuserVersion: '1.0.0',
            supportVersion: '26.+'
    ]

    groupId = "com.lure.infuser"
    siteUrl = "https://github.com/lxt352/Infuser"
    gitUrl = "https://github.com/lxt352/Infuser.git"

    supportV7 = "com.android.support:appcompat-v7:${versions.supportVersion}"
    infuserAnnotation = "com.lure.infuser:annotation:${versions.infuserVersion}"
    supportAnnotation = "com.android.support:support-annotations:${versions.supportVersion}"
}
```
<!-- more -->
如果配置文件只添加在module下，可以直接进行引用。
```groovy
apply from: "dependency.gradle"

def lib = rootProject.ext
dependencies {
    implementation lib.supportV7
    implementation lib.supportAnnotation
    api lib.infuserAnnotation
}
```
### 不足
在使用的开源项目较多时，类别划分的重要性就体现出来了，分类的好坏给人的感觉也是不一样的。

![http://ozc61ychw.bkt.clouddn.com/gradle_category.jpeg]

### 分类
 依据项目的关联性或者功能进行划分，例如依赖Dagger时需要用到另外两个相关的项目。
```groovy
def dagger() {
    dependencies.add("implementation", "com.google.dagger:dagger:2.0.2")
    dependencies.add("annotationProcessor", "com.google.dagger:dagger-compiler:2.0.2")
    dependencies.add("provided", "javax.annotation:jsr250-api:1.0")
}

dependencies {
    dagger()
}
```
简化
```groovy
def dagger() {
    dependencies.implementation com.google.dagger:dagger:2.0.2"
    dependencies.implementation com.google.dagger:dagger-compiler:2.0.2"
    dependencies.provided javax.annotation:jsr250-api:1.0"
}

dependencies {
    dagger()
}
```
 闭包具有一个代理对象(delegate object)，可以对Clousure的代理对象进行赋值操作。
```groovy
def injectDependency(Closure closure) {
     closure.delegate = dependencies
     return closure
 }

 def retrofit = injectDependency {
     implementation 'com.squareup.retrofit2:retrofit:2.0.2'
     implementation 'com.squareup.retrofit2:converter-gson:2.0.2'
     implementation 'com.squareup.okhttp3:logging-interceptor:3.2.0'
 }

 dependencies {
     dagger()
     retrofit()
 }
```
 如果想要对版本进行统一管理的话，还可以抽取版本信息。
```groovy
ext {
    versions = [
            retrofitVersion : '2.0.2',
            logInterceptorVersion: '3.2.0'
    ]
}

def injectDependency(Closure closure) {
     closure.delegate = dependencies
     return closure
 }

 def retrofit = injectDependency {
     implementation 'com.squareup.retrofit2:retrofit:${versions.retrofitVersion}'
     implementation 'com.squareup.retrofit2:converter-gson:${versions.retrofitVersion}'
     implementation 'com.squareup.okhttp3:logging-interceptor:${versions.logInterceptorVersion}'
 }

 dependencies {
     dagger()
     retrofit()
 }
```
最后引用依赖
```groovy
apply from: "dependency.gradle"
```
