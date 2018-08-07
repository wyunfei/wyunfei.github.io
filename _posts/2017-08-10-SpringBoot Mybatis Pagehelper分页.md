---
layout: post
title: 'SpringBoot Mybatis PageHelper分页'
date: 2017-08-10
author: yunfei
tags: SpringBoot Mybatis
---

### 1、引入依赖
````xml
		<!--pagehelper 分页插件 -->
		<dependency>
			<groupId>com.github.pagehelper</groupId>
			<artifactId>pagehelper-spring-boot-starter</artifactId>
			<version>1.1.0</version>
		</dependency>
````

### 2、pagehelper PageInfo
```java
package com.github.pagehelper;

import java.io.Serializable;
import java.util.Collection;
import java.util.List;

public class PageInfo<T> implements Serializable {
    private static final long serialVersionUID = 1L;
    private int pageNum;
    private int pageSize;
    private int size;
    private int startRow;
    private int endRow;
    private long total;
    private int pages;
    private List<T> list;
    private int prePage;
    private int nextPage;
    private boolean isFirstPage;
    private boolean isLastPage;
    private boolean hasPreviousPage;
    private boolean hasNextPage;
    private int navigatePages;
    private int[] navigatepageNums;
    private int navigateFirstPage;
    private int navigateLastPage;

    public PageInfo() {
        this.isFirstPage = false;
        this.isLastPage = false;
        this.hasPreviousPage = false;
        this.hasNextPage = false;
    }
    // 省略部分代码
 }
```

### 3、新建sql返回list集合
```xml
  <select id="getLogList" parameterType="java.util.HashMap" resultMap="BaseResultMap">
    select
    <include refid="Base_Column_List"/>
    from sys_log
    <where>
      <if test="userName != null and userName != '' ">
        user_name like concat(concat('%',#{userName,jdbcType=VARCHAR}),'%')
      </if>
      <if test="startTime != null and startTime != '' ">
        and create_time &gt;=#{startTime,jdbcType=VARCHAR}
      </if>
      <if test="endTime != null and endTime != '' ">
        and create_time &lt;=#{endTime,jdbcType=VARCHAR}
      </if>
      <if test="operation != null and operation != '' ">
        and operation like concat(concat('%',#{operation,jdbcType=VARCHAR}),'%')
      </if>
    </where>
    order by create_time desc
  </select>
```


### 4、service代码
```java
@Service
public class SysLogServiceImpl implements SysLogService {
    private Logger logger = LoggerFactory.getLogger(this.getClass());
    @Autowired
    SysLogDao sysLogDao;

    @Override
    public List<SysLog> getAll(Map<String, Object> params, SysLog sysLog) {
        if (params != null) {
            PageHelper.startPage(Integer.parseInt(params.get("page").toString()),
                    Integer.parseInt(params.get("rows").toString()));
        }
        return sysLogDao.select(sysLog);
    }

    @Override
    public List<SysLog> getLogList(Map<String, Object> params, Map<String, Object> map) {
        if (params != null) {
            PageHelper.startPage(Integer.parseInt(params.get("page").toString()),
                    Integer.parseInt(params.get("rows").toString()));
        }
        return sysLogDao.getLogList(map);
    }

}
```

### 5、controller代码
```java
@RestController
@RequestMapping("/log")
public class SysLogController {
	Logger logger = LoggerFactory.getLogger(this.getClass());

	@Autowired
	private SysLogService sysLogService;


	/**
	 * @Description 获取日志列表
	 *
	 * @author wangzengfeng
	 * @method getLogList
	 * @date  2017/11/20 17:04
	 * @param request
	 * @return java.lang.String
	 */
	@RequestMapping(value = "/getLogList")
	@ResponseBody
	public String getLogList(HttpServletRequest request) {
		String page = request.getParameter("page");
		String rows = request.getParameter("rows");
		String userName = request.getParameter("userName");
		String startTime = request.getParameter("startTime");
		String endTime = request.getParameter("endTime");
		String operation = request.getParameter("operation");

		Map<String, Object> params = new HashMap<String, Object>();
		params.put("page", page);
		params.put("rows", rows);

		Map<String, Object> map = new HashMap<String, Object>();
		map.put("userName", userName);
		map.put("startTime", startTime);
		map.put("endTime", endTime);
		map.put("operation", operation);
		List<SysLog> logList = sysLogService.getLogList(params,map);
		PageInfo<SysLog> pageInfo = new PageInfo<SysLog>(logList);

		JSONObject jo = new JSONObject();
		jo.put("rows", logList);
		// 总页数
		jo.put("total", pageInfo.getPages());
		// 查询出的总记录数
		jo.put("records", pageInfo.getTotal());
		jo.put("code",200);
		return CombaCommonUtil.jsonParseString(jo);
	}

}
```

