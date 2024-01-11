# gitOps

GitOps是一个近年来变得非常流行的术语，正迅速变得与DevOps一样充满神秘。有关GitOps的一些定义将其描述为一种传递和维护基于Kubernetes应用运行的基础设施的机制。其他定义包括基础设施焦点，并将其与为平台创建的业务应用的定义相结合。在这一系列文章中，我们将介绍GitOps的原则和实践，解释自动化流程的原因和方式，从而快速高效地交付安全、高质量的基于微服务的应用。

GitOps方法的目标和好处是很容易理解的。通过将所需基础设施和/或应用配置的定义存储在Git存储库中，团队拥有了在结构化和安全存储中部署应用所需的一切。如果由于灾难恢复或弹性的原因需要将应用重新部署到新位置，那么可以轻松而有信心地完成。此外，GitOps模型将帮助组织在将应用推向生产环境的过程中通过各种测试阶段。这个过程需要将应用部署到不同的集群或集群内的不同命名空间，而GitOps模型确保这可以完全、可靠且有信心地完成。

基础设施和应用是两个领域之间存在明显区别的领域，组织的不同团队分别负责每个领域。基础设施配置负责定义和交付Kubernetes平台，涉及计算节点、操作员、基于角色的访问控制、治理策略等属性，这些属性创建了一个应用可以运行的平台。应用可以分为源代码组件，这些组件被编译成容器映像中的可执行文件，以及在特定环境中配置容器映像的Kubernetes资源。

GitOps的指导原则简单明了：

- 在Git存储库中应维护唯一的真实数据来源。
    
- 将使用自动化来应用对Git存储库进行的任何更改。
    
- 对环境的任何手动更改将自动回滚。
    

GitOps方法基于DevOps的文化和技术原则，有许多更多的核心原则。因此，对于“GitOps是什么”的问题可能没有绝对正确或错误的答案，答案可能在于通过自动化提供价值和允许变更在组织内根据需要记录、传播和回溯的工作实践中。

简单来说，GitOps 是在 Git 存储库中定义应用程序配置的唯一真实数据来源的过程。如果团队希望更改应用的配置，他们只需修改适当的Git存储库中的文件，然后等待自动化流程更新应用。这个过程的扩展可以涉及使用外部工单系统和Git存储库中的拉取请求，以确保只有经批准和适当的更改被传播到生产环境。

## 源代码管理

有许多源代码管理系统可用，但正如名称所示，Git是GitOps方法的核心。任何提供Git类型界面的源代码管理系统都可以使用，在本文的其余部分中，对Git的任何引用都将假定为GitHub，因为这是本文内容的开发和测试使用的平台。

源代码管理系统与GitOps模型的交互基于以下标准：

源代码管理系统在执行特定操作时触发外部流程的能力。例如，向特定分支提交新源代码可能会导致应用构建开始。构建过程将从相关分支克隆源代码的最新版本开始。或者，合并操作可能会触发构建过程。这些都是“推送”触发的操作的示例，其中对源代码管理系统的更改会引起外部事件。当发生这些触发事件时，将向特定URL发送有关事件的Webhook数据负载。推送模型如图1所示，其中用户执行“git push”操作。请注意，还可以使用其他Git操作，如创建拉取请求。
![](Pasted%20image%2020240111201530.png)
自动化部署解决方案监控源代码管理系统以识别新的提交能力。这将允许在“拉取”模式中触发操作，其中在观察到Git内容的变化后，外部实体决定执行某个动作。在拉取模型中，不包含任何负载内容，外部系统必须请求其需要的任何信息。
![](Pasted%20image%2020240111201547.png)
拉取和推送基于触发操作将用于GitOps模型的不同方面。

## CICD
为了实现上述拉取和推送模型，团队需要自动化软件解决方案。在红帽OpenShift订阅中包含的并由红帽提供全面支持的开源解决方案如下：

- 红帽OpenShift Pipelines（Tekton）

