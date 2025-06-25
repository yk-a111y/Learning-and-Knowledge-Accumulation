**no-bundle一定好吗？**

```ad-cmt
网络请求瀑布流问题 -> 深层依赖链会产生大量请求瀑布

HTTP/1.1 环境下存在并发连接数限制 -> 故可能存在大量文件请求的效率问题

动态导入在no-bundle下可能存在如下问题 -> 运行时路径解析困难、预加载困难

解决方法有：依赖预构建优化、HTTP2等。
```



**vite相比webpack的优点**
```ad-cmt
vite在dev环境下采取no-bundle的设计，使得冷启动相比于webpack来说较快。因为webpack需要全量编译所有Module，并在打包后运行。

实际环境下，vite配置相较于webpack简单，但除了对rollup插件的完美兼容外，其他生态相对匮乏。webpack配置复杂，但有相对成熟的loader生态。
```