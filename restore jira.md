# HƯỚNG DẪN RESTORE JIRA 8.18.1 TRÊN DOCKER

> **Tài liệu tổng hợp từ quá trình test restore thực tế**
> Ngày: 14/04/2026

---

## MỤC LỤC

1. Yêu cầu trước khi bắt đầu
2. Chuẩn bị file backup
3. Clean database trước khi restore
4. Copy file backup vào container
5. Restore qua giao diện Jira
6. Fix lỗi License sau restore
7. Fix lỗi đăng nhập (reset password + CAPTCHA)
8. Re-index
9. Cấu hình sau restore
10. Copy Attachments từ Jira cũ
11. Cài lại Plugins

---

## 1. YÊU CẦU TRƯỚC KHI BẮT ĐẦU

- Jira 8.18.1 mới đã được cài đặt và chạy trên Docker (đã qua setup wizard)
- File backup `.zip` từ Jira cũ (chứa `activeobjects` và `entities`)
- Quyền truy cập SSH vào server Docker
- License Jira Software hợp lệ cho Server ID mới
- License các plugin (nếu có): Oboard OKR, Tempo, eazyBI, v.v.
- Tài khoản admin và mật khẩu của Jira cũ

### Thông tin hạ tầng Docker (ví dụ)

```yaml
# docker-compose.yml
services:
  jira-db:
    image: postgres:13-alpine
    container_name: jira-production-db
    environment:
      POSTGRES_DB: jiradb
      POSTGRES_USER: jira
      POSTGRES_PASSWORD: <password>
    volumes:
      - pgdata:/var/lib/postgresql/data

  jira:
    container_name: jira-production
    ports:
      - "8080:8080"
    environment:
      - ATL_JDBC_URL=jdbc:postgresql://jira-db:5432/jiradb
      - ATL_JDBC_USER=jira
      - ATL_JDBC_PASSWORD=<password>
      - ATL_DB_DRIVER=org.postgresql.Driver
      - ATL_DB_TYPE=postgres72
      - JVM_MINIMUM_MEMORY=2g
      - JVM_MAXIMUM_MEMORY=4g
    volumes:
      - jiradata:/var/atlassian/application-data/jira
```

---

## 2. CHUẨN BỊ FILE BACKUP

File export từ Jira cũ gồm 2 file: `activeobjects` và `entities`. Cần đảm bảo chúng nằm trong 1 file `.zip`:

```
jira-backup-20260414.zip
  ├── activeobjects.xml
  └── entities.xml
```

Copy file backup lên server mới:

```bash
scp jira-backup-20260414.zip user@server-moi:/tmp/
```

---

## 3. CLEAN DATABASE TRƯỚC KHI RESTORE

> **QUAN TRỌNG:** Bước này đảm bảo restore sạch, tránh lỗi ActiveObjects foreign key constraint.

### 3.1 Stop Jira

```bash
sudo docker stop jira-production
```

### 3.2 Drop và tạo lại database

```bash
sudo docker exec -it jira-production-db psql -U jira -d postgres \
  -c "DROP DATABASE jiradb;"

sudo docker exec -it jira-production-db psql -U jira -d postgres \
  -c "CREATE DATABASE jiradb OWNER jira ENCODING 'UTF8' LC_COLLATE='C' LC_CTYPE='C' TEMPLATE=template0;"
```

### 3.3 Xóa cache trong Jira Home

```bash
sudo docker start jira-production

sudo docker exec -u root jira-production \
  rm -rf /var/atlassian/application-data/jira/caches/*

sudo docker exec -u root jira-production \
  rm -rf /var/atlassian/application-data/jira/plugins/.osgi-plugins/*
```

### 3.4 Restart Jira

```bash
sudo docker restart jira-production
```

Chờ Jira khởi động. Vì DB trống, Jira sẽ hiện **setup wizard** → chạy qua setup wizard bình thường.

---

## 4. COPY FILE BACKUP VÀO CONTAINER

### 4.1 Copy file

```bash
sudo docker cp /tmp/jira-backup-20260414.zip \
  jira-production:/var/atlassian/application-data/jira/import/
```

### 4.2 Phân quyền (dùng -u root)

```bash
sudo docker exec -u root jira-production \
  chown jira:jira /var/atlassian/application-data/jira/import/jira-backup-20260414.zip
```

### 4.3 Kiểm tra

```bash
sudo docker exec jira-production \
  ls -la /var/atlassian/application-data/jira/import/
```

Phải thấy file với owner `jira jira`.

---

## 5. RESTORE QUA GIAO DIỆN JIRA

1. Đăng nhập Jira với tài khoản admin (tài khoản setup wizard)
2. Vào **⚙️ Administration → System → Import and Export → Restore System**
3. Điền thông tin:
   - **File name**: `jira-backup-20260414.zip`
   - **License**: Paste license Jira Software mới (generate cho Server ID mới)
   - **Outgoing mail**: Chọn **Disable**
