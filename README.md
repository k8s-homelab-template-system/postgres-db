# postgres-db

Helm chart chạy PostgreSQL trên Kubernetes. Chart này không tự viết template mà dùng lại chart [`bitnami/postgresql`](https://artifacthub.io/packages/helm/bitnami/postgresql) (18.12.4, lấy từ `oci://registry-1.docker.io/bitnamicharts`) làm dependency.

Cấu hình mặc định (xem [values.yaml](values.yaml)):

- PostgreSQL 18, chạy 1 instance (`standalone`), ổ 5Gi dùng StorageClass mặc định
- Database `app`, user `app`
- Không bật metrics

> **Lưu ý về image:** từ 2025 Bitnami chỉ phát hành miễn phí tag `latest` của image `bitnami/postgresql`, nên chart này đang chạy `postgresql:latest`. Image sẽ thay đổi mỗi khi pod được tạo lại. Nếu cần cố định, hãy ghi đè `postgresql.image.digest` bằng digest cụ thể.

## Cài đặt

```bash
helm dependency build .
helm upgrade --install postgres-db . --namespace homelab --create-namespace
```

Thường thì chart này được cài qua umbrella chart `scs` chứ không cài riêng.

## Kết nối

Với release tên `<release>`, chart tạo:

| Resource | Tên |
| --- | --- |
| Service | `<release>-postgresql` (port 5432) |
| Secret | `<release>-postgresql-auth` (key `password` cho user `app`, `postgres-password` cho user `postgres`) |

Lấy mật khẩu của user `app`:

```bash
kubectl get secret <release>-postgresql-auth -n homelab -o jsonpath='{.data.password}' | base64 -d
```

## Credential

Mật khẩu nằm trong Secret `<release>-postgresql-auth`, do chart này tạo ([templates/auth-secret.yaml](templates/auth-secret.yaml)). Không có credential nào trong git.

- Sinh ngẫu nhiên khi cài lần đầu, giữ nguyên khi upgrade (`lookup` Secret đang có).
- Nếu đã có Secret cũ `<release>-postgresql` (chart <= 0.2.x), mật khẩu được lấy lại từ đó để khớp dữ liệu trên PVC.
- Có annotation `helm.sh/resource-policy: keep`: gỡ release vẫn giữ Secret, cài lại dùng đúng mật khẩu cũ.

Đổi mật khẩu: sửa Secret rồi đổi mật khẩu trong PostgreSQL (`ALTER USER app PASSWORD ...`) cho khớp, sau đó restart pod.

Tự quản lý Secret (kubectl, Sealed Secrets, External Secrets...): đặt `authSecret.create: false` và tạo Secret tên `<release>-postgresql-auth` với 2 key trên, hoặc trỏ `postgresql.auth.existingSecret` sang tên khác.
