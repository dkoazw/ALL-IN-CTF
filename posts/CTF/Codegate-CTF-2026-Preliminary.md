# Ghost

## Đề bài

<details>
    <summary> <strong>utils.py</strong></summary>

    #!/usr/bin/env python3

    import secrets
    from utils import dm_compress, hex_to_words, round_core
    from utils import SBOXES, BANNER, FLAG

    def main():
        iv = secrets.randbits(64)
        chances = 2**7

        print(BANNER)
        print(f"IV = {iv:016x}")
        print(f"Chances = {chances}/{2**7}")

        while True:
            print(
                "\n"
                "[1] query\n"
                "[2] submit\n"
                "[3] quit"
            )

            choice = input("> ")

            if choice == "1":
                if chances <= 0:
                    print("Nope!\n")
                    continue

                right_s = input("right > ")
                key_s = input("subkey > ")

                try:
                    right = int(right_s, 16)
                    subkey = int(key_s, 16)
                except:
                    print("Bad input\n")
                    continue

                chances -= 1
                y = round_core(right, subkey, SBOXES)

                print(f"core = {y:08x}")

            elif choice == "2":
                m1s = input("m1 > ")
                m2s = input("m2 > ")

                try:
                    w1 = hex_to_words(m1s)
                    w2 = hex_to_words(m2s)
                except Exception as e:
                    continue

                if w1 == w2:
                    print("Blocks must differ\n")
                    continue

                h1 = dm_compress(iv, w1, SBOXES)
                h2 = dm_compress(iv, w2, SBOXES)

                if h1 == h2:
                    print("Good!")
                    print(f"flag = {FLAG}\n")
                else:
                    print("Nope!")

                return

            elif choice == "3":
                print("Bye!\n")
                return
            
            else:
                print("Only 1,2,3 are allowed\n")

    if __name__ == "__main__":
        main()
</details>

<details>
    <summary> <strong>utils.py</strong></summary>

    MASK32 = 0xFFFFFFFF
    MASK64 = 0xFFFFFFFFFFFFFFFF

    def dm_compress(iv, key_words, sboxes):
        return encrypt_block(iv, key_words, sboxes) ^ iv

    def encrypt_block(block, key_words, sboxes):
        state = split_block(block)
        state = encrypt_rounds_from_state(state, full_schedule(key_words), sboxes)
        return join_block(*state)

    def split_block(block):
        return ((block >> 32) & MASK32, block & MASK32)

    def encrypt_rounds_from_state(state, round_keys, sboxes):
        cur = state
        for k in round_keys:
            cur = apply_round(cur, k, sboxes)
        return cur

    def apply_round(state, subkey, sboxes):
        left, right = state
        return (right & MASK32, (left ^ round_core(right, subkey, sboxes)) & MASK32)

    def round_core(right, subkey, sboxes):
        return rotl32(sbox_layer((right + subkey) & MASK32, sboxes), 11)

    def rotl32(x, r):
        x &= MASK32
        return ((x << r) & MASK32) | (x >> (32 - r))

    def sbox_layer(x, sboxes):
        y = 0
        for i in range(8):
            nib = (x >> (4 * i)) & 0xF
            y |= (sboxes[i][nib] & 0xF) << (4 * i)
        return y & MASK32

    def full_schedule(key_words):
        if len(key_words) != 8:
            raise ValueError("expected 8 key words")
        return list(key_words) * 3 + list(reversed(key_words))

    def hex_to_words(hex_string):
        s = hex_string.strip().lower()
        if s.startswith("0x"):
            s = s[2:]
        if len(s) != 64 or any(c not in "0123456789abcdef" for c in s):
            raise ValueError("message block must be 64 hex chars")
        return [int(s[i : i + 8], 16) for i in range(0, 64, 8)]

    def join_block(left, right):
        return ((left & MASK32) << 32) | (right & MASK32)


    """
    tu add vao de chay local

    SBOXES = [
        [0x6, 0x4, 0xC, 0x5, 0x0, 0x7, 0x2, 0xE, 0x1, 0xF, 0x3, 0xD, 0x8, 0xA, 0x9, 0xB],
        [0xB, 0x2, 0x5, 0xE, 0xF, 0x0, 0x8, 0xD, 0xA, 0x3, 0xC, 0x7, 0x1, 0x4, 0x6, 0x9],
        [0x7, 0xC, 0x2, 0xA, 0xD, 0x1, 0xF, 0x4, 0x9, 0x0, 0x6, 0xB, 0x5, 0x8, 0xE, 0x3],
        [0xD, 0x1, 0x8, 0xF, 0x6, 0x2, 0x3, 0xC, 0xB, 0x7, 0x0, 0x9, 0xE, 0x5, 0xA, 0x4],
        [0x3, 0xE, 0x0, 0x6, 0x9, 0xD, 0xF, 0x8, 0x5, 0xC, 0xB, 0x7, 0x2, 0xA, 0x1, 0x4],
        [0xA, 0x7, 0xE, 0x1, 0xC, 0xB, 0x4, 0x2, 0x0, 0xD, 0x9, 0xF, 0x5, 0x3, 0x8, 0x6],
        [0x4, 0x9, 0x1, 0xB, 0xE, 0x8, 0xD, 0x6, 0xF, 0x2, 0x7, 0x0, 0xA, 0x5, 0x3, 0xC],
        [0x8, 0x5, 0xB, 0x0, 0x7, 0xC, 0x1, 0xA, 0xD, 0x6, 0xF, 0x2, 0x4, 0xE, 0x9, 0x3],
    ]

    BANNER = "HI CHAT"

    FLAG = "DIT_ME_PHAP_LUAT_DAI_CUONG"
    """
