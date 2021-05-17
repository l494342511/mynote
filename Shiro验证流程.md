# Shiro验证流程

## 1. 后端的登录实现

```java
代码1：
UsernamePasswordToken token = new UsernamePasswordToken(user.getName(),pwd);
subject.login(token);
```

登录的实现非常简单，shiro都将整个流程封装了起来，当然这里我们必须要配置自己的reaml

```java
代码2：
@Override
protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken auToken) throws AuthenticationException {
    //传入的token是用户填写的表单 如账户密码
    UsernamePasswordToken token = (UsernamePasswordToken) auToken;
    //这里通过用户传进来的账户到数据库中查找相应的账户信息包括正确的密码
    user user = myService.findUserByUserName(token.getUsername()).get(0);
    //返回认证信息，包括：user（包含账户所有信息），密码，reaml的名字（因为可能会存在多个reaml）
    return new SimpleAuthenticationInfo(user,user.getPwd(),getName());
}
```

我们在自己的reaml中配置了doGetAuthenticationInfo方法，作用是通过传入代码1中的token（这个token实际上就是用户在登录时填写的表单）来查找数据库中相应的账户数据并且返回。查找的方式由我们自己定义，可以通过mysql查找，也可以通过redis，或者其他，反正只要能达到找到对应账户信息就可以了，shiro需要它用来做后续的密码校验。该方法的调用和密码的校验原理接下来阐述。

## 2. 验证实现的真正流程（源码阅读）

首先登录的表层入口是：

```java
流程1：
subject.login(token);
```

```java
public void login(AuthenticationToken token) throws AuthenticationException {
    .............
    Subject subject = securityManager.login(this, token);
    .............
}
```

可以看到subject将登录委托给了securityManager，而DefaultSecurityManager实现了这个方法

```java
 public Subject login(Subject subject, AuthenticationToken token) throws AuthenticationException {
        AuthenticationInfo info;
        try {
            info = authenticate(token);
        }
		........
        return loggedIn;
    }
```

而DefaultSecurityManager调用了AuthenticatingSecurityManager

```java
    public AuthenticationInfo authenticate(AuthenticationToken token) throws AuthenticationException {
        return this.authenticator.authenticate(token);
    }
```

AuthenticatingSecurityManager调用的AbstractAuthenticator中的authenticate方法

```java
public final AuthenticationInfo authenticate(AuthenticationToken token) {
    ......
	info = doAuthenticate(token);
	......
    return info;
}
```

AbstractAuthenticator调用了ModularRealmAuthenticator中的doAuthenticate方法通过reaml来进行校验，并且根据reaml是单个还是多个处理的方法不同，

```java
protected AuthenticationInfo doAuthenticate(AuthenticationToken authenticationToken) throws AuthenticationException {
    assertRealmsConfigured();
    Collection<Realm> realms = getRealms();
    if (realms.size() == 1) {
        return doSingleRealmAuthentication(realms.iterator().next(), authenticationToken);
    } else {
        return doMultiRealmAuthentication(realms, authenticationToken);
    }
}
```

以单个reaml为例子，其处理的方法为：

```java
protected AuthenticationInfo doSingleRealmAuthentication(Realm realm, AuthenticationToken token) {
    if (!realm.supports(token)) {
    	......
    }
    AuthenticationInfo info = realm.getAuthenticationInfo(token);
    .....
    return info;
}
```

其中又调用了realm的getAuthenticationInfo(token)方法，AuthenticatingRealm实现了该方法

```java
public final AuthenticationInfo getAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {
    
    	//获取账户信息
        AuthenticationInfo info = getCachedAuthenticationInfo(token);
        if (info == null) {
            info = doGetAuthenticationInfo(token);
            .....
        } else {
            .....
        }

    	//密码的校验
        if (info != null) {
            assertCredentialsMatch(token, info);
        } else {
            .....
        }

        return info;
    }
```

**这里做了两个重要的操作，1是调用了我们自定义的doGetAuthenticationInfo(token)方法找到了对应的账户信息info，2是进行了密码的校对，assertCredentialsMatch(token, info)方法实现了校对，这个方法最终是将token和info转换为了字节码进行比较**

**流程图：**

![image-20210504161710290](Shiro%E9%AA%8C%E8%AF%81%E6%B5%81%E7%A8%8B.assets/image-20210504161710290.png)

## 3. 这个过程除了做了密码校验还做了什么

后续补上