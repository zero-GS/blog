---
title: utils常用工具类
createTime: 2024/12/06 16:11:16
permalink: /article/8ogfwx2j/
---
## TreeUtils
```ts
import java.util.List;
import java.util.function.BiConsumer;
import java.util.function.BiFunction;
import java.util.function.Predicate;
import java.util.stream.Collectors;

public class TreeUtils {
    public static <E> List<E> makeTree(List<E> list, Predicate<E> rootCheck, BiFunction<E, E, Boolean> parentCheck, BiConsumer<E, List<E>> setSubChildren) {
        return list.stream() 
                .filter(rootCheck) 
                .peek(x -> setSubChildren.accept(x, makeChildren(x, list, parentCheck, setSubChildren)))
                .collect(Collectors.toList());
    }
    private static <E> List<E> makeChildren(E parent, List<E> allData, BiFunction<E, E, Boolean> parentCheck, BiConsumer<E, List<E>> setSubChildren) 
        return allData.stream() 
                .filter(x -> parentCheck.apply(parent, x)) 
                .peek(x -> setSubChildren.accept(x, makeChildren(x, allData, parentCheck, setSubChildren)))
                .collect(Collectors.toList());
}

//快速使用
TreeUtils.makeTree(
        list,
        item -> item.getParentId() == 0L, // 根节点 parentId的值  一般为0或null
        (parent, child) -> parent.getCategoryId().equals(child.getParentId()), // 检查父子关系
        (item, children) -> item.setChildren(children) // 设置子节点
);
```

## IpUtils
```ts
import org.springframework.web.context.request.RequestContextHolder;
import org.springframework.web.context.request.ServletRequestAttributes;
import javax.servlet.http.HttpServletRequest;

public class IPUtils {
    public static String getIpAddr() {
        HttpServletRequest request = ((ServletRequestAttributes) RequestContextHolder.getRequestAttributes()).getRequest();
        String ip = request.getHeader("x-forwarded-for");
        if (ip != null && ip.length() != 0 && !"unknown".equalsIgnoreCase(ip)) {
            // 多次反向代理后会有多个ip值，第一个ip才是真实ip
            if (ip.indexOf(",") != -1) {
                ip = ip.split(",")[0];
            }
        }
        if (ip == null || ip.length() == 0 || "unknown".equalsIgnoreCase(ip)) {
            ip = request.getHeader("Proxy-Client-IP");
        }
        if (ip == null || ip.length() == 0 || "unknown".equalsIgnoreCase(ip)) {
            ip = request.getHeader("WL-Proxy-Client-IP");
        }
        if (ip == null || ip.length() == 0 || "unknown".equalsIgnoreCase(ip)) {
            ip = request.getHeader("HTTP_CLIENT_IP");
        }
        if (ip == null || ip.length() == 0 || "unknown".equalsIgnoreCase(ip)) {
            ip = request.getHeader("HTTP_X_FORWARDED_FOR");
        }
        if (ip == null || ip.length() == 0 || "unknown".equalsIgnoreCase(ip)) {
            ip = request.getHeader("X-Real-IP");
        }
        if (ip == null || ip.length() == 0 || "unknown".equalsIgnoreCase(ip)) {
            ip = request.getRemoteAddr();
        }
        return ip;
    }
}
```