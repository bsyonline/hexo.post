---
title: dynamic replace spring bean
date: 2021-05-18 09:43:55
tags:
---





```java
public void change(String beanName, Object bean) {
    BeanDefinition beanDefinition = beanFactory.getBeanDefinition(beanName);
    beanFactory.removeBeanDefinition(beanName);
    beanDefinition.setBeanClassName(bean.getClass().getCanonicalName());
    beanFactory.registerBeanDefinition(beanName, beanDefinition);
    beanFactory.registerSingleton(beanName, bean);
}
```