红帽将Tekton开源项目的支持和集成实现作为OpenShift Pipelines提供。这提供了一个完整的持续集成过程，能够执行软件构建、容器镜像创建、容器镜像管理、测试操作以及使用各种测试和验证解决方案进行安全扫描。OpenShift Pipelines通过在执行流水线过程的离散步骤中执行容器镜像内的命令来操作。任何可以托管在容器镜像中的命令行实用程序都可以使用。Tekton触发器可用于响应来自GitHub的Webhook请求，以便在GitHub（或其他源代码管理解决方案）中发生操作的结果下执行Tekton流水线。

流水线资产以YAML文件的形式交付，这些文件作为资源在OpenShift集群的特定命名空间中创建。常常使用上游项目名称“Tekton”来指代OpenShift Pipelines。

- 红帽OpenShift GitOps（ArgoCD）

红帽将ArgoCD开源项目的支持和集成实现作为OpenShift GitOps提供。这提供了一个完整的持续交付过程，能够监视GitHub存储库，并确保OpenShift平台上的Kubernetes资源与GitHub存储库中的内容保持同步。ArgoCD应用程序用于监视GitHub存储库中的特定文件集，并创建必要的Kubernetes资源。ArgoCD可用于向集群交付应用程序所需的资源，还可用于交付OpenShift Pipelines的资源，如任务。

## Git存储库内容
不同类别的内容需要存储在不同的Git存储库中。这将有助于实现实际的组织结构，并且还将允许根据组织内谁有权读取和/或写入内容的角色，采用不同的基于访问控制机制的角色进行管理。在此示例中，以下列表显示了将要使用的存储库的总体结构和内容：

1. 应用程序源代码 - 将被编译成可在生产容器映像中托管的可执行文件的内容。
    
2. GitOps配置 - 控制初始GitOps资产创建的资源。这将包括所有下面描述的其他资产位置的定义。
    
3. 持续集成 - 用于从源代码和基础容器映像构建新容器映像的管道过程的资源。
    
4. 持续交付 - 描述应用程序将如何部署到“上线路”各个环境的资源。此示例将在存储库中为每个环境显示不同的目录；然而，为了更大程度的隔离，可以为每个环境使用单独的存储库。

## 系统组成
在这种情况下有许多组成部分，因此了解哪种技术负责创建哪些内容是很重要的。

  

容器镜像

  

应用程序容器镜像是通过将基础容器镜像与构建的构件（例如JAR文件）组合而生成的。JAR文件（例如）是通过构建应用程序源代码产生的。容器构建操作由Tekton流程执行。

  

- 输入：Github存储库中的应用程序源代码
    

- 包含和所需运行时应用程序的基础容器镜像
    

- 输出：OpenShift镜像流中的容器镜像
    
- Operator：Tekton管道过程
    

  

Tekton管道

  

Tekton管道是在特定命名空间中创建的一组Kubernetes资源。ArgoCD应用程序负责从GitHub存储库创建和持续同步Tekton资源。

  

- 输入：Github存储库中的Tekton YAML文件
    
- 输出：OpenShift命名空间中的Tekton资源
    
- Operator：ArgoCD应用程序
    

  

部署到开发环境

  

新的容器镜像需要部署到OpenShift集群上的开发环境（命名空间）。这涉及使用部署、服务、路由等。YAML文件存储在GitHub存储库中。ArgoCD应用程序负责从GitHub存储库的开发目录创建和持续同步应用程序资源。

  

- 输入：GitHub存储库中的应用程序YAML文件
    

- OpenShift映像流中的容器映像
    

- 输出：开发命名空间中应用程序的运行实例
    
- Operator：ArgoCD应用程序
    

  

部署到QA环境

  

与开发环境一样，新的容器镜像需要部署到OpenShift集群上的QA环境（命名空间）。ArgoCD应用程序负责从GitHub存储库的QA目录创建和持续同步应用程序资源。

  

