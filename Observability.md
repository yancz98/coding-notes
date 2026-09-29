# 一、可观测性概念

## 1.1、可观测性入门

### 1.1.1、起源

随着微服务、容器技术、云原生技术的发展，使得在这些分布式系统中追踪事件的起源变得非常困难，监控技术和工具的革新迫在眉睫。而可观测性一词最初是由 CNCF 在云原生定义中提到的 Observability，并声称这是云原生时代的必备能力。

面对这样的问题，Google 发表了论文 [《Dapper, a Large-Scale Distributed Systems Tracing Infrastructure》](https://static.googleusercontent.com/media/research.google.com/zh-CN//archive/papers/dapper-2010-1.pdf)，介绍了他们的分布式跟踪技术，并认为分布式跟踪系统应该满足以下业务需求：

- 无处不在的部署
- 持续的监控
- 低消耗
- 应用级透明
- 延展性
- 低延迟

除了以上要求，该论文也针对分布式追踪的数据采集、数据持久化、数据展示三个核心环节进行了完整阐述。数据采集指在代码中埋点，设置请求中需要上报的内容。数据持久化指将上报的数据落盘存储。数据展示则是根据 TraceID 查询与之关联的请求在界面上呈现。

### 1.1.2、什么是可观测性

可观测性让我们从外部了解一个系统，让我们在不了解其内部工作的情况下询问有关该系统的问题。此外，它使我们能够轻松地排除和处理新问题，并帮助我们回答“为什么会发生这种情况？”。

为了能够向系统提出这些问题，必须对应用程序进行适当的检测。也就是说，应用程序代码必须发出 traces、metrics、logs 信号。

可观测性指的是一种能力，是通过检查其输出，来衡量系统内部状态的能力。这些输出体现系统内部状态的能力越强，可观测性也就越好。

Google 给出可观测性的核心价值很简单：快速排障。

在 CNCF 对于云原生的定义中，已经明确将可观测性列为一项必备要素。

### 1.1.3、可靠性和指标

Telemetry：是指系统发出的有关其行为的数据。数据可以采用 traces、metrics、logs 的形式。

Reliability：可靠性回答了一个问题：“服务正在做用户期望的事情吗？”。

Metrics：是一段时间内有关基础设施或应用程序的数字数据的聚合。示例包括：系统错误率、CPU 利用率，给定服务的请求速率。

SLI（Service Level Indicator）：服务级别指示器，表示对服务的度量行为。一个好的 SLI 从用户的角度来衡量你的服务。示例 SLI 可以是网页的加载速度。

SLO（Service Level Objective）：服务级别目标，是将可靠性传达给组织/其它团队的手段。这是通过将一个或多个 SLI 附加到业务价值上来实现的。

### 1.1.4、分布式跟踪

Logs：是由服务或其他组件发出的带有时间戳的消息。然而，与 Traces 不同，它们不一定与任何特定的用户请求或事务相关联。 

Span：表示一个工作或操作单元。它跟踪请求执行的特定操作，描绘执行该操作期间发生的事情。

一个 Span 包含名称、与时间相关的数据、结构化日志消息和其他元数据（属性），以提供有关其跟踪的操作的信息。



一个 **Span** 代表一个作业或者操作的单元，是链路的组成部分。在 OpenTelemetry 中，它们包括以下信息：

-   名字
-   父 Span ID（根 Span 为空）
-   开始和结束时间
-   上下文（context）：
-   属性（attributes）：是附加在 span 上的额外信息，为我们提供更多关于操作上下文的细节。
-   事件（event）：通常用于表示 Span 持续时间内有意义的单个时间点（可视为 Span 上的结构化日志消息）。
-   Span 链接
-   Span 状态：
    -   Unset：表示一个 Span 完成且没有错误（默认）
    -   Error：跟踪的操作中发生了一些错误
    -   OK：表示开发人员在某刻明确将 Span 显式标记为 OK。
-   Span 类型：Client、Server、Internal（默认）、Producer、Consumer。

```
{
  "name": "/v1/sys/health",
  "context": {
    "trace_id": "7bba9f33312b3dbb8b2c2c62bb7abe2d",
    "span_id": "086e83747d0e381e"
  },
  "parent_id": "",
  "start_time": "2021-10-22 16:04:01.209458162 +0000 UTC",
  "end_time": "2021-10-22 16:04:01.209514132 +0000 UTC",
  "status_code": "STATUS_CODE_OK",
  "status_message": "",
  "attributes": {
    "net.transport": "IP.TCP",
    "net.peer.ip": "172.17.0.1",
    "net.peer.port": "51820",
    "net.host.ip": "10.177.2.152",
    "net.host.port": "26040",
    "http.method": "GET",
    "http.target": "/v1/sys/health",
    "http.server_name": "mortar-gateway",
    "http.route": "/v1/sys/health",
    "http.user_agent": "Consul Health Check",
    "http.scheme": "http",
    "http.host": "10.177.2.152:26040",
    "http.flavor": "1.1"
  },
  "events": [
    {
      "name": "",
      "message": "OK",
      "timestamp": "2021-10-22 16:04:01.209512872 +0000 UTC"
    }
  ]
}
```



Traces：记录请求（由应用程序或最终用户发出）在多服务架构（如微服务和无服务器应用程序）中传播时所采用的路径。

Trace 提高了应用程序或系统运行状况的可见性，并使我们能够调试难以在本地重现的行为。追踪对于分布式系统至关重要，因为分布式系统通常存在不确定性问题，或过于复制，无法在本地复制。

在没有 Trace 的情况下，很难确定分布式系统中性能问题的原因。Trace 通过分解请求在分布式系统中流动时发生的事情，使调试和理解分布式系统变得不那么困难。

Trace 由一个或多个 Span 组成。第一个 Span 表示根 span。 每个根 span 表示从开始到结束的一个请求。父级下面的 span 提供了一个更深入的上下文，说明请求过程中发生了什么（或组成请求的步骤）。

### 1.1.5、上下文传播

通过上下文传播，无论信号在何处生成，信号彼此之间都可以相互关联。 尽管上下文传播的作用不仅限于链路追踪， 但上下文传播允许链路跨进程和网络边界任意分布的服务来构建相关系统的关联信息。

上下文（Context）：包含了发送和接收服务或执行单元的信息， 用于将一个信号与另一个信号关联起来。

传播（Propagation）：是上下文在服务和进程之间移动的机制。 它序列化或反序列化上下文对象并提供要从一个服务传播到另一个服务的相关信息。

## 1.2、可观测性的三大支柱（Signals）

### 1.2.1、日志：Logs

⽇志是在特定时间发⽣的事件的⽂本记录，包括说明事件发⽣时间的时间戳和提供上下⽂的有效负载。⽇志有三种格式：纯⽂本（非结构化）、结构化、⼆进制（binlog）。纯⽂本是最常⻅的，但结构化⽇志（包括额外的数据和元数据并且更容易查询）正变得越来越流⾏。当系统出现问题时，⽇志通常也是您⾸先查看的地方。

### 1.2.2、指标：Metrics

指标是与时间戳关联的一系列数据点，这导致了“时间序列”通常被认为是“指标”的同义词。

指标是在⼀段时间内测量的数值，包括特定属性（如：时间戳、名称、KPI）和值，指标采集的时间跨度决定了指标的细粒度，指标在默认情况下是结构化的，这使得查询和优化存储变得更加容易，让您能够将它们保留更⻓时间。

由于指标往往包含比日志更不敏感的数据，因此基础设施提供商和第三方服务更常见地提供有关他们代表用户执行的操作的指标，而不是日志。

### 1.2.3、追踪：Traces

跟踪表示请求通过分布式系统的端到端旅程。当请求通过主机系统时， 对其执⾏的每个操作（称为“跨度”）都使⽤与执⾏该操作的微服务相关的重要数据进⾏编码。通过查看跟踪，每个跟踪都包含⼀个或多个跨度，您可以通过分布式系统跟踪其进程并确定瓶颈或故障的原因。

跨度（span）通常包括以下数据：

- trace_id
- parent_id
- span_id
- 操作的名称
- 开始和结束时间戳
- 以键值格式编码的元数据，包括有关基础设施的信息（如：哪个容器处理了此请求）。
- 事件（如：日志、异常、错误）

追踪的价值不仅限于故障排除单个请求。如：通过汇总多个追踪中的数据，可以生成关于速率、错误和持续时间（RED）等指标的数据，这些指标是站点可靠性工程（SRE）实践中的所谓“黄金信号”的重要组成部分，正如 Google 最初定义的那样。

从一个简单的“系统探查--日志搜集--日志统计”流程来看：三者之间的关系从 traces 开始，通过探查等手段采集众多信息，形成 logs，logs 详细记录了各种边界行为（如登陆，开启服务，关闭服务，退出系统，修改数据），而对于系统运行来说，更重要的是特定事件发生的次数。这些信息可以从日志中提取，但是有一种更有效的方法：metrics。至此，如果你的 metrics 与告警相关联，则可在系统关键节点设置阈值。如果指标超过了阈值，值班人员就会收到消息，快速响应排出故障。

## 1.3、可观测数据的标准

随着 Dapper 论文的诞生，分布式追踪技术逐渐兴起。相关产品相继涌现，Uber 的 Jaeger、Twitter 的 ZipKin 等分布式追踪产品名声大噪。但在这个过程中也带来了一个问题：虽然每个产品都有自己的一套数据采集标准和 SDK，且大多都是基于 Dapper 协议，但是实现不尽相同，导致不同的分布式追踪系统 API 不兼容的问题。

[![OpenTracing](https://img-blog.csdnimg.cn/1314d063aa94479aa6d3497540a51bb4.png)](https://opentracing.io/)

### 1.3.1、OpenTracing

OpenTracing 的优势在于制定了一套厂商无关、平台无关的协议标准，使开发人员只需要修改 Tracer 就可以更迅捷的添加或更换底层监控的实现。也是基于这一点，2016 年云原生计算基金会 CNCF 正式接纳 OpenTracing，顺利成为 CNCF 第三个项目。

OpenTracing 由 API 规范、实现该规范的框架和库，以及项目文档组成，并进行以下努力：

- 后台无关的 API 接口标准化：被追踪的服务只需要调用相关 API 接口，就可被任何实现这套接口的追踪后台支持。
- 对跟踪最小单位 Span 管理标准化：定义开始 Span，结束 Span 和记录 Span 耗时的 API。
- 进程间跟踪数据传递方式标准化：定义 API 方便追踪数据的传递。
- 对多语言应用支持的标准化：全面覆盖 GO、Python、Javascript、Java、C#、Objective-C、C++ 、Ruby、PHP 等开发语言。它支持 Zipkin、LightStep、Appdash 跟踪器，并可以轻松集成到 GRPC、Flask、DropWizard、Django、Go-Kit 等框架中。

遵循 OpenTracing 协议的产品有 Jaeger、Zipkin、 LightStep 和 AppDash 等追踪组件。

[![OpenCensus](https://img-blog.csdnimg.cn/e9b7ec0d73c8478a9dae22bf08963b08.png)](https://opencensus.io/)

### 1.3.2、OpenCensus

在整个可观测领域，为了更好的实现 DevOps，除了分布式追踪 Trace，运维人员开始关注 Log 和 Metrics。Metrics 指标监控作为可观测的重要组成部分，包括 CPU、内存、硬盘、网络等机器指标，gRPC 请求延迟、错误率等网络协议指标，用户数、访问数等业务指标。

OpenCensus 提供了统一的测量工具：跨服务捕获跟踪跨度 Span、应用级别指标 Metrics。

优势：

- 相较于 OpenTracing 只支持 Traces，OpenCensus 支持 Traces 和 Metrics。
- 相较于 OpenTracing 制定规范，OpenCensus 不仅制定规范，还包含了 Agent 和 Collector。
- 家属团阵容相较 OpenTracing 更加庞大，获得 Google、微软支持。

功能：

- 标准通信协议和一致的 API：用于处理 Metrics 和 Trace。
- 多语言库支持：Java、C++、Go、.Net、Python、PHP、Node.js、Erlang 、Ruby。
- 与 RPC 框架的集成。
- 集成存储和分析工具。
- 完全开源并支持三方集成和输出的插件化。
- 不需要额外服务器或 Agent 来支持 OpenCensus。

遵循 OpenCensus 协议的产品有 Prometheus、SignalFX、Stackdriver 和 Zipkin。

OpenTracing 和 OpenCensus 从功能、特性等维度来评估，两者有明显优缺点：OpenTracing 支持语言更多、对其他系统耦合性更低；OpenCensus 支持 Metrics、Trace，同时从 API 层一直到基础设施层都进行支持。

能否有一个能够将 OpenTracing 和 OpenCensus 的优点结合，并且能够支持日志相关可观测数据的项目呢？



[![OpenTelemetry](https://img-blog.csdnimg.cn/2ae6cb5789904a1783b200581b2e39fc.png)](https://opentelemetry.io/)

### 1.3.3、OpenTelemetry

> **OpenTelemetry = OpenTracing + OpenCensus**

为了更好的将 Traces、Metrics、Logs 融合在一起，OpenTelemetry 诞生了。作为 CNCF 的孵化项目，OpenTelemetry 由 OpenTracing 和 OpenCensus 项目合并而成，是一组规范、API 接口、SDK、工具和集成。为众多开发人员带来 Metrics、Tracing、Logs 的统一标准，三者都有相同的元数据结构，可以轻松实现互相关联。

OpenTelemetry 与厂商、平台无关，不提供与可观测性相关的后端服务。OpenTelemetry 专注于生成、收集、管理和导出 telemetry data。可根据用户需求将可观测类数据导出到存储、查询、可视化等不同后端，如 Prometheus、Jaeger 、云厂商服务中。

优势：

- 完全打破各个厂商的 Lock-on 隐患。
- 规范的指定和协议的统一。
- 多语言 SDK 的实现和集成。
- 数据收集系统的实现
- 自动代码注入技术
- 云原生架构

OpenTelemetry 的核心工作目前主要集中在3个部分：

- 规范的制定和协议的统一，规范包含数据传输、API 的规范，协议的统一包含：HTTP W3C 的标准支持及 GRPC 等框架的协议标准。
- 多语言 SDK 的实现和集成，用户可以使用 SDK 进行代码自动注入和手动埋点，同时对其他三方库（Log4j、LogBack 等）进行集成支持。
- 数据收集系统的实现，当前是基于 OpenCensus Service 的收集系统，包括 Agent 和 Collector。

OpenTelemetry 的定位：

OpenTelemetry 要解决的是可观测性数据统一的第一步，通过 API 和 SDK 来标准化可观测数据的采集和传输，即数据采集和标准规范的统一。对于后续如何存储、利用这些数据进行分析、告警等，需要根据实际情况进行选型。

推荐使用 Prometheus + Grafana 做 Metrics 存储和展示，使用 Jaeger 做分布式跟踪的存储和展示。





# 二、日志：Logs

对各个应用中产生的日志进行收集并提供查询能力。



# 三、指标：Metrics

[Prometheus](Prometheus_9090.md)

推荐使用 **Prometheus + Grafana **做 **Metrics** 存储、展示。

# 四、追踪：Traces

推荐使用 **Jaeger** 做分布式跟踪的存储和展示。

前面介绍的 Logs 系统使用的是开发者打印的日志，所以它是最贴近业务的。而 Traces 系统就离业务更远一些了，它关注的是一个请求进来以后，经过了哪些应用、哪些方法，分别在各个节点耗费了多少时间，在哪个地方抛出的异常等，用来快速定位问题。



### 4.1、OpenTelemetry 采集



### 4.2、Jaeger 



### 4.3、ZipKin

 

```
HTTP Request → Gin Handler → SendMessageWithContext (Inject) 
                                           ↓
                                    Kafka Message (带 trace headers)
                                           ↓
                         ConsumerGroup (Extract) → processMessage → Span
```

![image-20260305202251576](C:/Users/sh/AppData/Roaming/Typora/typora-user-images/image-20260305202251576.png)