</details>

## Ý tưởng 
- Khi đọc `server.py` thì ta nhận ra bài này chính là một bài tìm collision. Mới vào nó sẽ cho ta `iv` và `chances = 2**7` và server sẽ cho ta 2 lựa chọn: 1 là `querry`, 2 là `submit`. 

### querry
- Server yêu cầu ta nhập `right`, `subbkey` để đưa vào hàm `round_core` và xuất ra gì đó. Mình nhận thấy rằng trong 3 tham số của `round_core` có `SBOXES` nên mình dự đoán phải tìm lại nó từ hàm này. Nó có 3 hàm liên quan:
```python
def round_core(right, subkey, sboxes):
    return rotl32(sbox_layer((right + subkey) & MASK32, sboxes), 11)
```
- Trước tiên là `round_core` thì về cơ bản thì ta chưa phân tích gì nhiều chỉ có điều là `(right + subkey) & MASK32` để giá trị luôn được cố định trong 32 bits.

```python
def rotl32(x, r):
    x &= MASK32
    return ((x << r) & MASK32) | (x >> (32 - r))
```
- Tiếp đến là `rotl32` thì hàm này thực hiện dịch vòng 11 bits. Và mình cũng tiện build luôn `inverse_rotl32`.
```python
def inv_rotl32(x, r):
    x &= MASK32
    return (x >> r) | ((x << (32 - r)) & MASK32)
```