- 输入：GitHub存储库中的应用程序YAML文件
    

- OpenShift映像流中的容器映像
    

- 输出：QA命名空间中应用程序的运行实例
    
- Operator：ArgoCD应用程序
    

  

ArgoCD应用程序实例 - 配置应用程序

  

上面存在三个ArgoCD应用程序实例，而在涉及更多环境的客户实施中可能需要更多。为确保按需创建所有ArgoCD应用程序，可以使用一个配置应用程序。这通常被称为“应用程序的应用程序”，因为它包含对其他ArgoCD应用程序的引用。

  

- 输入：ArgoCD应用程序定义（包括自身）
    
- 输出：配置的ArgoCD应用程序（控制器）
    

- CI流程的ArgoCD应用程序
    
- Dev环境的ArgoCD应用程序
    
- QA环境的ArgoCD应用程序
    

- Operator：ArgoCD应用程序

## 配置过程
图3显示了上述存储库及其关系。ArgoCD资源（绿色所示）负责响应GitHub存储库中的更改并创建内容。它们生成的资产可以是蓝色显示的Tekton资源（图3中的步骤2），也可以是红色显示的Kubernetes资源（图3中的步骤4）。

![](Pasted%20image%2020240111202153.png)
应用程序源代码存储库已配置了一个Webhook，当提交了新内容时，它将通知持续集成构建过程。这由图3中的步骤1表示。持续集成构建过程由Tekton提供，用于创建任务和管道的资源保存在持续集成存储库中。当团队修改并提交了任务和管道时，需要一种机制来提供更新后的任务和管道。为此，使用了一个ArgoCD应用程序，它将监视持续集成存储库，并确保所有资源都与用于提供Tekton构建过程的Kubernetes资源同步。这种监视在图3的步骤2中显示，其中ArgoCD项目正在监视Git存储库（红线）并更新Kubernetes资源（步骤2，灰线）。

  

Git配置存储库负责创建和管理用于提供持续集成和持续交付过程的所有ArgoCD应用程序。在创建过程时，团队需要手动创建图3中第3步中显示的初始ArgoCD项目（配置应用程序）。该项目指向包含所有ArgoCD应用程序定义的GitOps配置Git存储库。如上述的每个ArgoCD应用程序创建时，它们将开始同步到它们引用的资源的过程。

  

图3的第4步显示了持续交付过程的引入。要将应用程序部署到环境需要一组YAML文件，通常包括部署、服务、路由、配置映射、密码、持久卷定义等。这些资源存储在连续交付存储库中，其中包含一个允许定义一组共同资源的基本结构（步骤5），以及特定文件的覆盖以适应环境变化。为了部署每组特定于环境的Kubernetes资源，需要为每个环境创建一个ArgoCD应用程序。例如，要将应用程序部署到01-dev环境，将创建一个ArgoCD应用程序，该应用程序引用连续交付Git存储库中的/environment/01-dev目录。使用ArgoCD应用程序可以确保对/environment/01-dev目录内容的任何更改都会被忠实地应用到Kubernetes环境中。作为负责将应用程序部署到每个环境的工程师，我只需对YAML文件进行更改，将其提交到Git存储库，并允许ArgoCD将这些更改应用到Kubernetes环境。如果要引入新环境到上线路，然后团队执行以下任务：

  

- 在连续交付Git存储库的环境目录中定义所需的YAML文件
    
- 在GitOps配置Git存储库的环境目录中创建新的ArgoCD应用程序定义。此应用程序引用在连续交付存储库中创建的新目录
    
- 允许图3中第3步指示的配置ArgoCD应用程序识别到结构已更改，这将创建在上述第2阶段创建的新ArgoCD应用程序
    
- 允许新环境的新ArgoCD应用程序部署应用程序Kubernetes资源

![](Pasted%20image%2020240111202351.png)

