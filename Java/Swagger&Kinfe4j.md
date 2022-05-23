# Swagger

查看地址url：项目地址/doc.html

## 一.是什么？

Swagger是一款RESTFUL接口的文档在线自动生成+功能测试功能软件。Swagger是一个规范和完整的框架,用于生成、描述、调用和可视化RESTful风格的Web服务。目标是使客户端和文件系统作为服务器以同样的速度来更新文件的方法,参数和模型紧密集成到服务器。
这个解释简单点来讲就是说，swagger是一款可以根据resutful风格生成的生成的接口开发文档，并且支持做测试的一款中间软件。

## 二.常用注解

### 1.请求类方法描述注解

#### @ApiIgnore

用在类、方法、参数上，表示这个类或者方法被忽略（不需要swagger处理）。
**注解的属性**

| 属性名称 | 属性类型 | 属性默认值 | 属性描述 |
| --- | --- | --- | --- |
| value | String | "" | 简要说明忽略的原因 |

#### @Api
用在**类**上，对请求类进行描述。标识一个Controller类是Swagger文档类。
**注解的属性**

| **属性名称** | **属性类型** | **属性默认值** | **属性描述** |
| --- | --- | --- | --- |
| value | String | "" | 描述，无实际意义。 |
| tags | String[] | "" | 分组 |
| description | String | "" | 描述（已弃用） |
| basePath | String | "" | 基本路径（已弃用）已无效 |
| position | int | 0 | 显示顺序（已弃用）已无效 |
| produces | String | int | 输出数据内容类型（已弃用）已无效 |
| consumes | String | int | 输入数据内容类型（已弃用）已无效 |
| protocols | String | int | 请求协议 |
| authorizations | Authorization[] | @Authorization(value = "") | 高级特性认证时的配置 |
| hidden | boolean | false | 是否隐藏（不显示） |



**注解属性value、tags二者的区别**

- value属性作用在类和作用在方法上都用于描述；
- tags属性作用在类和作用在方法上都用于分组，但分组的效果区别很大：
- tags作用在类上时，会对全局的方法分组，即根据tags属性值复制多份，此时方法上的tags值无效，方法上tags配或不配效果都一样。
- tags作用在方法上时，会根据当前类的所有方法的tags值做分组，粒度更细。
```java
@RestController
@Slf4j
@Api(tags = "书本模块")
@RequestMapping(value = "/provide")
public class BookController {
}
```
#### @ApiOperation
用在**方法**上，对请求方法进行描述。
**注解的属性**

