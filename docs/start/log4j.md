# 启用 log4j 调试信息

## 调试

Notchian 客户端和服务器（以及启动器）可以生成正常显示的额外日志信息，其中一些信息对调试非常有帮助。这可以通过指定自定义 [Log4j 配置](http://logging.apache.org/log4j/2.x/manual/configuration.html) 来完成。

本地启动器提供了一个 "Log config" （日志配置）部分，可直接执行此操作。在其他情况下，可以通过游戏的 `-Dlog4j.configurationFile=fullpathtoconfigfile.xml` 参数来实现：

- [在 Java 启动器的 JVM 参数部分](https://wiki.vg/images/e/e1/ClientDebugging.png)
- 在服务器启动命令中的 `-jar` 之前
- 直接调用 `launcher.jar ` 时，在 `-jar ` 之前

当使用原生启动器时，请使用日志配置的 XML 变体，并勾选 "自定义日志配置输出 XML " 。这样可以使启动器在输出信息时更清晰。当使用 Java 启动器或启动服务器时，请使用非 XML 变体（不要在控制台中使用 XML ，而且 XML 变体不适用于服务器 GUI）。

!!! info "如果使用 `-Dlog4j.configurationFile` 指定的配置文件无法找到，将默认静默地使用默认配置文件。" 

您可以通过从您希望使用的 jar 中提取 `log4j2.xml` 来获得游戏的默认配置。

## 选项

在 log4j 配置中，只有少数几个选项具有有用的行为。

主要的是 `<Root>` 中的 `level`。通常，这个值设置为 `info`，但它可以接受 `trace`、`debug`、`info`、`warn`、`error`、`all` 和 `off`。Minecraft 实际使用的最低级别是 `debug`，因此为了获得最多的附加信息，应使用该级别。

另一个重要的选项是 `<filter>` 列表。默认情况下，游戏通过一个 `<filter>` 来过滤标记为 `NETWORK_PACKETS` 的所有日志消息。你可以改为只接受 `NETWORK_PACKETS` 并拒绝其他所有类型，或者完全移除 `<filter>`，以便显示所有消息。除了 `NETWORK_PACKETS` 外，还有几个其他的日志标记：`SOUNDS`、`NETWORK`、`PACKET_SENT` 和 `PACKET_RECEIVED`。`NETWORK` 和 `NETWORK_PACKETS` 的行为是一样的；`PACKET_SENT` 和 `PACKET_RECEIVED` 可以用来仅显示一个数据包集；而 `SOUNDS` 则用于输出声音通道启动和停止的信息（主要在诊断声音播放两次的问题时有用）。

另外需要注意的是，默认情况下，如果一天的日志超过7个，log4j 会覆盖旧日志（参见 [MC-100524](https://bugs.mojang.com/browse/MC-100524)）。你可以通过在 `<RollingRandomAccessFile>` 中添加 `<DefaultRolloverStrategy max="1000"/>` 来禁用这种行为，这样一天内的最大日志文件数就是1000个（这样做不太可能被达到，因为当数量变大时会影响性能）。

## 简单配置

**仅网络数据包**

*适用于：原版客户端和服务器*

生成服务器/客户端之间发送和接收的所有数据包的日志，**且仅此而已**（这也会禁用普通内容的日志记录，包括聊天内容）。不幸的是，自从 Netty 重写（Minecraft 1.7）后，只有数据包的 ID 和类会被记录，而不是它们的内容；然而，这些信息仍然可能有用。

!!! warning "数据包ID在日志中以十进制显示；它们不是十六进制的。"

=== "Config"

    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <Configuration status="WARN" packages="com.mojang.util">
        <Appenders>
            <Console name="SysOut" target="SYSTEM_OUT">
                <PatternLayout pattern="[%d{HH:mm:ss}] [%t/%level]: %msg%n" />
            </Console>
            <Queue name="ServerGuiConsole">
                <PatternLayout pattern="[%d{HH:mm:ss} %level]: %msg%n" />
            </Queue>
            <RollingRandomAccessFile name="File" fileName="logs/latest.log" filePattern="logs/%d{yyyy-MM-dd}-%i.log.gz">
                <PatternLayout pattern="[%d{HH:mm:ss}] [%t/%level]: %msg%n" />
                <Policies>
                    <TimeBasedTriggeringPolicy />
                    <OnStartupTriggeringPolicy />
                </Policies>
                <DefaultRolloverStrategy max="1000"/>
            </RollingRandomAccessFile>
        </Appenders>
        <Loggers>
            <Root level="debug">
                <filters>
                    <MarkerFilter marker="NETWORK_PACKETS" onMatch="ACCEPT" onMismatch="DENY" />
                </filters>
                <AppenderRef ref="SysOut"/>
                <AppenderRef ref="File"/>
                <AppenderRef ref="ServerGuiConsole"/>
            </Root>
        </Loggers> 
    </Configuration>
    ```

=== "Config (XML output)"

    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <Configuration status="WARN">
        <Appenders>
            <Console name="SysOut" target="SYSTEM_OUT">
                <XMLLayout />
            </Console>
            <RollingRandomAccessFile name="File" fileName="logs/latest.log" filePattern="logs/%d{yyyy-MM-dd}-%i.log.gz">
                <PatternLayout pattern="[%d{HH:mm:ss}] [%t/%level]: %msg%n" />
                <Policies>
                    <TimeBasedTriggeringPolicy />
                    <OnStartupTriggeringPolicy />
                </Policies>
                <DefaultRolloverStrategy max="1000"/>
            </RollingRandomAccessFile>
        </Appenders>
        <Loggers>
            <Root level="debug">
                <filters>
                    <MarkerFilter marker="NETWORK_PACKETS" onMatch="ACCEPT" onMismatch="DENY" />
                </filters>
                <AppenderRef ref="SysOut"/>
                <AppenderRef ref="File"/>
            </Root>
        </Loggers> 
    </Configuration>
    ```

## 所有调试信息

*适用于：原版客户端和服务器*

你可以记录更多的信息。这样不仅会打印常规的日志消息（如聊天信息），还会记录网络数据包、声音的开始/停止播放（仅客户端）、一些身份验证请求和其他杂项信息。

=== "Config"
    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <Configuration status="WARN" packages="com.mojang.util">
        <Appenders>
            <Console name="SysOut" target="SYSTEM_OUT">
                <PatternLayout pattern="[%d{HH:mm:ss}] [%t/%level]: %msg%n" />
            </Console>
            <Queue name="ServerGuiConsole">
                <PatternLayout pattern="[%d{HH:mm:ss} %level]: %msg%n" />
            </Queue>
            <RollingRandomAccessFile name="File" fileName="logs/latest.log" filePattern="logs/%d{yyyy-MM-dd}-%i.log.gz">
                <PatternLayout pattern="[%d{HH:mm:ss}] [%t/%level]: %msg%n" />
                <Policies>
                    <TimeBasedTriggeringPolicy />
                    <OnStartupTriggeringPolicy />
                </Policies>
                <DefaultRolloverStrategy max="1000"/>
            </RollingRandomAccessFile>
        </Appenders>
        <Loggers>
            <Root level="debug">
                <AppenderRef ref="SysOut"/>
                <AppenderRef ref="File"/>
                <AppenderRef ref="ServerGuiConsole"/>
            </Root>
        </Loggers>
    </Configuration>
    ```

=== "Config (XML output)"
    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <Configuration status="WARN">
        <Appenders>
            <Console name="SysOut" target="SYSTEM_OUT">
                <XMLLayout />
            </Console>
            <RollingRandomAccessFile name="File" fileName="logs/latest.log" filePattern="logs/%d{yyyy-MM-dd}-%i.log.gz">
                <PatternLayout pattern="[%d{HH:mm:ss}] [%t/%level]: %msg%n" />
                <Policies>
                    <TimeBasedTriggeringPolicy />
                    <OnStartupTriggeringPolicy />
                </Policies>
                <DefaultRolloverStrategy max="1000"/>
            </RollingRandomAccessFile>
        </Appenders>
        <Loggers>
            <Root level="debug">
                <AppenderRef ref="SysOut"/>
                <AppenderRef ref="File"/>
            </Root>
        </Loggers>
    </Configuration>
    ```

## CraftBukket / Spigot上的所有调试信息

*适用于：CraftBukkit 服务器和衍生自 CraftBulkit 的服务器*

由于 CraftBukkit 使用略有不同的控制台输出（并且没有服务器GUI），因此需要稍微调整配置以正确显示。但是，这样做也将显示重新映射数据包类名。

=== "Config"
    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <Configuration status="WARN" packages="net.minecraft,com.mojang">
        <Appenders>
            <Console name="WINDOWS_COMPAT" target="SYSTEM_OUT"></Console>
            <Queue name="TerminalConsole">
                <PatternLayout pattern="[%d{HH:mm:ss} %level]: %msg%n" />
            </Queue>
            <RollingRandomAccessFile name="File" fileName="logs/latest.log" filePattern="logs/%d{yyyy-MM-dd}-%i.log.gz">
                <PatternLayout pattern="[%d{HH:mm:ss}] [%t/%level]: %msg%n" />
                <Policies>
                    <TimeBasedTriggeringPolicy />
                    <OnStartupTriggeringPolicy />
                </Policies>
                <DefaultRolloverStrategy max="1000"/>
            </RollingRandomAccessFile>
        </Appenders>
        <Loggers>
            <Root level="debug">
                <AppenderRef ref="WINDOWS_COMPAT"/>
                <AppenderRef ref="File"/>
                <AppenderRef ref="TerminalConsole"/>
            </Root>
        </Loggers>
    </Configuration>
    ```

## 启动器会话信息

*适用于：Launcher.jar*

通过在（Java 启动器的）launcher.jar上启用调试启动，您可以到 [Mojang API](https://wiki.vg/Mojang_API) 查看 - 如果您想为自己的帐户查看到每个端点的示例连接，则会有所帮助。

请注意，如果您调用正常的启动器 EXE/JAR，则此配置 **可能** 不起作用，而是在位于`%appdata%\.minecraft`中的 `launcher.jar` 上使用它。

=== "Config"
    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <Configuration status="WARN" packages="com.mojang">
        <Appenders>
            <Console name="SysOut" target="SYSTEM_OUT">
                <PatternLayout pattern="[%d{HH:mm:ss} %level]: %msg%n" />
            </Console>
            <Queue name="DevelopmentConsole">
                <PatternLayout pattern="[%d{HH:mm:ss} %level]: %msg%n" />
            </Queue>
            <Async name="Async">
                <AppenderRef ref="SysOut"/>
                <AppenderRef ref="DevelopmentConsole"/>
            </Async>
        </Appenders>
        <Loggers>
            <Root level="debug">
                <AppenderRef ref="Async"/>
            </Root>
        </Loggers>
    </Configuration>
    ```

