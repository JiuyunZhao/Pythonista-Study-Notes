# **Jinja2模板**

Python的Jinja2模板，其实就是在HTML文档中使用控制语句和表达语句替换HTML文档中的变量来控制HTML的显示格式，Python的Jinja2模板可以更加灵活和方便的控制HTML的显示，而且大大地减少了编程人员的工作量。

本文是作者的学习笔记，并不全面，感兴趣的朋友可以参考[http://docs.jinkan.org/docs/jinja2/templates.html](http://docs.jinkan.org/docs/jinja2/templates.html)

1. **注释{\# .. \#}：**HTML中的注释可以使用`<!-- ... -->`注释，但在jinja2的web模板中也可以使用{\# .. \#}注释掉一行或多行HTML中的内容。
2. **访问字典值或对象属性值：**有两种方式，一是“object\['name'\]”采用字典的访问方式，二是“object.name”采用类属性的访问方式。
3. **表达语句**`{{ ... }}`**：**一条表达语句就相当于一条Python语句，不需要结束语句，`{{`和`}}`之间可以放入任何Python表达式，Jinja2模板语法可以自动运行`{{`和`}}`中的语句并把运行结果显示在HTML模板中。
4. **控制语句**`{% ... %}`**：**控制语句需要使用一个结束标志表明该语句的结束，通常用来作循环控制、条件控制、模块控制等，可以更加方便的控制HTML内容的显示  
   ：

   * **if语句：**使用`{% if xxx %}...{% elif xxx %}...{% else %}...{% endif %}`语句来进行条件控制。如图：  
     ![](/assets/jinja2-if.png)

   * **for语句：**使用如`{% for x in xx %} ... {{% else %}}...{% endfor %}`语句来进行循环控制：

     * **else：**当for遍历的序列中没有值时就会执行else中的内容。
     * **range：**用在for循环中，用法和Python一样，返回一定范围的序列，具体可查看Python中range的用法。
     * **if判断：**可以在for语句后使用if判断来限制循环，例如：`{% for i in range(10) if i < 5 %}...{% endfor %}`，此循环只会循环5次，即i为0，1，2，3，4时。
     * **loop：**loop可以看作是jinja2内置的对象，用在for循环中。

       * **loop.index：**表示当前迭代的索引（从1开始计数）。
       * **loop.index0：**表示当前迭代的索引（从0开始计数）。
       * **loop.first：**当前迭代如果是第一次，则返回True，否则返回False。
       * **loop.last：**如果当前迭代是最后一次迭代，则返回True，否则返回False。
       * **loop.length：**返回整个序列的长度。

     * jinja2的for循环是没有**continue**和**break**语法的。

     * 示例代码及运行结果如图：
       ![](/assets/jinja2-1.png)
       ![](/assets/jinja2-2.png)
       ![](/assets/jinja2-3.png)

5. **过滤器“\|”：**形如`{{ var | xxx }}`使用过滤器将变量“var”处理后再渲染展示出来，管道符号“\|”前面为变量，后面是一个过滤器（过滤器就相当于一个函数），最外层可以是`{{ ... }}`或者`{% ... %}`。常用的过滤器有：

   * **abs：**返回变量的绝对值。
   * **default\(value, boolean=False\)：**给变量设置默认值，当变量不存在时，则使用设置的默认值“value”；如果设置了boolean为True，且变量存在时，当它的Python计算结果为False时，也会使用设置的默认值，比如变量的值为None，\[\]，''等。
   * **or：**or不是过滤器，但是效果相当于default过滤器，当有多个变量时，按照Python中or的逻辑，当有一个变量存在且Python计算结果为True时，取其值，不存在（也算作False）或者Python计算机结果为False时，则继续检查下一个变量。例如：`{{ var | default('什么也没有', boolean=True) }}`的效果与`{{ var or '什么也没有' }}`的效果是一样的。
   * **length：**给出前面变量的“长度”，比如字典、列表等的元素个数。
   * **escape：**将值为字符串的变量中的`<`、`>`等HTML中的特殊符号转义成普通字符串。如果没有转义，字符串中含有HTML标签等特殊符号的话，浏览器会将它解释为HTML的内容，就不再是原先的字符串了。由于jinja2是开启了自动转义的，所以变量的字符串是什么，HTML渲染展示的就是什么，这时候就用不着escape过滤器。可以使用`{% autoescape off %}...{% endautoescape %}`来关闭处于其中的HTML内容的自动转义，这时候想要其中的某个字符串进行转义的话，就可以使用这个过滤器了。
   * **safe：**关闭变量字符串的自动转义。safe是针对变量的，而`{% autoescape on/off %}...{% endautoescape %}`是打开/关闭处于其中的HTML内容的自动转义。
   * **first：**返回序列的第一个元素。
   * **last：**返回序列的最后一个元素。
   * **format：**格式化字符串，与Python中的%用法类似（只能是%，不能是{}占位符），例如：`{{ "这是%s27"|format("Python") }}`，渲染结果为“这是Python27”。
   * **join\(d=u''\)：**返回使用d的值来将序列中的元素连接在一起的字符串，类似Python中的join方法。
   * **int：**将值转换为int类型。
   * **float：**将值转换为float类型。
   * **lower：**将字符串转换为小写。
   * **upper：**将字符串转换为大写。
   * **replace\(old, new\)：**将字符串中子串old替换为新的子串new。
   * **string：**将变量转换为字符串。
   * **truncate\(length=255\)：**只截取字符串的前length个字符。
   * **striptags：**删除字符串中的所有HTML标签，如果字符串中有多个连续的空格，则替换成一个空格。
   * **wordcounts：**统计一个字符串中的单词个数。
   * **更多过滤器：**[http://jinja.pocoo.org/docs/dev/templates/\#builtin-filters。](http://jinja.pocoo.org/docs/dev/templates/#builtin-filters。)

6. **自定义过滤器：**过滤器其实就是一个普通的函数，该函数至少有一个参数，HTML中使用过滤器时会将变量作为该函数的第一个参数传入进去，然后将该函数的返回值渲染到HTML中，定义好过滤器的函数后，需要使用装饰器@app.template\_filter\('filter\_name'\)将函数注册到jinja2中才能使用，HTML中使用该自定义的过滤器时，使用自定义的filter\_name，jinj2就会自动调用对应的函数了。如下示例浏览器显示为“Python36”：  
   ![](/assets/jinja2-4.png)

7. **宏macro：**模板中宏就像Python中的函数一样，当多个地方需要使用相似代码片段时，通过定义宏，然后根据不同需求给宏传递参数即可达到目的，不用重新写一遍代码。

   * **宏定义：**使用形如`{% macro macro_name(name, value="", type="") -%}...{%- endmacro %}`即可定义一个名叫`macro_name`的宏，name和value等就是宏的参数。注意-%}和{%-这两个符号中的减号是为了取出宏的首尾的换行符，除非整个宏是定义在一行代码中的，否则通过浏览器查看源代码时就会发现宏的位置的代码是被换行了的。
   * **作用和使用：**它的作用就像Python中的函数一样，可以将经常用到的代码片段定义为一个宏，`macro_name`就相当于函数名称，name和value等就相当于函数的参数，这些参数在定义时可以像变量一样使用`{{ value }}`去使用它们。使用宏的使用就像使用Python函数一样即可，例如`{{ macro("username", value="username", type="text") }}`。
   * **导入宏：**我们一般会将宏专门放在一个HTML模板中，然后导入到其他的模板中使用，导入有两种方式：`{% from "macros.html" import macro_name [as new_name] %}`，或者`{% import "macros.html" as macros %}`，其中"mcros.html"为存放宏的模板路径。前者可以使用as来起一个别名，但不是必须的，后者就必须给模板起一个别名，然后以macros.macro\_name的方式使用。
   * **宏模板路径：**导入宏模板的路径不能是相对路径，必须是templates下的绝对路径。
   * **在宏模板中使用参数变量：**想要在定义宏的模板中使用导入该宏时它所在模板中的参数变量，可以在导入语句后加入with context，这样宏的模板就可以使用当前模板中的所有参数变量了，例如`{% import "macros.html" as macros with context %}`。

8. **模板继承block和extends：**

   * **语法：**在父模板中使用`{% block block_name %}{% endblock %}`进行模块的占位，block\_name可以自己定义，其他的都是固定的语法格式；在子模板中使用`{% extends  "xxx.html" %}`表示此HTML模板继承自“xxx.html”模板（父模板），然后在子模板中使用`{% block block_name %} ... {% endblock %}`重新定父模板中占位的block模块（注意子模板此时不用再写父模板中已有的内容，包括HTML标签），子模板中定义模块内容就会显示在母板中占位的位置，不同的子模板中可以定义不同的模块内容来满足自身的需要。
   * **使用：**`{% extends  "xxx.html" %}`需要放在所有代码的前面，一般就放在第一行。还有另外一点就是子模板如果使用了继承，那么子模板中所有的代码就需要放在block中实现，也就是说想要实现一些没有在父模板block中的代码，那么就需要在父模板中再添加一个block进行占位了，不然在子模板中写在block之外的代码不会被展示出来。
   * **block：**父模板的block中也可以写代码，当时如果子模板中重新定义了此block的内容，那么父模板此block就会被覆盖掉，当然如果没有重新定义，自然就还是展示父模板中的block内容。
   * **super：**如果在子模板中的block中想要使用父模板中此block的内容，那么使用`{{ super() }}`就可以让父模板中block的内容“copy”这个位置了。
   * **self：**如果想要在block中使用另一个block中的内容，可以使用`{{ self.block_name() }}`就可以让另一个block中的内容“copy”到这个位置。当然，如果子模板中重新定义了block的内容，那么self就会使用子模板中的block了。

9. **模板导入include：**在一个HTML模板中使用`{% include "xxx.html" %}`，就会将xxx.html的内容导入（嵌入）当前HTML模板中，效果相当于将xxx.html的内容copy到执行include语句时的位置，并且可以在xxx.html中直接使用当前模板中的参数变量。一般会将一些公共内容写在一个模板中，然后导入到需要使用的模板中，以达到代码重用的效果，使模板编码更加简洁方便，比如导航条、底部信息等大多数页面都会用到的内容；

10. **set语句：**类似Python的定义变量，在模板中使用形如`{% set para_name=para_value %`}就可以在这个语句之后使用这个变量了（在本模板中）。
11. **with语句：**使用形如`{% with [para_name=para_value] %}...{% endwith %}`，如果这里定义了变量，那么这个变量就只能在with的作用域中使用了，也可以在with的作用域中使用set语句，那么set定义的变量也是只能在with作用域中生效。
12. **静态文件加载：**静态文件则使用如`<link rel="stylesheet" href="{{ url_for('static', filename='css/index.css') }}" >`，其中“static”是静态文件存放的总文件夹，“filename”的值则是静态文件的具体路径。如图：
    ![](/assets/jinja2-5.png)