```python
def sbox_layer(x, sboxes):
    y = 0
    for i in range(8):
        nib = (x >> (4 * i)) & 0xF
        y |= (sboxes[i][nib] & 0xF) << (4 * i)
    return y & MASK32
```
- Cuối cùng là `sbox_layer` thì trước hết dựa trên cách nó triển khai vòng lặp với `range(8)` và `nib = (x >> (4 * i)) & 0xF` thì ta biết được ngay đây là `sbox` 8x16 giá trị vì `sboxes[i][nib]`. Thì với cách triển khai vòng lặp này giả sử ta có `x` là `0x12345678`, thì nó sẽ lần lượt cắt từng giá trị từ `8` rồi tới `7`...và những giá trị đó lúc này sẽ là `nib`, sau đó thực hiện ghép `sboxes[i][nib]` lại thành `y`. Tức là `y` lúc này nó lần lượt là `sboxes[7][1]sboxes[6][2]...sboxes[0][8]`.
- Từ đó mình nghĩ ra cách để tìm lại `SBOXES` là `x` ở hàm `sbox_layer` nó chỉ là những số hex có đúng 1 giá trị hay `0x11111111`...`0xFFFFFFFF` để ta có thể truy ngược dần về `SBOXES`. Và với 2 giá trị `right`, `subbkey` thì ta chỉ đơn giản là cho `right = 0` còn `subbkey = TTTTTTTT`.
```python
from pwn import *

MASK32 = 0xFFFFFFFF

def inverse_rotl32(x, r):
    x &= MASK32
    return (x >> r) | ((x << (32 - r)) & MASK32)

def recover_sboxes():
    sboxes = [[0] * 16 for _ in range(8)]

    r = process(["python3", "server.py"])
    
    r.recvuntil(b"> ")

    for j in range(16):
        hex_char = hex(j)[2:]
        subkey_str = hex_char * 8 
        
        r.sendline(b"1")
        
        r.recvuntil(b"right > ")
        r.sendline(b"0")
        
        r.recvuntil(b"subkey > ")
        r.sendline(subkey_str.encode())
        
        r.recvuntil(b"core = ")
        core_hex = r.recvline().strip()
        core_val = int(core_hex, 16)
        
        y = inverse_rotl32(core_val, 11)
        
        for i in range(8):
            val = (y >> (4 * i)) & 0xF
            sboxes[i][j] = val
            
        r.recvuntil(b"> ")

    r.close()
    
    return sboxes

sboxes = recover_sboxes()
    
for row in sboxes:
    for val in row:
        print(hex(val), end=" ") 
    print()
```
- Mình đã thu về lại được `SBOXES`
```python
SBOXES = [
    [0x6, 0x4, 0xC, 0x5, 0x0, 0x7, 0x2, 0xE, 0x1, 0xF, 0x3, 0xD, 0x8, 0xA, 0x9, 0xB],
    [0xB, 0x2, 0x5, 0xE, 0xF, 0x0, 0x8, 0xD, 0xA, 0x3, 0xC, 0x7, 0x1, 0x4, 0x6, 0x9],
    [0x7, 0xC, 0x2, 0xA, 0xD, 0x1, 0xF, 0x4, 0x9, 0x0, 0x6, 0xB, 0x5, 0x8, 0xE, 0x3],
    [0xD, 0x1, 0x8, 0xF, 0x6, 0x2, 0x3, 0xC, 0xB, 0x7, 0x0, 0x9, 0xE, 0x5, 0xA, 0x4],
    [0x3, 0xE, 0x0, 0x6, 0x9, 0xD, 0xF, 0x8, 0x5, 0xC, 0xB, 0x7, 0x2, 0xA, 0x1, 0x4],
    [0xA, 0x7, 0xE, 0x1, 0xC, 0xB, 0x4, 0x2, 0x0, 0xD, 0x9, 0xF, 0x5, 0x3, 0x8, 0x6],
    [0x4, 0x9, 0x1, 0xB, 0xE, 0x8, 0xD, 0x6, 0xF, 0x2, 0x7, 0x0, 0xA, 0x5, 0x3, 0xC],
    [0x8, 0x5, 0xB, 0x0, 0x7, 0xC, 0x1, 0xA, 0xD, 0x6, 0xF, 0x2, 0x4, 0xE, 0x9, 0x3],
]
```
- Và đồng thời tạo sẵn luôn `INV_SBOXES`, cách tạo thì đơn giản là xét từng hàng `SBOXES[i][digit] = value` thành `SBOXES[i][value] = digit`. Bởi vì trong toàn bộ `SBOXES` không bị trùng lặp giá trị nên mới có thể tồn tại `INV_SBOXES`.
```python
INV_SBOXES = [
    [0x4, 0x8, 0x6, 0xA, 0x1, 0x3, 0x0, 0x5, 0xC, 0xE, 0xD, 0xF, 0x2, 0xB, 0x7, 0x9],
    [0x5, 0xC, 0x1, 0x9, 0xD, 0x2, 0xE, 0xB, 0x6, 0xF, 0x8, 0x0, 0xA, 0x7, 0x3, 0x4],
    [0x9, 0x5, 0x2, 0xF, 0x7, 0xC, 0xA, 0x0, 0xD, 0x8, 0x3, 0xB, 0x1, 0x4, 0xE, 0x6],
    [0xA, 0x1, 0x5, 0x6, 0xF, 0xD, 0x4, 0x9, 0x2, 0xB, 0xE, 0x8, 0x7, 0x0, 0xC, 0x3],
    [0x2, 0xE, 0xC, 0x0, 0xF, 0x8, 0x3, 0xB, 0x7, 0x4, 0xD, 0xA, 0x9, 0x5, 0x1, 0x6],
    [0x8, 0x3, 0x7, 0xD, 0x6, 0xC, 0xF, 0x1, 0xE, 0xA, 0x0, 0x5, 0x4, 0x9, 0x2, 0xB],
    [0xB, 0x2, 0x9, 0xE, 0x0, 0xD, 0x7, 0xA, 0x5, 0x1, 0xC, 0x3, 0xF, 0x6, 0x4, 0x8],
    [0x3, 0x6, 0xB, 0xF, 0xC, 0x1, 0x9, 0x4, 0x0, 0xE, 0x7, 0x2, 0x5, 0x8, 0xD, 0xA],
]
```

