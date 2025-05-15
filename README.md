#Kubernetes部署Hoppscotch Community Version平替Postman
[原文章](https://www.pangzai.win/kubernetes%e9%83%a8%e7%bd%b2hoppscotch%e5%b9%b3%e6%9b%bfpostman/)

1\. 创建namespace

    kubectl create ns hoppscotch

2\. 我使用的是cert manager自动生成ssl证书，所以需要在这个新那么space当中，创建cloudflare api token 和 issuer

参考： [https://www.pangzai.win/kubernetes-%e4%bd%bf%e7%94%a8-cert-manager-%e8%87%aa%e5%8a%a8%e7%ad%be%e5%8f%91-https-%e8%af%81%e4%b9%a6-%e3%80%903%e3%80%91/](https://www.pangzai.win/kubernetes-%e4%bd%bf%e7%94%a8-cert-manager-%e8%87%aa%e5%8a%a8%e7%ad%be%e5%8f%91-https-%e8%af%81%e4%b9%a6-%e3%80%903%e3%80%91/)

3\. 创建certificate

创建Hoppscotch需要3个subdomain , 分别是frontend , backend 和 admin

    apiVersion: cert-manager.io/v1
    kind: Certificate
    metadata:
      name: hoppscotch-cert-tls
      namespace: hoppscotch
    spec:
      dnsNames:
        - hcfrontend.pangzai.win
        - hcbackend.pangzai.win
        - hcadmin.pangzai.win
      secretName: hoppscotch-cert-tls
      issuerRef:
        name: letsencrypt-dns01

4\. git clone 这个helm charts , 我是从官方拷贝出来的，然后修了一点bug，因为官方提供的service和deployment无法连接，所以我修了，使用我提供的helm charts就好。  
**官方HelmCharts :** [https://github.com/hoppscotch/helm-charts](https://github.com/hoppscotch/helm-charts)

    git clone https://github.com/rudian/hoppscotch-helm-charts.git

5\. 去到这个path内

    cd hoppscotch-helm-charts

6\. 修改your\_config.yaml 到你自己的参数，你也可以参考原版的yaml。我的版本是设置了SSL的

必须设置email，因为一开始登入只能使用email发送的方式来登入。

**原版的yaml路径：**hoppscotch-helm-charts/blob/main/charts/shc/values.yaml

关于postgres的storageClass由于我使用的是阿里云的ACK，所以默认有好几种storageClass可以选，而且最低起步必须是20GB。

![](https://www.pangzai.win/wp-content/uploads/2025/05/image-41.png)


7\. 修改完your\_config.yaml之后就是在kubernetes cluster当中执行helm来安装Hoppscotch

如果没有附上最后的 ./your\_config.yaml 那么程序就会拿原版的yaml

    helm install community-hoppscotch ./charts/shc -f ./your_config.yaml

如果你之后有进行任何更改your\_config.yaml的话，可以执行以下的命令更新setting

    helm upgrade community-hoppscotch ./charts/shc -f ./your_config.yaml

![](https://www.pangzai.win/wp-content/uploads/2025/05/image-42.png)

如果你想要删除整个安装的Hoppscotch的话，可以使用以下的命令

    helm uninstall community-hoppscotch  

8\. 安装完成之后就可以进入hcadmin.pangzai.win , 然后填写你的email，系统就会发出登入token给你的。你点击email就能登入到admin panel了。  
  
**注意：** 第一个email进入admin panel的就是admin了，第二个就不是了。

**我遇到的问题：**一开始登入到admin panel，然后去到setup页面遇到了CORS的问题，我尝试了在ingress当中允许cors，设了之后还是一样无法解决问题，最终我使用了chrome插件来解决。这个setup页面第一次进入设定之后就不会再进入了。

**允许CORS的chrome插件：**[https://chromewebstore.google.com/detail/allow-cors-access-control/lhobafahddgcelffkeicbaginigeejlf?hl=en](https://chromewebstore.google.com/detail/allow-cors-access-control/lhobafahddgcelffkeicbaginigeejlf?hl=en)
