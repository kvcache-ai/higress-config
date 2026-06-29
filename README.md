# Higress YAML 配置

## 全局配置

### 创建全局 EnvoyFilter

```Shell
kubectl create -f bases/runtime/envoyfilters/global
```

### 配置 access\.log / upstream\-attempt\.log 日志轮转

#### 创建日志轮转规则

```Shell
kubectl create -f bases/runtime/configmaps/higress-logrotate.yaml
```

#### Patch deploy/higress\-gateway

执行以下命令，这会触发 higress\-gateway 滚动更新

```Shell
kubectl -n higress-system patch deployment higress-gateway --type='json' -p='[
    {
      "op": "add",
      "path": "/spec/template/spec/containers/0/securityContext/capabilities/add/-",
      "value": "SETUID"
    },
    {
      "op": "add",
      "path": "/spec/template/spec/containers/0/securityContext/capabilities/add/-",
      "value": "SETGID"
    },
    {
      "op": "add",
      "path": "/spec/template/spec/volumes/-",
      "value": {
        "name": "higress-logrotate",
        "configMap": {
          "name": "higress-logrotate"
        }
      }
    },
    {
      "op": "add",
      "path": "/spec/template/spec/volumes/-",
      "value": {
        "name": "logrotate-state",
        "emptyDir": {}
      }
    },
    {
      "op": "add",
      "path": "/spec/template/spec/containers/0/volumeMounts/-",
      "value": {
        "name": "higress-logrotate",
        "mountPath": "/etc/logrotate.d/higress-logrotate",
        "subPath": "higress-logrotate",
        "readOnly": true
      }
    },
    {
      "op": "add",
      "path": "/spec/template/spec/containers/0/volumeMounts/-",
      "value": {
        "name": "logrotate-state",
        "mountPath": "/var/lib/logrotate"
      }
    }
  ]'
```

### 配置全局插件

在 Higress console 左侧导航栏点击“插件配置”，找到以下插件，将对应配置写入，开启插件并保存

#### Key认证

修改其中的 API Key 和其对应的名称

```Shell
bases/runtime/wasmplugins/global/key-auth-1.0.0.yaml
```

#### 基于Key集群限流

替换其中的 API Key，与“Key认证”中的 API Key 保持一致，可选择 qps 限流（query\_per\_second ）或 qpm 限流（query\_per\_minute），并设置合适的限流大小

```Shell
bases/runtime/wasmplugins/global/cluster-key-rate-limit-1.0.0.yaml
```

#### ai\-history

```Shell
bases/runtime/wasmplugins/global/approaching-ai-history-1.0.0.yaml
```

#### ai\-statistics

```Shell
bases/runtime/wasmplugins/global/approaching-ai-statistics-1.0.0.yaml
```

#### 请求/响应转换

```Shell
bases/runtime/wasmplugins/global/transformer-1.0.0.yaml
```

## 路由配置

### 创建 ServiceEntry / DestinationRule crd

```Shell
kubectl create -f bases/crd
```

### 创建 ServiceEntry

其中默认是 mock 服务，修改成对应服务名

```Shell
kubectl create -f bases/runtime/serviceentries/glm.yaml
```

### 创建 DestinationRule

```Shell
kubectl create -f bases/runtime/destinationrules/glm.yaml
```

### 创建 Ingress

```Shell
kubectl create -f bases/runtime/ingress/glm.yaml
```

### 创建路由 EnvoyFilter

```Shell
kubectl create -f bases/runtime/envoyfilters/routes/glm-cluster.yaml
kubectl create -f bases/runtime/envoyfilters/routes/glm-router.yaml
```

### 配置路由插件

创建路由之后，在 Higress console 左侧导航栏点击“路由配置”，找到对应路由，点击“策略”，找到以下插件，将对应配置写入，开启插件并保存

#### Key认证

修改成“Key认证”全局配置中填写的 API Key 名

```Shell
bases/runtime/wasmplugins/routes/key-auth-1.0.0.yaml
```

#### chat\-completion\-session\-id

```Shell
bases/runtime/wasmplugins/routes/chat-completion-session-id-1.0.0.yaml
```

#### request\-mapper

```Shell
bases/runtime/wasmplugins/routes/request-mapper-1.0.0.yaml
```

#### chat\-images\-limit

如果没有图片限流要求，不需要配置

```Shell
bases/runtime/wasmplugins/routes/chat-images-limit-1.0.0.yaml
```
