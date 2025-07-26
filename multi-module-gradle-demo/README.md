# multi-module-gradle-demo

下面是一个多模块Spring Boot项目的搭建指南，包含父项目和三个子模块（common、service、web），使用JDK 17和Gradle构建。我将逐步说明如何在IDEA中创建：

### 项目结构
```
multi-module-demo (父项目)
├── common (公共模块)
├── service (业务逻辑模块)
├── web (Web接口模块)
└── settings.gradle
```

### 创建步骤

#### 1. 创建父项目（Gradle项目）
1. 打开IDEA → New Project
2. 选择 **Gradle** → 勾选 **Java** → 选择JDK 17
3. 输入项目名：`multi-module-demo`
4. 取消勾选 "Create Git repository"（可选）
5. 完成创建

#### 2. 配置父项目的`settings.gradle`
```groovy
// settings.gradle
rootProject.name = 'multi-module-demo'
include 'common', 'service', 'web'
```

#### 3. 配置父项目的`build.gradle`
```groovy
// build.gradle
plugins {
    id 'java'
    id 'org.springframework.boot' version '3.2.0' apply false // 不在父项目应用
    id 'io.spring.dependency-management' version '1.1.4'
}

allprojects {
    group = 'com.example'
    version = '0.0.1-SNAPSHOT'
    sourceCompatibility = '17'
    
    repositories {
        mavenCentral()
    }
}

subprojects {
    apply plugin: 'java'
    apply plugin: 'io.spring.dependency-management'
    
    dependencyManagement {
        imports {
            mavenBom org.springframework.boot.gradle.plugin.SpringBootPlugin.BOM_COORDINATES
        }
    }
    
    dependencies {
        implementation 'org.springframework.boot:spring-boot-starter'
        testImplementation 'org.springframework.boot:spring-boot-starter-test'
    }
    
    test {
        useJUnitPlatform()
    }
}
```

#### 4. 创建子模块
在项目根目录右键：
- **New → Module**
- 选择 **Gradle** → 勾选 **Java**
- 模块名：`common` → 完成
- 重复创建 `service` 和 `web` 模块

#### 5. 配置子模块依赖

**common模块 (`common/build.gradle`):**
```groovy
dependencies {
    // 公共工具类依赖
    implementation 'org.apache.commons:commons-lang3:3.12.0'
}
```

**service模块 (`service/build.gradle`):**
```groovy
dependencies {
    implementation project(':common') // 依赖common模块
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
}
```

**web模块 (`web/build.gradle`):**
```groovy
dependencies {
    implementation project(':service') // 依赖service模块
    implementation project(':common')  // 依赖common模块
    implementation 'org.springframework.boot:spring-boot-starter-web'
}
```

#### 6. 创建示例代码

**common模块:**
```java
// src/main/java/com/example/common/CommonUtils.java
package com.example.common;

public class CommonUtils {
    public static String greet() {
        return "Hello from Common Module!";
    }
}
```

**service模块:**
```java
// src/main/java/com/example/service/UserService.java
package com.example.service;

import com.example.common.CommonUtils;
import org.springframework.stereotype.Service;

@Service
public class UserService {
    public String getUserInfo() {
        return CommonUtils.greet() + "\nUser: John Doe (from Service Module)";
    }
}
```

**web模块:**
```java
// src/main/java/com/example/web/WebApplication.java
package com.example.web;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class WebApplication {
    public static void main(String[] args) {
        SpringApplication.run(WebApplication.class, args);
    }
}

// src/main/java/com/example/web/controller/UserController.java
package com.example.web.controller;

import com.example.service.UserService;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class UserController {
    private final UserService userService;

    public UserController(UserService userService) {
        this.userService = userService;
    }

    @GetMapping("/user")
    public String getUser() {
        return userService.getUserInfo();
    }
}
```

### 7. 运行项目
1. 打开 `web/src/main/java/com/example/web/WebApplication.java`
2. 右键 → `Run 'WebApplication.main()'`
3. 访问 http://localhost:8080/user
4. 应看到输出：
```
Hello from Common Module!
User: John Doe (from Service Module)
```

### 关键点说明
1. **模块依赖**：
    - web → service → common
    - 使用`implementation project(':module')`声明依赖

2. **Spring Boot配置**：
    - 父项目管理插件版本（apply false）
    - 子项目通过`dependencyManagement`统一管理依赖版本

3. **项目结构验证**：
    - 执行 `gradle build` 查看构建是否成功
    - 在IDEA右侧Gradle面板查看任务树

4. **热部署支持**：
   在`web/build.gradle`添加：
   ```groovy
   bootRun {
       sourceResources sourceSets.main
   }
   ```

### 常见问题解决
1. **依赖找不到**：
    - 刷新Gradle：IDEA右侧Gradle面板 → 🔄按钮
    - 检查模块名称拼写

2. **主类找不到**：
    - 确保只在web模块有`@SpringBootApplication`
    - 清理重建：`gradle clean build`

3. **JDK版本问题**：
   在`gradle.properties`添加：
   ```
   org.gradle.java.home=/path/to/jdk17
   ```

这个结构提供了清晰的模块分离：
- common：通用工具/常量
- service：业务逻辑/数据访问
- web：REST接口/控制器

你可以根据需要扩展更多模块（如`domain`、`security`等）