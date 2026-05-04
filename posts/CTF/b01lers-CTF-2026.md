# crypto/ecb-lasagna

## Đề bài 
<details>
    <summary> <strong>chall.py</strong></summary>
    
    import base64

    from Crypto.Cipher import AES
    from Crypto.Util.strxor import strxor

    flag = open("../flag.txt").read().strip()

    s = ""
    for c in flag:
        s += c * 2
    flag = s

    cipher = AES.new(b"lasagna!" * 2, AES.MODE_ECB)
    result = b"\0" * len(flag)

    for i in range(len(result)):
        ciphertext = cipher.encrypt(flag[i].encode() * 16)
        layer = b"\0" * i + ciphertext
        if len(layer) < len(result):
            layer += b"\0" * (len(result) - len(layer))
        if len(layer) > len(result):
            layer = layer[len(result):] + layer[len(layer)-len(result):len(result)]
        result = strxor(result, layer)

    print(base64.b64encode(result).decode())
</details>

<details>
    <summary> <strong>output.txt</strong></summary>
    3XpvaycmXO/ycXW4lFfEzkeOcA6d+JDBBBODX9AFUc6L4IX7X4kHIj51/jYDjCnYfvKDBDueCg/2PrTMQZPw2HAlcUIDXioO2HTpxAShWQH3jz3aL7dOfoqv9cLChR/+HwIjyG0lqx78kwtVv4WgCN0+V0eeg1xnA8fxC/u0Gu83Cw5AeApiuVMch4r53IM+Q4+LUyUqs1LVFCemPBXeensXZHtzYREO7gvjyvdS/NvaMLfqjNYpOeKB4bB7WKrNhsKqIS2STUjxF8w42oNWbFAZ7CBlTalYaaw6blIefyqVVNwmw2kqEE8sUjgSATEyfuZAlKghZ/kBwseH5DLRIEYbfVa/TxehxMeHPas7/3l3dhq3C6/OjUvfz0bUQoj7Kh5/KpVUpXKSLFhTi7zMk02+Rmig/0RrxAys3eQNjD3sFc2quWLkUfQpPWLZCwOFC8oPJwcobjA23LxwTxO1Cv6QIAtNFnO7
</details>

## Ý tưởng

### Phân tích đề bài 
- Trước hết, đề bài thực hiện kéo dài string bằng cách nhân đôi.
```python
    s = ""
    for c in flag:
        s += c * 2
    flag = s
```
Điều này khiến cho {% raw %}`bctf{ -> bbccttff{{`{% endraw %}. Điều này là một trong những lỗ hổng để giúp ta giải mã nó.
- Sau đó đề lấy từng byte trong flag và thực hiện:
```python
cipher = AES.new(b"lasagna!" * 2, AES.MODE_ECB)
ciphertext = cipher.encrypt(flag[i].encode() * 16)
```
Vì mode của đề là `ECB` và lộ ra key nên ta hoàn toàn có thể tạo lại một bảng để tra ngược `ciphertext` thành `flag[i]`.
- Tiếp theo chính là cơ chế chính của việc mã hoas:
```python
    result = b"\0" * len(flag)

    for i in range(len(result)):
        ciphertext = cipher.encrypt(flag[i].encode() * 16)
        layer = b"\0" * i + ciphertext
        if len(layer) < len(result):
            layer += b"\0" * (len(result) - len(layer))
        if len(layer) > len(result):
            layer = layer[len(result):] + layer[len(layer)-len(result):len(result)]
        result = strxor(result, layer)
```
Cả tổng thể thì hàm này đang thực hiện `xor` dập lên toàn bộ kết quả có trước đó. Đây là cái nhìn tổng quát nhất với 2 trường hợp `i=15` và `i=17`:

