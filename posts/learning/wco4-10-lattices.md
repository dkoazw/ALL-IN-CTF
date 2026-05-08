# Cryptohack

## Vectors
- Bài này cho ta $v = (2, 6, 3)$, $w = (1, 0, 0)$, $u = (7, 7, 2)$. Nhiệm vụ của ta là tính: $3 \cdot (2\cdot v - w) \cdot 2 \cdot u$.


## Size and Basis
- Bài này cho ta $v = (4, 6, 2, 5)$. Nhiệm vụ là ta cần tính size của nó.
- Đơn giản là ta chỉ cần tính $\lVert v \rVert$ với $\lVert v \rVert^2 = v \cdot v$. 

## Gram Schmidt

### Đề bài
- Bài này cho ta các vector $v_1 = (4, 1, 3, -1)$, $v_2 = (2, 1, -3, 4)$, $v_3 = (1, 0, -2, 7)$, $v_4 = (6, 2, 9, -5)$ đều thuộc không gian vector $V$. Đề bài yêu cầu ta sử dụng `Algorithm for Gram-Schmidt` để tìm cơ sở trực giao của không gian vector $V$ đó.
![1](/img/learning/wco4-lattices/1.png)

### Code
```python
from sage.all import *

v1 = vector([4, 1, 3, -1])
v2 = vector([2, 1, -3, 4])
v3 = vector([1, 0, -2, 7])
v4 = vector([6, 2, 9, -5])

def gram_schmidt(vectors):
    list_u = []
    u1 = vectors[0]
    list_u.append(u1)
    for i in range(1, len(vectors)):
        for j in range(i):
            temp = vectors[i] * list_u[j] / (list_u[j] * list_u[j])
            vectors[i] = vectors[i] - temp * list_u[j]
        list_u.append(vectors[i])
    return list_u
       
orthogonal_basis = gram_schmidt([v1, v2, v3, v4])

for u in orthogonal_basis:
    print(u)
```

## What's a Lattice? 
- Bài này yêu cầu ta tính thể tích của miền cơ bản với các vector cơ sở: $v_1 = (6,2,-3)$, $v_2 = (5, 1, 4)$, $v_3 = (2, 7, 1)$.
- Để giải bài này thì đơn giản là ta tính det ma trận thôi.

## Gaussian Reduction

### Đề bài 
- Bài này cho ta $v = (846835985,9834798552)$, $u = (87502093,123094980)$. Nhiệm vụ của ta là dùng `Algorithm for Gaussian Lattice Reduction` để giải quyết bài toán `Shortest Vector Problem` - một trong hai bài toán khó của lattices. 
![2](/img/learning/wco4-lattices/2.png)

### Ý tưởng
- Mình vẫn sẽ triển khai thuật toán này, đồng thời ta còn một thuật toán mạnh mẽ khác để giải là `Lenstra–Lenstra–Lovász lattice basis reduction algorithm (LLL)` và nó được tích hợp sẵn trong sagemath.

### Code 
```python
from sage.all import *

v = Matrix(ZZ,
    [[846835985, 9834798552],
     [87502093, 123094980],])


def Gaussian_Lattice_Reduction(v):
    while True:
        if v[1].norm() < v[0].norm():
            v[1], v[0] = v[0], v[1]

        m = v[0]*v[1]/(v[0]*v[0])   
        m = round(m)      
        if m == 0:
            return v
            
        v[1] = v[1] - m * v[0]
        
print(Gaussian_Lattice_Reduction(v))
print(v.LLL())
```

## Find the Lattice 

