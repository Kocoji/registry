# Giới thiệu
Tổng quan bài tập này được dựng và sử dụng trên Kind-Cluster sử dụng nginx ingress.

*Lưu ý*: Chart nằm trong mục đích test - không thuộc production, vậy nên giới hạn hiện tại sẽ là:
- Chỉ sử dụng volume hostPath với path là: `/tmp` của worker.
- Hiện tại không bao gồm generate TLS/SSL vào secret, sử dụng trong Ingress.

Dựa vào đề bài helm template sử dụng các CRDs/options:
- Deployment với initContainer 
- Service (clusterIP)
- CronJobs: Mục đích chạy Garbage collection


# Sử dụng Helm Chart 

Values được đặt với các cấu hình tùy chỉnh như sau.
```yaml
redis:
  enabled: true
  # replicaCount: 1
  # memoryLimit: 2Gi
  # cpuLimit: "1"

registry:
  replicaCount: 1
  # username: 
  # password: 
  maxUploadSize: 30m # maximum upload file
  host: registry.local
  garbageCron: 0 1 * * * # set the time to run docker image garbage collection, default is run daily at 00:00 AM
  # memoryLimit: 2Gi
  # cpuLimit: "1"
```

- Trong đó, cho phép bạn bật tắt tùy chọn Redis, nếu không sử dụng, service mặc định sử dụng `inmemory`
- `username`/`password` sử dụng trong basic auth (mặc định sẽ là admin/admin nếu không được cấu hình). 
- Set số lượng Rep qua key `replicaCount`. 
- Cấu hình hostname sử dụng trong nginx Ingress và cấu hình maximum file có thể tải lên với key:`maxUploadSize`.
- Resource limit với 2 keys: `memoryLimit`, `cpuLimit` mặc định limit ở mức 512MB, 0.5CPU.
- `garbageCron` - Thời gian chạy cron có thể tham khảo tại: https://crontab.guru/ (mặc định sẽ chạy mỗi ngày `0 0 * * *` UTC (7h sáng VN))

# Sử dụng chart

Có thể sử dụng bằng cách tải xuống file trong mục release, vd: sử dụng file release `registry-0.1.1.tgz`:

```sh
tar -zxvf registry-0.1.1.tgz
```
Lệnh trên sẽ giải nén thư mục mới có tên `registry` tại path giải nén.

Sử dụng lệnh helm install để cài dặt, vd:
```sh
helm install --namespace {namespace_name} --create-namespace {service_name} registry 
```

bạn có thể sử dụng:

```sh
helm repo add registry-server https://kocoji.github.io/registry
```
Sau đó cài đặt

```sh
helm install {service_name} registry-server/registry --namespace {namespace_name} --create-namespace --values {path/to/values.file}
```