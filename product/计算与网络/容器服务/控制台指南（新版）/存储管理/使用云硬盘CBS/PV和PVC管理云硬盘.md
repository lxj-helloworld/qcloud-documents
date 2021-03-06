## 操作场景
腾讯云容器服务支持通过创建 PV/PVC，并在创建工作负载添加数据卷时使用已有 PVC，实现通过 PV 和 PVC 管理云硬盘。本文介绍如何通过控制台、Kubectl 两种方式实现 PV 和 PVC 管理云硬盘。
>!
>-  云硬盘不支持跨可用区挂载。若挂载云硬盘类型 PV 的 Pod 迁移至其他可用区，将会导致挂载失败。
>- 容器服务控制台不支持云硬盘扩容，可前往 [云硬盘控制台](https://console.cloud.tencent.com/cvm/cbs/index) 进行扩容操作。详情请参见 [扩容云硬盘](https://cloud.tencent.com/document/product/362/5747)。
>






## 操作步骤

### 控制台操作指引

#### 通过控制台创建 StorageClass<span id="create"></span>
由于静态创建云硬盘类型的 PV 时，需要绑定同类型可用 StorageClass，请参考 [创建 StorageClass](https://tcloud-doc.isd.com/document/product/457/44239?!preview&!editLang=zh#create) 完成创建。


#### 静态创建 PV
>? 静态创建 PV 适用于已有存量云盘，并在集群内使用的场景。
>
1. 登录容器服务控制台，选择左侧导航栏中的【[集群](https://console.cloud.tencent.com/tke2/cluster)】。
2. 选择需创建 PV 的集群 ID，进入该集群详情页面。
3. 选择左侧菜单栏中的【存储】>【PersistentVolume】，进入 “PersistentVolume” 页面。如下图所示：
![](https://main.qcloudimg.com/raw/f643eb293a3cbb42073218e478ebc6cf.png)
4. 单击【新建】进入“新建PersistentVolume” 页面，参考以下信息进行创建。如下图所示：
![](https://main.qcloudimg.com/raw/1a08788dced76ea52cb08d6e559545f1.png)
主要参数信息如下：
   - **来源设置**：选择【静态创建】。
   - **名称**：自定义，本文以 `cbs-pv` 为例。
   - **Provisioner**：选择【云硬盘CBS】。
   - **读写权限**：云硬盘仅支持单机读写。
   - **StorageClass**：按需选择合适的 StorageClass。本文以选择在 [通过控制台创建 StorageClass](#create) 步骤中创建的 `cbs-test` 为例。
5. 单击【创建PersistentVolume】即可完成创建。



#### 创建 PVC<span id="createPVC2"></span>
1. 在集群详情页，选择左侧菜单栏中的【存储】>【PersistentVolumeClaim】，进入 “PersistentVolumeClaim” 页面。如下图所示：
![](https://main.qcloudimg.com/raw/df6c4f1d31510fdcfa6cf9d914e8f382.png)
4. 单击【新建】进入 “新建PersistentVolumeClaim” 页面，参考以下信息进行创建。如下图所示：
![](https://main.qcloudimg.com/raw/296aa5fce1ce2e341990419cc26c66e5.png)
主要参数信息如下：
   - **名称**：自定义，本文以 `cbs-pvc` 为例。
   - **命名空间**：选择 “default”。
   - **Provisioner**：选择【云硬盘CBS】。
   - **读写权限**：云硬盘只支持单机读写。
   - **StorageClass**：按需选择合适的 StorageClass。本文以选择在 [通过控制台创建 StorageClass](#create) 步骤中创建的 `cbs-test` 为例。
   - **容量**：根据实际需求进行设置。云硬盘容量最小值由云硬盘产品规格决定，详情请参见 [云硬盘类型](https://cloud.tencent.com/product/cbs/types)。
6. 单击【创建PersistentVolumeClaim】，即可完成创建。
> ? 系统在创建 PVC 时，若发现已有 PV 不足，则将自动创建新的 PV。

#### 创建 Workload 使用 PVC 数据卷
>?该步骤以创建工作负载 Deployment 为例。
>
1. 在“集群管理”页面，选择目标集群 ID，进入待部署 Workload 的集群的 “Deployment” 页面。
2. 单击【新建】，进入“新建Workload” 页面，参考 [创建 Deployment](https://cloud.tencent.com/document/product/457/31705#.E5.88.9B.E5.BB.BA-deployment) 进行创建，并参考以下信息进行数据卷挂载。如下图所示：
![](https://main.qcloudimg.com/raw/168fec34c65f192be3981ae17c4b7144.png)
	- **数据卷（选填）**：
		- **挂载方式**：选择“使用已有PVC”。
		- **数据卷名称**：自定义，本文以 `cbs-vol` 为例。
		- **选择 PVC**：选择在步骤 [创建 PVC](#createPVC2) 中已创建的 “cfs-pvc”。
	- **实例内容器**：单击【添加挂载点】，进行挂载点设置。
       - **数据卷**：选择该步骤中已添加的数据卷 “cbs-vol”。
       - **目标路径**：填写目标路径，本文以 `/cache` 为例。
       - **挂载子路径**：仅挂载选中数据卷中的子路径或单一文件。例如，`/data` 或 `/test.txt`。
4. 单击【创建Workload】即可完成创建。
 > ! 如使用 PVC 挂载模式，则数据卷只能挂载到一台 Node 主机上。

### Kubectl 操作指引

您可通过以下 YAML 示例文件，使用 Kubectl 进行创建操作。


#### （可选）创建 PV<span id="createPV"></span>

可以通过已有云硬盘创建 PV，也可以直接 [创建 PVC](#createPVC) ，系统将自动创建对应的 PV。YAML 文件示例如下：
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: nginx-pv
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  qcloudCbs:
      cbsDiskId: disk-xxxxxxx ## 指定已有的CBS id
      fsType: ext4
  storageClassName: cbs
```



#### 创建 PVC<span id="createPVC"></span>

若未 [创建 PV](#createPV)，则在创建 PVC 时，系统将自动创建对应的 PV。YAML 文件示例如下：
```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: nginx-pv-claim
spec:
  storageClassName: cbs
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
```
- 普通云盘大小必须是10的倍数，最小为10GB。
- 高性能云硬盘最小为50GB。
- SSD 云硬盘最小为100GB，详情请参见 [云硬盘类型](https://cloud.tencent.com/product/cbs/types)。

#### 使用 PVC

可通过创建 Workload 使用 PVC 数据卷。YAML 示例如下：
```yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      qcloud-app: nginx-deployment
  template:
    metadata:
      labels:
        qcloud-app: nginx-deployment
    spec:
      containers:
      - image: nginx
        imagePullPolicy: Always
        name: nginx
        volumeMounts:
        - mountPath: "/opt/"
          name: pvc-test
      volumes:
      - name: pvc-test
        persistentVolumeClaim:
          claimName: nginx-pv-claim # 已经创建好的 PVC
```
