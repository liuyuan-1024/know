代码实现

## WebFlux获取方法的注解
```java
// 关键，通过RequestMappingHandlerMapping 我们可以获取到MethodHandler  
@Resource  
private RequestMappingHandlerMapping handlerMapping;  
  
// 校验请求是否有级别足够的权限 todo 校验权限  
handlerMapping.getHandler(exchange).switchIfEmpty(chain.filter(exchange))  
    .flatMap(handler -> {  
        if (handler instanceof HandlerMethod) {  
            HandlerMethod methodHandle = (HandlerMethod) handler;  
            AuthCheck auth = methodHandle.getMethodAnnotation(AuthCheck.class);  
            // 必需权限不为空  
            if (ObjectUtils.isNotEmpty(auth)) {  
                // 目标权限的枚举对象  
                UserRoleEnum mustRoleEnum = UserRoleEnum.getEnumByRole(auth.mustRole());  
                // 目标权限存在时  
                if (ObjectUtils.isNotEmpty(mustRoleEnum)) {  
                    // 如果 userRole的优先级低于mustRole的 则抛出异常  
                    if (!UserRoleEnum.isPriority(loginUserRoleEnum, mustRoleEnum)) {  
                        return Mono.error(new BusinessException(ErrorCode.NO_AUTH_ERROR));  
                    }  
                }  
            }  
        }  
    });
```

