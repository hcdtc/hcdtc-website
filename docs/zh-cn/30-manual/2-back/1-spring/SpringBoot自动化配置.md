# 探究@SpringBootApplication注解

通过查看源码可知，它是一个复合注解，主要由如下三个注解组成：

1. @SpringBootConfiguration：复合注解，由@Configuration组成，任何一个标注了@Configuration的Java类定义，都是一个JavaConfig配置类。

2. @EnableAutoConfiguration：与其他@Enable开头的注解(例如，@EnableScheduling、@EnableCaching、@EnableMBeanExport等)功能一样，借助@Import的支持，收集和注册特定场景相关的Bean定义。例如：
@EnableScheduling是通过@Import将Spring调度框架相关的Bean定义都加载到Spring IoC容器。
@EnableMBeanExport是通过@Import将JMX相关的Bean定义加载到Spring IoC容器。
`而@EnableAutoConfiguration则是借助@Import的帮助，将所有符合自动配置条件的Bean定义加载到Spring IoC容器`。
```
重点解析：从classpath中搜寻所有META-INF/spring.factories配置文件，并将其中org.spring.framework.boot.autoconfigure.EnableAutoConfiguration对应的配置项通过反射（Java Reflection）机制实例化为对应的标注了@Configuration的JavaConfig形式的IoC容器配置类，然后汇总为一个并加载到IoC容器。
```

3. @ComponentScan

```java
package org.springframework.boot.autoconfigure;

import ...;
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(
    excludeFilters = {        @Filter(
            type = FilterType.CUSTOM,
            classes = {TypeExcludeFilter.class}
        ),         @Filter(
            type = FilterType.CUSTOM,
            classes = {AutoConfigurationExcludeFilter.class}
        )}
)
public @interface SpringBootApplication {
    @AliasFor(
        annotation = EnableAutoConfiguration.class,
        attribute = "exclude"
    )
    Class<?>[] exclude() default {};

    @AliasFor(
        annotation = EnableAutoConfiguration.class,
        attribute = "excludeName"
    )
    String[] excludeName() default {};

    @AliasFor(
        annotation = ComponentScan.class,
        attribute = "basePackages"
    )
    String[] scanBasePackages() default {};

    @AliasFor(
        annotation = ComponentScan.class,
        attribute = "basePackageClasses"
    )
    Class<?>[] scanBasePackageClasses() default {};
}
```


## 探究@EnableAutoConfiguration注解

@EnableAutoConfiguration注解追本溯源
@EnableAutoConfiguration -> EnableAutoConfigurationImportSelector（@Import注解） -> AutoConfigurationImportSelector（父类加载配置类时，指定getSpringFactoriesLoaderFactoryClass为EnableAutoConfiguration.class-也即@EnableAutoConfiguration ，并调用SpringFactoriesLoader.loadFactoryNames方法加载配置文件） --> SpringFactoriesLoader（加载"META-INF/spring.factories"中属性Key为EnableAutoConfiguration.class完整类名的配置类定义） -> "META-INF/spring.factories"；

1、查看复合注解定义
```java
package org.springframework.boot.autoconfigure;

import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Inherited;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;
import org.springframework.boot.autoconfigure.AutoConfigurationPackage;
import org.springframework.boot.autoconfigure.EnableAutoConfigurationImportSelector;
import org.springframework.context.annotation.Import;

@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import({EnableAutoConfigurationImportSelector.class})
public @interface EnableAutoConfiguration {
    String ENABLED_OVERRIDE_PROPERTY = "spring.boot.enableautoconfiguration";

    Class<?>[] exclude() default {};

    String[] excludeName() default {};
}
```
2、重点关注下Import注解
```java
package org.springframework.boot.autoconfigure;

import org.springframework.boot.autoconfigure.AutoConfigurationImportSelector;
import org.springframework.core.type.AnnotationMetadata;

/** @deprecated */
@Deprecated
public class EnableAutoConfigurationImportSelector extends AutoConfigurationImportSelector {
    public EnableAutoConfigurationImportSelector() {
    }

    protected boolean isEnabled(AnnotationMetadata metadata) {
        return this.getClass().equals(EnableAutoConfigurationImportSelector.class)?((Boolean)this.getEnvironment().getProperty("spring.boot.enableautoconfiguration", Boolean.class, Boolean.valueOf(true))).booleanValue():true;
    }
}
```
3、向上延伸查看父类
```java
//
// Source code recreated from a .class file by IntelliJ IDEA
// (powered by Fernflower decompiler)
//

package org.springframework.boot.autoconfigure;

import java.io.***;

public class AutoConfigurationImportSelector implements DeferredImportSelector, BeanClassLoaderAware, ResourceLoaderAware, BeanFactoryAware, EnvironmentAware, Ordered {
    private static final String[] NO_IMPORTS = new String[0];
    private static final Log logger = LogFactory.getLog(AutoConfigurationImportSelector.class);
    private ConfigurableListableBeanFactory beanFactory;
    private Environment environment;
    private ClassLoader beanClassLoader;
    private ResourceLoader resourceLoader;

    public AutoConfigurationImportSelector() {
    }

    public String[] selectImports(AnnotationMetadata annotationMetadata) {
        if(!this.isEnabled(annotationMetadata)) {
            return NO_IMPORTS;
        } else {
            try {
                AutoConfigurationMetadata ex = AutoConfigurationMetadataLoader.loadMetadata(this.beanClassLoader);
                AnnotationAttributes attributes = this.getAttributes(annotationMetadata);
                List configurations = this.getCandidateConfigurations(annotationMetadata, attributes);
                configurations = this.removeDuplicates(configurations);
                configurations = this.sort(configurations, ex);
                Set exclusions = this.getExclusions(annotationMetadata, attributes);
                this.checkExcludedClasses(configurations, exclusions);
                configurations.removeAll(exclusions);
                configurations = this.filter(configurations, ex);
                this.fireAutoConfigurationImportEvents(configurations, exclusions);
                return (String[])configurations.toArray(new String[configurations.size()]);
            } catch (IOException var6) {
                throw new IllegalStateException(var6);
            }
        }
    }

    protected boolean isEnabled(AnnotationMetadata metadata) {
        return true;
    }

    protected AnnotationAttributes getAttributes(AnnotationMetadata metadata) {
        String name = this.getAnnotationClass().getName();
        AnnotationAttributes attributes = AnnotationAttributes.fromMap(metadata.getAnnotationAttributes(name, true));
        Assert.notNull(attributes, "No auto-configuration attributes found. Is " + metadata.getClassName() + " annotated with " + ClassUtils.getShortName(name) + "?");
        return attributes;
    }

    protected Class<?> getAnnotationClass() {
        return EnableAutoConfiguration.class;
    }

    protected List<String> getCandidateConfigurations(AnnotationMetadata metadata, AnnotationAttributes attributes) {
        List configurations = SpringFactoriesLoader.loadFactoryNames(this.getSpringFactoriesLoaderFactoryClass(), this.getBeanClassLoader());
        Assert.notEmpty(configurations, "No auto configuration classes found in META-INF/spring.factories. If you are using a custom packaging, make sure that file is correct.");
        return configurations;
    }

    protected Class<?> getSpringFactoriesLoaderFactoryClass() {
        return EnableAutoConfiguration.class;
    }
   
   ...
}
```

