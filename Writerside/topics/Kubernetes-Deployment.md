# Kubernetes Deployment
* EKS를 사용하지 않고 실제 인스턴스에 k8s를 클러스터 설치할 수 있음
  * [kOps Documentation](https://kubernetes.io/ko/docs/setup/production-environment/tools/kops/)
## AWS EKS (Elastic Kubernetes Service)
* EKS를 사용하면 내부적으로 EC2 인스턴스를 활용
* 클러스터 명을 설정하고 IAM Role설정 후 클러스터 설정
* [EKS Documentation](https://docs.aws.amazon.com/eks/latest/userguide/creating-a-vpc.html#create-vpc)

### kubectl
* kubectl 명령어는 `.kube/config` 내부 파일을 확인 후 어느 클러스터에 명령을 보낼지 선택
```yaml
apiVersion: v1
clusters:
  - cluster:
      certificate-authority: ~xxx/ca.crt
      server: https://xxx
    name: {}
contexts:
  - context:
      cluster: {}
      user: {}
    name: {}
current-context: {}
kind: Config
users:
  - name: {}
    user:
      client-certificate: ~xxx/client.crt
      client-key: ~xxx/client.key
```
* [AWS CLI Documentation](https://aws.amazon.com/cli/?nc1=h_ls)
  * Security Credential 에서 Access Key 발급 후 `aws configure` 명령 수행하여 설정
  * `aws eks --region {region} update-kubeconfig --name {clusterName}` 수행하면 config 설정이 바뀜
  * 이후 `kubectl` 명령어는 EKS 클러스터로 전달

## Volume
* [EFS CSI Driver Documentation](https://github.com/kubernetes-sigs/aws-efs-csi-driver?tab=readme-ov-file#upgrading-the-amazon-efs-csi-driver)
### EFS
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: efs-sc
provisioner: efs.csi.aws.com
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: efs-pv
spec:
  capacity:
    storage: 5Gi
  volumeMode: Filesystem
  accessModes:
    -ReadWriteMany
  storageClassName: efs-sc
  csi:
    driver: efs.csi.aws.com
    volumeHandle: {efs name}
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: efs-pvc
spec:
  accessModes:
    - ReadWriteMany
  storageClassName: efs-sc
  resources:
    requests:
      storage: 5Gi
```