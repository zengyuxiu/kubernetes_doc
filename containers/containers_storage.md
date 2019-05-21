Images

您创建Docker镜像并将其推送到注册表，然后在Kubernetes窗格中引用它。\
容器的image属性支持与docker命令相同的语法，包括私有注册表和标记。

-   [**Updating
    Images**](https://kubernetes.io/docs/concepts/containers/images/#updating-images)

-   [**Building Multi-architecture Images with
    Manifests**](https://kubernetes.io/docs/concepts/containers/images/#building-multi-architecture-images-with-manifests)

-   [**Using a Private
    Registry**](https://kubernetes.io/docs/concepts/containers/images/#using-a-private-registry)

    Updating Images

默认的拉取策略是IfNotPresent，这会导致Kubelet跳过拉动图像（如果已存在）。
如果您想总是强制拉动，可以执行以下操作之一：\
•将容器的imagePullPolicy设置为Always。\
•省略imagePullPolicy并使用：latest作为要使用的图像的标记。\
•省略imagePullPolicy和要使用的图像的标记。\
•启用AlwaysPullImages准入控制器。\
请注意，您应该避免使用：latest tag，有关详细信息，请参阅配置的最佳实践。

Building Multi-architecture Images with Manifests

Docker CLI现在支持以下命令docker
manifest以及create，annotate和push等子命令。这些命令可用于构建和推送清单。您可以使用docker
manifest inspect来查看清单。\
请在此处查看docker文档：https：//docs.docker.com/edge/engine/reference/commandline/manifest/\
请参阅我们在构建工具中如何使用它的示例：https：//cs.k8s.io/？q =
drager％20manifest％20（create = 7Cpush％7Cannotate）＆i = nope＆files
=＆last =\
这些命令完全依赖于Docker CLI，并且完全在Docker CLI上实现。您需要编辑\$
HOME / .docker /
config.json并将实验密钥设置为启用，或者您可以在调用CLI命令时将DOCKER\_CLI\_EXPERIMENTAL环境变量设置为启用。\
注意：请使用Docker
18.06或更高版本，以下版本有错误或不支持实验命令行选项。示例https://github.com/docker/cli/issues/1135会在containerd下导致问题。\
如果您在上传陈旧的清单时遇到问题，只需清理\$ HOME / .docker /
manifests中的旧清单即可重新开始。\
对于Kubernetes，我们通常使用带后缀的图像 -
\$（ARCH）。为了向后兼容，请生成带有后缀的旧图像。想法是生成具有所有拱的清单的说暂停图像，并且说暂停-amd64，其向后兼容旧配置或YAML文件，其可能硬编码具有后缀的图像。

Using a Private Registry

私人注册管理机构可能需要密钥才能从中读取图像。
凭证可以通过多种方式提供：\
•使用Google Container Registry\
•每簇\
•在Google Compute Engine或Google Kubernetes Engine上自动配置\
•所有pod都可以读取项目的私有注册表\
•使用AWS EC2容器注册表（ECR）\
•使用IAM角色和策略来控制对ECR存储库的访问\
•自动刷新ECR登录凭据\
•使用Azure容器注册表（ACR）\
•使用IBM Cloud Container Registry\
•配置节点以对私有注册表进行身份验证\
•所有pod都可以读取任何已配置的私有注册表\
•需要群集管理员进行节点配置\
•预拉图像\
•所有pod都可以使用节点上缓存的任何图像\
•要求对所有节点进行root访问以进行设置\
•在Pod上指定ImagePullSecrets\
•只有提供自己密钥的pod才能访问私有注册表\
下面更详细地描述每个选项。

Using Google Container Registry

在Google Compute Engine（GCE）上运行时，Kubernetes对Google Container
Registry（GCR）提供原生支持。 如果您在GCE或Google Kubernetes
Engine上运行群集，只需使用完整的图像名称（例如gcr.io/my\_project/image：tag）。\
群集中的所有pod都具有此注册表中图像的读取权限。\
kubelet将使用实例的Google服务帐户向GCR进行身份验证。
该实例上的服务帐户将具有https://www.googleapis.com/auth/devstorage.read\_only，因此它可以从项目的GCR中提取，但不能推送。

Using AWS EC2 Container Registry

-   当节点是AWS EC2实例时，Kubernetes对AWS EC2 Container
    Registry具有本机支持。\
    只需在Pod定义中使用完整的图像名称（例如ACCOUNT.dkr.ecr.REGION.amazonaws.com/imagename：tag）。\
    可以创建pod的群集的所有用户都可以运行使用ECR注册表中任何映像的pod。\
    kubelet将获取并定期刷新ECR凭据。它需要以下权限才能执行此操作：\
    •ECR：GetAuthorizationToken\
    •ECR：BatchCheckLayerAvailability\
    •ECR：GetDownloadUrlForLayer\
    •ECR：GetRepositoryPolicy\
    •ECR：DescribeRepositories\
    •ECR：ListImages\
    •ECR：BatchGetImage\
    要求：\
    •您必须使用kubelet v1.2.0或更高版本。 （例如，运行/ usr / bin /
    kubelet \--version = true）。\
    •如果您的节点位于区域A中且您的注册表位于不同的区域B中，则需要版本v1.3.0或更高版本。\
    •必须在您所在的地区提供ECR\
    故障排除：\
    •验证上述所有要求。\
    •在工作站上获取\$ REGION（例如us-west-2）凭据。
    SSH进入主机并使用这些信用卡手动运行Docker。它有用吗？\
    •使用\--cloud-provider = aws验证kubelet是否正在运行。\
    •检查kubelet日志（例如journalctl -u kubelet）以获取日志行，例如：\
    •plugins.go：56\]注册凭证提供程序：aws-ecr-key\
    •provider.go：91\]刷新提供程序的缓存：\*
    aws\_credentials.ecrProvider

    Using Azure Container Registry (ACR)

使用Azure容器注册表时，您可以使用管理员用户或服务主体进行身份验证。
在任何一种情况下，身份验证都通过标准Docker身份验证完成
这些说明假定使用azure-cli命令行工具。\
您首先需要创建一个注册表并生成凭据，完整的文档可以在Azure容器注册表文档中找到。\
创建容器注册表后，您将使用以下凭据登录：\
•DOCKER\_USER：服务主体或管理员用户名\
•DOCKER\_PASSWORD：服务主体密码或管理员用户密码\
•DOCKER\_REGISTRY\_SERVER：\$ {some-registry-name} .azurecr.io\
•DOCKER\_EMAIL：\$ {some-email-address}\
填好这些变量后，您可以配置Kubernetes Secret并使用它来部署Pod.

Using IBM Cloud Container Registry

IBM Cloud Container
Registry提供了一个多租户私有映像注册表，您可以使用它来安全地存储和共享Docker映像。
默认情况下，集成的漏洞顾问会扫描私有注册表中的映像，以检测安全问题和潜在漏洞。
IBM
Cloud帐户中的用户可以访问您的映像，也可以创建令牌以授予对注册表名称空间的访问权限。\
要安装IBM Cloud Container Registry
CLI插件并为映像创建命名空间，请参阅IBM Cloud Container Registry入门。\
您可以使用IBM Cloud Container Registry将容器从IBM
Cloud公共映像和私有映像部署到IBM Cloud Kubernetes
Service集群的缺省名称空间中。
要将容器部署到其他名称空间，或使用来自其他IBM Cloud Container
Registry区域或IBM Cloud帐户的映像，请创建Kubernetes imagePullSecret。
有关更多信息，请参阅从映像构建容器。

Configuring Nodes to Authenticate to a Private Registry

-   注意：如果您在Google Kubernetes
    Engine上运行，则每个节点上都会有一个.dockercfg，其中包含Google
    Container Registry的凭据。 你不能使用这种方法。\
    注意：如果您在AWS
    EC2上运行并且正在使用EC2容器注册表（ECR），则每个节点上的kubelet将管理和更新ECR登录凭据。
    你不能使用这种方法。\
    注意：如果您可以控制节点配置，则此方法是合适的。
    它不能可靠地在GCE和任何其他进行自动节点替换的云提供商上运行。\
    注意：Kubernetes目前仅支持docker config的auths和HttpHeaders部分。
    这意味着不支持凭证助手（credHelpers或credsStore）。\
    Docker将私有注册表的密钥存储在\$ HOME / .dockercfg或\$ HOME /
    .docker / config.json文件中。
    如果您将相同的文件放在下面的搜索路径列表中，则kubelet会在拉取图像时将其用作凭据提供程序。

-   **{\--root-dir:-/var/lib/kubelet}/config.json**

-   **{cwd of kubelet}/config.json**

-   **\${HOME}/.docker/config.json**

-   **/.docker/config.json**

-   **{\--root-dir:-/var/lib/kubelet}/.dockercfg**

-   **{cwd of kubelet}/.dockercfg**

-   **\${HOME}/.dockercfg**

-   **/.dockercfg**

注意：您可能必须在环境文件中明确设置HOME = / root for kubelet。\
以下是配置节点以使用私有注册表的建议步骤。
在此示例中，在桌面/笔记本电脑上运行这些：\
1.为要使用的每组凭据运行docker login \[server\]。 这会更新\$ HOME /
.docker / config.json。\
2.在编辑器中查看\$ HOME / .docker /
config.json，以确保它只包含您要使用的凭据。\
3.获取节点列表，例如：\
•如果您需要名称：nodes = \$（kubectl get nodes -o jsonpath
=\'{range.items \[\*\] .meadata} {.name} {end}\'）\
•如果你想得到IP：nodes = \$（kubectl get nodes -o jsonpath =\'{range
.items \[\*\] .status.addresses \[？（@。type =="ExternalIP"）\]}
{.address} {结束}\'）\
4.将本地.docker / config.json复制到上面的搜索路径列表之一。\
•例如：\$节点中的n; 做scp\~ / .docker / config.json root @ \$
n：/var/lib/kubelet/config.json;DONE\
通过创建使用私有图像的pod进行验证，例如：

**kubectl apply -f - \<\<EOF**

**apiVersion: v1**

**kind: Pod**

**metadata:**

**name: private-image-test-1**

**spec:**

**containers:**

**- name: uses-private-image**

**image: \$PRIVATE\_IMAGE\_NAME**

**imagePullPolicy: Always**

