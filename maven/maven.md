#### maven仓库

![maven仓库关系](E:\wdq\note\maven\maven仓库关系.jpg)

**本地仓库**

存放在本地的包，相当于本地缓存，如果本地仓库中没有匹配的包，就查找私服，如果私服中没有就去中央仓库查找。

**私服**

公司内部使用的仓库，存放公司内部的jar包。如果员工A新建了一个jar包B而你本地没有，这时候只需要员工A将jar包上传到私服公司的其他员工就可以下载使用了

**中央仓库**

该仓库存储了互联网上的jar，由Maven团队来维护，地址是：http://repo1.maven.org/maven2/



#### maven依赖冲突

##### 依赖传递（transitive）

如果A依赖B，B依赖C，那么引入A，意味着B和C都会被引入。

##### Maven的最近依赖策略

如果一个项目依赖相同的groupId、artifactId的多个版本，那么在**依赖树（mvn dependency:tree）中离项目最近的那个版本将会被使用**。（从这里可以看出Maven是不是有点小问题呢？能不能选择高版本的进行依赖么？据了解，Gradle就是version+策略）

##### 解决依赖冲突

1、声明子模块中的版本

使用<dependencyManagement>  [这种主要用于子模块的版本一致性中]、

2、去掉不想要的依赖

使用<exclusions> [在实际中我们可以在IDEA中直接利用插件帮助我们生成]

3、最近依赖策略

直接使用显式依赖指定版本，那不就是最靠近项目的么？

使用<dependency>

#### maven规范化目录结构

![maven标准目录结构](E:\wdq\note\maven\maven标准目录结构.jpg)



#### maven生命周期

![1573625795215](C:\Users\jiachao\AppData\Roaming\Typora\typora-user-images\1573625795215.png)

1. **执行后面的命令前面的命令也会得到执行**
2. **package：打成Jar or War包，会自动进行clean+compile**
3. **install：将包上传到本地仓库**
4. **deploy：将包上传到私服**



#### 依赖作用域

Dependency Scope

依赖范围用于限制依赖项的可传递性，并影响用于各种生成任务的类路径

- **compile**

  默认作用域

- **provided**

- **runtime**

- **test**

- **system**

- **import**