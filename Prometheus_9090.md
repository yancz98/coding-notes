

# [Prometheus](https://prometheus.io/)

## 1.1、介绍

### 1.1.1、特性

Prometheus 是一个开源的系统监控和警报平台，通过抓取被监控目标上的 HTTP 端点来收集这些目标的指标。并将指标存储为时间序列数据，即指标信息与记录它的时间戳一起存储，以及称为标签的可选键值对。

- 一个由指标名称和键值对标识的时间序列数据的多维度数据模型。
- PromQL，一种利用这种维度的灵活查询语言。
- 不依赖分布式存储，单服务器节点是自治的。
- 时间序列收集是通过 HTTP 上的 pull 模型进行的。
- 通过中间网关支持 push 时间序列。
- 通过服务发现或静态配置发现目标。
- 多种绘图和仪表板支持模式。

> Prometheus 与其它监控系统对比

|             | 开发语言 | 成熟度 | 扩展性 | 高性能 | 社区活跃度 | 容器支持 | 企业使用情况 | 部署和维护成本 |
| ----------- | -------- | ------ | ------ | ------ | ---------- | -------- | ------------ | -------------- |
| Zabbix      | C & PHP  | 高     | 高     | 低     | 中         | 低       | 高           | 中             |
| Open-Falcon | Go       | 中     | 中     | 高     | 中         | 中       | 中           | 高             |
| Prometheus  | Go       | 中     | 高     | 高     | 高         | 高       | 高           | 低             |

### 1.1.2、架构

