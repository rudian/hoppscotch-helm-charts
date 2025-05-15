# Kubernetes 部署 Hoppscotch Community Version 平替 Postman

[原文章](https://www.pangzai.win/kubernetes%e9%83%a8%e7%bd%b2hoppscotch%e5%b9%b3%e6%9b%bfpostman/)
<br/><br/>
## 安装步骤

1.  **创建 namespace**
    ```bash
    kubectl create ns hoppscotch
    ```

2.  **创建 Cloudflare API Token 和 Issuer**
    我使用的是 cert-manager 自动生成 SSL 证书，所以需要在 `hoppscotch` 这个 namespace 当中，创建 Cloudflare API Token 和 Issuer。
    参考： [https://www.pangzai.win/kubernetes-%e4%bd%bf%e7%94%a8-cert-manager-%e8%87%aa%e5%8a%a8%e7%ad%be%e5%8f%91-https-%e8%af%81%e4%b9%a6-%e3%80%903%e3%80%91/](https://www.pangzai.win/kubernetes-%e4%bd%bf%e7%94%a8-cert-manager-%e8%87%aa%e5%8a%a8%e7%ad%be%e5%8f%91-https-%e8%af%81%e4%b9%a6-%e3%80%903%e3%80%91/)

3.  **创建 Certificate**
    创建Hoppscotch需要1个subdomain因为我开启了subpath。 如果你的enableSubpathBasedAccess是false的话就需要3个domain了
    ```yaml
    apiVersion: cert-manager.io/v1
    kind: Certificate
    metadata:
      name: hoppscotch-cert-tls
      namespace: hoppscotch
    spec:
      dnsNames:
        - hcbackend.pangzai.win
      secretName: hoppscotch-cert-tls
      issuerRef:
        name: letsencrypt-dns01
    ```

4.  **Clone Helm Charts**
    Clone 这个 Helm Charts，我是从官方拷贝出来的，然后修了一点 bug，因为官方提供的 service 和 deployment 无法连接，所以我修了，使用我提供的 Helm Charts 就好。
    **官方 Helm Charts:** [https://github.com/hoppscotch/helm-charts](https://github.com/hoppscotch/helm-charts)
    ```bash
    git clone https://github.com/rudian/hoppscotch-helm-charts.git
    ```

5.  **进入 Helm Charts 目录**
    ```bash
    cd hoppscotch-helm-charts
    ```

6.  **修改 `your_config.yaml`**
    修改 `your_config.yaml` 到你自己的参数，你也可以参考原版的 `yaml`。我的版本是设置了 SSL 的。
    必须设置 `email`，因为一开始登入只能使用 email 发送的方式来登入。
    <br/><br/>
    如果你想要desktop app要使用的话，那么enableSubpathBasedAccess就必须是true
    <br/><br/>
    **原版的 `yaml` 路径:** `hoppscotch-helm-charts/blob/main/charts/shc/values.yaml`
    <br/>
    **注意** 原版yaml的例子就是关掉了subpath，所以才需要3个不同的domain
    <br/><br/>
    如果你想要desktop app要使用的话，那么enableSubpathBasedAccess就必须是true（必须开启subpath）
    <br/><br/>
    关于 PostgreSQL 的 `storageClass`，由于我使用的是阿里云的 ACK，所以默认有好几种 `storageClass` 可以选，而且最低起步必须是 20GB。
    <br/><br/>
    ![](https://www.pangzai.win/wp-content/uploads/2025/05/image-41.png)

7.  **使用 Helm 安装 Hoppscotch**
    修改完 `your_config.yaml` 之后就是在 Kubernetes 集群当中执行 Helm 来安装 Hoppscotch。
    如果没有附上最后的 `./your_config.yaml`，那么程序就会拿原版的 `yaml`。
    ```bash
    helm install community-hoppscotch ./charts/shc -f ./your_config.yaml
    ```
    如果你之后有进行任何更改 `your_config.yaml` 的话，可以执行以下的命令更新 setting。
    ```bash
    helm upgrade community-hoppscotch ./charts/shc -f ./your_config.yaml
    ```
    ![](https://www.pangzai.win/wp-content/uploads/2025/05/image-42.png)
    如果你想要删除整个安装的 Hoppscotch 的话，可以使用以下的命令。
    ```bash
    helm uninstall community-hoppscotch
    ```

8.  **访问 Hoppscotch Admin Panel**
    安装完成之后就可以进入 `https://hcbackend.pangzai.win/admin`，然后填写你的 email，系统就会发出登入 token 给你的。你点击 email 就能登入到 admin panel 了。
    <br/><br/>
    **注意：** 第一个 email 进入 admin panel 的就是 admin 了，第二个就不是了。
    <br/><br/>
    **我遇到的问题：** 我一开始的的设定subpath是关闭的，所以登入到admin panel之后去到setup页面遇到了CORS的问题，我尝试了在ingress当中允许cors，设了之后还是一样无法解决问题，最终我使用了chrome插件来解决。这个setup页面第一次进入设定之后就不会再进入了。如果你开启subpath的话就不会遇到CORS的问题，因为都是用着同样的domain。
    <br/><br/>
    **允许 CORS 的 Chrome 插件：** [https://chromewebstore.google.com/detail/allow-cors-access-control/lhobafahddgcelffkeicbaginigeejlf?hl=en](https://chromewebstore.google.com/detail/allow-cors-access-control/lhobafahddgcelffkeicbaginigeejlf?hl=en)