### Đề bài 
<details>
    <summary> <strong>source.py</strong></summary>
    
    from Crypto.Util.number import getPrime, inverse, bytes_to_long
    import random
    import math

    FLAG = b'crypto{?????????????????????}'


    def gen_key():
        q = getPrime(512)
        upper_bound = int(math.sqrt(q // 2))
        lower_bound = int(math.sqrt(q // 4))
        f = random.randint(2, upper_bound)
        while True:
            g = random.randint(lower_bound, upper_bound)
            if math.gcd(f, g) == 1:
                break
        h = (inverse(f, q)*g) % q
        return (q, h), (f, g)


    def encrypt(q, h, m):
        assert m < int(math.sqrt(q // 2))
        r = random.randint(2, int(math.sqrt(q // 2)))
        e = (r*h + m) % q
        return e


    def decrypt(q, h, f, g, e):
        a = (f*e) % q
        m = (a*inverse(f, g)) % g
        return m


    public, private = gen_key()
    q, h = public
    f, g = private

    m = bytes_to_long(FLAG)
    e = encrypt(q, h, m)

    print(f'Public key: {(q,h)}')
    print(f'Encrypted Flag: {e}')
</details>

<details>
    <summary> <strong>output.txt</strong></summary>
    
    Public key: (7638232120454925879231554234011842347641017888219021175304217358715878636183252433454896490677496516149889316745664606749499241420160898019203925115292257, 2163268902194560093843693572170199707501787797497998463462129592239973581462651622978282637513865274199374452805292639586264791317439029535926401109074800)
    Encrypted Flag: 5605696495253720664142881956908624307570671858477482119657436163663663844731169035682344974286379049123733356009125671924280312532755241162267269123486523
</details>

### Ý tưởng
- Bài này cho ta `public key` và `encrypted Flag`, nhiệm vụ của ta là cần vận dụng kiến thức từ lattices để tìm lại `private key` để tìm về flag.
- Ta có public key là `q, h` còn private key là `f, g` và công thức chung của chúng là:
```python
    def gen_key():
        q = getPrime(512)
        upper_bound = int(math.sqrt(q // 2))
        lower_bound = int(math.sqrt(q // 4))
        f = random.randint(2, upper_bound)
        while True:
            g = random.randint(lower_bound, upper_bound)
            if math.gcd(f, g) == 1:
                break
        h = (inverse(f, q)*g) % q
        return (q, h), (f, g)

```
Điều này tức là:
$$h \cdot f - k \cdot q = g$$
Đồng thời `f`, `g` cũng là một số rất nhỏ so với `q` với 512 bits.
- Vậy nên ý tưởng đặt ra là ta sẽ tạo ra một lattice gì đó có dạng là 2x2 là tối thiểu, và 2 kết quả `f`, `g` sẽ nằm trong danh sách các ma trận nhỏ nhất mà ta tìm được.
- Cái khó nhất của mọi dạng bài này luôn là tìm ra lattice cần tìm. Với bài này thì mình sẽ có ý tưởng như sau:
$$\begin{bmatrix} a & b \end{bmatrix} \cdot \begin{bmatrix} x & y \\ z & t \end{bmatrix} = \begin{bmatrix} f & g \end{bmatrix}$$
Với `a, b` là 2 hệ số tự do và `x, y, z, t` là ma trận cần tìm.
Ta có nếu theo dạng trên thì $a \cdot y + b \cdot t = g$ mà ta lại có công thức của `g` ở trên. Vì thế ta suy ra được `a = f`, `q = h`, `b = k`, `t = q`.
Kế đó ta quay về phương trình trên với $a \cdot x + b \cdot z = f \cdot x + k \cdot z = f$. Khúc này hiển nhiên `x = 1`, `z = 0`.
Từ đó mình suy ra được ma trận:
$$\begin{bmatrix} 1 & h \\ 0 & q \end{bmatrix}$$
- Đến đây thì ta dễ dàng tìm được flag.
![3](/img/learning/wco4-lattices/3.png)

### Code
```python
from sage.all import *
from Crypto.Util.number import *

q = 7638232120454925879231554234011842347641017888219021175304217358715878636183252433454896490677496516149889316745664606749499241420160898019203925115292257 
h = 2163268902194560093843693572170199707501787797497998463462129592239973581462651622978282637513865274199374452805292639586264791317439029535926401109074800 
e = 5605696495253720664142881956908624307570671858477482119657436163663663844731169035682344974286379049123733356009125671924280312532755241162267269123486523 

M = Matrix(ZZ, [
    [1, h],
    [0, q]
])

reduced_M = M.LLL()

f = reduced_M[0][0]
g = reduced_M[0][1]

def decrypt(q, h, f, g, e):
    a = (f*e) % q
    m = (a*inverse(f, g)) % g
    return m

m = decrypt(q, h, f, g, e)
flag = long_to_bytes(m)
print(flag.decode())
```

## Backpack Cryptography

### Đề bài 
<details>
    <summary> <strong>source.py</strong></summary>

    import random
    from collections import namedtuple
    import gmpy2
    from Crypto.Util.number import isPrime, bytes_to_long, inverse, long_to_bytes

    FLAG = b'crypto{??????????????????????????}'
    PrivateKey = namedtuple("PrivateKey", ['b', 'r', 'q'])

    def gen_private_key(size):
        s = 10000
        b = []
        for _ in range(size):
            ai = random.randint(s + 1, 2 * s)
            assert ai > sum(b)
            b.append(ai)
            s += ai
        while True:
            q = random.randint(2 * s, 32 * s)
            if isPrime(q):
                break
        r = random.randint(s, q)
        assert q > sum(b)
        assert gmpy2.gcd(q,r) == 1
        return PrivateKey(b, r, q)


    def gen_public_key(private_key: PrivateKey):
        a = []
        for x in private_key.b:
            a.append((private_key.r * x) % private_key.q)
        return a


    def encrypt(msg, public_key):
        assert len(msg) * 8 <= len(public_key)
        ct = 0
        msg = bytes_to_long(msg)
        for bi in public_key:
            ct += (msg & 1) * bi
            msg >>= 1
        return ct


    def decrypt(ct, private_key: PrivateKey):
        ct = inverse(private_key.r, private_key.q) * ct % private_key.q
        msg = 0
        for i in range(len(private_key.b) - 1, -1, -1):
            if ct >= private_key.b[i]:
                msg |= 1 << i
                ct -= private_key.b[i]
        return long_to_bytes(msg)


    private_key = gen_private_key(len(FLAG) * 8)
    public_key = gen_public_key(private_key)
    encrypted = encrypt(FLAG, public_key)
    decrypted = decrypt(encrypted, private_key)
    assert decrypted == FLAG

    print(f'Public key: {public_key}')
    print(f'Encrypted Flag: {encrypted}')
</details>

<details>
    <summary> <strong>source.py</strong></summary>
    
    - Hơi dài nên mình lược bỏ =)))
</details>

### Ý tưởng
- Mấu chốt của bài này thì ta chỉ cần quan tâm đến hàm `encrypt`.
```python
    def encrypt(msg, public_key):
        assert len(msg) * 8 <= len(public_key)
        ct = 0
        msg = bytes_to_long(msg)
        for bi in public_key:
            ct += (msg & 1) * bi
            msg >>= 1
        return ct
```
Ta có `msg` chính là `flag`, và `public_key` đã được tạo ở trên mà ta không cần để tâm lắm. Bản chất hàm này đang thực hiện check từng bit của flag, nếu bit đó = 1 thì cộng giá trị của `bi` trong `public_key` vào ct, nếu = 0 thì giữ nguyên. Vậy nên nó thỏa một phương trình toán học là:
$$a_1 \cdot b_1 + ... + a_n \cdot b_n = ct$$
Với `a` là từng bit của `flag`.
- Từ ý tưởng trên mình đã tự dựng lại một bài toán đơn giản kèm một lattices để xem thử vì nhiệm vụ của ta là tìm về các bit 0-1 của flag nên có thể nói độ dài của vector đó khá nhỏ. Nhiệm vụ của ta là cần tìm về msg `1101`gốc.
![4](/img/learning/wco4-lattices/4.png)
Và code lại trả về không ổn lắm
```python
from sage.all import *

M = Matrix(ZZ, [
    [10,   1,  0,  0,  0],
    [25,   0,  1,  0,  0],
    [60,  0,  0,  1,  0],
    [70,  0,  0,  0,  1],
    [-105, 0, 0, 0, 0]
])

reduced_M = M.LLL()
print(reduced_M)
"""
[ 0 -1  0 -1  1]
[ 0  0 -1  1  1]
[ 0 -1 -1  0 -1]
[ 0  2 -2 -3  0]
[ 5 -1  1  1 -1]
"""
```
- Sau đó mình đã lên mạng tìm thêm một số paper để đọc về dạng bài này `Knapsack and Subset Sum` và tìm ra được [cái này](https://eprint.iacr.org/2009/537.pdf). Đây là kiểu bài `Low-density attack` và mình chỉ đơn giản dựa trên lattices nó đề xuất để giải.

### Code
```python
from sage.all import *
from Crypto.Util.number import *

public_key = ...
ct = ...

n = len(public_key)

m = n
N = ZZ(2) ** 64

M = Matrix(ZZ, m + 1, m + 1)

for i in range(m):
    M[i, i] = 2
    M[i, m] = 2 * N * public_key[i]

for j in range(m):
    M[m, j] = 1

M[m, m] = 2 * N * ct

reduced_M = M.LLL()

def find_flag(row):
    # case 1: -1 -> 0, 1 -> 1
    bits = []

    for i in range(n):
        if row[i] == -1:
            bits.append(0)
        else:
            bits.append(1)

    bit_string = ""

    for i in range(n - 1, -1, -1):
        if bits[i] == 1:
            bit_string += "1"
        else:
            bit_string += "0"

    msg = int(bit_string, 2)
    flag = long_to_bytes(msg)

    if b"crypto" in flag:
        print(flag.decode())
        return True

    # case 2: -1 -> 1, 1 -> 0
    bits = []

    for i in range(n):
        if row[i] == -1:
            bits.append(1)
        else:
            bits.append(0)

    bit_string = ""

    for i in range(n - 1, -1, -1):
        if bits[i] == 1:
            bit_string += "1"
        else:
            bit_string += "0"

    msg = int(bit_string, 2)
    flag = long_to_bytes(msg)

    if b"crypto" in flag:
        print(flag.decode())
        return True

    return False

for rows in range(reduced_M.nrows()):
    row = reduced_M.row(rows)

    if row[m] != 0:
        continue

    if not all(row[col] == -1 or row[col] == 1 for col in range(m)):
        continue

    if find_flag(row):
        break
```
Note: paper set cho ta 1/2 nhưng mình cảm thấy hơi bất tiện nên đã nhân 2 lên. Đồng thời ở bước tìm flag mình chia thành 2 case, lý do là nếu $v$ là nghiệm thì $-v$ cũng là nghiệm nên mình phải check cả 2.