#### submit 
- Ở đây nó bắt ta phải tìm collision của 2 message nhưng trước khi vào thì nó thực hiện:
```python
w1 = hex_to_words(m1s)
w2 = hex_to_words(m2s)
```

```python
def hex_to_words(hex_string):
    s = hex_string.strip().lower()
    if s.startswith("0x"):
        s = s[2:]
    if len(s) != 64 or any(c not in "0123456789abcdef" for c in s):
        raise ValueError("message block must be 64 hex chars")
    return [int(s[i : i + 8], 16) for i in range(0, 64, 8)]
```
- Và hàm `hex_to_words` chính là đang thực hiện chuyển một hex có 64 kí tự tương ứng với 256 bits thành 8 words với mỗi words có đúng 32 bits.

```python 
h1 = dm_compress(iv, w1, SBOXES)
h2 = dm_compress(iv, w2, SBOXES)
```
- Sau đó nó thực hiện `dm_compress` với yêu cầu `w1 != w2` và `h1 == h2`. 

```python
def dm_compress(iv, key_words, sboxes):
    return encrypt_block(iv, key_words, sboxes) ^ iv
```
- Phân tích sâu vào `dm_compress` thì lúc này chính xác collision ta phải tìm là `encrypt_block(iv, key_words1, sboxes) == encrypt_block(iv, key_words2, sboxes)`.

```python
def encrypt_block(block, key_words, sboxes):
    state = split_block(block)
    state = encrypt_rounds_from_state(state, full_schedule(key_words), sboxes)
    return join_block(*state)
```
- Ta sẽ đi sâu vào các hàm bên trong nó rồi mới quay lại đây.

```python
def split_block(block):
    return ((block >> 32) & MASK32, block & MASK32)
```
- Hàm này đơn giản là đang chia đôi `block` ra và `block` ở đây chính là `iv`. Giá trị trả về của nó là một tuple và mình sẽ gọi nó là `(L,R)`.

```python
def encrypt_rounds_from_state(state, round_keys, sboxes):
    cur = state
    for k in round_keys:
        cur = apply_round(cur, k, sboxes)
    return cur

def apply_round(state, subkey, sboxes):
    left, right = state
    return (right & MASK32, (left ^ round_core(right, subkey, sboxes)) & MASK32)
```
- Với hàm `apply_round` thì nó có dạng `Feistel` là `(L, R) -> (R, L ^ P(R + k))` (`round_core` mình đặt là `P()`). Quay lên `encrypt_rounds_from_state` thì nó sẽ liên tục chạy vòng lặp như vậy với các giá trị `k in round_keys`. Mà `round_keys` thì lại được hình thành từ `full_schedule`.

```python
def full_schedule(key_words):
    if len(key_words) != 8:
        raise ValueError("expected 8 key words")
    return list(key_words) * 3 + list(reversed(key_words))
```
- Hàm này thì output của nó đơn giản sẽ là:
```
k0 k1 k2 k3 k4 k5 k6 k7
k0 k1 k2 k3 k4 k5 k6 k7
k0 k1 k2 k3 k4 k5 k6 k7
k7 k6 k5 k4 k3 k2 k1 k0
```
Đây là cấu trúc quan trọng sẽ giúp ta giải mã bài này.

- Ta quay về hàm `encrypt_block` thì sau khi nó thực hiện những điều trên, nó sẽ gọi ra `join_block`
```python
def join_block(left, right):
    return ((left & MASK32) << 32) | (right & MASK32)
```
Nó sẽ thực hiện ghép `(L, R)` lại với nhau và xuất ra giá trị. 

