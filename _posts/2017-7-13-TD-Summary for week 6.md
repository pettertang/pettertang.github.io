---
layout: post_layout
title: TD-Summary for week 6
time: 2017年7月13日 星期四
location: 北京
pulished: true
excerpt_separator: "**2**.项目中遇到的第一个问题"
---
## 第6周总结
**1**.这周主要做运营管理后台的后端工作，简单的说就是一些后端对数据库的增删改查操作。
技术栈上数据持久层框架项目中用的是Mybatis，相比于上一个项目中用的Spring MVC自己的JDBC处理框架，Mybatis确实更加强大。将SQL语句和代码完全分离，大大增强了代码的可读性和项目的可维护性，这点真的是切身体会到了。

---
**2**.项目中遇到的第一个问题是一个和SQL相关的问题。需求背景是：用一条SQL语句找到连续编号的序列的第一个间断处的开始值。比如：对于序列{1,2,5,6,9,13...}，则返回3；对于序列{1,5,9,23...}，则返回2。一种解决的SQL语句如下：
```SQL
SELECT sort + 1
       FROM `subject_item` t1
       WHERE NOT EXISTS (SELECT sort
                          FROM `subject_item` t2
                          WHERE t2.sort = t1.sort + 1
                          AND t2.deleted = 0
                          AND t2.parent_id = #{parentId}
                        )
       AND t1.deleted = 0
       AND t1.parent_id = #{parentId}
       ORDER BY sort ASC limit 1;
```
说明：sort是我需要查询的字段。抛开具体业务场景通用的SQL描述如下：
```SQL
SELECT {name} + 1
       FROM `table` t1
       WHERE NOT EXISTS (SELECT sort
                          FROM `table` t2
                          WHERE t2.{name} = t1.{name} + 1
                        )
       ORDER BY {name} ASC limit 1;
```
说明：{name}是要查询的字段，table是要查找的表。

---
**3**.也是SQL相关的一个问题。因为第一次用Mybatis，所以碰到很多初级问题，还是记录一下吧。
```SQL
<update id="update" parameterType="com.td.sdmk.om.domain.CarouselItem">
        UPDATE `carousel_item`
        SET
        <if test="img != null">
            `img` = #{img}, 
        </if>
        <if test="extension != null">
            `extension` = #{extension},
        </if>
        <if test="extensionType != null">
            `extension_type` = #{extensionType},
        </if>
        <if test="sort != null">
            `sort` = #{sort}, 
        </if>
            `modifier` = #{modifier}
        WHERE
            deleted = 0
        AND `id` = #{id}
</update>
```
Mybatis的某个xml文件中有这么一个更新操作，"test="后面的变量名应该是和对应的POJO对象中申明的变量名一样的；下面赋值语句等号左边的变量名应该是和对应数据库表中的字段变量名一样的。
```SQL
<select id="read" resultMap="readMap" parameterType="int" useCache="false">
        SELECT
            t1.id,
            t1.cover_img,
            t1.cover_img_thumbnail,
            t1.sort,
            t2.id itemId,
            t2.service_type,
            t2.service_id,
            t2.sort itemSort
        FROM
            `subject` t1,
            `subject_item` t2
        WHERE
            t1.deleted = 0
            AND t2.deleted = 0
            AND t1.id = t2.parent_id
            AND t1.id = #{id}
        ORDER BY
            t1.sort,
            t2.sort
</select>
```
再比如这样一个查询操作，需要在一次操作中查询两张表，两张表中有相同的字段名，比如这里的id和sort，那么我们在SELECT语句中应该给相同的字段名取别名，以免混淆。比如这里的 t2.id itemId 和 t2.sort itemSort，就是为了和t1.id以及t1.sort区别开来。这里一旦没有用别名区别开来，那么SQL语句是查询不到正确的值得。比如我刚开始没有给t2.sort取一个别名，最后取到的sort值永远就是t1.sort的值，后来发现问题后想想是很明显的。  
相应的，上面这个查询语句中的resultMap定义如下：
```SQL
<resultMap id="readMap" type="com.td.sdmk.om.domain.Subject">
        <id property="id" column="id"/>
        <result property="coverImg" column="cover_img"/>
        <result property="coverImgThumbnail" column="cover_img_thumbnail"/>
        <result property="sort" column="sort"/>
        <collection property="items" ofType="com.td.sdmk.om.domain.SubjectItem">
            <id property="id" column="itemId"/>
            <result property="serviceType" column="service_type"/>
            <result property="serviceId" column="service_id"/>
            <result property="sort" column="itemSort"/>
        </collection>
</resultMap>
```
结构很清晰，也直观易懂。这里要提醒注意的是，在内部嵌套中，我们可以看到，column值对应的本该是相应property值在数据库表中对应的列名，但是因为上面说到的两个表可能有相同列名的情况，我们要将对应的修改了别名的column值在这里也做相应的修改，比如这里的column="itemId"和column="itemSort"。  
以上这两个问题对于刚接触Mybatis来说可能是会不小心犯的错误，而且由于SQL语句和代码完全分离开了，这种问题出现后在程序调试过程中还不太好发现，不过相信熟悉后这种问题是基本可以避免的。

---
**4**.项目中对json的解析处理采用的是阿里巴巴开源的FastJson处理包，关于fastjson这里暂时不展开详细的叙述。fastjson在性能方面非常强大。

---
**5**.关于处理请求异常时后端给前端返回必要错误信息的情况。  
有时候当请求出现异常时，前端需要根据从后端返回的Response信息中获取到异常发生的原因，从而做出相应的提示处理。一般来说，http请求中我们用Json格式来传输相应的一些信息。下面用我在项目中碰到的具体情况来简单说明一下。
```Java
if (StringUtils.isBlank(subjectItem.getServiceId())) {
            throw new HttpBadRequestException("The serviceId is null");
        }
```
比如上面这段代码，需要让前端获取到这个异常信息，我们首先找到相应的处理请求Response的代码处，如下：
```Java
public class HttpExceptionMapper implements ExceptionMapper<Exception> {
    public Response toResponse(Exception e) {
        if(e instanceof HttpException){
            HttpException httpException = (HttpException)e;
            return Response
                    .status(httpException.httpStatus())
                    .entity(new MessageResponse(httpException.httpStatus(),httpException.getMessage()))
                    .type(MediaType.APPLICATION_JSON_TYPE)
                    .build();
        }
        LogWriter.getErrorLog().error("", e);
        return Response.status(HttpStatus.INTERNAL_SERVER_ERROR).build();

    }
}
```
可以很清楚的看到返回信息的类型为JSON_TYPE，相应返回信息在entity中，前端可以从这里获取到。对应的MessageResponse类为我们自己定义的应答信息返回类，我定义的返回信息有两个字段，为相应的Http状态码和msg描述信息。
```Java
public class MessageResponse {
    private Integer code;
    private String msg;

    public MessageResponse() {
    }

    public MessageResponse(int code, String msg) {
        super();
        this.code = code;
        this.msg = msg;
    }
    ...
    ...
    ...
}
```
这样Response的Body部分如下所示：
```Json
{
    "code": 400,
    "msg": "The sort number is null"
}
```
当然，项目中的异常处理肯定是会单独抽离出来做成一个切面，不同的框架的异常处理也各有不同，这里也不再继续展开了。