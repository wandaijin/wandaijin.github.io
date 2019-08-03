---
layout: post
title:  "microk8s使用cert-manager自动管理let's encrypt证书"
categories: k8s microk8s
---

1. 安装cert-manager版本0.8.1，也可以使用helm命令进行安装

```bash
# microk8s.kubectl create namespace cert-manager
# microk8s.kubectl label namespace cert-manager certmanager.k8s.io/disable-validation=true
# microk8s.kubectl apply -f https://github.com/jetstack/cert-manager/releases/download/v0.8.1/cert-manager.yaml
# microk8s.kubectl get pods --namespace cert-manager
NAME                                       READY   STATUS    RESTARTS   AGE
cert-manager-5d5cc69d6-9zvgw               1/1     Running   0          53s
cert-manager-cainjector-7688bf9689-r4nrc   1/1     Running   0          53s
cert-manager-webhook-dfcbcc64b-vljnw       1/1     Running   0          53s
# microk8s.kubectl describe pods cert-manager-5d5cc69d6-9zvgw --namespace cert-manager #查看版本
···
        Image:         quay.io/jetstack/cert-manager-controller:v0.8.1
···
```

2. 验证cert-manager是否正确安装

```bash
# cat <<EOF > test-resources.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: cert-manager-test
---
apiVersion: certmanager.k8s.io/v1alpha1
kind: Issuer
metadata:
  name: test-selfsigned
  namespace: cert-manager-test
spec:
  selfSigned: {}
---
apiVersion: certmanager.k8s.io/v1alpha1
kind: Certificate
metadata:
  name: selfsigned-cert
  namespace: cert-manager-test
spec:
  commonName: example.com
  secretName: selfsigned-cert-tls
  issuerRef:
    name: test-selfsigned
EOF
# microk8s.kubectl apply -f test-resources.yaml
# microk8s.kubectl describe certificate -n cert-manager-test
...
Events:
  Type    Reason      Age   From          Message
  ----    ------      ----  ----          -------
  Normal  CertIssued  20s   cert-manager  Certificate issued successfully
# microk8s.kubectl delete -f test-resources.yaml #删除测试相关资源
# rm test-resources.yaml
```

3. 配置let's encrypt

0.8版本与0.7版本的配置发生了变化，增加了.spec.solvers配置项目.

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
# microk8s.kubectl describe clusterissuer letsencrypt-prod
# microk8s.kubectl get clusterissuer
```

创建Certificate对象，注意：0.8版本去掉了.spec.acme, 官方文档部分地方没有及时更新。

```bash
# microk8s.kubectl get certificate
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
# microk8s.kubectl describe certificate acme-crt
···
 Normal   CertIssued     5m37s (x2099 over 25m)  cert-manager  Certificate issued successfully
···
```

4. ingress使用let's encrypt

```bash
# cat <<EOF | microk8s.kubectl apply -f -
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: web-ingress
spec:
  tls:
    - hosts:
      - demo.example.com
      # This assumes acme-crt-secret exists and the SSL
      # certificate contains a CN for foo.bar.com
      secretName: acme-crt-secret
  rules:
    - host: demo.example.com
      http:
        paths:
        - path: /web
          backend:
            serviceName: my-web
            servicePort: 80
EOF
# microk8s.kubectl get ingress
```

5. 验证是否配置成功

```bash
# curl -vk https://demo.example.com
...
* Server certificate:
*  subject: CN=demo.example.com
*  start date: Aug  3 11:54:49 2019 GMT
*  expire date: Nov  1 11:54:49 2019 GMT
*  issuer: C=US; O=Let's Encrypt; CN=Let's Encrypt Authority X3
*  SSL certificate verify ok.
...
```

### 参考资料
https://docs.cert-manager.io/en/latest/getting-started/install/kubernetes.html
https://docs.cert-manager.io/en/latest/tasks/issuers/index.html