#### Giải 
- Mấu chốt của bài này chính là ta phải kiểm soát được 32 vòng lặp trong hàm `encrypt_rounds_from_state` hay là `apply_round` mà đi sâu vào nó thì chính xác lại là `round_core` hay `P()`.
- Giả sử ở vòng hiện tại, ta muốn giá trị xuất hiện là `P(R + k) = t` thì ta có thể tìm lại `k` bằng cách `k = P_inv(t) - R` (`P_inv = inv_sbox_layer(inv_rotl32(y, 11), inv_sboxes)`).
- Với cấu trúc `Fiestel` thì nếu ta có thể chọn output một cách khéo léo thì ta có thể đưa trạng thái của block về lại ban đầu sau một số vòng nhất định. Cụ thể là 4 vòng với output mục tiêu lần lượt là `u, v, u, v`:
```
Vòng 1: (R, L ^ u)
Vòng 2: (L ^ u, R ^ v)
Vòng 3: (R ^ v, L ^ u ^ u) = (R ^ v, L)
Vòng 4: (L, R ^ v ^ v) = (L, R)
```
- 1 message có 8 words mà ta có chu kì 4 vòng nên ta sẽ nhét chu kì này 2 lần vào w1 = [k0, k1, k2, k3, k0, k1, k2, k3]. Còn w2 = [k3, k2, k1, k0, k3, k2, k1, 0]. Để giải thích thì với w1 8 khóa đầu nó sẽ `(L, R)`, 8 khóa tiếp theo cũng là `(L, R)`, 8 khóa tiếp theo cũng là `(L, R)` nhưng 8 khóa cuối thì nó sẽ đảo. Và tương tự với w2. Vì thế ta buộc phải chọn `u`, `v` sao cho w1 xuôi và w2 ngược sẽ gặp nhau ở điểm va chạm.

- Để biết được nên chọn `u`, `v` ra sao thì ta sẽ làm như sau:
    - Gọi `delta = (L - R) mod 2**32`
    - Gọi `u = BitVec("u", 32)`
    - Gọi `phi = P(P_inv(u) + delta)`
    - Ta sẽ dùng thư viện `Z3` để tính `(R ^ phi) - R == (L ^ u) - L` và tìm ra `u`.
    - Gọi `v = BitVec("v", 32)`
    - Gọi `chi = P(P_inv(v)) + delta`
    - Ta sẽ dùng thư viện `Z3` để tính `(R ^ chi) - R == (L ^ v) - L` và tìm ra `v`.
- Giải thích về `phi` và `chi`: nếu như `u` được hình thành từ `u = P(R + k)` thì `phi = P(P_inv(u) + delta) = P(R + k + L - R) = P(L + k)`. Như ta thấy thì `u` chính là sản phẩm từ `R` còn `phi` là sản phẩm từ `L`, cũng như `v` là sản phẩm từ `L` và `chi` sẽ là sản phẩm từ `R`. Nếu như tính ra được `u` với `v` thì gần như ta đã xong bài và đôi khi không phải `iv` nào cũng giải được nên ta phải thực hiện `while True` nhiều lần.

