---
title: golang-中间件
link: golang-中间件
categories:
- Golang
date: 2024-04-22T16:22:49+08:00
date_modified: 2024-04-22T16:22:49+08:00
---

## 参考项目
xh-middleware

## trace
<img src="/images/2024/04/22/trace.png" alt="" title="trace.png" border="0" width="1470" height="1376" />

### TracerProvider
主要负责创建Tracer

### Tracer
表示一次完整的追踪链路,由一个或多个span组成,创建方法如下
```
gtrace.NewTracer(tracerName)
```

### Span
Span是一条追踪链路中的基本组成要素，一个span表示一个独立的工作单元，比如可以表示一次函数调用，一次http请求等等。span会记录如下基本要素,创建方法如下
```
gtrace.NewSpan(ctx, spanName, opts...)
```

### Attributes
Attributes以K/V键值对的形式保存用户自定义标签，主要用于链路追踪结果的查询过滤。例如：  http.method="GET",http.status_code=200。其中key值必须为字符串，value必须是字符串，布尔型或者数值型。  span中的Attributes仅自己可见，不会随着  SpanContext传递给后续span。 设置Attributes方式例如：
```
span.SetAttributes(
	label.String("http.remote", conn.RemoteAddr().String()),
	label.String("http.local", conn.LocalAddr().String()),
)
```

### Events
Events与Attributes类似，也是K/V键值对形式。与Attributes不同的是，Events还会记录写入Events的时间，因此Events主要用于记录某些事件发生的时间。Events的key值同样必须为字符串，但对value类型则没有限制。例如：
```
span.AddEvent("http.request", trace.WithAttributes(
	label.Any("http.request.header", headers),
	label.Any("http.request.baggage", gtrace.GetBaggageMap(ctx)),
	label.String("http.request.body", bodyContent),
))
```

### SpanContext
SpanContext携带着一些用于跨服务通信的（跨进程）数据，主要包含：

- 足够在系统中标识该span的信息，比如：span_id, trace_id

- `Baggage` - 为整条追踪连保存跨服务（跨进程）的K/V格式的用户自定义数据。Baggage 与 Attributes 类似，也是 K/V 键值对。与 Attributes 不同的是,其key跟value都只能是字符串格式

- Baggage不仅当前span可见，其会随着SpanContext传递给后续所有的子span。要小心谨慎的使用Baggage - 因为在所有的span中传递这些K,V会带来不小的网络和CPU开销

### Propagator
Propagator传播器用于端对端的数据编码/解码，例如：Client到Server端的数据传输，TraceId、SpanId和Baggage也是需要通过传播器来管理数据传输。业务端开发者往往对Propagator无感知，只有中间件/拦截器的开发者需要知道它的作用。OpenTelemetry的标准协议实现库提供了常用的TextMapPropagator，用于常见的文本数据端到端传输。此外，为保证TextMapPropagator中的传输数据兼容性，不应当带有特殊字符，具体请参考：`https://github.com/open-telemetry/opentelemetry-specification/blob/main/specification/context/api-propagators.md`,xh-middleware使用全局进行定义
```
// defaultTextMapPropagator is the default propagator for context propagation between peers.
defaultTextMapPropagator = propagation.NewCompositeTextMapPropagator(
	propagation.TraceContext{},
	propagation.Baggage{},
)
```

### jager部署
http://localhost:16686
```
docker run --rm --name jaeger \
  -e COLLECTOR_ZIPKIN_HOST_PORT=:9411 \
  -p 6831:6831/udp \
  -p 6832:6832/udp \
  -p 5778:5778 \
  -p 16686:16686 \
  -p 4317:4317 \
  -p 4318:4318 \
  -p 14250:14250 \
  -p 14268:14268 \
  -p 14269:14269 \
  -p 9411:9411 \
  jaegertracing/all-in-one:1.55
```

### 单体函数示例
```
func main() {
	const (
		serviceName = "otlp-http-client"
		endpoint    = "tracing-analysis-dc-hz.aliyuncs.com"
		path        = "adapt_******_******/api/otlp/traces"
	)

    // 自动生成traceID
	var ctx = gctx.New()
    shutdown, err := otlphttp.Init(serviceName, endpoint, path)
	if err != nil {
		g.Log().Fatal(ctx, err)
	}
	defer shutdown()

	ctx, span := gtrace.NewSpan(ctx, "main")
	defer span.End()

	// Trace 1.
	user1 := GetUser(ctx, 1)
	g.Dump(user1)

	// Trace 2.
	user100 := GetUser(ctx, 100)
	g.Dump(user100)
}
```

方法间创建
```
// GetUser retrieves and returns hard coded user data for demonstration.
func GetUser(ctx context.Context, id int) g.Map {
	ctx, span := gtrace.NewSpan(ctx, "GetUser")
	defer span.End()
	m := g.Map{}
	gutil.MapMerge(
		m,
		GetInfo(ctx, id),
		GetDetail(ctx, id),
		GetScores(ctx, id),
	)
	return m
}

// GetInfo retrieves and returns hard coded user info for demonstration.
func GetInfo(ctx context.Context, id int) g.Map {
	ctx, span := gtrace.NewSpan(ctx, "GetInfo")
	defer span.End()
	if id == 100 {
		return g.Map{
			"id":     100,
			"name":   "john",
			"gender": 1,
		}
	}
	return nil
}

// GetDetail retrieves and returns hard coded user detail for demonstration.
func GetDetail(ctx context.Context, id int) g.Map {
	ctx, span := gtrace.NewSpan(ctx, "GetDetail")
	defer span.End()
	if id == 100 {
		return g.Map{
			"site":  "https://goframe.org",
			"email": "john@goframe.org",
		}
	}
	return nil
}

// GetScores retrieves and returns hard coded user scores for demonstration.
func GetScores(ctx context.Context, id int) g.Map {
	ctx, span := gtrace.NewSpan(ctx, "GetScores")
	defer span.End()
	if id == 100 {
		return g.Map{
			"math":    100,
			"english": 60,
			"chinese": 50,
		}
	}
	return nil
}
```