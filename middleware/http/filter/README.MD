# dapr-middleware-http-filter
A customized filter middleware of DAPR.

主要是给应用系统层面使用的一个中间件，所以适应场景是很专注的，不考虑过多横向扩展性。
这里使用的场景是这样的，构建一套适应多租户使用的类SAAS系统，需要把业务范围的微服务群和多租户鉴权/权限管理控制/基于租户功能灰度上线等范围的微服务群这2部分工作完全分离，分别由2个开发小组负责开发及后续运维。为此通过DAPR及这个http filter middleware,达到期望目标。

filter代码编译，按DAPR的开发要求设置环境， git代码要从这2个地方获取。
>https://github.com/gattyzhang/dapr.git<br>
>https://github.com/gattyzhang/components-contrib.git


filter集成测试步骤<br>
>a. 运行 webserver.go。 端口8001，只能通过IP访问，不能使用localhost,127.0.0.1访问。<br>
>b. filter yaml的filter_url 指向上一步的webserver的地址。<br>
>c. postman 访问 filter部署到的dapr环境。<br>


我的开发机器OS： 深度操作系统(Deepin GNU/Linux 20.3), Docker v19.03.8, Minikube v1.18.1, go v1.17.2<br>
>为了可以同时使用host方式的dapr和k8s的daps， minikube运行在gatty用户下，host的dapr是运行在dapr用户下的。<br>
>设置本地开发环境参考了这里<br>
>>https://github.com/gattyzhang/dapr/tree/master/docs/development<br>
>>https://docs.dapr.io


创建Docker Image的备注<br>
命令： make docker-build
1. env<br>   
>export DAPR_REGISTRY=gatty.pcauto<br>
>export DAPR_TAG=dev<br><br>
>>a.dapr用户是使用host方式运行dapr，开发和调试方便.<br>
>>b.需要部署到minikube k8s时， 需要通过dock image save和load命令，导入最新的daprd image到gatty用户的docker上.<br>
>>c.在k8s上部署自己定制dapr的middleware时，只需修改业务微服务上的dapr sidecar-image的注入就可以了，dapr-system namespace下面的DAPR deploy是不需要任何修改的。<br>
>>d.业务微服务注入包含filter的daprd的写法：<br>
>>>annotations:<br>
>>>>dapr.io/sidecar-image: gatty.pcauto:dev...<br>
>>>>dapr.io/sidecar-listen-addresses: 0.0.0.0


2. docker 设置
sudo vi /etc/systemd/system/docker.service.d/http-proxy.conf<br>

[Service]
>Environment="HTTP_PROXY=192.x.x.x:xxxx" "HTTPS_PROXY=192.x.x.x:xxxx" "NO_PROXY=localhost,127.0.0.1,192.168.49.2"<br>

>另，启动minikube 时，如果要访问墙外url，加上proxy env, 再启动minikube。<br>
>>export HTTP_PROXY=http://192.x.x.x:xxxx<br>
>>export HTTPS_PROXY=http://192.x.x.x:xxxx<br>
>>export NO_PROXY=localhost,127.0.0.1,192.168.49.2(缺省已经有了，可以不运行)<br>


其他
>HTTP的自定义header item， key不能包含"_"字符, 应该使用"-"， 这个才有更好兼容性，否则会给nginx等过滤掉的。 例如"Test-Header-Key"。
