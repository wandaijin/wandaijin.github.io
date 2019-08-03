---
layout: post
title:  "microk8s使用cert-manager配置letsencrypt时order一直处于pending状态"
categories: k8s microk8s
---

### 问题

在按照官方文档创建Issuer和Certificate时，发现order一直卡在pending状态。

```bash
# microk8s.kubectl get clusterissuer
# cat <<EOF | microk8s.kubectl apply -f -
apiVersion: certmanager.k8s.io/v1alpha1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    # You must replace this email address with your own.
    # Let's Encrypt will use this to contact you about expiring
    # certificates, and issues related to your account.
    email: admin@example.com
    server: https://acme-v02.api.letsencrypt.org/directory
    privateKeySecretRef:
      # Secret resource used to store the account's private key.
      name: letsencrypt-prod
    # Add a single challenge solver, HTTP01 using nginx
    solvers:
    - selector: {}
    - http01:
        ingress:
          class: nginx
EOF
# cat <<EOF | microk8s.kubectl apply -f -
apiVersion: certmanager.k8s.io/v1alpha1
kind: Certificate
metadata:
  name: acme-crt
spec:
  secretName: acme-crt-secret
  dnsNames:
  - foo.example.com
  - bar.example.com
  acme:
    config:
    - http01:
        ingressClass: nginx
      domains:
      - foo.example.com
      - bar.example.com
  issuerRef:
    name: letsencrypt-prod
    # We can reference ClusterIssuers by changing the kind here.
    # The default value is Issuer (i.e. a locally namespaced Issuer)
    kind: ClusterIssuer
EOF
# microk8s.kubectl describe certificate acme-crt
# microk8s.kubectl describe order acme-crt-<yournumber>
···
Status:
  Finalize URL:  https://acme-staging-v02.api.letsencrypt.org/acme/finalize/xxx/xxx
  State:         pending
  URL:           https://acme-staging-v02.api.letsencrypt.org/acme/order/xxx/xxx
Events:          <none>
```

### 原因

查看pod日志，提示报错：

>...constructing old format Challenge resource for authorization: ACME server does not allow selected challenge type or no provider is configured for domain..

查看[代码文件](https://github.com/jetstack/cert-manager/blob/0050432f08651e75469def7cfcf963979838af9e/pkg/controller/acmeorders/sync.go)发现一段代码`if useOldFormat {..}else{...}`, 发现程序有[升级](https://docs.cert-manager.io/en/latest/tasks/upgrading/upgrading-0.7-0.8.html).
查看当前安装版本:
```bash
# microk8s.kubectl describe pods cert-manager-5d5cc69d6-9zvgw --namespace cert-manager #查看版本
···
        Image:         quay.io/jetstack/cert-manager-controller:v0.8.1
···
```
正好应该使用0.8的配置格式。于是更改配置格式，问题解决。

### 解决方式

使用新的配置格式，去掉原来的0.7版本的.spec.acme配置项。

```bash
# cat <<EOF | microk8s.kubectl apply -f -
apiVersion: certmanager.k8s.io/v1alpha1
kind: Certificate
metadata:
  name: acme-crt
spec:
  secretName: acme-crt-secret
  dnsNames:
  - foo.example.com
  - bar.example.com
  issuerRef:
    name: letsencrypt-prod
    # We can reference ClusterIssuers by changing the kind here.
    # The default value is Issuer (i.e. a locally namespaced Issuer)
    kind: ClusterIssuer
EOF
```

### 参考资料
https://docs.cert-manager.io/en/latest/tasks/upgrading/upgrading-0.7-0.8.html
https://github.com/jetstack/cert-manager/blob/0050432f08651e75469def7cfcf963979838af9e/pkg/controller/acmeorders/sync.go
