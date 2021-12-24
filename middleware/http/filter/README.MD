# dapr-middleware-http-filter
a customized filter middleware of DAPR.

主要是给应用系统层面使用的一个中间件，所以适应场景是很专注的，不考虑过多横向扩展性。
这里使用的场景是这样的，构建一套适应多租户使用的类SAAS系统，需要把业务范围的微服务群和多租户鉴权/权限管理控制/基于租户功能灰度上线等范围的微服务群这2部分工作完全分离，分别由2个开发小组负责开发及后续运维。为此通过DAPR及这个http filter middleware,达到期望目标。

filter 集成测试步骤
1. 运行 webserver.go。 端口8001，只能通过IP访问，不能使用localhost,127.0.0.1访问。
2. filter yaml的filter_url 指向上一步的webserver的地址。
3. postman 访问 filter部署到的dapr环境。
4. 自定义的filter仅仅包含在daprd image，所以只需要修改业务微服务的注入(deployment.yaml -> annotations:）<br>
                dapr.io/sidecar-image: gatty.pcauto/daprd:x.x.x<br>
                dapr.io/sidecar-listen-addresses: 0.0.0.0


开发机器OS： 深度操作系统(Deepin GNU/Linux 20.3), Docker v19.03.8, Minikube v1.18.1, go v1.17.2
设置本地开发环境参考了这里<br>
    https://github.com/gattyzhang/dapr/tree/master/docs/development<br>
    https://docs.dapr.io

Docker Image的创建备注：
命令： make docker-build
1. env   
export DAPR_REGISTRY=gatty.pcauto<br>
export DAPR_TAG=dev<br>
(a.本地设置了gatty.pcauto， 通过本地镜像，直接在k8s运行, 在dapr用户build， 导到gatty用户minik8s 运行. ）

b.dapr用户是使用host方式运行dapr，开发和调试的.
c.在k8s上部署自己定制dapr的middleware时，只需修改业务微服务上的dapr sidecar-image的注入就可以了，dapr-system namespace下面的DAPR 原始deploy是不需要任何修改的。
d.业务微服务注入包含了filter的daprd的写法：
annotations:
    dapr.io/sidecar-image: gatty.pcauto:dev...
    dapr.io/sidecar-listen-addresses: 0.0.0.0


2. docker 设置
sudo vi /etc/systemd/system/docker.service.d/http-proxy.conf

[Service]
Environment="HTTP_PROXY=192.x.x.x:xxxx" "HTTPS_PROXY=192.x.x.x:xxxx" "NO_PROXY=localhost,127.0.0.1,192.168.49.2"

启动minikube 时，如果要访问墙外url，加上proxy env, 再启动minikube。<br>
export HTTP_PROXY=http://192.x.x.x:xxxx<br>
export HTTPS_PROXY=http://192.x.x.x:xxxx<br>
export NO_PROXY=localhost,127.0.0.1,192.168.49.2(缺省已经有了，可以不运行)


3. 注意：
HTTP的自定义header item， key不能包含"_"字符, 应该使用"-"， 这个才有更好兼容性，否则会给nginx等过滤掉的。 例如"Test-Header-Key"。