4、查看SpringFactoriesLoader
```java
//
// Source code recreated from a .class file by IntelliJ IDEA
// (powered by Fernflower decompiler)
//

package org.springframework.core.io.support;

import ...;

public abstract class SpringFactoriesLoader {
    private static final Log logger = LogFactory.getLog(SpringFactoriesLoader.class);
    public static final String FACTORIES_RESOURCE_LOCATION = "META-INF/spring.factories";

    public SpringFactoriesLoader() {
    }

    public static <T> List<T> loadFactories(Class<T> factoryClass, ClassLoader classLoader) {
        Assert.notNull(factoryClass, "\'factoryClass\' must not be null");
        ClassLoader classLoaderToUse = classLoader;
        if(classLoader == null) {
            classLoaderToUse = SpringFactoriesLoader.class.getClassLoader();
        }

        List factoryNames = loadFactoryNames(factoryClass, classLoaderToUse);
        if(logger.isTraceEnabled()) {
            logger.trace("Loaded [" + factoryClass.getName() + "] names: " + factoryNames);
        }

        ArrayList result = new ArrayList(factoryNames.size());
        Iterator var5 = factoryNames.iterator();

        while(var5.hasNext()) {
            String factoryName = (String)var5.next();
            result.add(instantiateFactory(factoryName, factoryClass, classLoaderToUse));
        }

        AnnotationAwareOrderComparator.sort(result);
        return result;
    }

    public static List<String> loadFactoryNames(Class<?> factoryClass, ClassLoader classLoader) {
        String factoryClassName = factoryClass.getName();

        try {
            Enumeration ex = classLoader != null?classLoader.getResources("META-INF/spring.factories"):ClassLoader.getSystemResources("META-INF/spring.factories");
            ArrayList result = new ArrayList();

            while(ex.hasMoreElements()) {
                URL url = (URL)ex.nextElement();
                Properties properties = PropertiesLoaderUtils.loadProperties(new UrlResource(url));
                String factoryClassNames = properties.getProperty(factoryClassName);
                result.addAll(Arrays.asList(StringUtils.commaDelimitedListToStringArray(factoryClassNames)));
            }

            return result;
        } catch (IOException var8) {
            throw new IllegalArgumentException("Unable to load [" + factoryClass.getName() + "] factories from location [" + "META-INF/spring.factories" + "]", var8);
        }
    }

    private static <T> T instantiateFactory(String instanceClassName, Class<T> factoryClass, ClassLoader classLoader) {
        try {
            Class ex = ClassUtils.forName(instanceClassName, classLoader);
            if(!factoryClass.isAssignableFrom(ex)) {
                throw new IllegalArgumentException("Class [" + instanceClassName + "] is not assignable to [" + factoryClass.getName() + "]");
            } else {
                Constructor constructor = ex.getDeclaredConstructor(new Class[0]);
                ReflectionUtils.makeAccessible(constructor);
                return constructor.newInstance(new Object[0]);
            }
        } catch (Throwable var5) {
            throw new IllegalArgumentException("Unable to instantiate factory class: " + factoryClass.getName(), var5);
        }
    }
}
```