| **属性名称** | **属性类型** | **属性默认值** | **属性描述** |
| --- | --- | --- | --- |
| value | String |  | 描述 |
| notes | String | "" | 详细描述 |
| tags | String[] | "" | 分组 |
| response | Class<?> | Void.class | 响应参数类型 |
| responseContainer | String | "" | 该属性对这些对象是有效的：List、Set、Map，其他无效。 |
| responseReference | String | "" | 指定对响应类型的引用，本地/远程引用，并将覆盖任何其它指定的response()类 |
| httpMethod | String | "" | http请求方式，如：GET、HEAD、POST、PUT、DELETE、OPTION、SPATCH |
| position | int | 0 | 显示顺序（已弃用）已无效 |
| nickname | String | "" | 第三方工具使用operationId来唯一表示此操作 |
| produces | String | "" | 输出数据内容类型，如：application/json,application/xml（多个用英文逗号隔开） |
| consumes | String | "" | 输入数据内容类型，如：application/json,application/xml（多个用英文逗号隔开） |
| protocols | String | "" | 请求协议 |
| authorizations | Authorization[] | @Authorization(value = "") | 高级特性认证时的配置 |
| hidden | boolean | false | 是否隐藏（不显示） |
| responseHeaders | ResponseHeader[] | @ResponseHeader(name = "", response = Void.class) | 响应头信息，@ResponseHeader数组 |
| code | int | 200 | http的状态码 |
| extensions | Extension[] | @Extension(properties = @ExtensionProperty(name = "", value = "") | 扩展属性 |



**属性包含的注解**

- **@Authorization**
- **@ResponseHeader**
- **@Extension**
```java
@GetMapping("test1")
@ApiOperation(value = "test1接口",notes = "test1接口详细描述")
public ApiResult<String> test1(@RequestParam String aa, @RequestParam String bb, @RequestParam String cc) {
    return ApiUtil.success("success");
}
```
### 2.请求参数描述注解
#### @ApiImplicitParams
常用在**方法**上，对请求参数列表进行描述。
其中的value属性可包含多个@ApiImplicitParam，对每个参加进行具体的描述。
**注解的属性**

| **属性名称** | **属性类型** | **属性默认值** | **属性描述** |
| --- | --- | --- | --- |
| value | ApiImplicitParam[] |  | 参数列表描述 |

**属性包含的注解**

- **@ApiImplicitParam**
#### @ApiImplicitParam
用在**方法**上，对请求参数进行描述。当需要对多个参数进行描述时，作为@ApiImplicitParams的属性使用。
**注解的属性**

| **属性名称** | **属性类型** | **属性默认值** | **属性描述** |
| --- | --- | --- | --- |
| name | String | "" | 参数名 |
| value | String | "" | 参数说明 |
| defaultValue | String | "" | 默认值 |
| allowableValues | String | "" | 参数允许值 |
| required | boolean | false | 是否必填 |
| access | String | "" | 参数过滤 |
| allowMultiple | boolean | false | 参数是否可以通过多次出现来接受多个值，默认不允许 |
| dataType | String | "" | 参数的数据类型，可以使类名或原始数据类型 |
| dataTypeClass | Class<?> | Void.class | 参数的数据类型，如果提供则覆盖 dataType |
| paramType | String | "" | 参数类型，有效值为 path, query, body, header, form |
| example | String | "" | 非body类型的参数示例 |
| examples | Example | @Example(value = @ExampleProperty(mediaType = “”, value = “”)) | body类型的参数示例 |
| type | String | "" | 添加覆盖检测到的类型的功能 |
| format | String | "" | 添加提供自定义format格式的功能 |
| allowEmptyValue | boolean | false | 添加将格式设置为空的功能 |
| readOnly | boolean | false | 添加被指定为只读的功能 |
| collectionFormat | String | "" | 添加使用array类型覆盖collectionFormat的功能 |

**属性包含的注解**

- **@Example**

**属性paramType取值**

- **header** ：通过@RequestHeader获取
- **query**：通过@RequestParam获取
- **path**：通过@PathVariable获取
- **body**：通过@RequestBody获取
- **form**：通过@RequestParam获取
```java
@ApiOperation("修改书籍信息")
@ApiImplicitParam(type = "body", dataType = "BookParams", name = "bookParams", value = "图书修改信息", required = true)
@PostMapping("/alterBookInfo")
@ApiResponses(value = {@ApiResponse(code = 200, message = "修改成功"), @ApiResponse(code = 443, message = "书名不能为空"),
    @ApiResponse(code = 444, message = "修改图书信息失败")})
public JsonModel alterBookInfo(@RequestBody BookParams bookParams) {
    
}


//@ApiImplicitParams(value = {@ApiImplicitParam(name = "bookName", value = "书本名", required = true),@ApiImplicitParam(name = "state", value = "书籍状态", required = true)})
```
#### @ApiParam
用在**方法、参数、类的字段**上，对请求参数进行描述。
**注解的属性**

| **属性名称** | **属性类型** | **属性默认值** | **属性描述** |
| --- | --- | --- | --- |
| name | String | "" | 参数名 |
| value | String | "" | 参数说明 |
| defaultValue | String | "" | 默认值 |
| allowableValues | String | "" | 参数允许值 |
| required | boolean | false | 是否必填 |
| access | String | "" | 参数过滤 |
| allowMultiple | boolean | false | 参数是否可以通过多次出现来接受多个值，默认不允许 |
| hidden | boolean | false | 是否隐藏（不显示），默认false |
| example | String | "" | 非body类型的参数示例 |
| examples | Example | @Example(value = @ExampleProperty(mediaType = "", value = "")) | body类型的参数示例 |
| type | String | "" | 添加覆盖检测到的类型的功能 |
| format | String | "" | 添加提供自定义format格式的功能 |
| allowEmptyValue | boolean | false | 添加将格式设置为空的功能 |
| readOnly | boolean | false | 添加被指定为只读的功能 |
| collectionFormat | String | "" | 添加使用array类型覆盖collectionFormat的功能 |

**属性包含的注解**

- **@Example**

**使用方法**
```java
 @GetMapping("test2")
@ApiOperation(value = "test2接口",notes = "test2接口详细描述")
public ApiResult<TestRes> test2(@ApiParam(value = "aa的说明") @RequestParam String aa, @ApiParam(value = "bb的说明") @RequestParam String bb) {
    return ApiUtil.success(new TestRes());
}
```
### 3.对象描述(请求响应)注解
#### @ApiModel
用在**类**上，对请求、响应类进行描述。
**注解的属性**

| **属性名称** | **属性类型** | **属性默认值** | **属性描述** |
| --- | --- | --- | --- |
| value | String | "" | 为提供模型的替代名称，默认情况下，使用类名 |
| description | String | "" | 类的描述 |
| parent | Class<?> parent | Void.class | 为模型提供父类以允许描述继承关系 |
| discriminatory | String | "" | 支持模型继承和多态，使用鉴别器的字段的名称，可以断言需要使用哪个子类型 |
| subTypes | Class<?>[] | {} | 从此模型继承的子类型数组 |
| reference | String | "" | 指定对应类型定义的引用，覆盖指定的任何其他元数据 |

#### @ApiModelProperty
常用在**字段**上，对请求、响应类的字段进行描述。
**注解的属性**

| **属性名称** | **属性类型** | **属性默认值** | **属性描述** |
| --- | --- | --- | --- |
| name | String | "" | 覆盖属性的名称 |
| value | String | "" | 属性说明 |
| allowableValues | String | "" | 参数允许值 |
| access | String | "" | 过滤属性 |
| notes | String | "" | 目前尚未使用 |
| dataType | String | "" | 参数的数据类型，可以使类名或原始数据类型 |
| required | boolean | false | 是否必填 |
| position | int | 0 | 显示顺序 |
| hidden | boolean | false | 是否隐藏 |
| example | String | "" | 属性的示例值 |
| readOnly | boolean | false | 添加被指定为只读的功能 |
| reference | String | "" | 指定对应类型定义的引用，覆盖指定的任何其他元数据 |
| allowEmptyValue | boolean | false | 允许传空值 |

```java
@Data
@ApiModel(description = "测试请求类")
public class TestReq {

    @ApiModelProperty(value = "用户ID",required = true)
    private Long userId;
    @ApiModelProperty(value = "用户名",example = "张三")
    private String userName;
    @ApiModelProperty(value = "性别：1男、2女")
    private Integer sex;
    @ApiModelProperty(value = "昵称")
    private String nickName;
    @ApiModelProperty(value = "邮箱")
    private String email;
    @ApiModelProperty(value = "创建时间")
    private Date createTime;
}

@Data
@ApiModel(description = "测试响应类")
public class TestRes {

    @ApiModelProperty(value = "用户ID",required = true)
    private Long userId;
    @ApiModelProperty(value = "用户名")
    private String userName;
    @ApiModelProperty(value = "性别：1男、2女")
    private Integer sex;
    @ApiModelProperty(value = "昵称")
    private String nickName;
    @ApiModelProperty(value = "邮箱")
    private String email;
    @ApiModelProperty(value = "创建时间")
    private Date createTime;
}
```
### 4.响应参数描述注解
#### @ApiResponses
用在**方法、类**上，对响应状态码列表进行描述。
**注解的属性**

| **属性名称** | **属性类型** | **属性默认值** | **属性描述** |
| --- | --- | --- | --- |
| value | ApiResponse[] |  | 响应状态码列表的描述 |

**属性包含的注解**

- **@ApiResponse**
#### @ApiResponse
用在**方法**上，对响应状态码进行描述。一般作为@ApiResponses的属性使用。
**注解的属性**

| **属性名称** | **属性类型** | **属性默认值** | **属性描述** |
| --- | --- | --- | --- |
| code | int |  | 响应的HTTP状态码 |
| message | String |  | 响应的描述 |
| response | Class<?> | Void.class | 用于描述消息有效负载的可选响应类，对应于响应消息对象的 schema 字段 |
| reference | String | "" | 指定对响应类型的引用，指定的应用可以使本地引用，也可以是远程引用，将按原样使用，并将覆盖任何指定的response()类 |
| responseHeaders | ResponseHeader[] | @ResponseHeader(name = "", response = Void.class) | 可能响应的header列表 |
| responseContainer | String | "" | 声明响应的容器，有效值为List,Set,Map，任何其他值都将被忽略 |

**属性包含的注解**

- **@ResponseHeader**
## @ResponseHeader
用在**方法**上，对响应头信息进行描述。一般作为@ApiResponse的属性使用。
**注解的属性**

| **属性名称** | **属性类型** | **属性默认值** | **属性描述** |
| --- | --- | --- | --- |
| name | String | "" | 响应头参数名称 |
| description | String | "" | 响应头参数详细描述 |
| response | Class<?> | Void.class | 响应头数据类型 |
| responseContainer | String | "" | 声明包装响应的容器，有效值为List或Set，任何其他值都将被覆盖 |

```java
@GetMapping("test2")
@ApiOperation(value = "test2接口",notes = "test2接口详细描述")
@ApiResponses(value = {
        @ApiResponse(code = 200, message = "请求成功", responseHeaders = {@ResponseHeader(name = "header1", description = "header1的描述",response = String.class)}),
        @ApiResponse(code = 401, message = "没有权限"),
        @ApiResponse(code = 403, message = "禁止访问")
})
public ApiResult<TestRes> test2(@ApiParam(value = "aa的说明") @RequestParam String aa, @ApiParam(value = "bb的说明") @RequestParam String bb) {
    return ApiUtil.success(new TestRes());
}
```
## Knife4j

### 1.导入依赖

```xml
<dependency>
  <groupId>com.github.xiaoymin</groupId>
  <artifactId>knife4j-spring-boot-starter</artifactId>
  <!--在引用时请在maven中央仓库搜索2.X最新版本号-->
  <version>2.0.9</version>
</dependency>
```
### 2.创建Swagger配置依赖

```java
@Configuration
@EnableSwagger2WebMvc
public class Knife4jConfiguration {
    @Bean(value = "defaultApi2")
    public Docket defaultApi2() {
        Docket docket=new Docket(DocumentationType.SWAGGER_2)
            .apiInfo(new ApiInfoBuilder()
                     //.title("swagger-bootstrap-ui-demo RESTful APIs")
                     .description("# swagger-bootstrap-ui-demo RESTful APIs")
                     .termsOfServiceUrl("http://www.xx.com/")
                     .contact("xx@qq.com")
                     .version("1.0")
                     .build())
            //分组名称
            .groupName("2.X版本")
            .select()
            //这里指定Controller扫描包路径
            .apis(RequestHandlerSelectors.basePackage("com.github.xiaoymin.knife4j.controller"))
            .paths(PathSelectors.any())
            .build();
        return docket;
    }
}

```
