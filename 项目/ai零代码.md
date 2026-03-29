

# 后端初始化

## 初始化

springweb

lombok

mysqldriver



把apprication.properited 改成yml格式

然后加入

```yml
spring:
  application:
    name: zds-ai  项目名字

server:
  port: 2778
  servlet:
    context-path: /api  后端访问的路径吧
```



### 依赖整合

hutool knife4j

```xml

        <!-- hutool工具类-->
        <dependency>
            <groupId>cn.hutool</groupId>
            <artifactId>hutool-all</artifactId>
            <version>5.8.38</version>
        </dependency>
       <!--   knife4j接口文档-->    -->
        <dependency>
            <groupId>com.github.xiaoymin</groupId>
            <artifactId>knife4j-openapi3-jakarta-spring-boot-starter</artifactId>
            <version>4.4.0</version>
        </dependency>
```





 改xml配置

```yml
# springdoc-openapi
springdoc:
  group-configs:
    - group: 'default'
      packages-to-scan: com.zds.zdsai.controller
# knife4j
knife4j:
  enable: true
  setting:
    language: zh_cn

```





### 写模板代码 自定义异常

已经在包中 用就可以了





### 全局跨域配置（样板代码）

直接在config包中 样板代码

```java
@Configuration
public class CorsConfig implements WebMvcConfigurer {

    @Override
    public void addCorsMappings(CorsRegistry registry) {
        // 覆盖所有请求
        registry.addMapping("/**")
                // 允许发送 Cookie
                .allowCredentials(true)
                // 放行哪些域名（必须用 patterns，否则 * 会和 allowCredentials 冲突）
                .allowedOriginPatterns("*")
                .allowedMethods("GET", "POST", "PUT", "DELETE", "OPTIONS")
                .allowedHeaders("*")
                .exposedHeaders("*");
    }
}

```





# 前端初始化

emm

使用nvm 来管理node.js



安装ant design

```shell
npm i --save ant-design-vue@4.x
```

全局装配

```typescript
import './assets/main.css'

import { createApp } from 'vue'
import { createPinia } from 'pinia'

import App from './App.vue'
import router from './router'

import Antd from 'ant-design-vue'
import 'ant-design-vue/dist/reset.css'

const app = createApp(App)

app.use(createPinia())
app.use(router)
app.use(Antd)

app.mount('#app')

```



安装

axios

```
npm install axios
```



```ts
import axios from 'axios'
import { message } from 'ant-design-vue'

// 创建 Axios 实例
const myAxios = axios.create({
  baseURL: 'http://localhost:8123/api',
  timeout: 60000,
  withCredentials: true,
})

// 全局请求拦截器
myAxios.interceptors.request.use(
  function (config) {
    // Do something before request is sent
    return config
  },
  function (error) {
    // Do something with request error
    return Promise.reject(error)
  },
)

// 全局响应拦截器
myAxios.interceptors.response.use(
  function (response) {
    const { data } = response
    // 未登录
    if (data.code === 40100) {
      // 不是获取用户信息的请求，并且用户目前不是已经在用户登录页面，则跳转到登录页面
      if (
        !response.request.responseURL.includes('user/get/login') &&
        !window.location.pathname.includes('/user/login')
      ) {
        message.warning('请先登录')
        window.location.href = `/user/login?redirect=${window.location.href}`
      }
    }
    return response
  },
  function (error) {
    // Any status codes that falls outside the range of 2xx cause this function to trigger
    // Do something with response error
    return Promise.reject(error)
  },
)

export default myAxios

```





## openapi 前端代码生成器

