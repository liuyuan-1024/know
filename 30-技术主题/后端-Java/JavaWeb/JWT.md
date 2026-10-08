# JWT


Json Web Token是开放的行业标准，它包含了简洁、自包含的协议格式；用于在通信双方传送Json对象，传递的数据经过数字签名可以被验证和信任。JWT可以使用HMAC算法或RSA的公钥/私钥进行签名，防止被篡改。



简单来说：JWT就是一个加密了的JSON数据，但JWT本身是未加密的，即JWT中的JSON数据可被任何人解密读取。



## JWT的特点


（1）JWT 默认是不加密，但也是可以加密的。生成原始 Token 以后，可以用密钥再加密一次。



（2）JWT 不加密的情况下，不能将秘密数据写入 JWT。



（3）JWT 不仅可以用于认证，也可以用于交换信息。有效使用 JWT，可以降低服务器查询数据库的次数。



（4）JWT 的最大缺点是，由于服务器不保存 session 状态，因此无法在使用过程中废止某个 token，或者更改 token 的权限。也就是说，一旦 JWT 签发了，在到期之前就会始终有效，除非服务器部署额外的逻辑。



（5）JWT 本身包含了认证信息，一旦泄露，任何人都可以获得该令牌的所有权限。为了减少盗用，JWT 的有效期应该设置得比较短。对于一些比较重要的权限，使用时应该再次对用户进行认证。



（6）为了减少盗用，JWT 不应该使用 HTTP 协议明码传输，要使用 HTTPS 协议传输。



## JWT 请求流程


1. 用户使用账号和密码发起**POST请求**；
2. 服务器使用**私钥**创建一个**JWT**；
3. 服务器返回这个 JWT 给浏览器；
4. 浏览器将该JWT**串在请求头中**向服务器发送请求；
5. 服务器验证该JWT；
6. 返回响应的资源给浏览器。



## JWT 的主要应用场景


身份认证在这种场景下，一旦用户完成了登录，在接下来的每个请求中都包含 JWT，可以用来验证用户身份以及对路由、服务和资源的访问权限进行验证。由于它的**开销非常小**，可以轻松的在不同域名的系统中传递，所有目前在单点登录（SSO）中比较广泛的使用了该技术。 信息交换在通信的双方之间使用 JWT 对数据进行编码是一种非常安全的方式，由于它的信息是经过签名的，可以确保发送者发送的信息是没有经过伪造的。



## JWT的组成


JWT实际就是一个字符串，由**“头部(Header)、载荷(Payload)、签名(signature)”**组成



### 头部(Header)


头部信息用于描述JWT的基本信息，eg：alg(签名算法HMAC/SHA256/RSA)，也可以表示成一个JSON对象



```json
{
    "alg": "HS256", // 算法
    "typ": "JWT"	// 令牌的类型
}
```



### 载荷(Payload)


Payload 部分也是一个 JSON 对象，用来存放实际需要传递的有效信息。有效信息包含三个部分：



1. 标准中注册的声明
2. 公共的声明
3. 私有的声明



+  JWT标准注册的声明 (建议但不强制使用) ： 
    - iss（issuer）：jwt的签发者
    - sub（subject）：jwt所面向的用户，主题
    - aud（audience）：接收jwt的一方，受众
    - iat（Issuer At）：jwt的签发时间
    - exp（expiration time）：jwt的过期时间，必须大于签发时间
    - nbf（Not Before）：定义在什么时间之前，该jwt是不可用的
    - jti（JWT ID）：jwt的唯一身份标识，主要用来作为一次性token，从而回避重放攻击
+  公共声明  
可以添加任意的信息，一般添加用户的信息或业务需要的必要信息；但不建议添加敏感信息，因为Base64是对称解密的，该部分在客户端可以解密 
+  私有声明  
提供者和使用者共同定义的声明，一般不建议存放敏感信息，因为Base64是对称解密的，意味着该部分信息可以归类为明文信息。这个 JSON 对象也要使用 Base64URL 算法转成字符串。  
就是自定义的claim，这些claim与JWT定义的claim的区别：JWT定义的claim，JWT的接收方拿到后，知道怎么对标准claim进行验证（不一定验证），而**private claim不会验证，除非明确告知接收方要对这些claim进行验证并且指定验证规则** 



