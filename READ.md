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





# Write-up Day 02: Binary Search 2D, Cronjobs & Shell Script Injection

**Date:** 08/10/2026
**Topics:** Algorithms (Binary Search), Linux Sysadmin (Cron), Privilege Escalation, Git/GitHub.

---

## 1. Thuật toán: LeetCode 74 - Search a 2D Matrix

**Vấn đề gặp phải:** 
Ban đầu, tôi cố gắng áp dụng nguyên vẹn thuật toán Binary Search 1 chiều lên ma trận 2 chiều (`vector<vector<int>>`). Lỗi xảy ra khi tôi gọi `matrix[mid]` để so sánh với `target`. Trong C++, `matrix[mid]` trả về toàn bộ một mảng con (một hàng), chứ không phải một số nguyên cụ thể, dẫn đến lỗi biên dịch. Hơn nữa, tôi chưa khởi tạo số lượng hàng và cột của ma trận.

**Cách giải quyết & Giải thích Pattern:**
Để đạt độ phức tạp `O(log(m * n))`, ta phải coi ma trận 2D như một mảng 1D trải dài.

1. **Khai báo kích thước:**
   - `int m = matrix.size();` (Lấy số lượng mảng con -> Số Hàng)
   - `int n = matrix[0].size();` (Lấy số lượng phần tử của mảng con đầu tiên -> Số Cột)