应用程序源代码的开发人员可以更改代码，并放心地知道他们的提交将由自动的 Tekton 流程捡起并进行构建。环境工程师可以通过更新和提交到Git的Kubernetes资源文件来更改应用的行为。部署自动化工程师可以通过向配置存储库添加新文件来创建新环境，并定义ArgoCD应用程序以引用它们。
所有与业务应用的创建、构建和部署相关的资产都被安全地存储在Git存储库中，并受到访问控制、审计和日志记录的保护。

## 容器镜像创建

在GitOps模型中，微服务应用的一个关键要素是创建新的容器镜像。这些容器镜像不仅具有存储位置，还有一个唯一标签来标识镜像的特定版本。通常，容器镜像还会应用一些标签，用于描述镜像的重要特征，例如维护者、版本标识符和生产镜像的组织。

有时候可能会诱人地将“latest”用作镜像的标签，因为这样在将镜像部署到环境中时更加方便。每个部署过程只需要使用镜像名称，后面加上“:latest”即可获取最新版本。然而，使用“latest”会丧失在特定环境中立即区分部署镜像的机会，并且在后续管道流程中（例如在部署到生产环境时）需要重新标记的过程。从创建镜像时开始使用唯一的标签是一个更好的做法，但当流程到达初始部署阶段时，这可能会使管道流程变得复杂。管道需要使用在流程开始时并不知道的镜像标签将容器部署到开发环境。这引出了一个需求，即在部署YAML文件中进行动态修补以使用新标签。

图1展示了一个流程，其中执行了以下任务：
构建源代码以生成一个jar文件。
将jar文件注入到包含应用所需运行时软件的干净基础镜像中。
在管道执行期间，新的容器镜像将被推送到一个带有唯一标签的镜像注册表。
部署容器以进行初始测试。

![](Pasted%20image%2020240111203041.png)

### deployment.yaml要求


部署的YAML文件（我将假设它是一个简单示例中的deployment.yaml）如图2所示。在下面的YAML文件中，属性spec.template.spec.image标识了在部署中要使用的容器镜像。在图2的示例中，该镜像是从Red Hat OpenShift镜像流中获取的。在本文末尾，描述了用于将镜像永久存储以供部署到生产环境的过程。镜像规范行具有一个<tag'>的标签，该标签将在容器镜像创建流水线期间替换为创建的标签。

```yaml
kind: Deployment  
apiVersion: apps/v1  
metadata:  
  labels:  
    app: myapp  
    app.kubernetes.io/part-of: liberty  
  name: myapp  
  namespace: myapp-development  
spec:  
  replicas: 1  
  selector:  
    matchLabels:  
      app: myapp  
  template:  
    metadata:  
      labels:  
        app: myapp  
    spec:  
      containers:  
      - name: myapp  
        image: image-registry.openshift-image-registry.svc:5000/myapp-ci/myapp-runtime:<tag>    
        imagePullPolicy: Always  
        ports:  
        - containerPort: 9080  
          name: http  
          protocol: TCP
```

### kustomize 用于自定义部署
开源工具Kustomize 在修改 Kubernetes YAML 文件方面有多种用途。它可以在部署之前生成一个新文件，用于应用于集群，也可以直接由 OC（或 kubectl）命令使用，将动态修补的资源应用于集群。Kustomize 的使用核心是管理一个指向要部署到集群的 Kubernetes 资源的 kustomization.yaml 文件。除了能够用单个命令将一组文件应用到集群之外，kustomization.yaml 文件还可以包含要应用于其引用的 YAML 文件的文本替换规范。

下面是一个基本kustomization.yaml文件的示例，如图所示。
```yaml
apiVersion: kustomize.config.k8s.io/v1beta1  
kind: Kustomization  
resources:  
- namespace.yaml  
- myapp-deployment.yaml  
- myapp-service.yaml  
- myapp-route.yaml  
- 01-rolebindings/argocd-admin.yaml  
- 01-rolebindings/ci-pipeline-role.yaml
```
图3中的文件将应用当前目录中的四个文件和子目录01-rolebindings中的两个文件。此文件中没有使用文本替换。