### 签名(signature)


Signature 部分是对前两部分的签名，防止数据篡改。



签证信息，由三部分组成：



1. header（base64加密后）
2. payload（base64加密后）
3. secret（盐、密钥）：保存在服务器端，jwt的签发也是在服务器端的，secret就是用来进行jwt的签发、验证的；它就是你在服务端的密钥，在任何场景都不应该泄露，一旦客户端知道这个secret，就以为这客户端可以自我签发jwt



按照一定规则生成签名：**HMACSHA256( base64UrlEncode( header ) + "." + base64UrlEncode( payload ), secret )**



## JWT 的使用方式


客户端收到服务器返回的 JWT 之后需要在本地做保存。此后，客户端每次与服务器通信，都要带上这个 JWT。一般的的做法是放在 HTTP 请求的头信息 `Authorization` 字段里面。



```html
Authorization: Bearer <token>
```



这样每个请求中，服务端就可以在请求头中拿到 JWT  进行解析与认证。



## jwt token无感刷新


1. 登录成功会拿到：access_token 和 refresh_token
2. 当 access_token 过期时，前端 **axiosl拦截器** 将会发送 refresh_token 到令牌刷新接口，请求重新签发令牌
3. 新令牌替换旧令牌，继续用户当前的请求



## JWT 的特性


