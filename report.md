# Báo cáo Bài 3: Tối ưu tốc độ CI bằng Gradle Dependency Caching

## 1. Cấu hình đã thay đổi

Trong `.github/workflows/ci.yml`, bước `actions/setup-java@v5` được bổ sung tham số:

```yaml
- name: Thiết lập môi trường Java JDK 17
  uses: actions/setup-java@v5
  with:
    java-version: '17'
    distribution: 'temurin'
    cache: 'gradle'
```

Cache key được tạo từ hash của các file `*.gradle*`, `gradle-wrapper.properties` (có dạng `setup-java-Linux-x64-gradle-<hash>`). Khi các file này không đổi, các lần chạy sau sẽ khôi phục cache thay vì tải lại dependency từ Internet.

## 2. Cách thực hiện

1. Push commit 1 (chưa có cache): workflow tải toàn bộ dependency, cuối job lưu cache.
2. Push commit 2 (không đổi file build): workflow khôi phục cache.
3. Xem thời gian từng bước ở tab **Actions** → chọn run → chọn job.

## 3. Kết quả so sánh (điền số liệu thực tế từ tab Actions)

| Chỉ số | Lần 1 (chưa cache) | Lần 2 (đã cache) | Chênh lệch |
|--------|--------------------|------------------|------------|
| Bước `Set up JDK 17` | ... s | ... s | ... |
| Bước `./gradlew test` | ... s | ... s | ... |
| Post `Set up JDK 17` (lưu cache) | ... s | ... s | ... |
| Tổng thời gian Job | ... s | ... s | ... |

### Dòng log xác nhận

- Lần 1: `Cache not found for input keys: setup-java-Linux-x64-gradle-...`
- Lần 2: `Cache restored successfully` / `Cache restored from key: setup-java-Linux-x64-gradle-...`

(Dán ảnh chụp màn hình log hoặc trích log thực tế tại đây.)

## 4. Nhận xét

- Lần 1: Bước `./gradlew test` mất nhiều thời gian vì phải tải Gradle distribution và các thư viện từ Maven Central; sau job, cache được lưu ở bước Post.
- Lần 2: Cache được khôi phục ở bước `setup-java`, `./gradlew test` không cần tải lại dependency nên chạy nhanh hơn. Thời gian khôi phục cache thường chỉ vài giây.
- Cache sẽ bị vô hiệu khi file Gradle (`build.gradle`, `settings.gradle`, `gradle-wrapper.properties`, `libs.versions.toml`...) thay đổi.