以下命令可用于处理Kustomize文件并将六个资源应用到集群。该命令假定kustomization.yaml文件位于当前目录中。
```sh
oc apply -k .
```
Kustomize文件通常用于引用一组基础文件，然后对其进行文本替换。下面是一个示例，图4显示了其中引用了一组基础文件的情况。
```sh
apiVersion: kustomize.config.k8s.io/v1beta1  
kind: Kustomization  
resources:  
- ../base
```
可以像上面图3中所示那样，添加额外的特定文件来补充基础资源集。

下面是另一个Kustomize文件的示例，图5显示了其中引用了一组基础文件，并包含文本替换规范：

命名空间声明被替换为值 'myapp-pre-prod'
副本数更改为4

这些更改将应用于名为 'myapp' 的部署资源。
```sh
apiVersion: kustomize.config.k8s.io/v1beta1  
kind: Kustomization  
patchesJson6902:  
- patch: |-  
    - op: replace  
      path: /metadata/namespace  
      value: myapp-pre-prod  
- patch: |-  
    - op: replace  
      path: /spec/replicas  
      value: 4  
  target:  
    kind: Deployment  
    name: myapp     
- ../base
```
Kustomize具有许多创造性修改Kubernetes资源的功能，而上述描述仅仅触及了它的一小部分。建议读者查看官方Kustomize文档以获取更多详细信息。

### 更新部署文件中的镜像标签

使用Kustomize可以确保在将应用部署到开发环境时使用构建过程中生成的新镜像标签。这个过程不会直接修改deployment.yaml文件；相反，在Kustomize文件中添加了一个新的部分，因此在执行部署操作时，Kustomize文件会以及时的方式更新deployment.yaml文件。执行修改Kustomize文件的命令如下：
```sh
kustomize edit set image <new-image-tag>
```
这个命令将向Kustomize文件添加类似于图6中所示的行：
```sh
images:  
- name: image-registry.openshift-image-registry.svc:5000/myapp-ci/myapp-runtime  
  newTag: <new-image-tag>
```
例如，使用以下命令：
```sh
kustomize edit set image image-registry.openshift-image-registry.svc:5000/myapp-ci/myapp-runtime:abcd-1234ef
```

将导致向Kustomize文件中添加或修改，如图7所示：
```sh
images:  
- name: image-registry.openshift-image-registry.svc:5000/myapp-ci/myapp-runtime  
  newTag: abcd-1234ef
```
为了测试Kustomize流程，并查看上述更改对deployment.yaml文件的影响，可以使用以下命令：

Kustomize build

此命令将显示处理Kustomize文件的结果，所有引用的YAML文件将显示在一起，每个不同的文件之间用一行 '-' 字符分隔。

### 自动化kustomization文件更改
为了将该过程自动化为管道操作的一部分，可以使用Tekton任务来执行Kustomize命令，将镜像标签添加到kustomization.yaml文件。下面是一个执行此操作的Tekton任务示例，如图8所示：
```
apiVersion: tekton.dev/v1alpha1  
kind: Task  
metadata:  
  name: configure-kustomization-file  
  namespace: myapp-ci  
spec:  
  params:  
  - name: image-url  
    type: string  
  steps:    
    - name: configure-kustomization-file  
      script: >-  
        #!/usr/bin/env bash  
        set +x  
        ls -al  
        kustomize edit set image $(params.image-url)  
        ls -al  
        cat kustomization.yaml  
      image: quay.io/marrober/kustomize:latest  
      workingDir: /files/myapp-cd/env/overlays/01-dev       
  workspaces:  
  - name: files  
    mountPath: /files
```
图8中configure-kustomization-file步骤中指定的容器镜像（quay.io/marrober/kustomize:latest）存储在作者的quay.io公共注册表中，任何人都可以使用。用于生成此镜像的Docker文件位于此处。添加到容器镜像中的其他工具在随后的一篇文章中进行了解释，该文章详细介绍了资源扫描过程。

