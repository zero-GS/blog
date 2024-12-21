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
## FileUtils
```ts
package com.easy.utils;

import java.io.File;
import java.io.IOException;
import java.nio.charset.MalformedInputException;
import java.nio.charset.StandardCharsets;
import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.Paths;
import java.nio.file.StandardCopyOption;
import java.util.List;

public class FileUtil {
    public static void main(String[] args) {
//        copyDirectory("e:/123", "e:/1234");
        readTextFile("e:/123/18c96fbf4e8b6f43f75db39593388cc6.mp4");
    }

    /**
     * 创建目录 如果不存在就创建
     *
     * @param filePath 路径
     * @return
     */
    public static void createDirectory(String filePath) {
        try {
            Path path = Paths.get(filePath);
            if (!Files.exists(path)) {
                Files.createDirectories(path);
                System.out.println("目录创建成功");
                return;
            }
            System.out.println("目录已存在");
        } catch (IOException e) {
            throw new RuntimeException("目录创建失败");
        }
    }

    /**
     * 创建文件
     *
     * @param file    文件
     * @param content 文件内容
     */
    public static void createFile(File file, String content) {
        try {
            Files.writeString(file.toPath(), content);
            System.out.println("文件创建成功");
        } catch (IOException e) {
            throw new RuntimeException("文件创建失败");
        }
    }

    /**
     * 文件拷贝方法
     * 该方法用于将指定路径的文件拷贝到另一个路径下
     *
     * @param sourcePath 源文件路径，表示需要拷贝的文件的路径
     * @param targetPath 目标文件路径，表示文件拷贝后的存放路径
     */
    public static void copyFile(String sourcePath, String targetPath) {
        try {
            Path source = Paths.get(sourcePath);
            Path target = Paths.get(targetPath);
            // 检查源文件是否存在
            if (!Files.exists(source)) {
                throw new RuntimeException("源文件不存在: " + sourcePath);
            }
            // 检查目标目录是否存在，如果不存在则创建
            Path targetDir = target.getParent();
            if (!Files.exists(targetDir)) {
                Files.createDirectories(targetDir);
            }

            // 使用 Files 类的 copy 方法进行文件拷贝 如果目标文件已存在，则覆盖
            Files.copy(source, target, StandardCopyOption.REPLACE_EXISTING);
            System.out.println("文件拷贝成功");
        } catch (IOException e) {
            throw new RuntimeException("文件拷贝失败");
        }
    }


    /**
     * 删除文件
     *
     * @param filePath 文件的路径
     */
    public static void deleteFile(String filePath) {
        File file = new File(filePath);
        if (file.exists()) {
            file.delete();
            System.out.println("文件删除成功");
            return;
        }
        System.out.println("文件不存在");
    }

    /**
     * 拷贝目录 如果这个目录下有文件或者目录 则一并拷贝 否则只拷贝目录
     */
    public static void copyDirectory(String sourcePath, String targetPath) {
        try {
            Path source = Paths.get(sourcePath);
            Path target = Paths.get(targetPath);
            // 检查源目录是否存在
            if (!Files.exists(source)) {
                throw new RuntimeException("源目录不存在: " + sourcePath);
            }
            // 检查目标目录是否存在，如果不存在则创建
            if (!Files.exists(target)) {
                Files.createDirectories(target);
            }
            // 遍历源目录下的所有文件和子目录
            Files.walk(source).forEach(sourceFile -> {
                String targetFilePath = target.resolve(source.relativize(sourceFile)).toString();
                try {
                    // 如果是文件，则进行拷贝
                    if (Files.isRegularFile(sourceFile)) {
                        Files.copy(sourceFile, Paths.get(targetFilePath), StandardCopyOption.REPLACE_EXISTING);
                    }
                    // 如果是目录，则创建目标目录
                    if (Files.isDirectory(sourceFile)) {
                        Files.createDirectories(Paths.get(targetFilePath));
                    }
                } catch (IOException e) {
                }
            });

            System.out.println("目录拷贝成功");
        } catch (IOException e) {
            throw new RuntimeException("目录拷贝失败");
        }
    }

    // 获取文本文件的内容
    public static String readTextFile(String filePath) {
        try {
            List<String> lines = Files.readAllLines(Paths.get(filePath), StandardCharsets.UTF_8);
            return String.join("\n", lines);
        } catch (MalformedInputException e) {
            System.err.println("文件编码错误: " + e.getMessage());
            // 可以选择返回空字符串或其他默认值
            return "";
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }

    // 文件字节数组
    public static byte[] readVideoFile(String filePath) {
        try {
            return Files.readAllBytes(Paths.get(filePath));
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }

    /**
     * 将字节数组内容写入到指定的文件路径
     * 此方法主要用于将视频文件的字节数据保存到磁盘上的特定位置
     * 它使用Java NIO库来执行文件写入操作，简化了文件写入的复杂性
     *
     * @param filePath 指定保存文件的路径包括文件名
     * @param bytes    要写入文件的字节数组
     */
    public static void writeFile(String filePath, byte[] bytes) {
        try {
            Path path = Paths.get(filePath);
            if (!Files.exists(path)) {
                Files.createDirectories(path);
            }
            Files.write(path, bytes);
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }
}
```