| Cột | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 | 11 | 12 | 13 | 14 | 15 |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| **L0** | B0 | B1 | B2 | B3 | B4 | B5 | B6 | B7 | B8 | B9 | B10 | B11 | B12 | B13 | B14 | B15 |
| **L1** | B15 | B0 | B1 | B2 | B3 | B4 | B5 | B6 | B7 | B8 | B9 | B10 | B11 | B12 | B13 | B14 |
| **L2** | B14 | B15 | B0 | B1 | B2 | B3 | B4 | B5 | B6 | B7 | B8 | B9 | B10 | B11 | B12 | B13 |
| **L3** | B13 | B14 | B15 | B0 | B1 | B2 | B3 | B4 | B5 | B6 | B7 | B8 | B9 | B10 | B11 | B12 |
| **L4** | B12 | B13 | B14 | B15 | B0 | B1 | B2 | B3 | B4 | B5 | B6 | B7 | B8 | B9 | B10 | B11 |
| **L5** | B11 | B12 | B13 | B14 | B15 | B0 | B1 | B2 | B3 | B4 | B5 | B6 | B7 | B8 | B9 | B10 |
| **L6** | B10 | B11 | B12 | B13 | B14 | B15 | B0 | B1 | B2 | B3 | B4 | B5 | B6 | B7 | B8 | B9 |
| **L7** | B9 | B10 | B11 | B12 | B13 | B14 | B15 | B0 | B1 | B2 | B3 | B4 | B5 | B6 | B7 | B8 |
| **L8** | B8 | B9 | B10 | B11 | B12 | B13 | B14 | B15 | B0 | B1 | B2 | B3 | B4 | B5 | B6 | B7 |
| **L9** | B7 | B8 | B9 | B10 | B11 | B12 | B13 | B14 | B15 | B0 | B1 | B2 | B3 | B4 | B5 | B6 |
| **L10** | B6 | B7 | B8 | B9 | B10 | B11 | B12 | B13 | B14 | B15 | B0 | B1 | B2 | B3 | B4 | B5 |
| **L11** | B5 | B6 | B7 | B8 | B9 | B10 | B11 | B12 | B13 | B14 | B15 | B0 | B1 | B2 | B3 | B4 |
| **L12** | B4 | B5 | B6 | B7 | B8 | B9 | B10 | B11 | B12 | B13 | B14 | B15 | B0 | B1 | B2 | B3 |
| **L13** | B3 | B4 | B5 | B6 | B7 | B8 | B9 | B10 | B11 | B12 | B13 | B14 | B15 | B0 | B1 | B2 |
| **L14** | B2 | B3 | B4 | B5 | B6 | B7 | B8 | B9 | B10 | B11 | B12 | B13 | B14 | B15 | B0 | B1 |
| **L15** | B1 | B2 | B3 | B4 | B5 | B6 | B7 | B8 | B9 | B10 | B11 | B12 | B13 | B14 | B15 | B0 |

| Cột | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 | 11 | 12 | 13 | 14 | 15 | 16 | 17 |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| **L0** | B0 | B1 | B2 | B3 | B4 | B5 | B6 | B7 | B8 | B9 | B10 | B11 | B12 | B13 | B14 | B15 | | |
| **L1** | | B0 | B1 | B2 | B3 | B4 | B5 | B6 | B7 | B8 | B9 | B10 | B11 | B12 | B13 | B14 | B15 | |
| **L2** | | | B0 | B1 | B2 | B3 | B4 | B5 | B6 | B7 | B8 | B9 | B10 | B11 | B12 | B13 | B14 | B15 |
| **L3** | B15 | | | B0 | B1 | B2 | B3 | B4 | B5 | B6 | B7 | B8 | B9 | B10 | B11 | B12 | B13 | B14 |
| **L4** | B14 | B15 | | | B0 | B1 | B2 | B3 | B4 | B5 | B6 | B7 | B8 | B9 | B10 | B11 | B12 | B13 |
| **L5** | B13 | B14 | B15 | | | B0 | B1 | B2 | B3 | B4 | B5 | B6 | B7 | B8 | B9 | B10 | B11 | B12 |
| **L6** | B12 | B13 | B14 | B15 | | | B0 | B1 | B2 | B3 | B4 | B5 | B6 | B7 | B8 | B9 | B10 | B11 |
| **L7** | B11 | B12 | B13 | B14 | B15 | | | B0 | B1 | B2 | B3 | B4 | B5 | B6 | B7 | B8 | B9 | B10 |
| **L8** | B10 | B11 | B12 | B13 | B14 | B15 | | | B0 | B1 | B2 | B3 | B4 | B5 | B6 | B7 | B8 | B9 |
| **L9** | B9 | B10 | B11 | B12 | B13 | B14 | B15 | | | B0 | B1 | B2 | B3 | B4 | B5 | B6 | B7 | B8 |
| **L10** | B8 | B9 | B10 | B11 | B12 | B13 | B14 | B15 | | | B0 | B1 | B2 | B3 | B4 | B5 | B6 | B7 |
| **L11** | B7 | B8 | B9 | B10 | B11 | B12 | B13 | B14 | B15 | | | B0 | B1 | B2 | B3 | B4 | B5 | B6 |
| **L12** | B6 | B7 | B8 | B9 | B10 | B11 | B12 | B13 | B14 | B15 | | | B0 | B1 | B2 | B3 | B4 | B5 |
| **L13** | B5 | B6 | B7 | B8 | B9 | B10 | B11 | B12 | B13 | B14 | B15 | | | B0 | B1 | B2 | B3 | B4 |
| **L14** | B4 | B5 | B6 | B7 | B8 | B9 | B10 | B11 | B12 | B13 | B14 | B15 | | | B0 | B1 | B2 | B3 |
| **L15** | B3 | B4 | B5 | B6 | B7 | B8 | B9 | B10 | B11 | B12 | B13 | B14 | B15 | | | B0 | B1 | B2 |
| **L16** | B2 | B3 | B4 | B5 | B6 | B7 | B8 | B9 | B10 | B11 | B12 | B13 | B14 | B15 | | | B0 | B1 |
| **L17** | B1 | B2 | B3 | B4 | B5 | B6 | B7 | B8 | B9 | B10 | B11 | B12 | B13 | B14 | B15 | | | B0 |

