---
title: "Day 1: Binary Search, SUID Privilege Escalation & Cron Jobs"
date: "2026-10-07"
author: "Phạm Nguyễn Hoàng Long"
category: "DevSecOps Training"
tags: ["LeetCode", "CTF", "Bandit", "DevSecOps", "BinarySearch", "Linux"]
---

# 🚀 Tóm tắt rèn luyện ngày 07/10/2026

## 🧠 Phân môn Thuật toán (Algorithm)

### [LeetCode 35: Search Insert Position (Easy)](https://leetcode.com/problems/search-insert-position/)
* **Chủ đề:** Mảng (Array), Tìm kiếm nhị phân (Binary Search).
* **Độ phức tạp:** Thời gian $O(\log n)$ | Không gian $O(1)$

**Mã nguồn (C++):**
```cpp
class Solution {
public:
    int searchInsert(vector<int>& nums, int target) {
        int n = nums.size();
        int left = 0;
        int right = n - 1;
        while(left <= right){
            // Sử dụng phép trừ để tính mid, ngăn chặn nguy cơ Integer Overflow 
            // nếu mảng có kích thước cực lớn (Một thói quen tốt cho DevSecOps)
            int mid = left + (right - left) / 2; 
            
            if(nums[mid] == target){
                return mid;
            }
            else if(nums[mid] < target){
                left = mid + 1;
            }
            else {
                right = mid - 1;
            }
        }
        // Điểm mấu chốt: Trả về left khi không tìm thấy target
        return left; 
    }
};
Điểm dễ sai sót nhất của bài này là xác định giá trị trả về khi vòng lặp while kết thúc. Khi điều kiện left <= right bị phá vỡ (tức left > right), con trỏ left luôn vượt qua các phần tử nhỏ hơn target và trỏ chính xác vào phần tử đầu tiên lớn hơn target. Do đó, left chính là index hoàn hảo để chèn số mới vào mà vẫn giữ được tính chất mảng tăng dần.
'''
🛡️ Phân môn Hệ thống & Bảo mật (OverTheWire: Bandit)
1. Bandit Level 19 → 20: Kỹ thuật SUID & Job Control
Mục tiêu: Đọc mật khẩu thông qua một file thực thi có gắn quyền SUID (suconnect).
Lỗ hổng khai thác: Local Privilege Escalation (Leo thang đặc quyền cục bộ).
# Bước 1: Mở cổng mạng lắng nghe ngầm và nạp sẵn mật khẩu cũ
cat /etc/bandit_pass/bandit20 | nc -l -p 1234 &

# Bước 2: Chạy file SUID để tự động gọi vào cổng
./suconnect 1234
Phân tích kỹ thuật:
Bản thân tài khoản bandit20 không có quyền đọc file mật khẩu của bandit21. Tuy nhiên, file suconnect được gắn cờ s (SUID) và thuộc sở hữu của bandit21. Khi chạy lệnh này, Linux tạm thời cấp đặc quyền của người sở hữu cho tiến trình. Kết hợp với việc sử dụng ký tự & (đẩy tiến trình Netcat xuống chạy nền), ta có thể thao tác đồng thời cả hai luồng mạng trên cùng một terminal.
2. Bandit Level 21 → 22: Rủi ro từ Cron Jobs
Mục tiêu: Truy vết mật khẩu bị rò rỉ qua các tiến trình tự động của hệ thống.
Quy trình truy vết:
# 1. Liệt kê và kiểm tra các tác vụ cron đang được cấu hình
ls -la /etc/cron.d/
cat /etc/cron.d/cronjob_bandit22     

# Kết quả: cron tự động chạy file /usr/bin/cronjob_bandit22.sh mỗi phút (* * * * *)

# 2. Phân tích mã nguồn script để xem logic xử lý mật khẩu
cat /usr/bin/cronjob_bandit22.sh     

# Kết quả: Script lưu bản sao mật khẩu vào thư mục /tmp/

# 3. Lấy cờ từ file rò rỉ
cat /tmp/t7O6lds9S0RqQh9aMcz6ShpAoZKF7fgv
🎯 Góc nhìn DevSecOps (Mindset Review)
Quá trình rèn luyện hôm nay là sự đối chiếu rõ nét giữa tư duy xây dựng và tư duy phá hủy:
1. Góc nhìn Dev: Khi viết mã (như trong bài LeetCode), việc kiểm soát các trường hợp biên như Integer Overflow (tràn số) bằng cách tối ưu công thức tính mid là lớp phòng thủ đầu tiên.
2. Góc nhìn Sec/Ops: Việc sử dụng bừa bãi quyền SUID (như bài Bandit 20) hoặc thiết lập Cron jobs với các file bash có quyền ghi (write) lỏng lẻo (như bài Bandit 22) có thể làm vô hiệu hóa mọi nỗ lực bảo mật hệ thống. Một tiến trình chạy ngầm với đặc quyền cao luôn là mục tiêu ưa thích của tin tặc để cài cắm Backdoor.
