---
---
# Abstract

Mục đích ban đầu của [[Kubernetes]] là để triển khai các stateless app. Nhưng sau đó nhờ sự hỗ trợ của cộng đồng, nhiều tính năng mới được phát triển. Storage tách riêng tài nguyên lưu trữ ra khỏi các tài nguyên tính toán. Nếu như không có storage, thì mỗi khi pod hay service bị kill, các thông tin được lưu trữ trong pods/servcies sẽ bị mất theo. Điều này chỉ thực sự phù hợp cho các stateless application. Storage có khả năng lưu trữ persistent data, khiến nó có thể được backup và phục hồi mỗi khi có sự cố liên quan đến pod.

# Glossaries

- CSI - Container Storage Interface. 
- Volume: 
- Persistent Volume
- Persistent Volume Claim (PVC): chỉ là **request** không ảnh hưởng 

## Volume

Volume trong kubernetes cũng khá giống với volume trong [[Virtualization/Container/Docker]], tuy nhiên volume của kubernetes có nhiều điểm nổi bật hơn. 
Về cơ bản thì volume là 1 directory, có thể chứa data bên trong. Mỗi pod muốn sử dụng volume thì phải mount volume đó vào 1 path trong file system của nó. Volume nằm trong pod specification, tức là life cylce của volume phụ thuộc vào pod, nó có nhiệm vụ là tách storage ra khỏi container chứ ko phải tách storage ra khỏi pod. Mục đích chính của volume là đảm bảo data sẽ không bị mất khi restart container và sharring data giữa các container trong cùng pod. VÌ vậy mà thường thì volume sẽ không được sử dụng nhiều so với Persistent volume.

### Types of volumes

Kubernetes là 1 sản phẩm open source, có cung cấp interface cho storage -> có rất nhiều loại volume được các vendor họ tạo ra để kinh doanh các sản phẩm của mình, có thể kể đến các loại Volume như:
- AWS EBS CSI Migration, sẽ sử dụng [[AWS]] EBS để lưu trữ volume data.
- azureDisk CSI Migration, sử dụng [[Azure]] Disk để lưu trữu volume.
- cephfs
- ...

Ở đây, chúng ta chỉ nói đến những giải pháp free hoặc open source, có thể deploy ở bất kì đây (trên cloud hay on-premise) và có thể được sử dụng ở hiện tại ([[2023-01-31]]), chưa bị deprecated

#### emptyDir
emptyDir sẽ được tạo trước tiên khi ta tạo pod trên 1 node, và nó chỉ tồn tại khi pod đó tồn tại trên node đó. Các container trong cùng pod có thể đọc ghi vào cùng file trong emptyDir. Và mỗi container có thể mount emptyDir ra 1 path khác nhau.

> Chú ý: emtyDir ở level node và pod. Khi pod bị crash thì emtyDir sẽ không mất đi. Chỉ khi pod bị remove khỏi node thì emtyDir mới bị xóa theo.

Các use cases của emptyDir:
- scratch space, such as for a disk-based merge sort
- checkpointing a long computation for recovery from crashes
- holding files that a content-manager container fetches while a webserver container serves the data

Ta có thể set giá trị cho `emptyDir.medium` để xác định vị trí lưu data. Mặc định là emptyDir sẽ lưu vào bất cứ thiết bị lưu trữ nào có trên node như: HDD, [[Hard disk]], [[NAS]], ... Ngoài ra có thể lưu trên [[RAM]] bằng cách set là `memory`

Example config:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-pd
spec:
  containers:
  - image: registry.k8s.io/test-webserver
    name: test-container
    volumeMounts:
    - mountPath: /cache
      name: cache-volume
  volumes:
  - name: cache-volume
    emptyDir:
      sizeLimit: 500Mi
```

#### fc (fiber channel)
#### OpenStack CSI migration
#### downwardAPI
#### hostPath
#### iscsi
#### local
#### nfs
### Using subPath

## Persistent volumes

Persistent volume là 1 piece of storage, được cung cấp bởi administrator hay dynamic provisioning bởi storage class. Theo 1 cách đơn giản, PV chính là phần storage được cung cấp cho pod, services để sử dụng. Nó có thể là local file system trên node chứa pod hay 1 loại storage khác như object store hay block store. 

## Projected volumes


## Storage Class

Storage Class là các loại lưu trữ mà các container có thể sử dụng

Storage class cần defined được các thông tin như sau:
- Provisioner
- parameter
- reclaimPolicy

# References
1. [Kubernetes storage is complex, but it’s getting better | Cloud Native Computing Foundation (cncf.io)](https://www.cncf.io/blog/2023/03/28/kubernetes-storage-is-complex-but-its-getting-better/)