### Code 
```python
from pwn import *
from z3 import *

MASK32 = 0xFFFFFFFF

SBOXES = [
    [0x6, 0x4, 0xC, 0x5, 0x0, 0x7, 0x2, 0xE, 0x1, 0xF, 0x3, 0xD, 0x8, 0xA, 0x9, 0xB],
    [0xB, 0x2, 0x5, 0xE, 0xF, 0x0, 0x8, 0xD, 0xA, 0x3, 0xC, 0x7, 0x1, 0x4, 0x6, 0x9],
    [0x7, 0xC, 0x2, 0xA, 0xD, 0x1, 0xF, 0x4, 0x9, 0x0, 0x6, 0xB, 0x5, 0x8, 0xE, 0x3],
    [0xD, 0x1, 0x8, 0xF, 0x6, 0x2, 0x3, 0xC, 0xB, 0x7, 0x0, 0x9, 0xE, 0x5, 0xA, 0x4],
    [0x3, 0xE, 0x0, 0x6, 0x9, 0xD, 0xF, 0x8, 0x5, 0xC, 0xB, 0x7, 0x2, 0xA, 0x1, 0x4],
    [0xA, 0x7, 0xE, 0x1, 0xC, 0xB, 0x4, 0x2, 0x0, 0xD, 0x9, 0xF, 0x5, 0x3, 0x8, 0x6],
    [0x4, 0x9, 0x1, 0xB, 0xE, 0x8, 0xD, 0x6, 0xF, 0x2, 0x7, 0x0, 0xA, 0x5, 0x3, 0xC],
    [0x8, 0x5, 0xB, 0x0, 0x7, 0xC, 0x1, 0xA, 0xD, 0x6, 0xF, 0x2, 0x4, 0xE, 0x9, 0x3],
]

INV_SBOXES = [[0] * 16 for _ in range(8)]
for i in range(8):
    for j in range(16):
        INV_SBOXES[i][SBOXES[i][j]] = j


def bv_lookup4(nib, table):
    expr = BitVecVal(table[0], 4)
    for i in range(1, 16):
        expr = If(nib == BitVecVal(i, 4), BitVecVal(table[i], 4), expr)
    return expr

def bv_sbox_layer(x, sboxes):
    out = BitVecVal(0, 32)
    for i in range(8):
        nib = Extract(4 * i + 3, 4 * i, x)
        val = bv_lookup4(nib, sboxes[i])
        out = out | (ZeroExt(28, val) << (4 * i))
    return out

def bv_inv_sbox_layer(x, inv_sboxes):
    out = BitVecVal(0, 32)
    for i in range(8):
        nib = Extract(4 * i + 3, 4 * i, x)
        val = bv_lookup4(nib, inv_sboxes[i])
        out = out | (ZeroExt(28, val) << (4 * i))
    return out

def bv_P(x, sboxes):
    return RotateLeft(bv_sbox_layer(x, sboxes), 11)

def bv_Pinv(x, inv_sboxes):
    return bv_inv_sbox_layer(RotateRight(x, 11), inv_sboxes)

def solve_u(L, R):
    delta = (L - R) & MASK32
    u = BitVec("u", 32)
    phi = bv_P(bv_Pinv(u, INV_SBOXES) + BitVecVal(delta, 32), SBOXES)
    
    solver = Solver()
    solver.add(((BitVecVal(R, 32) ^ phi) - BitVecVal(R, 32)) == ((BitVecVal(L, 32) ^ u) - BitVecVal(L, 32)))
    if solver.check() != sat: 
        return None 
    return solver.model()[u].as_long() & MASK32

def solve_v(L, R):
    delta = (L - R) & MASK32
    v = BitVec("v", 32)
    chi = bv_P(bv_Pinv(v, INV_SBOXES) - BitVecVal(delta, 32), SBOXES)
    
    solver = Solver()
    solver.add(((BitVecVal(L, 32) ^ chi) - BitVecVal(L, 32)) == ((BitVecVal(R, 32) ^ v) - BitVecVal(R, 32)))
    if solver.check() != sat: 
        return None
    return solver.model()[v].as_long() & MASK32

def rotr32(x, r):
    x &= MASK32
    return (x >> r) | ((x << (32 - r)) & MASK32)

def inv_sbox_layer(x, inv_sboxes):
    y = 0
    for i in range(8):
        nib = (x >> (4 * i)) & 0xF
        y |= (inv_sboxes[i][nib] & 0xF) << (4 * i)
    return y & MASK32

def Pinv_num(x):
    return inv_sbox_layer(rotr32(x, 11), INV_SBOXES)

def build_key_words(iv, u, v):
    L = (iv >> 32) & MASK32
    R = iv & MASK32
    k0 = (Pinv_num(u) - R) & MASK32
    k1 = (Pinv_num(v) - (L ^ u)) & MASK32
    k2 = (Pinv_num(u) - (R ^ v)) & MASK32
    k3 = (Pinv_num(v) - L) & MASK32
    return [k0, k1, k2, k3, k0, k1, k2, k3]

def words_to_hex(words):
    return "".join(f"{w & MASK32:08x}" for w in words)


while True:
    print(i)
    r = process(["python3", "server.py"])
    
    r.recvuntil(b"IV = ")
    iv = int(r.recvline().strip(), 16)
    print(iv)
    
    L = (iv >> 32) & MASK32
    R = iv & MASK32
    
    try:
        u = solve_u(L, R)
        v = solve_v(L, R)
    except RuntimeError as e:
        r.close()
        continue

    if u is None or v is None:
        r.close()
        continue
    
    w1 = build_key_words(iv, u, v)
    w2 = list(reversed(w1))
    
    if w1 == w2:
        r.close()
        continue
        
    m1 = words_to_hex(w1)
    m2 = words_to_hex(w2)
    
    r.recvuntil(b"> ")
    r.sendline(b"2")
    
    r.sendlineafter(b"m1 > ", m1.encode())
    r.sendlineafter(b"m2 > ", m2.encode())
    
    response = r.recvall().decode()
    if "Good!" in response:
        print(response.strip())
        break
    else:
        r.close()
        break  
```