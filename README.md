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
| Secret | `<release>-postgresql` (các key `password` cho user `app`, `postgres-password` cho user `postgres`) |

Lấy mật khẩu của user `app`:

```bash
kubectl get secret <release>-postgresql -n homelab -o jsonpath='{.data.password}' | base64 -d
```

Mật khẩu được sinh ngẫu nhiên khi cài lần đầu. Để tránh bị đổi khi upgrade, hãy tạo Secret trước rồi trỏ tới nó bằng `postgresql.auth.existingSecret`.
