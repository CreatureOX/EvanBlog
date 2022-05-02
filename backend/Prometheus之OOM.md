## 现象
当数据量过大时访问服务prometheus端口拿不到响应

## 分析
断点调试追踪到 MicrometerCollector 递归调用 ConcatSpliterator.estimateSize()，当数据量过大时有OOM风险

## 解决 
* micrometer.version=1.0.4时，MicrometerCollector 递归调用 ConcatSpliterator.estimateSize()
* micrometer.version=1.0.7时官方修复了该问题[[769](https://github.com/micrometer-metrics/micrometer/issues/769)]

## 代码片段

```java
//v1.0.3
return children.stream()
   .flatMap(child -> child.samples(conventionName, tagKeys))
   .collect(Collectors.groupingBy(
       Family::getConventionName,
       // 下面这行 Stream.concat
       Collectors.reducing((a, b) -> new Family(a.type, a.conventionName, Stream.concat(a.samples, b.samples)))))
   .values().stream()
   .map(Optional::get)
   .map(family -> new MetricFamilySamples(family.conventionName, family.type, help, family.samples.collect(toList())))
   .collect(toList());
```

```java
//v1.0.7
Map<String, Family> families = new HashMap<>();
 
for (Child child : children) {
    child.samples(conventionName, tagKeys).forEach(family -> {
        // 弃用 Stream.concat，改用for循环 + HashMap 实现
        families.compute(family.getConventionName(), (name, matchingFamily) -> matchingFamily != null ?
            matchingFamily.addSamples(family.samples) : family);
    });
}
 
return families.values().stream()
        .map(family -> new MetricFamilySamples(family.conventionName, family.type, help, family.samples))
        .collect(toList());
```