[@umijs/openapi - npm](https://www.npmjs.com/package/@umijs/openapi) 官网自己去学把

```
npm i --save-dev @umijs/openapi

npm i --save-dev tslib
```

```
export default {
  requestLibPath: "import request from '@/request'",
  schemaPath: 'http://localhost:2778/api/v3/api-docs',
  serversPath: './src',
}

```



# 用户登录

<img src="https://pic.code-nav.cn/course_picture/1608440217629360130/ahLctL73Pp9LatVp.svg" alt="img" style="zoom: 80%;" />





接口权限

1.未登录也可以使用
2，登录用户才能使用
3，未登录也可以使用，但是登录用户能进行更多操作（比如登录后查看全文）
4，仅管理员才能使用





### mybatis flex

生成器 不用学看官方就行 

然后就是说 这个

entity是数据库与实体类的连接





### @MapperScan

可能扫描到mapper包 



### Dto就是业务代码互相传递的类



Vo是脱敏的类



### 用户登录接口

```java
 @Override
    public LoginUserVO userLogin(String userAccount, String userPassword, HttpServletRequest request) {
        //校验参数
        if (StringUtils.isAnyBlank(userAccount, userPassword)) {
            throw new BusinessException(ErrorCode.PARAMS_ERROR, "参数不能为空");
        }
        if (userAccount.length() < 4) {
            throw new BusinessException(ErrorCode.PARAMS_ERROR, "用户账号过知短");
        }
        if (userPassword.length() < 8) {
            throw new BusinessException(ErrorCode.PARAMS_ERROR, "用户密码过知短");
        }

        //加密密码
        String encryptPassword = getEncryptPassword(userPassword);

        //查询用户

        QueryWrapper queryWrapper = new QueryWrapper();
        queryWrapper.eq("userAccount", userAccount);
        queryWrapper.eq("userPassword", encryptPassword);
        User user = this.mapper.selectOneByQuery(queryWrapper);
        ThrowUtils.throwIf(user == null, ErrorCode.PARAMS_ERROR, "用户不存在或密码错误");
        //记录用户的登录态
        
        request.getSession().setAttribute(USER_LOGIN_STATE, user);
        return this.getLoginUserVO(user);
    }
```

**request.getSession().setAttribute(USER_LOGIN_STATE, user);** 很重要的 种session





## 注解（重要）

### 0先引入依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>

```



### 1在annotation包下 创建注解类

```java
@Target(ElementType.METHOD) //在方法上的注解
@Retention(RetentionPolicy.RUNTIME) //注解的策略
public @interface AuthCheck {

    /**  
     * 必须有某个角色
     */
    String mustRole() default "";
}

```

### 2 创建一个切面 在aop包里面



```java
package com.zds.zdsai.aop;

import com.zds.zdsai.annotation.AuthCheck;
import com.zds.zdsai.exception.BusinessException;
import com.zds.zdsai.exception.ErrorCode;
import com.zds.zdsai.model.entity.User;
import com.zds.zdsai.model.enums.UserRoleEnum;
import com.zds.zdsai.service.UserService;
import jakarta.annotation.Resource;
import jakarta.servlet.http.HttpServletRequest;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.springframework.stereotype.Component;
import org.springframework.web.context.request.RequestAttributes;
import org.springframework.web.context.request.RequestContextHolder;
import org.springframework.web.context.request.ServletRequestAttributes;

@Aspect
@Component
public class AuthInterceptor {

    @Resource
    private UserService userService;

    /**
     * 执行拦截
     *
     * @param joinPoint 切入点
     * @param authCheck 权限校验注解
     */
    //切点 -->你在什么时候拦截这个方法
    @Around("@annotation(authCheck)")
    public Object doInterceptor(ProceedingJoinPoint joinPoint, AuthCheck authCheck) throws Throwable {
        String mustRole = authCheck.mustRole();
        RequestAttributes requestAttributes = RequestContextHolder.currentRequestAttributes();
        HttpServletRequest request = ((ServletRequestAttributes) requestAttributes).getRequest();
        // 当前登录用户
        User loginUser = userService.getLoginUser(request);
        UserRoleEnum mustRoleEnum = UserRoleEnum.getEnumByValue(mustRole);
        // 不需要权限，放行
        if (mustRoleEnum == null) {
            return joinPoint.proceed();
        }
        // 以下为：必须有该权限才通过
        // 获取当前用户具有的权限
        UserRoleEnum userRoleEnum = UserRoleEnum.getEnumByValue(loginUser.getUserRole());
        // 没有权限，拒绝
        if (userRoleEnum == null) {
            throw new BusinessException(ErrorCode.NO_AUTH_ERROR);
        }
        // 要求必须有管理员权限，但用户没有管理员权限，拒绝
        if (UserRoleEnum.ADMIN.equals(mustRoleEnum) && !UserRoleEnum.ADMIN.equals(userRoleEnum)) {
            throw new BusinessException(ErrorCode.NO_AUTH_ERROR);
        }
        // 通过权限校验，放行
        return joinPoint.proceed();
    }
}

```





**@Aspect**来定义这是一个切面

**@annotation(authcheck)** --》就是通知  只有在有authcheck注解的 才能进行权限校验

**joinPoint.proceed()**; 放行





## 前端id与后端id传值不一致

因为后端的int值太tmd大 而前端的js的不支持那么大的 所以会少那么一些

加个样板代码

```java
/**
 * Spring MVC Json 配置
 */
@JsonComponent
public class JsonConfig {

    /**
     * 添加 Long 转 json 精度丢失的配置
     */
    @Bean	
    public ObjectMapper jacksonObjectMapper(Jackson2ObjectMapperBuilder builder) {
        ObjectMapper objectMapper = builder.createXmlMapper(false).build();
        SimpleModule module = new SimpleModule();
        module.addSerializer(Long.class, ToStringSerializer.instance);
        module.addSerializer(Long.TYPE, ToStringSerializer.instance);
        objectMapper.registerModule(module);
        return objectMapper;
    }
}

```





## 前端

前端想要获取用户的登陆数据 是不是每次刷新页面 就要写一个fetch函数来获取呢？

不要 我们直接写一个引入pinna

> pinna

```ts
import { useLoginUserStore } from '@/stores/loginUser.ts'

const loginUserStore = useLoginUserStore()
loginUserStore.fetchLoginUser()

```

获取用户





# Ai大模型接入

## langchain4j

> 引入依赖

```xml
<dependency>
    <groupId>dev.langchain4j</groupId>
    <artifactId>langchain4j</artifactId>
    <version>1.1.0</version>
</dependency>
<dependency>
    <groupId>dev.langchain4j</groupId>
    <artifactId>langchain4j-open-ai-spring-boot-starter</artifactId>
    <version>1.1.0-beta7</version>
</dependency>

```



> application.xml

```yml
# AI
langchain4j:
  open-ai:
    chat-model:
      base-url: https://api.deepseek.com
      api-key: <Your API Key>
      model-name: deepseek-chat
      log-requests: true
      log-responses: true

```



### 创建接口AiCodeGeneratorService

```java
package com.zds.zdsai.ai;

import dev.langchain4j.service.SystemMessage;

/**
 * AI代码生成服务
 */
public interface AiCodeGeneratorService {

    /**
     * 生成HTML代码
     *
     * @param userMessage 用户输入
     * @return 代码
     */
    @SystemMessage(fromResource = "prompt/codegen-html-system-prompt.txt")
    String generateCode(String userMessage);

    /**
     * 生成多文件代码
     *
     * @param userMessage 用户输入
     * @return 代码
     */
    @SystemMessage(fromResource = "prompt/codegen-multi-file-system-prompt.txt")
    String generateMultiFileCode(String userMessage);
}

```

写提示词 当然 如果词的长度少 你可以不用建立txt



然后就是 写一个代理工厂 来创建实例

```java
package com.zds.zdsai.ai;/*
 *@auther 郑笃实
 *@version 1.0
 *
 */

import dev.langchain4j.model.chat.ChatModel;
import dev.langchain4j.service.AiServices;
import jakarta.annotation.Resource;
import org.springframework.context.annotation.Bean;

//Ai服务创建工厂
public class AiCodeGeneratorServiceFactory {

    //指定使用的大模型
    @Resource
    private ChatModel chatModel;

    /**
     * 创建AI 代码生成器代码服务
     * @return
     */
    @Bean
    public AiCodeGeneratorService createAiCodeGeneratorService() {
        return AiServices.create(AiCodeGeneratorService.class, chatModel);
    }
}

```





是这样的 所以及就获得了一个String的 返回结果 

但是问题来了你该如何将String给用户看？？直接看？？也行 或许

我们可以将String转化为JSON  让ai按照格式生成相应key 这样我们就可以 更好的展示相关的内容了

 

```java
package com.zds.zdsai.ai.model;

import lombok.Data;
//Html代码结果
@Data
public class HtmlCodeResult {

    /**
     * Html代码
     */
    private String htmlCode;

    //描述
    private String description;
}

```

![image-20251111194720859](F:\记录\项目\image-20251111194720859-1762861641318-1.png)

然后改一下接口的返回值 这样就好了

- method: POST
- url: https://api.deepseek.com/chat/completions
- headers: [Authorization: Beare...c6], [User-Agent: langchain4j-openai], [Content-Type: application/json]
- body: {
  "model" : "deepseek-chat",
  "messages" : [ {
    "role" : "system",
    "content" : "你是一位资深的 Web 前端开发专家，精通 HTML、CSS 和原生 JavaScript。你擅长构建响应式、美观且代码整洁的单页面网站。\r\n\r\n你的任务是根据用户提供的网站描述，生成一个完整、独立的单页面网站。你需要一步步思考，并最终将所有代码整合到一个 HTML 文件中。\r\n\r\n约束:\r\n1. 技术栈: 只能使用 HTML、CSS 和原生 JavaScript。\r\n2. 禁止外部依赖: 绝对不允许使用任何外部 CSS 框架、JS 库或字体库。所有功能必须用原生代码实现。\r\n3. 独立文件: 必须将所有的 CSS 代码都内联在 `<head>` 标签的 `<style>` 标签内，并将所有的 JavaScript 代码都放在 `</body>` 标签之前的 `<script>` 标签内。最终只输出一个 `.html` 文件，不包含任何外部文件引用。\r\n4. 响应式设计: 网站必须是响应式的，能够在桌面和移动设备上良好显示。请优先使用 Flexbox 或 Grid 进行布局。\r\n5. 内容填充: 如果用户描述中缺少具体文本或图片，请使用有意义的占位符。例如，文本可以使用 Lorem Ipsum，图片可以使用 https://picsum.photos 的服务 (例如 `<img src=\"https://picsum.photos/800/600\" alt=\"Placeholder Image\">`)。\r\n6. 代码质量: 代码必须结构清晰、有适当的注释，易于阅读和维护。\r\n7. 交互性: 如果用户描述了交互功能 (如 Tab 切换、图片轮播、表单提交提示等)，请使用原生 JavaScript 来实现。\r\n8. 安全性: 不要包含任何服务器端代码或逻辑。所有功能都是纯客户端的。\r\n9. 输出格式: 你的最终输出必须包含 HTML 代码块，可以在代码块之外添加解释、标题或总**结性文字。格式如下：\r\n\r\n```html\r\n... HTML 代码 ...\r\n\r\n"
  }, {
    "role" : "user",
    "content" : "做个程序员鱼皮的工作记录小工具\nYou must answer strictly in the following JSON format: {\n\"htmlCode\": (type: string),\n\"description\": (type: string)\n}"
  } ],**
  "stream" : false
  }