1. JWT 默认是不加密，但也是可以加密的。生成原始 Token 以后，可以用密钥再加密一次。
2. JWT 不加密的情况下，不能将秘密数据写入 JWT。
3. JWT 不仅可以用于认证，也可以用于交换信息。有效使用 JWT，可以降低服务器查询数据库的次数。
4. JWT 的最大缺点是，由于服务器不保存 [session](https://so.csdn.net/so/search?q=session&spm=1001.2101.3001.7020) 状态，因此无法在使用过程中废止某个 token，或者更改 token 的权限。也就是说，一旦 JWT 签发了，在到期之前就会始终有效，除非服务器部署额外的逻辑。
5. JWT 本身包含了认证信息，一旦泄露，任何人都可以获得该令牌的所有权限。为了减少盗用，JWT 的有效期应该设置得比较短。对于一些比较重要的权限，使用时应该再次对用户进行认证。
6. 为了减少盗用，JWT 不应该使用 HTTP 协议明码传输，要使用 HTTPS 协议传输。



## JWT工具类


```java
package com.bug.utils;

import com.bug.bean.User;
import io.jsonwebtoken.Claims;
import io.jsonwebtoken.ExpiredJwtException;
import io.jsonwebtoken.Jwts;
import io.jsonwebtoken.SignatureAlgorithm;
import org.springframework.security.core.Authentication;
import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.userdetails.UserDetails;

import java.util.Collection;
import java.util.Date;
import java.util.HashMap;
import java.util.Map;

/**
 * JWT工具类
 */
public class JwtUtils {
    //一般放在官方声明中
    // 密钥
    private static final String TOKEN_SECRET = "BugOS";
    // 签发时间
    private static final String CLAIM_KEY_ISSUED_AT = "iat";
    // 过期时间
    private static final long EXPIRATION_TIME = 1000 * 60 * 30;


    //一般放在自定义声明中
    // jwt主题
    private static final String CLAIM_KEY_SUBJECT = "sub";
    // 用户权限
    private static final String CLAIM_KEY_AUTHORITIES = "authorities";


    /**
     * 拼接用户权限，转为字符串
     */
    private static StringBuffer spliceAuthorities(Collection<? extends GrantedAuthority> authorities) {
        StringBuffer permissionString = new StringBuffer();

        // 拼接用户权限字符串
        for (GrantedAuthority authority : authorities) {
            permissionString.append(authority.getAuthority()).append(",");
        }

        return permissionString;
    }

    /**
     * 根据Authentication对象生成JWT
     */
    public static String generateTokenByAuthentication(Authentication authentication) {
        //获取登录用户的角色权限
        StringBuffer authoritiesString = spliceAuthorities(authentication.getAuthorities());

        //自定义属性
        Map<String, Object> claims = new HashMap<>();
        claims.put(CLAIM_KEY_SUBJECT, authentication.getName());
        claims.put(CLAIM_KEY_AUTHORITIES, authoritiesString);

        return Jwts.builder()
                //JWT的签发日期
                .setIssuedAt(new Date())
                //过期时间
                .setExpiration(new Date(System.currentTimeMillis() + EXPIRATION_TIME))
                //签名
                .signWith(SignatureAlgorithm.HS256, TOKEN_SECRET)
                //自定义属性
                .setClaims(claims)
                .compact();
    }

    /**
     * 根据UserDetails对象创建claims集合然后生成对应JWT
     */
    public static String generateTokenByUserDetails(User userDetails) {
        //获取登录用户的角色权限
        StringBuffer stringBuffer = spliceAuthorities(userDetails.getAuthorities());

        //自定义声明（private claims）
        Map<String, Object> claims = new HashMap<>();

        //JWT主题
        claims.put(CLAIM_KEY_SUBJECT, userDetails.getUsername());
        //用户权限
        claims.put(CLAIM_KEY_AUTHORITIES, stringBuffer);

        return generateTokenByClaims(claims);
    }

    /**
     * 根据Claims生成JWT
     */
    public static String generateTokenByClaims(Map<String, Object> claims) {
        return Jwts.builder()
                .setIssuedAt(new Date())
                .setExpiration(new Date(System.currentTimeMillis() + EXPIRATION_TIME))
                .signWith(SignatureAlgorithm.HS256, TOKEN_SECRET)
                .setClaims(claims)
                .compact();
    }

    //解析JWT, 获得Claims
    public static Claims parseToken(String token) {
        Claims claims;
        try {
            claims = Jwts.parser()
                    .setSigningKey(TOKEN_SECRET)
                    .parseClaimsJws(token)
                    .getBody();
        } catch (ExpiredJwtException e) {
            //如果过期,在异常中调用, 返回claims, 否则无法解析过期的token
            claims = e.getClaims();
        } catch (Exception e) {
            claims = null;
        }
        return claims;
    }

    //从JWT中获得主题
    public static String getSubjectFromToken(String token) {
        try {
            return parseToken(token).getSubject();
        } catch (ExpiredJwtException e) {
            //如果过期, 需要在此处异常调用如下的方法, 否则拿不到用户名
            return e.getClaims().getSubject();
        } catch (Exception e) {
            e.printStackTrace();
            return null;
        }
    }

    //从JWT中获取签发时间
    public static Date getIssuedAtFromToken(String token) {
        Date issuedAt;
        try {
            Claims claims = parseToken(token);
            issuedAt = claims.getIssuedAt();
        } catch (Exception e) {
            issuedAt = null;
        }
        return issuedAt;
    }

    //从JWT中获取过期时间
    public static Date getExpirationFromToken(String token) {
        Date expiration;
        try {
            Claims claims = parseToken(token);
            expiration = claims.getExpiration();
        } catch (Exception e) {
            expiration = null;
        }
        return expiration;
    }

    //判断JWT是否过期
    private static Boolean isTokenExpired(String token) {
        Date expiration = getExpirationFromToken(token);
        //判断过期时间是否在当前时间之前
        return expiration.before(new Date());
    }

    //JWT是否可以被刷新(过期就可以被刷新) isCanRefreshToken
    public Boolean isRefreshed(String token) {
        return isTokenExpired(token);
    }

    //刷新JWT
    public static String refreshToken(String token) {
        String refreshedToken;
        try {
            //获得Token的Claims, 由于在生成JWT的时候会根据当前时间更新过期时间, 我们只需要手动修改
            //放在自定义属性中的创建时间就可以了
            Claims claims = parseToken(token);
            claims.put(CLAIM_KEY_ISSUED_AT, new Date());
            //利用修改后的claims再次生成token, 就不需要我们每次都去查用户的信息和权限了
            refreshedToken = generateTokenByClaims(claims);
        } catch (Exception e) {
            refreshedToken = null;
        }
        return refreshedToken;
    }

    //判断Token是否合法
    public static Boolean isValidate(String token, UserDetails userDetails) {
        User user = (User) userDetails;
        String subject = getSubjectFromToken(token);
        return (
                //如果用户名与token一致且token没有过期, 则认为合法
                user.getUsername().equals(subject) && !isTokenExpired(token)
        );
    }

}
```



## Redis工具类


```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.stereotype.Component;

import java.util.List;
import java.util.Map;
import java.util.Set;
import java.util.concurrent.TimeUnit;

@Component
public final class RedisUtils {

    @Autowired
    private RedisTemplate<String, Object> redisTemplate;

    // =============================common============================

    /**
     * 指定缓存失效时间
     *
     * @param key  键
     * @param time 时间(秒)
     */
    public boolean expire(String key, long time) {
        try {
            if (time > 0) {
                redisTemplate.expire(key, time, TimeUnit.SECONDS);
            }
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    /**
     * 根据key 获取过期时间
     *
     * @param key 键 不能为null
     * @return 时间(秒) 返回0代表为永久有效
     */
    public long getExpire(String key) {
        return redisTemplate.getExpire(key, TimeUnit.SECONDS);
    }


    /**
     * 判断key是否存在
     *
     * @param key 键
     * @return true 存在 false不存在
     */
    public boolean hasKey(String key) {
        try {
            return redisTemplate.hasKey(key);
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }


    /**
     * 删除缓存
     *
     * @param key 可以传一个值 或多个
     */
    @SuppressWarnings("unchecked")
//  public void del(String... key) {
//    if (key != null && key.length > 0) {
//      if (key.length == 1) {
//        redisTemplate.delete(key[0]);
//      } else {
//        redisTemplate.delete(CollectionUtils.arrayToList(key));
//      }
//    }
//  }


    // ============================String=============================

    /**
     * 普通缓存获取
     *
     * @param key 键
     * @return 值
     */
    public Object get(String key) {
        return key == null ? null : redisTemplate.opsForValue().get(key);
    }

    /**
     * 普通缓存放入
     *
     * @param key   键
     * @param value 值
     * @return true成功 false失败
     */

    public boolean set(String key, Object value) {
        try {
            redisTemplate.opsForValue().set(key, value);
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }


    /**
     * 普通缓存放入并设置时间
     *
     * @param key   键
     * @param value 值
     * @param time  时间(秒) time要大于0 如果time小于等于0 将设置无限期
     * @return true成功 false 失败
     */

    public boolean set(String key, Object value, long time) {
        try {
            if (time > 0) {
                redisTemplate.opsForValue().set(key, value, time, TimeUnit.SECONDS);
            } else {
                set(key, value);
            }
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }


    /**
     * 递增
     *
     * @param key   键
     * @param delta 要增加几(大于0)
     */
    public long incr(String key, long delta) {
        if (delta < 0) {
            throw new RuntimeException("递增因子必须大于0");
        }
        return redisTemplate.opsForValue().increment(key, delta);
    }


    /**
     * 递减
     *
     * @param key   键
     * @param delta 要减少几(小于0)
     */
    public long decr(String key, long delta) {
        if (delta < 0) {
            throw new RuntimeException("递减因子必须大于0");
        }
        return redisTemplate.opsForValue().increment(key, -delta);
    }


    // ================================Map=================================

    /**
     * HashGet
     *
     * @param key  键 不能为null
     * @param item 项 不能为null
     */
    public Object hget(String key, String item) {
        return redisTemplate.opsForHash().get(key, item);
    }

    /**
     * 获取hashKey对应的所有键值
     *
     * @param key 键
     * @return 对应的多个键值
     */
    public Map<Object, Object> hmget(String key) {
        return redisTemplate.opsForHash().entries(key);
    }

    /**
     * HashSet
     *
     * @param key 键
     * @param map 对应多个键值
     */
    public boolean hmset(String key, Map<String, Object> map) {
        try {
            redisTemplate.opsForHash().putAll(key, map);
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }


    /**
     * HashSet 并设置时间
     *
     * @param key  键
     * @param map  对应多个键值
     * @param time 时间(秒)
     * @return true成功 false失败
     */
    public boolean hmset(String key, Map<String, Object> map, long time) {
        try {
            redisTemplate.opsForHash().putAll(key, map);
            if (time > 0) {
                expire(key, time);
            }
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }


    /**
     * 向一张hash表中放入数据,如果不存在将创建
     *
     * @param key   键
     * @param item  项
     * @param value 值
     * @return true 成功 false失败
     */
    public boolean hset(String key, String item, Object value) {
        try {
            redisTemplate.opsForHash().put(key, item, value);
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    /**
     * 向一张hash表中放入数据,如果不存在将创建
     *
     * @param key   键
     * @param item  项
     * @param value 值
     * @param time  时间(秒) 注意:如果已存在的hash表有时间,这里将会替换原有的时间
     * @return true 成功 false失败
     */
    public boolean hset(String key, String item, Object value, long time) {
        try {
            redisTemplate.opsForHash().put(key, item, value);
            if (time > 0) {
                expire(key, time);
            }
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }


    /**
     * 删除hash表中的值
     *
     * @param key  键 不能为null
     * @param item 项 可以使多个 不能为null
     */
    public void hdel(String key, Object... item) {
        redisTemplate.opsForHash().delete(key, item);
    }


    /**
     * 判断hash表中是否有该项的值
     *
     * @param key  键 不能为null
     * @param item 项 不能为null
     * @return true 存在 false不存在
     */
    public boolean hHasKey(String key, String item) {
        return redisTemplate.opsForHash().hasKey(key, item);
    }


    /**
     * hash递增 如果不存在,就会创建一个 并把新增后的值返回
     *
     * @param key  键
     * @param item 项
     * @param by   要增加几(大于0)
     */
    public double hincr(String key, String item, double by) {
        return redisTemplate.opsForHash().increment(key, item, by);
    }


    /**
     * hash递减
     *
     * @param key  键
     * @param item 项
     * @param by   要减少记(小于0)
     */
    public double hdecr(String key, String item, double by) {
        return redisTemplate.opsForHash().increment(key, item, -by);
    }


    // ============================set=============================

    /**
     * 根据key获取Set中的所有值
     *
     * @param key 键
     */
    public Set<Object> sGet(String key) {
        try {
            return redisTemplate.opsForSet().members(key);
        } catch (Exception e) {
            e.printStackTrace();
            return null;
        }
    }


    /**
     * 根据value从一个set中查询,是否存在
     *
     * @param key   键
     * @param value 值
     * @return true 存在 false不存在
     */
    public boolean sHasKey(String key, Object value) {
        try {
            return redisTemplate.opsForSet().isMember(key, value);
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }


    /**
     * 将数据放入set缓存
     *
     * @param key    键
     * @param values 值 可以是多个
     * @return 成功个数
     */
    public long sSet(String key, Object... values) {
        try {
            return redisTemplate.opsForSet().add(key, values);
        } catch (Exception e) {
            e.printStackTrace();
            return 0;
        }
    }


    /**
     * 将set数据放入缓存
     *
     * @param key    键
     * @param time   时间(秒)
     * @param values 值 可以是多个
     * @return 成功个数
     */
    public long sSetAndTime(String key, long time, Object... values) {
        try {
            Long count = redisTemplate.opsForSet().add(key, values);
            if (time > 0)
                expire(key, time);
            return count;
        } catch (Exception e) {
            e.printStackTrace();
            return 0;
        }
    }


    /**
     * 获取set缓存的长度
     *
     * @param key 键
     */
    public long sGetSetSize(String key) {
        try {
            return redisTemplate.opsForSet().size(key);
        } catch (Exception e) {
            e.printStackTrace();
            return 0;
        }
    }


    /**
     * 移除值为value的
     *
     * @param key    键
     * @param values 值 可以是多个
     * @return 移除的个数
     */

    public long setRemove(String key, Object... values) {
        try {
            Long count = redisTemplate.opsForSet().remove(key, values);
            return count;
        } catch (Exception e) {
            e.printStackTrace();
            return 0;
        }
    }

    // ===============================list=================================

    /**
     * 获取list缓存的内容
     *
     * @param key   键
     * @param start 开始
     * @param end   结束 0 到 -1代表所有值
     */
    public List<Object> lGet(String key, long start, long end) {
        try {
            return redisTemplate.opsForList().range(key, start, end);
        } catch (Exception e) {
            e.printStackTrace();
            return null;
        }
    }


    /**
     * 获取list缓存的长度
     *
     * @param key 键
     */
    public long lGetListSize(String key) {
        try {
            return redisTemplate.opsForList().size(key);
        } catch (Exception e) {
            e.printStackTrace();
            return 0;
        }
    }


    /**
     * 通过索引 获取list中的值
     *
     * @param key   键
     * @param index 索引 index>=0时， 0 表头，1 第二个元素，依次类推；index<0时，-1，表尾，-2倒数第二个元素，依次类推
     */
    public Object lGetIndex(String key, long index) {
        try {
            return redisTemplate.opsForList().index(key, index);
        } catch (Exception e) {
            e.printStackTrace();
            return null;
        }
    }


    /**
     * 将list放入缓存
     *
     * @param key   键
     * @param value 值
     */
    public boolean lSet(String key, Object value) {
        try {
            redisTemplate.opsForList().rightPush(key, value);
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }


    /**
     * 将list放入缓存
     *
     * @param key   键
     * @param value 值
     * @param time  时间(秒)
     */
    public boolean lSet(String key, Object value, long time) {
        try {
            redisTemplate.opsForList().rightPush(key, value);
            if (time > 0)
                expire(key, time);
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }

    }


    /**
     * 将list放入缓存
     *
     * @param key   键
     * @param value 值
     * @return
     */
    public boolean lSet(String key, List<Object> value) {
        try {
            redisTemplate.opsForList().rightPushAll(key, value);
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }

    }


    /**
     * 将list放入缓存
     *
     * @param key   键
     * @param value 值
     * @param time  时间(秒)
     * @return
     */
    public boolean lSet(String key, List<Object> value, long time) {
        try {
            redisTemplate.opsForList().rightPushAll(key, value);
            if (time > 0)
                expire(key, time);
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }


    /**
     * 根据索引修改list中的某条数据
     *
     * @param key   键
     * @param index 索引
     * @param value 值
     * @return
     */

    public boolean lUpdateIndex(String key, long index, Object value) {
        try {
            redisTemplate.opsForList().set(key, index, value);
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }


    /**
     * 移除N个值为value
     *
     * @param key   键
     * @param count 移除多少个
     * @param value 值
     * @return 移除的个数
     */

    public long lRemove(String key, long count, Object value) {
        try {
            Long remove = redisTemplate.opsForList().remove(key, count, value);
            return remove;
        } catch (Exception e) {
            e.printStackTrace();
            return 0;
        }

    }

}
```



## JWT过滤器


### jwt认证过滤器


JwtLoginFilter extends UsernamePasswordAuthenticationFilter



```java
package com.bug.filter;

import com.alibaba.fastjson.JSON;
import com.bug.bean.User;
import com.bug.service.MyUserDetailsService;
import com.bug.utils.JwtUtils;
import com.bug.utils.RedisUtils;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
import org.springframework.security.core.Authentication;
import org.springframework.security.core.AuthenticationException;
import org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter;
import org.springframework.web.context.WebApplicationContext;
import org.springframework.web.context.support.WebApplicationContextUtils;

import javax.servlet.FilterChain;
import javax.servlet.ServletException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

/**
 * 启动登录认证流程过滤器
 */
public class JwtLoginFilter extends UsernamePasswordAuthenticationFilter {

    @Autowired
    private RedisUtils redisUtils;

    @Autowired
    private MyUserDetailsService userDetailsService;

    public JwtLoginFilter(AuthenticationManager authManager) {
        setAuthenticationManager(authManager);
    }

    @Override
    public Authentication attemptAuthentication(HttpServletRequest request, HttpServletResponse response) throws AuthenticationException {
        // 可以在此覆写尝试进行登录认证的逻辑，登录成功之后等操作不再此方法内
        // 如果使用此过滤器来触发登录认证流程，注意登录请求数据格式的问题
        // 此过滤器的用户名密码默认从request.getParameter()获取，但是这种
        // 读取方式不能读取到如 application/json 等 post 请求数据，需要把
        // 用户名密码的读取逻辑修改为到流中读取request.getInputStream()
        //只允许post请求

        if (request.getMethod().equals("POST")) {
            String username = request.getParameter("username");
            String password = request.getParameter("password");

            UsernamePasswordAuthenticationToken authRequest = new UsernamePasswordAuthenticationToken(username, password);

            return this.getAuthenticationManager().authenticate(authRequest);
        } else {
            throw new RuntimeException("only post requests are allowed to login");
        }

    }

    @Override
    public void successfulAuthentication(HttpServletRequest request, HttpServletResponse response, FilterChain chain,
                                         Authentication authResult) throws IOException, ServletException {

        User user = (User) authResult.getPrincipal();

        // 创建Token
        String token = JwtUtils.generateTokenByAuthentication(authResult);

        // 设置编码 防止乱码问题
        response.setCharacterEncoding("UTF-8");
        response.setContentType("application/json; charset=utf-8");

        // 在请求头里返回创建成功的token
        response.setHeader("token", token);

        //由于filter的加载在bean加载之前，所以自动注入bean会出现null，使用以下方案解决
        if (redisUtils == null) {
            WebApplicationContext wac = WebApplicationContextUtils.getRequiredWebApplicationContext(request.getServletContext());
            redisUtils = wac.getBean(RedisUtils.class);
        }

        if (userDetailsService == null) {
            WebApplicationContext wac = WebApplicationContextUtils.getRequiredWebApplicationContext(request.getServletContext());
            userDetailsService = wac.getBean(MyUserDetailsService.class);
        }

        //将token存入redis中
        String tokenRedisKey = redisUtils.getUserToken() + user.getUsername();
        redisUtils.set(tokenRedisKey, token, redisUtils.getExpirationTime());

        // 处理编码方式 防止中文乱码
        response.setContentType("text/json;charset=utf-8");
        // 将反馈塞到HttpServletResponse中返回给前台
        response.getWriter().write(JSON.toJSONString("登录成功"));
    }

    @Override
    protected void unsuccessfulAuthentication(HttpServletRequest request, HttpServletResponse response, AuthenticationException failed)
            throws IOException, ServletException {

        response.getWriter().print("登陆失败");
    }
}
```



### jwt授权过滤器


```java
import com.bug.utils.JwtUtils;
import com.bug.utils.RedisUtils;
import io.jsonwebtoken.Claims;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.security.web.authentication.www.BasicAuthenticationFilter;
import org.springframework.util.ObjectUtils;
import org.springframework.util.StringUtils;

import javax.servlet.FilterChain;
import javax.servlet.ServletException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;


/**
 * 认证检查过滤器
 */
public class JwtAuthenticationFilter extends BasicAuthenticationFilter {

    @Autowired
    private RedisUtils redisUtils;

    public JwtAuthenticationFilter(AuthenticationManager authenticationManager) {
        super(authenticationManager);
    }

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain)
            throws IOException, ServletException {

        //获取token，token是放在请求头中的
        String token = request.getHeader("token");

        if (!StringUtils.hasText(token)) {//没有token或token是空字符串
            //直接放行
            filterChain.doFilter(request, response);
            return;
        }

        //解析token
        Claims claims = JwtUtils.parseToken(token);
        String subject = claims.getSubject();

        //从redis中获取用户信息
        String tokenRedisKey = "login:" + subject;
        Object obj = redisUtils.get(tokenRedisKey);

        if (ObjectUtils.isEmpty(obj)) {
            throw new RuntimeException("redis中不存在该用户");
        }

        //存入SecurityContext
        // TODO: 2022/4/18 获取权限信息封装到authenticationToken
        UsernamePasswordAuthenticationToken authenticationToken =
                new UsernamePasswordAuthenticationToken(obj, null, null);

        SecurityContextHolder.getContext().setAuthentication(authenticationToken);

        filterChain.doFilter(request, response);
    }

}
```



## 添加JWT过滤器


在SecurityConfig的config(HttpSecurity http)方法中添加以下语句



```java
//添加jwt过滤器
http.addFilter(new JwtLoginFilter(authenticationManager()))
    .addFilter(new JwtAuthenticationFilter(authenticationManager()));
```



## JWT 登录时报错


使用 Spring Boot 和 io.jsonwebtoken:jjwt 来实现 JWT 登录时报错



报错： java.lang.ClassNotFoundException: javax.xml.bind.DatatypeConverter



原因：可能是JDK版本太高



```xml
<dependency>
    <groupId>javax.xml.bind</groupId>
    <artifactId>jaxb-api</artifactId>
</dependency>
```