### Giải mã
- Nếu như ta để ý thì với `B0` ở `layer 15` trở đi, nó chỉ phụ thuộc đúng vào các `Bi` ở 15 layer trước đó. Và việc độ dài của layer có dài ra bao nhiêu đi nữa thì quy tắc đó vẫn luôn không hề thay đổi.
- Vì thế ý tưởng cốt lõi là tìm lại được `B0`. Giả sử khi ta tìm dược `B0` rồi thì muốn xác định tính đúng/sai của nó ta cần phải tạo lại một bảng tra cứu, như mình nói ở đầu thì vì ta dùng mode `ECB` nên với các plaintext ta hoàn toàn có thể tạo lại một bảng ciphertext, sau đó xét `ciphertext[0]` có đúng = `B[0]` không.
```python
CHARSET = string.ascii_lowercase + string.digits + "_{}',.? !"

cipher = AES.new(KEY, AES.MODE_ECB)
BLOCKS = {ch: cipher.encrypt(ch.encode() * 16) for ch in CHARSET}

INV0 = defaultdict(list)
for ch, blk in BLOCKS.items():
    INV0[blk[0]].append(ch)
```
- Giờ phần quan trọng nhất vẫn là công thức thì ta chưa nói. Đến với công thức thì nó đơn giản là output tại vị trí `p` hay cụ thể ở đây là 16 thì `target[16] = B[0](layer 16) xor B[1](layer 15) ... xor B[15](layer 1)`. Bản thân ta đã biết được flag format là `bctf{` nên ta cũng đã có sẵn 10 layer đầu là `bbccttff{{`, giờ thì ta chỉ cần bruteforce nốt 6 layer cuối hay 3 kí tự.  
- Giờ câu hỏi đặt ra là sau khi ta xác định được `B[0]` thì tra bảng nó bị trùng lặp 2 kí tự thì sao? Cách của mình sẽ là ta thực hiện quy hoạch động thì nó có thể đúng ở phần đó nhưng chắc chắn khi sang những phần sau nó sẽ sai và kết quả đúng chỉ luôn có 1.

## Code 
```python
import base64
import itertools
import string
from collections import defaultdict
from functools import lru_cache
from Crypto.Cipher import AES

KEY = b"lasagna!" * 2
TARGET = base64.b64decode("3XpvaycmXO/ycXW4lFfEzkeOcA6d+JDBBBODX9AFUc6L4IX7X4kHIj51/jYDjCnYfvKDBDueCg/2PrTMQZPw2HAlcUIDXioO2HTpxAShWQH3jz3aL7dOfoqv9cLChR/+HwIjyG0lqx78kwtVv4WgCN0+V0eeg1xnA8fxC/u0Gu83Cw5AeApiuVMch4r53IM+Q4+LUyUqs1LVFCemPBXeensXZHtzYREO7gvjyvdS/NvaMLfqjNYpOeKB4bB7WKrNhsKqIS2STUjxF8w42oNWbFAZ7CBlTalYaaw6blIefyqVVNwmw2kqEE8sUjgSATEyfuZAlKghZ/kBwseH5DLRIEYbfVa/TxehxMeHPas7/3l3dhq3C6/OjUvfz0bUQoj7Kh5/KpVUpXKSLFhTi7zMk02+Rmig/0RrxAys3eQNjD3sFc2quWLkUfQpPWLZCwOFC8oPJwcobjA23LxwTxO1Cv6QIAtNFnO7")
CHARSET = string.ascii_lowercase + string.digits + "_{}',.? !"

cipher = AES.new(KEY, AES.MODE_ECB)
BLOCKS = {ch: cipher.encrypt(ch.encode() * 16) for ch in CHARSET}

INV0 = defaultdict(list)
for ch, blk in BLOCKS.items():
    INV0[blk[0]].append(ch)

def main():
    prefix = "bctf{"

    for extra in itertools.product(CHARSET, repeat=3):
        seed_orig = prefix + "".join(extra)
        internal_seed = "".join(ch * 2 for ch in seed_orig)  

        @lru_cache(None)
        def dfs(p, state15):
            if p == len(TARGET):
                return ""

            noise = 0
            for k in range(1, 16):
                noise ^= BLOCKS[state15[-k]][k]

            expected_b0 = TARGET[p] ^ noise
            
            for c in INV0.get(expected_b0, []):
                nxt = dfs(p + 1, (state15 + c)[-15:])
                if nxt is not None:
                    return c + nxt

            return None
        
        suffix = dfs(16, internal_seed[-15:])
        if suffix is not None:
            flag = seed_orig + suffix[::2]
            print(flag)
            break

        dfs.cache_clear()

main() 
```

