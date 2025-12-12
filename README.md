# pfSense/OPNsense + Elastic Stack (pfELK)

**Nguồn tham khảo:** https://github.com/pfelk/pfelk/

pfELK là giải pháp nguồn mở giúp thu thập, làm giàu và trực quan hóa lưu lượng tường lửa pfSense/OPNsense bằng Elasticsearch, Logstash và Kibana. Repository này cung cấp cấu hình, dashboard và script cài đặt được chỉnh sửa để trỏ về GitHub của tôi: https://github.com/GiveMe1Star/pfense-suricata-elk.

## Mục lục
- [Yêu cầu](#yêu-cầu)
- [Tính năng chính](#tính-năng-chính)
- [Các thành phần hỗ trợ](#các-thành-phần-hỗ-trợ)
- [Tổng quan pfELK](#tổng-quan-pfelk)
- [Cài đặt nhanh](#cài-đặt-nhanh)
  - [Cài bằng script](#cài-bằng-script)
  - [Cài đặt thủ công](#cài-đặt-thủ-công)
- [So sánh với giải pháp tương tự](#so-sánh-với-giải-pháp-tương-tự)
- [Giấy phép](#giấy-phép)

## Yêu cầu
- Ubuntu Server 20.04+ hoặc Debian 11+ (đã thử nghiệm trên stretch và buster).
- pfSense 2.5.0+
- Tối thiểu 8GB RAM (Docker cần nhiều hơn), khuyến nghị 32GB.
- Đã cấu hình gửi log từ pfSense tới Logstash (xem Wiki trong GitHub: [https://github.com/pfelk/pfelk/wiki/]).

## Tính năng chính
- Thu thập và làm giàu log tường lửa pfSense với Logstash.
- Tìm kiếm dữ liệu gần thời gian thực bằng Elasticsearch.
- Trực quan hóa lưu lượng mạng qua dashboard, bản đồ, biểu đồ tương tác trong Kibana.

## Các thành phần hỗ trợ
- pfSense firewall logs với template và dashboard firewall.
- Suricata IDS logs với dashboard và tuân thủ Kibana SIEM.

## Tổng quan pfELK
pfELK hướng tới việc thay thế giao diện web mặc định của pfSense bằng khả năng tìm kiếm và trực quan hóa nâng cao. Bạn có thể triển khai bằng ansible-playbook, docker-compose, bash script hoặc thủ công. Các thành phần cốt lõi và dashboard đã được cập nhật để trỏ về GitHub này nhằm dễ dàng tải xuống và đóng góp.

## Cài đặt nhanh
### Cài bằng script
Tải và chạy script cài đặt từ GitHub của tôi:

```bash
wget https://raw.githubusercontent.com/GiveMe1Star/pfense-suricata-elk/main/etc/pfelk/scripts/pfelk-installer.sh
chmod +x pfelk-installer.sh
./pfelk-installer.sh
```

Sau khi cài đặt, truy cập Kibana tại `http://<địa_chỉ_pfelk>:5601` để nhập template và dashboard.

#### 1️⃣ Templates
1. Mở Kibana → ☰ → **Dev Tools**.
2. Lần lượt dán nội dung từng file template (theo đúng thứ tự) từ GitHub:
   - Component Template 1: `component_template_pfelk-mappings` – https://github.com/GiveMe1Star/pfense-suricata-elk/blob/main/etc/pfelk/templates/component_template_pfelk-mappings
   - Component Template 2: `component_template_pfelk-settings` – https://github.com/GiveMe1Star/pfense-suricata-elk/blob/main/etc/pfelk/templates/component_template_pfelk-settings
   - Component Template 3: `ilm-pfelk` – https://github.com/GiveMe1Star/pfense-suricata-elk/blob/main/etc/pfelk/templates/ilm-pfelk
   - Index Template: `index_template_pfelk-firewall` – https://github.com/GiveMe1Star/pfense-suricata-elk/blob/main/etc/pfelk/templates/index_template_pfelk-firewall
   - Index Template: `index_template_pfelk-suricata` – https://github.com/GiveMe1Star/pfense-suricata-elk/blob/main/etc/pfelk/templates/index_template_pfelk-suricata
3. Nhấn nút **Run** (tam giác xanh) cho từng template.

#### 2️⃣ Dashboards
1. Vào Kibana → ☰ → **Stack Management** → **Saved Objects**.
2. Import từng dashboard từ GitHub:
   - Firewall Dashboard: https://github.com/GiveMe1Star/pfense-suricata-elk/blob/main/etc/pfelk/dashboard/23.09-firewall.ndjson
   - Suricata Dashboard (tùy chọn): https://github.com/GiveMe1Star/pfense-suricata-elk/blob/main/etc/pfelk/dashboard/22.01-suricata.ndjson
3. Nhấn **Import** và xác nhận overwrite nếu cần.

### Cài đặt thủ công
- Clone repository: `git clone https://github.com/GiveMe1Star/pfense-suricata-elk.git`.
- Triển khai Logstash, Elasticsearch, Kibana theo hướng dẫn trong Wiki của repository.
- Sao chép file cấu hình từ thư mục `etc/pfelk` tới máy chủ ELK của bạn và chỉnh sửa theo môi trường thực tế.
- Nhập template và dashboard như hướng dẫn ở trên.

## So sánh với giải pháp tương tự
pfELK cung cấp pipeline log tinh gọn và dashboard chuyên sâu cho pfSense/OPNsense, giúp triển khai nhanh hơn so với tự dựng stack thủ công và có nhiều dashboard sẵn cho IDS/Proxy/Load Balancer.

## Giấy phép
Dự án phát hành theo giấy phép [LICENSE](LICENSE).
