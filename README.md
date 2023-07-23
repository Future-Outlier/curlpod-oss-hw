操作說明
## 可能需要的 prerequiste
我的電腦遇到 Kubernetes 的 Role-Based Access Control (RBAC) 默认不允许 default ServiceAccount 列出 default 命名空间中的 Pods 的問題
```
{
  "kind": "Status",
  "apiVersion": "v1",
  "metadata": {},
  "status": "Failure",
  "message": "pods is forbidden: User \"system:serviceaccount:default:default\" cannot list resource \"pods\" in API group \"\" in the namespace \"default\"",
  "reason": "Forbidden",
  "details": {
    "kind": "pods"
  },
  "code": 403
}
```
因此 在 requirements 裡面新增了兩個 yaml file
```
kubectl apply -f ./requirements/cluster-role.yaml
kubectl apply -f ./requirements/role-binding.yaml
```

## 把 pod 啟動起來
```

kubectl apply -f pod-with-curl.yaml
```
## 進入 curlpod
```
kubectl exec -it curlpod -- /bin/bash
```
## 安裝 curl
```
apt update && apt install -y curl
```
## 查看 dafult namespace 下 pod 的資訊
```
curl -k -H "Authorization: Bearer $(cat /var/run/secrets/kubernetes.io/serviceaccount/token)" https://kubernetes.default.svc/api/v1/namespaces/default/pods
```

### 解釋 curl 在做甚麼
這條 `curl` 命令是用來請求 Kubernetes API 以獲取 `default` 命名空間中的所有 Pods 資訊。下面我會詳細解釋命令中的每個部分：

1. `curl`: 是一個命令行工具，用於發送 HTTP 請求。

2. `-k`: 這是一個 `curl` 的選項，它告訴 `curl` 忽略 SSL 證書驗證。這很有用，特別是當你訪問的服務使用的是自簽名證書時。

3. `-H`: 這是另一個 `curl` 的選項，用於添加 HTTP 請求標頭。它後面跟著的字符串 `"Authorization: Bearer ..."` 就是要添加的標頭。

4. `"Authorization: Bearer ..."`: 這是一個 HTTP 請求標頭，用於認證請求的身份。
   - `Authorization`: 是標頭的名稱。
   - `Bearer`: 表示使用的是 Bearer Token 認證方式。
   - `$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)`: 這部分使用命令替換 (command substitution) 的語法，它將執行 `cat /var/run/secrets/kubernetes.io/serviceaccount/token` 命令，將命令的輸出（即 Service Account 的 token）插入到該位置。在 Kubernetes 中，每個 Pod 都自動掛載了一個 Service Account 的 token，用於與 Kubernetes API 通信。

5. `https://kubernetes.default.svc/api/v1/namespaces/default/pods`: 這是你要請求的 URL。
   - `https://kubernetes.default.svc`: 這是 Kubernetes API 伺服器的內部域名。
   - `/api/v1`: 指定了你要使用的 API 版本。
   - `/namespaces/default/pods`: 這是 API 的端點，表示你想要獲取 `default` 命名空間中的所有 Pods 資訊。

總之，這條命令的目的是使用 Service Account 的 token 作為認證，請求 Kubernetes API 以獲取 `default` 命名空間中的所有 Pods 資訊。