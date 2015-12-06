title: javascript组件化心得
date: 2015-12-06
categories:
    - 心得体会
tags:
	- javascript
	- 组件
---

### 模板原理

> 首先了解一下模板的工作原理，可以简单的分成两个步骤：模板解析和数据渲染。根据对应的语法解析出对应的占位符，然后把占位符替换成对应的数据。例如我们在JSP中写`<a><%= name %></a>`的时候，其实就是在应用模板，在后台这句话会被转换成`out.print(“<a>”+name+”</a>”)`。模板的数据渲染就是把模板的占位符（如name）替换成传入的值。

### 简单分析

> 在对进行组件开发的时候，经常要根据需求的变动分析出来相同点和异同点，然后把相同点进行封装，把异同点进行对外开放，也就是‘求同存异’。那如何来对相同点进行封装？这里介绍一个常用的理念：模板在js中的应用。

### 通过例子来理解如何利用模板在组件中的应用。

##### 需求是在一个列表中，有一个列表要求按照年月日来显示，如2015-01-26。看到这个需求，很多人会认为很简单，只需要几句代码就可以搞定。

<!--more-->

```
function time(date){
        var year = date.getFullYear();
        var month = date.getMonth();
        var day = date.getDay();
        return year + '-' + month + '-' + day
    }
    console.log(time(new Date()))
```

##### 从功能的角度来看，其实所需要的需求已经可以达到。但是从组件的角度来看，这样还是远远不够的。如果需求突然变换，需要显示完整的日期，包括分、秒。如：2015-01-26 15：22 此时这样的封装就无法满足这个需求。或许有人想，在上面稍微做一下扩展，传进来一个参数，来判断日期格式是否要显示完整。修改够的代码如下：

```
<script>
    function time(date,isFlag){
        var year = date.getFullYear();
        var month = date.getMonth();
        var day = date.getDay();
        var minute = date.getMinutes();
        var hour = date.getHours();
        if(isFlag){
            return year + '-' + month + '-' + day + ' ' + hour + ':' + minute
        }else{
            return year + '-' + month + '-' + day
        }
    }
    console.log(time(new Date(),true))
</script>
```
#####  这样的封装，显然不够好，因为大家都知道需求是随时变化的，或许过一段时间需求要把格式改成2015年01月26日15：22，这时上面的封装方法是无法满足这个需求了。或许有人想直接增加一个显示类型参数来区分，然后来判断对应的日期。按照这个思路，代码如下：

```
<script>
    function time(str,type){
        if(!str){
            return '';
        }
        var newStr = str.replace(/[.-年份]/g,'/').replace(/[时分]/g,':').replace(/[日秒]/g,'');
        var date = new Date(newStr);
        if(isNaN(date)){
            return str;
        }
        var year = date.getFullYear();
        var month = date.getMonth() + 1;
        var day = date.getDay();
        var minute = date.getMinutes();
        var hour = date.getHours();
        var second = date.getSeconds();
        switch(type){
            case 'short'://短格式 如 2015-01-26
                return year + '-' + month + '-' + day;
            break;
            case 'long'://长格式 如 2015-01-26 15:22:06
                return year + '-' + month + '-' + day + ' ' + hour + ':' + minute + ':' + second;
            break;
            case 'chinese'://中文格式 如 2015年01月26日 15:22:06
                return year + '年' + month + '月' + day + '日' + hour + ':' + minute + ':' + second;
            break;
            default :
                return str;
            break
        }
    }
    console.log(time(new Date().toString(),'chinese'))
</script>
```

##### 这个方法确实有效，但是如果要是在换另外一种格式的话 难道要在加一个case吗？
> 如 2015/01/26。那这样的话又要修改代码。这样是不对的, 我们就要换另一个思路去寻找更适合的办法。
> 在显示日期的时候，相同的元素有哪些呢？有年、月、日、时、分、秒、季度、周；这些都是相同点；哪些是异同点呢？日期格式是由上述的共同元素各种组合。包括，但不局限下面的这些：
> “yyyy/MM/dd”
> “yyyy-MM-dd”
> “yy/MM/dd”
> “yy-MM-dd”
> “yyyy/MM/dd hh:mm:ss”
> “yyyy-MM-dd hh:mm:ss”
> “yy/MM/dd hh:mm:ss”
> “yy-MM-dd hh:mm:ss”
> “hh:mm:ss”
> “hh:mm”
> …………

##### 按照上面提出的模板理念，先要定制解析的语法与规则；

> 用元素符号y、M、d、h、m、s、q、w(区分大小写)来分别代表日期中的固定元素：年、月、日、时、分、秒、季度、星期几；如果要显示N位，则可重复N次对应的元素符号，但N不能大于其位的长度，如年是最长4位，不能用YYYYY来表示，可以用YYYY或者YY来表示。
> 利用正则可以来匹配上面所说的方法来解析。并替换成对应的数据。

```
<script>
    Date.prototype.format = function(template){
        var fullYear = this.getFullYear();
        var o = {
            'M+':this.getMonth() + 1,
            'd+':this.getDate(),
            'h+':this.getHours(),
            'm+':this.getMinutes(),
            'q+':Math.floor((this.getMonth() + 3) / 3),
            's+':this.getSeconds(),
            'S':this.getMilliseconds(),
            'w':'日一二三四五六'.charAt(this.getDay())
        };
        template = template.replace(/y{4}/,fullYear).replace(/y{2}/,fullYear.toString().substring(2));
        for (var k in o){
            if(o.hasOwnProperty(k)){
                var reg = new RegExp(k);
                template = template.replace(reg,match)
            }
        }
        function match (m){
            return m.length == 1 ? o[k] : ('00' + o[k]).substr(('' + o[k]).length)
        }
        return template
    };
    console.log(new Date().format('yy年M月d日 礼拜w 第q季度 hh:m:ss'))
</script>
```

#### 利用这种模板理念后有以下好处：

> 1.使用范围更广泛，可以满足各式各样的日期格式，只要在用的时候指定下template格式就好。
> 2.思路清晰，代码简洁，避免很多判断。
> 3.共同的元素是固定的，而这些共同的元素还可以随意的去组合展示。

****