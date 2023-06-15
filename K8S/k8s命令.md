查看集群和版本相关信息

```shell
kubectl version				//显示客户端和服务器侧版本信息
kubectl api-versions		//列出当前版本的kubernetes的服务器端所支持的api版本信息
kubectl cluster-info		//获取k8s集群信息
kubectl config view         //获取k8s集群管理配置信息，也就是 .kube/config 文件内容
```

获取帮助详细信息

```shell
kubectl explain po			//查看帮助信息和help类似，尤其是资源清单的结构字段信息
//查看帮助信息，资源下的cpu和memory等，每个配置项都有详细的网页手册地址
kubectl explain Deployment.spec.template.spec.containers.resources
```

常用获取资源方式

```shell
kubectl get pods			//查看pod信息
kubectl get pods -wide		//已监控方式查看pod信息，有新的创建和销毁会立刻打印出来
kubectl get pods -o wide	//查看pod详细信息
kubectl get nodes -o wide	//查看node详细信息
kubectl get namespaces		//列出所有的namespace
kubectl get rc,service      //查看rc和service列表
kubectl get deployment		//获取指定控制器pod信息
kubectl get pods -n kube-system  //查看kube-system命名空间中pod信息
kubectl get pods/podName -o yaml  //查看pod的yaml信息
```

描述资源

```shell
kubectl describe node nodeName 			//获取详细资源清单信息（包括CPU和Memory）
kubectl describe po podName 			//获取详细资源清单信息（包括错误信息和实时状态）
```

查看日志

```shell
kubectl logs podName -f					//或者指定pod的日志信息
```

进入容器

```shell
kubectl exec -it podName -- sh				//进入pod容器，但是对权限要求也较多
kubectl exec -it podName -c containerName -- bash   //通过bash获得Pod中某个容器的TTY，相当于登录容器
```

创建资源(推荐用apply)

```shell
kubectl create/apply -f yamls/sonar.yaml 			//根据yaml文件创建容器
kubectl create/apply -f yamls/					//多个yaml文件创建容器
kubectl create/apply -f my-service.yaml -f my-rc.yaml //根据yaml配置文件一次性创建service和rc
```

删除资源

```shell
kubectl delete -f yamls/sonar.yaml 			//删除指定pod 
kubectl delete -f yamls/					//删除多个pod 
kubectl delete pods podName					//删除指定pod 
kubectl delete pod podName --force --grace-period=0    //强制删除pod
kubectl delete deployment ControllerName	//有控制器的pod不能直接删除，需先删除其控制器
kubectl delete pods,services -l name=labelName  //删除所有包含某个label的Pod和service
kubectl delete pods --all                   //删除所有Pod
```

标签匹配

```shell
kubectl get pods --show-labels
kubectl get pods --show-labels -l env=dev,tie=front    //多个标签同时满足条件
kubectl get pods --show-labels -l 'env in (dev,test)'    [in,notin]
kubectl label pods podName env=test      //设置标签 env=test
kubectl label pods podName env=test --overwrite     //若env标签存在，强制设置标签 env=test
kubectl lable pods podName env-       //删除podname中env标签
```

注解

```shell
kubectl annotate pods nginx1 my-annotate='my annotate,ok' //对pod增加一个注解
kubectl annotate pods nginx1 my-annotate='my annotate,no' --overwrite //修改已存在的注解
```

暴露服务，也就是创建service

```shell
kubectl expose pod podName [--port=80 --target-port=8000]
kubectl expose deployment deployName [--port=80 --target-port=8000]
```

[自动]扩缩容

```shell
kubectl scale deployment deployName --replicas=3               //执行扩缩容Pod的操作
kubectl autoscale deployment deployName --min=2 --max=10       //设置pod数量在2到10之间
kubectl autoscale deployment deployName --max=5 --cpu-percent=80           //pod数量在1到5之间，目标CPU利用率为80%
```

在线设置镜像版本

```shell

#官网滚动更新图
https://kubernetes.io/images/docs/kubectl_rollingupdate.svg
```

升级和回滚操作

```shell
kubectl apply -f deploy.yaml --record
kubectl set image deployment/nginx nginx=nginx:1.9.1 --record   //设置nginx镜像为1.9.1版本

kubectl rollout status deploy deployName //查看上线状态
kubectl rollout history deployment deployName                //显示deployment的更新记录
kubectl rollout history deployment deployName --revision=3   //显示版本3 deployment的详情
kubectl rollout undo deployment/deployName                    //回滚到上一个版本
kubectl rollout undo deployment/deployName --to-revision=3    //回滚到第3个版本
kubectl rollout undo --dry-run=true deployment/deployName     //回滚到上一个版本，调试但不执行
```

管理多集群

```shell
kubectl cluster-info           //获取k8s集群信息
kubectl config view            //获取k8s集群管理配置信息，也就是 .kube/config 文件内容
kubectl config get-contexts    //查看集群名称的context
kubectl config set-context 上下文名称 --user=minikube --cluster=minikube --namespace=demo  //设置上下文
kubectl config set current-context minikube   //切换到名称为 minikube 的集群中
kubectl config use-context minikube           //切换到名称为 minikube 的集群中
```

污点和容忍

```shell
kubectl taint nodes test1 node-role.kubernetes.io/master=true:NoSchedule   //给master节点打上一个污点
kubectl taint nodes test1 node-role.kubernetes.io/master-        //删除指定key的所有effect
```

设置kubectl shell命令自动补全

```shell
kubectl completion -h
sudo yum -y install bash-completion
source /usr/share/bash-completion/bash_completion
type _init_completion
echo 'source <(kubectl completion bash)' >> ~/.bashrc
source ~/.bashrc
```

排错

```shell
kubectl get po podName //查看pod状态
kubectl describe no NodeName //显示指定节点的详细信息
kubectl describe po poName //显示指定pod的详细信息
kubectl get event //显示整个集群的事件日志
kubectl top pod podName //查看pod的内存和cpu使用情况
```

