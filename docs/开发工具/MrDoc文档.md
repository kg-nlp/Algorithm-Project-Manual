---
sort: 1
---

# MrDoc文档

* [算法开发手册](https://kg-nlp.github.io/Algorithm-Project-Manual/开发工具/MrDoc文档.html)

* [个人知乎](https://www.zhihu.com/people/zhangyj-n)

##  普通安装

集群内安装

1. 创建配置文件

   - ```sh
     mkdir mrdoc
     cd /root/mrdoc
     touch mrdoc-deployment.yaml
     touch mrdoc-service.yaml
     ```

   - mrdoc-deployment.yaml

     - ```yaml
       apiVersion: apps/v1
       kind: Deployment
       metadata:
         name: mrdoc-deployment
       spec:
         selector:
           matchLabels:
             app: mrdoc-pod
         template:
           metadata:
             labels:
               app: mrdoc-pod
           spec:
             containers:
             - name: mrdoc
               image: jonnyan404/mrdoc-alpine:0.8.0
               ports:
               - containerPort: 10086
       ```

   - mrdoc-service.yaml

     - ```yaml
       apiVersion: v1
       kind: Service
       metadata:
         name: mrdoc-service
       spec:
         selector:
           app: mrdoc-pod
         type: NodePort # service类型
         ports:
         - port: 80
           nodePort: 30086 # 指定绑定的node的端口(默认的取值范围是：30000-32767), 如果不指定，会默认分配
           targetPort: 10086
       ```

   - 执行

     - ```
       kubectl create -f mrdoc-deployment.yaml
       kubectl create -f mrdoc-service.yaml
       ```

   - 开通端口服务

     - 元哥开通

2. 访问

   > http://10.127.91.94:30086/

## 挂载数据卷安装

1. 创建目录，配置文件

```sh
mkdir /root/data/mrdoc/media -p 
cd /root/data/mrdoc/
vim mrdoc-deployment.yaml
vim mrdoc-service.yaml
```

2. deployment配置文件

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mrdoc-d
spec:
  selector:
    matchLabels:
      app: mrdoc-p
  template:
    metadata:
      labels:
        app: mrdoc-p
    spec:
      containers:
      - name: mrdoc
        image: jonnyan404/mrdoc-alpine:0.8.0
        ports:
        - containerPort: 10086
        volumeMounts:
        - name: mrdoc-config
          mountPath: /app/MrDoc/config
        - name: mrdoc-media
          mountPath: /app/MrDoc/media
      volumes:
      - name: mrdoc-config
        nfs:
          server: 10.0.2.4  #nfs服务器地址
          path: /root/data/mrdoc/config #共享文件路径
          # type: DirectoryOrCreate
      - name: mrdoc-media
        nfs:
          server: 10.0.2.4
          path: /root/data/mrdoc/media
```

3. service配置文件

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mrdoc-s
spec:
  selector:
    app: mrdoc-p
  type: NodePort # service类型
  ports:
  - port: 80
    nodePort: 30086 # 指定绑定的node的端口(默认的取值范围是：30000-32767), 如果不指定，会默认分配
    targetPort: 10086
```

4. 访问：

http://10.127.91.94:30086/



## 使用说明

1. 访问界面
   - ![](MrDoc文档.assets\1.jpg)

2. 获取密码

   - 获取pod

     - ```sh
       kubectl get pods
       ```

     - ![](MrDoc文档.assets\2.jpg)

   - 获取密码

     - ```sh
       kubectl logs -f mrdoc-deployment-54b6d85b95-8ltpn -c mrdoc 2>&1|grep pwd
       
       ```
       
       > -- First container startup --user:admin pwd:560e1a12

   - 登录mrdoc界面修改密码

     - 修改后用户名：admin  密码：glodon

3. 上传文档

   - 进入**个人中心**，在**我的文集**下选择**导入文集** ，点击**Markdown.zip** 打开资源管理器上传自定义文件包
     - ![](MrDoc文档.assets\3.jpg)

4. 查看文档
   - 进入**个人中心**，在我的文档下选择**文档管理**，点击刚上传的文档
     - ![](MrDoc文档.assets\4.jpg)



## 参考链接

[github](https://github.com/zmister2016/MrDoc/blob/master/README-zh.md)

[安装手册](https://doc.mrdoc.pro/doc/515/)

[Docker安装mrdoc](https://www.mrdoc.fun/project-1/doc-18/)

[使用手册](https://doc.mrdoc.pro/project/54/)

[文档效果示例](https://doc.mrdoc.pro/project/20/)

