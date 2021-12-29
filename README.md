kube-prometheus的增强配置。
利用prometheus的kubernetes_sd_config服务发现功能和Relabel功能实现：
自动发现pod，并从pod的annotation中识别抓取规则，然后进行抓取。

另外还添加了Prometheus、grafana的NodePort配置。

最终实现的效果如下，
这是一个pushGateway的pod的配置,则Prometheus会通过其19091端口访问/metrics路径获取其指标数据
```yaml
  annotations:
    prometheus.io/scrape: "true"
    prometheus.io/scheme: "http"
    prometheus.io/path: "/metrics"
    prometheus.io/port: "19091"
```

参考文章：https://linshenkx.github.io/kube-prometheus-bigdata
## 使用说明
### create
该文件夹为新增配置，可直接apply
注意：prometheus-additional.yaml.file不用apply，

prometheus-additional.yaml(.file)到additional-scrape-configs.yaml的转换参考：
https://github.com/prometheus-operator/prometheus-operator/blob/main/Documentation/additional-scrape-config.md

命令为：
```shell
kubectl create secret generic additional-scrape-configs --from-file=prometheus-additional.yaml --dry-run -oyaml > additional-scrape-configs.yaml

```

### edit
该文件夹为在原kube-prometheus配置基础上进行修改，需根据注释或对比原配置获取需要修改内容，进行手动修改

修改的主要内容如下：
#### prometheus-prometheus.yaml
主要是添加additional-scrape-configs
```yaml
  additionalScrapeConfigs:
    name: additional-scrape-configs
    key: prometheus-additional.yaml
```
#### prometheus-clusterRole.yaml
主要是添加对k8s的资源访问权限
```yaml
- apiGroups: [""]
  resources:
  - nodes
  - services
  - endpoints
  - pods
  verbs: ["get", "list", "watch"]
- apiGroups: [""]
  resources:
    - configmaps
  verbs: ["get"]
```

