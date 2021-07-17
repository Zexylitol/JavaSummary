# Alibaba Java Coding Guidelines

- Alibaba代码规约插件

# Lombok

>  A plugin that adds first-class support for Project Lombok

Lombok 是一种 Java 实用工具，可用来帮助开发人员消除 Java 的冗长，尤其是对于简单的 Java 对象（POJO）。它通过注解实现这一目的。

lombok的主要作用是通过注解，消除一些必须有但显得很臃肿的 Java 代码（一个对象中的getter/setter方法、构造器方法、equals方法、toString方法等，这些方法很冗长且没有技术含量），虽然IDEA里面都自带自动生成这些方法的功能，但是使用lombok会代码看起来更加简洁，写起来也更加方便。

```java
@Getter and @Setter
@FieldNameConstants
@ToString
@EqualsAndHashCode
@AllArgsConstructor, @RequiredArgsConstructor and @NoArgsConstructor
@Log, @Log4j, @Log4j2, @Slf4j, @XSlf4j, @CommonsLog, @JBossLog, @Flogger, @CustomLog
@Data
@Builder
@SuperBuilder
@Singular
@Delegate
@Value
@Accessors
@Wither
@With
@SneakyThrows
@val
@var
experimental @var
@UtilityClass
@ExtensionMethod (Experimental, activate manually in plugin settings)
```

# SonarLint

SonarLint is a free IDE extension to find and fix bugs, vulnerabilities and code smells as you write code! Like a spell checker, SonarLint highlights issues on the fly, with clear remediation guidance so you can fix them before the code is even committed. With support for several popular and classic languages, SonarLint helps developers of all experience and skill levels write efficient, safe code.

SonarLint integrates with most JetBrains IDEs including IntelliJ IDEA, CLion, WebStorm, PHPStorm, PyCharm, Android Studio & RubyMine. Supported languages include C, C++, Java, JavaScript, TypeScript, Python, Kotlin, Ruby, HTML & PHP.

SonarLint isn't just about your code, it's also an opportunity to bring your passion for quality code to the whole team. When paired with SonarQube or SonarCloud, your team can share common language rulesets, project analysis settings and more. The combination forms a continuous analysis solution that keeps code quality and security issues out of your branches.

SonarLint requires your IDE to be run with a JVM 8+ (this is the case for all recent JetBrains IDE); the analysis of JavaScript and TypeScript requires Node.js >= 10.12 to be installed on your computer.

SonarLint是一个代码质量检测插件，可以帮助我们检测出代码中的**坏味道**（潜在的空指针异常、循环嵌套等），对于每一个问题，SonarLint都给出了示例，还有相应的解决方案，极大的方便了开发

有了代码规范与质量检测工具以后，很多东西就可以**量化**了，比如bug率、代码重复率等，还可以自定义各种指标，方便管理人员查看，SonarQube是一个开源的代码质量管理平台，记录每次检测分析的结果，这样就可以进行分析和统计，并且可以直观的看到这一切

# Free MyBatis plugin

Free-idea-mybatis是一款增强idea对mybatis支持的插件，主要功能如下： 

- 生成mapper xml文件
- 快速从代码跳转到mapper及从mapper返回代码
- mybatis自动补全及语法错误提示
- 集成mybatis generator gui界面
- 根据数据库注解，生成swagger model注解

# Jclasslib Bytecode Viewer

GitHub 地址：https://github.com/ingokegel/jclasslib

**jclasslib bytecode viewer is a tool that visualizes all aspects of compiled Java class files and the contained bytecode.**

jclasslib bytecode viewer 是一个可以可视化已编译Java类文件和所包含的字节码的工具。 另外，它还提供一个库，可以让开发人员读写Java类文件和字节码。

类似`javap`指令

# Rainbow Brackets

彩虹颜色的括号，看着很舒服，敲代码效率变高

<center><img src="https://ss.im5i.com/2021/07/17/gmGfW.png" alt="gmGfW.png" border="0" /></center>

# Translation

翻译插件

<center><img src="https://ss.im5i.com/2021/07/17/gmK2G.png" alt="gmK2G.png" border="0" /></center>