> 但是 啊 但是 如果ai生成的文字太大 以至于 Stirng类型装不下 或者ai不听你的 json不按照格式来 怎么办呢？？、

1. 设置maxtoken

2.严格限制 

```
langchain4j:
  open-ai:
    chat-model:
      strict-json-schema: true
      response-format: json_object

```

3. 加上注释 Description("HTML代码")

   这样 我们发送这个model的时候 就可以带上描述 这样ai可以更好的理解



## 设计模式 门面模式

先建立枚举类 **CodeGenTypeEnum**

然后再core包下面写一个文件存储类FileSaver 就是将代码存到文件中

![image-20251111205105080](F:\记录\项目\image-20251111205105080.png)



先写出来了 门面   门面调用aiCodeGeneratorService 和 Filesave 

**AiCodeGeneratorFacade**

```java
package com.zds.zdsai.core;

import com.zds.zdsai.ai.AiCodeGeneratorService;
import com.zds.zdsai.ai.model.HtmlCodeResult;
import com.zds.zdsai.ai.model.MultiFileCodeResult;
import com.zds.zdsai.exception.BusinessException;
import com.zds.zdsai.exception.ErrorCode;
import com.zds.zdsai.model.enums.CodeGenTypeEnum;
import jakarta.annotation.Resource;
import org.springframework.stereotype.Service;

import java.io.File;

/**
 * AI 代码生成外观类，组合生成和保存功能
 */
@Service
public class AiCodeGeneratorFacade {

    @Resource
    private AiCodeGeneratorService aiCodeGeneratorService;

    /**
     * 统一入口：根据类型生成并保存代码
     *
     * @param userMessage     用户提示词
     * @param codeGenTypeEnum 生成类型
     * @return 保存的目录
     */
    public File generateAndSaveCode(String userMessage, CodeGenTypeEnum codeGenTypeEnum) {
        if (codeGenTypeEnum == null) {
            throw new BusinessException(ErrorCode.SYSTEM_ERROR, "生成类型为空");
        }
        return switch (codeGenTypeEnum) {
            case HTML -> generateAndSaveHtmlCode(userMessage);
            case MULTI_FILE -> generateAndSaveMultiFileCode(userMessage);
            default -> {
                String errorMessage = "不支持的生成类型：" + codeGenTypeEnum.getValue();
                throw new BusinessException(ErrorCode.SYSTEM_ERROR, errorMessage);
            }
        };
    }

    /**
     * 生成 HTML 模式的代码并保存
     *
     * @param userMessage 用户提示词
     * @return 保存的目录
     */
    private File generateAndSaveHtmlCode(String userMessage) {
        HtmlCodeResult result = aiCodeGeneratorService.generateHtmlCode(userMessage);
        return CodeFileSaver.saveHtmlCodeResult(result);
    }

    /**
     * 生成多文件模式的代码并保存
     *
     * @param userMessage 用户提示词
     * @return 保存的目录
     */
    private File generateAndSaveMultiFileCode(String userMessage) {
        MultiFileCodeResult result = aiCodeGeneratorService.generateMultiFileCode(userMessage);
        return CodeFileSaver.saveMultiFileCodeResult(result);
    }
}

```