# osint/purdosint
- **Bài osint này do mình và @devobass cùng giải.**

## Đề bài 
```
You are given 22 images numbered 1 through 22. Exactly 19 were taken inside Purdue campus buildings, and 3 were taken off campus.

For each image, determine the building where it was taken. Then use the first letter of that building’s official Purdue building code as the corresponding character in the flag. For any off-campus image, use _ instead.

Read the images in numerical order (1 → 22) to obtain the flag.

Flag format: bctf{XXXXXXXXXXXXXXXXXXXXXX} (22 characters inside braces, consisting only of capital letters and underscores.)
```
[purdosint.zip](/img/CTF/b01lers-CTF-2026/purdosint/purdosint.zip)
[campusmap.pdf](/img/CTF/b01lers-CTF-2026/purdosint/campusmap.pdf)

## Ý tưởng 

### Part 1
![1](/img/CTF/b01lers-CTF-2026/purdosint/1.jpg)
- Trước tiên thì mình scan bằng google lens cũng như google AI để xem có ra được gì không.
![1_1](/img/CTF/b01lers-CTF-2026/purdosint/1_1.png)
- Ta được đề xuất là `Purdue Memorial Union` và mình đã chủ động vào google map để xác thực lại xem có chuẩn không.
![1_2](/img/CTF/b01lers-CTF-2026/purdosint/1_2.png) 
- Từ ảnh trên thì ta không bàn cãi gì nữa.
> PMU - Purdue Memorial Union

