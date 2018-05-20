---
layout:     post
title:      "PowerShell配置环境变量" 
subtitle:   ""
date:       2018-05-20 16:20:00
author:     "SiyuanWang"
#header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - PwerShell
---

# PowerShell配置JDK环境变量

        $curpath=Get-Location
        $curpath=$curpath.ToString()
        echo "JAVA_HOME:"
        $curpath

        setx /M JAVA_HOME $curpath
        $newpath=$env:Path+';'+"%JAVA_HOME%\bin"
        echo "Path:"
        $newpath
        setx /M Path $newpath

        $classpath=$env:CLASSPATH+";"+"%JAVA_HOME%\lib"
        echo "CLASSPATH:"
        $classpath
        setx /M CLASSPATH $classpath

# PowerShell配置MAVEN环境变量

        $curpath=Get-Location
        $curpath=$curpath.ToString()
        echo "M2_HOME:"
        $curpath

        setx /M M2_HOME $curpath

        $newpath=$env:Path+';'+"%M2_HOME%\bin"
        echo "Path:"
        $newpath
        setx /M Path $newpath
