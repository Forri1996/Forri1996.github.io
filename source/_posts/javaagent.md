---
title: mock服务在桌面云场景下的设计方案
date: 2023-09-28 15:49:43
tags: java
---

## 背景
桌面云场景下，管理台非常多的接口会需要与底层接口交互，比如桌面开关机，网络规则的创建与更新等等。

在对底层openstack接口强依赖的前提下，管理台系统服务的压测成为一个难题：底层接口无法支持大规模批量压测请求。因此，mock需求应运而生。

## 需求分析
市面上常见的mock服务，大多数的方案是通过配置实现mock接口的复用。更高级些，可以通过流量日志采集，更方便，更搞笑地获取更真实的响应mock结果。

但是在云桌面场景下，该方案走不通。因为在大多数资源变更场景下，对于接口响应的上下文是有依赖关系的。举个例子：在桌面创建流程中，调用完create接口，紧接着就要调用getDetail接口。

此时如果只是通过配置静态的mock返回结果，显然很难将该流程链路串联起来。

因此，我们需要这样一个服务，在调用完createMock后，能够将mock创建出来的资源，看起来《真实》地创建在底层，并且能够通过getMock接口返回其创建的状态。

## 难点拆解
为了实现上述需求，我们大致需要做到两件事情：

- 深刻理解openstack底层资源状态扭转的业务流程，并将其在mock服务层中实现。
- 对压测目标服务进行代理，动态地修改其请求opesntack的endpoint配置，识别压测请求，代理到mock服务。


## 业务流程分析

在多region场景下，每个可用区region的endpoint可能不一样，因此需要提供一套openstackapi请求组件，根据请求的region，az等信息，动态获取endpoint。

基于此，agent可以在获取相关请求对象之后，判断是否为压测流量（更简单点，通过判断环境变量开关来判断是否走压测服务，压测时开启开关，压测结束关闭开关即可）

这里给出demo版本代码，供参考

```
public byte[] transform(ClassLoader loader, String className, Class<?> classBeingRedefined,
                            ProtectionDomain protectionDomain, byte[] classfileBuffer) throws IllegalClassFormatException {
        if (className.contains("OsBaseRestApi")) {
            System.out.println(className);
        }
        if (className.equals("com/ctgcd/clouddesktop/resserver/service/impl/OsInstService")) {
            try {
                // 使用Javaassist加载被代理类的字节码
                ClassPool classPool = ClassPool.getDefault();
                CtClass ctClass = classPool.makeClass(new ByteArrayInputStream(classfileBuffer));

                // 找到需要修改的方法，假设方法名为"methodName"
                CtMethod targetMethod = ctClass.getDeclaredMethod("getOsCredential");
                System.out.println("ctmethod:" + targetMethod.toString());
                // 在方法执行结束后插入代码，修改返回对象的属性
                String code = "{" +
                        "    if (System.getenv(\"IS_PRESS_TEST\").equals(\"1\")) {" +
                        "        $_.setOsInstHost(\"10.10.208.201\");" +
                        "    }" +
                        "}";
                targetMethod.insertAfter(code);

                // 返回修改后的字节码
                return ctClass.toBytecode();
            } catch (Exception e) {
                e.printStackTrace();
            }
        }

        return null;
    }
```

![image](https://github.com/Forri1996/blog-talk/assets/128824087/f24ebb67-26c9-42e3-a1e8-14836e0ae400)

## mock服务实现
要完整实现虚机创建在db中的流程，需要好好看看nova-compute的源码。可以参考下面两个网站：

https://blog.csdn.net/don_chiang709/article/details/90448861

https://jckling.github.io/2021/05/23/OpenStack/OpenStack%20Nova/index.html