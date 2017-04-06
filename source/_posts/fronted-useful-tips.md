---
title: 前端知识点汇总
date: 2016-03-04 10:59:01
tags: [jquery, html]
---

### 1.Bootstrap Modal Draggable

```
$(document).on("show.bs.modal", ".modal", function(){
    $(this).draggable({
        // handle: ".modal-header" 
        // 只能点击头部拖动
    });
    $(this).css("overflow", "hidden"); 
    // 防止出现滚动条，出现的话，你会把滚动条一起拖着走的
});
```

show.bs.modal: 在调用 show 方法后触发。	        
```
$("#identifier").on('show.bs.modal', function(){//do sth. });
```

show.bs.modal	当模态框对用户可见时触发（将等待 CSS 过渡效果完成）。	
```
$("#identifier").on('shown.bs.modal', function(){//do sth. });
```

hide.bs.modal	当调用 hide 方法时触发。	
```
$("#identifier").on('hide.bs.modal', function(){//do sth. });
```

hidden.bs.modal	当模态框完全对用户隐藏时触发。	
```
$("#identifier").on('hidden.bs.modal', function(){//do sth. })
```

- 需加载`jquery-ui.min.js`
```
$(document).ready(function(){
    //为模态对话框添加拖拽
    $("#modalDialog").draggable();
    
    //禁止模态对话框的半透明背景滚动
    $("#myModal").css("overflow", "hidden");

})
```

### 2.Location Object

