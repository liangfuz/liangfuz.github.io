---
layout: post
title: IDEA 类及方法注释（主要讲解DESCRIPTION自动生成）
category: IDEA
tags: IDEA
keywords: IDEA,DESCRIPTION，注释
---

## IDEA 自定义注释模版
IDEA 全称IntelliJ IDEA，是java语言开发的集成环境，IntelliJ在业界被公认为最好的java开发工具之一，本文主要讲述如何设置类和方法的自定义注释模版
### 类的自定义注释模版设置

  IDEA文件头设置模版之后在文件生成的同时自动增加注释，设置顺序为File->Settings->Editor->File and Code Templates
  注释代码模版：
```
#if (${PACKAGE_NAME} && ${PACKAGE_NAME} != "")package ${PACKAGE_NAME};#end
/**
 * Description:${DESCRIPTION}
 * Author: zhangliangfu
 * Create on: ${YEAR}-${MONTH}-${DAY} ${TIME}
 */
public class ${NAME} {
}
```

<img src="http://github-blog.oss-cn-shenzhen.aliyuncs.com/1.png"/>
  也可以写在File Header里面，然后include进去
<img src="http://github-blog.oss-cn-shenzhen.aliyuncs.com/3.png"/>
<img src="http://github-blog.oss-cn-shenzhen.aliyuncs.com/2.png"/>
  但是从我的实践来看，用include的方式不能在新建类的时候输入description，如图
<img src="http://github-blog.oss-cn-shenzhen.aliyuncs.com/1201-4.png"/>  

  如果按第一种方式直接写在文件类型的头文件注释的话则可以在新建的时候输入描述，如图
<img src="http://github-blog.oss-cn-shenzhen.aliyuncs.com/1201-5.png"/>  

<img src="http://github-blog.oss-cn-shenzhen.aliyuncs.com/1201-6.png"/>

### 方法的自定义注释模版设置

  IDEA的方法自定义的设置顺序为File->Settings->Editor->Live Templates
  点击右侧绿色的+添加一个group，之后在group上面增加注释模版
<img src="http://github-blog.oss-cn-shenzhen.aliyuncs.com/1201-7.png"/>
<img src="http://github-blog.oss-cn-shenzhen.aliyuncs.com/1201-8.png"/>
Abbreviation填写模板名称，点击define或者change选择需要使用的地方
<img src="http://github-blog.oss-cn-shenzhen.aliyuncs.com/1201-11.png"/>
<img src="http://github-blog.oss-cn-shenzhen.aliyuncs.com/1201-10.png"/>
Template test填写模板内容为:
```
/**
 * 
 $param$
 * @return      $return$
 * @exception   $exception$
 */
```
  可根据需要增减，之后点击Edit variables绑定模板变量的具体内容，其中param使用groovy编写，模板如下：
```
groovyScript("def result=''; def params=\"${_1}\".replaceAll('[\\\\[|\\\\]|\\\\s]', '').split(',').toList(); for(i = 0; i < params.size(); i++) {result+=' * @param ' + params[i] + ((i < params.size() - 1) ? '\\n\\b' : '')}; return result", methodParameters())
```
使用的时候只需要在方法体内输入字母即可联想该模版
<img src="http://github-blog.oss-cn-shenzhen.aliyuncs.com/1201-9.png"/>
<img src="http://github-blog.oss-cn-shenzhen.aliyuncs.com/1201-12.png"/>  
  注意需要在方法体内使用，否则获取不到方法的参数
<img src="http://github-blog.oss-cn-shenzhen.aliyuncs.com/1201-13.png"/>
OK，IDEA的类和方法注释到这里就设置好啦！