4. Click **Restore**
5. Theo dõi log:

```bash
sudo docker logs -f jira-production
```

> **LƯU Ý:** Restore sẽ **ghi đè toàn bộ database**. Sau restore, tài khoản setup wizard sẽ không dùng được nữa — phải dùng tài khoản admin từ Jira cũ.

---

## 6. FIX LỖI LICENSE SAU RESTORE

Nếu sau restore bị lỗi 500 với log `Unable to parse license`, cần xóa license cũ trong DB:

### 6.1 Xóa license trong DB

```bash
sudo docker exec -it jira-production-db psql -U jira -d jiradb
```

```sql
-- Xem license hiện tại
SELECT id, license FROM productlicense;

-- Xóa license cũ
DELETE FROM productlicense;

\q
```

### 6.2 Restart Jira

```bash
sudo docker restart jira-production
```

Jira sẽ hiện trang nhập license → paste license Jira Software mới vào.

---

## 7. FIX LỖI ĐĂNG NHẬP

Sau restore, cần đăng nhập bằng **tài khoản admin từ Jira cũ**. Nếu không đăng nhập được:

### 7.1 Generate hash password đúng

```bash
sudo docker exec -u root jira-production bash -c '
cat > /tmp/GenPass.java << "EOF"
import com.atlassian.crowd.password.encoder.AtlassianSecurityPasswordEncoder;
public class GenPass {
    public static void main(String[] args) {
        AtlassianSecurityPasswordEncoder encoder = new AtlassianSecurityPasswordEncoder();
        System.out.println(encoder.encodePassword("admin", null));
    }
}
EOF
cd /tmp && javac -cp "/opt/atlassian/jira/atlassian-jira/WEB-INF/lib/*" GenPass.java \
  && java -cp ".:/opt/atlassian/jira/atlassian-jira/WEB-INF/lib/*" GenPass'
```

Ghi lại hash output (ví dụ: `{PKCS5S2}1aZ/o8jZ92nm...`)

### 7.2 Tìm user admin

```bash
sudo docker exec -it jira-production-db psql -U jira -d jiradb
```

```sql
SELECT u.user_name, u.display_name
FROM cwd_user u
JOIN cwd_membership m ON u.id = m.child_id
JOIN cwd_group g ON m.parent_id = g.id
WHERE g.group_name = 'jira-administrators'
LIMIT 20;
```

### 7.3 Reset password

```sql
-- Thay 'adminjira' bằng username thực và hash bằng output từ bước 7.1
UPDATE cwd_user
SET credential = '{PKCS5S2}1aZ/o8jZ92nmmeNQN4vil9CXLbq06Rh//IrMHewfGW6jRwO6Mesr10HZxFkdA5P5'
WHERE user_name = 'adminjira';
```

### 7.4 Reset CAPTCHA / failed login cho TẤT CẢ user

```sql
UPDATE cwd_user_attributes
SET attribute_value = '0', lower_attribute_value = '0'
WHERE attribute_name = 'login.currentFailedCount';

UPDATE cwd_user_attributes
SET attribute_value = '0', lower_attribute_value = '0'
WHERE attribute_name = 'invalidPasswordAttempts';

UPDATE cwd_user_attributes
SET attribute_value = '0', lower_attribute_value = '0'
WHERE attribute_name = 'login.totalFailedCount';

\q
```

### 7.5 Tắt CAPTCHA qua config

```bash
sudo docker exec -u root jira-production bash -c \
  'echo "jira.websudo.is.disabled = true" >> /var/atlassian/application-data/jira/jira-config.properties'

sudo docker exec -u root jira-production bash -c \
  'echo "jira.show.captcha.on.login=false" >> /var/atlassian/application-data/jira/jira-config.properties'
```

### 7.6 Restart và test

```bash
sudo docker restart jira-production

# Chờ Jira ready, test login qua API
curl -s -o /dev/null -w "%{http_code}" -u adminjira:admin \
  http://<JIRA_IP>:8080/rest/api/2/myself
```

Kết quả **200** = đăng nhập thành công. Vào web đăng nhập với username/password đã set.

---

## 8. RE-INDEX

Sau khi đăng nhập thành công:

1. Vào **Administration → System → Indexing**
2. Chọn **Full re-index**
3. Click **Re-index**

> Nếu re-index báo lỗi ThreadPoolExecutor, restart Jira rồi thử lại:
> ```bash
> sudo docker restart jira-production
> ```

---

## 9. CẤU HÌNH SAU RESTORE

### 9.1 Cập nhật Base URL

**Administration → System → General Configuration → Edit Settings**
→ Đổi Base URL sang `http://<IP_MỚI>:8080`

### 9.2 Đổi mật khẩu admin

Profile → Security → Change Password → đổi từ `admin` sang mật khẩu mạnh.

### 9.3 Kiểm tra data

