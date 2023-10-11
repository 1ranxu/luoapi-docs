# 落樱接口

背景：

1. 前端开发需要用到后端j接口
2. 使用别人现成系统的功能[搏天api-免费api接口平台 (btstu.cn)](https://api.btstu.cn/)

考虑：

1. 防止攻击（安全性）
2. 不能随便调用（增加限制）
3. 统计调用次数
4. 计费
5. 流量保护（限制单位i时间请求数量）
6. API接入（每开发一个API要复用以上的特性）

## 项目介绍

落樱接口是一个提供接口调用的平台，用户可以注册登录，开通接口调用权限，用户可以调用接口，每次调用会进行统计。

管理员可以发布接口，下线接口，接入接口，可视化接口的调用情况

## 业务流程

![img](assets/1693740845735-c8b683ac-f925-47dd-9da3-5988b43d78c0.png)

## 技术选型

**前端**

- Ant Design Pro

- React

- Ant Design & Pro Components

- Umi
- Umi Request （Axios的封装）

**后端**

- Java
- SpringBoot

## 计划

**初始化**

- 前端
- 后端

**数据库表设计**

**后端**

- 基础功能
  - 管理员可以对接口进行增删改查
- 接口调用
  - 开发模拟接口
  - 调用模拟接口
  - 保证调用的安全性（API签名认证）
  - 客户端SDK开发
  - 管理员发布/下线接口
  - 申请签名
  - 在线调用
- 

**前端**

- 基础功能

  - 管理员可以访问前台查看接口

  - 管理员创建接口，更新接口，删除接口

- 接口调用

  - 管理员发布/下线接口
  - 用户浏览接口
  - 用户查看接口文档
  - 在线调用
  - 统计用户调用接口次数
  - 优化系统 -  API网关

**接口计费与保护**

- 统计用户调用次数

- 限流

- 计费 

- 日志

- 开通

**接口统计**

- 提供可视化平台，用图表展示所有接口的调用情况，便于管理
- 接口预警

## 需求分析

1. 管理员可以对接口进行增删改查
2. 用户可以访问前台查看，查看接口信息

## 初始化

### 前端

1. 初始化脚手架

```sh
#使用pro-cli 来快速的初始化脚手架。
npm i @ant-design/pro-cli -g
pro create API-OpenPlatform-frontend
```

![image-20231003154738176](assets/image-20231003154738176.png)

2. 确认版本

![image-20231003155255506](assets/image-20231003155255506.png)

3. 安装依赖

```sh
yarn
```

### 后端

使用SpringBoot 项目初始模板(旧)

## 数据库表设计

**接口信息表interface_info**

id

name 接口名称

description 接口描述

url 接口地址

requestHeader 请求头

responseHeader 响应头

status 接口状态 0-关闭 1-开启

method 请求类型

userId 创建人

isDelete 

createTime

updateTime

```sql
create table interface_info
(
    id             bigint auto_increment comment '主键'
        primary key,
    name           varchar(256)                       not null comment '接口名称',
    description    varchar(1024)                      null comment '接口描述',
    url            varchar(512)                       not null comment '接口地址',
    requestParams  text                               null comment '请求参数',
    requestHeader  text                               null comment '请求头',
    responseHeader text                               null comment '响应头',
    status         tinyint  default 0                 not null comment '接口状态 0-关闭 1-开启',
    method         varchar(256)                       not null comment '请求类型',
    userId         bigint                             not null comment '创建人',
    createTime     datetime default CURRENT_TIMESTAMP not null comment '创建时间',
    updateTime     datetime default CURRENT_TIMESTAMP not null on update CURRENT_TIMESTAMP comment '更新时间',
    idDelete       tinyint  default 0                 not null comment '逻辑删除'
);
```

## 后端

### 基础功能

#### **管理员可以对接口进行增删改查**

1. 使用MybatisX自动生成mapper层和service层的代码

![image-20231005150931600](assets/image-20231005150931600.png)

![image-20231005151308188](assets/image-20231005151308188.png)

```java
@Data
public class InterfaceInfoAddRequest implements Serializable {
    /**
     * 接口名称
     */
    private String name;

    /**
     * 接口描述
     */
    private String description;

    /**
     * 接口地址
     */
    private String url;

    /**
     * 请求头
     */
    private String requestHeader;

    /**
     * 响应头
     */
    private String responseHeader;

    /**
     * 请求类型
     */
    private String method;
}
```

```java
@EqualsAndHashCode(callSuper = true)
@Data
public class InterfaceInfoQueryRequest extends PageRequest implements Serializable {
    /**
     * id
     */
    private Long id;

    /**
     * 接口名称
     */
    private String name;

    /**
     * 接口描述
     */
    private String description;

    /**
     * 接口地址
     */
    private String url;

    /**
     * 请求头
     */
    private String requestHeader;

    /**
     * 响应头
     */
    private String responseHeader;

    /**
     * 接口状态 0-关闭 1-开启
     */
    private Integer status;

    /**
     * 请求类型
     */
    private String method;

    /**
     * 创建人
     */
    private Long userId;
}
```

```java
@Data
public class InterfaceInfoUpdateRequest implements Serializable {
    /**
     * 主键
     */
    private Long id;

    /**
     * 接口名称
     */
    private String name;

    /**
     * 接口描述
     */
    private String description;

    /**
     * 接口地址
     */
    private String url;

    /**
     * 请求头
     */
    private String requestHeader;

    /**
     * 响应头
     */
    private String responseHeader;

    /**
     * 接口状态 0-关闭 1-开启
     */
    private Integer status;

    /**
     * 请求类型
     */
    private String method;
}
```

```java
public interface InterfaceInfoService extends IService<InterfaceInfo> {

    /**
     * 校验
     *
     * @param interfaceInfo
     * @param add 是否为创建校验
     */
    void validInterfaceInfo(InterfaceInfo interfaceInfo, boolean add);
}
```

```java
@Service
public class InterfaceInfoServiceImpl extends ServiceImpl<InterfaceInfoMapper, InterfaceInfo>
        implements InterfaceInfoService {
    @Override
    public void validInterfaceInfo(InterfaceInfo interfaceInfo, boolean add) {
        Long id = interfaceInfo.getId();
        String name = interfaceInfo.getName();
        String description = interfaceInfo.getDescription();
        String url = interfaceInfo.getUrl();
        String requestHeader = interfaceInfo.getRequestHeader();
        String responseHeader = interfaceInfo.getResponseHeader();
        Integer status = interfaceInfo.getStatus();
        String method = interfaceInfo.getMethod();
        Long userId = interfaceInfo.getUserId();
        Date createTime = interfaceInfo.getCreateTime();
        Date updateTime = interfaceInfo.getUpdateTime();
        Integer idDelete = interfaceInfo.getIdDelete();
        if (interfaceInfo == null) {
            throw new BusinessException(ErrorCode.PARAMS_ERROR);
        }
        // 创建时，所有参数必须非空
        if (add) {
            if (StringUtils.isAnyBlank(name)) {
                throw new BusinessException(ErrorCode.PARAMS_ERROR);
            }
        }
        if (StringUtils.isAnyBlank(name) && name.length() > 50) {
            throw new BusinessException(ErrorCode.PARAMS_ERROR, "接口名称过长");
        }
    }
}
```

2. 复制PostContoller改写成InterfacerInfoController

```java
/**
 * 接口信息
 *
 * @author 落樱的悔恨
 */
@RestController
@RequestMapping("/interfaceInfo")
@Slf4j
public class InterfaceInfoController {

    @Resource
    private InterfaceInfoService interfaceInfoService;

    @Resource
    private UserService userService;

    // region 增删改查

    /**
     * 创建
     *
     * @param interfaceInfoAddRequest
     * @param request
     * @return
     */
    @PostMapping("/add")
    public BaseResponse<Long> addInterfaceInfo(@RequestBody InterfaceInfoAddRequest interfaceInfoAddRequest, HttpServletRequest request) {
        if (interfaceInfoAddRequest == null) {
            throw new BusinessException(ErrorCode.PARAMS_ERROR);
        }
        InterfaceInfo interfaceInfo = new InterfaceInfo();
        BeanUtils.copyProperties(interfaceInfoAddRequest, interfaceInfo);
        // 校验
        interfaceInfoService.validInterfaceInfo(interfaceInfo, true);
        User loginUser = userService.getLoginUser(request);
        interfaceInfo.setUserId(loginUser.getId());
        boolean result = interfaceInfoService.save(interfaceInfo);
        if (!result) {
            throw new BusinessException(ErrorCode.OPERATION_ERROR);
        }
        long newInterfaceInfoId = interfaceInfo.getId();
        return ResultUtils.success(newInterfaceInfoId);
    }

    /**
     * 删除
     *
     * @param deleteRequest
     * @param request
     * @return
     */
    @PostMapping("/delete")
    public BaseResponse<Boolean> deleteInterfaceInfo(@RequestBody DeleteRequest deleteRequest, HttpServletRequest request) {
        if (deleteRequest == null || deleteRequest.getId() <= 0) {
            throw new BusinessException(ErrorCode.PARAMS_ERROR);
        }
        User user = userService.getLoginUser(request);
        long id = deleteRequest.getId();
        // 判断是否存在
        InterfaceInfo oldInterfaceInfo = interfaceInfoService.getById(id);
        if (oldInterfaceInfo == null) {
            throw new BusinessException(ErrorCode.NOT_FOUND_ERROR);
        }
        // 仅本人或管理员可删除
        if (!oldInterfaceInfo.getUserId().equals(user.getId()) && !userService.isAdmin(request)) {
            throw new BusinessException(ErrorCode.NO_AUTH_ERROR);
        }
        boolean b = interfaceInfoService.removeById(id);
        return ResultUtils.success(b);
    }

    /**
     * 更新
     *
     * @param interfaceInfoUpdateRequest
     * @param request
     * @return
     */
    @PostMapping("/update")
    public BaseResponse<Boolean> updateInterfaceInfo(@RequestBody InterfaceInfoUpdateRequest interfaceInfoUpdateRequest,
                                            HttpServletRequest request) {
        if (interfaceInfoUpdateRequest == null || interfaceInfoUpdateRequest.getId() <= 0) {
            throw new BusinessException(ErrorCode.PARAMS_ERROR);
        }
        InterfaceInfo interfaceInfo = new InterfaceInfo();
        BeanUtils.copyProperties(interfaceInfoUpdateRequest, interfaceInfo);
        // 参数校验
        interfaceInfoService.validInterfaceInfo(interfaceInfo, false);
        User user = userService.getLoginUser(request);
        long id = interfaceInfoUpdateRequest.getId();
        // 判断是否存在
        InterfaceInfo oldInterfaceInfo = interfaceInfoService.getById(id);
        if (oldInterfaceInfo == null) {
            throw new BusinessException(ErrorCode.NOT_FOUND_ERROR);
        }
        // 仅本人或管理员可修改
        if (!oldInterfaceInfo.getUserId().equals(user.getId()) && !userService.isAdmin(request)) {
            throw new BusinessException(ErrorCode.NO_AUTH_ERROR);
        }
        boolean result = interfaceInfoService.updateById(interfaceInfo);
        return ResultUtils.success(result);
    }

    /**
     * 根据 id 获取
     *
     * @param id
     * @return
     */
    @GetMapping("/get")
    public BaseResponse<InterfaceInfo> getInterfaceInfoById(long id) {
        if (id <= 0) {
            throw new BusinessException(ErrorCode.PARAMS_ERROR);
        }
        InterfaceInfo interfaceInfo = interfaceInfoService.getById(id);
        return ResultUtils.success(interfaceInfo);
    }

    /**
     * 获取列表（仅管理员可使用）
     *
     * @param interfaceInfoQueryRequest
     * @return
     */
    @AuthCheck(mustRole = "admin")
    @GetMapping("/list")
    public BaseResponse<List<InterfaceInfo>> listInterfaceInfo(InterfaceInfoQueryRequest interfaceInfoQueryRequest) {
        InterfaceInfo interfaceInfoQuery = new InterfaceInfo();
        if (interfaceInfoQueryRequest != null) {
            BeanUtils.copyProperties(interfaceInfoQueryRequest, interfaceInfoQuery);
        }
        QueryWrapper<InterfaceInfo> queryWrapper = new QueryWrapper<>(interfaceInfoQuery);
        List<InterfaceInfo> interfaceInfoList = interfaceInfoService.list(queryWrapper);
        return ResultUtils.success(interfaceInfoList);
    }

    /**
     * 分页获取列表
     *
     * @param interfaceInfoQueryRequest
     * @param request
     * @return
     */
    @GetMapping("/list/page")
    public BaseResponse<Page<InterfaceInfo>> listInterfaceInfoByPage(InterfaceInfoQueryRequest interfaceInfoQueryRequest, HttpServletRequest request) {
        if (interfaceInfoQueryRequest == null) {
            throw new BusinessException(ErrorCode.PARAMS_ERROR);
        }
        InterfaceInfo interfaceInfoQuery = new InterfaceInfo();
        BeanUtils.copyProperties(interfaceInfoQueryRequest, interfaceInfoQuery);
        long current = interfaceInfoQueryRequest.getCurrent();
        long size = interfaceInfoQueryRequest.getPageSize();
        String sortField = interfaceInfoQueryRequest.getSortField();
        String sortOrder = interfaceInfoQueryRequest.getSortOrder();
        String description = interfaceInfoQuery.getDescription();
        // content 需支持模糊搜索
        interfaceInfoQuery.setDescription(null);
        // 限制爬虫
        if (size > 50) {
            throw new BusinessException(ErrorCode.PARAMS_ERROR);
        }
        QueryWrapper<InterfaceInfo> queryWrapper = new QueryWrapper<>(interfaceInfoQuery);
        queryWrapper.like(StringUtils.isNotBlank(description), "description", description);
        queryWrapper.orderBy(StringUtils.isNotBlank(sortField),
                sortOrder.equals(CommonConstant.SORT_ORDER_ASC), sortField);
        Page<InterfaceInfo> interfaceInfoPage = interfaceInfoService.page(new Page<>(current, size), queryWrapper);
        return ResultUtils.success(interfaceInfoPage);
    }

    // endregion

}
```

### 接口调用

#### 开发模拟接口

![image-20231008220820591](assets/image-20231008220820591.png)

![image-20231008221011560](assets/image-20231008221011560.png)

![image-20231008221526440](assets/image-20231008221526440.png)

![image-20231008221220921](assets/image-20231008221220921.png)

![image-20231008221327611](assets/image-20231008221327611.png)

#### 调用模拟接口

![image-20231008222119499](assets/image-20231008222119499.png)

![image-20231008222229448](assets/image-20231008222229448.png)

#### API签名认证

**本质**：

1. 签发签名
2. 使用签名（校验签名）

**为什么需要？**

1. 保证安全性，不能随便一个人调用，必须有签发的签名
2. 适用于无需保存登录态的场景，只认签名，不关注用户登录态

**签名认证实现：**

通过http request header头实现

参数1：accessKey：调用者的标识。相当于用户名

参数2：secretKey:密钥（复杂，无规律，无序），开通获取。相当于密码

不能把secretKey放到请求头中，密钥在服务器之间传递，有可能被拦截

参数3：用户参数

参数4：sign，使用 参数3+加密算法生成（不可解密）服务器端用一模一样的参数和算法生成签名，只要和用户传的一致，就能通过。

**防重放：**

参数5：加nonce随机数，只能用一次，服务器要保存用过的随机数

参数6：时间戳，校验时间戳是否过期

```java
/**
 * 调用第三方接口的客户端
 * @author 落樱的悔恨
 */
public class LuoApiClient {
    private String accessKey;
    private String secretKey;

    public LuoApiClient(String accessKey, String secretKey) {
        this.accessKey = accessKey;
        this.secretKey = secretKey;
    }


    public Map<String,String> getHeaders(String body){
        Map<String,String> map=new HashMap<>();
        map.put("accessKey",accessKey);
        // map.put("secretKey",secretKey);不能发送
        // map.put("nonce", RandomUtil.randomNumbers(100));
        map.put("body",body);
        map.put("timestamp",String.valueOf(System.currentTimeMillis()+1*60*1000));
        map.put("sign", SignUtil.genSign(body,secretKey));
        return map;
    }


    public String getUsernameByPost(User user) {
        String jsonStr = JSONUtil.toJsonStr(user);
        HttpResponse response =  HttpRequest.post("http://localhost:8002/api/name")
                .addHeaders(getHeaders(jsonStr))
                .body(jsonStr)
                .execute();
        System.out.println(response.getStatus());

        System.out.println("result = " + response.body());
        return response.body();
    }
    public static void main(String[] args) {
        LuoApiClient luoApiClient = new LuoApiClient("ranxu","abc");
        User user = new User();
        user.setUsername("ranxu");
        luoApiClient.getUsernameByPost(user);
    }
}
```

```java
public class SignUtil {
    public static String genSign(String body, String secretKey){
        Digester SHA256 = new Digester(DigestAlgorithm.SHA256);
        String content = body + ".secretKey" + secretKey;
        return SHA256.digestHex(content.getBytes());
    }
}
```

```java
@RestController
@RequestMapping("/name")
public class NameController {
    @PostMapping
    public String getUsernameByPost(@RequestBody User user, HttpServletRequest request) {
        String accessKey = request.getHeader("accessKey");
        String sign = request.getHeader("sign");
        String body = JSONUtil.toJsonStr(user);
        String timestamp= request.getHeader("timestamp");
        Date timestamp1 = new Date(Long.valueOf(timestamp).longValue());
        //todo 根据accessKey查询数据库，是否存在包含改accessKey的记录
        if (!"ranxu".equals(accessKey) ) {
            throw new RuntimeException("无权限");
        }
        if (timestamp1.before(new Date())){
            throw new RuntimeException("无权限");
        }
        // todo 根据记录获取secretKey
        String dbSign = SignUtil.genSign(body, "abc");
        if (!dbSign.equals(sign)){
            throw new RuntimeException("无权限");
        }
        return "post 你的用户名：" + user.getUsername();
    }
}
```

#### 客户端SDK开发

项目名：luoapi-client-sdk

**为什么需要starter?**

理想情况：开发者只需要关心调用哪些接口，传递哪些参数，就跟调用自己的代码一样

开发starter的好处：开发者引入之后，可以在application.yml中写配置，自动创建客户端

**starter开发流程**

初始化，引入依赖

spring-boot-configuration-processor的作用是自动生成配置的代码提示

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-configuration-processor</artifactId>
    <optional>true</optional>
</dependency>
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <optional>true</optional>
</dependency>
<dependency>
    <groupId>cn.hutool</groupId>
    <artifactId>hutool-all</artifactId>
    <version>5.8.8</version>
</dependency>
```

编写配置类

```java
@Configuration
@ConfigurationProperties("luoapi.client")
@Data
@ComponentScan
public class LuoApiClientConfig {

    private String accessKey;
    private String secretKey;

    @Bean
    public LuoApiClient luoApiClient(){
        return new LuoApiClient(accessKey, secretKey);
    }
}
```

注册配置类，resources/META-INF/spring.factories文件

```xml
# springboot starter
org.springframework.boot.autoconfigure.EnableAutoConfiguration=com.luoying.LuoApiClientConfig
```

mvn install打包代码为本地依赖包

创建新项目，测试

#### 管理员发布/下线接口

**接口发布**（仅管理员可操作）

1. 校验该接口是否存在
2. 判断该接口是否可以调用
3. 修改接口数据库中接口的状态字段为1

```java
/**
 * 发布
 *
 * @param idRequest
 * @param request
 * @return
 */
@PostMapping("/online")
@AuthCheck(mustRole = "admin")
public BaseResponse<Boolean> onlineInterfaceInfo(@RequestBody IdRequest idRequest,
                                                 HttpServletRequest request) {
    if (idRequest == null || idRequest.getId() <= 0) {
        throw new BusinessException(ErrorCode.PARAMS_ERROR);
    }
    // 判断接口是否存在
    long id = idRequest.getId();
    InterfaceInfo oldInterfaceInfo = interfaceInfoService.getById(id);
    if (oldInterfaceInfo == null) {
        throw new BusinessException(ErrorCode.NOT_FOUND_ERROR);
    }
    //判断该接口是否可以调用
    com.luoying.model.User user = new com.luoying.model.User();
    user.setUsername("冉旭");
    String username = luoApiClient.getUsernameByPost(user);
    if (StringUtils.isBlank(username)){
        throw new BusinessException(ErrorCode.SYSTEM_ERROR,"接口验证失败");
    }
    //修改接口数据库中接口的状态字段为1
    oldInterfaceInfo.setStatus(InterfaceInfoStatusEnum.ONLINE.getValue());
    boolean result = interfaceInfoService.save(oldInterfaceInfo);
    return ResultUtils.success(result);
}
```

**接口下线**（仅管理员可操作）

1. 校验该接口是否存在
2. 修改接口数据库中接口的状态字段为0

```java
/**
 * 下线
 *
 * @param idRequest
 * @param request
 * @return
 */
@PostMapping("/offline")
@AuthCheck(mustRole = "admin")
public BaseResponse<Boolean> offlineInterfaceInfo(@RequestBody IdRequest idRequest,
                                                  HttpServletRequest request) {
    // 参数校验
    if (idRequest == null || idRequest.getId() <= 0) {
        throw new BusinessException(ErrorCode.PARAMS_ERROR);
    }
    long id = idRequest.getId();
    // 判断接口是否存在
    InterfaceInfo oldInterfaceInfo = interfaceInfoService.getById(id);
    if (oldInterfaceInfo == null) {
        throw new BusinessException(ErrorCode.NOT_FOUND_ERROR);
    }
    //修改接口数据库中接口的状态字段为1
    oldInterfaceInfo.setStatus(InterfaceInfoStatusEnum.OFFLINE.getValue());
    boolean result = interfaceInfoService.save(oldInterfaceInfo);
    return ResultUtils.success(result);
}
```

#### 申请签名

**用户注册时自动分配**

**![image-20231010205032948](assets/image-20231010205032948.png)**

![image-20231010205129096](assets/image-20231010205129096.png)

#### 在线调用

1. 前端将用户输入的请求参数和要测试的接口id发给接口管理平台后端
2. 调用前校验
3. 接口管理平台后端使用sdk客户端去调用模拟接口

```java
/**
 * 接口调用
 *
 * @param interfaceInfoInvokeRequest
 * @param request
 * @return
 */
@PostMapping("/invoke")
public BaseResponse<Object> invokeInterfaceInfo(@RequestBody InterfaceInfoInvokeRequest interfaceInfoInvokeRequest,
                                                 HttpServletRequest request) {
    // 参数校验
    if (interfaceInfoInvokeRequest == null || interfaceInfoInvokeRequest.getId() <= 0) {
        throw new BusinessException(ErrorCode.PARAMS_ERROR);
    }
    long id = interfaceInfoInvokeRequest.getId();
    String userRequestParams = interfaceInfoInvokeRequest.getUserRequestParams();
    // 判断接口是否存在
    InterfaceInfo oldInterfaceInfo = interfaceInfoService.getById(id);
    if (oldInterfaceInfo == null) {
        throw new BusinessException(ErrorCode.NOT_FOUND_ERROR);
    }
    //判断接口状态
    if (oldInterfaceInfo.getStatus().equals(InterfaceInfoStatusEnum.OFFLINE.getValue())) {
        throw new BusinessException(ErrorCode.PARAMS_ERROR,"接口未开放");
    }
    //判断该用户是否有签名
    User loginUser = userService.getLoginUser(request);
    String accessKey = loginUser.getAccessKey();
    String secretKey = loginUser.getSecretKey();
    LuoApiClient tempLuoApiClient = new LuoApiClient(accessKey, secretKey);
    Gson gson=new Gson();
    com.luoying.model.User user = gson.fromJson(userRequestParams, com.luoying.model.User.class);
    String result = tempLuoApiClient.getUsernameByPost(user);
    return ResultUtils.success(result);
}
```

## 前端

### 基础功能

#### 前端配置代码美化插件

1. 配置代码规范美化插件，方便快速格式化 

![image-20231005152458917](assets/image-20231005152458917.png)

![image-20231005152731434](assets/image-20231005152731434.png)

#### 自动生成后端请求代码

使用前端Ant Design Pro框架集成的openapi插件生成向后端发请求的代码（该插件依赖后端遵循openapi规范的swagger文档）

![image-20231005154711349](assets/image-20231005154711349.png)

![image-20231005154630774](assets/image-20231005154630774.png)

![image-20231005154846218](assets/image-20231005154846218.png)

![image-20231005154913394](assets/image-20231005154913394.png)

![image-20231005155146284](assets/image-20231005155146284.png)

![image-20231005155449387](assets/image-20231005155449387.png)

#### 修改requestErrorConfig

![image-20231005160127336](assets/image-20231005160127336.png)

![image-20231005160035130](assets/image-20231005160035130.png)

![image-20231005170916555](assets/image-20231005170916555.png)

![image-20231007102657764](assets/image-20231007102657764.png)

![image-20231007102743482](assets/image-20231007102743482.png)

![image-20231005160150723](assets/image-20231005160150723.png)

![image-20231005160213602](assets/image-20231005160213602.png)

#### 测试登录接口

![image-20231005195102966](assets/image-20231005195102966.png)

![image-20231005195728380](assets/image-20231005195728380.png)

![image-20231006143702071](assets/image-20231006143702071.png)

#### 管理员查看接口

![image-20231006155730790](assets/image-20231006155730790.png)

![image-20231006160044122](assets/image-20231006160044122.png)

![image-20231007110433560](assets/image-20231007110433560.png)

![image-20231007110639157](assets/image-20231007110639157.png)

#### 修改路由

![image-20231007101716582](assets/image-20231007101716582.png)

![image-20231007101808749](assets/image-20231007101808749.png)

#### 整体页面风格设置

![image-20231007101912967](assets/image-20231007101912967.png)

![image-20231007102016064](assets/image-20231007102016064.png)

![image-20231007102131397](assets/image-20231007102131397.png)

![image-20231007102254060](assets/image-20231007102254060.png)

#### 管理员创建接口

![image-20231007103618859](assets/image-20231007103618859.png)

![image-20231007103807805](assets/image-20231007103807805.png)

![image-20231007103923663](assets/image-20231007103923663.png)

#### 管理员更新接口

![image-20231007140905528](assets/image-20231007140905528.png)

![image-20231007141214214](assets/image-20231007141214214.png)

![image-20231007141346745](assets/image-20231007141346745.png)

![image-20231007141521459](assets/image-20231007141521459.png)

![image-20231007141636710](assets/image-20231007141636710.png)

#### 管理员删除接口

![image-20231007143951112](assets/image-20231007143951112.png)

![image-20231007144114294](assets/image-20231007144114294.png)

### 接口调用

![image-20231010163740474](assets/image-20231010163740474.png)

![image-20231010163921582](assets/image-20231010163921582.png)

![image-20231010162423718](assets/image-20231010162423718.png)

#### 管理员发布/下线接口

![image-20231010162308263](assets/image-20231010162308263.png)

![image-20231010162447354](assets/image-20231010162447354.png)

![image-20231010162511988](assets/image-20231010162511988.png)

#### 用户浏览接口

![image-20231010164942684](assets/image-20231010164942684.png)

![image-20231010173927929](assets/image-20231010173927929.png)

![image-20231010173616965](assets/image-20231010173616965.png)

![image-20231010173706622](assets/image-20231010173706622.png)

![image-20231010173800877](assets/image-20231010173800877.png)

![image-20231010174228726](assets/image-20231010174228726.png)

![image-20231010174317690](assets/image-20231010174317690.png)

![image-20231010174409515](assets/image-20231010174409515.png)

![image-20231010174454389](assets/image-20231010174454389.png)

![image-20231010174523161](assets/image-20231010174523161.png)

#### 用户查看接口文档

![image-20231010193204277](assets/image-20231010193204277.png)

![image-20231010193751738](assets/image-20231010193751738.png)

![image-20231010194808697](assets/image-20231010194808697.png)

![image-20231010200258552](assets/image-20231010200258552.png)

![image-20231010201440599](assets/image-20231010201440599.png)

![image-20231010201632255](assets/image-20231010201632255.png)

![image-20231010203520963](assets/image-20231010203520963.png)

![image-20231010203550258](assets/image-20231010203550258.png)

![image-20231010205218800](assets/image-20231010205218800.png)

#### 在线调用

![image-20231011115959514](assets/image-20231011115959514.png)



![image-20231011144215449](assets/image-20231011144215449.png)

![image-20231011144257324](assets/image-20231011144257324.png)

![image-20231011144539053](assets/image-20231011144539053.png)

![image-20231011144500559](assets/image-20231011144500559.png)

## todo

1. 判断该接口是否可以调用，固定方法名改为根据测试地址来调用

2. 用户调用接口，固定方法名改为根据测试地址来调用

3. 模拟接口改为从数据库校验ak和sk

4. 用户可以申请更换签名

5. 把sdk上传到maven仓库

   
