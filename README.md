# postgres-db

Helm chart chạy PostgreSQL trên Kubernetes bằng [CloudNativePG](https://cloudnative-pg.io). Chart này không tự viết template mà dùng lại chart có sẵn [`cnpg/cluster`](https://github.com/cloudnative-pg/charts/tree/main/charts/cluster) (0.9.0) làm dependency.

Cấu hình mặc định (xem [values.yaml](values.yaml)):

- PostgreSQL 17, 1 instance, ổ 5Gi (StorageClass mặc định)
- Database `app`, owner `app`
- Không bật superuser, backup và monitoring

## Yêu cầu

Cluster phải có sẵn operator CloudNativePG, vì chart này chỉ tạo resource `Cluster`:

```bash
helm repo add cnpg https://cloudnative-pg.github.io/charts
helm upgrade --install cnpg cnpg/cloudnative-pg --version 0.29.1 \
  --namespace cnpg-system --create-namespace
```

## Cài đặt

```bash
helm dependency build .
helm upgrade --install postgres-db . --namespace homelab --create-namespace
```

Thường thì chart này được cài qua umbrella chart `scs` chứ không cài riêng.

## Kết nối

Với release tên `<release>`, CloudNativePG tạo:

| Resource | Tên |
| --- | --- |
| Service đọc/ghi (primary) | `<release>-cluster-rw` |
| Service chỉ đọc | `<release>-cluster-ro` |
| Secret user `app` | `<release>-cluster-app` (các key `username`, `password`, `uri`, ...) |

Lấy mật khẩu:

```bash
kubectl get secret <release>-cluster-app -n homelab -o jsonpath='{.data.password}' | base64 -d
```

Kiểm tra kết nối:

```bash
helm test <release> -n homelab
```