2. **Ánh xạ Index (1D -> 2D):** 
   Giả sử mảng có `n` cột. Khi con trỏ Binary Search đang ở vị trí `mid` trên "mảng 1D ảo", ta tìm tọa độ thật của nó trên ma trận bằng công thức:
   - **Hàng (Row) = `mid / n`**: Phép chia lấy phần nguyên cho biết ta đã lướt qua bao nhiêu hàng trọn vẹn.
   - **Cột (Col) = `mid % n`**: Phép chia lấy phần dư cho biết vị trí chính xác của phần tử đó ở hàng hiện tại.
   
   *Áp dụng vào code:* 
   ```cpp
   int mid_value = matrix[mid / n][mid % n];


## 2. Linux Security: OverTheWire - Bandit (Level 22 -> 24)
#  Bandit 22 → 23: Dịch ngược cơ chế sinh tên file
Nhiệm vụ: Tìm cờ được giấu bởi một cronjob chạy ngầm.

Vấn đề: File kịch bản của hệ thống sử dụng biến $myname để tạo tên file chứa mật khẩu. Khi gọi cat /tmp/$mytarget trên terminal, hệ thống báo lỗi cat: /tmp/: Is a directory vì biến rỗng (chỉ có giá trị khi script được chạy bởi cron).

Cách giải quyết: Đóng giả hệ thống bằng cách tự chạy lệnh hash thủ công với user là

echo I am user bandit23 | md5sum | cut -d ' ' -f 1
echo: In chuỗi đầu vào.

| (Pipe): Chuyền chuỗi vừa in sang lệnh tiếp theo.

md5sum: Mã hóa chuỗi thành mã băm 32 ký tự.

cut -d ' ' -f 1: Dùng khoảng trắng (-d ' ') làm vách ngăn để cắt chuỗi, và chỉ lấy khối dữ liệu đầu tiên (-f 1), qua đó loại bỏ các ký tự rác (dấu gạch ngang) sinh ra từ hàm băm.
#  Bandit 23 → 24: Leo thang bằng Shell Script (Payload Injection)
Nhiệm vụ: Lợi dụng cronjob quét thư mục /var/spool/bandit24/foo (chạy mọi script bên trong với quyền bandit24 rồi xóa đi) để đánh cắp mật khẩu.
1. Tạo Workspace an toàn:
Tạo thư mục làm việc và cấp quyền 777 (cho phép tiến trình ngầm bandit24 có quyền ghi kết quả trả về).
mkdir /tmp/hoanglong24
chmod 777 /tmp/hoanglong24
cd /tmp/hoanglong24
2. Viết Script khai thác (Payload):
echo '#!/bin/bash' > myscript.sh
echo 'cat /etc/bandit_pass/bandit24 > /tmp/hoanglong24/password.txt' >> myscript.sh
chmod +x myscript.sh
#!/bin/bash (Shebang): Báo cho hệ thống biết đây là script chạy bằng Bash. Dấu > ghi đè tạo file mới.

Dấu >> giúp viết nối lệnh lấy cờ vào cuối file mà không đè mất dòng Shebang.

Lệnh cat bên trong script sẽ lấy quyền của bandit24 đọc file mật khẩu gốc và đẩy kết quả về thư mục trung gian của mình.

chmod +x: Cấp quyền "Thực thi" (eXecute) để script chạy được.
3. Kích hoạt bẫy:
Copy myscript.sh vào thư mục đích /var/spool/bandit24/foo, đợi tối đa 1 phút cho cronjob chạy, sau đó đọc file password.txt để lấy cờ.
#3. MINDSET
Bảo mật không phải là phép thuật, mà là sự thấu hiểu hệ thống sâu sắc hơn cả người tạo ra nó. Hôm nay, khi bạn lợi dụng cronjob để đánh cắp mật khẩu trong Bandit, bạn không dùng một phần mềm hack nào phức tạp. Bạn chỉ đơn giản là hiểu rõ cách Linux tự động hóa công việc và chèn mã kịch bản vào đúng điểm mù của luồng thực thi. Đó chính là tư duy cốt lõi của lộ trình này: Nghĩ như một kẻ tấn công, nhưng hành động với sự kỷ luật của một kỹ sư hệ thống.

Bài toán ma trận LeetCode 74 lại mang đến một bài học khác về sự linh hoạt trong góc nhìn. Việc trải phẳng một không gian 2D thành một đường thẳng 1D trong tâm trí chứng minh rằng nhiều chướng ngại vật kỹ thuật có thể được giải quyết rất nhanh chóng nếu ta biết cách chuyển đổi cấu trúc dữ liệu. Khả năng nhìn thấu bề mặt của hệ thống và đưa nó về dạng cơ bản nhất là kỹ năng sống còn khi bạn phải thiết kế kiến trúc DevSecOps sau này.
# Write-up: LeetCode 875 & OverTheWire Bandit 24–25

## 1. LeetCode 875 — Koko Eating Bananas

**Đề bài:** Tìm tốc độ ăn `k` nhỏ nhất để ăn hết `n` đống chuối trong `h` giờ
(mỗi giờ chỉ ăn từ một đống, tối đa `k` quả).

**Ý tưởng:** `k` càng lớn thì số giờ cần càng ít → tính đơn điệu → binary
search trên **không gian đáp án** `[1, max(piles)]`.

```cpp
class Solution {
public:
    int minEatingSpeed(vector<int>& piles, int h) {
        int left = 1, right = *max_element(piles.begin(), piles.end());
        while (left < right) {
            int mid = left + (right - left) / 2;
            if (canfinish(piles, h, mid)) right = mid;
            else left = mid + 1;
        }
        return left;
    }
private:
    bool canfinish(const vector<int>& piles, int h, int k) {
        long long hours = 0;
        for (int p : piles) hours += (p + (long long)k - 1) / k;
        return hours <= h;
    }
};
```

**Điểm then chốt:**
- **Làm tròn lên bằng số nguyên:** `⌈p/k⌉ = (p + k - 1) / k`, vì chia nguyên
  mặc định làm tròn xuống.
- **`left < right` + `right = mid`**: dùng cho bài toán "tìm giá trị nhỏ nhất
  thỏa điều kiện" — khác với "tìm giá trị chính xác" (`left <= right`,
  `right = mid - 1`). Dùng sai cặp quy ước dễ gây lặp vô hạn hoặc lệch kết quả.
- `hours` phải kiểu `long long` vì `piles[i]` tới 10⁹, dễ tràn `int`.

**Độ phức tạp:** `O(n log(max(piles)))` thời gian, `O(1)` bộ nhớ.

---

## 2. Bandit 24 → 25 — Brute-force pincode

**Đề bài:** Daemon ở cổng `30002` trả về mật khẩu bandit25 nếu nhận đúng mật
khẩu bandit24 + pin 4 số, qua **một** kết nối duy nhất.

```bash
(for i in $(seq 0 9999); do
  printf "%s %04d\n" "<password_bandit24>" "$i"
done) | nc localhost 30002
```

- `seq 0 9999` sinh 10000 số; `for` lặp qua từng số, gọi `printf` cho mỗi số.
- `%s` → mật khẩu; `%04d` → pin đệm đủ 4 chữ số; `\n` → mỗi lần thử một dòng
  (daemon đọc theo dòng).
- `( ... ) | nc` gộp toàn bộ 10000 dòng gửi qua một kết nối TCP duy nhất.

**Lỗi hay gặp:** thiếu `%` trước `04d`, viết hoa `%S`, thiếu `\n` (dính liền
10000 dòng), thiếu `$` trước `(seq ...)`, hoặc chạy cú pháp Bash trong
PowerShell (phải chạy trong Bash thật trên server).

---

## Bài học

- Binary search trên đáp án áp dụng khi điều kiện là hàm đơn điệu theo tham
  số cần tìm, không chỉ trên mảng có sẵn.
- Khi sinh dữ liệu brute-force bằng `printf`/`for`, cần kiểm soát chặt định
  dạng (`%s`, `%04d`, `\n`) vì mỗi ký tự sai đều khiến request gửi đi không
  hợp lệ.
## 3. Linux Security: OverTheWire - Bandit (Level 25 -> 26)

**Nhiệm vụ:** Đăng nhập vào user `bandit26` bằng khóa SSH (Private Key) lấy từ `bandit25`. Tuy nhiên, shell mặc định của `bandit26` không phải là `/bin/bash` mà là một kịch bản tùy chỉnh khiến kết nối bị đóng ngay lập tức. Mục tiêu là tìm cách thoát khỏi lớp vỏ bọc này (Shell Breakout) để chiếm quyền điều khiển.

### Phân tích cơ chế giới hạn (Restricted Shell)
Khi kiểm tra file `/etc/passwd`, shell của `bandit26` được cấu hình là `/usr/bin/showtext`. Script này sử dụng lệnh `more` để hiển thị một đoạn logo văn bản. 
Đặc tính của lệnh `more` là:
- Nếu màn hình Terminal đủ lớn để hiển thị toàn bộ nội dung, nó sẽ in ra hết và lập tức kết thúc tiến trình.
- Khi tiến trình `more` kết thúc, phiên SSH cũng tự động bị ngắt (Disconnected).
- Nếu màn hình Terminal **nhỏ hơn** đoạn văn bản, `more` sẽ tạm dừng, hiển thị chữ `--More--` ở góc dưới để chờ người dùng cuộn trang. Tại trạng thái dừng này, ta có thể gọi trình soạn thảo `vi` bằng cách nhấn phím `v`.

### Các rào cản đã gặp và Cách khắc phục (Troubleshooting)

**1. Vấn đề 1: Lệnh `more` không chịu dừng lại dù đã thu nhỏ cửa sổ.**
- **Nguyên nhân:** Do các phần mềm Terminal hiện đại trên Windows đôi khi không gửi đúng thông số kích thước cửa sổ cho máy chủ Linux, khiến Linux vẫn tưởng màn hình đang rất to.
- **Giải pháp:** Sử dụng lệnh `stty rows 4` để "ép" hệ thống tin rằng chiều cao của terminal hiện tại chỉ có đúng 4 dòng, buộc lệnh `more` phải kích hoạt trạng thái chờ `--More--`.

**2. Vấn đề 2: Lỗi "Connecting from/to localhost is blocked".**
- **Nguyên nhân:** Hệ thống OverTheWire đã thay đổi chính sách, cấm người dùng kết nối vòng (localhost) từ `bandit25` sang `bandit26` để tiết kiệm tài nguyên.
- **Giải pháp:** Thay vì chạy lệnh SSH từ bên trong `bandit25`, ta phải sao chép nội dung file `bandit26.sshkey`, mang về máy cá nhân (local) và dùng nó để kết nối thẳng lên `bandit26`.

**3. Vấn đề 3: Lỗi "UNPROTECTED PRIVATE KEY FILE!" (Permissions 0777).**
- **Sự cố:** Dù đã dùng lệnh `chmod 600 bandit26_1` để bảo mật file khóa, SSH vẫn báo file này đang mở toang (0777) và từ chối kết nối.
- **Giải thích bản chất DevSecOps:** Môi trường tôi đang dùng là **WSL (Windows Subsystem for Linux)**, và file khóa đang nằm ở đường dẫn `/mnt/c/Users/ACER/...`. Đây là thư mục thuộc phân vùng NTFS của Windows được ánh xạ vào Linux. Hệ thống tập tin NTFS của Windows **không tương thích** với cơ chế phân quyền POSIX của Linux. Do đó, lệnh `chmod` bị vô hiệu hóa ngầm, Linux không thể can thiệp vào quyền của file này.
- **Cách khắc phục:** Di chuyển file khóa ra khỏi vùng `/mnt/c/...` và đưa về thư mục gốc thuần Linux của user hiện tại (`~/`). 
  ```bash
  mv bandit26_1 ~/
  cd ~/
  chmod 600 bandit26_1
