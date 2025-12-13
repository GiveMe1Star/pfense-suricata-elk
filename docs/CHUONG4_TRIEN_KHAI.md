# Chương 4: Triển khai và Cấu hình thủ công (Bare Metal/Manual)

Tài liệu này hướng dẫn triển khai pfELK thủ công trên máy chủ Linux mà **không dùng Docker và không chạy lệnh với GitHub**. Các cấu hình được viết trực tiếp từ nội dung trong repository (thư mục `etc/pfelk`).

## 4.1. Chuẩn bị môi trường

**Máy chủ ELK**
- Hệ điều hành: Ubuntu Server 20.04 LTS hoặc Debian 11.
- Quyền hạn: `root` hoặc `sudo`.
- Gói phụ thuộc: `wget`, `curl`, `gnupg`, `apt-transport-https`, Java (OpenJDK 11 hoặc 17).

**Mã nguồn cấu hình**
- Đảm bảo thư mục chứa cấu hình có sẵn trên máy (ví dụ: `/opt/pfense-suricata-elk`).
- Không sử dụng lệnh tải từ GitHub; sao chép thủ công (USB/SCP) toàn bộ thư mục `etc/pfelk` vào máy.

## 4.2. Cài đặt và cấu hình Elasticsearch

1. **Thêm kho lưu trữ Elastic**
   ```bash
   wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
   echo "deb https://artifacts.elastic.co/packages/8.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-8.x.list
   sudo apt update
   ```
2. **Cài Elasticsearch**
   ```bash
   sudo apt install elasticsearch
   ```
3. **Cấu hình** tại `/etc/elasticsearch/elasticsearch.yml`:
   - `cluster.name: pfelk`
   - `network.host: 0.0.0.0` (hoặc IP LAN của máy chủ)
4. **Điều chỉnh bộ nhớ JVM** trong `/etc/elasticsearch/jvm.options` (ví dụ 4GB):
   ```
   -Xms4g
   -Xmx4g
   ```
5. **Khởi động**
   ```bash
   sudo systemctl enable elasticsearch
   sudo systemctl start elasticsearch
   ```

## 4.3. Cài đặt và cấu hình Kibana

1. **Cài Kibana**
   ```bash
   sudo apt install kibana
   ```
2. **Chỉnh `/etc/kibana/kibana.yml`**:
   - `server.port: 5601`
   - `server.host: "0.0.0.0"`
   - `elasticsearch.hosts: ["http://localhost:9200"]`
3. **Khởi động**
   ```bash
   sudo systemctl enable kibana
   sudo systemctl start kibana
   ```

## 4.4. Cài đặt và cấu hình Logstash

> Toàn bộ cấu hình được viết thủ công dựa trên thư mục `etc/pfelk` sẵn có; không dùng lệnh tải từ GitHub.

1. **Cài Logstash**
   ```bash
   sudo apt install logstash
   ```
2. **Triển khai pipeline (`/etc/logstash/conf.d`)**
   - Xóa cấu hình mặc định nếu cần:
     ```bash
     sudo rm -f /etc/logstash/conf.d/*
     ```
   - Tạo từng file `.pfelk` với nội dung tương ứng trong thư mục `etc/pfelk/conf.d` của repo (dùng trình soạn thảo như `nano`/`vi`). Các file chính:
     - `01-inputs.pfelk`: mở cổng 5140 (TCP/UDP) nhận log từ pfSense.
     - `02-firewall.pfelk`: bộ lọc firewall sử dụng grok patterns.
     - `30-geoip.pfelk`: enrich dữ liệu GeoIP.
     - `50-outputs.pfelk`: xuất dữ liệu sang Elasticsearch.
3. **Tạo thư mục patterns**
   ```bash
   sudo mkdir -p /etc/logstash/patterns
   ```
   Sao chép nội dung từ `etc/pfelk/patterns/pfelk.grok` trong repo và lưu thành `/etc/logstash/patterns/pfelk.grok`.
4. **Tạo thư mục databases**
   ```bash
   sudo mkdir -p /etc/logstash/databases
   ```
   Viết lại (copy thủ công) các file CSV từ `etc/pfelk/databases/` vào `/etc/logstash/databases/`, gồm `service-names-port-numbers.csv`, `rule-names.csv`, v.v.
5. **Phân quyền và khởi động**
   ```bash
   sudo chown -R logstash:logstash /etc/logstash
   sudo systemctl enable logstash
   sudo systemctl start logstash
   ```

## 4.5. Cấu hình pfSense và Suricata gửi log

**pfSense – Remote Syslog**
1. Đăng nhập pfSense WebGUI → **Status → System Logs → Settings**.
2. Bật **Enable Remote Logging**.
3. Nhập **Remote Log Servers**: `IP_ELK:5140` (theo `01-inputs.pfelk`).
4. Chọn **Remote Syslog Contents**: Firewall Events, System Events (tùy nhu cầu).
5. Lưu cấu hình.

**Suricata (trên pfSense)**
1. Vào **Services → Suricata** và chọn interface đang chạy.
2. Tại **Global Settings** (hoặc tab cấu hình logging), bật gửi log qua Syslog.
3. Đặt đích đến `IP_ELK:5140`, định dạng EVE JSON.

## 4.6. Kiểm thử và nhập dashboard

1. **Kiểm tra log tới cổng 5140**
   ```bash
   sudo tcpdump -i <interface_mạng> port 5140
   ```
   Nếu thấy gói từ pfSense, đường truyền đã thông.
2. **Nhập template/dashboards vào Kibana** (thực hiện thủ công, không dùng lệnh tải GitHub):
   - Mở `http://<IP_ELK>:5601` → **Dev Tools** → dán nội dung từ các file trong `etc/pfelk/templates/` theo thứ tự và chạy.
   - Vào **Stack Management → Saved Objects → Import**, tải thủ công file `23.09-firewall.ndjson` (và `22.01-suricata.ndjson` nếu dùng). Bạn cần lấy nội dung file trực tiếp từ bản sao repository đã chép vào máy.

## 4.7. Lưu ý vận hành

- Theo dõi `journalctl -u elasticsearch`, `journalctl -u logstash`, `journalctl -u kibana` để kiểm tra lỗi.
- Điều chỉnh Heap Elasticsearch và pipeline Logstash dựa trên tài nguyên thực tế.
- Sao lưu định kỳ các file `.pfelk`, `.grok` và CSV trong `/etc/logstash/` sau khi chỉnh sửa.