- Vào **Projects** → xem danh sách project có đầy đủ không
- Vào **Issues** → search issues
- Vào **Administration → User Management** → xem user có đủ không
- Vào **Administration → Issues → Workflows** → xem workflow có đúng không

---

## 10. COPY ATTACHMENTS TỪ JIRA CŨ

XML backup **không chứa file attachments**. Cần copy thủ công:

### 10.1 Trên server Jira cũ

```bash
cd /var/atlassian/application-data/jira/data
tar -czf /tmp/jira-attachments.tar.gz attachments/
```

### 10.2 Copy sang server mới

```bash
scp user@jira-cu:/tmp/jira-attachments.tar.gz /tmp/
```

### 10.3 Copy vào container và giải nén

```bash
sudo docker cp /tmp/jira-attachments.tar.gz jira-production:/tmp/

sudo docker exec -u root jira-production bash -c \
  'cd /var/atlassian/application-data/jira/data && tar -xzf /tmp/jira-attachments.tar.gz'

sudo docker exec -u root jira-production \
  chown -R jira:jira /var/atlassian/application-data/jira/data/attachments
```

### 10.4 Copy thêm avatars/logos (nếu có)

```bash
# Trên Jira cũ
tar -czf /tmp/jira-avatars.tar.gz avatars/ logos/

# Copy và giải nén tương tự như attachments
```

---

## 11. CÀI LẠI PLUGINS

Sau restore, cần cài lại các plugin đúng version như Jira cũ.

### 11.1 Xem danh sách plugin cần cài

Trên Jira cũ → **Administration → Manage Apps** → ghi lại tên + version tất cả plugin.

### 11.2 Cài plugin trên Jira mới

**Cách 1: Qua Marketplace** (nếu Jira mới có internet)
→ Administration → Manage Apps → Find new apps → tìm và cài

**Cách 2: Upload file .jar** (nếu không có internet)
→ Tải .jar từ Marketplace → Administration → Manage Apps → Upload app

### 11.3 Nhập license cho từng plugin

Mỗi plugin cần license riêng. Vào [my.atlassian.com](https://my.atlassian.com) → generate license cho Server ID mới.

### 11.4 Danh sách plugin phổ biến cần cài (ví dụ)

| Plugin | Chức năng |
|--------|-----------|
| Oboard OKR | Quản lý OKR |
| Tempo Timesheets | Time tracking |
| ActivityTimeline | Resource planning |
| AIO Tests | Test management |
| Smart QL | Advanced querying |
| Insight | Asset management |
| WBS Gantt-Chart | Gantt chart |
| eazyBI | Reporting & BI |
| SQL Connector | DB connector |

> **LƯU Ý:** Data của plugin (OKR, Tempo...) đã được restore trong bảng ActiveObjects. Chỉ cần cài đúng plugin + version, data sẽ tự hiển thị.

---

## CHECKLIST SAU RESTORE

- [ ] Jira khởi động bình thường, không lỗi 500
- [ ] License Jira Software hợp lệ
- [ ] Đăng nhập bằng tài khoản admin cũ thành công
- [ ] Re-index thành công
- [ ] Base URL đã cập nhật
- [ ] Mật khẩu admin đã đổi sang mật khẩu mạnh
- [ ] Projects hiển thị đầy đủ
- [ ] Issues/data hiển thị đúng
- [ ] Users và permissions đúng
- [ ] Workflows đúng
- [ ] Attachments hiển thị (đã copy từ Jira cũ)
- [ ] Plugins đã cài đủ + có license
- [ ] Plugin data (OKR, Tempo...) hiển thị đúng
- [ ] Outgoing mail đã Disable (trong lúc test)
- [ ] Outgoing mail Enable lại khi go-live

---

## XỬ LÝ SỰ CỐ

| Lỗi | Nguyên nhân | Cách fix |
|-----|-------------|----------|
| 500 - Unable to parse license | License trong DB không hợp lệ | Xóa `DELETE FROM productlicense;` → restart → nhập license mới |
| ActiveObjects foreign key error | DB chưa clean trước restore | Clean DB (bước 3) → restore lại |
| Login failed + CAPTCHA | Failed login quá nhiều | Reset `cwd_user_attributes` + tắt CAPTCHA trong config |
| 401 Unauthorized (API) | Password hash sai | Generate hash đúng bằng Java trong container (bước 7.1) |
| 403 Forbidden (API) | Password đúng nhưng bị block | Reset failed login count trong DB (bước 7.4) |
| Re-index failed ThreadPool | Jira cần restart | `sudo docker restart jira-production` → re-index lại |
| Attachments không hiển thị | Thiếu file attachments | Copy từ Jira cũ (bước 10) |
| Hiện "Jira Core" thay vì "Jira Software" | Thiếu license Jira Software | Nhập license Jira Software |
| Plugin data không hiển thị | Chưa cài plugin | Cài đúng plugin + version |