### 6、前端代码
<h3>前端用的是SpringBoot thymeleaf</h3>
```html
<html xmlns:th="http://www.thymeleaf.org">
<head th:include="include :: sys-header('系统日志')"></head>
<body>
<!--<div style="margin-top: 5px;">-->
	<!--<blockquote class="layui-elem-quote">系统日志列表</blockquote>-->
<!--</div>-->
<div class="logTable" style="margin-top: 10px;margin-left: 10px;">
	用户名：
	<div class="layui-inline">
		<input class="layui-input" name="userName" id="userName" autocomplete="off">
	</div>
	操作：
	<div class="layui-inline">
		<input class="layui-input" name="operation" id="operation" autocomplete="off">
	</div>
	开始时间：
	<div class="layui-inline">
		<input class="layui-input date-item" name="startTime" id="startTime" autocomplete="off" placeholder="yyyy-MM-dd hh:mm:ss">
	</div>
	结束时间：
	<div class="layui-inline">
		<input class="layui-input date-item" name="endTime" id="endTime" autocomplete="off" placeholder="yyyy-MM-dd hh:mm:ss">
	</div>
	<button class="layui-btn" data-type="reload">查询</button>
</div>
<table class="layui-hide" id="table_log" lay-filter="log"></table>

</body>
<script th:replace="basepath :: basePath" th:inline="javascript"></script>
<script>
    initTable();

    layui.use('laydate', function () {
        var laydate = layui.laydate;

        lay('.date-item').each(function(){
            laydate.render({
                elem: this
                ,type: 'datetime'
                ,trigger: 'click'
            });
        });
    });


    function initTable() {
        layui.use('table', function () {
            var table = layui.table;

            //方法级渲染
            table.render({
                elem: '#table_log'
                , url: basePath + '/log/getLogList'
                , method: 'POST'
                ,cellMinWidth: 90
                , cols: [[
//                {checkbox: false, fixed: false}
                    {field: 'userName', title: '用户',fixed: true}
                    , {field: 'operation', title: '操作', sort: true}
                    , {field: 'time', title: '响应时间', sort: true,width:90}
                    , {field: 'method', title: '请求方法', sort: true}
                    , {field: 'params', title: '请求参数'}
                    , {field: 'ip', title: 'ip地址', sort: true}
                    , {field: 'uri', title: '请求路径'}
                    , {field: 'createTime', title: '创建时间', sort: true}
                ]]
                , id: 'id'
                , page: true
                , limit:15
                , limits:[15,20,25,30,35,40,50,60,80]
                ,height: 'full-85'
                ,initSort: {
                    field: 'createTime' //排序字段，对应 cols 设定的各字段名
                    ,type: 'desc' //排序方式  asc: 升序、desc: 降序、null: 默认排序
                }
                , request: {
                    pageName: 'page' //页码的参数名称，默认：page
                    , limitName: 'rows' //每页数据量的参数名，默认：limit
                }
                ,response: { //定义后端 json 格式，详细参见官方文档
                    statusName: 'code', //状态字段名称
                    statusCode: '200', //状态字段成功值
                    countName: 'records', //总数字段
                    dataName: 'rows' //数据字段
                }
            });

            var $ = layui.$, active = {
                reload: function () {
                    var userName = $('#userName').val();
                    var startTime = $('#startTime').val();
                    var endTime = $('#endTime').val();
                    var operation = $('#operation').val();

                    table.reload('id', {
                        where: {
                            userName: userName,
                            startTime: startTime,
                            endTime: endTime,
                            operation: operation
                        }
                    });
                }
            };

            $('.logTable .layui-btn').on('click', function () {
                var type = $(this).data('type');
                active[type] ? active[type].call(this) : '';
            });
        });
    }
</script>

</html>
```