### Part 2 
![2](/img/CTF/b01lers-CTF-2026/purdosint/2.jpg)
- Part này thì mình cũng làm tương tự là dùng google lens nhưng nó không có kết quả gì cho lắm. 
- Sau đó mình đã thử search từ khóa `LOVE LIBERAL ARTS` ở trong ảnh lên gg. 
![2_1](/img/CTF/b01lers-CTF-2026/purdosint/2_1.png)
- Mình lại tiếp tục lên google map để check xem `University Hall` có chuẩn không, nhưng không tiến triển lắm. Sau đó mình đã thử search với từ khóa `Purdue University Hall` lên youtube và ra thật sự ra được [kết quả](https://www.youtube.com/watch?v=H2yBpe1EI8A&t=56s) ở giây 56 và vị trí chụp của bức ảnh chính là ở trên tầng lửng. 
![2_2](/img/CTF/b01lers-CTF-2026/purdosint/2_2.png)
> UNIV - University Hall

### Part 3
![3](/img/CTF/b01lers-CTF-2026/purdosint/3.jpg)
- Mình lại tiếp tục dùng google lens và nó cho ra kết quả:
![3_1](/img/CTF/b01lers-CTF-2026/purdosint/3_1.png)
- Đúng thật là đề bài có bảo sẽ có 3 bức ảnh không thuộc `Purdue University` nhưng hiện tại mình cảm thấy không tin tưởng google lens lắm nên tạm thời để đó.
- Sau đó mình đã thử zoom ảnh thật kĩ để soi xem có ra thêm được gì không và quả thật là có:
![3_2](/img/CTF/b01lers-CTF-2026/purdosint/3_2.jpg)
- Đó là `Union Club Hotel` - một địa điểm cũng thuộc `Purdue University`, sau đó mình đã dùng chế độ xem phố để kiểm tra thử.
![3_3](/img/CTF/b01lers-CTF-2026/purdosint/3_3.png)
- Khúc này thì chắc chắn ta đã đúng rồi và dựa vào hình ảnh đề cho thì mình đã xoay góc nhìn sang hướng ngược lại thì đây là tòa nhà của ta - `Rawls (Jerry S.) Hall`:
![3_4](/img/CTF/b01lers-CTF-2026/purdosint/3_4.png)
- Dựa trên phán đoán của mình thì bức ảnh được chụp ở vùng màu đỏ. 
> RAWL - Rawls (Jerry S.) Hall

### Part 4
![4](/img/CTF/b01lers-CTF-2026/purdosint/4.jpg)
- Lại là google lens nhưng lần này mình add thêm từ khóa `Purdue University` vào cho chắc.
![4_1](/img/CTF/b01lers-CTF-2026/purdosint/4_1.png)
- Nó đề xuất cho ta `Dudley Hall` và `Lambertus Hall`.
- Sau đó mình thi search từ khóa `Dudley Hall Purdue` vào google và tìm được web [này](https://www.bsadesign.com/project/dudley-hall-and-lambertus-hall/)
![4_2](/img/CTF/b01lers-CTF-2026/purdosint/4_2.png)
- Khúc này thì khá chuẩn rồi đó, nhưng ngặt cái là web của nó gộp thành `Dudley and Lambertus Hall` vì hai tòa này thông với nhau, nhưng trong campusmap thì nó lại tách biệt nên mình sẽ note ra 2 cái luôn.
- Sau đó mình có thử tìm `Dudley and Lambertus Hall floor plan` của trên google và ra được vid youtube [này](https://youtu.be/DLY4Soph-jA?si=KbofADIXuJv3OhII&t=251), ta chỉ cần chú ý 3 cái cầu thang.
![4_3](/img/CTF/b01lers-CTF-2026/purdosint/4_3.png)
![4_4](/img/CTF/b01lers-CTF-2026/purdosint/4_4.png)
- Đây là vị trí của 3 cái cầu thang mà mình nói và hướng nhìn của nó từ tầng trên xuống chiếu nghỉ. Nếu ta để ý kĩ ảnh gốc thì ở ngoài cửa sổ đó là một con đường và có xe đang chạy, nên khi nhìn vào ảnh từ vệ tinh thì ta hoàn toàn nhận ra kết quả chỉ có thể là `Dudley Hall`.
> DUDL - Dudley Hall

### Part 5
![5](/img/CTF/b01lers-CTF-2026/purdosint/5.jpg)
- Mình lại tiếp tục google lens với từ khóa `Purdue University`.
![5_1](/img/CTF/b01lers-CTF-2026/purdosint/5_1.png)
- Cái mình quan tâm là link reddit và nhìn nó khá uy tín.
![5_2](/img/CTF/b01lers-CTF-2026/purdosint/5_2.png)
- Đây đúng là cái ta cần tìm rồi, sau đó mình mò dưới cmt để tìm tên.
![5_3](/img/CTF/b01lers-CTF-2026/purdosint/5_3.png)
> UC - University Church

### Part 6 
![6](/img/CTF/b01lers-CTF-2026/purdosint/6.jpg)
- Mình lại bắt đầu với google lens và từ khóa `Purdue University`.
![6_1](/img/CTF/b01lers-CTF-2026/purdosint/6_1.png)
- Nó có đề xuất cho mình đó là `Edward C. Elliott Hall of Music` hay `Elliott (Edward C.) Hall of Music`.
- Sau đó mình đã vào youtube để tìm thông tin với từ khóa `Edward C. Elliott Hall of Music` và ra được [vid1](https://youtu.be/YzpUN79ilcQ?si=U0Da4O-dNuK6Ypm4) và [vid2](https://youtu.be/IROd2pdvM6g?si=niB3ymqgp-BAEJiG&t=57). Vid1 khá dài nhưng không giúp được bao nhiêu hết, còn vid2 thì tại giây 57-58 tác giả đã quay đúng chỗ chụp bức ảnh. 
![6_2](/img/CTF/b01lers-CTF-2026/purdosint/6_2.png)
> ELLT - Elliott (Edward C.) Hall of Musi

### Part 7 
![7](/img/CTF/b01lers-CTF-2026/purdosint/7.jpg)
- Mình lại mở đầu bài này với google len và từ khóa `Purdue University`.
![7_1](/img/CTF/b01lers-CTF-2026/purdosint/7_1.png)
- Nhưng tới bài này thì kết quả không còn chắc chắn nữa, mình nghĩ 90% đây không phải là `Purdue University`. Và cái cần thuyết phục ở đây là ta phải chứng minh được nó nằm ở University nào.
- Sau đó mình sửa lại từ khóa chỉ còn `University`.
![7_2](/img/CTF/b01lers-CTF-2026/purdosint/7_2.png)
- Đây là trường `Carnegie Mellon University (CMU)` và phòng học thuộc `Gates and Hillman Centers`. 
- Mình đã lên google tìm kiếm từ khóa `Gates and Hillman Centers classroom` và ra được web [này](https://www.cmu.edu/computing/services/teach-learn/tes/classrooms/locations/gates-hillman.html). 
![7_3](/img/CTF/b01lers-CTF-2026/purdosint/7_3.png)
- Đây chính xác là phòng học trong ảnh.
> _

### Part 8 
![8](/img/CTF/b01lers-CTF-2026/purdosint/8.jpg)
- Vì bài này mình nghĩ là danh nhân thế giới nên không xài từ khóa khi google lens nữa.
![8_1](/img/CTF/b01lers-CTF-2026/purdosint/8_1.png)
- Bức tượng này được đặt tại `Wetherill Chemistry Building` hay chính xác là `Wetherill Hall of Chemistry` hoặc `Wetherill (Richard Benbridge) Laboratory of Chemistry`. 
- Mình lại tiến vào google map để check xem chuẩn không.
![8_2](/img/CTF/b01lers-CTF-2026/purdosint/8_2.png)
> WTHR - Wetherill (Richard Benbridge) Laboratory of Chemistry

### Part 9 
![9](/img/CTF/b01lers-CTF-2026/purdosint/9.jpg)
- Bài này trước mặt ta là `Ross–Ade Stadium` nên mình chọn lên google map xem vệ tinh luôn cho dễ làm.
![9_1](/img/CTF/b01lers-CTF-2026/purdosint/9_1.png)
- Bài này vì trước mặt người chụp hình chỉ có cây chứ không còn công trình nào khác nên mình đoán chắc nó là `Cary (Franklin Levering) Quadrangle`. Và dựa vào góc nhìn thì mình đoán bức ảnh được chụp ngay góc của `CQNW` nhưng cơ bản flag chỉ nhận chữ cái đầu nên việc mình cứ để `CARY` hay phân tích từng tòa nhà của nó (`CQW`, `CQNW`, `CQNE`, `CQE`) cũng chẳng ảnh hưởng gì mấy.
> CARY - Cary (Franklin Levering) Quadrangle

### Part 10 
![10](/img/CTF/b01lers-CTF-2026/purdosint/10.jpg)
- Đây là một trong những part khó nhằn nhất đối với mình. Trước hết thì mình chú ý tới 2 tiểu tiết: 1 là ngã tư đường với đèn xanh đèn đỏ, 2 là loại ốp kim loại trên tường.
- Mình lại bắt đầu google lens nó với từ khóa `Purdue University`. 
![10_1](/img/CTF/b01lers-CTF-2026/purdosint/10_1.png)
- Check google map của `WALC` thì chắc chắn là nó không hợp lý với những gì mình cần rồi. 
- Sau đó mình nảy ra ý tưởng là list ra các trục đường lớn ở đó, vì qua quá trình quan sát mình để ý rằng các trục đường nhỏ thường không có đèn giao thông và mình chỉ cần mò theo các trục đường lớn đó thôi.
![10_2](/img/CTF/b01lers-CTF-2026/purdosint/10_2.png)
- Đây là toàn bộ các ngã tư đường và 3 trục đường mà mình nhìn vào là thấy để từ từ đi mò.
![10_3](/img/CTF/b01lers-CTF-2026/purdosint/10_3.png)
- Nhìn vào những chỗ mình khoanh tròn thì chắc ta hoàn toàn có thể nhận ra chính xác nó ở đâu, và vị trí chụp mình đoán là ngay chỗ mũi tên của mình.
> BIDC - Bechtel Innovation Design Center 

### Part 11
![11](/img/CTF/b01lers-CTF-2026/purdosint/11.jpg)
- Tiếp tục là google lens với từ khóa `Purdue University`.
![11_1](/img/CTF/b01lers-CTF-2026/purdosint/11_1.png)
- Mình thấy nó đề xuất `Wilmeth Active Learning Center` nhưng khi mình lên google map check thì không thấy hình ảnh phòng học giống vậy nên chưa tin tưởng lắm.
- Sau đó mình để ý 2 cái link mà nó đề xuất, điều đặc biệt nằm ở [link2](https://www.tradelineinc.com/reports/2018-8/purdue-university-combines-classroom-and-library-space-promote-active-learning#:~:text=The%20Eye%20to%20Eye%20classrooms,Andersson%2C%20director%20of%20library%20facilities.)
![11_2](/img/CTF/b01lers-CTF-2026/purdosint/11_2.png)
- Bản thân mình thấy phòng học này khá giống với ảnh gốc tuy nhiên khi nhìn vào map thì nó chỉ có 1 máy chiếu ngay trung tâm nên mình cảm thấy cấn cấn.
- Sau đó mình tiếp tục search `1055 classroom in walc` và ra được trang web [này](https://www.purdue.edu/activelearning/walcrooms1/Theater.php)
![11_3](/img/CTF/b01lers-CTF-2026/purdosint/11_3.png)
- Ở thông số `doc cam` nó là 2 nên bản thân mình nghĩ rằng đây đúng là đáp án ta cần, còn cái map kia thì có thể là map cũ từ trước khi nó nâng cấp.
> WALC - Wilmeth (Thomas S. and Harvey D.) Active Learning Center 

### Part 12
![12](/img/CTF/b01lers-CTF-2026/purdosint/12.jpg)
- Part này cũng nằm trong những part khó nhằn mà mình làm được. Bài này thì ban đầu mục tiêu của mình là 4 thứ: bãi cỏ hình giống cánh buồm, cái đèn và cái ụ đất ngay cái đèn, chỗ để xe đạp và cái cây, tòa nhà có tường gạch đối diện. 
- Mới đầu vào mình nghĩ rằng google lens sẽ không tìm ra được nhưng mình vẫn muốn test cho chắc.
![12_1](/img/CTF/b01lers-CTF-2026/purdosint/12_1.png)
- Sau đó tương tự như `part 10` ở trên, mình tiến hành lên vệ tinh trên google map. Ý tưởng của mình là ta sẽ tiến hành chia ra từng khu vực nhỏ, sau đó kiểm tra xem có bãi cỏ nào là hình cánh buồm không. 
![12_2](/img/CTF/b01lers-CTF-2026/purdosint/12_2.png)
- Đáp án nằm ở bãi cỏ mà mình khoanh đỏ và có mũi tên và giờ mình sẽ chứng minh tại sao nó đúng.
![12_3](/img/CTF/b01lers-CTF-2026/purdosint/12_3.png)
- Những chỗ mình đánh dấu và khoanh tròn chính là những mục tiêu mà mình đề cập ban đầu. Còn vị trí chụp thì mình đoán là ngay hình chữ nhật.
> PKRF - Parker (Frieda) Residence Hall (formerly Griffin Residence Halls)

### Part 13
![13](/img/CTF/b01lers-CTF-2026/purdosint/13.jpg)
- Part này khá dễ khi nhìn kĩ vào ảnh ta sẽ thấy `E18` và `E25` thì rõ ràng cách đặt tên các tòa nhà của `Purdue University` không hề giống như này. Nên từ giờ mình sẽ tiến hành tìm xem nơi này nằm chính xác ở university nào.
- Mình sẽ tiến hành google lens với từ khóa `University`.
![13_1](/img/CTF/b01lers-CTF-2026/purdosint/13_1.png)
- Ta khoanh vùng được `Massachusetts Institute of Technology`. Sau đó mình tiến hành lên google map xem vệ tinh để check vị trí của 2 tòa nhà này có gần nhau không, kế đó là nếu ta nhìn kĩ bức ảnh gốc thì nó có show map của tòa `E18` nên chỉ cần đối chiếu là xong.
![13_2](/img/CTF/b01lers-CTF-2026/purdosint/13_2.png)
- Với kiểu cấu trúc này thì hiển nhiên nó thỏa mãn 2 yêu cầu của ta rồi.
> _

### Part 14
![14](/img/CTF/b01lers-CTF-2026/purdosint/14.jpg)
- Part này thì đối diện nó là 1 parking garage nên mình sẽ vào thẳng campusmap và list ra luôn.
![14_1](/img/CTF/b01lers-CTF-2026/purdosint/14_1.png)
- Mình tiến hành check map của các garage này và tại `PGU - Parking Garage, University Stree`.
![14_2](/img/CTF/b01lers-CTF-2026/purdosint/14_2.png)
- Nhìn theo kiểu cấu trúc này thì chính xác đây là kết quả cần tìm. Với vị trí chụp là cái cửa đối diện garage.
> HAAS - Haas (Felix) Hall

### Part 15
![15](/img/CTF/b01lers-CTF-2026/purdosint/15.jpg)
- Part này mình cảm giác nó khá khó và có lẽ mình may mắn hơn là kĩ năng.
- Mình lại tiến hành google lens nhưng lần này là từ khóa `Purdue University place name` đồng thời cũng search theo nhiều khung ảnh nhỏ vì nếu để nguyên ảnh lớn thì kết quả trả ra gần như chỉ là cái cây ngay đó.
![15_1](/img/CTF/b01lers-CTF-2026/purdosint/15_1.png)
- Ở đây nó đề xuất cho mình 3 building và mình tiến hành check 3 nơi đó. Với `Purdue Memorial Union` thì mình sẽ loại ra đầu tiên vì nơi đó đã nằm ở một part trước đó rồi, `Horticulture Building / Greenhouses` thì rõ ràng là không phải, còn `Lilly Hall of Life Sciences` mình tìm được khi vào check google map của nó.
![15_2](/img/CTF/b01lers-CTF-2026/purdosint/15_2.png)
- Cái góc trước mặt chính xác là vị trí chụp khung hình.
> LILY - Lilly Hall of Life Sciences
- Bài này với những khung ảnh khác nhau thì sẽ ra các kết quả khác nhau và hầu hết các kết quả mình nhận được chỉ là đề xuất của 2 building đầu chứ không hề có `LILY`. Nhưng mà bản thân mình cũng nghĩ rằng may mắn là một loại thực lực =)))))

### Part 16
![16](/img/CTF/b01lers-CTF-2026/purdosint/16.jpg)
- Bài này mình tiến hành google lens tiếp với từ khóa `Purdue University`.
![16_1](/img/CTF/b01lers-CTF-2026/purdosint/16_1.png)
- Mặc dù kết quả trả ra là không phải nhưng đường [link](https://www.purdueexponent.org/purdue-study-areas/collection_624c9392-1377-11e5-87f0-af8e7080430f.html) bên dưới mới là thứ mình quan tâm.
![16_2](/img/CTF/b01lers-CTF-2026/purdosint/16_2.png)
- Đây là những gì ta cần.
> POTR - Potter (A.A.) Engineering Center

### Part 17
![17](/img/CTF/b01lers-CTF-2026/purdosint/17.jpg)
- Lại bắt đầu với google lens tiếp với từ khóa `Purdue University`.
![17_1](/img/CTF/b01lers-CTF-2026/purdosint/17_1.png)
- Kết quả trả ra là `the Neil Armstrong Hall of Engineering` và mình vào [post instagram](https://www.instagram.com/p/DDKgJ4MxV18/?img_index=1) kia để check.
![17_2](/img/CTF/b01lers-CTF-2026/purdosint/17_2.png)
- Đây là những gì ta cần.
> ARMS - Armstrong (Neil) Hall of Engineering

### Part 18
![18](/img/CTF/b01lers-CTF-2026/purdosint/18.jpg)
- Bài này mình bắt đầu với việc search `1167 office Purdue University`.
![18_1](/img/CTF/b01lers-CTF-2026/purdosint/18_1.png)
- Nó có đề xuất 1 [file pdf](https://www.cs.purdue.edu/people/directory.pdf)
![18_2](/img/CTF/b01lers-CTF-2026/purdosint/18_2.png)
- Mình nghĩ đây là kết quả ta cần, nhưng mình muốn chính xác hơn vì có thể có nhiều phòng 1167 nên có mò tên của professor `Antonio Bianchi` trên google 
![18_3](/img/CTF/b01lers-CTF-2026/purdosint/18_3.png)
- Profile này thì khó mà sai.
> LWSN - Lawson (Richard and Patricia) Computer Science Building

### Part 19
![19](/img/CTF/b01lers-CTF-2026/purdosint/19.jpg)
- Bài này thì khi phân tích tấm ảnh, ở cái xe f1 có dòng chữ `HAIL TO THE ORANGE`. Sau khi search trên google thì chính xác dòng chữ đó là bài hát của `University of Illinois Urbana-Champaign (UIUC)`. Còn vị trí cụ thể thì mình vẫn chưa tìm ra được nhưng mình đoán nó sẽ nằm trong xưởng của khoa cơ khí hoặc kĩ thuật.
> _

### Part 20
![20](/img/CTF/b01lers-CTF-2026/purdosint/20.jpg)
- Bài này như thường lệ mình lại tìm trên google lens và kiểm tra nhiều khung ảnh lớn nhỏ khác nhau nhưng hầu hết các kết quả đều là vô nghĩa.
![20_1](/img/CTF/b01lers-CTF-2026/purdosint/20_1.png)
- Sau đó mình thử thêm nhiều cách khác nhau và kết quả tới khi mình dùng gemini và nó đề xuất `France A. Córdova Recreational Sports Center`. 
![20_2](/img/CTF/b01lers-CTF-2026/purdosint/20_2.png)
- Rồi mình lại vào google map cũng như nhiều nơi khác để kiểm chứng.
![20_3](/img/CTF/b01lers-CTF-2026/purdosint/20_3.png)
- Ở ảnh gốc nếu nhìn kĩ thì phía góc trái có một bảng led nhỏ, điều này cũng là lý do mình chắc đây là địa điểm đúng.
> CREC - Córdova (France A.) Recreational Sports Center

### Part 21
![21](/img/CTF/b01lers-CTF-2026/purdosint/21.jpg)
- Tới part này thì khá dễ vì ở những part trên mình có vô tình search ra `Class of 1950 Lecture Hall` và check google map của nó nên cũng làm được bài này theo luôn.
![21_1](/img/CTF/b01lers-CTF-2026/purdosint/21_1.png)
> CL50 - Class of 1950 Lecture Hall

### Part 22
![22](/img/CTF/b01lers-CTF-2026/purdosint/22.jpg)
- Part này cũng nằm trong những part khó nhằn đối với mình. Mới đầu vào thì mình chú ý tới 2 cái cây được trồng ở 2 ụ khác nhau, đồng thời là mảng cỏ nhìn giống hình thang.
- Sau đó dựa trên cách chia map ở `part 12` thì mình cũng tốn ~~kha khá~~ rất rất nhiều thời gian vì 2 ụ cây của nó hơi nhỏ và mình chủ quan nên set độ cao của vệ tinh khá cao khiến nó càng khó hơn.
![22_1](/img/CTF/b01lers-CTF-2026/purdosint/22_1.png)
- Đây là vị trí chính xác của nó.
> MTHW - Matthews Hall

## FLAG 
> bctf{PURDUE_WCBWP_HLPAL_CCM}

## Nhiều hơn là CTF
- Bài osint này bản thân mình nói riêng đã dành gần 18 tiếng liên tục không ngủ để giải nó. Đối với mình thì bài này nó không quá khó, nhưng nó thực sự là một thử thách về ý chí với chính mình và cũng như nó khá hay khi mình phải dành toàn bộ sức lực vào việc tìm kiếm các thông tin rải rác trên mạng.
- Nếu bạn đọc tới đây và thắc mắc vì sao với mỗi part mình chỉ cần mò một chút là ra được key thì câu trả lời là mình chỉ giữ lại những cột mốc quan trọng của bài toán, chứ thời gian và những gì mình đã làm trong mỗi phần thực sự hơn rất nhiều những gì trong writeup và nếu như mình ghi ra toàn bộ thì bài này sẽ rất lan man cũng như khó đọc.
