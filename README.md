niuxiaodong add 1st
danny-010 add 1st


danny 2end commit

在 Kubernetes (K8s) 集群中部署的 Prometheus 默认可以监控集群内部的资源，但如果你想监控和收集集群外部节点（如物理服务器或虚拟机）上的 node_exporter 数据，可以通过以下步骤实现：

1. 部署 Node Exporter 到外部节点
   首先，确保你在集群外部的节点上已经部署了 node_exporter。具体步骤如下：

下载并运行 node_exporter：

bash
Copy code
wget https://github.com/prometheus/node_exporter/releases/download/v*/node_exporter-*.*-linux-amd64.tar.gz
tar xvfz node_exporter-*.*-linux-amd64.tar.gz
cd node_exporter-*.*-linux-amd64
./node_exporter
确保 node_exporter 监听端口是开放的（通常为 9100），并且外部网络可以访问这个端口。可以通过防火墙配置允许 Prometheus 服务器访问这个端口。

2. 在 Prometheus 配置文件中添加外部节点的node_exporter
   要让 Prometheus 监控集群外的节点，你需要手动在 Prometheus 的配置文件 (prometheus.yml) 中添加外部节点的 node_exporter 作为静态目标。

具体步骤如下：

步骤 1：修改 prometheus.yml 配置文件
在 Prometheus 配置文件 prometheus.yml 中，找到 scrape_configs 部分，添加一个新的 scrape job 来收集外部节点的 node_exporter 数据：

yaml
Copy code
scrape_configs:
# Scrape configuration for monitoring external nodes
- job_name: 'external-node-exporters'
  scrape_interval: 15s
  static_configs:
    - targets:
        - 'external-node-ip-1:9100'
        - 'external-node-ip-2:9100'
          job_name：可以自由命名，例如 external-node-exporters。
          scrape_interval：这是 Prometheus 抓取数据的频率，通常为 15s。
          static_configs.targets：将每个集群外的节点 IP 地址和端口（9100 是 node_exporter 默认端口）作为目标列出。
          步骤 2：重新加载 Prometheus 配置
          在修改了 prometheus.yml 文件后，你需要重新加载 Prometheus 的配置，使其生效。你可以通过以下方式重新加载：

如果 Prometheus 是通过 Kubernetes 部署的，可以使用 kubectl 命令：

bash
Copy code
kubectl exec -n <namespace> <prometheus-pod-name> -- kill -HUP 1
这将向 Prometheus 进程发送信号，触发它重新加载配置。

也可以通过 Prometheus 的 HTTP 接口手动触发重新加载：

bash
Copy code
curl -X POST http://<prometheus-server-ip>:9090/-/reload
3. 在 Prometheus UI 中验证配置
   重新加载配置后，可以通过 Prometheus UI 查看外部节点是否已成功被监控：

访问 Prometheus 的 Web 界面，URL 一般为 http://<prometheus-server-ip>:9090。
在 “Targets” 页面 (在导航栏选择 “Status” > “Targets”)，你应该可以看到名为 external-node-exporters 的新 job，以及对应的外部节点 node_exporter 目标。
如果所有节点的状态是 UP，说明 Prometheus 已经成功开始抓取这些节点的数据。
4. 配置 Grafana 监控外部节点
   如果你使用 Grafana 作为监控前端，可以通过在 Grafana 的仪表板上配置 node_exporter 数据来源来监控这些外部节点。

打开 Grafana，进入你的 Node Exporter 仪表板。
在仪表板的查询中，增加一个 Prometheus 查询，将新添加的外部节点包含进来。例如，使用 Prometheus 查询语句：
promql
Copy code
node_cpu_seconds_total{instance=~"external-node-ip-.*"}
这将显示所有 node_exporter 数据中来自外部节点的监控信息。

总结
部署 node_exporter 到外部节点，并确保 9100 端口对 Prometheus 可访问。
修改 Prometheus 的 prometheus.yml 配置文件，添加外部节点为静态目标。
重新加载 Prometheus 配置，并验证配置是否成功。
可在 Grafana 或 Prometheus UI 中查看和监控这些外部节点的指标。
通过这种方式，你可以轻松监控 Kubernetes 集群之外的节点的状态。