**command: \[ \"echo\", \"SUCCESS\" \]**

**EOF**

**pod/private-image-test-1 created**

If everything is working, then, after a few moments, you should see:

**kubectl logs private-image-test-1**

**SUCCESS**

If it failed, then you will see:

**kubectl describe pods/private-image-test-1 \| grep \"Failed\"**

**Fri, 26 Jun 2015 15:36:13 -0700 Fri, 26 Jun 2015 15:39:13 -0700 19
{kubelet node-i2hq} spec.containers{uses-private-image} failed Failed to
pull image \"user/privaterepo:v1\": Error: image user/privaterepo:v1 not
found**

您必须确保群集中的所有节点都具有相同的.docker / config.json。
否则，pod将在某些节点上运行，而无法在其他节点上运行。
例如，如果使用节点自动缩放，则每个实例模板都需要包含.docker /
config.json或挂载包含它的驱动器。\
私有注册表项添加到任何私有注册表后，所有pod都将具有对图像的读访问权限
**.docker/config.json**.

Pre-pulling Images

注意：如果您在Google Kubernetes
Engine上运行，则每个节点上都会有一个.dockercfg，其中包含Google Container
Registry的凭据。 你不能使用这种方法。\
注意：如果您可以控制节点配置，则此方法是合适的。
它不能可靠地在GCE和任何其他进行自动节点替换的云提供商上运行。\
默认情况下，kubelet将尝试从指定的注册表中提取每个图像。
但是，如果容器的imagePullPolicy属性设置为IfNotPresent或Never，则使用本地映像（分别优先或排他）。\
如果您希望依赖预先提取的图像作为注册表身份验证的替代，则必须确保群集中的所有节点都具有相同的预拉图像。\
这可以用于预加载某些图像以提高速度，或者作为对私有注册表进行身份验证的替代方法。\
所有pod都可以读取任何预拉图像。

Specifying ImagePullSecrets on a Pod

注意：此方法目前是Google Kubernetes
Engine，GCE以及自动创建节点的任何云提供商的推荐方法。\
Kubernetes支持在pod上指定注册表项。

Creating a Secret with a Docker Config

Run the following command, substituting the appropriate uppercase
values:

**cat \<\<EOF \> ./kustomization.yaml**

**secretGenerator:**

**- name: myregistrykey**

**type: docker-registry**

**literals:**

**- docker-server=DOCKER\_REGISTRY\_SERVER**

**- docker-username=DOCKER\_USER**

**- docker-password=DOCKER\_PASSWORD**

**- docker-email=DOCKER\_EMAIL**

**EOF**

**kubectl apply -k .**

**secret/myregistrykey-66h7d4d986 created**

如果您已经拥有Docker凭证文件，则可以将凭证文件作为Kubernetes密钥导入，而不是使用上述命令。
基于现有Docker凭证创建一个秘密解释了如何设置它。
如果您使用多个私有容器注册表，这将特别有用，因为kubectl创建秘密docker-registry创建一个仅适用于单个私有注册表的Secret。\
注意：Pod只能在自己的命名空间中引用图像拉取秘密，因此每个命名空间需要执行一次此过程。

Referring to an imagePullSecrets on a Pod

Now, you can create pods which reference that secret by adding an
**imagePullSecrets** section to a pod definition.

**cat \<\<EOF \> pod.yaml**

**apiVersion: v1**

**kind: Pod**

**metadata:**

**name: foo**

**namespace: awesomeapps**

**spec:**

**containers:**

**- name: foo**

**image: janedoe/awesomeapp:v1**

**imagePullSecrets:**

**- name: myregistrykey**

**EOF**

**cat \<\<EOF \>\> ./kustomization.yaml**

**resources:**

**- pod.yaml**

**EOF**

需要对使用私有注册表的每个pod执行此操作。\
但是，可以通过在serviceAccount资源中设置imagePullSecrets来自动设置此字段。
检查将ImagePullSecrets添加到服务帐户以获取详细说明。\
您可以将其与每个节点的.docker / config.json结合使用。 凭证将被合并。
这种方法适用于Google Kubernetes Engine。

Use Cases

有许多配置私有注册表的解决方案。以下是一些常见用例和建议的解决方案。\
1.仅运行非专有（例如开源）图像的集群。无需隐藏图像。\
•使用Docker集线器上的公共映像。\
•无需配置。\
•在GCE / Google Kubernetes
Engine上，自动使用本地镜像来提高速度和可用性。\
2.集群运行一些专有映像，这些映像应该隐藏给公司外部的人，但对所有集群用户都是可见的。\
•使用托管的私有Docker注册表。\
•它可能托管在Docker Hub或其他地方。\
•如上所述，在每个节点上手动配置.docker / config.json。\
•或者，使用开放读取访问权限在防火墙后面运行内部私有注册表。\
•不需要Kubernetes配置。\
•或者，在使用GCE / Google Kubernetes
Engine时，请使用项目的Google容器注册表。\
•与集群自动调节相比，它可以比手动节点配置更好地工作。\
•或者，在更改节点配置不方便的群集上，使用imagePullSecrets。\
3.具有专有图像的集群，其中一些需要更严格的访问控制。\
•确保AlwaysPullImages准入控制器处于活动状态。否则，所有Pod都可能访问所有图像。\
•将敏感数据移动到"秘密"资源，而不是将其打包在图像中。\
4.多租户群集，每个租户需要拥有私有注册表。\
•确保AlwaysPullImages准入控制器处于活动状态。否则，所有租户的所有Pod都可能访问所有图像。\
•运行需要授权的私有注册表。\
•为每个租户生成注册表凭据，保密，并为每个租户命名空间填充机密。\
•租户将该秘密添加到每个命名空间的imagePullSecrets。\
如果需要访问多个注册表，则可以为每个注册表创建一个秘密。
Kubelet会将任何imagePullSecrets合并到一个虚拟的.docker / config.json中

Container Environment Variables
===============================

此页面描述Container环境中Container可用的资源。

-   [**Container
    environment**](https://kubernetes.io/docs/concepts/containers/container-environment-variables/#container-environment)

-   [**What\'s
    next**](https://kubernetes.io/docs/concepts/containers/container-environment-variables/#what-s-next)

Container environment
---------------------

Kubernetes Container环境为容器提供了几个重要资源：\
•文件系统，它是图像和一个或多个卷的组合。\
•有关Container本身的信息。\
•有关群集中其他对象的信息。

### Container information

Container的主机名是运行Container的Pod的名称。
它可以通过hostname命令或libc中的gethostname函数调用获得。\
Pod名称和命名空间可通过向下API作为环境变量使用。\
Pod定义中的用户定义环境变量也可用于Container，Docker镜像中静态指定的任何环境变量也是如此。

### Cluster information

创建Container时运行的所有服务的列表可作为环境变量用于该Container。
这些环境变量与Docker链接的语法相匹配。\
对于名为foo的映射到名为bar的Container的服务，定义了以下变量：

**FOO\_SERVICE\_HOST=\<the host the service is running on\>**

**FOO\_SERVICE\_PORT=\<the port the service is running on\>**

Services have dedicated IP addresses and are available to the Container
via DNS, if [DNS
addon](http://releases.k8s.io/master/cluster/addons/dns/) is enabled. 

Runtime Class
=============

该页面描述了RuntimeClass资源和运行时选择机制。\
警告：RuntimeClass包含v1.14中beta升级的重大更改。
如果您在v1.14之前使用RuntimeClass，请参阅将RuntimeClass从Alpha升级到Beta。

-   [**Runtime
    Class**](https://kubernetes.io/docs/concepts/containers/runtime-class/#runtime-class)

Runtime Class
-------------

RuntimeClass是用于选择容器运行时配置的功能。
容器运行时配置用于运行Pod的容器。

### Set Up

确保已启用RuntimeClass功能门（默认情况下）。
有关启用要素门的说明，请参见功能门。
必须在apiservers和kubelet上启用RuntimeClass功能门。\
1.在节点上配置CRI实现（取决于运行时）\
2.创建相应的RuntimeClass资源

#### 1. Configure the CRI implementation on nodes

RuntimeClass提供的配置依赖于Container Runtime Interface（CRI）实现。
有关如何配置的CRI实现，请参阅相应的文档（如下）。\
注意：RuntimeClass当前假定整个集群中的同类节点配置（这意味着所有节点的配置方式与容器运行时相同）。
任何异构性（变化的配置）必须通过调度功能独立于RuntimeClass进行管理（请参阅将Pod分配给节点）。\
配置具有相应的处理程序名称，由RuntimeClass引用。 处理程序必须是有效的DNS
1123标签（字母数字+ - 字符）。

#### 2. Create the corresponding RuntimeClass resources

步骤1中的配置设置应各自具有关联的处理程序名称，用于标识配置。
对于每个处理程序，创建相应的RuntimeClass对象。\
RuntimeClass资源目前只有2个重要字段：RuntimeClass名称（metadata.name）和处理程序（处理程序）。
对象定义如下所示：

**apiVersion: node.k8s.io/v1beta1 *\# RuntimeClass is defined in the
node.k8s.io API group***

**kind: RuntimeClass**

**metadata:**

**name: myclass *\# The name the RuntimeClass will be referenced by***

***\# RuntimeClass is a non-namespaced resource***

**handler: myconfiguration *\# The name of the corresponding CRI
configuration***

注意：建议将RuntimeClass写入操作（create / update / patch /
delete）限制为群集管理员。 这通常是默认值。
有关详细信息，请参阅授权概述。

### Usage

为集群配置RuntimeClasses后，使用它们非常简单。
在Pod规范中指定runtimeClassName。 例如：

**apiVersion: v1**

**kind: Pod**

**metadata:**

**name: mypod**

**spec:**

**runtimeClassName: myclass**

***\# \...***

### 这将指示Kubelet使用命名的RuntimeClass来运行此pod。 如果指定的RuntimeClass不存在，或者CRI无法运行相应的处理程序，则pod将进入Failed终结阶段。 查找错误消息的相应事件。 如果未指定runtimeClassName，则将使用默认的RuntimeHandler，这相当于禁用RuntimeClass功能时的行为。

### CRI Configuration

For more details on setting up CRI runtimes, see [CRI
installation](https://kubernetes.io/docs/setup/cri/).

#### dockershim

Kubernetes built-in dockershim CRI does not support runtime handlers.

Kubernetes内置dockershim CRI不支持运行时处理程序。

#### [containerd](https://containerd.io/)

Runtime handlers are configured through containerd's configuration at
**/etc/containerd/config.toml**. Valid handlers are configured under the
runtimes section:

**\[plugins.cri.containerd.runtimes.\${HANDLER\_NAME}\]**

See containerd's config documentation for more details:
<https://github.com/containerd/cri/blob/master/docs/config.md>

#### [cri-o](https://cri-o.io/)

Runtime handlers are configured through cri-o's configuration at
**/etc/crio/crio.conf**. Valid handlers are configured under the
[crio.runtime
table](https://github.com/kubernetes-sigs/cri-o/blob/master/docs/crio.conf.5.md#crioruntime-table):

**\[crio.runtime.runtimes.\${HANDLER\_NAME}\]**

**runtime\_path = \"\${PATH\_TO\_BINARY}\"**

See cri-o's config documentation for more details:
<https://github.com/kubernetes-sigs/cri-o/blob/master/cmd/crio/config.go>

### Upgrading RuntimeClass from Alpha to Beta

### 

RuntimeClass Beta功能包括以下更改：\
•node.k8s.io
API组和runtimeclasses.node.k8s.io资源已从CustomResourceDefinition迁移到内置API。\
•规范已在RuntimeClass定义中内联（即不再有RuntimeClassSpec）。\
•runtimeHandler字段已重命名为handler。\
•现在，所有API版本都需要处理程序字段。这意味着还需要Alpha
API中的runtimeHandler字段。\
•处理程序字段必须是有效的DNS标签（RFC
1123），这意味着它不能再包含。字符（在所有版本中）。有效处理程序匹配以下正则表达式：\^
\[a-z0-9\]（\[ - a-z0-9\] \* \[a-z0-9\]）？\$。\
需要采取措施：从RuntimeClass功能的alpha版本升级到测试版需要执行以下操作：\
•升级到v1.14后必须重新创建RuntimeClass资源，并且应手动删除runtimeclasses.node.k8s.io
CRD：kubectl delete customresourcedefinitions.apiextensions.k8s.io
runtimeclasses.node.k8s.io\
•带有未指定或空的runtimeHandler的Alpha
RuntimeClasses或使用。处理程序中的字符不再有效，必须迁移到有效的处理程序配置（参见上文）。

Container Lifecycle Hooks
=========================

此页面描述了kubelet管理的容器如何使用Container生命周期钩子框架来运行在管理生命周期中由事件触发的代码。

-   [**Overview**](https://kubernetes.io/docs/concepts/containers/container-lifecycle-hooks/#overview)

-   [**Container
    hooks**](https://kubernetes.io/docs/concepts/containers/container-lifecycle-hooks/#container-hooks)

-   [**What\'s
    next**](https://kubernetes.io/docs/concepts/containers/container-lifecycle-hooks/#what-s-next)

Overview
--------

类似于许多具有组件生命周期钩子的编程语言框架，例如Angular，Kubernetes为Containers提供了生命周期钩子。
钩子使Container能够了解其管理生命周期中的事件，并在执行相应的生命周期钩子时运行在处理程序中实现的代码。

Container hooks
---------------

有两个暴露给容器的钩子：\
启动后\
在创建容器后立即执行此挂钩。
但是，无法保证挂钩将在容器ENTRYPOINT之前执行。 没有参数传递给处理程序。\
的prestop\
由于API请求或管理事件（例如活动探测失败，抢占，资源争用等），在容器终止之前立即调用此挂接。
如果容器已处于已终止或已完成状态，则对preStop挂接的调用将失败。
它是阻塞的，意味着它是同步的，所以它必须在删除容器的调用之前完成。
没有参数传递给处理程序。\
终止行为的更详细描述可以在终端中找到。

### Hook handler implementations

容器可以通过实现和注册该钩子的处理程序来访问钩子。
可以为Container实现两种类型的钩子处理程序：\
•Exec - 在Container的cgroups和名称空间内执行特定命令，例如pre-stop.sh。
命令消耗的资源将根据Container计算。\
•HTTP - 针对Container上的特定端点执行HTTP请求。

### Hook handler execution

调用Container生命周期管理挂钩时，Kubernetes管理系统会在为该挂钩注册的Container中执行处理程序。\
钩子处理程序调用在包含Container的Pod的上下文中是同步的。
这意味着对于PostStart挂钩，Container ENTRYPOINT和钩子异步激活。
但是，如果挂钩运行或挂起太长时间，则Container无法达到运行状态。\
PreStop挂钩的行为类似。
如果挂钩在执行期间挂起，则Pod阶段将保持Terminating状态，并在终止的FinallyGracePeriodSeconds结束后被终止。
如果PostStart或PreStop挂钩失败，则会终止Container。\
用户应使其钩子处理程序尽可能轻量级。
但是，有些情况下，长时间运行的命令是有意义的，例如在停止Container之前保存状态。

### Hook delivery guarantees

钩子传递至少是一次，这意味着对于任何给定的事件，例如对于PostStart或PreStop，可以多次调用钩子。
由钩子实现来正确处理这个问题。\
通常，只进行单次交付。
例如，如果HTTP挂钩接收器关闭且无法获取流量，则不会尝试重新发送。
然而，在一些罕见的情况下，可能会发生双重递送。
例如，如果一个kubelet在发送一个钩子的过程中重新启动，那么在该kubelet重新启动后可能会重新发送一个钩子。

### Debugging Hook handlers

在Pod事件中不公开Hook处理程序的日志。
如果处理程序由于某种原因失败，它会广播一个事件。
对于PostStart，这是FailedPostStartHook事件，对于PreStop，这是FailedPreStopHook事件。
您可以通过运行kubectl describe pod \<pod\_name\>来查看这些事件。
以下是运行此命令的一些事件输出示例：

**Events:**

**FirstSeen LastSeen Count From SubobjectPath Type Reason Message**

**\-\-\-\-\-\-\-\-- \-\-\-\-\-\-\-- \-\-\-\-- \-\-\--
\-\-\-\-\-\-\-\-\-\-\-\-- \-\-\-\-\-\-\-- \-\-\-\-\-- \-\-\-\-\-\--**

**1m 1m 1 {default-scheduler } Normal Scheduled Successfully assigned
test-1730497541-cq1d2 to gke-test-cluster-default-pool-a07e5d30-siqd**

**1m 1m 1 {kubelet gke-test-cluster-default-pool-a07e5d30-siqd}
spec.containers{main} Normal Pulling pulling image \"test:1.0\"**

**1m 1m 1 {kubelet gke-test-cluster-default-pool-a07e5d30-siqd}
spec.containers{main} Normal Created Created container with docker id
5c6a256a2567; Security:\[seccomp=unconfined\]**

**1m 1m 1 {kubelet gke-test-cluster-default-pool-a07e5d30-siqd}
spec.containers{main} Normal Pulled Successfully pulled image
\"test:1.0\"**

**1m 1m 1 {kubelet gke-test-cluster-default-pool-a07e5d30-siqd}
spec.containers{main} Normal Started Started container with docker id
5c6a256a2567**

**38s 38s 1 {kubelet gke-test-cluster-default-pool-a07e5d30-siqd}
spec.containers{main} Normal Killing Killing container with docker id
5c6a256a2567: PostStart handler: Error executing in Docker Container:
1**

**37s 37s 1 {kubelet gke-test-cluster-default-pool-a07e5d30-siqd}
spec.containers{main} Normal Killing Killing container with docker id
8df9fdfd7054: PostStart handler: Error executing in Docker Container:
1**

**38s 37s 2 {kubelet gke-test-cluster-default-pool-a07e5d30-siqd}
Warning FailedSync Error syncing pod, skipping: failed to
\"StartContainer\" for \"main\" with RunContainerError: \"PostStart
handler: Error executing in Docker Container: 1\"**

**1m 22s 2 {kubelet gke-test-cluster-default-pool-a07e5d30-siqd}
spec.containers{main} Warning FailedPostStartHook**

Volumes

-   Container中的磁盘文件是短暂的，这在容器中运行时会给非平凡的应用程序带来一些问题。
    首先，当容器崩溃时，kubelet将重新启动它，但文件将丢失 -
    Container以干净状态启动。
    其次，在Pod中一起运行Container时，通常需要在这些容器之间共享文件。
    Kubernetes Volume抽象解决了这两个问题。\
    建议熟悉Pods。

-   [**Background**](https://kubernetes.io/docs/concepts/storage/volumes/#background)

-   [**Types of
    Volumes**](https://kubernetes.io/docs/concepts/storage/volumes/#types-of-volumes)

-   [**Using
    subPath**](https://kubernetes.io/docs/concepts/storage/volumes/#using-subpath)

-   [**Resources**](https://kubernetes.io/docs/concepts/storage/volumes/#resources)

-   [**Out-of-Tree Volume
    Plugins**](https://kubernetes.io/docs/concepts/storage/volumes/#out-of-tree-volume-plugins)

-   [**Mount
    propagation**](https://kubernetes.io/docs/concepts/storage/volumes/#mount-propagation)

-   [**What\'s
    next**](https://kubernetes.io/docs/concepts/storage/volumes/#what-s-next)

    Background

Docker也有卷的概念，虽然它有点宽松和管理较少。在Docker中，卷只是磁盘上或另一个Container中的目录。生命周期不受管理，直到最近才有本地磁盘支持的卷。
Docker现在提供了卷驱动程序，但是现在功能非常有限（例如，从Docker
1.7开始，每个Container只允许一个卷驱动程序，并且无法将参数传递给卷）。\
另一方面，Kubernetes卷具有明确的生命周期 -
与包围它的Pod相同。因此，卷可以比Pod中运行的任何Container更长，并且可以在Container重新启动之间保留数据。当然，当Pod不再存在时，音量也将不复存在。也许更重要的是，Kubernetes支持多种类型的卷，Pod可以同时使用任意数量的卷。\
从本质上讲，卷只是一个目录，可能包含一些数据，Pod中的容器可以访问它。该目录是如何形成的，支持它的介质以及它的内容由所使用的特定卷类型决定。\
要使用卷，Pod指定要为Pod提供的卷（.spec.volumes字段）以及将这些卷安装到Container中的位置（.spec.containers.volumeMounts字段）。\
容器中的进程可以看到由Docker镜像和卷组成的文件系统视图。
Docker镜像位于文件系统层次结构的根目录下，任何卷都安装在图像中的指定路径上。卷无法装入其他卷或具有到其他卷的硬链接。
Pod中的每个Container必须独立指定每个卷的安装位置。

Types of Volumes

Kubernetes supports several types of Volumes:

-   [awsElasticBlockStore](https://kubernetes.io/docs/concepts/storage/volumes/#awselasticblockstore)

-   [azureDisk](https://kubernetes.io/docs/concepts/storage/volumes/#azuredisk)

-   [azureFile](https://kubernetes.io/docs/concepts/storage/volumes/#azurefile)

-   [cephfs](https://kubernetes.io/docs/concepts/storage/volumes/#cephfs)

-   [cinder](https://kubernetes.io/docs/concepts/storage/volumes/#cinder)

-   [configMap](https://kubernetes.io/docs/concepts/storage/volumes/#configmap)

-   [csi](https://kubernetes.io/docs/concepts/storage/volumes/#csi)

-   [downwardAPI](https://kubernetes.io/docs/concepts/storage/volumes/#downwardapi)

-   [emptyDir](https://kubernetes.io/docs/concepts/storage/volumes/#emptydir)

-   [fc (fibre
    channel)](https://kubernetes.io/docs/concepts/storage/volumes/#fc)

-   [flexVolume](https://kubernetes.io/docs/concepts/storage/volumes/#flexVolume)

-   [flocker](https://kubernetes.io/docs/concepts/storage/volumes/#flocker)

-   [gcePersistentDisk](https://kubernetes.io/docs/concepts/storage/volumes/#gcepersistentdisk)

-   [gitRepo
    (deprecated)](https://kubernetes.io/docs/concepts/storage/volumes/#gitrepo)

-   [glusterfs](https://kubernetes.io/docs/concepts/storage/volumes/#glusterfs)

-   [hostPath](https://kubernetes.io/docs/concepts/storage/volumes/#hostpath)

-   [iscsi](https://kubernetes.io/docs/concepts/storage/volumes/#iscsi)

-   [local](https://kubernetes.io/docs/concepts/storage/volumes/#local)

-   [nfs](https://kubernetes.io/docs/concepts/storage/volumes/#nfs)

-   [persistentVolumeClaim](https://kubernetes.io/docs/concepts/storage/volumes/#persistentvolumeclaim)

-   [projected](https://kubernetes.io/docs/concepts/storage/volumes/#projected)

-   [portworxVolume](https://kubernetes.io/docs/concepts/storage/volumes/#portworxvolume)

-   [quobyte](https://kubernetes.io/docs/concepts/storage/volumes/#quobyte)

-   [rbd](https://kubernetes.io/docs/concepts/storage/volumes/#rbd)

-   [scaleIO](https://kubernetes.io/docs/concepts/storage/volumes/#scaleio)

-   [secret](https://kubernetes.io/docs/concepts/storage/volumes/#secret)

-   [storageos](https://kubernetes.io/docs/concepts/storage/volumes/#storageos)

-   [vsphereVolume](https://kubernetes.io/docs/concepts/storage/volumes/#vspherevolume)

We welcome additional contributions.

awsElasticBlockStore

awsElasticBlockStore卷将Amazon Web Services（AWS）EBS卷安装到您的Pod中。
与删除Pod时删除的emptyDir不同，EBS卷的内容将被保留，并且仅卸载卷。
这意味着可以使用数据预先填充EBS卷，并且可以在Pod之间"切换"该数据。\
警告：您必须先使用aws ec2 create-volume或AWS
API创建EBS卷，然后才能使用它。\
使用awsElasticBlockStore卷时有一些限制：\
•正在运行Pod的节点必须是AWS EC2实例\
•这些实例需要与EBS卷位于同一区域和可用区域\
•EBS仅支持安装卷的单个EC2实例

Creating an EBS volume

在将Pod与EBS卷一起使用之前，需要先创建它。\
aws ec2 create-volume \--availability-zone = eu-west-1a \--size = 10
\--volume-type = gp2\
确保该区域与您启动群集的区域匹配。（并检查大小和EBS卷类型是否适合您的使用！）

AWS EBS Example configuration

**apiVersion: v1**

**kind: Pod**

**metadata:**

**name: test-ebs**

**spec:**

**containers:**

**- image: k8s.gcr.io/test-webserver**

**name: test-container**

**volumeMounts:**

**- mountPath: /test-ebs**

**name: test-volume**

**volumes:**

**- name: test-volume**

***\# This AWS EBS volume must already exist.***

**awsElasticBlockStore:**

**volumeID: \<volume-id*\>***

***fsType: ext4***

CSI Migration

**FEATURE STATE:** **Kubernetes v1.14**
[alpha](https://kubernetes.io/docs/concepts/storage/volumes/)

awsElasticBlockStore的CSI迁移功能在启用时，会将所有插件操作从现有的in-tree插件中填充到ebs.csi.aws.com容器存储接口（CSI）驱动程序中。
要使用此功能，必须在群集上安装AWS EBS
CSI驱动程序，并且必须启用CSIMigration和CSIMigrationAWS Alpha功能。

azureDisk

azureDisk用于将Microsoft Azure数据磁盘装入Pod。\
更多详细信息可以在这里找到。

azureFile

azureFile用于将Microsoft Azure文件卷（SMB 2.1和3.0）安装到Pod中。\
更多详细信息可以在这里找到。

cephfs

cephfs卷允许将现有的CephFS卷安装到Pod中。
与删除Pod时删除的emptyDir不同，将保留cephfs卷的内容，并且仅卸载卷。
这意味着可以使用数据预先填充CephFS卷，并且可以在Pod之间"切换"该数据。
CephFS可以由多个编写器同时安装。\
警告：在使用共享之前，必须运行自己的Ceph服务器并运行共享。\
有关更多详细信息，请参阅CephFS示例。

cinder

注意：先决条件：已配置OpenStack Cloud Provider的Kubernetes。
对于cloudprovider配置，请参阅云提供商openstack。\
cinder用于将OpenStack Cinder Volume安装到Pod中。

Cinder Volume Example configuration

**apiVersion: v1**

**kind: Pod**

**metadata:**

**name: test-cinder**

**spec:**

**containers:**

**- image: k8s.gcr.io/test-webserver**

**name: test-cinder-container**

**volumeMounts:**

**- mountPath: /test-cinder**

**name: test-volume**

**volumes:**

**- name: test-volume**

***\# This OpenStack volume must already exist.***

**cinder:**

**volumeID: \<volume-id*\>***

***fsType: ext4***

CSI Migration

特征状态：Kubernetes v1.14 alpha\
Cinder的CSI迁移功能在启用时，会将所有插件操作从现有的in-tree插件中恢复到cinder.csi.openstack.org容器存储接口（CSI）驱动程序。
要使用此功能，必须在群集上安装Openstack Cinder
CSI驱动程序，并且必须启用CSIMigration和CSIMigrationOpenStack Alpha功能。

configMap

configMap资源提供了一种将配置数据注入Pod的方法。
存储在ConfigMap对象中的数据可以在configMap类型的卷中引用，然后由在Pod中运行的容器化应用程序使用。\
引用configMap对象时，只需在卷中提供其名称即可引用它。
您还可以自定义ConfigMap中特定条目的路径。 例如，要将log-config
ConfigMap挂载到名为configmap-pod的Pod上，您可以使用下面的YAML：

**apiVersion: v1**

**kind: Pod**

**metadata:**

**name: configmap-pod**

**spec:**

**containers:**

**- name: test**

**image: busybox**

**volumeMounts:**

**- name: config-vol**

**mountPath: /etc/config**

**volumes:**

**- name: config-vol**

**configMap:**

**name: log-config**

**items:**

**- key: log\_level**

**path: log\_level**

log-config
ConfigMap作为卷安装，存储在其log\_level条目中的所有内容都安装在路径"/
etc / config / log\_level"的Pod中。
请注意，此路径派生自卷的mountPath和使用log\_level键入的路径。\
警告：必须先创建ConfigMap才能使用它。\
注意：使用ConfigMap作为子路径卷装入的Container将不会收到ConfigMap更新。

downwardAPI

向下API卷用于向应用程序提供向下的API数据。
它安装目录并将所请求的数据写入纯文本文件。\
注意：使用向下API作为子路径卷装入的Container不会接收向下API更新。\
有关更多详细信息，请参阅downwardAPI卷示例。

emptyDir

将Pod分配给节点时，首先会创建一个emptyDir卷，只要Pod在该节点上运行，就会存在。顾名思义，它最初是空的。
Pod中的容器都可以在emptyDir卷中读取和写入相同的文件，尽管该卷可以安装在每个Container中相同或不同的路径上。当出于任何原因从节点中删除Pod时，将永久删除emptyDir中的数据。\
注意：容器崩溃不会从节点中删除Pod，因此emptyDir卷中的数据在Container崩溃中是安全的。\
emptyDir的一些用途是：\
•临时空间，例如基于磁盘的合并排序\
•检查从崩溃中恢复的长计算\
•保存内容管理器Container在Web服务器容器提供数据时提取的文件\
默认情况下，emptyDir卷存储在支持节点的任何介质上 -
可能是磁盘或SSD或网络存储，具体取决于您的环境。但是，您可以将emptyDir.medium字段设置为"Memory"，以告知Kubernetes为您安装tmpfs（RAM支持的文件系统）。虽然tmpfs非常快，但请注意，与磁盘不同，tmpfs在节点重新启动时被清除，并且您编写的任何文件都将计入Container的内存限制。

Example Pod

**apiVersion: v1**

**kind: Pod**

**metadata:**

**name: test-pd**

**spec:**

**containers:**

**- image: k8s.gcr.io/test-webserver**

**name: test-container**

**volumeMounts:**

**- mountPath: /cache**

**name: cache-volume**

**volumes:**

**- name: cache-volume**

**emptyDir: {}**

fc (fibre channel)

fc音量允许将现有光纤通道音量安装在Pod中。
您可以使用卷配置中的参数targetWWNs指定单个或多个目标全球通用名称。
如果指定了多个WWN，则targetWWN期望这些WWN来自多路径连接。\
警告：您必须配置FC
SAN分区以预先将这些LUN（卷）分配和屏蔽到目标WWN，以便Kubernetes主机可以访问它们。\
有关更多详细信息，请参阅FC示例。

flocker

Flocker是一个开源的集群Container数据卷管理器。
它提供由各种存储后端支持的数据卷的管理和编排。\
flocker卷允许将Flocker数据集安装到Pod中。
如果Flocker中尚不存在数据集，则需要首先使用Flocker CLI或使用Flocker
API创建数据集。
如果数据集已经存在，它将被Flocker重新附加到调度Pod的节点。
这意味着可以根据需要在Pod之间"切换"数据。\
警告：必须先运行自己的Flocker安装才能使用它。\
有关更多详细信息，请参阅Flocker示例。

gcePersistentDisk

gcePersistentDisk卷将Google Compute
Engine（GCE）永久磁盘装入Pod。与删除Pod时删除的emptyDir不同，PD的内容将被保留，并且仅卸载卷。这意味着PD可以预先填充数据，并且该数据可以在Pod之间"切换"。\
警告：您必须先使用gcloud或GCE API或UI创建PD，然后才能使用它。\
使用gcePersistentDisk时有一些限制：\
•运行Pod的节点必须是GCE VM\
•这些VM需要与PD位于同一GCE项目和区域中\
PD的一个特征是它们可以同时由多个消费者以只读方式安装。这意味着您可以使用数据集预先填充PD，然后根据需要从多个Pod中并行提供。不幸的是，PD只能由一个消费者以读写模式安装
- 不允许同时写入。\
除非PD是只读的或副本计数为0或1，否则在由ReplicationController控制的Pod上使用PD将失败。

Creating a PD

Before you can use a GCE PD with a Pod, you need to create it.

**gcloud compute disks create \--size=500GB \--zone=us-central1-a
my-data-disk**

Example Pod

**apiVersion: v1**

**kind: Pod**

**metadata:**

**name: test-pd**

**spec:**

**containers:**

**- image: k8s.gcr.io/test-webserver**

**name: test-container**

**volumeMounts:**

**- mountPath: /test-pd**

**name: test-volume**

**volumes:**

**- name: test-volume**

***\# This GCE PD must already exist.***

**gcePersistentDisk:**

**pdName: my-data-disk**

**fsType: ext4**

Regional Persistent Disks

**FEATURE STATE:** **Kubernetes v1.10**
[beta](https://kubernetes.io/docs/concepts/storage/volumes/)

区域持久磁盘功能允许创建在同一区域内的两个区域中可用的永久磁盘。
要使用此功能，必须将卷配置为PersistentVolume; 不支持直接从pod引用卷。

Manually provisioning a Regional PD PersistentVolume

使用StorageClass for GCE PD可以进行动态配置。
在创建PersistentVolume之前，您必须创建PD：

**gcloud beta compute disks create \--size=500GB my-data-disk**

**\--region us-central1**

**\--replica-zones us-central1-a,us-central1-b**

Example PersistentVolume spec:

**apiVersion: v1**

**kind: PersistentVolume**

**metadata:**

**name: test-volume**

**labels:**

**failure-domain.beta.kubernetes.io/zone:
us-central1-a\_\_us-central1-b**

**spec:**

**capacity:**

**storage: 400Gi**

**accessModes:**

**- ReadWriteOnce**

**gcePersistentDisk:**

**pdName: my-data-disk**

**fsType: ext4**

CSI Migration

特征状态：Kubernetes v1.14 alpha\
GCE
PD的CSI迁移功能在启用时，将所有插件操作从现有的in-tree插件中填充到pd.csi.storage.gke.io容器存储接口（CSI）驱动程序。
要使用此功能，必须在群集上安装GCE PD
CSI驱动程序，并且必须启用CSIMigration和CSIMigrationGCE Alpha功能。

gitRepo (deprecated)

警告：不推荐使用gitRepo卷类型。 要使用git
repo配置容器，请将EmptyDir挂载到使用git克隆repo的InitContainer中，然后将EmptyDir挂载到Pod的容器中。\
gitRepo卷是可以作为卷插件完成的示例。
它安装了一个空目录，并将一个git存储库克隆到它中，供Pod使用。
将来，这些卷可能会转移到更加分离的模型，而不是为每个这样的用例扩展Kubernetes
API。

Here is an example of gitRepo volume:

**apiVersion: v1**

**kind: Pod**

**metadata:**

**name: server**

**spec:**

**containers:**

**- image: nginx**

**name: nginx**

**volumeMounts:**

**- mountPath: /mypath**

**name: git-volume**

**volumes:**

**- name: git-volume**

**gitRepo:**

**repository: \"git\@somewhere:me/my-git-repository.git\"**

**revision: \"22f1d8406d464b0c0874075539c1f2e96c253775\"**

glusterfs

glusterfs卷允许将Glusterfs（开源网络文件系统）卷安装到Pod中。
与删除Pod时删除的emptyDir不同，glusterfs卷的内容将被保留，并且仅卸载卷。
这意味着可以使用数据预先填充glusterfs卷，并且可以在Pod之间"切换"该数据。
GlusterFS可以由多个编写器同时安装。\
警告：必须先运行自己的GlusterFS安装，然后才能使用它。\
有关更多详细信息，请参阅GlusterFS示例。

hostPath

hostPath卷将文件或目录从主机节点的文件系统安装到Pod中。
这不是大多数Pod需要的东西，但它为某些应用程序提供了强大的逃生舱。\
例如，hostPath的一些用途是：\
•运行需要访问Docker内部的Container; 使用/ var / lib / docker的hostPath\
•在容器中运行cAdvisor; 使用/ sys的hostPath\
•允许Pod指定在Pod运行之前是否应该存在给定的hostPath，是否应该创建它以及它应该存在的内容\
除了必需的path属性之外，用户还可以选择为hostPath卷指定类型。

The supported values for field **type** are:

  Value                   Behavior
  ----------------------- ------------------------------------------------------------------------------------------------------------------------------------------------------------------------
                          Empty string (default) is for backward compatibility, which means that no checks will be performed before mounting the hostPath volume.
  **DirectoryOrCreate**   If nothing exists at the given path, an empty directory will be created there as needed with permission set to 0755, having the same group and ownership with Kubelet.
  **Directory**           A directory must exist at the given path
  **FileOrCreate**        If nothing exists at the given path, an empty file will be created there as needed with permission set to 0644, having the same group and ownership with Kubelet.
  **File**                A file must exist at the given path
  **Socket**              A UNIX socket must exist at the given path
  **CharDevice**          A character device must exist at the given path
  **BlockDevice**         A block device must exist at the given path

请注意何时使用此类型的卷，因为：\
•由于节点上的文件不同，具有相同配置的Pod（例如从podTemplate创建）在不同节点上的行为可能会有所不同\
•当Kubernetes按计划添加资源感知调度时，它将无法考虑hostPath使用的资源\
•在基础主机上创建的文件或目录只能由root写入。
您需要以特权Container中的root身份运行您的进程，或者修改主机上的文件权限以便能够写入hostPath卷

Example Pod

**apiVersion: v1**

**kind: Pod**

**metadata:**

**name: test-pd**

**spec:**

**containers:**

**- image: k8s.gcr.io/test-webserver**

**name: test-container**

**volumeMounts:**

**- mountPath: /test-pd**

**name: test-volume**

**volumes:**

**- name: test-volume**

**hostPath:**

***\# directory location on host***

**path: /data**

***\# this field is optional***

**type: Directory**

iscsi

iscsi卷允许将现有iSCSI（SCSI over IP）卷安装到Pod中。
与删除Pod时删除的emptyDir不同，iscsi卷的内容将被保留，并且仅卸载卷。
这意味着可以使用数据预先填充iscsi卷，并且可以在Pod之间"切换"该数据。\
警告：必须先使用创建的卷运行自己的iSCSI服务器，然后才能使用它。\
iSCSI的一个特性是它可以同时由多个消费者以只读方式安装。
这意味着您可以使用数据集预填充卷，然后根据需要从多个Pod中并行提供。
遗憾的是，iSCSI卷只能由单个用户以读写模式安装 - 不允许同时写入。\
有关更多详细信息，请参阅iSCSI示例。

local

**FEATURE STATE:** **Kubernetes v1.14**
[stable](https://kubernetes.io/docs/concepts/storage/volumes/)

本地卷表示已安装的本地存储设备，例如磁盘，分区或目录。\
本地卷只能用作静态创建的PersistentVolume。 尚不支持动态配置。\
与hostPath卷相比，可以以持久且可移植的方式使用本地卷，而无需手动将Pod调度到节点，因为系统通过查看PersistentVolume上的节点关联性来了解卷的节点约束。\
但是，本地卷仍受基础节点可用性的限制，并不适用于所有应用程序。
如果节点变得不健康，则本地卷也将变得不可访问，并且使用它的Pod将无法运行。
使用本地卷的应用程序必须能够容忍这种降低的可用性以及潜在的数据丢失，具体取决于底层磁盘的持久性特征。

以下是使用本地卷和nodeAffinity的PersistentVolume规范的示例：

**apiVersion: v1**

**kind: PersistentVolume**

**metadata:**

**name: example-pv**

**spec:**

**capacity:**

**storage: 100Gi**

***\# volumeMode field requires BlockVolume Alpha feature gate to be
enabled.***

**volumeMode: Filesystem**

**accessModes:**

**- ReadWriteOnce**

**persistentVolumeReclaimPolicy: Delete**

**storageClassName: local-storage**

**local:**

**path: /mnt/disks/ssd1**

**nodeAffinity:**

**required:**

**nodeSelectorTerms:**

**- matchExpressions:**

**- key: kubernetes.io/hostname**

**operator: In**

**values:**

**- example-node**

使用本地卷时需要PersistentVolume
nodeAffinity。它使Kubernetes调度程序能够使用本地卷正确地将Pod调度到正确的节点。\
PersistentVolume
volumeMode现在可以设置为"Block"（而不是默认值"Filesystem"），以将本地卷公开为原始块设备。
volumeMode字段需要启用BlockVolume Alpha功能门。\
使用本地卷时，建议创建一个将volumeBindingMode设置为WaitForFirstConsumer的StorageClass。查看示例。延迟卷绑定可确保PersistentVolumeClaim绑定决策也将使用Pod可能具有的任何其他节点约束进行评估，例如节点资源要求，节点选择器，Pod关联和Pod反关联。\
可以单独运行外部静态配置程序，以改进对本地卷生命周期的管理。请注意，此配置程序尚不支持动态配置。有关如何运行外部本地配置程序的示例，请参阅本地卷配置程序用户指南。\
注意：如果未使用外部静态配置器来管理卷生命周期，则本地PersistentVolume需要用户手动清理和删除。

nfs

nfs卷允许将现有NFS（网络文件系统）共享安装到Pod中。
与删除Pod时删除的emptyDir不同，nfs卷的内容将被保留，并且仅卸载卷。
这意味着可以使用数据预先填充NFS卷，并且可以在Pod之间"切换"该数据。
NFS可以由多个编写器同时安装。\
警告：必须先使用已导出的共享运行自己的NFS服务器，然后才能使用它。\
有关更多详细信息，请参见NFS示例。

persistentVolumeClaim

persistentVolumeClaim卷用于将PersistentVolume挂载到Pod中。
PersistentVolumes是用户在不知道特定云环境的详细信息的情况下"声明"持久存储（例如GCE
PersistentDisk或iSCSI卷）的一种方式。\
有关更多详细信息，请参阅PersistentVolumes示例。

projected

预计卷将几个现有卷源映射到同一目录中。\
目前，可以投射以下类型的音量源：

-   [**[secret]{.underline}**](https://kubernetes.io/docs/concepts/storage/volumes/#secret)

-   [**[downwardAPI]{.underline}**](https://kubernetes.io/docs/concepts/storage/volumes/#downwardapi)

-   [**[configMap]{.underline}**](https://kubernetes.io/docs/concepts/storage/volumes/#configmap)

-   **serviceAccountToken**

所有源都需要与Pod在同一名称空间中。
有关更多详细信息，请参阅一体化卷设计文档。\
服务帐户令牌的预测是Kubernetes 1.11中引入的功能，并在1.12中提升为Beta。
要在1.11上启用此功能，您需要将TokenRequestProjection功能门显式设置为True。

Example Pod with a secret, a downward API, and a configmap.

**apiVersion: v1**

**kind: Pod**

**metadata:**

**name: volume-test**

**spec:**

**containers:**

**- name: container-test**

**image: busybox**

**volumeMounts:**

**- name: all-in-one**

**mountPath: \"/projected-volume\"**

**readOnly: true**

**volumes:**

**- name: all-in-one**

**projected:**

**sources:**

**- secret:**

**name: mysecret**

**items:**

**- key: username**

**path: my-group/my-username**

**- downwardAPI:**

**items:**

**- path: \"labels\"**

**fieldRef:**

**fieldPath: metadata.labels**

**- path: \"cpu\_limit\"**

**resourceFieldRef:**

**containerName: container-test**

**resource: limits.cpu**

**- configMap:**

**name: myconfigmap**

**items:**

**- key: config**

**path: my-group/my-config**

Example Pod with multiple secrets with a non-default permission mode
set.

**apiVersion: v1**

**kind: Pod**

**metadata:**

**name: volume-test**

**spec:**

**containers:**

**- name: container-test**

**image: busybox**

**volumeMounts:**

**- name: all-in-one**

**mountPath: \"/projected-volume\"**

**readOnly: true**

**volumes:**

**- name: all-in-one**

**projected:**

**sources:**

**- secret:**

**name: mysecret**

**items:**

**- key: username**

**path: my-group/my-username**

**- secret:**

**name: mysecret2**

**items:**

**- key: password**

**path: my-group/my-password**

**mode: 511**

每个预计的音量源都列在源下的规范中。 参数几乎相同，但有两个例外：\
•对于机密，secretName字段已更改为name以与ConfigMap命名一致。\
•defaultMode只能在预计级别指定，而不能在每个卷源中指定。
但是，如上所示，您可以为每个单独的投影明确设置模式。\
启用TokenRequestProjection功能后，您可以将当前服务帐户的令牌注入指定路径的Pod。
以下是一个例子：

**apiVersion: v1**

**kind: Pod**

**metadata:**

**name: sa-token-test**

**spec:**

**containers:**

**- name: container-test**

**image: busybox**

**volumeMounts:**

**- name: token-vol**

**mountPath: \"/service-account\"**

**readOnly: true**

**volumes:**

**- name: token-vol**

**projected:**

**sources:**

**- serviceAccountToken:**

**audience: api**

**expirationSeconds: 3600**

**path: token**

示例Pod具有包含注入的服务帐户令牌的预计卷。
例如，Pod容器可以使用此标记来访问Kubernetes API服务器。
受众群体字段包含令牌的目标受众。
令牌的接收者必须使用令牌的受众中指定的标识符来标识自己，否则应拒绝该令牌。
此字段是可选字段，默认为API服务器的标识符。\
expirationSeconds是服务帐户令牌的预期有效期。
默认为1小时，且必须至少为10分钟（600秒）。
管理员还可以通过为API服务器指定\--service-account-max-token-expiration选项来限制其最大值。
path字段指定投影体积的安装点的相对路径。\
注意：使用预计卷源作为子路径卷装入的Container将不会接收这些卷源的更新。

portworxVolume

portworxVolume是一个弹性块存储层，与Kubernetes运行超融合。
Portworx指纹存储在服务器中，基于功能层，并聚合多个服务器之间的容量。
Portworx在虚拟机或裸机Linux节点上运行in-guest。\
portworxVolume可以通过Kubernetes动态创建，也可以在Kubernetes
Pod中预先配置和引用。 以下是引用预先配置的PortworxVolume的Pod示例：

**apiVersion: v1**

**kind: Pod**

**metadata:**

**name: test-portworx-volume-pod**

**spec:**

**containers:**

**- image: k8s.gcr.io/test-webserver**

**name: test-container**

**volumeMounts:**

**- mountPath: /mnt**

**name: pxvol**

**volumes:**

**- name: pxvol**

***\# This Portworx volume must already exist.***

**portworxVolume:**

**volumeID: \"pxvol\"**

**fsType: \"\<fs-type\>\"**

警告：在Pod中使用之前，请确保您具有名为pxvol的现有PortworxVolume。\
更多细节和示例可以在这里找到。

quobyte

一个quobyte卷允许将现有的Quobyte卷安装到Pod中。\
警告：必须先使用创建的卷运行自己的Quobyte设置，然后才能使用它。

Quobyte supports the [Container Storage
Interface](https://kubernetes.io/docs/concepts/storage/volumes/#csi)The
Container Storage Interface (CSI) defines a standard interface to expose
storage systems to containers. .
CSI是推荐在Kubernetes中使用Quobyte卷的插件。
Quobyte的GitHub项目提供了使用CSI部署Quobyte的说明以及示例。

rbd

rbd卷允许将Rados Block Device卷安装到Pod中。
与删除Pod时删除的emptyDir不同，rbd卷的内容将被保留，并且仅卸载卷。
这意味着RBD卷可以预先填充数据，并且该数据可以在Pod之间"切换"。\
警告：在使用RBD之前，必须先运行自己的Ceph安装。\
RBD的一个特点是它可以同时由多个消费者以只读方式安装。
这意味着您可以使用数据集预填充卷，然后根据需要从多个Pod中并行提供。
不幸的是，RBD卷只能由一个消费者以读写模式安装 - 不允许同时写入。\
有关更多详细信息，请参阅RBD示例。

scaleIO

ScaleIO是一个基于软件的存储平台，可以使用现有硬件来创建可伸缩共享块网络存储的集群。
scaleIO卷插件允许已部署的Pod访问现有的ScaleIO卷（或者可以为持久卷声明动态配置新卷，请参阅ScaleIO持久卷）。\
警告：必须先安装现有的ScaleIO群集，并在创建卷之前运行它们，然后才能使用它们。

The following is an example of Pod configuration with ScaleIO:

**apiVersion: v1**

**kind: Pod**

**metadata:**

**name: pod-0**

**spec:**

**containers:**

**- image: k8s.gcr.io/test-webserver**

**name: pod-0**

**volumeMounts:**

**- mountPath: /test-pd**

**name: vol-0**

**volumes:**

**- name: vol-0**

**scaleIO:**

**gateway: https://localhost:443/api**

**system: scaleio**

**protectionDomain: sd0**

**storagePool: sp1**

**volumeName: vol-0**

**secretRef:**

**name: sio-secret**

**fsType: xfs**

For further detail, please see the [ScaleIO
examples](https://github.com/kubernetes/examples/tree/master/staging/volumes/scaleio).

secret

秘密卷用于将敏感信息（如密码）传递给Pod。 您可以将密钥存储在Kubernetes
API中，并将其作为文件挂载以供Pod使用，而无需直接耦合到Kubernetes。
秘密卷由tmpfs（RAM支持的文件系统）支持，因此它们永远不会写入非易失性存储。\
警告：您必须先在Kubernetes API中创建一个秘密，然后才能使用它。\
注意：使用Secret作为子路径卷装入的Container将不会收到Secret更新。\
这里将更详细地描述秘密。

storageOS

storageos卷允许将现有StorageOS卷安装到Pod中。\
StorageOS在Kubernetes环境中作为Container运行，从而可以从Kubernetes集群中的任何节点访问本地或附加存储。
可以复制数据以防止节点故障。 精简配置和压缩可以提高利用率并降低成本。\
StorageOS的核心是为容器提供块存储，可通过文件系统访问。\
StorageOS Container需要64位Linux，并且没有其他依赖项。
提供免费的开发人员许可。\
警告：您必须在要访问StorageOS卷的每个节点上运行StorageOS
Container，或者为池提供存储容量。 有关安装说明，请参阅StorageOS文档。

**apiVersion: v1**

**kind: Pod**

**metadata:**

**labels:**

**name: redis**

**role: master**

**name: test-storageos-redis**

**spec:**

**containers:**

**- name: master**

**image: kubernetes/redis:v1**

**env:**

**- name: MASTER**

**value: \"true\"**

**ports:**

**- containerPort: 6379**

**volumeMounts:**

**- mountPath: /redis-master-data**

**name: redis-data**

**volumes:**

**- name: redis-data**

**storageos:**

***\# The \`redis-vol01\` volume must already exist within StorageOS in
the \`default\` namespace.***

**volumeName: redis-vol01**

**fsType: ext4**

For more information including Dynamic Provisioning and Persistent
Volume Claims, please see the [StorageOS
examples](https://github.com/kubernetes/examples/blob/master/staging/volumes/storageos).

vsphereVolume

注意：先决条件：已配置vSphere Cloud Provider的Kubernetes。
有关cloudprovider配置，请参阅vSphere入门指南。\
vsphereVolume用于将vSphere VMDK卷装入Pod。 在卸载卷时，将保留卷的内容。
它支持VMFS和VSAN数据存储。\
警告：在使用Pod之前，必须使用以下方法之一创建VMDK。

Creating a VMDK volume

Choose one of the following methods to create a VMDK.

-   [Create using
    vmkfstools](https://kubernetes.io/docs/concepts/storage/volumes/#tabs-volumes-0)

-   [Create using
    vmware-vdiskmanager](https://kubernetes.io/docs/concepts/storage/volumes/#tabs-volumes-1)

First ssh into ESX, then use the following command to create a VMDK:

**vmkfstools -c 2G /vmfs/volumes/DatastoreName/volumes/myDisk.vmdk**

Use the following command to create a VMDK:

**vmware-vdiskmanager -c -t 0 -s 40GB -a lsilogic myDisk.vmdk**

vSphere VMDK Example configuration

**apiVersion: v1**

**kind: Pod**

**metadata:**

**name: test-vmdk**

**spec:**

**containers:**

**- image: k8s.gcr.io/test-webserver**

**name: test-container**

**volumeMounts:**

**- mountPath: /test-vmdk**

**name: test-volume**

**volumes:**

**- name: test-volume**

***\# This VMDK volume must already exist.***

**vsphereVolume:**

**volumePath: \"\[DatastoreName\] volumes/myDisk\"**

**fsType: ext4**

More examples can be found
[here](https://github.com/kubernetes/examples/tree/master/staging/volumes/vsphere).

Using subPath

有时，在单个Pod中共享一个卷用于多个用途很有用。
volumeMounts.subPath属性可用于指定引用卷内的子路径而不是其根。\
下面是使用单个共享卷的Pod与LAMP堆栈（Linux Apache Mysql PHP）的示例。
HTML内容映射到其html文件夹，数据库将存储在其mysql文件夹中：

**apiVersion: v1**

**kind: Pod**

**metadata:**

**name: my-lamp-site**

**spec:**

**containers:**

**- name: mysql**

**image: mysql**

**env:**

**- name: MYSQL\_ROOT\_PASSWORD**

**value: \"rootpasswd\"**

**volumeMounts:**

**- mountPath: /var/lib/mysql**

**name: site-data**

**subPath: mysql**

**- name: php**

**image: php:7.0-apache**

**volumeMounts:**

**- mountPath: /var/www/html**

**name: site-data**

**subPath: html**

**volumes:**

**- name: site-data**

**persistentVolumeClaim:**

**claimName: my-lamp-site-data**

Using subPath with expanded environment variables

**FEATURE STATE:** **Kubernetes v1.14**
[alpha](https://kubernetes.io/docs/concepts/storage/volumes/)

使用subPathExpr字段从Downward API环境变量构造subPath目录名。
在使用此功能之前，必须启用VolumeSubpathEnvExpansion功能门。
subPath和subPathExpr属性是互斥的。\
在此示例中，Pod使用subPathExpr使用Downward API中的pod名称在hostPath
volume / var / log / pods中创建目录pod1。 主机目录/ var / log / pods /
pod1安装在容器中的/ logs中。

**apiVersion: v1**

**kind: Pod**

**metadata:**

**name: pod1**

**spec:**

**containers:**

**- name: container1**

**env:**

**- name: POD\_NAME**

**valueFrom:**

**fieldRef:**

**apiVersion: v1**

**fieldPath: metadata.name**

**image: busybox**

**command: \[ \"sh\", \"-c\", \"while \[ true \]; do echo \'Hello\';
sleep 10; done \| tee -a /logs/hello.txt\" \]**

**volumeMounts:**

**- name: workdir1**

**mountPath: /logs**

**subPathExpr: \$(POD\_NAME)**

**restartPolicy: Never**

**volumes:**

**- name: workdir1**

**hostPath:**

**path: /var/log/pods**

Resources

emptyDir卷的存储介质（磁盘，SSD等）由保存kubelet根目录的文件系统的介质（通常为/
var / lib / kubelet）确定。
emptyDir或hostPath卷可以占用多少空间，Container之间或Pod之间没有隔离。\
将来，我们希望emptyDir和hostPath卷能够使用资源规范请求一定量的空间，并为具有多种媒体类型的群集选择要使用的媒体类型。

Out-of-Tree Volume Plugins

Out-of-tree卷插件包括Container Storage Interface（CSI）和Flexvolume。
它们使存储供应商能够创建自定义存储插件，而无需将其添加到Kubernetes存储库。\
在引入CSI和Flexvolume之前，所有卷插件（如上面列出的卷类型）都是"in-tree"，这意味着它们是与核心Kubernetes二进制文件一起构建，链接，编译和附带的，并扩展了核心Kubernetes
API。
这意味着向Kubernetes（卷插件）添加新的存储系统需要将代码检入核心Kubernetes代码库。\
CSI和Flexvolume都允许独立于Kubernetes代码库开发卷插件，并作为扩展部署（安装）在Kubernetes集群上。\
对于希望创建树外卷插件的存储供应商，请参阅此常见问题解答。

CSI

-   容器存储接口（CSI）为容器编排系统（如Kubernetes）定义了一个标准接口，以将任意存储系统暴露给其容器工作负载。\
    请阅读CSI设计方案以获取更多信息。\
    CSI支持在Kubernetes v1.9中作为alpha引入，在Kubernetes
    v1.10中转为beta，在Kubernetes v1.13中是GA。\
    注意：在Kubernetes
    v1.13中不推荐支持CSI规范版本0.2和0.3，并将在以后的版本中删除。\
    注意：CSI驱动程序可能与所有Kubernetes版本不兼容。请查看特定的CSI驱动程序文档，了解每个Kubernetes版本的支持部署步骤和兼容性矩阵。\
    在Kubernetes集群上部署CSI兼容卷驱动程序后，用户可以使用csi卷类型来附加，装载等CSI驱动程序公开的卷。\
    csi卷类型不支持Pod的直接引用，只能通过PersistentVolumeClaim对象在Pod中引用。\
    存储管理员可以使用以下字段来配置CSI持久卷：\
    •driver：一个字符串值，指定要使用的卷驱动程序的名称。该值必须与CSI规范中定义的CSI驱动程序在GetPluginInfoResponse中返回的值相对应。
    Kubernetes使用它来识别要呼叫的CSI驱动程序，并通过CSI驱动程序组件来识别哪些PV对象属于CSI驱动程序。\
    •volumeHandle：唯一标识卷的字符串值。此值必须与CSI规范中定义的CSI驱动程序在CreateVolumeResponse的volume.id字段中返回的值相对应。在引用卷时，该值将作为volume\_id传递给CSI卷驱动程序的所有调用。\
    •readOnly：一个可选的布尔值，指示卷是否为"ControllerPublished"（附加）为只读。默认值为false。该值通过ControllerPublishVolumeRequest中的只读字段传递给CSI驱动程序。\
    •fsType：如果PV的VolumeMode是Filesystem，则此字段可用于指定应用于装入卷的文件系统。如果未格式化卷并且支持格式化，则此值将用于格式化卷。此值通过ControllerPublishVolumeRequest，NodeStageVolumeRequest和NodePublishVolumeRequest的VolumeCapability字段传递给CSI驱动程序。\
    •volumeAttributes：字符串到字符串的映射，指定卷的静态属性。此映射必须对应于CSI规范中定义的CSI驱动程序在CreateVolumeResponse的volume.attributes字段中返回的映射。地图通过ControllerPublishVolumeRequest，NodeStageVolumeRequest和NodePublishVolumeRequest中的volume\_attributes字段传递给CSI驱动程序。\
    •controllerPublishSecretRef：对包含敏感信息的秘密对象的引用，以传递给CSI驱动程序以完成CSI
    ControllerPublishVolume和ControllerUnpublishVolume调用。此字段是可选的，如果不需要密码，则该字段可能为空。如果秘密对象包含多个秘密，则传递所有秘密。\
    •nodeStageSecretRef：对包含敏感信息的秘密对象的引用，以传递给CSI驱动程序以完成CSI
    NodeStageVolume调用。此字段是可选的，如果不需要密码，则该字段可能为空。如果秘密对象包含多个秘密，则传递所有秘密。\
    •nodePublishSecretRef：对包含敏感信息的秘密对象的引用，以传递给CSI驱动程序以完成CSI
    NodePublishVolume调用。此字段是可选的，如果不需要密码，则该字段可能为空。如果秘密对象包含多个秘密，则传递所有秘密。

    CSI raw block volume support

**FEATURE STATE:** **Kubernetes v1.14**
[beta](https://kubernetes.io/docs/concepts/storage/volumes/)

从版本1.11开始，CSI引入了对原始块体积的支持，它依赖于先前版本的Kubernetes中引入的原始块体积功能。
此功能将使具有外部CSI驱动程序的供应商能够在Kubernetes工作负载中实现原始块容量支持。\
CSI块卷支持是功能门控，但默认启用。
必须为此功能启用的两个功能门是BlockVolume和CSIBlockVolume。

Learn how to [setup your PV/PVC with raw block volume
support](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#raw-block-volume-support).

Developer resources

For more information on how to develop a CSI driver, refer to the
[kubernetes-csi documentation](https://kubernetes-csi.github.io/docs/)

Migrating to CSI drivers from in-tree plugins

**FEATURE STATE:** **Kubernetes v1.14**
[alpha](https://kubernetes.io/docs/concepts/storage/volumes/)

CSI迁移功能在启用时，会将针对现有树内插件的操作定向到相应的CSI插件（预计将安装和配置）。
该功能实现了必要的转换逻辑和垫片，以无缝方式重新路由操作。
因此，在转换到取代树内插件的CSI驱动程序时，运营商不必对现有存储类，PV或PVC（指向树内插件）进行任何配置更改。\
在alpha状态下，支持的操作和功能包括配置/删除，附加/分离以及将volumeMode设置为filesystem的卷的装载/卸载\
支持CSI迁移并具有相应CSI驱动程序的树内插件列在上面的"卷类型"部分中。

Flexvolume

Flexvolume是一个树外插件接口，自1.2版本（CSI之前）就存在于Kubernetes中。
它使用基于exec的模型来与驱动程序进行交互。
Flexvolume驱动程序二进制文件必须安装在每个节点（在某些情况下为master）的预定义卷插件路径中。\
Pod通过flexvolume in-tree插件与Flexvolume驱动程序进行交互。
更多详细信息可以在这里找到。

Mount propagation

挂载传播允许将Container挂载的卷共享到同一Pod中的其他Container，甚至可以共享到同一节点上的其他Pod。\
卷的传播传播由Container.volumeMounts中的mountPropagation字段控制。它的价值观是：\
•无 -
此卷装入不会接收主机安装到此卷或其任何子目录的任何后续装载。以类似的方式，在主机上不会显示Container创建的装载。这是默认模式。\
此模式等同于Linux内核文档中所述的私有安装传播\
•HostToContainer -
此卷装入将接收安装到此卷或其任何子目录的所有后续装入。\
换句话说，如果主机在卷安装内安装任何东西，Container将看到它安装在那里。\
同样，如果任何具有双向挂载传播的Pod挂载到同一个卷中，那么具有HostToContainer挂载传播的Container将会看到它。\
此模式等同于Linux内核文档中描述的rslave挂载传播\
•双向 -
此卷安装与HostToContainer安装的行为相同。此外，Container创建的所有卷安装都将传播回主机和所有使用相同卷的Pod的所有容器。\
此模式的典型用例是具有Flexvolume或CSI驱动程序的Pod或需要使用hostPath卷在主机上安装内容的Pod。\
此模式等同于Linux内核文档中描述的rshared安装传播\
注意：双向安装传播可能很危险。它可能会损坏主机操作系统，因此只允许在特权容器中使用它。强烈建议您熟悉Linux内核行为。此外，容器在容器中创建的任何卷装入必须在终止时由容器销毁（卸载）。

Configuration

在挂载传播可以在某些部署（CoreOS，RedHat /
Centos，Ubuntu）上正常工作之前，必须在Docker中正确配置挂载共享，如下所示。

Edit your Docker's **systemd** service file. Set **MountFlags** as
follows:

**MountFlags=shared**

Or, remove **MountFlags=slave** if present. Then restart the Docker
daemon:

**sudo systemctl daemon-reload**

**sudo systemctl restart docker**

Persistent Volumes

本文档描述了Kubernetes中PersistentVolumes的当前状态。 建议熟悉卷。

-   [**Introduction**](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#introduction)

-   [**Lifecycle of a volume and
    claim**](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#lifecycle-of-a-volume-and-claim)

-   [**Types of Persistent
    Volumes**](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#types-of-persistent-volumes)

-   [**Persistent
    Volumes**](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#persistent-volumes)

-   [**PersistentVolumeClaims**](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#persistentvolumeclaims)

-   [**Claims As
    Volumes**](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#claims-as-volumes)

-   [**Raw Block Volume
    Support**](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#raw-block-volume-support)

-   [**Volume Snapshot and Restore Volume from Snapshot
    Support**](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#volume-snapshot-and-restore-volume-from-snapshot-support)

-   [**Writing Portable
    Configuration**](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#writing-portable-configuration)

    Introduction

管理存储是管理计算的一个明显问题。
PersistentVolume子系统为用户和管理员提供了一个API，它提供了如何根据消费方式提供存储的详细信息。为此，我们引入了两个新的API资源：PersistentVolume和PersistentVolumeClaim。\
PersistentVolume（PV）是群集中由管理员配置的一块存储。它是集群中的资源，就像节点是集群资源一样。
PV是容量插件，如Volumes，但其生命周期独立于使用PV的任何单个pod。此API对象捕获存储实现的详细信息，包括NFS，iSCSI或特定于云提供程序的存储系统。\
PersistentVolumeClaim（PVC）是用户存储的请求。它类似于一个吊舱。
Pod消耗节点资源，PVC消耗PV资源。
Pod可以请求特定级别的资源（CPU和内存）。声明可以请求特定的大小和访问模式（例如，可以一次读/写或多次只读）。\
虽然PersistentVolumeClaims允许用户使用抽象存储资源，但是对于不同的问题，用户通常需要具有不同属性（如性能）的PersistentVolumes。群集管理员需要能够提供各种PersistentVolume，这些PersistentVolume在多种方式上不仅仅是大小和访问模式，而不会让用户了解这些卷的实现方式。对于这些需求，存在StorageClass资源。

Please see the [detailed walkthrough with working
examples](https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/).

Lifecycle of a volume and claim

PV是群集中的资源。 PVC是对这些资源的请求，并且还充当对资源的索赔检查。
PV和PVC之间的相互作用遵循以下生命周期：

Provisioning

可以通过两种方式配置PV：静态或动态

Static

集群管理员创建了许多PV。 它们包含可供群集用户使用的实际存储的详细信息。
它们存在于Kubernetes API中，可供使用

Dynamic

当管理员创建的静态PV都不匹配用户的PersistentVolumeClaim时，群集可能会尝试为PVC专门动态配置卷。
此配置基于StorageClasses：PVC必须请求存储类，管理员必须已创建并配置该类，以便进行动态配置。
请求类""的声明有效地禁用了自己的动态配置。\
要基于存储类启用动态存储配置，群集管理员需要在API服务器上启用DefaultStorageClass准入控制器。
例如，可以通过确保DefaultStorageClass是API服务器组件的\--enable-admission-plugins标志的逗号分隔的有序值列表来完成此操作。
有关API服务器命令行标志的更多信息，请查看kube-apiserver文档。

Binding

用户在动态配置的情况下创建或已经创建了PersistentVolumeClaim，其具有所请求的特定存储量并具有某些访问模式。
主站中的控制回路监视新PVC，找到匹配的PV（如果可能），并将它们绑定在一起。
如果为新PVC动态配置PV，则环路将始终将该PV绑定到PVC。
否则，用户将始终至少得到他们要求的内容，但是音量可能超过所要求的数量。
绑定后，PersistentVolumeClaim绑定是独占的，无论它们如何绑定。
PVC到PV绑定是一对一映射。\
如果不存在匹配的卷，则索赔将无限期地保持未绑定状态。
索赔将在匹配卷可用时受到约束。 例如，配置了许多50Gi
PV的集群与请求100Gi的PVC不匹配。 当100Gi PV添加到集群时，可以绑定PVC。

Using

Pod使用声明作为卷。 群集检查声明以查找绑定卷并为该容器安装该卷。
对于支持多种访问模式的卷，用户在将其声明用作窗格中的卷时指定所需的模式。\
一旦用户具有声明并且该声明被绑定，绑定的PV就属于用户，只要他们需要它。
用户通过在Pod的卷块中包含persistentVolumeClaim来安排Pod并访问其声明的PV。
请参阅下面的语法详细信息

Storage Object in Use Protection

"使用中的存储对象保护"功能的目的是确保不会从系统中删除由绑定到PVC的容器和持久卷（PV）主动使用的持久性卷声明（PVC），因为这可能会导致数据丢失。\
注意：当存在使用PVC的pod对象时，pod正在主动使用PVC。\
如果用户删除了pod正在使用的PVC，则不会立即删除PVC。
PVC的移除被推迟，直到PVC不再被任何容器主动使用，并且如果管理员删除了绑定到PVC的PV，则PV不会立即被移除。
PV推迟被推迟，直到PV不再与PVC结合。

You can see that a PVC is protected when the PVC's status is
**Terminating** and the **Finalizers** list includes
**kubernetes.io/pvc-protection**:

**kubectl describe pvc hostpath**

**Name: hostpath**

**Namespace: default**

**StorageClass: example-hostpath**

**Status: Terminating**

**Volume: **

**Labels: \<none\>**

**Annotations:
volume.beta.kubernetes.io/storage-class=example-hostpath**

**volume.beta.kubernetes.io/storage-provisioner=example.com/hostpath**

**Finalizers: \[kubernetes.io/pvc-protection\]**

**\...**

You can see that a PV is protected when the PV's status is
**Terminating** and the **Finalizers** list includes
**kubernetes.io/pv-protection** too:

**kubectl describe pv task-pv-volume**

**Name: task-pv-volume**

**Labels: type=local**

**Annotations: \<none\>**

**Finalizers: \[kubernetes.io/pv-protection\]**

**StorageClass: standard**

**Status: Available**

**Claim: **

**Reclaim Policy: Delete**

**Access Modes: RWO**

**Capacity: 1Gi**

**Message: **

**Source:**

**Type: HostPath (bare host directory volume)**

**Path: /tmp/data**

**HostPathType: **

**Events: \<none\>**

Reclaiming

当用户完成其卷时，他们可以从API中删除PVC对象，从而允许回收资源。
PersistentVolume的回收策略告诉群集在释放其声明后如何处理该卷。
目前，卷可以保留，回收或删除

Retain

保留回收政策允许手动回收资源。
删除PersistentVolumeClaim时，PersistentVolume仍然存在，并且该卷被视为"已释放"。
但它尚未提供另一项索赔，因为之前的索赔人的数据仍在数量上。
管理员可以使用以下步骤手动回收卷。\
1.删除PersistentVolume。 删除PV后，外部基础架构（例如AWS EBS，GCE
PD，Azure磁盘或Cinder卷）中的关联存储资产仍然存在。\
2.相应地手动清理相关存储资产上的数据。\
3.手动删除关联的存储资产，或者如果要重用相同的存储资产，请使用存储资产定义创建新的PersistentVolume

Delete

对于支持Delete回收策略的卷插件，删除会从Kubernetes中删除PersistentVolume对象，以及外部基础结构中的关联存储资产，例如AWS
EBS，GCE PD，Azure磁盘或Cinder卷。
动态配置的卷继承其StorageClass的回收策略，默认为Delete。
管理员应根据用户的期望配置StorageClass，否则PV必须在创建后进行编辑或修补。See
[Change the Reclaim Policy of a
PersistentVolume](https://kubernetes.io/docs/tasks/administer-cluster/change-pv-reclaim-policy/).

Recycle

警告：不推荐使用Recycle reclaim策略。 相反，推荐的方法是使用动态配置。\
如果基础卷插件支持，则"回收回收"策略会对卷执行基本清理（rm -rf /
thevolume / \*），并使其再次可用于新索引。\
但是，管理员可以使用此处所述的Kubernetes控制器管理器命令行参数配置自定义回收站pod模板。
自定义回收站pod模板必须包含卷规范，如以下示例所示：

**apiVersion: v1**

**kind: Pod**

**metadata:**

**name: pv-recycler**

**namespace: default**

**spec:**

**restartPolicy: Never**

**volumes:**

**- name: vol**

**hostPath:**

**path: /any/path/it/will/be/replaced**

**containers:**

**- name: pv-recycler**

**image: \"k8s.gcr.io/busybox\"**

**command: \[\"/bin/sh\", \"-c\", \"test -e /scrub && rm -rf
/scrub/..?\* /scrub/.\[!.\]\* /scrub/\* && test -z \\\"\$(ls -A
/scrub)\\\" \|\| exit 1\"\]**

**volumeMounts:**

**- name: vol**

**mountPath: /scrub**

但是，卷部分中自定义回收站窗格模板中指定的特定路径将替换为正在回收的卷的特定路径

Expanding Persistent Volumes Claims

**FEATURE STATE:** **Kubernetes v1.11**
[beta](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)

Support for expanding PersistentVolumeClaims (PVCs) is now enabled by
default. You can expand the following types of volumes:

-   gcePersistentDisk

-   awsElasticBlockStore

-   Cinder

-   glusterfs

-   rbd

-   Azure File

-   Azure Disk

-   Portworx

-   FlexVolumes

-   CSI

You can only expand a PVC if its storage class's
**allowVolumeExpansion** field is set to true.

**apiVersion: storage.k8s.io/v1**

**kind: StorageClass**

**metadata:**

**name: gluster-vol-default**

**provisioner: kubernetes.io/glusterfs**

**parameters:**

**resturl: \"http://192.168.10.100:8080\"**

**restuser: \"\"**

**secretNamespace: \"\"**

**secretName: \"\"**

**allowVolumeExpansion: true**

要为PVC请求更大的卷，请编辑PVC对象并指定更大的大小。
这会触发支持底层PersistentVolume的卷的扩展。
永远不会创建新的PersistentVolume来满足声明。 而是调整现有卷的大小。

CSI Volume expansion

**FEATURE STATE:** **Kubernetes v1.14**
[alpha](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)

CSI卷扩展需要启用ExpandCSIVolumes功能门，并且还需要特定的CSI驱动程序来支持卷扩展。
有关更多信息，请参阅特定CSI驱动程序的文档。

Resizing a volume containing a file system

如果文件系统是XFS，Ext3或Ext4，则只能调整包含文件系统的卷的大小。\
当卷包含文件系统时，仅在使用ReadWrite模式下的PersistentVolumeClaim启动新Pod时才调整文件系统的大小。
因此，如果容器或部署正在使用卷并且您要扩展它，则需要在控制器管理器中的云提供程序扩展卷之后删除或重新创建容器。You
can check the status of resize operation by running the **kubectl
describe pvc** command:

**kubectl describe pvc \<pvc\_name\>**

If the **PersistentVolumeClaim** has the status
**FileSystemResizePending**, it is safe to recreate the pod using the
PersistentVolumeClaim.

FlexVolumes allow resize if the driver is set with the
**RequiresFSResize** capability to true. The FlexVolume can be resized
on pod restart.

**FEATURE STATE:** **Kubernetes v1.11**
[alpha](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)

Resizing an in-use PersistentVolumeClaim

扩展使用中的PVC是一种alpha功能。要使用它，请启用ExpandInUsePersistentVolumes功能门。在这种情况下，您无需删除并重新创建使用现有PVC的Pod或部署。任何正在使用的PVC在其文件系统扩展后自动可用于其Pod。此功能对Pod或部署未使用的PVC没有影响。您必须创建一个使用PVC的Pod，然后才能完成扩展。\
在版本1.13中添加了扩展FlexVolumes的使用中的PVC。要启用此功能，请使用ExpandInUsePersistentVolumes和ExpandPersistentVolumes功能门。默认情况下，ExpandPersistentVolumes功能门已启用。如果设置了ExpandInUsePersistentVolumes，则可以在不重新启动pod的情况下在线调整FlexVolume的大小。\
注意：仅当底层驱动程序支持调整大小时，才可以调整FlexVolume。\
注意：扩展EBS卷是一项耗时的操作。此外，每6小时有一次修改的每卷配额。

Types of Persistent Volumes

**PersistentVolume** types are implemented as plugins. Kubernetes
currently supports the following plugins:

-   GCEPersistentDisk

-   AWSElasticBlockStore

-   AzureFile

-   AzureDisk

-   FC (Fibre Channel)

-   Flexvolume

-   Flocker

-   NFS

-   iSCSI

-   RBD (Ceph Block Device)

-   CephFS

-   Cinder (OpenStack block storage)

-   Glusterfs

-   VsphereVolume

-   Quobyte Volumes

-   HostPath (Single node testing only -- local storage is not supported
    in any way and WILL NOT WORK in a multi-node cluster)

-   Portworx Volumes

-   ScaleIO Volumes

-   StorageOS

    Persistent Volumes

Each PV contains a spec and status, which is the specification and
status of the volume.

**apiVersion: v1**

**kind: PersistentVolume**

**metadata:**

**name: pv0003**

**spec:**

**capacity:**

**storage: 5Gi**

**volumeMode: Filesystem**

**accessModes:**

**- ReadWriteOnce**

**persistentVolumeReclaimPolicy: Recycle**

**storageClassName: slow**

**mountOptions:**

**- hard**

**- nfsvers=4.1**

**nfs:**

**path: /tmp**

**server: 172.17.0.2**

Capacity

通常，PV将具有特定的存储容量。 这是使用PV的capacity属性设置的。
请参阅Kubernetes资源模型以了解容量预期的单位。\
目前，存储大小是唯一可以设置或请求的资源。
未来属性可能包括IOPS，吞吐量等。

Volume Mode

**FEATURE STATE:** **Kubernetes v1.13**
[beta](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)

在Kubernetes 1.9之前，所有卷插件都在持久卷上创建了一个文件系统。
现在，您可以将volumeMode的值设置为block以使用原始块设备，或将文件系统设置为使用文件系统。
如果省略该值，则filesystem是默认值。 这是一个可选的API参数。

Access Modes

PersistentVolume可以以资源提供者支持的任何方式安装在主机上。
如下表所示，提供程序将具有不同的功能，并且每个PV的访问模式都设置为该特定卷支持的特定模式。
例如，NFS可以支持多个读/写客户端，但是特定的NFS
PV可能以只读方式导出到服务器上。
每个PV都有自己的一组访问模式，用于描述特定PV的功能。

The access modes are:

-   ReadWriteOnce -- the volume can be mounted as read-write by a single
    node

-   ReadOnlyMany -- the volume can be mounted read-only by many nodes

-   ReadWriteMany -- the volume can be mounted as read-write by many
    nodes

In the CLI, the access modes are abbreviated to:

-   RWO - ReadWriteOnce

-   ROX - ReadOnlyMany

-   RWX - ReadWriteMany

**Important!** A volume can only be mounted using one access mode at a
time, even if it supports many. For example, a GCEPersistentDisk can be
mounted as ReadWriteOnce by a single node or ReadOnlyMany by many nodes,
but not at the same time.

  Volume Plugin          ReadWriteOnce   ReadOnlyMany   ReadWriteMany
  ---------------------- --------------- -------------- -------------------------------------
  AWSElasticBlockStore   ✓               \-             \-
  AzureFile              ✓               ✓              ✓
  AzureDisk              ✓               \-             \-
  CephFS                 ✓               ✓              ✓
  Cinder                 ✓               \-             \-
  FC                     ✓               ✓              \-
  Flexvolume             ✓               ✓              depends on the driver
  Flocker                ✓               \-             \-
  GCEPersistentDisk      ✓               ✓              \-
  Glusterfs              ✓               ✓              ✓
  HostPath               ✓               \-             \-
  iSCSI                  ✓               ✓              \-
  Quobyte                ✓               ✓              ✓
  NFS                    ✓               ✓              ✓
  RBD                    ✓               ✓              \-
  VsphereVolume          ✓               \-             \- (works when pods are collocated)
  PortworxVolume         ✓               \-             ✓
  ScaleIO                ✓               ✓              \-
  StorageOS              ✓               \-             \-

Class

PV可以有一个类，通过将storageClassName属性设置为StorageClass的名称来指定该类。
特定类的PV只能绑定到请求该类的PVC。
没有storageClassName的PV没有类，只能绑定到不请求特定类的PVC。\
过去，使用注释volume.beta.kubernetes.io/storage-class而不是storageClassName属性。
这个注释仍然有效，但在未来的Kubernetes版本中它将被完全弃用。

Reclaim Policy

Current reclaim policies are:

-   Retain -- manual reclamation

-   Recycle -- basic scrub (**rm -rf /thevolume/\***)

-   Delete -- associated storage asset such as AWS EBS, GCE PD, Azure
    Disk, or OpenStack Cinder volume is deleted

Currently, only NFS and HostPath support recycling. AWS EBS, GCE PD,
Azure Disk, and Cinder volumes support deletion.

Mount Options

Kubernetes管理员可以在节点上安装永久卷时指定其他安装选项。\
注意：并非所有Persistent卷类型都支持装入选项

The following volume types support mount options:

-   AWSElasticBlockStore

-   AzureDisk

-   AzureFile

-   CephFS

-   Cinder (OpenStack block storage)

-   GCEPersistentDisk

-   Glusterfs

-   NFS

-   Quobyte Volumes

-   RBD (Ceph Block Device)

-   StorageOS

-   VsphereVolume

-   iSCSI

挂载选项未经过验证，因此如果一个无效，挂载将会失败。\
过去，使用注释volume.beta.kubernetes.io/mount-options而不是mountOptions属性。
这个注释仍然有效，但在未来的Kubernetes版本中它将被完全弃用。

Node Affinity

注意：对于大多数卷类型，您无需设置此字段。 它会自动填充AWS EBS，GCE
PD和Azure磁盘卷块类型。 您需要为本地卷明确设置它。\
PV可以指定节点关联，以定义限制可以访问此卷的节点的约束。
使用PV的Pod将仅调度到由节点关联选择的节点

Phase

A volume will be in one of the following phases:

-   Available -- a free resource that is not yet bound to a claim

-   Bound -- the volume is bound to a claim

-   Released -- the claim has been deleted, but the resource is not yet
    reclaimed by the cluster

-   Failed -- the volume has failed its automatic reclamation

The CLI will show the name of the PVC bound to the PV.

PersistentVolumeClaims

Each PVC contains a spec and status, which is the specification and
status of the claim.

**apiVersion: v1**

**kind: PersistentVolumeClaim**

**metadata:**

**name: myclaim**

**spec:**

**accessModes:**

**- ReadWriteOnce**

**volumeMode: Filesystem**

**resources:**

**requests:**

**storage: 8Gi**

**storageClassName: slow**

**selector:**

**matchLabels:**

**release: \"stable\"**

**matchExpressions:**

**- {key: environment, operator: In, values: \[dev\]}**

Access Modes

在请求具有特定访问模式的存储时，声明使用与卷相同的约定

Volume Modes

声明使用与卷相同的约定来指示卷的消耗作为文件系统或块设备

Resources

与pod一样，声明可以请求特定数量的资源。 在这种情况下，请求是用于存储。
相同的资源模型适用于卷和声明。

Selector

-   声明可以指定标签选择器以进一步过滤卷集。
    只有标签与选择器匹配的卷才能绑定到声明。 选择器可以包含两个字段：

-   **matchLabels** - the volume must have a label with this value

-   **matchExpressions** - a list of requirements made by specifying
    key, list of values, and operator that relates the key and values.
    Valid operators include In, NotIn, Exists, and DoesNotExist.

All of the requirements, from both **matchLabels** and
**matchExpressions** are ANDed together -- they must all be satisfied in
order to match.

Class

声明可以通过使用属性storageClassName指定StorageClass的名称来请求特定类。只有所请求类的PV（具有与PVC相同的storageClassName的PV）才能绑定到PVC。\
PVC不一定要求上课。其storageClassName设置为""的PVC始终被解释为请求没有类的PV，因此它只能绑定到没有类的PV（没有注释或一组等于""）。没有storageClassName的PVC不完全相同，群集会区别对待，具体取决于DefaultStorageClass准入插件是否已打开。\
•如果启用了准入插件，管理员可以指定默认的StorageClass。所有没有storageClassName的PVC只能绑定到该默认值的PV。通过在StorageClass对象中将注释storageclass.kubernetes.io/is-default-class设置为"true"来指定默认的StorageClass。如果管理员未指定默认值，则群集将响应PVC创建，就像关闭了准入插件一样。如果指定了多个默认值，则允许插件禁止创建所有PVC。\
•如果关闭了准入插件，则没有默认StorageClass的概念。所有没有storageClassName的PVC只能绑定到没有类的PV。在这种情况下，没有storageClassName的PVC的处理方式与其storageClassName设置为""的PVC的处理方式相同。\
根据安装方法，安装期间可以通过插件管理器将默认StorageClass部署到Kubernetes集群。\
当PVC指定选择器以及请求StorageClass时，需求一起进行AND运算：只有所请求类的PV和所请求的标签可以绑定到PVC。\
注意：目前，具有非空选择器的PVC无法为其动态配置PV。\
过去，使用注释volume.beta.kubernetes.io/storage-class而不是storageClassName属性。此注释仍然有效，但在将来的Kubernetes版本中不会支持它。

Claims As Volumes

Pod使用声明作为卷来访问存储。
声明必须与使用声明的pod存在于同一名称空间中。
群集在pod的命名空间中查找声明，并使用它来获取支持声明的PersistentVolume。
然后将卷安装到主机并进入容器。

**apiVersion: v1**

**kind: Pod**

**metadata:**

**name: mypod**

**spec:**

**containers:**

**- name: myfrontend**

**image: nginx**

**volumeMounts:**

**- mountPath: \"/var/www/html\"**

**name: mypd**

**volumes:**

**- name: mypd**

**persistentVolumeClaim:**

**claimName: myclaim**

A Note on Namespaces

PersistentVolumes绑定是独占的，并且由于PersistentVolumeClaims是命名空间对象，因此只能在一个命名空间内安装具有"多个"模式（ROX，RWX）的声明。

Raw Block Volume Support

**FEATURE STATE:** **Kubernetes v1.13**
[beta](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)

The following volume plugins support raw block volumes, including
dynamic provisioning where applicable.

-   AWSElasticBlockStore

-   AzureDisk

-   FC (Fibre Channel)

-   GCEPersistentDisk

-   iSCSI

-   Local volume

-   RBD (Ceph Block Device)

-   VsphereVolume (alpha)

**Note:** Only FC and iSCSI volumes supported raw block volumes in
Kubernetes 1.9. Support for the additional plugins was added in 1.10.

Persistent Volumes using a Raw Block Volume

**apiVersion: v1**

**kind: PersistentVolume**

**metadata:**

**name: block-pv**

**spec:**

**capacity:**

**storage: 10Gi**

**accessModes:**

**- ReadWriteOnce**

**volumeMode: Block**

**persistentVolumeReclaimPolicy: Retain**

**fc:**

**targetWWNs: \[\"50060e801049cfd1\"\]**

**lun: 0**

**readOnly: false**

Persistent Volume Claim requesting a Raw Block Volume

**apiVersion: v1**

**kind: PersistentVolumeClaim**

**metadata:**

**name: block-pvc**

**spec:**

**accessModes:**

**- ReadWriteOnce**

**volumeMode: Block**

**resources:**

**requests:**

**storage: 10Gi**

Pod specification adding Raw Block Device path in container

**apiVersion: v1**

**kind: Pod**

**metadata:**

**name: pod-with-block-volume**

**spec:**

**containers:**

**- name: fc-container**

**image: fedora:26**

**command: \[\"/bin/sh\", \"-c\"\]**

**args: \[ \"tail -f /dev/null\" \]**

**volumeDevices:**

**- name: data**

**devicePath: /dev/xvda**

**volumes:**

**- name: data**

**persistentVolumeClaim:**

**claimName: block-pvc**

**Note:** When adding a raw block device for a Pod, we specify the
device path in the container instead of a mount path.

Binding Block Volumes

如果用户通过使用PersistentVolumeClaim规范中的volumeMode字段指示这一点来请求原始块卷，则绑定规则与之前未将此模式视为规范一部分的版本略有不同。
列出的是用户和管理员可能为请求原始块设备指定的可能组合的表。
该表指示在给定组合的情况下是否绑定卷：静态配置卷的卷绑定矩阵：

  PV volumeMode   PVC volumeMode   Result
  --------------- ---------------- ---------
  unspecified     unspecified      BIND
  unspecified     Block            NO BIND
  unspecified     Filesystem       BIND
  Block           unspecified      NO BIND
  Block           Block            BIND
  Block           Filesystem       NO BIND
  Filesystem      Filesystem       BIND
  Filesystem      Block            NO BIND
  Filesystem      unspecified      BIND

注意：alpha版本仅支持静态配置的卷。
在使用原始块设备时，管理员应注意考虑这些值。

Volume Snapshot and Restore Volume from Snapshot Support

**FEATURE STATE:** **Kubernetes v1.12**
[alpha](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)

添加了卷快照功能以仅支持CSI卷插件。 有关详细信息，请参阅卷快照。\
要启用从卷快照数据源还原卷的支持，请在apiserver和controller-manager上启用VolumeSnapshotDataSource功能门。

Create Persistent Volume Claim from Volume Snapshot

**apiVersion: v1**

**kind: PersistentVolumeClaim**

**metadata:**

**name: restore-pvc**

**spec:**

**storageClassName: csi-hostpath-sc**

**dataSource:**

**name: new-snapshot-test**

**kind: VolumeSnapshot**

**apiGroup: snapshot.storage.k8s.io**

**accessModes:**

**- ReadWriteOnce**

**resources:**

**requests:**

**storage: 10Gi**

Writing Portable Configuration

如果您正在编写在各种群集上运行并需要持久存储的配置模板或示例，我们建议您使用以下模式：\
•在您的配置包中包含PersistentVolumeClaim对象（以及Deployments，ConfigMaps等）。\
•不要在配置中包含PersistentVolume对象，因为实例化配置的用户可能没有创建PersistentVolumes的权限。\
•为用户提供在实例化模板时提供存储类名称的选项。\
•如果用户提供存储类名称，请将该值放入persistentVolumeClaim.storageClassName字段中。如果群集已由管理员启用StorageClasses，这将导致PVC与正确的存储类匹配。\
•如果用户未提供存储类名称，请将persistentVolumeClaim.storageClassName字段保留为nil。\
•这将导致为具有群集中的默认StorageClass的用户自动配置PV。许多群集环境都安装了默认的StorageClass，或者管理员可以创建自己的默认StorageClass。\
•在您的工具中，请注意一段时间后未受约束的PVC并将其显示给用户，因为这可能表明群集没有动态存储支持（在这种情况下，用户应创建匹配的PV）或集群没有存储系统（在这种情况下，用户无法部署需要PVC的配置）。

Volume Snapshots
================

This document describes the current state of **VolumeSnapshots** in
Kubernetes. Familiarity with [persistent
volumes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)
is suggested.

-   [**Introduction**](https://kubernetes.io/docs/concepts/storage/volume-snapshots/#introduction)

-   [**Lifecycle of a volume snapshot and volume snapshot
    content**](https://kubernetes.io/docs/concepts/storage/volume-snapshots/#lifecycle-of-a-volume-snapshot-and-volume-snapshot-content)

-   [**Volume Snapshot
    Contents**](https://kubernetes.io/docs/concepts/storage/volume-snapshots/#volume-snapshot-contents)

-   [**VolumeSnapshots**](https://kubernetes.io/docs/concepts/storage/volume-snapshots/#volumesnapshots)

Introduction
------------

与API资源PersistentVolume和PersistentVolumeClaim用于为用户和管理员配置卷的方式类似，提供VolumeSnapshotContent和VolumeSnapshot
API资源以为用户和管理员创建卷快照。\
VolumeSnapshotContent是从群集中已由管理员配置的卷中获取的快照。它是集群中的资源，就像PersistentVolume是集群资源一样。\
VolumeSnapshot是用户对卷的快照请求。它类似于PersistentVolumeClaim。\
虽然VolumeSnapshots允许用户使用抽象存储资源，但是集群管理员需要能够提供各种VolumeSnapshotContents，而不会让用户了解应如何配置这些卷快照的详细信息。对于这些需求，有VolumeSnapshotClass资源。\
使用此功能时，用户需要注意以下事项：\
•API对象VolumeSnapshot，VolumeSnapshotContent和VolumeSnapshotClass是CRD，不是核心API的一部分。\
•VolumeSnapshot支持仅适用于CSI驱动程序。\
•作为部署过程的一部分，Kubernetes团队为快照控制器提供了一个名为external-snapshotter的sidecar帮助程序容器。它监视VolumeSnapshot对象并针对CSI端点触发CreateSnapshot和DeleteSnapshot操作。\
•CSI驱动程序可能已实现或未实现卷快照功能。为卷快照提供支持的CSI驱动程序可能会使用外部快照程序。\
•支持卷快照的CSI驱动程序将自动安装为卷快照定义的CRD

Lifecycle of a volume snapshot and volume snapshot content
----------------------------------------------------------

**VolumeSnapshotContents** are resources in the cluster.
**VolumeSnapshots** are requests for those resources. The interaction
between **VolumeSnapshotContents** and **VolumeSnapshots** follow this
lifecycle:

### Provisioning Volume Snapshot

There are two ways snapshots may be provisioned: statically or
dynamically.

#### Static

集群管理员创建许多VolumeSnapshotContents。
它们包含可供群集用户使用的实际存储的详细信息。 它们存在于Kubernetes
API中，可供使用

#### Dynamic

当管理员创建的静态VolumeSnapshotContents都不匹配用户的VolumeSnapshot时，群集可能会尝试为VolumeSnapshot对象专门动态配置卷快照。
此配置基于VolumeSnapshotClasses：VolumeSnapshot必须请求卷快照类，并且管理员必须已创建并配置该类，以便进行动态配置

### Binding

用户在动态配置的情况下创建或已经创建了具有所请求的特定存储量和某些访问模式的VolumeSnapshot。
控制循环监视新的VolumeSnapshots，找到匹配的VolumeSnapshotContent（如果可能），并将它们绑定在一起。
如果为新的VolumeSnapshot动态调配VolumeSnapshotContent，则循环将始终将VolumeSnapshotContent绑定到VolumeSnapshot。
绑定后，VolumeSnapshot绑定是独占的，无论它们如何绑定。
VolumeSnapshot与VolumeSnapshotContent绑定是一对一映射。\
如果不存在匹配的VolumeSnapshotContent，VolumeSnapshots将无限期地保持未绑定状态。
VolumeSnapshots将在匹配的VolumeSnapshotContents可用时绑定。

### Delete

删除将从Kubernetes
API中删除VolumeSnapshotContent对象，以及外部基础结构中的关联存储资产

Volume Snapshot Contents
------------------------

Each VolumeSnapshotContent contains a spec, which is the specification
of the volume snapshot.

**apiVersion: snapshot.storage.k8s.io/v1alpha1**

**kind: VolumeSnapshotContent**

**metadata:**

**name: new-snapshot-content-test**

**spec:**

**snapshotClassName: csi-hostpath-snapclass**

**source:**

**name: pvc-test**

**kind: PersistentVolumeClaim**

**volumeSnapshotSource:**

**csiVolumeSnapshotSource:**

**creationTime: 1535478900692119403**

**driver: csi-hostpath**

**restoreSize: 10Gi**

**snapshotHandle: 7bdd0de3-aaeb-11e8-9aae-0242ac110002**

### Class

VolumeSnapshotContent可以有一个类，通过将snapshotClassName属性设置为VolumeSnapshotClass的名称来指定该类。
特定类的VolumeSnapshotContent只能绑定到请求该类的VolumeSnapshots。
没有snapshotClassName的VolumeSnapshotContent没有类，只能绑定到不请求特定类的VolumeSnapshots

VolumeSnapshots
---------------

Each VolumeSnapshot contains a spec and a status, which is the
specification and status of the volume snapshot.

**apiVersion: snapshot.storage.k8s.io/v1alpha1**

**kind: VolumeSnapshot**

**metadata:**

**name: new-snapshot-test**

**spec:**

**snapshotClassName: csi-hostpath-snapclass**

**source:**

**name: pvc-test**

**kind: PersistentVolumeClaim**

### Class

卷快照可以通过使用属性snapshotClassName指定VolumeSnapshotClass的名称来请求特定类。
只有所请求类的VolumeSnapshotContents（具有与VolumeSnapshot相同的snapshotClassName）可以绑定到VolumeSnapshot

Storage Classes

This document describes the concept of a StorageClass in Kubernetes.
Familiarity with
[volumes](https://kubernetes.io/docs/concepts/storage/volumes/) and
[persistent
volumes](https://kubernetes.io/docs/concepts/storage/persistent-volumes)
is suggested.

-   [**Introduction**](https://kubernetes.io/docs/concepts/storage/storage-classes/#introduction)

-   [**The StorageClass
    Resource**](https://kubernetes.io/docs/concepts/storage/storage-classes/#the-storageclass-resource)

-   [**Parameters**](https://kubernetes.io/docs/concepts/storage/storage-classes/#parameters)

    Introduction

StorageClass为管理员提供了一种描述他们提供的"存储类"的方法。
不同的类可能映射到服务质量级别，或备份策略，或者由集群管理员确定的任意策略。
Kubernetes本身对于什么类代表是不受任何影响的。
这个概念有时在其他存储系统中称为"配置文件"。

The StorageClass Resource

每个StorageClass都包含字段provisioner，parameters和reclaimPolicy，当需要动态配置属于该类的PersistentVolume时使用这些字段。\
StorageClass对象的名称很重要，是用户可以请求特定类的方式。
管理员在首次创建StorageClass对象时设置类的名称和其他参数，并且在创建对象后无法更新这些对象。

Administrators can specify a default **StorageClass** just for PVCs that
don't request any particular class to bind to: see the
[**[PersistentVolumeClaim]{.underline}**](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#class-1)
section for details.

**apiVersion: storage.k8s.io/v1**

**kind: StorageClass**

**metadata:**

**name: standard**

**provisioner: kubernetes.io/aws-ebs**

**parameters:**

**type: gp2**

**reclaimPolicy: Retain**

**mountOptions:**

**- debug**

**volumeBindingMode: Immediate**

Provisioner

Storage classes have a provisioner that determines what volume plugin is
used for provisioning PVs. This field must be specified.

  Volume Plugin          Internal Provisioner   Config Example
  ---------------------- ---------------------- ---------------------------------------------------------------------------------------------------
  AWSElasticBlockStore   ✓                      [AWS EBS](https://kubernetes.io/docs/concepts/storage/storage-classes/#aws-ebs)
  AzureFile              ✓                      [Azure File](https://kubernetes.io/docs/concepts/storage/storage-classes/#azure-file)
  AzureDisk              ✓                      [Azure Disk](https://kubernetes.io/docs/concepts/storage/storage-classes/#azure-disk)
  CephFS                 \-                     \-
  Cinder                 ✓                      [OpenStack Cinder](https://kubernetes.io/docs/concepts/storage/storage-classes/#openstack-cinder)
  FC                     \-                     \-
  Flexvolume             \-                     \-
  Flocker                ✓                      \-
  GCEPersistentDisk      ✓                      [GCE PD](https://kubernetes.io/docs/concepts/storage/storage-classes/#gce-pd)
  Glusterfs              ✓                      [Glusterfs](https://kubernetes.io/docs/concepts/storage/storage-classes/#glusterfs)
  iSCSI                  \-                     \-
  Quobyte                ✓                      [Quobyte](https://kubernetes.io/docs/concepts/storage/storage-classes/#quobyte)
  NFS                    \-                     \-
  RBD                    ✓                      [Ceph RBD](https://kubernetes.io/docs/concepts/storage/storage-classes/#ceph-rbd)
  VsphereVolume          ✓                      [vSphere](https://kubernetes.io/docs/concepts/storage/storage-classes/#vsphere)
  PortworxVolume         ✓                      [Portworx Volume](https://kubernetes.io/docs/concepts/storage/storage-classes/#portworx-volume)
  ScaleIO                ✓                      [ScaleIO](https://kubernetes.io/docs/concepts/storage/storage-classes/#scaleio)
  StorageOS              ✓                      [StorageOS](https://kubernetes.io/docs/concepts/storage/storage-classes/#storageos)
  Local                  \-                     [Local](https://kubernetes.io/docs/concepts/storage/storage-classes/#local)

您不仅限于指定此处列出的"内部"配置程序（其名称以"kubernetes.io"为前缀并与Kubernetes一起提供）。
您还可以运行和指定外部供应商，这些供应商是遵循Kubernetes定义的规范的独立程序。
外部供应商的作者可以完全自行决定代码的存在位置，供应商的运输方式，运行方式，使用的卷插件（包括Flex）等。存储库kubernetes-incubator
/ external-storage包含库
用于编写实现大量规范的外部供应商以及各种社区维护的外部供应商。\
例如，NFS不提供内部配置程序，但可以使用外部配置程序。
一些外部供应商列在存储库kubernetes-incubator / external-storage下。
还有第三方存储供应商提供自己的外部供应商的情况。

Reclaim Policy

由存储类动态创建的持久卷将具有在类的reclaimPolicy字段中指定的回收策略，该策略可以是Delete或Retain。
如果在创建StorageClass对象时未指定reclaimPolicy，则默认为Delete。\
手动创建并通过存储类管理的持久卷将具有在创建时分配的任何回收策略。

Mount Options

由存储类动态创建的持久卷将具有在类的mountOptions字段中指定的挂载选项。\
如果卷插件不支持装入选项但指定了装入选项，则配置将失败。
挂载选项未在类或PV上验证，因此如果一个无效，则PV的挂载将失败

Volume Binding Mode

volumeBindingMode字段控制何时应进行卷绑定和动态配置。\
默认情况下，立即模式表示一旦创建了PersistentVolumeClaim，就会发生卷绑定和动态配置。
对于受拓扑约束且无法从群集中的所有节点全局访问的存储后端，将在不知道Pod的调度要求的情况下绑定或配置PersistentVolumes。
这可能导致不可调度的Pod。\
集群管理员可以通过指定WaitForFirstConsumer模式来解决此问题，该模式将延迟绑定和配置PersistentVolume，直到创建使用PersistentVolumeClaim的Pod。
将根据Pod的调度约束指定的拓扑选择或配置PersistentVolumes。
这些包括但不限于资源需求，节点选择器，pod亲和力和反亲和力，以及污点和容忍度。

The following plugins support **WaitForFirstConsumer** with dynamic
provisioning:

-   [AWSElasticBlockStore](https://kubernetes.io/docs/concepts/storage/storage-classes/#aws-ebs)

-   [GCEPersistentDisk](https://kubernetes.io/docs/concepts/storage/storage-classes/#gce-pd)

-   [AzureDisk](https://kubernetes.io/docs/concepts/storage/storage-classes/#azure-disk)

The following plugins support **WaitForFirstConsumer** with pre-created
PersistentVolume binding:

-   All of the above

-   [Local](https://kubernetes.io/docs/concepts/storage/storage-classes/#local)

**FEATURE STATE:** **Kubernetes 1.14**
[beta](https://kubernetes.io/docs/concepts/storage/storage-classes/)

动态配置和预先创建的PV也支持CSI卷，但您需要查看特定CSI驱动程序的文档，以查看其支持的拓扑关键字和示例。
必须启用CSINodeInfo功能门。

Allowed Topologies

当集群运营商指定WaitForFirstConsumer卷绑定模式时，在大多数情况下不再需要将配置限制为特定拓扑。
但是，如果仍然需要，可以指定allowedTopologies。\
此示例演示如何将配置卷的拓扑限制为特定区域，并且应该用作替换支持的插件的区域和区域参数。

**apiVersion: storage.k8s.io/v1**

**kind: StorageClass**

**metadata:**

**name: standard**

**provisioner: kubernetes.io/gce-pd**

**parameters:**

**type: pd-standard**

**volumeBindingMode: WaitForFirstConsumer**

**allowedTopologies:**

**- matchLabelExpressions:**

**- key: failure-domain.beta.kubernetes.io/zone**

**values:**

**- us-central1-a**

**- us-central1-b**

Parameters

存储类具有描述属于存储类的卷的参数。 取决于供应者，可以接受不同的参数。
例如，参数类型的值io1和参数iopsPerGB特定于EBS。
省略参数时，会使用某些默认值。

AWS EBS

**apiVersion: storage.k8s.io/v1**

**kind: StorageClass**

**metadata:**

**name: slow**

**provisioner: kubernetes.io/aws-ebs**

**parameters:**

**type: io1**

**iopsPerGB: \"10\"**

**fsType: ext4**

-   **type**: **io1**, **gp2**, **sc1**, **st1**. See [AWS
    docs](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EBSVolumeTypes.html)
    for details. Default: **gp2**.

-   **zone** (Deprecated): AWS zone. If neither **zone** nor **zones**
    is specified, volumes are generally round-robin-ed across all active
    zones where Kubernetes cluster has a node. **zone** and **zones**
    parameters must not be used at the same time.

（•zone（已弃用）：AWS区域。
如果既未指定区域也未指定区域，则卷通常在Kubernetes群集具有节点的所有活动区域中进行循环。
区域和区域参数不得同时使用。）

-   **zones** (Deprecated): A comma separated list of AWS zone(s). If
    neither **zone** nor **zones** is specified, volumes are generally
    round-robin-ed across all active zones where Kubernetes cluster has
    a node. **zone** and **zones** parameters must not be used at the
    same time.

**（**•zones（已弃用）：以逗号分隔的AWS区域列表。
如果既未指定区域也未指定区域，则卷通常在Kubernetes群集具有节点的所有活动区域中进行循环。
区域和区域参数不得同时使用**）**

-   **iopsPerGB**: only for **io1** volumes. I/O operations per second
    per GiB. AWS volume plugin multiplies this with size of requested
    volume to compute IOPS of the volume and caps it at 20 000 IOPS
    (maximum supported by AWS, see [AWS
    docs](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EBSVolumeTypes.html).
    A string is expected here, i.e. **\"10\"**, not **10**.

-   **fsType**: fsType that is supported by kubernetes. Default:
    **\"ext4\"**.

-   **encrypted**: denotes whether the EBS volume should be encrypted or
    not. Valid values are **\"true\"** or **\"false\"**. A string is
    expected here, i.e. **\"true\"**, not **true**.

-   **kmsKeyId**: optional. The full Amazon Resource Name of the key to
    use when encrypting the volume. If none is supplied but
    **encrypted** is true, a key is generated by AWS. See AWS docs for
    valid ARN value.

**(**•kmsKeyId：可选。 加密卷时使用的密钥的完整Amazon资源名称。
如果未提供但加密为真，则AWS生成密钥。
请参阅AWS文档以获取有效的ARN值。**)**

**Note:** **zone** and **zones** parameters are deprecated and replaced
with
[allowedTopologies](https://kubernetes.io/docs/concepts/storage/storage-classes/#allowed-topologies)

GCE PD

**apiVersion: storage.k8s.io/v1**

**kind: StorageClass**

**metadata:**

**name: slow**

**provisioner: kubernetes.io/gce-pd**

**parameters:**

**type: pd-standard**

**replication-type: none**

-   **type**: **pd-standard** or **pd-ssd**. Default: **pd-standard**

-   **zone** (Deprecated): GCE zone. If neither **zone** nor **zones**
    is specified, volumes are generally round-robin-ed across all active
    zones where Kubernetes cluster has a node. **zone** and **zones**
    parameters must not be used at the same time.

**(**•zone（已弃用）：GCE区域。
如果既未指定区域也未指定区域，则卷通常在Kubernetes群集具有节点的所有活动区域中进行循环。
区域和区域参数不得同时使用。**)**

-   **zones** (Deprecated): A comma separated list of GCE zone(s). If
    neither **zone** nor **zones** is specified, volumes are generally
    round-robin-ed across all active zones where Kubernetes cluster has
    a node. **zone** and **zones** parameters must not be used at the
    same time.

**(**•zones（已弃用）：以逗号分隔的GCE区域列表。
如果既未指定区域也未指定区域，则卷通常在Kubernetes群集具有节点的所有活动区域中进行循环。
区域和区域参数不得同时使用。**)**

-   **replication-type**: **none** or **regional-pd**. Default:
    **none**.

If **replication-type** is set to **none**, a regular (zonal) PD will be
provisioned.

If **replication-type** is set to **regional-pd**, a [Regional
Persistent Disk](https://cloud.google.com/compute/docs/disks/#repds)
will be provisioned. In this case, users must use **zones** instead of
**zone** to specify the desired replication zones. If exactly two zones
are specified, the Regional PD will be provisioned in those zones. If
more than two zones are specified, Kubernetes will arbitrarily choose
among the specified zones. If the **zones** parameter is omitted,
Kubernetes will arbitrarily choose among zones managed by the cluster.

(如果replication-type设置为regional-pd，则将配置区域永久磁盘。
在这种情况下，用户必须使用区域而不是区域来指定所需的复制区域。
如果指定了两个区域，则将在这些区域中配置区域PD。
如果指定了两个以上的区域，Kubernetes将在指定的区域中任意选择。
如果省略了zones参数，Kubernetes将在群集管理的区域之间任意选择。)

**Note:** **zone** and **zones** parameters are deprecated and replaced
with
[allowedTopologies](https://kubernetes.io/docs/concepts/storage/storage-classes/#allowed-topologies)

Glusterfs

**apiVersion: storage.k8s.io/v1**

**kind: StorageClass**

**metadata:**

**name: slow**

**provisioner: kubernetes.io/glusterfs**

**parameters:**

**resturl: \"http://127.0.0.1:8081\"**

**clusterid: \"630372ccdc720a92c681fb928f27b53f\"**

**restauthenabled: \"true\"**

**restuser: \"admin\"**

**secretNamespace: \"default\"**

**secretName: \"heketi-secret\"**

**gidMin: \"40000\"**

**gidMax: \"50000\"**

**volumetype: \"replicate:3\"**

-   **resturl**: Gluster REST service/Heketi service url which provision
    gluster volumes on demand. The general format(格式) should be
    **IPaddress:Port** and this is a mandatory parameter（强制参数） for
    GlusterFS dynamic provisioner. If Heketi service is exposed as a
    routable service(可路由的服务) in openshift/kubernetes setup, this
    can have a format similar to
    **http://heketi-storage-project.cloudapps.mystorage.com** where the
    fqdn is a resolvable Heketi service url.

-   **restauthenabled** : Gluster REST service authentication
    boolean（认证布尔值） that enables authentication to the REST
    server. If this value is **\"true\"**, **restuser** and
    **restuserkey** or **secretNamespace** + **secretName** have to be
    filled. This option is deprecated(弃用), authentication is enabled
    when any of **restuser**, **restuserkey**, **secretName** or
    **secretNamespace** is specified.

-   **restuser** : Gluster REST service/Heketi user who has access to
    create volumes in the Gluster Trusted Pool.

-   **restuserkey** : Gluster REST service/Heketi user's password which
    will be used for authentication to the REST server. This parameter
    is deprecated in favor of **secretNamespace** + **secretName**.

-   **secretNamespace**, **secretName** : Identification of Secret
    instance that contains user password to use when talking to Gluster
    REST service. These parameters are optional, empty password will be
    used when both **secretNamespace** and **secretName** are omitted.
    The provided secret must have type **\"kubernetes.io/glusterfs\"**,
    e.g. created in this way:

**(**标识包含与Gluster REST服务通信时使用的用户密码的Secret实例。
这些参数是可选的，当省略secretNamespace和secretName时，将使用空密码。
提供的秘密必须具有类型"kubernetes.io/glusterfs"，例如
以这种方式创建：**)**

-   **kubectl create secret generic heketi-secret \\**

-   **\--type=\"kubernetes.io/glusterfs\"
    \--from-literal=key=\'opensesame\' \\**

-   **\--namespace=default**

Example of a secret can be found in
[glusterfs-provisioning-secret.yaml](https://github.com/kubernetes/examples/tree/master/staging/persistent-volume-provisioning/glusterfs/glusterfs-secret.yaml).

-   **clusterid**: **630372ccdc720a92c681fb928f27b53f** is the ID of the
    cluster which will be used by Heketi when provisioning the volume.
    It can also be a list of clusterids, for example:
    **\"8452344e2becec931ece4e33c4674e4e,42982310de6c63381718ccfa6d8cf397\"**.
    This is an optional parameter.

-   **gidMin**, **gidMax** : The minimum and maximum value of GID range
    for the storage class. A unique value (GID) in this range (
    gidMin-gidMax ) will be used for dynamically provisioned volumes.
    These are optional values. If not specified, the volume will be
    provisioned with a value between 2000-2147483647 which are defaults
    for gidMin and gidMax respectively.

**(**存储类的GID范围的最小值和最大值。
此范围内的唯一值（GID）（gidMin-gidMax）将用于动态调配的卷。
这些是可选值。
如果未指定，则将为卷配置介于2000-2147483647之间的值，这些值分别是gidMin和gidMax的默认值。**)**

-   **volumetype** : The volume type and its parameters can be
    configured with this optional value. If the volume type is not
    mentioned, it's up to the provisioner to decide the volume type.

**(**可以使用此可选值配置卷类型及其参数。
如果未提及卷类型，则由供应商决定卷类型。**)**

For example:

-   Replica volume: **volumetype: replicate:3** where '3' is replica
    count.

-   Disperse/EC volume: **volumetype: disperse:4:2** where '4' is data
    and '2' is the redundancy count.

-   Distribute volume: **volumetype: none**

有关可用的卷类型和管理选项，请参阅"管理指南"。\
有关更多参考信息，请参阅如何配置Heketi。\
当动态配置持久卷时，Gluster插件会自动在名称gluster-dynamic-
\<claimname\>中创建端点和无头服务。
删除持久卷声明时，将自动删除动态端点和服务。

OpenStack Cinder

**apiVersion: storage.k8s.io/v1**

**kind: StorageClass**

**metadata:**

**name: gold**

**provisioner: kubernetes.io/cinder**

**parameters:**

**availability: nova**

-   **availability**: Availability Zone. If not specified, volumes are
    generally round-robin-ed across all active zones where Kubernetes
    cluster has a node.

**Note:**

**FEATURE STATE:** **Kubernetes 1.11**
[deprecated](https://kubernetes.io/docs/concepts/storage/storage-classes/)

This internal provisioner of OpenStack is deprecated. Please use [the
external cloud provider for
OpenStack](https://github.com/kubernetes/cloud-provider-openstack).

vSphere

1.  Create a StorageClass with a user specified disk format.(
    指定的磁盘格式)

2.  **apiVersion: storage.k8s.io/v1**

3.  **kind: StorageClass**

4.  **metadata:**

5.  **name: fast**

6.  **provisioner: kubernetes.io/vsphere-volume**

7.  **parameters:**

**diskformat: zeroedthick**

**diskformat**: **thin**, **zeroedthick** and **eagerzeroedthick**.
Default: **\"thin\"**.

8.  Create a StorageClass with a disk format on a user specified
    datastore.

9.  **apiVersion: storage.k8s.io/v1**

10. **kind: StorageClass**

11. **metadata:**

12. **name: fast**

13. **provisioner: kubernetes.io/vsphere-volume**

14. **parameters:**

15. **diskformat: zeroedthick**

**datastore: VSANDatastore**

**datastore**: The user can also specify the datastore in the
StorageClass. The volume will be created on the datastore specified in
the storage class, which in this case is **VSANDatastore**. This field
is optional. If the datastore is not specified, then the volume will be
created on the datastore specified in the vSphere config file used to
initialize the vSphere Cloud Provider.

(用户还可以在StorageClass中指定数据存储。
将在存储类中指定的数据存储上创建卷，在本例中为VSANDatastore。
该字段是可选的。 如果未指定数据存储，则将在用于初始化vSphere Cloud
Provider的vSphere配置文件中指定的数据存储上创建卷)

16. Storage Policy Management inside kubernetes

    -   Using existing vCenter SPBM policy

vSphere for Storage Management最重要的功能之一是基于策略的管理。
基于存储策略的管理（SPBM）是一种存储策略框架，可为广泛的数据服务和存储解决方案提供单一的统一控制平面。
SPBM使vSphere管理员能够克服前期存储配置挑战，例如容量规划，差异化服务级别和管理容量扩展空间。\
可以使用storagePolicyName参数在StorageClass中指定SPBM策略

-   Virtual SAN policy support inside Kubernetes

Vsphere
Infrastructure（VI）管理员将能够在动态卷配置期间指定自定义Virtual
SAN存储功能。
您现在可以在动态卷配置期间以存储功能的形式定义存储要求，例如性能和可用性。
存储功能要求将转换为Virtual
SAN策略，然后在创建持久卷（虚拟磁盘）时将其下推到Virtual SAN层。
虚拟磁盘分布在Virtual SAN数据存储区中以满足要求。\
您可以查看基于存储策略的管理以动态配置卷，以获取有关如何使用存储策略进行持久卷管理的更多详细信息。\
您尝试在Kubernetes for vSphere中进行持久性卷管理的vSphere示例很少。

Ceph RBD

**apiVersion: storage.k8s.io/v1**

**kind: StorageClass**

**metadata:**

**name: fast**

**provisioner: kubernetes.io/rbd**

**parameters:**

**monitors: 10.16.153.105:6789**

**adminId: kube**

**adminSecretName: ceph-secret**

**adminSecretNamespace: kube-system**

**pool: kube**

**userId: kube**

**userSecretName: ceph-secret-user**

**userSecretNamespace: default**

**fsType: ext4**

**imageFormat: \"2\"**

**imageFeatures: \"layering\"**

-   **monitors**: Ceph monitors, comma delimited. This parameter is
    required.

-   **adminId**: Ceph client ID that is capable of creating images in
    the pool. Default is "admin".

-   **adminSecretName**: Secret Name for **adminId**. This parameter is
    required. The provided secret must have type "kubernetes.io/rbd".

-   **adminSecretNamespace**: The namespace for **adminSecretName**.
    Default is "default".

-   **pool**: Ceph RBD pool. Default is "rbd".

-   **userId**: Ceph client ID that is used to map the RBD image.
    Default is the same as **adminId**.

-   **userSecretName**: The name of Ceph Secret for **userId** to map
    RBD image. It must exist in the same namespace as PVCs. This
    parameter is required. The provided secret must have type
    "kubernetes.io/rbd", e.g. created in this way:

-   **kubectl create secret generic ceph-secret
    \--type=\"kubernetes.io/rbd\" \\**

-   **\--from-literal=key=\'QVFEQ1pMdFhPUnQrSmhBQUFYaERWNHJsZ3BsMmNjcDR6RFZST0E9PQ==\'
    \\**

**\--namespace=kube-system**

-   **userSecretNamespace**: The namespace for **userSecretName**.

-   **fsType**: fsType that is supported by kubernetes. Default:
    **\"ext4\"**.

-   **imageFormat**: Ceph RBD image format, "1" or "2". Default is "2".

-   **imageFeatures**: This parameter is optional and should only be
    used if you set **imageFormat** to "2". Currently supported features
    are **layering** only. Default is "", and no features are turned on.

    Quobyte

**apiVersion: storage.k8s.io/v1**

**kind: StorageClass**

**metadata:**

**name: slow**

**provisioner: kubernetes.io/quobyte**

**parameters:**

**quobyteAPIServer: \"http://138.68.74.142:7860\"**

**registry: \"138.68.74.142:7861\"**

**adminSecretName: \"quobyte-admin-secret\"**

**adminSecretNamespace: \"kube-system\"**

**user: \"root\"**

**group: \"root\"**

**quobyteConfig: \"BASE\"**

**quobyteTenant: \"DEFAULT\"**

-   **quobyteAPIServer**: API Server of Quobyte in the format
    **\"http(s)://api-server:7860\"**

-   **registry**: Quobyte registry to use to mount the volume.(
    用于装入卷的注册表) You can specify the registry as
    **\<host\>:\<port\>** pair or if you want to specify multiple
    registries you just have to put a comma between them e.q.
    **\<host1\>:\<port\>,\<host2\>:\<port\>,\<host3\>:\<port\>**. The
    host can be an IP address or if you have a working DNS you can also
    provide the DNS names.

-   **adminSecretNamespace**: The namespace for **adminSecretName**.
    Default is "default".

-   **adminSecretName**: secret that holds information about the Quobyte
    user and the password to authenticate(认证) against the API server.
    The provided secret must have type "kubernetes.io/quobyte", e.g.
    created in this way:

-   **kubectl create secret generic quobyte-admin-secret \\**

-   **\--type=\"kubernetes.io/quobyte\"
    \--from-literal=key=\'opensesame\' \\**

**\--namespace=kube-system**

-   **user**: maps all access to this user. Default is "root".

-   **group**: maps all access to this group. Default is "nfsnobody".

-   **quobyteConfig**: use the specified configuration to create the
    volume. You can create a new configuration or modify an existing one
    with the Web console or the quobyte CLI. Default is "BASE".

-   **quobyteTenant**: use the specified tenant ID to create/delete the
    volume. This Quobyte tenant has to be already present in Quobyte.
    Default is "DEFAULT".

    Azure Disk

    Azure Unmanaged Disk Storage Class

**apiVersion: storage.k8s.io/v1**

**kind: StorageClass**

**metadata:**

**name: slow**

**provisioner: kubernetes.io/azure-disk**

**parameters:**

**skuName: Standard\_LRS**

**location: eastus**

**storageAccount: azure\_storage\_account\_name**

-   **skuName**: Azure storage account Sku tier. Default is empty.

-   **location**: Azure storage account location. Default is empty.

-   **storageAccount**: Azure storage account name. If a storage account
    is provided, it must reside in the same resource group as the
    cluster, and **location** is ignored. If a storage account is not
    provided, a new storage account will be created in the same resource
    group as the cluster.

**(**Azure存储帐户名称。
如果提供了存储帐户，则它必须与群集位于同一资源组中，并忽略位置。
如果未提供存储帐户，则将在与群集相同的资源组中创建新的存储帐户。**)**

New Azure Disk Storage Class (starting from v1.7.2)

**apiVersion: storage.k8s.io/v1**

**kind: StorageClass**

**metadata:**

**name: slow**

**provisioner: kubernetes.io/azure-disk**

**parameters:**

**storageaccounttype: Standard\_LRS**

**kind: Shared**

-   **storageaccounttype**: Azure storage account Sku tier. Default is
    empty.

-   **kind**: Possible values are **shared** (default), **dedicated**,
    and **managed**. When **kind** is **shared**, all unmanaged disks
    are created in a few shared storage accounts in the same resource
    group as the cluster. When **kind** is **dedicated**, a new
    dedicated storage account will be created for the new unmanaged disk
    in the same resource group as the cluster. When **kind** is
    **managed**, all managed disks are created in the same resource
    group as the cluster.

-   Premium VM can attach both Standard\_LRS and Premium\_LRS disks,
    while Standard VM can only attach Standard\_LRS disks.

-   Managed VM can only attach managed disks and unmanaged VM can only
    attach unmanaged disks.

    Azure File

**apiVersion: storage.k8s.io/v1**

**kind: StorageClass**

**metadata:**

**name: azurefile**

**provisioner: kubernetes.io/azure-file**

**parameters:**

**skuName: Standard\_LRS**

**location: eastus**

**storageAccount: azure\_storage\_account\_name**

-   **skuName**: Azure storage account Sku tier. Default is empty.

-   **location**: Azure storage account location. Default is empty.

-   **storageAccount**: Azure storage account name. Default is empty. If
    a storage account is not provided, all storage accounts associated
    with the resource group are searched to find one that matches
    **skuName** and **location**. If a storage account is provided, it
    must reside in the same resource group as the cluster, and
    **skuName** and **location** are ignored.

**(**Azure存储帐户名称。 默认为空。
如果未提供存储帐户，则搜索与该资源组关联的所有存储帐户以查找与skuName和location匹配的帐户。
如果提供了存储帐户，则它必须与群集位于同一资源组中，并且将忽略skuName和location。**)**

During provision, a secret is created for mounting credentials.(
安装凭证) If the cluster has enabled both
[RBAC](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)
and [Controller
Roles](https://kubernetes.io/docs/reference/access-authn-authz/rbac/#controller-roles),
add the **create** permission of resource **secret** for clusterrole
**system:controller:persistent-volume-binder**.

Portworx Volume

**apiVersion: storage.k8s.io/v1**

**kind: StorageClass**

**metadata:**

**name: portworx-io-priority-high**

**provisioner: kubernetes.io/portworx-volume**

**parameters:**

**repl: \"1\"**

**snap\_interval: \"70\"**

**io\_priority: \"high\"**

-   **fs**: filesystem to be laid out: **none/xfs/ext4** (default:
    **ext4**).

-   **block\_size**: block size in Kbytes (default: **32**).

-   **repl**: number of synchronous replicas(同步副本) to be provided in
    the form of replication factor(复制因子) **1..3** (default: **1**) A
    string is expected here i.e. **\"1\"** and not **1**.

-   **io\_priority**: determines whether the volume will be created from
    higher performance or a lower priority(优先) storage
    **high/medium/low** (default: **low**).

-   **snap\_interval**: clock/time interval in minutes for when to
    trigger snapshots. Snapshots are incremental based on difference
    with the prior snapshot, 0 disables snaps (default: **0**). A string
    is expected here i.e. **\"70\"** and not **70**.

**（**何时触发快照的时钟/时间间隔（分钟）。
快照是基于与先前快照的差异而增量的，0禁用快照（默认值：0）。
这里预期一个字符串，即"70"而不是70。**）**

-   **aggregation\_level**: specifies the number of chunks the volume
    would be distributed into, 0 indicates a non-aggregated volume
    (default: **0**). A string is expected here i.e. **\"0\"** and not
    **0**

-   **ephemeral**: specifies whether the volume should be cleaned-up
    after unmount or should be persistent. **emptyDir** use case can set
    this value to true and **persistent volumes** use case such as for
    databases like Cassandra should set to false, **true/false**
    (default **false**). A string is expected here i.e. **\"true\"** and
    not **true**.

    ScaleIO

**apiVersion: storage.k8s.io/v1**

**kind: StorageClass**

**metadata:**

**name: slow**

**provisioner: kubernetes.io/scaleio**

**parameters:**

**gateway: https://192.168.99.200:443/api**

**system: scaleio**

**protectionDomain: pd0**

**storagePool: sp1**

**storageMode: ThinProvisioned**

**secretRef: sio-secret**

**readOnly: false**

**fsType: xfs**

-   **provisioner**: attribute is set to **kubernetes.io/scaleio**

-   **gateway**: address to a ScaleIO API gateway (required)

-   **system**: the name of the ScaleIO system (required)

-   **protectionDomain**: the name of the ScaleIO protection domain
    (required)

-   **storagePool**: the name of the volume storage pool (required)

-   **storageMode**: the storage provision mode: **ThinProvisioned**
    (default) or **ThickProvisioned**

-   **secretRef**: reference to a configured Secret object (required)

-   **readOnly**: specifies the access mode to the mounted volume
    (default false)

-   **fsType**: the file system to use for the volume (default ext4)

ScaleIO Kubernetes卷插件需要配置的Secret对象。
必须使用类型kubernetes.io/scaleio创建密钥，并使用与引用它的PVC相同的命名空间值，如以下命令所示：

**kubectl create secret generic sio-secret
\--type=\"kubernetes.io/scaleio\" \\**

**\--from-literal=username=sioadmin
\--from-literal=password=d2NABDNjMA== \\**

**\--namespace=default**

StorageOS

**apiVersion: storage.k8s.io/v1**

**kind: StorageClass**

**metadata:**

**name: fast**

**provisioner: kubernetes.io/storageos**

**parameters:**

**pool: default**

**description: Kubernetes volume**

**fsType: ext4**

**adminSecretNamespace: default**

**adminSecretName: storageos-secret**

-   **pool**: The name of the StorageOS distributed capacity pool to
    provision the volume from. Uses the **default** pool which is
    normally present if not specified.

-   **description**: 分配给动态创建的卷的描述。
    存储类的所有卷描述都是相同的，但可以使用不同的存储类来描述不同的用例。
    默认为Kubernetes音量

-   **fsType**: The default filesystem type to request. Note that
    user-defined rules within StorageOS may override this value.
    Defaults to **ext4**.

-   **adminSecretNamespace**: The namespace where the API configuration
    secret is located. Required if adminSecretName set.

-   **adminSecretName**: The name of the secret to use for obtaining the
    StorageOS API credentials. If not specified, default values will be
    attempted.

StorageOS
Kubernetes卷插件可以使用Secret对象来指定端点和凭据以访问StorageOS API。
只有在更改默认值时才需要这样做。
必须使用kubernetes.io/storageos类型创建密钥，如以下命令所示：

**kubectl create secret generic storageos-secret \\**

**\--type=\"kubernetes.io/storageos\" \\**

**\--from-literal=apiAddress=tcp://localhost:5705 \\**

**\--from-literal=apiUsername=storageos \\**

**\--from-literal=apiPassword=storageos \\**

**\--namespace=default**

可以在任何命名空间中创建用于动态配置卷的秘密，并使用adminSecretNamespace参数进行引用。
必须在与引用它的PVC相同的命名空间中创建预配置卷使用的秘密。

Local

**FEATURE STATE:** **Kubernetes v1.14**
[stable](https://kubernetes.io/docs/concepts/storage/storage-classes/)

**apiVersion: storage.k8s.io/v1**

**kind: StorageClass**

**metadata:**

**name: local-storage**

**provisioner: kubernetes.io/no-provisioner**

**volumeBindingMode: WaitForFirstConsumer**

本地卷当前不支持动态配置，但仍应创建StorageClass以延迟卷绑定，直到pod调度。
这由WaitForFirstConsumer卷绑定模式指定。

延迟卷绑定允许调度程序在为PersistentVolumeClaim选择适当的PersistentVolume时考虑所有pod的调度约束。

Volume Snapshot Classes
=======================

This document describes the concept of **VolumeSnapshotClass** in
Kubernetes. Familiarity with [volume
snapshots](https://kubernetes.io/docs/concepts/storage/volume-snapshots/)
and [storage
classes](https://kubernetes.io/docs/concepts/storage/storage-classes) is
suggested.

-   [**Introduction**](https://kubernetes.io/docs/concepts/storage/volume-snapshot-classes/#introduction)

-   [**The VolumeSnapshotClass
    Resource**](https://kubernetes.io/docs/concepts/storage/volume-snapshot-classes/#the-volumesnapshotclass-resource)

-   [**Parameters**](https://kubernetes.io/docs/concepts/storage/volume-snapshot-classes/#parameters)

Introduction
------------

就像StorageClass为管理员提供了一种描述他们在配置卷时提供的"存储类"的方法一样，VolumeSnapshotClass提供了一种在配置卷快照时描述存储"类"的方法。

The VolumeSnapshotClass Resource
--------------------------------

每个VolumeSnapshotClass都包含字段snapshotter和parameters，当需要动态配置属于该类的VolumeSnapshot时使用这些字段。\
VolumeSnapshotClass对象的名称很重要，是用户可以请求特定类的方式。
管理员在首次创建VolumeSnapshotClass对象时设置类的名称和其他参数，并且在创建对象后无法更新这些对象。\
管理员可以为不请求任何特定类绑定的VolumeSnapshots指定默认的VolumeSnapshotClass。

**apiVersion: snapshot.storage.k8s.io/v1alpha1**

**kind: VolumeSnapshotClass**

**metadata:**

**name: csi-hostpath-snapclass**

**snapshotter: csi-hostpath**

**parameters:**

### Snapshotter

卷快照类具有一个快照程序，用于确定用于配置VolumeSnapshot的CSI卷插件。
必须指定此字段。

Parameters
----------

卷快照类具有描述属于卷快照类的卷快照的参数。
取决于快照器，可以接受不同的参数。

Dynamic Volume Provisioning
===========================

动态卷配置允许按需创建存储卷。
如果没有动态配置，集群管理员必须手动调用其云或存储提供程序来创建新的存储卷，然后创建PersistentVolume对象以在Kubernetes中表示它们。
动态配置功能使集群管理员无需预先配置存储。
相反，它会在用户请求时自动配置存储。

-   [**Background**](https://kubernetes.io/docs/concepts/storage/dynamic-provisioning/#background)

-   [**Enabling Dynamic
    Provisioning**](https://kubernetes.io/docs/concepts/storage/dynamic-provisioning/#enabling-dynamic-provisioning)

-   [**Using Dynamic
    Provisioning**](https://kubernetes.io/docs/concepts/storage/dynamic-provisioning/#using-dynamic-provisioning)

-   [**Defaulting
    Behavior**](https://kubernetes.io/docs/concepts/storage/dynamic-provisioning/#defaulting-behavior)

-   [**Topology
    Awareness**](https://kubernetes.io/docs/concepts/storage/dynamic-provisioning/#topology-awareness)

Background
----------

动态卷配置的实现基于API组storage.k8s.io中的API对象StorageClass。
集群管理员可以根据需要定义任意数量的StorageClass对象，每个对象都指定一个卷插件（也称为配置器），用于配置卷以及在配置时传递给该配置器的参数集。
集群管理员可以在集群中定义和公开多种存储（来自相同或不同的存储系统），每种存储都具有一组自定义参数。
此设计还确保最终用户不必担心如何配置存储的复杂性和细微差别，但仍可以从多个存储选项中进行选择。

More information on storage classes can be found
[here](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#storageclasses).

Enabling Dynamic Provisioning
-----------------------------

要启用动态配置，集群管理员需要为用户预先创建一个或多个StorageClass对象。
StorageClass对象定义应该使用哪个配置程序以及在调用动态配置时应将哪些参数传递给该配置程序。
以下清单创建一个存储类"slow"，它提供标准的类磁盘永久磁盘。

**apiVersion: storage.k8s.io/v1**

**kind: StorageClass**

**metadata:**

**name: slow**

**provisioner: kubernetes.io/gce-pd**

**parameters:**

**type: pd-standard**

The following manifest creates a storage class "fast" which provisions
SSD-like persistent disks.

**apiVersion: storage.k8s.io/v1**

**kind: StorageClass**

**metadata:**

**name: fast**

**provisioner: kubernetes.io/gce-pd**

**parameters:**

**type: pd-ssd**

Using Dynamic Provisioning
--------------------------

用户通过在其PersistentVolumeClaim中包含存储类来请求动态调配存储。
在Kubernetes
v1.6之前，这是通过volume.beta.kubernetes.io/storage-class注释完成的。
但是，自v1.6起，此批注已弃用。
用户现在可以而且应该使用PersistentVolumeClaim对象的storageClassName字段。
此字段的值必须与管理员配置的StorageClass的名称匹配（请参阅下文）

To select the "fast" storage class, for example, a user would create the
following **PersistentVolumeClaim**:

**apiVersion: v1**

**kind: PersistentVolumeClaim**

**metadata:**

**name: claim1**

**spec:**

**accessModes:**

**- ReadWriteOnce**

**storageClassName: fast**

**resources:**

**requests:**

**storage: 30Gi**

此声明导致自动配置类似SSD的永久磁盘。 删除声明后，将销毁该卷。

Defaulting Behavior
-------------------

可以在群集上启用动态预配置，以便在未指定存储类的情况下动态设置所有声明。
集群管理员可以通过以下方式启用此行为\
•将一个StorageClass对象标记为默认值;\
•确保在API服务器上启用DefaultStorageClass许可控制器。\
管理员可以通过向其添加storageclass.kubernetes.io/is-default-class注释来将特定StorageClass标记为默认值。
当集群中存在默认StorageClass且用户创建未指定storageClassName的PersistentVolumeClaim时，DefaultStorageClass许可控制器会自动添加指向默认存储类的storageClassName字段。\
请注意，群集上最多只能有一个默认存储类，或者无法创建未明确指定storageClassName的PersistentVolumeClaim。

Topology Awareness
------------------

在多区域群集中，Pod可以分布在区域中的区域中。
应在安排Pod的区域中配置单区存储后端。 这可以通过设置卷绑定模式来完成。

Node-specific Volume Limits
===========================

此页面描述了可以为各种云提供程序附加到节点的最大卷数。\
Google，Amazon和Microsoft等云提供商通常对可以连接到节点的卷数量进行限制。
对于Kubernetes来说，尊重这些限制很重要。
否则，在节点上安排的Pod可能会卡在等待卷附加。

-   [**Kubernetes default
    limits**](https://kubernetes.io/docs/concepts/storage/storage-limits/#kubernetes-default-limits)

-   [**Custom
    limits**](https://kubernetes.io/docs/concepts/storage/storage-limits/#custom-limits)

-   [**Dynamic volume
    limits**](https://kubernetes.io/docs/concepts/storage/storage-limits/#dynamic-volume-limits)

Kubernetes default limits
-------------------------

The Kubernetes scheduler has default limits on the number of volumes
that can be attached to a Node:

（Kubernetes调度程序对可以附加到节点的卷数有默认限制）

  Cloud service                                                                                    Maximum volumes per Node
  ------------------------------------------------------------------------------------------------ --------------------------
  [Amazon Elastic Block Store (EBS)](https://aws.amazon.com/ebs/)                                  39
  [Google Persistent Disk](https://cloud.google.com/persistent-disk/)                              16
  [Microsoft Azure Disk Storage](https://azure.microsoft.com/en-us/services/storage/main-disks/)   16

Custom limits
-------------

您可以通过设置KUBE\_MAX\_PD\_VOLS环境变量的值，然后启动调度程序来更改这些限制。\
如果设置的限制高于默认限制，请务必小心。
请参阅云提供商的文档，以确保节点实际上可以支持您设置的限制。\
该限制适用于整个群集，因此它会影响所有节点。

Dynamic volume limits
---------------------

**FEATURE STATE:** **Kubernetes v1.12**
[beta](https://kubernetes.io/docs/concepts/storage/storage-limits/)

Kubernetes 1.11引入了对基于节点类型的动态音量限制的支持作为Alpha功能。
在Kubernetes 1.12中，此功能即将升级到Beta版，默认情况下将启用

Dynamic volume limits are supported for following volume types.

-   Amazon EBS

-   Google Persistent Disk

-   Azure Disk

-   CSI

启用动态卷限制功能后，Kubernetes会自动确定节点类型并为节点强制执行适当数量的可附加卷。
例如：

On [Google Compute Engine](https://cloud.google.com/compute/), up to 128
volumes can be attached to a node, [depending on the node
type](https://cloud.google.com/compute/docs/disks/#pdnumberlimits).

-   For Amazon EBS disks on M5,C5,R5,T3 and Z1D instance types,
    Kubernetes allows only 25 volumes to be attached to a Node. For
    other instance types on [Amazon Elastic Compute Cloud
    (EC2)](https://aws.amazon.com/ec2/), Kubernetes allows 39 volumes to
    be attached to a Node.

-   On Azure, up to 64 disks can be attached to a node, depending on the
    node type. For more details, refer to [Sizes for virtual machines in
    Azure](https://docs.microsoft.com/en-us/azure/virtual-machines/windows/sizes).

-   For CSI,
    •任何通过CSI规范公布卷附加限制的驱动程序将具有可用作节点的可分配属性的那些限制，并且调度程序不会在已经达到其容量的任何节点上调度具有卷的Pod。
    有关更多详细信息，请参阅CSI规范。
