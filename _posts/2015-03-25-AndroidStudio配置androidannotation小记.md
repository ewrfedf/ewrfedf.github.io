---
layout: post
title: AndroidStudio配置androidannotation小记 
categories:


- android
- java
tags: []
status: publish
type: post
published: true
--- 

1. 第一步配置apt 

在project下找到build.gradle文件，在dependencies内添加
classpath 'com.neenbedankt.gradle.plugins:android-apt:1.4'

{% highlight xml %}
// Top-level build file where you can add configuration options common to all sub-projects/modules.

buildscript {
    repositories {
        jcenter()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:1.1.0'
        classpath 'com.neenbedankt.gradle.plugins:android-apt:1.4'
        // NOTE: Do not place your application dependencies here; they belong
        // in the individual module build.gradle files
    }
}

allprojects {
    repositories {
        jcenter()
    }
}

{% endhighlight %}

2. 第二步，替换在model下build.gradle文件中的内容为以下内容：
{% highlight xml %}
apply plugin: 'com.android.application'
apply plugin: 'android-apt'
def AAVersion = '3.2'

buildscript {
    repositories {
        mavenCentral()
    }
    dependencies {
        classpath 'com.neenbedankt.gradle.plugins:android-apt:1.4'
    }
}

apt {
    arguments {
        androidManifestFile variant.outputs[0].processResources.manifestFile
        resourcePackageName 'app.umoke.com.myapplication'
    }
}

android {
    compileSdkVersion 21
    buildToolsVersion "21.1.1"

    defaultConfig {
        applicationId "app.umoke.com.myapplication"
        minSdkVersion 14
        targetSdkVersion 21
        versionCode 1
        versionName "1.0"
    }
    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
}

dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
    compile 'com.android.support:appcompat-v7:21.0.3'
    apt "org.androidannotations:androidannotations:$AAVersion"
    compile "org.androidannotations:androidannotations-api:$AAVersion"
}

{% endhighlight %}

 依赖加载成功后，重新编译项目，出现androidannotation中格式错误时，重新编译一般都可以解决。enjoy your annotation time！