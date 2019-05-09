---
title: JS 数组组合算法
date: 2017-06-07 11:22:54
tags:
---

<p>先来看看要实现什么样的效果</p>

<pre><code class=\"javascript\">var data = [[1,2,3],[4,5,6]];

var result =  data.group();

// 结果：[1,4] [1,5], [1,6],[2,4],[2,5],[2,6]....以此类推;

</code></pre>

<!-- more -->
<p>【源码】扩展Array方法：</p>

<pre><code class=\"javascript\">Array.prototype.group = function() {
    var $this = this;
    var result = new Array();
    var findNext = function next(currentIndex, arr) {
        var item = $this[currentIndex];
        var tempArr = arr;
        var index = currentIndex;
        for (var k = 0; k &amp;lt; item.length; k++) {
            if (!arr) tempArr = new Array();
            tempArr[tempArr.length] = item[k];
            if (tempArr.length == $this.length) {
                result[result.length] = tempArr;
                tempArr = arr.slice();
                tempArr.length = tempArr.length - 1;
                continue;
            }
            index++;
            if (index &amp;lt; $this.length &amp;amp;&amp;amp; next(index, tempArr)) {
                index--; // 恢复递归之前的索引
                tempArr = arr ? arr.slice() : new Array();
                tempArr.length = tempArr.length &amp;gt; 0 ? tempArr.length - ($this.length - index) : 0;
            }
        }
        return true;
    };

    findNext(0);

    return result;
}
</code></pre>
