---
layout: post
title: "JQ获取BOM对象宽高的方法"
description: ""
category: frontend
tags: []
---

    获取浏览器显示区域（可视区域）的高度 ：   
    $(window).height();   

    获取浏览器显示区域（可视区域）的宽度 ：
    $(window).width();   

---

    获取页面的文档高度   
    $(document).height();   

    获取页面的文档宽度 ：
    $(document).width(); 

---

    浏览器当前窗口文档body的高度：  
    $(document.body).height();

    浏览器当前窗口文档body的宽度： 
    $(document.body).width();

---

    获取滚动条到顶部的垂直高度 (即网页被卷上去的高度)  
    $(document).scrollTop();   

    获取滚动条到左边的垂直宽度 ：
    $(document).scrollLeft(); 

---

    获取或设置元素的宽度：
    $(obj).width();

    获取或设置元素的高度：
    $(obj).height();

---

    某个元素的上边界到body最顶部的距离：obj.offset().top;（在元素的包含元素不含滚动条的情况下）

    某个元素的左边界到body最左边的距离：obj.offset().left;（在元素的包含元素不含滚动条的情况下）

---

    返回当前元素的上边界到它的包含元素的上边界的偏移量：obj.offset().top（在元素的包含元素含滚动条的情况下）

    返回当前元素的左边界到它的包含元素的左边界的偏移量：obj.offset().left（在元素的包含元素含滚动条的情况下）

---