‘kustomize edit set image’命令中要使用的镜像标签是由管道过程作为任务参数提供的。此属性由先前的任务（create-runtime-image）生成，该任务负责在OpenShift镜像流中创建容器镜像。标签被生成，与镜像关联，然后设置为任务的结果属性。这使得Tekton管道能够将该标签作为输入属性传递给图8中显示的configure-kustomization-file任务。本文中描述的任务和流水线资源的GitHub存储库位于此处。构成GitOps流程的不同GitHub存储库的整体结构在此系列的第一篇文章中得到了详细说明，该文章位于此处。图8中的任务显示，该命令是在工作目录的上下文中执行的，该工作目录引用了从myapp-cd GitHub存储库克隆的资产。具体而言，env/overlays/01-dev目录被确定为在开发环境中部署应用程序的部署资源的位置。可以使用参数使这更加灵活，以便团队可以在管道运行资源的一部分中提供特定的存储库和目录信息。

上述目录中的kustomization.yaml文件将以图7所示的格式添加正确的标签。

可以直接使用更新后的kustomization.yaml文件部署应用，使用以下命令：

```
oc apply -k <directory-containing-the-kustimization-file>
```
```
oc apply -k env/overlays/01-dev
```
更符合GitOps模型的做法是将文件提交到GitHub，以便ArgoCD应用程序可以识别变更并通过ArgoCD同步流程部署。这个过程在随后的文章中有详细描述。

### 访问容器镜像
在这个例子中，容器镜像是从名为myapp-ci的项目中的红帽OpenShift镜像流中获取的。部署的命名空间是myapp-development，因此显然部署正在执行的命名空间与包含镜像流的命名空间不同。myapp-ci命名空间用于包含连续集成流水线Tekton资源。以这种方式跨命名空间分割资源为团队提供了一个机会，可以实施严格的基于角色的访问控制，例如，限制开发人员对流水线进行更改，但允许他们在myapp-development命名空间中创建资产。根据每个团队的要求，确切的基于角色的访问控制实施需要一些深思熟虑。在不同的命名空间和不同的GitHub存储库之间分割内容将使任何制度更容易实施。

为了允许myapp-development命名空间从myapp-ci命名空间中拉取镜像，需要一个特定的角色绑定。下面是角色绑定的示例，如图9所示。

```
kind: RoleBinding  
apiVersion: rbac.authorization.k8s.io/v1  
metadata:  
  name: myapp-development  
  namespace: myapp-ci  
subjects:  
  - kind: ServiceAccount  
    name: default  
    namespace: myapp-development  
roleRef:  
  apiGroup: rbac.authorization.k8s.io  
  kind: ClusterRole  
  name: 'system:image-puller'
```
上述角色绑定可以理解为：

在myapp-ci命名空间内创建一个名为myapp-development的新角色绑定，授予myapp-development命名空间内默认服务账户的system:image-puller集群角色。

或者另一种描述方式：

myapp-development命名空间中的默认服务账户具有对myapp-ci命名空间的image-puller权限。

### 可追溯性和镜像标签
为了保持追溯性，这里提供的示例使用了由源代码提交ID的一部分和管道运行标识符的一部分构成的标签。这使用户能够追溯容器镜像到生成可执行文件的源代码，并允许用户查看生成容器镜像的管道流程运行。实际上，为了追溯性，也可以在构建过程中为容器镜像添加包含完整提交ID和管道运行标识符的标签，如果需要的话。由于使用唯一标签是个好主意，而且简单地使用'latest'是不鼓励的，所以这里使用的标签具有实际用途。创建镜像标签的过程包含在任务create-runtime-image的步骤generate-tag-push-to-ocp中。