## SSE结构化流式输出

等待全部返回太麻烦了 于是我们选择 SSE结构化流式输出

> langchain4j +Reactor 

![image-20251111212329732](F:\记录\项目\image-20251111212329732.png)



首先我们应该 修改工厂 符合流式

```java
package com.zds.zdsai.ai;/*
 *@auther 郑笃实
 *@version 1.0
 *
 */

import dev.langchain4j.model.chat.ChatModel;
import dev.langchain4j.model.chat.StreamingChatModel;
import dev.langchain4j.service.AiServices;
import jakarta.annotation.Resource;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

//Ai服务创建工厂
@Configuration
public class AiCodeGeneratorServiceFactory {

    //指定使用的大模型
    @Resource
    private ChatModel chatModel;

    @Resource
    private StreamingChatModel streamingChatModel;

    /**
     * 创建AI 代码生成器代码服务
     * @return
     */
    @Bean
    public AiCodeGeneratorService createAiCodeGeneratorService() {
        return AiServices.builder(AiCodeGeneratorService.class)
                .chatModel(chatModel)//哪个chatmodel
                .streamingChatModel(streamingChatModel)//哪个streamingChatModel
                .build();

    }
}

```

然后再在**AiCodeGeneratorService** 中 添加两个适合流式的代码

```java

    /**
     * 生成HTML代码
     *
     * @param userMessage 用户输入
     * @return 代码
     */
    @SystemMessage(fromResource = "prompt/codegen-html-system-prompt.txt")
    Flux<String> generateHtmlCodeStream(String userMessage);

    /**
     * 生成多文件代码
     *
     * @param userMessage 用户输入
     * @return 代码
     */
    @SystemMessage(fromResource = "prompt/codegen-multi-file-system-prompt.txt")
    Flux<String> generateMultiFileCodeStream(String userMessage);
```