![Prometheus architecture](https://prometheus.io/assets/docs/architecture.svg)

Prometheus 直接从仪表化作业中抓取指标，直接或通过中间 PushGateway 用于短期作业。它在本地存储所有抓取的样本，并对这些数据运行规则，以从现有数据中聚合和记录新的时间序列或生成警报。Grafana 或其他 API 消费者可用于可视化收集的数据。

组件：

- Pushgateway：用于支持短期作业的推送网关。
- Exporters：HAProxy、StatsD、Graphite 等服务的专用出口商。
- ClientLibraries：用于检测应用程序代码的客户端库。
- PrometheusServer：用于抓取和存储时间序列数据。
- AlertManager：用于处理警报的警报管理器。
- 其它各种支持的工具。

大多数 Prometheus 组件都是用 Go 编写的。

### 1.1.3、适用场景

Prometheus 非常适合记录任务纯数字时间序列。它既适用于以机器为中心的监控，也适用于高度动态的面向服务架构的监控。在微服务的世界里，它对多维数据收集和查询的支持是一个特别的优势。

Prometheus 专为可靠性而设计，在系统中断期间可以快速诊断问题。每个 Prometheus 服务都是独立的，不依赖于网络存储或其他远程服务。

Prometheus 不适用于收集严格精准的数据（如，按请求计费的数据），因为 Prometheus 收集的数据可能不够详细和完整。

### 1.1.4、指标（metrics）类型

Prometheus 支持四种类型的指标，分别是： Counter - Gauge - Histogram - Summary。

- Counter：计数器是一个只能增加或重置的度量值，即该值不能比前一个值减少。它可以用于请求数量、错误数等指标。

    ```
    # HTTP 请求总数，只能增加
    # HELP prometheus_http_requests_total Counter of HTTP requests.
    # TYPE prometheus_http_requests_total counter
    prometheus_http_requests_total{code="200",handler="/metrics"}	40741
    ```

- Gauge：计量器是一个可以上升或下降的数字。它可以用于衡量指标，如集群中的 Pod 数量、队列中的事件数量等。

    ```
    # 创建的操作系统线程数，可增可减
    # HELP go_threads Number of OS threads created.
    # TYPE go_threads gauge
    go_threads 23
    ```

- Histogram：柱状图/直方图是一种更复杂的度量类型。直方图可用于任何计算值，这些值根据桶值进行计数。存储桶边界可由开发人员配置。一个常见的示例是回复请求所需的时间，称为延迟。

    ```
    # HTTP 请求的延迟直方图
    # HELP prometheus_http_request_duration_seconds Histogram of latencies for HTTP requests.
    # TYPE prometheus_http_request_duration_seconds histogram
    prometheus_http_request_duration_seconds_bucket{handler="/metrics",le="0.1"} 40741
    prometheus_http_request_duration_seconds_bucket{handler="/metrics",le="0.2"} 40741
    prometheus_http_request_duration_seconds_bucket{handler="/metrics",le="0.4"} 40741
    prometheus_http_request_duration_seconds_bucket{handler="/metrics",le="1"} 40741
    prometheus_http_request_duration_seconds_bucket{handler="/metrics",le="3"} 40741
    prometheus_http_request_duration_seconds_bucket{handler="/metrics",le="8"} 40741
    prometheus_http_request_duration_seconds_bucket{handler="/metrics",le="20"} 40741
    prometheus_http_request_duration_seconds_bucket{handler="/metrics",le="60"} 40741
    prometheus_http_request_duration_seconds_bucket{handler="/metrics",le="120"} 40741
    prometheus_http_request_duration_seconds_bucket{handler="/metrics",le="+Inf"} 40741
    prometheus_http_request_duration_seconds_sum{handler="/metrics"} 303.65504988400164
    prometheus_http_request_duration_seconds_count{handler="/metrics"} 40741
    ```

- Summary：摘要还可以衡量事件，是直方图的替代方法。它更低廉但会丢失更多数据。它是在应用程序级别计算的，因此无法聚合来自同一流程的多个实例的指标。当事先不知道指标的存储桶时会使用它，但强烈建议尽可能使用 Histogram 而不是 Summary。

    ```
    # 目标抓取之间的实际间隔
    # HELP prometheus_target_interval_length_seconds Actual intervals between scrapes.
    # TYPE prometheus_target_interval_length_seconds summary
    prometheus_target_interval_length_seconds{interval="15s",quantile="0.01"} 14.999205085
    prometheus_target_interval_length_seconds{interval="15s",quantile="0.05"} 14.999382841
    prometheus_target_interval_length_seconds{interval="15s",quantile="0.5"} 14.999982114
    prometheus_target_interval_length_seconds{interval="15s",quantile="0.9"} 15.000528604
    prometheus_target_interval_length_seconds{interval="15s",quantile="0.99"} 15.000956215
    prometheus_target_interval_length_seconds_sum{interval="15s"} 1.222185769867555e+06
    prometheus_target_interval_length_seconds_count{interval="15s"} 81479
    ```


### 1.1.5、向量（vector）类型

向量（vector）：按标签的维度对样本进行划分的指标（metrics）。

- CounterVec
- GaugeVec
- HistogramVec
- SummaryVec



## 1.2、安装 & 配置

二进制文件下载：https://prometheus.io/download/

Docker 镜像下载：[Quay Container Registry](https://quay.io/organization/prometheus) （或 Docker Hub）

### 1.2.1、Prometheus

```sh
$ wget https://github.com/prometheus/prometheus/releases/download/v2.53.1/prometheus-2.53.1.linux-amd64.tar.gz
$ tar xvfz prometheus-2.53.1.linux-amd64.tar.gz
$ cd prometheus-2.53.1.linux-amd64
$ ll
drwxr-xr-x 2 dev dev      4096 7月  10 18:31 console_libraries/
drwxr-xr-x 2 dev dev      4096 7月  10 18:31 consoles/
-rw-r--r-- 1 dev dev     11357 7月  10 18:31 LICENSE
-rw-r--r-- 1 dev dev      3773 7月  10 18:31 NOTICE
-rwxr-xr-x 1 dev dev 137823986 7月  10 18:18 prometheus*
-rw-r--r-- 1 dev dev       934 7月  10 18:31 prometheus.yml
-rwxr-xr-x 1 dev dev 129719462 7月  10 18:18 promtool*

# 启动服务
$ ./prometheus --config.file=prometheus.yml

# 访问：http://localhost:9090

# 重新加载配置（不用重启）
$ kill -s SIGHUP <PID>

# 优雅的关闭
$ kill -s SIGTERM <PID>
```

> Docker

```sh
# 由于 Docker Hub 在国内不可用（prom/prometheus）
$ docker pull quay.io/prometheus/prometheus

$ docker run --name prometheus -d \
  -u root \
  --restart always \
  -p 9090:9090 \
  -v /data/docker-run/prometheus/prometheus.yml:/etc/prometheus/prometheus.yml \
  -v /data/docker-run/prometheus/data/:/prometheus \
  quay.io/prometheus/prometheus
```

> 配置文件：prometheus.yml

```yaml
# my global config
global:
  # 控制 Prometheus 抓取目标的频率
  scrape_interval: 15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  # 控制 Prometheus 评估规则的频率（生成警报）
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

# 警报配置
# Alertmanager configuration
alerting:
  alertmanagers:
    - static_configs:
        - targets:
          # - alertmanager:9093

# 规则配置
# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  # - "first_rules.yml"
  # - "second_rules.yml"

# 控制 Prometheus 监控的资源（抓取目标配置）
# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  # 第一个抓取对象 `job="prometheus"`
  # 它抓取 Prometheus 服务器公开的时间序列数据，用于监控 Prometheus 自己的健康状况。
  - job_name: "prometheus"

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.
	# 静态配置的目标：localhost:9090 （Prometheus web UI）
	# Prometheus 希望 metrics 在目标的 /metrics 路径上可用
	# 因此该 Job 通过 http://localhost:9090/metrics 抓取指标
	# 直接访问 http://localhost:9090/metrics 可查看 Prometheus 自身的指标
    static_configs:
      - targets: ["localhost:9090"]
```

### 1.2.2、基本身份验证

> 默认 Prometheus web UI 不需要身份验证。
>
> Prometheus 可通过加载 Web 配置文件，来支持 Web UI 和 API 的基本身份验证。

```
# Usernames and hashed passwords that have full access to the web
# server via basic authentication. If empty, no basic authentication is
# required. Passwords are hashed with bcrypt.
basic_auth_users:
  [ <string>: <secret> ... ]
```

> 使用 python3-bcrypt 生成 password 的 bcrypt 哈希值。

```sh
$ sudo apt install python3-bcrypt
$ python3 --version
Python 3.8.10
$ vim gen-pass.py
import getpass
import bcrypt

password = getpass.getpass("password: ")
hashed_password = bcrypt.hashpw(password.encode("utf-8"), bcrypt.gensalt())
print(hashed_password.decode())

$ python3 gen-pass.py 
password:   # 123456
$2b$12$yJJk9DWzI/eUDWifXe31BuU/oiVzdzCBmMU9Iq97xp2Sjd2IiV31a
```

> web-config.yml & prometheus.yml 配置文件

```sh
# web-config.yml
$ vim web-config.yml
basic_auth_users:
  admin: $2b$12$1FqzIdYn.GwgqKw7yQTs8ef6MxoyyAV0jrx2PiyYSgzTgOXzKI5rS
  
$ promtool check web-config web-config.yml
web-config.yml SUCCESS

# prometheus.yml 的抓取配置
scrape_configs:
  - job_name: "prometheus"
  
    # Sets the `Authorization` header on every scrape request with the
    # configured username and password.
    # password and password_file are mutually exclusive.
    basic_auth:
      username: admin
      password: 123456

    static_configs:
      - targets: ["localhost:9090"]

# 启动 Prometheus 服务
$ prometheus --config.file=prometheus.yml --web.config.file=web.yml
```

### 1.2.3、[配置]([Configuration | Prometheus](https://prometheus.io/docs/prometheus/latest/configuration/configuration/))



## 1.3、PromQL

PromQL（Prometheus Query Language）是 Prometheus 提供的一种函数式查询语言，允许用户实时选择和聚合时间序列数据。表达式结果既可以显示为图形，也可以在 Prometheus 的表达式浏览器中显示为表格数据，还可以通过 HTTP API 由外部系统使用。

> 注：PromeQL 语句可以在 Prometheus 的 [Graph](http://localhost:9090/graph) 页面验证及查看查询结果。

### 1.3.1、表达式语言数据类型

- 数据类型：

    - Instant vector（即时/瞬时向量）：一组时间序列，包含每个时间序列的单个样本，所有样本共享相同的时间戳。（唯一可以绘制图表的类型）

    - Range vector（范围向量）：一组时间序列，包含每个时间序列随时间变化的数据点范围。

- 数据格式：

    - Scalar（标量）：一个简单的数字浮点值。

    - String：一个简单的字符串值。（目前未使用）


### 1.3.2、时间序列选择器

时间序列选择器负责选择时间序列和原始或推断的样本时间戳和值。

- 瞬时向量选择器：允许在选择一组时间序列和一个在给定时间戳（时间点）下每个序列的单个样本值。

```PromQL
# ==============
#  即时向量选择器 
# ==============

# 语法（没有过滤条件时，花括号可省略）：
metrics
metrics{label='values'}

# 例1：匹配所有时间序列
# 仅指定指标名称，产生一个包含所有时间序列元素的即时向量
# 选择 prometheus_http_requests_total 的所有时间序列
> prometheus_http_requests_total{}
...


##### 标签匹配运算符 #####

  =   选择与提供的字符串完全相等的标签。
  !=  选择不等于提供的字符串的标签。
  =~  选择与提供的字符串进行正则表达式匹配的标签。
  !~  选择与提供的字符串不匹配的标签。



# 例2：通过标签匹配过滤
# 匹配 job = 'prometheus' & code != 200
> prometheus_http_requests_total{job='prometheus', code!='200'}
...

# 匹配 code 不是 4xx 和 5xx 的数据
> prometheus_http_requests_total{job='prometheus', code!~'4..|5..'}
...

# 例3：内部标签匹配
> {__name__='prometheus_http_requests_total', code='200'}
...

# 内部标签匹配常用于指标名称为关键字时 (bool, on, ignoring, group_left, group_right)
# 例：{__name__='on'}
```

- 范围向量选择器：与即时向量选择器类似，它从当前即时选择一个样本范围。

```
# ==============
#  范围向量选择器
# ==============

  # 语法：在向量选择器末尾的方括号中附加一个持续时间
  # 该时间范围是一个左右闭区间
  metrics[duration]
  metrics{}[duration]

# 例1：
# 选择过去5分钟内记录的所有值
> prometheus_http_requests_total{job='prometheus', code!='200'}[5m]
...

# ============
#  持续时间单位
# ============

  ms - 毫秒
  s  - 秒
  m  - 分
  h  - 小时
  d  - 天
  w  - 周
  y  - 年

# 持续时间组合（单位必须从最长到最短，且每个单位只能出现一次）
# 例：1小时30分
[1h30m]

# ==============
#  offset 修饰符 
# ==============

  该 offset 修饰符允许更改查询中单个即时向量和范围向量的时间偏移。

  # 语法：
  metrics{} offset <duration>

# 例1：即时向量
# 返回相对于当前查询计算时间前 5m 的值
> prometheus_http_requests_total{} offset 5m

# 例2：范围向量
# 查询 1d 前的 5m 内的平均请求率
> rate(prometheus_http_requests_total{}[5m] offset 1d)

# 例3：负偏移量
# 允许查询提前查看其评估时间
> rate(prometheus_http_requests_total{}[5m] offset -1w)


# =========
#  @ 修饰符
# =========

  该 @ 修饰符允许更改查询中单个瞬时向量和范围向量的评估时间。

  # 语法：
  metrics{} @ UnixTimestamp

# 例1：
# 返回瞬时的值（2024-08-06 14:30:00）
> prometheus_http_requests_total{} @ 1722925800

# 例2：范围向量
# 返回时间（2024-08-06 14:30:00）的 5m 内平均请求率
> prometheus_http_requests_total{}[5m] @ 1722925800

# 例3：offset 与 @ 修饰符组合（顺序无关）
# 其中 offset 是现对于 @ 时间应用的，以下结果相同
> prometheus_http_requests_total{} @ 1722925800 offset 5m
> prometheus_http_requests_total{} offset 5m @ 1722925800

# 例4：@ 的特殊值 start() 和 end()
# 对于范围查询，它们分别解析到范围查询的开始和结束，并在所有步骤中保持不变。
# 对于即时查询，start() 和 end() 都解析为评估时间。
> prometheus_http_requests_total @ start()
> prometheus_http_requests_total @ end()
```

### 1.3.3、子查询

子查询允许您针对给定的范围和分辨率运行即时查询。子查询的结果是范围向量。

```
# 语法：
# <resolution> 是可选的。默认值为全局评估间隔。
<instant_query> '[' <range> ':' [<resolution>] ']' [ @ <float_literal> ] [ offset <duration> ]


# 返回过去 30分钟 http_requests_total 指标的 5分钟平均增长率，分辨率为 1分钟。
rate(prometheus_http_requests_total[5m])[30m:1m]

# 使用默认值
rate(prometheus_http_requests_total[5m])[30m:]
```

### 1.3.4、运算符

Prometheus 的查询语言支持基本的逻辑和算术运算符。对于两个瞬时向量之间的操作，可以修改匹配行为。

> 算术运算符

```
# ==========
#  算术运算符
# ==========

  +  加
  -  减
  *  乘
  /  除
  %  模
  ^  幂

# 算术运算符可用在 标量/标量，向量/标量，向量/向量 值对之间
# 1 在两个标量之间，运算结果为标量。
# 例：
> 2^3
scalar	8

# 2 在瞬时向量和标量之间，运算符应用于向量中每个数据样本的值。指标名称将被删除。
# 例：格式化内存单位（GB）
> node_memory_MemTotal_bytes/(1024*1024*1024)
{instance="172.16.56.53:9100", job="node_exporter"}	15.415473937988281

# 3 在两个瞬时向量之间，运算符应用于左侧向量中的每个条目及其在右侧向量中的匹配元素。结果被传播到结果向量中，分组标签成为输出标签集。指标名称将被删除。在右侧向量中找不到匹配条目的条目不属于结果的一部分。
# 例：内存使用率
> (node_memory_MemTotal_bytes - node_memory_MemFree_bytes) / node_memory_MemTotal_bytes
{instance="172.16.56.53:9100", job="node_exporter"}	0.9759254594199462


# 三角运算符 atan2
# 三角运算符允许使用向量匹配在两个向量上执行三角函数，这在普通函数中是不可用的。它们的行为方式与算术运算符相同。
```

> 比较运算符

```
# ==========
#  比较运算符
# ==========

  ==
  !=
  >
  <
  >=
  <=

# 比较运算符可用在 标量/标量，向量/标量，向量/向量 值对之间
# 默认情况下，它们会过滤。可以通过在运算符后提供 bool 来修改它们的行为，这将为值返回 0 或 1，而不是过滤。

# 1 在两个标量之间，必须提供 bool 修饰符，这些运算符根据比较结果产生另一个标量 0 (false) 或 1 (true).
# 例：comparisons between scalars must use BOOL modifier
$ 2 > bool 1
scalar	1

# 2 在瞬时向量和标量之间，这些运算符应用于向量中每个数据样本的值，比较结果为假的向量元素从结果向量中删除。如果提供了 bool 修饰符，则将被丢弃的向量元素的值为 0，将被保留单向量元素值为 1，且指标名称会被删除。
# 例 2.1 过滤：筛选出请求量 > 1000 的接口
$ prometheus_http_requests_total > 1000
prometheus_http_requests_total{code="200", handler="/metrics", instance="localhost:9090", job="prometheus"}	34695

# 例 2.2 保留元素：bool 修饰符进行阈值判断
$ (node_memory_MemTotal_bytes - node_memory_MemFree_bytes) / node_memory_MemTotal_bytes >= bool 0.9
{instance="localhost:9100", job="node_exporter"}	0

# 3 在两个瞬时向量之间，这些运算符默认情况下充当过滤器，应用于匹配条目。表达式不为真或在表达式的另一侧找不到匹配的向量元素将从结果中删除，而其它元素将传播到结果向量中，分组标签将成为输出标签集。如果提供了 bool 修饰符，那么原本会被丢弃的向量元素将具有值 0，而将被保留的向量元素则具有值 1，分组标签再次成为输出标签集，且指标名称会被删除。
```

> 逻辑（集合）运算符

```
# =================
#  逻辑（集合）运算符
# =================

  # 逻辑（集合）运算符仅在瞬时向量之间定义
  
  and     与（交集）
  or      或（并集）
  unless  非（差集）


# 逻辑运算符

> prometheus_http_requests_total > 100 and prometheus_http_requests_total < 1000
> prometheus_http_requests_total == 100 or prometheus_http_requests_total == 200

# 集合运算符

# vector1 and vector2 
# 结果得到一个由 vector1 的元素组成的向量，其中 vector2 的元素具有完全匹配的标签集。其他元素被删除，指标名称和值从左侧向量中继承。

# vector1 or vector2 
# 产生的向量包含 vector1 的所有原始元素（标签集+值），此外还包含 vector2 的所有元素，这些元素在 vector1 中没有匹配的标签集。

# vector1 unless vector2 
# 结果为一个由 vector1 的元素组成的向量， 其中 vector2 中没有具有完全匹配标签集的元素。两个向量中的所有匹配元素都将被删除。
```

> 聚合运算符

```
# ==========
#  聚合运算符
# ==========

  sum           计算维度之和
  min           选择最小尺寸
  max           选择最大尺寸
  avg           计算尺寸的平均值
  group         结果向量中的所有值均为 1
  stddev        计算维度上的总体标准差
  stdvar        计算维度上的总体标准方差
  count         计算向量中的元素数
  count_values  计算具有相同值的元素数量
  bottomk       按样本值计算的最小 k 个元素（取后 k 名）
  topk          按样本值计算的最大 k 个元素（取前 k 名）
  quantile      计算维度的 φ 分位数 （0 ≤ φ ≤ 1） 

# 聚合运算符可用于聚合单个即时向量的元素，从而生成一个包含较少聚合值元素的新向量。

# 这些运算符既可以用于聚合所有标签维度，也可以通过包含without或by子句来保留不同的维度。
# 这些子句可以在表达式之前或之后使用。
# 语法：
<aggr-op> [without|by (<label list>)] ([parameter,] <vector expression>)
# 或
<aggr-op>([parameter,] <vector expression>) [without|by (<label list>)]

# prometheus_http_requests_total 的所有标签
prometheus_http_requests_total{code="200", handler="/metrics", instance="localhost:9090", job="prometheus"}

# 例1：without 用法
# 按除 without 列表外的标签分组，统计接口调用总数
# 显示信息的时候，不包括指定的标签列表
> sum (prometheus_http_requests_total) without (handler) 
{code="200", instance="localhost:9090", job="prometheus"}	76746
{code="499", instance="localhost:9090", job="prometheus"}	10
{code="400", instance="localhost:9090", job="prometheus"}	13
{code="302", instance="localhost:9090", job="prometheus"}	1

# 例2：by 用法
# 按 by 列表的标签分组，统计接口调用总数
# 显示信息的时候，仅显示指定的标签
> sum (prometheus_http_requests_total) by (code) 
{code="200"}	76741
{code="499"}	10
{code="400"}	13
{code="302"}	1

# 例3：count 统计 CPU 数量
> count(node_cpu_seconds_total{mode='system'})
{}	16

# 例4：count_values 计算具有相同访问次数的接口
# 第一个参数自定义 counter 计数器值的显示名称
> count_values("counts", prometheus_http_requests_total)
{counts="11"}	1
{counts="12"}	1
{counts="4"}	1
{counts="0"}	32
{counts="42"}	1
{counts="3555"}	1
{counts="6"}	1
{counts="8862"}	1
...

# 例5：请求量前 10 名的接口
> topk(10, prometheus_http_requests_total)
prometheus_http_requests_total{code="200", handler="/metrics", instance="localhost:9090", job="prometheus"}	35565
prometheus_http_requests_total{code="200", handler="/api/v1/rules", instance="localhost:9090", job="prometheus"}	8865
prometheus_http_requests_total{code="200", handler="/api/v1/query_range", instance="localhost:9090", job="prometheus"}	658
...
```

### 1.3.5、函数

> 数学函数

```
# =========
#  数学函数
# =========

  abs(v instant-vector)    返回所有样本的绝对值
  ceil(v instant-vector)   对所有样本值向上取整
  floor(v instant-vector)  对所有样本值向下取整
  exp(v instant-vector)    计算所有元素的指数，特例：
                             exp(+Inf) = +Inf
                             exp(NaN) = NaN
  ln(v instant-vector)     计算所有元素的自然对数，特例：
                             ln(+Inf) = +Inf
                             ln(0) = -Inf
                             ln(x < 0) = NaN
                             ln(NaN) = NaN
  log2(v instant-vector)   计算所有元素的二进制对数，特例同 ln()
  log10(v instant-vector)  计算所有元素的十进制对数，特例同 ln()
  sqrt(v instant-vector)   计算所有元素的平方根
  # 将所有元素的样本值四舍五入到最接近的整数。
  # 可选的 to_nearest 参数允许指定样本值应四舍五入到的最接近的倍数。这个倍数也可能是分数。
  round(v instant-vector, to_nearest=1 scalar)
```

> 日期时间函数

```
# ============
#  日期时间函数
# ============

  year()           返回当前年份：2024
  month()          返回当前月份：1 ~ 12
  hour()           返回当前小时数：0 ~ 23
  minute()         返回当前分钟数：0 ~ 59
  time()           返回当前当前时间戳：1723033262.709
  day_of_month()   返回当前是本月的第几天：1 ~ 31
  day_of_week()    返回当前是本周的第几天：0 ~ 6
  day_of_year()    返回当前是本年的第几天：1 ~ 365/366
  days_in_month()  返回本月的天数：28/29/30/31
```

> 排序函数

```
# =========
#  排序函数
# =========

  sort(v instant-vector)                                   指标值升序
  sort_desc(v instant-vector)                              指标值降序
  sort_by_label(v instant-vector, label string, ...)       按标签名升序
  sort_by_label_desc(v instant-vector, label string, ...)  按标签名降序
```

> 采样函数

```
# =========
#  采样函数
# =========

  # 当向量的样本非空时，返回空向量，样本为空时，返回 1。
  # 常用于在给定指标名称或标签组合过滤不存在序列时发出警报。
  absent(v instant-vector)            对瞬时向量采样
  absent_over_time(v range-vector)    对范围向量采样
                                      
  # 计算所有样本在时间范围内的变化数，将变化数和等效标签作为瞬时向量返回
  changes(v range-vector)
  
  # 将所有样本值固定为 min 和 max
  # 当样本值 < min 时，将样本值定为 min
  # 当样本值 > max 时，将样本值定为 max
  clamp(v instant-vector, min scalar, max scalar)  固定最大和最小值
  clamp_max(v instant-vector, max scalar)          固定最大值
  clamp_min(v instant-vector, min scalar)          固定最小值
  
  # 计算范围向量中每个时间序列的差值，将差值和等效标签作为瞬时向量返回
  delta(v range-vector)   第一个值和最后一个值的差值，只能与 Gauges 和 Histograms  一起使用
  idelta(v range-vector)  最后两个样本的差值，只能与 Gauges 一起使用
  
  # 使用简单的线性回归计算范围向量中时间序列的每秒导数（向量中必须至少有 2 个样本才能执行计算）。
  # 当在范围向量中找到 +Inf/-Inf 时，计算出的斜率和偏移值将为 NaN。
  # deriv 只能与 Gauges 一起使用。
  deriv(v range-vector) 
  
  # 使用简单的线性回归预测从现在开始的时间序列 t 秒的值（向量中必须至少有 2 个样本才能执行计算）。
  # 当在范围向量中找到 +Inf/-Inf 时，计算出的斜率和偏移值将为 NaN。
  # predict_linear 只能与 Gauges 一起使用。
  predict_linear(v range-vector, t scalar)
  
  # 根据时间范围为时间序列生成平滑值。
  # 平滑因子 sf 越低，对旧数据的重视程度越高。
  # 趋势因子 tf 越高，考虑的数据趋势越多。
  # sf 和 tf都必须介于 0 ~ 1 之间。
  # holt_winters 只能与 Gauges 一起使用。
  holt_winters(v range-vector, sf scalar, tf scalar)
  
  # 计算范围向量中时间序列的增量，将差值和等效标签作为瞬时向量返回
  # 单调性的中断（例如由于目标重新启动而导致的计数器重置）会自动调整。
  # increase 只应与 counters 和 histograms 一起使用。
  # increase(v) 是 rate(v) 乘以时间范围窗口下的秒数的句法糖，提高人类可读性。
  increase(v range-vector)
  
  # 计算范围向量中时间序列的每秒瞬时增长率。基于最后两个数据点。
  # 单调性的中断（例如由于目标重新启动而导致的计数器重置）会自动调整。
  irate(v range-vector)
  
  # 计算范围向量中时间序列的每秒平均增长率。
  # 单调性的中断（例如由于目标重新启动而导致的计数器重置）会自动调整。
  # 此外，计算外推到时间范围的末端，允许遗漏的刮擦或刮擦周期与范围的时间段不完全对齐。
  # rate 只应与 counters 和 histograms 一起使用，其中组件的行为类似于 counters。
  # 它最适合用于警报和绘制缓慢移动的 counters 的图形。
  # 当 rate 与聚合操作组合时，应先 rate 再聚合，否则 rate 无法在目标重启时检测到计数器重置
  rate(v range-vector)
  
  # 使用分隔符连接所有 src_labels 值，并返回带有包含连接值的标签 dst_label 的时间序列。
  label_join(v instant-vector, dst_label string, separator string, src_label_1 string, src_label_2 string, ...)
  
  # 将正则表达式 regex 与标签 src_label 的值进行匹配。
  # 如果匹配，则返回的时间序列中标签 dst_label 的值将是替换的扩展，以及输入中的原始标签。
  # 正则表达式中的捕获组可以用 $1、$2 等引用。
  # 正则表达式中命名的捕获组也可以用 $name 引用（其中name是捕获组名称）。
  # 如果正则表达式不匹配，则时间序列将原封不动地返回。
  label_replace(v instant-vector, dst_label string, replacement string, src_label string, regex string)
  
  # 将时间范围内计数器重置的次数作为瞬时向量返回。
  resets(v range-vector)
  
  # 返回一个向量，其中所有样本值都转换为符号。
  # 定义如下：如果v为正，则为1；如果v为负，则为-1；如果v等于零，则为0。
  sgn(v instant-vector)
  
  # 返回单个样本值的标量，如果输入向量不是单个元素（零或多个），将返回 NaN
  scalar(v instant-vector)
  # 将标量作为不带标签的向量返回
  vector(s scalar) 

##### absent() #####

# 指标非空返回空集合
> absent(prometheus_http_requests_total{})
Empty query result

# 指标不存在或标签过滤不存在数据时，返回 1
> absent(abc{})
{}	1
> absent(prometheus_http_requests_total{abc='123'})
{abc="123"}	1


##### changes #####

# scrape_interval: 15s（1 分钟采样 4 次）
# 因此 /metrics 接口的请求数量在 1 分钟的变化数为：3 ~ 4
> changes(prometheus_http_requests_total{}[61s]) 
{code="200", handler="/metrics", instance="localhost:9090", job="prometheus"}	4
...


##### delta & idelta #####

# scrape_interval: 15s（1 分钟采样 4 次）
# 因此 /metrics 接口的请求数量在 5 分钟的增量为：20
> delta(prometheus_http_requests_total{}[5m]) 
{code="200", handler="/metrics", instance="localhost:9090", job="prometheus"}	20
...

> idelta(prometheus_http_requests_total{}[5m]) 
{code="200", handler="/metrics", instance="localhost:9090", job="prometheus"}	1


##### deriv #####

# /metrics 接口请求数稳定在每分钟 4 次
# derivative = 4/60 = 0.06666666666666667
# 用导数生成的 Graph 便于观测各接口的流量洪峰
> deriv(prometheus_http_requests_total{}[5m]) 
{code="200", handler="/metrics", instance="localhost:9090", job="prometheus"}	0.06666666666666667


##### increase & rate #####

# 接口请求数在 5m 内的增量
> increase(prometheus_http_requests_total{}[5m]) 
{code="200", handler="/metrics", instance="localhost:9090", job="prometheus"}	20

# 接口请求数在 5m 内的每秒增长率
> rate(prometheus_http_requests_total{}[5m]) 
{code="200", handler="/metrics", instance="localhost:9090", job="prometheus"}	0.06666666666666667

# 结论：rate = increase/duration(s)


##### label_join #####

> label_join(prometheus_http_requests_total, "url", "", "instance", "handler")
prometheus_http_requests_total{code="200", handler="/", instance="localhost:9090", job="prometheus", url="localhost:9090/"}	0
prometheus_http_requests_total{code="200", handler="/metrics", instance="localhost:9090", job="prometheus", url="localhost:9090/metrics"}	50094
...


##### scalar() & vector() #####

# 返回单个样本的 scalar
> scalar(prometheus_http_requests_total{handler="/metrics"})
scalar	55707

# 零样本或多样本时，返回 NaN
> scalar(abc{})
scalar	NaN
> scalar(prometheus_http_requests_total{})
scalar	NaN

# scalar to vector
> vector(123)
{}	123
```

> 聚合函数

```
# =========
#  聚合函数
# =========
  <aggregation>_over_time()
  随着时间聚合给定范围向量的每个序列，并返回一个包含每个序列聚合结果的即时向量。

  avg_over_time(range-vector)               指定间隔内所有点的平均值
  min_over_time(range-vector)               指定间隔内所有点的最小值
  max_over_time(range-vector)               指定间隔内所有点的最大值
  sum_over_time(range-vector)               指定间隔内所有值的总和
  count_over_time(range-vector)             指定时间间隔内所有值的计数
  quantile_over_time(scalar, range-vector)  指定区间内值的 φ 分位数 （0 ≤ φ ≤ 1）
  stddev_over_time(range-vector)            指定区间内值的总体标准差
  stdvar_over_time(range-vector)            指定区间内值的总体标准方差
  last_over_time(range-vector)              指定间隔内的最新点值
  present_over_time(range-vector)           指定时间间隔内的任何序列的值为 1
```

> 其它函数

```
# ================
#  histogram 函数
# ================

  histogram_avg()
  histogram_count()
  histogram_sum()
  histogram_fraction()
  histogram_quantile()
  histogram_stddev() 
  histogram_stdvar()


# =========
#  三角函数
# =========

  acos(v instant-vector)
  acosh(v instant-vector)
  asin(v instant-vector)
  asinh(v instant-vector)
  atan(v instant-vector)
  atanh(v instant-vector)
  cos(v instant-vector)
  cosh(v instant-vector)
  sin(v instant-vector)
  sinh(v instant-vector)
  tan(v instant-vector)
  tanh(v instant-vector)
```



## 1.4、ClientLibs

#### [Go](https://github.com/prometheus/client_golang)

> 文档：https://prometheus.io/docs/guides/go-application/

```
go get github.com/prometheus/client_golang/prometheus
go get github.com/prometheus/client_golang/prometheus/promauto
go get github.com/prometheus/client_golang/prometheus/promhttp
```

> 公开 Go 应用程序的指标

```go
package main

import (
	"github.com/prometheus/client_golang/prometheus"
	"github.com/prometheus/client_golang/prometheus/promauto"
	"github.com/prometheus/client_golang/prometheus/promhttp"
	"net/http"
)

// 计数器（自动注册）
var pingCounter = promauto.NewCounter(
	prometheus.CounterOpts{
		Name: "ping_request_count",
		Help: "No of request handled by Ping handler",
	},
)

func ping(w http.ResponseWriter, req *http.Request) {
	// 自增 1
	pingCounter.Inc()

	w.Write([]byte("pong"))
}

func main() {
	// 自定义指标
	// 使用 promauto.NewCounter() 时，需手动注册
	// prometheus.MustRegister(counter)

	http.HandleFunc("/ping", ping)
	// promhttp.Handler() 仅公开默认的 Go 指标
    http.Handle("/metrics", promhttp.Handler())
	http.ListenAndServe(":1234", nil)
}
```

查看指标：http://localhost:1234/metrics

> 抓取指标配置

```yaml
scrape_configs:
  - job_name: myapp
    scrape_interval: 10s
    static_configs:
      - targets: ["localhost:1234"]
```

## 1.5、Exporters

### 1.5.1、Node Exporter

Node Exporter 公开 Linux 和其他 Unix 系统上的一组广泛的机器级指标，例如作为 CPU 使用率、内存、磁盘利用率、文件系统填充度和网络带宽。

```sh
$ wget https://github.com/prometheus/node_exporter/releases/download/v1.8.2/node_exporter-1.8.2.linux-amd64.tar.gz
$ tar -xzvf node_exporter-1.8.2.linux-amd64.tar.gz
$ cd node_exporter-1.8.2.linux-amd64
$ ll
-rw-r--r-- 1 dev dev    11357 7月  14 19:57 LICENSE
-rwxr-xr-x 1 dev dev 20500541 7月  14 19:54 node_exporter*
-rw-r--r-- 1 dev dev      463 7月  14 19:57 NOTICE

# Start 3 example targets in separate terminals:
# $ ./node_exporter --web.listen-address :9100
# $ ./node_exporter --web.listen-address :9101
# $ ./node_exporter --web.listen-address :9102

# 后台启动
$ ./node_exporter --web.listen-address :9100 1>node_exporter_9100.log 2>&1 &
[1] 2806172
$ ./node_exporter --web.listen-address :9101 1>node_exporter_9101.log 2>&1 &
[2] 2806250
$ ./node_exporter --web.listen-address :9102 1>node_exporter_9102.log 2>&1 &
[3] 2806257
$ ps -ef | grep node_exporter
dev      2806172 2714651  0 10:18 pts/1    00:00:00 ./node_exporter --web.listen-address :9100
dev      2806250 2714651  0 10:18 pts/1    00:00:00 ./node_exporter --web.listen-address :9101
dev      2806257 2714651  0 10:19 pts/1    00:00:00 ./node_exporter --web.listen-address :9102
dev      2806267 2714651  0 10:19 pts/1    00:00:00 grep --color=auto node_exporter


# 访问：http://localhost:9100/metrics
# 访问：http://localhost:9101/metrics
# 访问：http://localhost:9102/metrics
```

> Docker

```sh
$ docker pull quay.io/prometheus/node-exporter

$ docker run -d \
  --net="host" \
  --pid="host" \
  -v "/:/host:ro,rslave" \
  quay.io/prometheus/node-exporter:latest \
  --path.rootfs=/host
```

> Docker Compose
>
> `$ docker compose up -d`

```yaml
version: '3.8'

services:
  node_exporter:
    image: quay.io/prometheus/node-exporter:latest
    container_name: node_exporter_9100
    command:
      - '--path.rootfs=/host'
      - '--web.listen-address=:9100'
    network_mode: host
    pid: host
    restart: unless-stopped
    volumes:
      - '/:/host:ro,rslave'

  node_exporter:
    image: quay.io/prometheus/node-exporter:latest
    container_name: node_exporter_9101
    command:
      - '--path.rootfs=/host'
      - '--web.listen-address=:9101'
    network_mode: host
    pid: host
    restart: unless-stopped
    volumes:
      - '/:/host:ro,rslave'
  
  node_exporter:
    image: quay.io/prometheus/node-exporter:latest
    container_name: node_exporter_9102
    command:
      - '--path.rootfs=/host'
      - '--web.listen-address=:9102'
    network_mode: host
    pid: host
    restart: unless-stopped
    volumes:
      - '/:/host:ro,rslave'
```

> 配置 Prometheus 以监控目标

```yaml
# prometheus.yml

scrape_configs:
  - job_name: "prometheus"
    basic_auth:
      username: admin
      password: 123456
    static_configs:
      - targets: ["localhost:9090"]
      
  - job_name: 'node_exporter'

    # Override the global default and scrape targets from this job every 5 seconds.
    scrape_interval: 5s

    static_configs:
      - targets: ['localhost:9100', 'localhost:9101']
        labels:
          group: 'production'

      - targets: ['localhost:9102']
        labels:
          group: 'canary'
```

> 注：重启或重新加载配置后生效

### 1.5.2、Pushgateway





# [Grafana](https://grafana.com/)

## 2.1、安装 & 配置

[Download Grafana](https://grafana.com/grafana/download)

```sh
$ wget https://dl.grafana.com/enterprise/release/grafana-enterprise-11.1.0.linux-amd64.tar.gz
$ tar -zxvf grafana-enterprise-11.1.0.linux-amd64.tar.gz
$ cd grafana-v11.1.0/
$ ll
drwxr-xr-x  2 dev dev  4096 6月  24 21:54 bin/
drwxr-xr-x  3 dev dev  4096 6月  24 21:49 conf/
-rw-r--r--  1 dev dev  5844 6月  24 21:49 Dockerfile
drwxrwxr-x  3 dev dev  4096 8月   9 08:57 docs/
-rw-r--r--  1 dev dev 12155 6月  24 21:51 LICENSE
-rw-r--r--  1 dev dev   105 6月  24 21:49 NOTICE.md
drwxr-xr-x  2 dev dev  4096 6月  24 21:54 npm-artifacts/
drwxrwxr-x  6 dev dev  4096 8月   9 08:57 packaging/
drwxr-xr-x  2 dev dev  4096 6月  24 21:49 plugins-bundled/
drwxr-xr-x 16 dev dev  4096 6月  24 21:55 public/
-rw-r--r--  1 dev dev  3261 6月  24 21:49 README.md
drwxr-xr-x  8 dev dev 32768 6月  24 21:54 storybook/
drwxrwxr-x  2 dev dev  4096 8月   9 08:57 tools/
-rw-r--r--  1 dev dev     8 6月  24 21:51 VERSION
# 启动
$ cd bin
$ ./grafana server
...


# Docker
$ docker run -d \
  --name=grafana \
  -p 3000：3000 \
  grafana/grafana-enterprise
```



## 2.2、设计仪表板

从共享仪表板 [Grafana dashboards](https://grafana.com/grafana/dashboards/) 导入预构建的仪表板。





# AlertManager 

## 3.1、概述

Prometheus 警报分为两部分。Prometheus 服务器中的警报规则向 Alertmanager 发送警报。Alertmanager 管理这些警报，包括静音、抑制、聚合和通过电子邮件、随叫随到通知系统、聊天平台等方法发送通知。

设置警报和通知的主要步骤：

- 设置和配置 Alertmanager
- 配置 Prometheus 与 Alertmanager 通信
- 在 Prometheus 中创建报警规则

Alertmanager 处理客户端应用程序（如 Prometheus 服务器）发送的警报。它负责重复数据删除、分组，并将它们路由到正确的接收器集成，如电子邮件、PagerDuty、OpsGenie、webhook。它还负责警报的静默和抑制。

- Grouping（分组）：将类似性质的警报分类到单个通知中。这在许多系统同时发生故障的较大规模停机期间特别有用，并且成百上千个警报可能会同时触发。
- Inhibition（抑制）：是一种概念，如果某些其他警报已经触发，则抑制某些警报的通知。Inhibition 是通过 Alertmanager 的配置文件配置的。
- Silences（静默）：是一种简单的方法，可以在给定时间内将警报静音（如：服务器升级维护期间）。静默是基于匹配器配置的，就像路由树一样。检查传入警报是否与活动静默的所有相等或正则表达式匹配器匹配。如果他们这样做，将不会发送该警报的通知。静默在 Alertmanager 的 web 界面中配置。

## 3.2、告警流程

```
┌───────────────────┐               ┌──────────────┐          ┌─────────────┐
│ Prometheus Server │               │ Alertmanager │          |    Email    |
| ───────────────── │               | ──────────── │          |             |
|                   │               |              │          |   Webhook   |
│    Load Rules     │  push alerts  │    Router    |  notify  |             |
│         ↓         │ ============> |      ↓       | =======> |    Wechat   |
│       Alert       │               |   Receiver   |          |     ...     |
└───────────────────┘               └──────────────┘          └─────────────┘

Prometheus => 触发阈值 => 超出持续时间 => Alertmanager => 分组|抑制|静默 => 媒体类型 => Email|Dingtalk|Wechat 通知

Prometheus 根据 rule 规则判断指标是否超过阈值，且持续一段时间未恢复就发送告警到 Alertmanager。
Alertmanager 根据配置中定义的规则对告警进行分组|抑制|静默处理，最后以不同的形式发送给 Receiver。
```

## 3.3、安装

```sh
# 下载
$ wget https://github.com/prometheus/alertmanager/releases/download/v0.27.0/alertmanager-0.27.0.linux-amd64.tar.gz
$ tar -zxvf alertmanager-0.27.0.linux-amd64.tar.gz
$ cd alertmanager-0.27.0.linux-amd64
$ ll
-rwxr-xr-x 1 dev dev 37345962 2月  28 19:52 alertmanager*
-rw-r--r-- 1 dev dev      356 2月  28 19:55 alertmanager.yml
-rwxr-xr-x 1 dev dev 30130103 2月  28 19:52 amtool*
-rw-r--r-- 1 dev dev    11357 2月  28 19:55 LICENSE
-rw-r--r-- 1 dev dev      457 2月  28 19:55 NOTICE

# 启动
$ ./alertmanager --config.file=alertmanager.yml

# 访问：http://localhost:9093
```

## 3.4、配置参考

### 3.4.1、Alertmanager

- global（全局配置）：用来定义一些全局的公共参数，比如 Smtp 邮件服务器配置、企业微信配置等。
- template（告警模板）：用于定义告警通知时的模板，比如邮件模板、微信通知模板等。
- router（告警路由规则）：用于定义告警的分发策略。
- receivers（接收者）：用于定义告警接收人，可以是邮箱、微信、webhook等。一般配合告警路由规则来使用，实现不同的告警发给不同的接收人。
- inhibit_rules（告警抑制规则）：用于定义告警抑制规则，避免垃圾告警产生。

```yaml
global:

  ##### SMTP 邮件服务器配置 #####
  
  # 默认发件人
  # The default SMTP From header field.
  smtp_from: '123456@qq.com'   
  # The default SMTP smarthost used for sending emails, including port number.
  # Port number usually is 25, or 587 for SMTP over TLS (sometimes referred to as STARTTLS).
  # Example: smtp.example.org:587
  smtp_smarthost: 'smtp.qq.com:465'
  # The default hostname to identify to the SMTP server.
  [ smtp_hello: <string> | default = "localhost" ]
  # SMTP Auth using CRAM-MD5, LOGIN and PLAIN. If empty, Alertmanager doesn't authenticate to the SMTP server.
  smtp_auth_username: '123456@qq.com'
  # SMTP Auth using LOGIN and PLAIN.
  smtp_auth_password: 'secret'
  # SMTP Auth using LOGIN and PLAIN.
  [ smtp_auth_password_file: <string> ]
  # SMTP Auth using PLAIN.
  [ smtp_auth_identity: <string> ]
  # SMTP Auth using CRAM-MD5.
  [ smtp_auth_secret: <secret> ]
  # The default SMTP TLS requirement.
  # Note that Go does not support unencrypted connections to remote SMTP endpoints.
  [ smtp_require_tls: <bool> | default = true ]

##### 告警模板文件 #####

# Files from which custom notification template definitions are read.
# The last component may use a wildcard matcher, e.g. 'templates/*.tmpl'.
templates:
  [ - <filepath> ... ]

##### 告警路由规则，所有告警信息的根路由 #####

# The root node of the routing tree.
route: <route>

##### 通知接收者列表 #####

# A list of notification receivers.
receivers:
  - <receiver> ...

##### 抑制规则列表 #####

# A list of inhibition rules.
inhibit_rules:
  [ - <inhibit_rule> ... ]

##### 静默/激活路由的时间间隔列表 #####

# A list of time intervals for muting/activating routes.
time_intervals:
  [ - <time_interval> ... ]  
```

### 3.4.2、Rules

```yaml
groups:
  # The name of the group. Must be unique within a file.
  - name: <string>
    
    # How often rules in the group are evaluated.
    [ interval: <duration> | default = global.evaluation_interval ]
    
    # Limit the number of alerts an alerting rule and series a recording
    # rule can produce. 0 is no limit.
    [ limit: <int> | default = 0 ]
    
    # Offset the rule evaluation timestamp of this particular group by the specified duration into the past.
    [ query_offset: <duration> | default = global.rule_query_offset ]
    
    rules:
      # alertname 必须是有效的 Label（生成新的时间序列的指标名）
      # The name of the alert. Must be a valid label value.
      - alert: <string>

		# expr 要计算的PromQL表达式。
        # 每个评估周期都在当前时间进行评估，所有由此产生的时间序列都成为 pending/firing 警报。
        # The PromQL expression to evaluate. Every evaluation cycle this is
        # evaluated at the current time, and all resultant time series become
        # pending/firing alerts.
        expr: <string>
        
        # Alerts are considered firing once they have been returned for this long.
        # Alerts which have not yet fired for long enough are considered pending.
        [ for: <duration> | default = 0s ]
        
        # How long an alert will continue firing after the condition that triggered it
        # has cleared.
        [ keep_firing_for: <duration> | default = 0s ]
        
        # Labels to add or overwrite for each alert.
        labels:
          [ <labelname>: <tmpl_string> ]
        
        # Annotations to add to each alert.
        annotations:
          [ <labelname>: <tmpl_string> ]
```



## 3.5、webhook 告警

### 3.5.1、[webhook.site](https://webhook.site/)

转到 webhook.site 并复制 webhook URL

### 3.5.2、Alertmanager 配置

```yaml
# alertmanager.yml

global:
  # ResolveTimeout 是警报管理器在警报不包括 EndsAt 时使用的默认值，
  # 在此时间过后，如果警报尚未更新，它可以声明警报已解决。
  # 这对普罗米修斯的警报没有影响，因为它们总是包含 EndsAt。
  resolve_timeout: 5m

# 告警路由规则
# 目前只有根路由，所以所有的告警都会发给默认接收者 webhook
route:
  group_by: ['alertname']
  group_wait: 30s
  group_interval: 5m
  repeat_interval: 1h
  receiver: 'webhook'
  
receivers:
    # 接收者的唯一名称
  - name: 'webhook'
    webhook_configs:
        # webhook.site 复制的 URL
      - url: 'https://webhook.site/f552c8fc-ce6f-418c-8226-0c45c9d60a33'
        # 发送告警已解决消息
        send_resolved: false

inhibit_rules:
  - source_match:
      severity: 'critical'
    target_match:
      severity: 'warning'
    equal: ['alertname', 'dev', 'instance']
```

### 3.5.3、Prometheus 的 Rules 配置

> rule_files

```yaml
# rules.yml

# 规则组
groups:
  ################################
  # * 第 1 组：操作系统性能监控规则 * #
  ################################
  - name: 'OS_performance'
    rules:
    ##### 第 1.1 条规则：CPU 使用率 > 90% #####
    - alert: 'CPU_usage_rate_gt_90'
      # node_cpu_seconds_total CPU 在每种模式下花费的秒数（指标类型：Counter）
      # 例：
      # node_cpu_seconds_total{cpu="0",mode="idle"} 1.88949409e+06
      # node_cpu_seconds_total{cpu="0",mode="iowait"} 243.3
      # node_cpu_seconds_total{cpu="0",mode="irq"} 0
      # node_cpu_seconds_total{cpu="0",mode="nice"} 0.52
      # node_cpu_seconds_total{cpu="0",mode="softirq"} 253.46
      # node_cpu_seconds_total{cpu="0",mode="steal"} 0
      # node_cpu_seconds_total{cpu="0",mode="system"} 2602.88
      # node_cpu_seconds_total{cpu="0",mode="user"} 7627.68
      
      # rate(v range-vector) 计算范围向量中时间序列的每秒平均增长率。
      # 过去 1 分钟内测量的每秒平均空闲率
      expr: (1 - avg(rate(node_cpu_seconds_total{mode="idle"}[1m]))) >= 0.9
      # 一旦警报发出这么长时间，就会从 pending 变为 firing。
      for: 10s
      # 为每个警报添加或覆盖的标签
      labels:
        # 严重等级：警告
        severity: 'warning'
      # 为每个警报添加或覆盖的注释
      annotations:
        description: 'CPU 使用率大于 90% （Current value is: {{ $value }}）'
        summary: 'CPU 负载告警'
      
    ##### 第 1.2 条规则：Memory 使用率 > 90% #####
    - alert: 'Memory_usage_rate_gt_90'
      # node_memory_MemAvailable_bytes 内存可用字节（指标类型：Gauge）
      # node_memory_MemFree_bytes      内存空闲字节（指标类型：Gauge）
      # node_memory_MemTotal_bytes     内存总字节（指标类型：Gauge）
      # （总的 - 空闲）/ 总的 = 内存使用率 > 90% 时生成预警
      expr: ((node_memory_MemTotal_bytes{job="node_exporter"} - node_memory_MemFree_bytes{job="node_exporter"}) / node_memory_MemTotal_bytes{job="node_exporter"}) >= 0.9
      for: 10s
      labels:
        # 严重等级：严重
        severity: 'critical'
      annotations:
        description: 'Memory 使用率大于 90% （Current value is: {{ $value }}）'
        summary: 'Memory 负载告警'
      
    ##### 第 1.3 条规则：Root FS 使用率 > 90% #####
    - alert: 'Root_FS_usage_rate_gt_90'
      # node_filesystem_avail_bytes  非 root 用户可用的文件系统空间 Byte（指标类型：Gauge）
      # node_filesystem_size_bytes   文件系统大小 Byte（指标类型：Gauge）
      # （1 - 根目录可用 / 根目录总的）= 根目录磁盘使用率 > 90%
      expr: (1 - (node_filesystem_avail_bytes{job="node_exporter",mountpoint="/",fstype!="rootfs"}) / node_filesystem_size_bytes{job="node_exporter",mountpoint="/",fstype!="rootfs"}) > 0.9
      for: 10m
  
  ###############################
  # * 第 2 组：HTTP 请求监控规则 * #
  ###############################
  - name: 'HTTP_request'
    rules:
    ##### 第 2.1 条规则：高请求负载 #####
    - alert: 'HighRequestLoad'
      # prometheus_http_requests_total HTTP 请求计数器（指标类型：Counter）
      # 近 1m 的请求数达到 10，就认为是高负载
      expr: sum(rate(prometheus_http_requests_total[1m]))*100 > 10
      for: 1m
      labels:
        severity: page
      annotations:
        summary: 'High request load'
        
    ##### 第 2.2 条规则：请求错误率 > 1% #####
    - alert: 'RequestErrorRate'
      # prometheus_http_requests_total HTTP 请求计数器（指标类型：Counter）
      # 错误的/总的 = 错误率 > 0.01
      expr: prometheus_http_requests_total{code!="200"} / prometheus_http_requests_total > 0.01
      for: 1m
      labels:
        # 严重等级：呼叫
        severity: page
      annotations:
        summary: 'High request error rate'
```

> 检查 rules.yml

```
# 检查 rule 文件格式
$ promtool check rules rules.yml
Checking rules.yml
  SUCCESS: 5 rules found
```

> prometheus.yml

```yaml
global:
  # 抓取目标的频率
  scrape_interval: 15s
  # 评估规则的频率
  evaluation_interval: 10s

# 规则文件列表
rule_files:
  - rules.yml

# 警报管理器配置
alerting:
  alertmanagers:
    - static_configs:
        # Prometheus 会将告警发到这些指定的 Alertmanager
      - targets: ['localhost:9093']

# 抓取配置
scrape_configs:
  - job_name: prometheus
    static_configs:
      - targets: ['localhost:9090']
      
  - job_name: node_exporter
    static_configs:
      - targets: ['localhost:9100'] 
```

### 3.5.4、验证

- 查看 Prometheus 中的 Rules：http://localhost:9090/rules

- 查看 Prometheus 中的 Alerts：http://localhost:9090/alerts

    Alert 有三个状态：

    - Inactive：闲置的
    - Pending：
    - Firing：

- 查看 Alertmanager 中收到的告警：http://localhost:9093/#/alerts

- 查看 Webhook.site 中收到的消息：https://webhook.site

    > Memory 告警

    ```json
    {
      "receiver": "webhook",
      "status": "firing",
      "alerts": [
        {
          "status": "firing",
          "labels": {
            "alertname": "Memory_usage_rate_gt_90",
            "instance": "172.16.56.53:9100",
            "job": "node_exporter",
            "severity": "critical"
          },
          "annotations": {
            "description": "Memory 使用率大于 90% （Current value is: 0.9611197419299918）",
            "summary": "Memory 负载告警"
          },
          "startsAt": "2024-08-01T07:21:30.38Z",
          "endsAt": "0001-01-01T00:00:00Z",
          "generatorURL": "http://51ac5346d54f:9090/graph?g0.expr=%28%28node_memory_MemTotal_bytes%7Bjob%3D%22node_exporter%22%7D+-+node_memory_MemFree_bytes%7Bjob%3D%22node_exporter%22%7D%29+%2F+node_memory_MemTotal_bytes%7Bjob%3D%22node_exporter%22%7D%29+%3E%3D+0.9&g0.tab=1",
          "fingerprint": "91513bf168fe2cda"
        }
      ],
      "groupLabels": {},
      "commonLabels": {
        "alertname": "Memory_usage_rate_gt_90",
        "instance": "172.16.56.53:9100",
        "job": "node_exporter",
        "severity": "critical"
      },
      "commonAnnotations": {
        "description": "Memory 使用率大于 90% （Current value is: 0.9611197419299918）",
        "summary": "Memory 负载告警"
      },
      "externalURL": "http://172.16.56.53:9093",
      "version": "4",
      "groupKey": "{}:{}",
      "truncatedAlerts": 0
    }
    ```

## 3.6、Email 告警

```yaml
# alertmanager.yml

global:
  # 邮件服务器配置
  smtp_from: "test@qq.com"
  smtp_smarthost: "smtp.qq.com:465"
  smtp_hello: "qq.com"
  smtp_auth_username: "test@qq.com"
  smtp_auth_password: "***"

# 告警路由规则
# 目前只有根路由，所以所有的告警都会发给默认接收者 webhook
route:
  group_by: ['alertname']
  group_wait: 30s
  group_interval: 5m
  repeat_interval: 1h
  # receiver: 'webhook'
  receiver: 'email'
  
receivers:
  - name: 'webhook'
    webhook_configs:
        # webhook.site 复制的 URL
      - url: 'https://webhook.site/f552c8fc-ce6f-418c-8226-0c45c9d60a33'
        # 发送告警已解决消息
        send_resolved: false
        
  - name: 'email'
    email_configs:
      - to: yanchangzheng@szreadline.com
        send_resolved: true
```

## 3.7、DingTalk 告警

### 3.7.1、添加钉钉群机器人

复制群机器人的 webhook

### 3.7.2、安装 prometheus-webhook-dingtalk

```sh
$ wget https://github.com/timonwong/prometheus-webhook-dingtalk/releases/download/v2.1.0/prometheus-webhook-dingtalk-2.1.0.linux-amd64.tar.gz
$ tar -zxvf prometheus-webhook-dingtalk-2.1.0.linux-amd64.tar.gz
$ cd prometheus-webhook-dingtalk-2.1.0.linux-amd64/
$ ll
-rw-r--r-- 1 dev dev     1299 4月  21  2022 config.example.yml
drwxr-xr-x 4 dev dev     4096 4月  21  2022 contrib/
-rw-r--r-- 1 dev dev    11358 4月  21  2022 LICENSE
-rwxr-xr-x 1 dev dev 19172733 4月  21  2022 prometheus-webhook-dingtalk*
$ cp config.example.yml config.yml
$ vim config.yml
## Request timeout
# timeout: 5s

## Uncomment following line in order to write template from scratch (be careful!)
#no_builtin_template: true

## Customizable templates path
#templates:
#  - contrib/templates/legacy/template.tmpl

## You can also override default template using `default_message`
## The following example to use the 'legacy' template from v0.3.0
#default_message:
#  title: '{{ template "legacy.title" . }}'
#  text: '{{ template "legacy.content" . }}'

## Targets, previously was known as "profiles"
#targets:
#  webhook1:
#    url: https://oapi.dingtalk.com/robot/send?access_token=xxxxxxxxxxxx
#    # secret for signature
#    secret: SEC000000000000000000000
#  webhook2:
#    url: https://oapi.dingtalk.com/robot/send?access_token=xxxxxxxxxxxx
#  webhook_legacy:
#    url: https://oapi.dingtalk.com/robot/send?access_token=xxxxxxxxxxxx
#    # Customize template content
#    message:
#      # Use legacy template
#      title: '{{ template "legacy.title" . }}'
#      text: '{{ template "legacy.content" . }}'
#  webhook_mention_all:
#    url: https://oapi.dingtalk.com/robot/send?access_token=xxxxxxxxxxxx
#    mention:
#      all: true
#  webhook_mention_users:
#    url: https://oapi.dingtalk.com/robot/send?access_token=xxxxxxxxxxxx
#    mention:
#      mobiles: ['156xxxx8827', '189xxxx8325']

targets:
  webhook1:
    # 来自 DingTalk 群机器人的 webhook
    url: https://oapi.dingtalk.com/robot/send?access_token=ece577a445d2494a3d1abd7c46522ef733346a3f440bddef7fa979f63d904d7b

# 启动
$ ./prometheus-webhook-dingtalk --web.listen-address=':8060' --web.enable-ui --web.enable-lifecycle --config.file=config.yml > webhook.log 2>&1 &
[1] 3024637

# 查看启动日志
$ cat webhook.log
ts=2024-07-31T06:26:49.104Z caller=main.go:59 level=info msg="Starting prometheus-webhook-dingtalk" version="(version=2.1.0, branch=HEAD, revision=8580d1395f59490682fb2798136266bdb3005ab4)"
ts=2024-07-31T06:26:49.104Z caller=main.go:60 level=info msg="Build context" (gogo1.18.1,userroot@177bd003ba4d,date20220421-08:19:05)=(MISSING)
ts=2024-07-31T06:26:49.104Z caller=coordinator.go:83 level=info component=configuration file=config.yml msg="Loading configuration file"
ts=2024-07-31T06:26:49.104Z caller=coordinator.go:91 level=info component=configuration file=config.yml msg="Completed loading of configuration file"
ts=2024-07-31T06:26:49.104Z caller=main.go:97 level=info component=configuration msg="Loading templates" templates=
ts=2024-07-31T06:26:49.104Z caller=main.go:113 component=configuration msg="Webhook urls for prometheus alertmanager" urls=http://localhost:8060/dingtalk/webhook1/send
ts=2024-07-31T06:26:49.104Z caller=web.go:208 level=info component=web msg="Start listening for connections" address=:8060
```

### 3.7.3、Alertmanager 配置

```yaml
# alertmanager.yml

global:

# 告警路由规则
# 目前只有根路由，所以所有的告警都会发给默认接收者 webhook
route:
  group_by: ['alertname']
  group_wait: 30s
  group_interval: 5m
  repeat_interval: 1h
  # receiver: 'webhook'
  # receiver: 'email'
  receiver: 'dingtalk'
  
receivers:
  - name: 'webhook'
    webhook_configs:
        # webhook.site 复制的 URL
      - url: 'https://webhook.site/f552c8fc-ce6f-418c-8226-0c45c9d60a33'
        # 发送告警已解决消息
        send_resolved: false
        
  - name: 'email'
    webhook_configs:
      - to: ycz@qq.com
        send_resolved: true

  - name: 'dingtalk'
    webhook_configs:
      - url: 'http://localhost:8060/dingtalk/webhook1/send'
        send_resolved: true
```



# 