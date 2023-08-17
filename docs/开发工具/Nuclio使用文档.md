---
sort: 2
---



# Nuclio使用文档

* [算法开发手册](https://kg-nlp.github.io/Algorithm-Project-Manual/开发工具/Nuclio使用文档.html)

* [个人知乎](https://www.zhihu.com/people/zhangyj-n)

## 终端部署

### nuctl命令行参数

```sh
root@unode1:~/k8s_nuclio/my_function# nuctl build -h
Build a function
Usage:
  nuctl build function-name [options] [flags]
Aliases:
  build, bu
Flags:
      --base-image string               Name of the base image (default - per-runtime default)
      --build-code-entry-attrs string   JSON-encoded build code entry attributes for the function (default "{}")
      --build-command String            Commands to run when building the processor image  # 构建镜像时的cmd命令,如apt、pip等
      --build-runtime-attrs string      JSON-encoded build runtime attributes for the function (default "{}")
      --code-entry-type string          Type of code entry (for example, "url", "github", "image")
  -f, --file string                     Path to a function-configuration file
      --handler string                  Name of a function handler
  -h, --help                            help for build
  -i, --image string                    Name of a container image (default - the function name)  
      --no-cleanup                      Don't clean up temporary directories
      --no-pull                         Don't pull base images - use local versions
      --offline                         Don't assume internet connectivity exists
      --onbuild-image string            The runtime onbuild image used to build the processor image
      --output-image-file string        Path to output container image of the build
  -p, --path string                     Path to the function's source code  # 本地路径
  -r, --registry string                 URL of a container registry (env: NUCTL_REGISTRY)  # push 仓库
      --runtime string                  Runtime (for example, "golang", "python:3.7")
      --source string                   The function's source code (overrides "path")

Global Flags:
  -k, --kubeconfig string   Path to a Kubernetes configuration file (admin.conf)
  -n, --namespace string    Namespace
      --platform string     Platform identifier - "kube", "local", or "auto" (default "auto")
  -v, --verbose             Verbose output

```



```shell
root@unode1:~/k8s_nuclio/my_function# nuctl deploy -h
Build and deploy a function, or deploy from an existing image
Usage:
  nuctl deploy function-name [flags]
Flags:
      --annotations string                 Additional function annotations (ant1=val1[,ant2=val2,...])
      --base-image string                  Name of the base image (default - per-runtime default)  # 构建镜像的基础镜像
      --build-code-entry-attrs string      JSON-encoded build code entry attributes for the function (default "{}")
      --build-command String               Commands to run when building the processor image  # 构建镜像过程执行的命令
      --build-runtime-attrs string         JSON-encoded build runtime attributes for the function (default "{}")
      --code-entry-type string             Type of code entry (for example, "url", "github", "image")
      --data-bindings string               JSON-encoded data bindings for the function
      --desc string                        Function description
  -d, --disable                            Start the function as disabled (don't run yet)
  -e, --env String                         Environment variables env1=val1  # 环境变量
  -f, --file string                        Path to a function-configuration file
      --fsgroup int                        Run function process with supplementary groups (default -1)
      --handler string                     Name of a function handler  # 启动位置
  -h, --help                               help for deploy
      --http-trigger-service-type string   A Kubernetes ServiceType to apply to the HTTP trigger  # 配置访问模式
  -i, --image string                       Name of a container image (default - the function name)
      --input-image-file string            Path to an input function-image Docker archive file
  -l, --labels string                      Additional function labels (lbl1=val1[,lbl2=val2,...])
      --logger-level string                One of debug, info, warn, error. By default, uses platform configuration
      --max-replicas int                   Maximal number of function replicas (default -1)
      --min-replicas int                   Minimal number of function replicas (default -1)
      --no-cleanup                         Don't clean up temporary directories
      --no-pull                            Don't pull base images - use local versions
      --nodeName string                    Run function pod on a Node by name-matching selection constrain
      --nodeSelector string                Run function pod on a Node by key=value selection constraints (key1=val1[,key2=val2,...])
      --offline                            Don't assume internet connectivity exists
      --onbuild-image string               The runtime onbuild image used to build the processor image
  -p, --path string                        Path to the function's source code
      --platform-config string             JSON-encoded platform specific configuration
      --preemptionPolicy string            Function pod preemption policy
      --priorityClassName string           Indicates the importance of a function Pod relatively to other function pods
      --project-name string                The name of the function's parent project
      --publish                            Publish the function
      --readiness-timeout int              Maximum wait period for the function to be ready, in seconds (default -1)
  -r, --registry string                    URL of a container registry (env: NUCTL_REGISTRY)
      --replicas int                       Set to any non-negative integer to use a static number of replicas (default -1)
      --resource-limit String              Resource restrictions of the format '<resource name>=<quantity>' (for example, 'cpu=3')
      --resource-request String            Requested resources of the format '<resource name>=<quantity>' (for example, 'cpu=3')
      --run-as-group int                   Run function process with group ID (default -1)
      --run-as-user int                    Run function process with user ID (default -1)
      --run-image string                   Name of an existing image to deploy (default - build a new image to deploy)
      --run-registry string                URL of a registry for pulling the image, if differs from -r/--registry (env: NUCTL_RUN_REGISTRY)  # pull 仓库
      --runtime string                     Runtime (for example, "golang", "python:3.7")
      --runtime-attrs string               JSON-encoded runtime attributes for the function
      --source string                      The function's source code (overrides "path")
      --target-cpu int                     Target CPU-usage percentage when auto-scaling (default -1)
      --triggers string                    JSON-encoded triggers for the function
      --volume String                      Volumes for the deployment function (src1=dest1[,src2=dest2,...])

Global Flags:
  -k, --kubeconfig string   Path to a Kubernetes configuration file (admin.conf)
  -n, --namespace string    Namespace
      --platform string     Platform identifier - "kube", "local", or "auto" (default "auto")
  -v, --verbose             Verbose output
```



### 函数+配置文件部署

- 编写脚本文件 my_function.py

- 脚本路径：/root/k8s_nuclio/my_function/my_function.py

  - ```python
    import os
    def my_entry_point(context, event):
        # use the logger, outputting the event body
        context.logger.info_with('Got invoked',
            trigger_kind=event.trigger.kind,
            event_body=event.body,
            some_env=os.environ.get('MY_ENV_VALUE'))
        # check if the event came from cron
        if event.trigger.kind == 'cron':
            # log something
            context.logger.info('Invoked from cron')
        else:
            # return a response
            return 'A string response'
    ```

- 查看当前用户下功能函数部署情况

  - ```sh
    nuctl get function --namespace user
    
    No functions found
    ```

#### 通过nuctl

```sh
nuctl deploy my-function-1 \
--namespace user \
--path /root/k8s_nuclio/my_function/my_function.py \
--runtime python \
--handler my_function:my_entry_point \
--http-trigger-service-type NodePort \
--registry registry.cn-hangzhou.aliyuncs.com/liangjunbo \
--run-registry registry.cn-hangzhou.aliyuncs.com/liangjunbo
```

> 访问：nuctl invoke my-function-1 --namespace user --via external-ip

#### 通过function.yaml

- 配置文件名称必须为function.yaml

- 文件路径：/root/k8s_nuclio/my_function/function.yaml

  - ```yaml
    apiVersion: "nuclio.io/v1"
    kind: NuclioFunction
    metadata:
      name: my-function-yaml
      namespace: user
    spec:
      env:
      - name: MY_ENV_VALUE
        value: my value
      handler: my_function:my_entry_point
      runtime: python:3.7
      triggers:
        http:
          kind: http
          attributes:
            serviceType: NodePort
        periodic:
          attributes:
            interval: 3s
          class: ""
          kind: cron
    ```

- 执行，--path 执行配置文件和脚本文件所在目录

  - ```
    ├── my_function
    │   ├── function.yaml
    │   ├── my_function.py
    ```

    

  - ```sh
    nuctl deploy \
    --path /root/k8s_nuclio/my_function \
    --namespace user \
    --registry registry.cn-hangzhou.aliyuncs.com/liangjunbo \
    --run-registry registry.cn-hangzhou.aliyuncs.com/liangjunbo
    ```

#### 通过函数内置配置信息

- 脚本路径：/root/k8s_nuclio/my_function/my_function_with_config.py

  - ```python
    import os
    # @nuclio.configure
    # function.yaml:
    #   apiVersion: "nuclio.io/v1"
    #   kind: NuclioFunction
    #   metadata:
    #     name: my-function-config
    #     namespace: user
    #   spec:
    #     env:
    #     - name: MY_ENV_VALUE
    #       value: my value
    #     handler: my_function_with_config:my_entry_point
    #     runtime: python
    #     triggers:
    #       http:
    #         kind: http
    #         attributes:
    #           serviceType: NodePort
    #       periodic:
    #         attributes:
    #           interval: 3s
    #         class: ""
    #         kind: cron
    def my_entry_point(context, event):
        # use the logger, outputting the event body
        context.logger.info_with('Got invoked',
            trigger_kind=event.trigger.kind,
            event_body=event.body,
            some_env=os.environ.get('MY_ENV_VALUE'))
        # check if the event came from cron
        if event.trigger.kind == 'cron':
            # log something
            context.logger.info('Invoked from cron')
        else:
            # return a response
            return 'A string response'
    ```

- 执行

  - ```shell
    nuctl deploy \
    --path /root/k8s_nuclio/my_function/my_function_with_config.py \
    --namespace user \
    --registry registry.cn-hangzhou.aliyuncs.com/liangjunbo \
    --run-registry registry.cn-hangzhou.aliyuncs.com/liangjunbo
    ```

#### 暴露接口方法

1 命令行方式添加**--http-trigger-service-type=nodePort**

2 配置文件添加

```
triggers:
  http:
    kind: http
    attributes:
      serviceType: NodePort
```



### Image部署

1. 使用nuctl build 生成镜像
2. 使用nuctl deploy 部署服务

3. 示例

- 函数

```python
def handler(context, event):
    context.logger.info('This is an unstructured log')

    return context.Response(body='Hello, from nuclio :]',
                            headers={},
                            content_type='text/plain',
                            status_code=200)
```

- 生成镜像			

- 生成镜像

```shell
nuctl build nlp-4 \
--path /root/k8s_nuclio/my_function/helloworld.py \
--registry registry.cn-hangzhou.aliyuncs.com/liangjunbo 

结果：registry.cn-hangzhou.aliyuncs.com/liangjunbo/processor-nlp-4:latest
```

- 启动服务

```sh
nuctl deploy hello-world --run-image registry.cn-hangzhou.aliyuncs.com/liangjunbo/processor-nlp-4:latest \
    --runtime python \
    --handler handler:handler \
    --namespace user
```



## jupyter部署

[官方参考](https://github.com/nuclio/nuclio-jupyter) 目前版本为0.9.2

> nuclio: ignore   表示该cell在生成服务时不会当做源码加载

### 代码逻辑

当前jupyter脚本文件 nlp-1.ipynb

```python
# nuclio: ignore
!pip install --upgrade nuclio-jupyter -i https://pypi.tuna.tsinghua.edu.cn/simple
```

```python
# nuclio: ignore
import nuclio
```

```python
# 安装包，配置信息，如果使用%nuclio将当前的ipynb部署 或者 使用python -m nuclio将当前的ipynb部署，会把下面的配置存储在配置文件里
%nuclio cmd pip install textblob -i https://pypi.tuna.tsinghua.edu.cn/simple 
%nuclio env TO_LANG=fr
%nuclio config spec.build.baseImage = "python:3.7-buster"
```

```python
# 执行函数
from textblob import TextBlob
import os
def handler(context, event):
    context.logger.info('This is an NLP example! ')
    # process and correct the text
    blob = TextBlob(event.body.decode('utf-8'))
    corrected = blob.correct()

    # debug print the text before and after correction
    context.logger.info_with("Corrected text", corrected=str(corrected), orig=str(blob))
    # calculate sentiments
    context.logger.info_with("Sentiment",
                             polarity=str(corrected.sentiment.polarity),
                             subjectivity=str(corrected.sentiment.subjectivity))
    # read target language from environment and return translated text
    lang = os.getenv('TO_LANG','fr')
    return str(corrected)
```

```python
# nuclio: ignore
event = nuclio.Event(body=b'good morninng')
handler(context, event)
```

> ```
> Python> 2022-07-19 07:08:43,677 [info] This is an NLP example! 
> Python> 2022-07-19 07:08:43,679 [info] Corrected text: {'corrected': 'good morning', 'orig': 'good morninng'}
> Python> 2022-07-19 07:08:43,680 [info] Sentiment: {'polarity': '0.7', 'subjectivity': '0.6000000000000001'}
> ```

```
'good morning'
```

```python
# nuclio: ignore
%nuclio deploy nlp-1.ipynb -n nlp-1 -p ai -d http://10.127.91.94:31071
```

> ```
> [nuclio] 2022-07-19 07:16:39,729 (info) Build complete
> [nuclio] 2022-07-19 07:18:36,156 (info) Function deploy complete
> [nuclio] 2022-07-19 07:18:36,157 done creating nlp-1
> [nuclio] 2022-07-19 07:18:36,158 function internal invocation url: nuclio-nlp-1.user.svc.cluster.local:8080
> [nuclio] 2022-07-19 07:18:36,159 note: your function is not exposed externally
> %nuclio: function deployed
> ```

```python
# nuclio: ignore
!curl -X POST -d "how are you" nuclio-nlp-1.user.svc.cluster.local:8080
```

```
how are you
```

### 配置文件生成

在终端界面生成yaml配置文件

```sh
jupyter nbconvert nlp-1.ipynb --to nuclio
```

```yaml
# Generated by nuclio.export.NuclioExporter
apiVersion: nuclio.io/v1
kind: Function
metadata:
  annotations: {}
  labels: {}
  name: nlp-1
spec:
  build:
    baseImage: python:3.7-buster
    commands:
    - pip install textblob -i https://pypi.tuna.tsinghua.edu.cn/simple
    functionSourceCode: IyBHZW5lcmF0ZWQgYnkgbnVjbGlvLmV4cG9ydC5OdWNsaW9FeHBvcnRlcgoKZnJvbSB0ZXh0YmxvYiBpbXBvcnQgVGV4dEJsb2IKaW1wb3J0IG9zCgpkZWYgaGFuZGxlcihjb250ZXh0LCBldmVudCk6CiAgICBjb250ZXh0LmxvZ2dlci5pbmZvKCdUaGlzIGlzIGFuIE5MUCBleGFtcGxlISAnKQoKICAgICMgcHJvY2VzcyBhbmQgY29ycmVjdCB0aGUgdGV4dAogICAgYmxvYiA9IFRleHRCbG9iKGV2ZW50LmJvZHkuZGVjb2RlKCd1dGYtOCcpKQogICAgY29ycmVjdGVkID0gYmxvYi5jb3JyZWN0KCkKCiAgICAjIGRlYnVnIHByaW50IHRoZSB0ZXh0IGJlZm9yZSBhbmQgYWZ0ZXIgY29ycmVjdGlvbgogICAgY29udGV4dC5sb2dnZXIuaW5mb193aXRoKCJDb3JyZWN0ZWQgdGV4dCIsIGNvcnJlY3RlZD1zdHIoY29ycmVjdGVkKSwgb3JpZz1zdHIoYmxvYikpCgogICAgIyBjYWxjdWxhdGUgc2VudGltZW50cwogICAgY29udGV4dC5sb2dnZXIuaW5mb193aXRoKCJTZW50aW1lbnQiLAogICAgICAgICAgICAgICAgICAgICAgICAgICAgIHBvbGFyaXR5PXN0cihjb3JyZWN0ZWQuc2VudGltZW50LnBvbGFyaXR5KSwKICAgICAgICAgICAgICAgICAgICAgICAgICAgICBzdWJqZWN0aXZpdHk9c3RyKGNvcnJlY3RlZC5zZW50aW1lbnQuc3ViamVjdGl2aXR5KSkKCiAgICAjIHJlYWQgdGFyZ2V0IGxhbmd1YWdlIGZyb20gZW52aXJvbm1lbnQgYW5kIHJldHVybiB0cmFuc2xhdGVkIHRleHQKICAgIGxhbmcgPSBvcy5nZXRlbnYoJ1RPX0xBTkcnLCdmcicpCiAgICByZXR1cm4gc3RyKGNvcnJlY3RlZCkKCg==
    noBaseImagesPull: true
  env:
  - name: TO_LANG
    value: fr
  handler: handler:handler
  runtime: python
  volumes: []
```



## python脚本部署

### nuclio命令行参数

```
(tf2.3torch1.7) root@zhangyj-jupyter-1-0:~# nuclio deploy -h
usage: nuclio deploy [-h] [--dashboard-url DASHBOARD_URL] [--name NAME] [--project PROJECT] [--archive] [--output_dir OUTPUT_DIR] [--tag TAG] [--verbose] [--create-project] [--env ENV] [--env-json ENV_JSON]
                     [--spec-json SPEC_JSON] [--cmd-json CMD_JSON] [--mount MOUNT] [--kind KIND]
                     [file]
positional arguments:
  file                  notebook/code file
optional arguments:
  -h, --help            show this help message and exit
  --dashboard-url DASHBOARD_URL, -d DASHBOARD_URL
                        dashboard URL
  --name NAME, -n NAME  function name (notebook name by default)
  --project PROJECT, -p PROJECT
                        project name
  --archive, -a         remote archive for storing versioned functions
  --output_dir OUTPUT_DIR, -o OUTPUT_DIR
                        output dir for files/archives
  --tag TAG, -t TAG     version tag
  --verbose, -v         emit more logs
  --create-project, -c  create new project if doesnt exist
  --env ENV, -e ENV     override environment variable (key=value)
  --env-json ENV_JSON   override environment variable {key: value, ..}
  --spec-json SPEC_JSON
                        override function spec {spec.xy.z: value, ..}
  --cmd-json CMD_JSON   add build commands from list ["pip install x"]
  --mount MOUNT         volume mount, [vol-type:]<vol-url>:<dst>
  --kind KIND
```

### nuclio源码

```
├── archive.py
├── auth.py
├── build.py  
├── config.py 
├── deploy.py
├── export.py
├── __init__.py
├── magic.py
├── __main__.py
├── request.py
├── triggers.py
└── utils.py
```



**为了调试时 方便查看nuclio执行过程 需要下载源码放在自定义路径**

### deploy_code

直接在脚本中编写code 源码

```python
import sys
sys.path.append('/home/jovyan')
import requests
from nuclio.deploy import ConfigSpec,deploy_code,deploy_file
code='''
import glob
def handler(context, event):
    context.logger.info('{}')
    return str('ok')
'''  
# cmd_list = ['pip install --upgrade nuclio-jupyter -i https://pypi.tuna.tsinghua.edu.cn/simple']
env_dict = {'MYENV_VAR': 'something'}
config_dict = {"spec.build.baseImage":"python:3.7-buster",
              "spec.serviceType":"NodePort"}  
code = code.format('Hello World!')
spec = ConfigSpec(env=env_dict,config=config_dict).set_config('triggers.default-http.attributes.serviceType', 'NodePort')
addr = deploy_code(code,dashboard_url='http://10.127.91.94:31071',name='nlp-2',project='ai',verbose=True,spec=spec)
print('addr',addr)
```

### deploy_file

导入自定义脚本文件helloworld.py

```python 
import sys
sys.path.append('/home/jovyan')
import requests
from nuclio.deploy import ConfigSpec,deploy_code,deploy_file
local_path= '/home/jovyan/Algorithm_Frame/Deployment/nuclio_serve/helloworld.py'  # 最好写绝对路径
env_dict = {'MYENV_VAR': 'something'}
config_dict = {"spec.build.baseImage":"python:3.7-buster",
              "spec.serviceType":"NodePort"}
spec = ConfigSpec(env=env_dict,config=config_dict)
addr = deploy_file(local_path,dashboard_url='http://10.127.91.94:31071',name='nlp-2',project='ai',verbose=True,spec=spec,archive=False)
print('addr',addr)
```



上述deploy_code 和 deploy_file直接运行即可，会在dashboard_url中生成nuclio 服务

![](D:\2022新资料\平台\Nuclio使用文档\image\1.jpg)

也可以在终端输入命令行启动

```sh
python -m nuclio deploy helloworld.py -n nlp-1 -p ai -d http://10.127.91.94:31071  # 命令行启动
```



## dashboard部署

在UI 界面上部署，暂不考虑在Code中编写功能函数，只介绍直接上传配置文件或是上传镜像

### source_code

上传配置文件方式

假设在上述jupyter编写时 使用jupyter nbconvert nlp-1.ipynb --to nuclio 生成了配置文件，货值可以自己将功能函数源码转为base64 ，更新配置文件中spec.build.functionSourceCode

如下配置文件nlp-1.yaml

```yaml
# Generated by nuclio.export.NuclioExporter
apiVersion: nuclio.io/v1
kind: Function
metadata:
  annotations: {}
  labels: {}
  name: nlp-1
spec:
  build:
    baseImage: python:3.7-buster
    commands:
    - pip install textblob -i https://pypi.tuna.tsinghua.edu.cn/simple
    functionSourceCode: IyBHZW5lcmF0ZWQgYnkgbnVjbGlvLmV4cG9ydC5OdWNsaW9FeHBvcnRlcgoKZnJvbSB0ZXh0YmxvYiBpbXBvcnQgVGV4dEJsb2IKaW1wb3J0IG9zCgpkZWYgaGFuZGxlcihjb250ZXh0LCBldmVudCk6CiAgICBjb250ZXh0LmxvZ2dlci5pbmZvKCdUaGlzIGlzIGFuIE5MUCBleGFtcGxlISAnKQoKICAgICMgcHJvY2VzcyBhbmQgY29ycmVjdCB0aGUgdGV4dAogICAgYmxvYiA9IFRleHRCbG9iKGV2ZW50LmJvZHkuZGVjb2RlKCd1dGYtOCcpKQogICAgY29ycmVjdGVkID0gYmxvYi5jb3JyZWN0KCkKCiAgICAjIGRlYnVnIHByaW50IHRoZSB0ZXh0IGJlZm9yZSBhbmQgYWZ0ZXIgY29ycmVjdGlvbgogICAgY29udGV4dC5sb2dnZXIuaW5mb193aXRoKCJDb3JyZWN0ZWQgdGV4dCIsIGNvcnJlY3RlZD1zdHIoY29ycmVjdGVkKSwgb3JpZz1zdHIoYmxvYikpCgogICAgIyBjYWxjdWxhdGUgc2VudGltZW50cwogICAgY29udGV4dC5sb2dnZXIuaW5mb193aXRoKCJTZW50aW1lbnQiLAogICAgICAgICAgICAgICAgICAgICAgICAgICAgIHBvbGFyaXR5PXN0cihjb3JyZWN0ZWQuc2VudGltZW50LnBvbGFyaXR5KSwKICAgICAgICAgICAgICAgICAgICAgICAgICAgICBzdWJqZWN0aXZpdHk9c3RyKGNvcnJlY3RlZC5zZW50aW1lbnQuc3ViamVjdGl2aXR5KSkKCiAgICAjIHJlYWQgdGFyZ2V0IGxhbmd1YWdlIGZyb20gZW52aXJvbm1lbnQgYW5kIHJldHVybiB0cmFuc2xhdGVkIHRleHQKICAgIGxhbmcgPSBvcy5nZXRlbnYoJ1RPX0xBTkcnLCdmcicpCiAgICByZXR1cm4gc3RyKGNvcnJlY3RlZCkKCg==
    noBaseImagesPull: true
  triggers:
    http:
      kind: http
      attributes:
        serviceType: NodePort
  env:
  - name: TO_LANG
    value: fr
  handler: handler:handler
  runtime: python
  volumes: []
```

### archive.zip 

上传配置文件方式

自定义配置文件，如果是多个脚本文件，则需要压缩成一个zip包 上传至minio中（如果是一个脚本也可以这么操作）

```yaml
apiVersion: "nuclio.io/v1"
kind: NuclioFunction
metadata:
  name: nlp-4
  namespace: user
spec:
  env:
  - name: MY_ENV_VALUE
    value: my value
  handler: my_function:my_entry_point
  runtime: python:3.7
  build:
    path: "http://minio.storage.svc.cluster.local/zhangyj-n/project_1/nuclio/function.zip?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=4BZBNZAICDDU0N6I9XR3%2F20220721%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20220721T012723Z&X-Amz-Expires=604800&X-Amz-Security-Token=eyJhbGciOiJIUzUxMiIsInR5cCI6IkpXVCJ9.eyJhY2Nlc3NLZXkiOiI0QlpCTlpBSUNERFUwTjZJOVhSMyIsImV4cCI6MTY1ODM3MDQ0MCwicGFyZW50Ijoiemhhbmd5ai1uQGdsb2Rvbi5jb20ifQ.1vcIADSXx_Cqmki2MjwMXvFbarN1uoCUmYP6sJpp2pXRg_68OZVqbynbppqHvb-P_JLFZ7749ExXXDL7OhPOOw&X-Amz-SignedHeaders=host&versionId=null&X-Amz-Signature=4d1d73d9918489a8f88f817413cb0666f80641fb39b6143c413d1042f0361b23"
    registry: registry.cn-hangzhou.aliyuncs.com/liangjunbo
    noBaseImagePull: True
    codeEntryType: archive
  triggers:
    http:
      kind: http
      attributes:
        serviceType: NodePort
    periodic:
      attributes:
        interval: 3s
      class: ""
      kind: cron
```

- 上传部署流程
- ![](D:\2022新资料\平台\Nuclio使用文档\image\2.jpg)

- ![](D:\2022新资料\平台\Nuclio使用文档\image\3.jpg)

- ![](D:\2022新资料\平台\Nuclio使用文档\image\4.jpg)

- ![](D:\2022新资料\平台\Nuclio使用文档\image\5.jpg)

- ![](D:\2022新资料\平台\Nuclio使用文档\image\6.jpg)

### image部署

上传镜像

![](D:\2022新资料\平台\Nuclio使用文档\image\7.jpg)



## kubeflow部署

参考旧版本部署方式 [旧版本](https://v0-7.kubeflow.org/docs/components/misc/nuclio/)

旧版本nuclio路径 

- - [deploy](https://github.com/kubeflow/pipelines/blob/0.1.8/components/nuclio/deploy/component.yaml) 
  - [delete](https://github.com/kubeflow/pipelines/blob/0.1.8/components/nuclio/delete/component.yaml)
  - [invoker](https://github.com/kubeflow/pipelines/blob/0.1.8/components/nuclio/invoker/component.yaml)

### 镜像环境准备

1. 下载0.9.2版本源码 ，取nuclio文件夹

![](D:\2022新资料\平台\Nuclio使用文档\image\8.jpg)



2. 将源码拷贝进 **nuclio/pydeploy:latest**镜像容器的工作目录，替换掉0.7.3版本
   - 启动镜像,容器工作目录为/nuclio
   - 将第一步中的nuclio文件夹拷贝到容器工作目录下，替换原工作目录下的0.7.3版本的nuclio文件夹

3. 生成组件使用的镜像

### pipeline

1. 配置文件编写

   - 命令行参数可参考nuclio deploy -h

   - deploy_com.yaml

   - ```yaml
     name: nuclio deploy
     description: auto build and deploy nuclio function.
     inputs:
       - {name: Url, type: String, description: 'url/path to source code, zip archive, git path, or notebook'}
       - {name: Name, type: String, description: 'function name'}
       - {name: Project, type: String, description: 'project name', default: 'default'}
       - {name: Dashboard, type: String, description: 'nuclio dashboard service url', default: 'http://10.127.91.94:31071'}
     
     outputs:
       - {name: Endpoint, type: String, description: 'function endpoint url'}
     implementation:
       container:
         image: registry.cn-beijing.aliyuncs.com/huadong_beijing/pydeploy:v4
         command: [
           python, -m, nuclio, deploy, {inputValue: Url},
           --dashboard-url, {inputValue: Dashboard},
           --name, {inputValue: Name},
           --project, {inputValue: Project},
         ]
         fileOutputs:
           Endpoint: /tmp/output
     ```

2. pipeline逻辑代码

   - ```python
     import kfp
     from kfp import dsl
     
     # load nuclio kubeflow components
     nuclio_deploy = kfp.components.load_component_from_file('deploy_com.yaml')
     def nuc_pipeline():
         dashboard='http://10.127.91.94:31071'
         local_path = '/nuclio/helloworld.py'
         build = nuclio_deploy(url=local_path, name='my_function', project='ai', dashboard=dashboard)
     
     if __name__ == '__main__':
         kfp.compiler.Compiler().compile(
             pipeline_func=nuc_pipeline,
             package_path='nuc_pipeline.yaml')
         kfp.Client().create_run_from_pipeline_func(nuc_pipeline,arguments={})
     ```

## 参考文件

[参考文档](https://github.com/nuclio/nuclio/tree/development/docs/reference)

```
├── function-configuration
│   ├── code-entry-types.md  # 功能代码输入类型和相关的配置字段
│   └── function-configuration-reference.md  # 功能函数配置文件中的参数说明
├── nuctl
│   └── nuctl.md  # 命令行
├── runtimes
│   ├── dotnetcore
│   │   ├── dotnetcore-reference.md
│   │   └── writing-a-dotnetcore-function.md
│   ├── golang
│   │   └── golang-reference.md
│   ├── java
│   │   └── java-reference.md
│   ├── nodejs
│   │   └── nodejs-reference.md
│   ├── python
│   │   └── python-reference.md  # Python 构建和部署配置
│   ├── runtime-feature-matrix.md
│   └── shell
│       └── writing-a-shell-function.md
└── triggers
    ├── cron.md
    ├── eventhub.md
    ├── http.md
    ├── kafka.md
    ├── kinesis.md
    ├── mqtt.md
    ├── nats.md
    ├── rabbitmq.md
    └── v3iostream.md
```