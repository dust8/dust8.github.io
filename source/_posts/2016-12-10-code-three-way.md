---
title: 编码的三种方式
date: 2016-12-10 05:15:31
tags:
  - 编码
  - 思考
---

## 起因  
自学编程不少年了,看过不少东西,觉得什么都知道一点,让自己做出个项目却完全没有头绪.
所以就决定刷完 `freeCodeCamp`,打好基础.在 `Basic Algorithm Scripting`碰到一个问题
`Algorithm:Title Case a Sentence` 完全没有头绪.如果是`python`用内置的方法就可以实现了,
`javascript` 里面没有.点击[Read-Search-Ask](https://github.com/FreeCodeCamp/freecodecamp/wiki/Algorithm-Title-Case-A-Sentence)就可以看到一些解决方法.    

## 用最基础的方式解决  
这种方式比较简单, 不过代码比较长.    

    String.prototype.replaceAt = function(index, character) {
        return this.substr(0, index) + character + this.substr(index+character.length);
    };


    function titleCase(str) {
        var newTitle = str.split(' ');
        var updatedTitle = [];
        for (var st in newTitle) {
            updatedTitle[st] = newTitle[st].toLowerCase().replaceAt(0, newTitle[st].charAt(0).toUpperCase());
        }
        return updatedTitle.join(' ');
    }

## 用内置的高阶方法解决   
利用的语言内置的高阶函数,可以减少很多代码,例如 `map`.    

    function titleCase(str) {
        var convertToArray = str.toLowerCase().split(" ");
        var result = convertToArray.map(function(val){
          return val.replace(val.charAt(0), val.charAt(0).toUpperCase());
        });
        return result.join(" ");
    }

    titleCase("I'm a little tea pot");


## 利用各种黑魔法解决
这种方法比较难想到,也不太通用.     

    function titleCase(str) {
        return str.toLowerCase().replace(/( |^)[a-z]/g, (L) => L.toUpperCase());
    }



## 总结
其实这三种方法适用各种语言,代表着你掌握语言的深入程度.写代码要优先使用第二种方法.
因为它可以减少代码量,又容易阅读.因为好的代码要易读.不然过段时间连你自己都不知道为什么这么写了.