### 将镜像推送到Quay

为了完成镜像管理过程，需要将镜像从CI命名空间的OpenShift镜像流复制到永久存储解决方案，例如红帽Quay。团队可以在基于云或本地的OpenShift集群上运行Quay的私有实例，或者可以利用Quay在quay.io上托管的实例。当然，还存在其他镜像注册表，但是为了本文的目的，将假定使用Quay。

作为管道流程的一部分，镜像在通过红帽Kubernetes高级集群管理进行漏洞扫描后，可以将镜像推送到Quay。有许多命令可用于将镜像从OpenShift镜像流移动到quay.io，下面描述了其中的两个。


### Buildah
要使用Buildah命令将镜像推送到Quay，将执行以下步骤：

'buildah pull'命令将镜像拉到一个临时本地注册表。
'buildah tag'命令创建一个标签，标识在Quay上的位置。
'buildah push'命令将镜像推送到Quay。

需要使用Buildah命令作为用户或机器用户在Quay上进行身份验证，以便能够将镜像推送到注册表。Quay注册表可以生成Kubernetes secret格式的凭据文件，使得凭据可以存储在CI命名空间中，并使用图10中所示的命令格式进行使用：

```
apiVersion: tekton.dev/v1beta1  
kind: Task  
metadata:  
  name: push-image-to-quay  
spec:  
< parameters are skipped for readability >  
  steps:  
    - name: push-image-to-quay  
      command:  
        - buildah  
        - push  
        - '--storage-driver=$(params.STORAGE_DRIVER)'  
        - '--authfile'  
        - /etc/secret-volume/.dockerconfigjson  
        - '--root'  
        - '/files/buildah-containers'  
        - quay.io/$(params.quay-io-account)/$(params.quay-io-repository):$(params.quay-io-image-tag-name)  
      image: registry.redhat.io/rhel8/buildah  
      resources:  
        requests:  
          memory: 2Gi  
          cpu: '1'  
        limits:  
          memory: 4Gi  
          cpu: '2'  
      volumeMounts:  
        - name: quay-auth-secret-vol  
          mountPath: /etc/secret-volume  
          readOnly: true  
    volumes:  
    - name: quay-auth-secret-vol  
      secret:  
        secretName: quay-auth-secret
```

上述步骤显示，在运行任务的Pod中，从名为quay-auth-secret的Secret的内容挂载了一个卷。挂载到Pod的卷被命名为quay-auth-secret-vol。卷被挂载到路径/etc/secret-volume，因此Secret数据部分中的任何键都将显示为该位置的文件。当使用下面的命令检查Secret时，数据块显示为在名为.dockerconfigjson的键中包含内容。
```
oc get secret/quay-auth-secret -o yaml  
apiVersion: v1  
data:  
  .dockerconfigjson: <secret-data . . . . . . . >  
kind: Secret  
metadata:  
  name: quay-auth-secret
```
Buildah命令有一个参数：
```
--authfile=/etc/secret-volume/.dockerconfigjson
```
上述参数将确保Buildah命令使用存储在Secret中的身份验证令牌。

### skopeo
Buildah进程的另一种选择是使用Skopeo，它可以将镜像从一个注册表复制到另一个注册表，而无需执行拉-重标记-推过程。与Buildah使用的授权文件类似，Skopeo在单个命令中为源注册中心和目标注册中心使用authfile选项。关于使用Skopeo副本的更多信息可以在这里找到:github.com/containers/skopeo/blob/main/docs/skopeo-copy.1.md

### 结束
GitOps进程需要创建具有唯一标记的容器镜像。可以使用允许作为管道一部分的托管更新过程的Kustomize文件以结构化的方式存储这些标记。容器镜像需要仔细管理，虽然从OpenShift镜像流访问它们对于快速开发活动来说是很好的，但更好的做法是将镜像存储在企业镜像注册表中，以便长期使用并部署到生产环境中。