| Location | 对象属性                               |
|:---------|:---------------------------------------|
| 属性	   | 描述
| hash	   | 设置或返回从井号 (#) 开始的 URL（锚）。
| host	   | 设置或返回主机名和当前 URL 的端口号。
| hostname | 设置或返回当前 URL 的主机名。
| href	   | 设置或返回完整的 URL。
| pathname | 设置或返回当前 URL 的路径部分。
| port	   | 设置或返回当前 URL 的端口号。
| protocol  | 设置或返回当前 URL 的协议。
| search   | 设置或返回从问号 (?) 开始的 URL（查询部分）

```
var searchStr = location.search;
location.href = location.pathname + qry;
```

    onclick="javascript:location.href='/trouble/list'"

### 3.Select Object 

- (1) 删除全部option： `$("#select").empty();`
- (2) 添加一项option： 
```
$("#select_id").append("<option value='value'>text</option>");
```
- (3) 创建option对象：`opt = new Option(name, value);`

```
// 产品线与集群二级联动下拉菜单
$("#proline_select").change(function() {
    var cluster_list = $("#proline_select").find("option:selected").attr('tags').split(','); 
    var cluster_select = $("#cluster_select")
    cluster_select.empty()
    for(var i=0; i<cluster_list.length; i++) {
        opt = new Option(cluster_list[i], cluster_list[i]);    
        cluster_select.append(opt);
    }
    if ($("#proline_select").val() == 0 || cluster_list == "") {
        cluster_select.empty()
        var opt = new Option('集群名称', 0);
        cluster_select.append(opt);
    }
});
```

- (4) option选中与回显：
```
<select class="selectpicker form-control" name="mikey">
	<option value="">请选择监控指标</option>          
	{% for key in mikey_list %}                       
		<option value="{{ key['mikey'] }}"            
		{% if mikey == key %} selected = "selected" {% endif %}>
		{{ key['mikey'] }}</option>
	{% endfor %}
</select>
```

### 4.Checkbox Object
```
<th class="text-center">
<input type="checkbox" id="all_alive" onclick="checkAll('all_alive', 'checked')">
</th>
```

```
// Checkbox Function
//此函数用于checkbox的全选和反选
var checked=false;
function check_all(form) {
    var checkboxes = document.getElementById(form);
    if (checked == false) {
        checked = true
    } else {
        checked = false
    }
    for (var i = 0; i < checkboxes.elements.length; i++) {
        if (checkboxes.elements[i].type == "checkbox") {
            checkboxes.elements[i].checked = checked;
        }
    }
}
```

```
function checkAll(id, name){
    var checklist = document.getElementsByName(name);
    if(document.getElementById(id).checked) {
        for(var i=0;i<checklist.length;i++) {
            checklist[i].checked = 1;
            //checklist[i].classList.add("info");
            checklist[i].parentNode.parentNode.classList.add("info");
        }
    } else {
        for(var j=0;j<checklist.length;j++) {
            checklist[j].checked = 0;
            checklist[j].parentNode.parentNode.classList.remove("info");
        }
    }
}
```


### 5.JQuery Cookie

(1) jquery-cookie导入
```
<script type="text/javascript" src="js/jquery.cookie.js"></script>
```

(2) 新添加一个会话cookie
```
$.cookie('the_cookie', 'the_value');
```

> 当没有指明 cookie有效时间时，所创建的cookie有效期默认到用户关闭浏览器为止，所以被称为"会话cookie(session cookie)".

(3) 创建一个cookie并设置有效时间为7天
```
$.cookie('the_cookie', 'the_value', { expires: 7 });
```
> 当指明了cookie有效时间时，所创建的cookie被称为"持久cookie(persistent cookie)".

(4) 创建一个cookie并设置有效路径
```
$.cookie('the_cookie', 'the_value', { expires: 7, path: '/' });
```

- 在默认情况下，只有设置cookie的网页才能读取该cookie；

- 如果想让一个页面读取另一个页面设置的cookie，必须设置cookie的路径；

- cookie的路径用于设置能够读取cookie的顶级目录；

- 将这个路径设置为网站的根目录，可以让所有网页都能互相读取cookie；

> 一般不要这样设置，防止出现冲突；

(5) 读取cookie
```
$.cookie('the_cookie');     // cookie存在 => 'the_value' 
$.cookie('not_existing');   // cookie不存在 => null
```

(6) 删除cookie

> 通过传递null作为cookie的值即可：

```
$.cookie('the_cookie', null);
```
> 如果想删除一个带有效路径的cookie，如下：

```
$.cookie('cookieName', null, {path:'/'});
```

> 注： jquery-cookie.js的版本要用最新的`v1.4.1`, v1.3.x会报 `URIError: URI malformed`；

e.g (1)：
```
<script type="text/javascript" >
    var tabcookievalue = $.cookie("mytab");
    if (tabcookievalue != "") {
        nTabs(
            $("#myTab li").eq(tabcookievalue)[0], 
            tabcookievalue
        );     
    }

    $("#myTab li").click(function () {
        $.cookie("mytab", $(this).index());
    });
</script>
```

e.g (2)：
```
$(document).ready(function() {
    // Store cookie when click tabs
    $("#equip_ul li").click(function() {
        $.cookie("equip_tab", $(this).index(), { expires: 7, raw: true });
    });

    var tab_index = $.cookie("equip_tab");
    if (tab_index != "") {
        $("#equip_ul li").removeClass("active");
        $("#equip_pane .tab-pane").removeClass("active");
        var tab_li = $("#equip_ul li").eq(tab_index);
        var panel = $(tab_li).children('a').attr('href');
        $(tab_li).addClass("active");
        $(panel).addClass("active");
    }
}
```

### 6.Tables

(1) word-wrap
```
<table style="table-layout: fixed;">
  <tr>
    <td style="word-wrap: break-word;"></td>
  </tr>
</table>
```

(2) text-align-last
```
<td style="text-align-last: center;"></td>
<td style="text-align-last: center;"></td>
<td style="text-align-last: center;"></td>
```



### [REFERENCE]

- [cookie读写插件jquery.cookie.js](http://www.jq22.com/jquery-info254)
- http://www.layui.com/
- [Beautiful Flat Icons](https://www.elegantthemes.com/blog/freebie-of-the-week/beautiful-flat-icons-for-free)
