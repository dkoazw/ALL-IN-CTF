# Cryptohack

## Signing Server

### Đề bài
![1](img/learning/wco4-dsa/1.png)

<details>
    <summary> <strong>13374.py</strong></summary>
    
    #!/usr/bin/env python3

    from Crypto.Util.number import bytes_to_long, long_to_bytes
    from utils import listener

    class Challenge():
    def __init__(self):
        self.before_input = "Welcome to my signing server. You can get_pubkey, get_secret, or sign.\n"

    def challenge(self, your_input):
        if not 'option' in your_input:
            return {"error": "You must send an option to this server"}

        elif your_input['option'] == 'get_pubkey':
            return {"N": hex(N), "e": hex(E)}

        elif your_input['option'] == 'get_secret':
            secret = bytes_to_long(SECRET_MESSAGE)
            return {"secret": hex(pow(secret, E, N))}

        elif your_input['option'] == 'sign':
            msg = int(your_input['msg'], 16)
            return {"signature": hex(pow(msg, D, N))}

        else:
            return {"error": "Invalid option"}

    import builtins; builtins.Challenge = Challenge # hack to enable challenge to be run locally, see [https://cryptohack.org/faq/#listener](https://cryptohack.org/faq/#listener)
    listener.start_server(port=13374)
</details>

### Ý tưởng
- Mấu chốt của đề là ta cần đọc lại được `SECRET_MESSAGE`, và nó đã bị mã hóa ở `get_secret` bằng cách `return {"secret": hex(pow(secret, E, N))}`, còn ở `sign` thì ta phải nhập `msg` và tìm `signature` bằng cách `return {"signature": hex(pow(msg, D, N))}`.
- Ta có một tính chất quan trọng là $E \cdot D \equiv 1 (mod \ N)$ nên ta chỉ cần lấy `secret` từ `get_secret` và bỏ nó thành `msg` ở `sign` để từ đó tìm lại được `SECRET_MESSAGE` gốc. Vì thế hiển nhiên rằng ta không cần phải gọi `get_pubbkey` ra làm gì cả vì nó chẳng có tác dụng gì mấy trong bài này.

### Code 
```python
from Crypto.Util.number import *
import json
from pwn import *

r = remote('socket.cryptohack.org', 13374)

r.recvline()

payload1 = json.dumps({"option": "get_secret"}).encode()
r.sendline(payload1)
response1 = json.loads(r.recvline())
secret = response1["secret"]

payload2 = json.dumps({"option": "sign", "msg": secret}).encode()
r.sendline(payload2)
response2 = json.loads(r.recvline())
signature = response2["signature"]

print(long_to_bytes(int(signature, 16)).decode())
```

## Let's Decrypt

### Đề bài
![2](img/learning/wco4-dsa/2.png)

<details>
    <summary> <strong>13391.py</strong></summary>
    
    #!/usr/bin/env python3

    import re
    from Crypto.Hash import SHA256
    from Crypto.Util.number import bytes_to_long, long_to_bytes
    from utils import listener
    from pkcs1 import emsa_pkcs1_v15
    # from params import N, E, D

    FLAG = "crypto{?????????????????????????????????}"

    MSG = 'We are hyperreality and Jack and we own CryptoHack.org'
    DIGEST = emsa_pkcs1_v15.encode(MSG.encode(), 256)
    SIGNATURE = pow(bytes_to_long(DIGEST), D, N)


    class Challenge():
        def __init__(self):
            self.before_input = "This server validates domain ownership with RSA signatures. Present your message and public key, and if the signature matches ours, you must own the domain.\n"

        def challenge(self, your_input):
            if not 'option' in your_input:
                return {"error": "You must send an option to this server"}

            elif your_input['option'] == 'get_signature':
                return {
                    "N": hex(N),
                    "e": hex(E),
                    "signature": hex(SIGNATURE)
                }

            elif your_input['option'] == 'verify':
                msg = your_input['msg']
                n = int(your_input['N'], 16)
                e = int(your_input['e'], 16)

                digest = emsa_pkcs1_v15.encode(msg.encode(), 256)
                calculated_digest = pow(SIGNATURE, e, n)

                if bytes_to_long(digest) == calculated_digest:
                    r = re.match(r'^I am Mallory.*own CryptoHack.org$', msg)
                    if r:
                        return {"msg": f"Congratulations, here's a secret: {FLAG}"}
                    else:
                        return {"msg": f"Ownership verified."}
                else:
                    return {"error": "Invalid signature"}

            else:
                return {"error": "Invalid option"}


    import builtins; builtins.Challenge = Challenge # hack to enable challenge to be run locally, see https://cryptohack.org/faq/#listener
    listener.start_server(port=13391)
</details>

### Ý tưởng 
- Trước hết thì ở đầu code ta phải import thư viện `from pkcs1 import emsa_pkcs1_v15`, tuy nhiên đây không phải là thư viện chuẩn của python. Và thật ra là nó nằm ở link github [này](https://github.com/bdauvergne/python-pkcs1.git).
- Đi sâu vào logic bài toán thì ở `verify` nó yêu cầu ta phải khiến `digest = calculated_digest`.
```python
                msg = your_input['msg']
                n = int(your_input['N'], 16)
                e = int(your_input['e'], 16)

                digest = emsa_pkcs1_v15.encode(msg.encode(), 256)
                calculated_digest = pow(SIGNATURE, e, n)
```
- Với `msg` phải có `r = re.match(r'^I am Mallory.*own CryptoHack.org$', msg)`. Nên mình sẽ chọn luôn `msg = "I am Mallory own CryptoHack.org"`
- Lúc này ta phải chứng minh $S^e \ \\% \ n = D$. Ta có thể tự chọn e và n nên mình sẽ cho thẳng $e=1$ để biểu thức đơn giản hơn: $S \ \\% \ n = D$. Lúc này thì ta vẫn chưa thể tìm được n nên mình thực hiện giả sử $D<n$, giờ biểu thức lại trở thành: $S \equiv D \ (mod \ n)$. Và $n = \dfrac{S-D}{k}$, mình cũng lại cho $k=1$ để nó thành $n = S-D$.
- Mình sẽ chứng minh lại xem giả sử bản thân có đúng không: ta có $D<n$ và $n = S-D$ nên nó sẽ thành $2D<S$. Ta chỉ việc chạy đoạn code dưới đây để kiểm chứng:
```python
import json 
from pwn import *
from pkcs1 import emsa_pkcs1_v15
from Crypto.Util.number import *

r = remote("socket.cryptohack.org", 13391)
r.recvline()

r.sendline(json.dumps({"option": "get_signature"}))
respone1 = json.loads(r.recvline())
signature = respone1["signature"]

msg = "I am Mallory own CryptoHack.org"
digest = emsa_pkcs1_v15.encode(msg.encode(), 256)

print(2*bytes_to_long(digest) - int(signature, 16))
```

### Code
```python
import json 
from pwn import *
from pkcs1 import emsa_pkcs1_v15
from Crypto.Util.number import *

r = remote("socket.cryptohack.org", 13391)
r.recvline()

payload1 = json.dumps({"option": "get_signature"}).encode()
r.sendline(payload1)
respone1 = json.loads(r.recvline())
signature = respone1["signature"]

msg = "I am Mallory own CryptoHack.org"
digest = emsa_pkcs1_v15.encode(msg.encode(), 256)
N = int(signature, 16) - bytes_to_long(digest)
e = 1
payload2 = json.dumps({"option": "verify", "msg": msg, "N": hex(N), "e": hex(e)}).encode()
r.sendline(payload2)
r.interactive()
```

## Blinding Light

### Đề bài 
![3](img/learning/wco4-dsa/3.png)

<details>
    <summary> <strong>13376.py</strong></summary>
    
    #!/usr/bin/env python3

    from Crypto.Util.number import bytes_to_long, long_to_bytes
    from utils import listener

    FLAG = "crypto{?????????????????????????????}"
    ADMIN_TOKEN = b"admin=True"


    class Challenge():
        def __init__(self):
            self.before_input = "Watch out for the Blinding Light\n"

        def challenge(self, your_input):
            if 'option' not in your_input:
                return {"error": "You must send an option to this server"}

            elif your_input['option'] == 'get_pubkey':
                return {"N": hex(N), "e": hex(E) }

            elif your_input['option'] == 'sign':
                msg_b = bytes.fromhex(your_input['msg'])
                if ADMIN_TOKEN in msg_b:
                    return {"error": "You cannot sign an admin token"}

                msg_i = bytes_to_long(msg_b)
                return {"msg": your_input['msg'], "signature": hex(pow(msg_i, D, N)) }

            elif your_input['option'] == 'verify':
                msg_b = bytes.fromhex(your_input['msg'])
                msg_i = bytes_to_long(msg_b)
                signature = int(your_input['signature'], 16)

                if msg_i < 0 or msg_i > N:
                    # prevent attack where user submits admin token plus or minus N
                    return {"error": "Invalid msg"}

                verified = pow(signature, E, N)
                if msg_i == verified:
                    if long_to_bytes(msg_i) == ADMIN_TOKEN:
                        return {"response": FLAG}
                    else:
                        return {"response": "Valid signature"}
                else:
                    return {"response": "Invalid signature"}

            else:
                return {"error": "Invalid option"}


    import builtins; builtins.Challenge = Challenge # hack to enable challenge to be run locally, see https://cryptohack.org/faq/#listener
    listener.start_server(port=13376)
</details>

### Ý tưởng 
- Bài này sử dụng `Blinding attack` - như cái tên đề bài. Ta sẽ triển khai cách tấn công đó như sau:
    - Trước hết ở hàm `sign` ta nhập `m` nhưng không được chứa `admin=True`, sau đó ta thực hiện tính ra `signature` bằng cách `pow(msg_i, D, N)`. Sau đó dưới hàm `verify` nó thực hiện `verified = pow(signature, E, N)` và yêu cầu cụm từ `admin=True` phải có trong `verfied`.
    - Thì cách để giải bài này đó là ta sẽ chọn ra một số `r` bất kì (ở đây là 2). Sau đó khi ta phải nhập `m` thì ta set `m = m_org * 2^e` (`m_org` là cụm `admin=True`) để khi tính `signature` thì phép tính trong đó sẽ thành `m_org^d * 2^(e*d) = m_org^d * 2^(1)`. Lúc này ta khi đem xuống `verify` ở dưới thì ta chỉ việc lấy `signature` chia 2 đi để ra được `signature` chuẩn của `admin=True`.

### Code 
```python
from Crypto.Util.number import *
import json
from pwn import *

r = remote('socket.cryptohack.org', 13376, level='debug')

r.recvline()

payload1 = json.dumps({"option": "get_pubkey"}).encode()
r.sendline(payload1)
response1 = json.loads(r.recvline())
N = int(response1["N"], 16) 
E = int(response1["e"], 16)

ADMIN_TOKEN = b"admin=True"
msg = (bytes_to_long(ADMIN_TOKEN) * 2**E) % N
payload2 = json.dumps({"option": "sign", "msg": hex(msg)[2:]}).encode()
r.sendline(payload2)
response2 = json.loads(r.recvline())
signature = response2["signature"]

real_sig = (int(signature, 16) * pow(2, -1, N)) % N
payload3 = json.dumps({"option": "verify", "msg": ADMIN_TOKEN.hex(), "signature": hex(real_sig)}).encode()
r.sendline(payload3)
r.interactive()
```
- Có một lưu ý nhỏ là ở bài này, `payload2` mình phải set `hex(msg)[2:]` nhưng ở `payload3` thì lại chỉ là `hex(real_sig)`. Câu trả lời là do cách server nhận hex khác nhau, khi server nhận hex là byte thì nó sẽ không nhận kí tự `0x` cũng như khi chuyển một byte sang hex ở `ADMIN_TOKEN.hex()` thì nó cũng sẽ không có `0x`. Còn với số thì nó lại có `0x` và server nhận cũng sẽ thoải mái với điều đó hơn nên khi gửi `0x....` thì nó sẽ không bị lỗi.

## Vote for Pedro

### Đề bài 
![4](img/learning/wco4-dsa/4.png)

<details>
    <summary> <strong>13375.py</strong></summary>

    #!/usr/bin/env python3

    from Crypto.Util.number import bytes_to_long, long_to_bytes
    from utils import listener

    FLAG = "crypto{????????????????????}"


    class Challenge():
        def __init__(self):
            self.before_input = "Place your vote. Pedro offers a reward to anyone who votes for him!\n"

        def challenge(self, your_input):
            if 'option' not in your_input:
                return {"error": "You must send an option to this server"}

            elif your_input['option'] == 'vote':
                vote = int(your_input['vote'], 16)
                verified_vote = long_to_bytes(pow(vote, ALICE_E, ALICE_N))

                # remove padding
                vote = verified_vote.split(b'\00')[-1]

                if vote == b'VOTE FOR PEDRO':
                    return {"flag": FLAG}
                else:
                    return {"error": "You should have voted for Pedro"}

            else:
                return {"error": "Invalid option"}


    import builtins; builtins.Challenge = Challenge # hack to enable challenge to be run locally, see https://cryptohack.org/faq/#listener
    listener.start_server(port=13375)
</details>

<details>
    <summary> <strong>alice.key</strong></summary>
    
    N = 22266616657574989868109324252160663470925207690694094953312891282341426880506924648525181014287214350136557941201445475540830225059514652125310445352175047408966028497316806142156338927162621004774769949534239479839334209147097793526879762417526445739552772039876568156469224491682030314994880247983332964121759307658270083947005466578077153185206199759569902810832114058818478518470715726064960617482910172035743003538122402440142861494899725720505181663738931151677884218457824676140190841393217857683627886497104915390385283364971133316672332846071665082777884028170668140862010444247560019193505999704028222347577

    e = 3
</details>

### Ý tưởng 
- Bài này cho ta biểu thức $vote^e \equiv ver \ (mod \ N)$ và yêu cầu tìm lại `vote`, đồng thời `ver` sẽ có dạng `...\00...VOTE FOR PEDRO` nhưng mình đã đơn giản nó thành `...\00VOTE FOR PEDRO`. Hồi đầu mình nghĩ rằng chỉ cần tính `d` thông thường để tìm lại `vote` thôi nhưng ta không thể factor được `N`. 
- Lúc này mình nghĩ tới việc đơn giản hóa `ver` bằng cách biến nó về dạng `\00VOTE FOR PEDRO` (15 bytes), tức là ta sẽ thực hiện cắt chuỗi. Và theo góc nhìn của bit thì việc cắt chuỗi 15 bytes như này ta chỉ cần mod $2^{15\cdot8} = 2^{120}$. Để dễ hiểu hơn thì mình có code ví dụ dưới đây để kiểm chứng:
```python
from Crypto.Util.number import *

a = b"3667\00VOTE FOR PEDRO"
print(bytes_to_long(a))
print(bytes_to_long(a) % 2**120)

b = b"\00VOTE FOR PEDRO"        
print(bytes_to_long(b))
```
- Khi ta thực hiện $vote^e \equiv ver \ (mod \ 2^{120})$ thì mọi thứ đơn giản hơn rất nhiều vì $GCD(2^{120}, 3) = 1$ và ta có thể dễ dàng tính $\phi(2^{120}) = 2^{119}$ để tìm lại được `vote`.

### Code 
```python
from Crypto.Util.number import *
import json
from pwn import *

r = remote('socket.cryptohack.org', 13375)

N = 22266616657574989868109324252160663470925207690694094953312891282341426880506924648525181014287214350136557941201445475540830225059514652125310445352175047408966028497316806142156338927162621004774769949534239479839334209147097793526879762417526445739552772039876568156469224491682030314994880247983332964121759307658270083947005466578077153185206199759569902810832114058818478518470715726064960617482910172035743003538122402440142861494899725720505181663738931151677884218457824676140190841393217857683627886497104915390385283364971133316672332846071665082777884028170668140862010444247560019193505999704028222347577
e = 3
verified_vote = b'\00VOTE FOR PEDRO'

N_ = 2**120
phi = 2**119
d = pow(e, -1, phi)
vote = pow(bytes_to_long(verified_vote), d, N_)

r.recvline()
payload1 = {"option": "vote", "vote": hex(vote)}
r.sendline(json.dumps(payload1))
r.interactive()
```

## Let's Decrypt Again

### Đề bài 
![5](img/learning/wco4-dsa/5.png)

<details>
    <summary> <strong>13394.py</strong></summary>

    #!/usr/bin/env python3

    import hashlib
    import re
    import secrets
    from Crypto.Util.number import bytes_to_long, long_to_bytes, getPrime, inverse, isPrime
    from pkcs1 import emsa_pkcs1_v15
    from utils import listener
    # from params import N, E, D

    FLAG = b"crypto{????????????????????????????????????}"

    BIT_LENGTH = 768

    MSG = b'We are hyperreality and Jack and we own CryptoHack.org'
    DIGEST = emsa_pkcs1_v15.encode(MSG, BIT_LENGTH // 8)
    SIGNATURE = pow(bytes_to_long(DIGEST), D, N)
    BTC_PAT = re.compile("^Please send all my money to ([1-9A-HJ-NP-Za-km-z]+)$")


    def xor(a, b):
        assert len(a) == len(b)
        return bytes(x ^ y for x, y in zip(a, b))


    def btc_check(msg):
        alpha = "123456789ABCDEFGHJKLMNPQRSTUVWXYZabcdefghijkmnopqrstuvwxyz"
        addr = BTC_PAT.match(msg)
        if not addr:
            return False
        addr = addr.group(1)
        raw = b"\0" * (len(addr) - len(addr.lstrip(alpha[0])))
        res = 0
        for c in addr:
            res *= 58
            res += alpha.index(c)
        raw += long_to_bytes(res)

        if len(raw) != 25:
            return False
        if raw[0] not in [0, 5]:
            return False
        return raw[-4:] == hashlib.sha256(hashlib.sha256(raw[:-4]).digest()).digest()[:4]


    PATTERNS = [
        re.compile(r"^This is a test(.*)for a fake signature.$").match,
        re.compile(r"^My name is ([a-zA-Z\s]+) and I own CryptoHack.org$").match,
        btc_check
    ]


    class Challenge():
        def __init__(self):
            self.shares = [secrets.token_bytes(len(FLAG))
                        for _ in range(len(PATTERNS) - 1)]
            last_share = FLAG
            for s in self.shares:
                last_share = xor(last_share, s)
            self.shares.append(last_share)

            self.pubkey = None
            self.suffix = None

            self.before_input = "This server validates statements we make for you. Present your messages and public key, and if the signature matches ours, you must undoubtably be us. Just do it multiple times to make sure...\n"

        def challenge(self, your_input):
            if not 'option' in your_input:
                return {"error": "You must send an option to this server"}

            elif your_input['option'] == 'get_signature':
                return {
                    "N": hex(N),
                    "E": hex(E),
                    "signature": hex(SIGNATURE)
                }

            elif your_input['option'] == 'set_pubkey':
                if self.pubkey is None:
                    pubkey = int(your_input['pubkey'], 16)
                    if isPrime(pubkey):
                        return {"error": "Everyone knows RSA keys are not primes..."}
                    self.pubkey = pubkey
                    self.suffix = secrets.token_hex(32)

                    return {"status": "ok", "suffix": self.suffix}
                return {"error": "I already had one"}

            elif your_input['option'] == 'claim':
                if self.pubkey is None:
                    return {"error": "I need your modulus first, please"}

                msg = your_input['msg']
                n = self.pubkey
                e = int(your_input['e'], 16)
                index = your_input['index']
                if not (0 <= index < len(PATTERNS)):
                    return {"error": "invalid index"}

                if not msg.endswith(self.suffix):
                    return {"error": "Invalid message"}

                digest = emsa_pkcs1_v15.encode(msg.encode(), BIT_LENGTH // 8)
                calculated_digest = pow(SIGNATURE, e, n)

                if bytes_to_long(digest) == calculated_digest:
                    r = PATTERNS[index](msg[:-len(self.suffix)])
                    if r:
                        return {"msg": "Congratulations, here's a secret", "secret": self.shares[index].hex()}
                    else:
                        return {"msg": "Ownership verified."}
                else:
                    return {"error": "Invalid signature"}

            else:
                return {"error": "Invalid option"}


    import builtins; builtins.Challenge = Challenge # hack to enable challenge to be run locally, see https://cryptohack.org/faq/#listener
    listener.start_server(port=13394)
</details>

### Ý tưởng 
- Trước hết, bài này có 3 giá trị `shares` với `shares[0]`, `shares[1]` và `shares[2] = shares[0] ^ shares[1] ^ flag`. Và nhiệm vụ của ta là phải khiến server xuất ra các thằng `shares` này bằng các logic trong hàm `challenge`, cụ thể là:
    - Ta phải nhập 3 `msg` nằm trong `PATTERNS` với `msg[2]` là một địa chỉ btc thật. Sau đó ta nhập `n` (không phải là số nguyên tố và nhập trong hàm `set_pubkey`), `e`, `index`.
    - Để server trả ra `shares` thì ta buộc phải làm cho biểu thức $SIGNATURE^e \equiv digest_i \ (mod \ n)$ (ở đây `digest[i]` là các `msg` trong `PATTERNS` sau khi thực hiện `digest = emsa_pkcs1_v15.encode(msg.encode(), BIT_LENGTH // 8)`).
- Vấn đề lớn nhất của bài này là làm sao để tìm ra được `n` và tính ra được `e` cho chuẩn. Vì `n` không được phép là số nguyên tố, ban đầu mình nghĩ `n` sẽ có dạng $n = p \cdot q$ nhưng nó lại sinh ra ba vấn đề là `(p-1)`, `(q-1)` phải smooth,  $digest \ (mod \ i)$ phải nằm trong nhóm sinh của $signature \ (mod \ i)$ và đồng thời `e_i` được giải ra khi crt phải có cùng tính chẵn lẻ. 
- Vì thế thay vì chọn ra 2 số khác nhau `p`, `q` thì theo mình cách tốt nhất là chọn hẳn ra một `n = p^k` sao cho $\phi = p^k - p^{k-1}$ smooth, và giả sử $digest \ (mod \ p^k)$ không nằm trong nhóm sinh của $signature \ (mod \ p^k)$ thì ta chỉ việc chọn ra một số khác khá đơn giản ~~hoặc nhờ gemini generate từ đầu cũng được~~ và đương nhiên vì nó chỉ có một số nên sẽ không bị xung đột về tính chẵn lẻ ở đây.

### Code 
```python
from Crypto.Util.number import *
import json
from pwn import *
from sage.all import *
from pkcs1 import emsa_pkcs1_v15

r = remote('socket.cryptohack.org', 13394)

r.recvline()

payload1 = json.dumps({"option": "get_signature"}).encode()
r.sendline(payload1)
response1 = json.loads(r.recvline())
SIGNATURE = int(response1['signature'], 16)

p = 4000037
k = 40
n = p**k

payload2 = json.dumps({"option": "set_pubkey", "pubkey": hex(n)}).encode()
r.sendline(payload2)
response2 = json.loads(r.recvline())
suffix = response2['suffix']

m1 = "This is a test for a fake signature." + suffix
m2 = "My name is Zupp and I own CryptoHack.org" + suffix
m3 = "Please send all my money to 1FfmbHfnpaZjKFvyi1okTjJJusN455paPH" + suffix

dig1 = bytes_to_long(emsa_pkcs1_v15.encode(m1.encode(), 768 // 8))
dig2 = bytes_to_long(emsa_pkcs1_v15.encode(m2.encode(), 768 // 8))
dig3 = bytes_to_long(emsa_pkcs1_v15.encode(m3.encode(), 768 // 8))

n_ = Integers(n)
sig_ = n_(SIGNATURE)
dig1_ = n_(dig1)
dig2_ = n_(dig2)
dig3_ = n_(dig3)
e1_ = int(discrete_log(dig1_, sig_))
e2_ = int(discrete_log(dig2_, sig_))
e3_ = int(discrete_log(dig3_, sig_))

payload30 = json.dumps({"option": "claim", "msg": m1, "index": 0, "e": hex(e1_)}).encode() 
r.sendline(payload30)
response30 = json.loads(r.recvline())
secret0 = response30['secret']

payload31 = json.dumps({"option": "claim", "msg": m2, "index": 1, "e": hex(e2_)}).encode()
r.sendline(payload31)
response31 = json.loads(r.recvline())
secret1 = response31['secret']

payload32 = json.dumps({"option": "claim", "msg": m3, "index": 2, "e": hex(e3_)}).encode()
r.sendline(payload32)
response32 = json.loads(r.recvline())
secret2 = response32['secret']

flag = xor(xor(bytes.fromhex(secret0), bytes.fromhex(secret1)), bytes.fromhex(secret2))
print(flag.decode())
```

## Digestive

### Đề bài 
![6](img/learning/wco4-dsa/6.png)
**[play](https://web.cryptohack.org/digestive)**

<details>
    <summary> <strong>source</strong></summary>

    import hashlib
    import json
    import string
    from ecdsa import SigningKey

    SK = SigningKey.generate() # uses NIST192p
    VK = SK.verifying_key


    class HashFunc:
        def __init__(self, data):
            self.data = data

        def digest(self):
            # return hashlib.sha256(data).digest()
            return self.data



    @chal.route('/digestive/sign/<username>/')
    def sign(username):
        sanitized_username = "".join(a for a in username if a in string.ascii_lowercase)
        msg = json.dumps({"admin": False, "username": sanitized_username})
        signature = SK.sign(
            msg.encode(),
            hashfunc=HashFunc,
        )

        # remember to remove the backslashes from the double-encoded JSON
        return {"msg": msg, "signature": signature.hex()}


    @chal.route('/digestive/verify/<msg>/<signature>/')
    def verify(msg, signature):
        try:
            VK.verify(
                bytes.fromhex(signature),
                msg.encode(),
                hashfunc=HashFunc,
            )
        except:
            return {"error": "Signature verification failed"}

        verified_input = json.loads(msg)
        if "admin" in verified_input and verified_input["admin"] == True:
            return {"flag": FLAG}
        else:
            return {"error": f"{verified_input['username']} is not an admin"}
</details>


### Ý tưởng 
<img src="img/learning/wco4-dsa/6_1.png" width="450">

- Khi ta nhập username thì server sẽ trả về msg và signature, trong đó msg có thông số admin = false. Nhiệm vụ của ta là phải làm cách nào để làm giả msg, khiến cho nó tồn tại thông số admin = true để có thể tìm flag.
- Và vấn đề cũng đã được nhấn mạnh trong code:
```python
class HashFunc:
    def __init__(self, data):
        self.data = data

    def digest(self):
        # return hashlib.sha256(data).digest()
        return self.data
```
Với hàm digest, đúng ra thứ được trả về phải là hash sha256 nhưng ở đây nó lại trả về giá trị gốc. 
- Và vì thế với `NIST192p` nó sẽ nhận 24 bytes đầu trong msg. Ví dụ ở ảnh trên thì nó sẽ lấy `{"admin": false, "userna` để check. 
Từ đó ở khúc dưới thì ta có thể tùy ý thêm bất cứ thứ gì cũng được, điều này bao gồm cả việc ta add thêm `"admin": true` vào. Đó cũng là cách giải bài này.
<img src="img/learning/wco4-dsa/6_2.png" width="450">

> Nếu như hàm digest trả về hash sha256 thì bài toán sẽ hoàn toàn vô dụng bởi vì việc thêm 1 kí tự vào msg cũng khiến cho hash đó bị sai hoàn toàn.

## Curveball

### Đề bài 
![7](img/learning/wco4-dsa/7.png)

<details>
    <summary> <strong>13382.py</strong></summary>

    #!/usr/bin/env python3

    import fastecdsa
    from fastecdsa.point import Point
    from fastecdsa.curve import P256
    from utils import listener


    FLAG = "crypto{????????????????????????????????????}"
    G = P256.G
    assert G.x, G.y == [0x6B17D1F2E12C4247F8BCE6E563A440F277037D812DEB33A0F4A13945D898C296,
                        0x4FE342E2FE1A7F9B8EE7EB4A7C0F9E162BCE33576B315ECECBB6406837BF51F5]


    class Challenge():
        def __init__(self):
            self.before_input = "Welcome to my secure search engine backed by trusted certificate library!\n"
            self.trusted_certs = {
                'www.cryptohack.org': {
                    "public_key": Point(0xE9E4EBA2737E19663E993CF62DFBA4AF71C703ACA0A01CB003845178A51B859D, 0x179DF068FC5C380641DB2661121E568BB24BF13DE8A8968EF3D98CCF84DAF4A9, curve=P256),
                    "curve": "secp256r1",
                    "generator": [G.x, G.y]
                },
                'www.bing.com': {
                    "public_key": Point(0x3B827FF5E8EA151E6E51F8D0ABF08D90F571914A595891F9998A5BD49DFA3531, 0xAB61705C502CA0F7AA127DEC096B2BBDC9BD3B4281808B3740C320810888592A, curve=P256),
                    "curve": "secp256r1",
                    "generator": [G.x, G.y]
                },
                'www.gchq.gov.uk': {
                    "public_key": Point(0xDEDFC883FEEA09DE903ECCB03C756B382B2302FFA296B03E23EEDF94B9F5AF94, 0x15CEBDD07F7584DBC7B3F4DEBBA0C13ECD2D2D8B750CBF97438AF7357CEA953D, curve=P256),
                    "curve": "secp256r1",
                    "generator": [G.x, G.y]
                }
            }

        def search_trusted(self, Q):
            for host, cert in self.trusted_certs.items():
                if Q == cert['public_key']:
                    return True, host
            return False, None

        def sign_point(self, g, d):
            return g * d

        def connection_host(self, packet):
            d = packet['private_key']
            if abs(d) == 1:
                return "Private key is insecure, certificate rejected."
            packet_host = packet['host']
            curve = packet['curve']
            x, y = packet['generator']
            g = Point(x, y, curve=P256)
            Q = self.sign_point(g, d)
            cached, host = self.search_trusted(Q)
            if cached:
                return host
            else:
                self.trusted_certs[packet_host] = {
                    "public_key": Q,
                    "curve": "secp256r1",
                    "generator": G
                }
                return "Site added to trusted connections"

        def bing_it(self, s):
            return f"Hey bing! Tell me about {s}"

        #
        # This challenge function is called on your input, which must be JSON
        # encoded
        #
        def challenge(self, your_input):
            host = self.connection_host(your_input)
            if host == "www.bing.com":
                return self.bing_it(FLAG)
            else:
                return self.bing_it(host)


    import builtins; builtins.Challenge = Challenge # hack to enable challenge to be run locally, see https://cryptohack.org/faq/#listener
    listener.start_server(port=13382)
</details>

### Ý tưởng 
- Bài này cơ bản là cho đường cong P256, ta phải nhập một điểm G fake và d fake sao cho $Q = d \cdot G$ với Q là các điểm đã được trusted thì sẽ có flag, và ta không thể cho d = 1 vì khi đó bài sẽ rất dễ.
- Và mình chọn d = 2, thì lúc này đơn giản $G = Q/d$ hay $G = Q \cdot d^{-1}$. 

### Code 
```python
from sage.all import *

p = 0xFFFFFFFF00000001000000000000000000000000FFFFFFFFFFFFFFFFFFFFFFFF
a = -3 
b = 0x5AC635D8AA3A93E7B3EBBD55769886BC651D06B0CC53B0F63BCE3C3E27D2604B
n = 0xFFFFFFFF00000000FFFFFFFFFFFFFFFFBCE6FAADA7179E84F3B9CAC2FC632551

E = EllipticCurve(GF(p), [a, b])

Q = E(0x3B827FF5E8EA151E6E51F8D0ABF08D90F571914A595891F9998A5BD49DFA3531, 0xAB61705C502CA0F7AA127DEC096B2BBDC9BD3B4281808B3740C320810888592A)
d = 2 
G = Q * pow(d, -1, n)
print(G)

#{"host": "admin", "private_key": 2, "curve": "secp256r1", "generator": [15520159875205514130255899098025123715054849599936616868365830290232639266390, 35332573964480432986660122673305225849700662492297568815244635356931754804527]}
```

## ProSign 3

### Đề bài 
![8](img/learning/wco4-dsa/8.png)

<details>
    <summary> <strong>13381.py</strong></summary>

    #!/usr/bin/env python3

    import hashlib
    from Crypto.Util.number import bytes_to_long, long_to_bytes
    from ecdsa.ecdsa import Public_key, Private_key, Signature, generator_192
    from utils import listener
    from datetime import datetime
    from random import randrange

    FLAG = "crypto{?????????????????????????}"
    g = generator_192
    n = g.order()


    class Challenge():
        def __init__(self):
            self.before_input = "Welcome to ProSign 3. You can sign_time or verify.\n"
            secret = randrange(1, n)
            self.pubkey = Public_key(g, g * secret)
            self.privkey = Private_key(self.pubkey, secret)

        def sha1(self, data):
            sha1_hash = hashlib.sha1()
            sha1_hash.update(data)
            return sha1_hash.digest()

        def sign_time(self):
            now = datetime.now()
            m, n = int(now.strftime("%m")), int(now.strftime("%S"))
            current = f"{m}:{n}"
            msg = f"Current time is {current}"
            hsh = self.sha1(msg.encode())
            sig = self.privkey.sign(bytes_to_long(hsh), randrange(1, n))
            return {"msg": msg, "r": hex(sig.r), "s": hex(sig.s)}

        def verify(self, msg, sig_r, sig_s):
            hsh = bytes_to_long(self.sha1(msg.encode()))
            sig_r = int(sig_r, 16)
            sig_s = int(sig_s, 16)
            sig = Signature(sig_r, sig_s)

            if self.pubkey.verifies(hsh, sig):
                return True
            else:
                return False

        #
        # This challenge function is called on your input, which must be JSON
        # encoded
        #
        def challenge(self, your_input):
            if 'option' not in your_input:
                return {"error": "You must send an option to this server"}

            elif your_input['option'] == 'sign_time':
                signature = self.sign_time()
                return signature

            elif your_input['option'] == 'verify':
                msg = your_input['msg']
                r = your_input['r']
                s = your_input['s']
                verified = self.verify(msg, r, s)
                if verified:
                    if msg == "unlock":
                        self.exit = True
                        return {"flag": FLAG}
                    return {"result": "Message verified"}
                else:
                    return {"result": "Bad signature"}

            else:
                return {"error": "Decoding fail"}


    import builtins; builtins.Challenge = Challenge # hack to enable challenge to be run locally, see https://cryptohack.org/faq/#listener
    listener.start_server(port=13381)
</details>

### Ý tưởng 
- Bài này dựa trên ECDSA
![8_1](img/learning/wco4-dsa/8_1.png)
- Khi connect vào server thì thứ ta nhận được là m, r, s. Và về cơ bản nhiệm vụ của ta tức là phải tìm cách chứng minh ta thành chủ tin nhắn bằng cách xác định được d và k để tạo ra một cặp r, s hợp lệ. 
![8_2](img/learning/wco4-dsa/8_2.png)
- Như trong đây đề cập thì trước hết, ta phải tìm được thằng d bằng cách $d \equiv r_{old}^{-1} (s_{old} * k_{old} - h_{old}) \pmod n$. Và $k_{old}$ ở đây chính là thằng k đã hình thành nên cặp $r_{old}$, $s_{old}$ và $h_{old}$ mà server gửi cho ta bằng cách: 
```python
class Challenge():
    def __init__(self):
        self.before_input = "Welcome to ProSign 3. You can sign_time or verify.\n"
        secret = randrange(1, n)
        self.pubkey = Public_key(g, g * secret)
        self.privkey = Private_key(self.pubkey, secret)

    def sha1(self, data):
        sha1_hash = hashlib.sha1()
        sha1_hash.update(data)
        return sha1_hash.digest()

    def sign_time(self):
        now = datetime.now()
        m, n = int(now.strftime("%m")), int(now.strftime("%S"))
        current = f"{m}:{n}"
        msg = f"Current time is {current}"
        hsh = self.sha1(msg.encode())
        sig = self.privkey.sign(bytes_to_long(hsh), randrange(1, n))
        return {"msg": msg, "r": hex(sig.r), "s": hex(sig.s)}
```
Ban đầu nhìn vào thì ta cứ tưởng secret (tức k) lấy random từ [1, order(E)], nhưng thật chất n ở đây lại chính là thằng n ở dưới `sign_time` tức là giây hiện tại. Vậy k chỉ đơn thuần là chạy từ [1, 60]. Và ta có thể bruteforce $k_{old}$ để tìm ra d.
- Sau đó tạo ra một thằng $k_{new}$ mới + d ta vừa tính ra để ký lại và tìm ra s, r (ở đây mình chọn k = 1 vì server không cấm).
- Và vì k = 1 nên lúc này r = G.x luôn. Còn s thì cứ tính theo công thức với h ở đây là hash message "unlock".

### Code 
```python
from pwn import *
import json
import hashlib
from sage.all import *

p = 0xfffffffffffffffffffffffffffffffeffffffffffffffff
a = 0xfffffffffffffffffffffffffffffffefffffffffffffffc
b = 0x64210519e59c80e70fa7e9ab72243049feb8deecc146b9b1
n = 0xffffffffffffffffffffffff99def836146bc9b1b4d22831
E = EllipticCurve(GF(p), [a, b])
G = E(0x188da80eb03090f67cbf20eb43a18800f4ff0afd82ff1012, 0x07192b95ffc8da78631011ed6b24cdd573f977a11e794811)

r = remote('socket.cryptohack.org', 13381)

r.recvline()

payload1 = json.dumps({"option": "sign_time"}).encode()
r.sendline(payload1)

response1 = json.loads(r.recvline())

msg = response1['msg']
r_old = int(response1['r'], 16)
s_old = int(response1['s'], 16)

hash_msg = int(hashlib.sha1(msg.encode()).hexdigest(), 16)
hash_unlock = int(hashlib.sha1(b"unlock").hexdigest(), 16)

for k_old in range(1, 61):
    d = (pow(r_old, -1, n) * (s_old * k_old - hash_msg)) % n

    r_new = int(G[0])
    s_new = (hash_unlock + r_new*d)%n
    payload2 = json.dumps({"option": "verify", "msg": "unlock", "r": hex(r_new), "s": hex(s_new)}).encode()
    r.sendline(payload2)
    response2 = json.loads(r.recvline()) 
    if "flag" in response2:
        flag = response2["flag"]
        print(flag)
        break
    else:
        continue
```

## No Random, No Bias 

### Đề bài 
![9](img/learning/wco4-dsa/9.png)

<details>
    <summary> <strong>13381.py</strong></summary>

    from hashlib import sha1
    from Crypto.Util.number import bytes_to_long, long_to_bytes
    from ecdsa import ellipticcurve
    from ecdsa.ecdsa import curve_256, generator_256, Public_key, Private_key
    from random import randint

    G = generator_256
    q = G.order()

    FLAG = b'crypto{??????????????????}'


    def hide_flag(privkey):
        x = bytes_to_long(FLAG)
        p = curve_256.p()
        b = curve_256.b()
        ysqr = (x**3 - 3*x + b) % p
        y = pow(ysqr, (p+1)//4, p)
        Q = ellipticcurve.Point(curve_256, x, y)
        T = privkey.secret_multiplier*Q
        return (int(T.x()), int(T.y()))


    def genKeyPair():
        d = randint(1,q-1)
        pubkey = Public_key(G, d*G)
        privkey = Private_key(pubkey, d)
        return pubkey, privkey


    def ecdsa_sign(msg, privkey):
        hsh = sha1(msg.encode()).digest()
        nonce = sha1(long_to_bytes(privkey.secret_multiplier) + hsh).digest()
        sig = privkey.sign(bytes_to_long(hsh), bytes_to_long(nonce))
        return {"msg": msg, "r": hex(sig.r), "s": hex(sig.s)}



    pubkey, privkey = genKeyPair()
    hidden_flag = hide_flag(privkey)

    sig1 = ecdsa_sign('I have hidden the secret flag as a point of an elliptic curve using my private key.', privkey)
    sig2 = ecdsa_sign('The discrete logarithm problem is very hard to solve, so it will remain a secret forever.', privkey)
    sig3 = ecdsa_sign('Good luck!', privkey)

    print('Hidden flag:', hidden_flag)
    print('\nPublic key:', (int(pubkey.point.x()), int(pubkey.point.y())), '\n')
    print(sig1)
    print(sig2)
    print(sig3)
</details>

### Ý tưởng
- Đọc đề thì ta có thể đoán ngay bài này là `biased nonce attack`. Và đúng thật khi `nonce` chỉ là một số 160 bit do `sha1` còn `n` thì lại là 256 bit. Giờ nhiệm vụ của ta là cần tìm lại `d`.
- Khi đã chắc chắn thì ta sẽ tiến hành biến đổi lại công thức của `s` thành:
$$(s^{-1}_i \cdot r_i)d - (-s_i^{-1} \cdot h_i) \equiv k \pmod{n}$$ 
- Đây là phương trình `HNP` dạng:
$$t_i \cdot \alpha - a_i \equiv b_i \pmod{p}$$
- Từ đó ta dựng lên được 1 cơ sở lattice với $B = 2^{160}$:
$$\mathbf{B} = \begin{bmatrix} p & 0 & 0 & 0 & 0 \\\\0 & p & 0 & 0 & 0 \\\\0 & 0 & p & 0 & 0 \\\\t_1 &t_2 & t_3 & B/p & 0 \\\\a_1 & a_2 & a_3 & 0 & B\end{bmatrix}$$
- Sau đó ta áp dụng `LLL` và output cũng sẽ có dạng là ma trận 5x5 với từng hàng lần lượt là $(b_1, b_2, b_3,\alpha B/p,B)$. Từ L[3] ta sẽ tính ngược lại các giá trị $\alpha$ và kiểm tra xem giá trị nào là hợp lệ với bài toán. 
- Sau khi có $\alpha$ tức `d` thì ta chỉ việc tìm flag như bình thường.
[DSA/ECDSA and biased-nonce attack](https://sruthisekar.wordpress.com/wp-content/uploads/2024/11/ct13.pdf)
[Some Elliptic Curve Cryptography Attacks](https://hackmd.io/h58BVyU6TtysmLI1gOH7YA?both#Biased-Nonces)

### Code 
```python
from hashlib import sha1
from sage.all import *
from Crypto.Util.number import long_to_bytes

p = 0xFFFFFFFF00000001000000000000000000000000FFFFFFFFFFFFFFFFFFFFFFFF
a = -3
b = 0x5AC635D8AA3A93E7B3EBBD55769886BC651D06B0CC53B0F63BCE3C3E27D2604B
n = 0xFFFFFFFF00000000FFFFFFFFFFFFFFFFBCE6FAADA7179E84F3B9CAC2FC632551

E = EllipticCurve(GF(p), [a, b])

Gx = 0x6b17d1f2e12c4247f8bce6e563a440f277037d812deb33a0f4a13945d898c296
Gy = 0x4fe342e2fe1a7f9b8ee7eb4a7c0f9e162bce33576b315ececbb6406837bf51f5
G = E(Gx, Gy)

Pubx = 48780765048182146279105449292746800142985733726316629478905429239240156048277
Puby = 74172919609718191102228451394074168154654001177799772446328904575002795731796
Pub = E(Pubx, Puby)

Hidx = 16807196250009982482930925323199249441776811719221084165690521045921016398804
Hidy = 72892323560996016030675756815328265928288098939353836408589138718802282948311
Hid = E(Hidx, Hidy)

msg1 = "I have hidden the secret flag as a point of an elliptic curve using my private key."
h1 = sha1(msg1.encode()).digest()
h1 = int(h1.hex(), 16)
r1 = 0x91f66ac7557233b41b3044ab9daf0ad891a8ffcaf99820c3cd8a44fc709ed3ae
s1 = 0x1dd0a378454692eb4ad68c86732404af3e73c6bf23a8ecc5449500fcab05208d

msg2 = "The discrete logarithm problem is very hard to solve, so it will remain a secret forever."
h2 = sha1(msg2.encode()).digest()
h2 = int(h2.hex(), 16)
r2 = 0xe8875e56b79956d446d24f06604b7705905edac466d5469f815547dea7a3171c
s2 = 0x582ecf967e0e3acf5e3853dbe65a84ba59c3ec8a43951bcff08c64cb614023f8

msg3 = "Good luck!"
h3 = sha1(msg3.encode()).digest()
h3 = int(h3.hex(), 16)
r3 = 0x566ce1db407edae4f32a20defc381f7efb63f712493c3106cf8e85f464351ca6
s3 = 0x9e4304a36d2c83ef94e19a60fb98f659fa874bfb999712ceb58382e2ccda26ba

t1 = (r1 * pow(s1, -1, n)) % n
a1 = (h1 * pow(s1, -1, n)) % n

t2 = (r2 * pow(s2, -1, n)) % n
a2 = (h2 * pow(s2, -1, n)) % n

t3 = (r3 * pow(s3, -1, n)) % n
a3 = (h3 * pow(s3, -1, n)) % n

B = 2**160
M = Matrix(QQ, [
    [n,  0,  0,    0, 0],
    [0,  n,  0,    0, 0],
    [0,  0,  n,    0, 0],
    [t1, t2, t3, QQ(B) // QQ(n), 0],
    [a1, a2, a3,     0, B],
])
L = M.LLL()

d = 0
for i in range(L.nrows()):
    row = L.row(i)
    target = row[3]
    d_test = QQ(target) * QQ(n) / QQ(B)
    d_test = d_test % n
    if d_test != 0 and d_test * G == Pub:
        d = d_test
        break

Q = pow(d, -1, n) * Hid
print(long_to_bytes(int(Q[0])).decode())
```


# Hackropole

## El Gamal Fait 1/2

### Đề bài 

<details>
    <summary> <strong>el-gamal-fait-1.py</strong></summary>

    from Crypto.Random.random import randrange
    from Crypto.Util.number import getPrime

    def generate(bits):
        p = getPrime(bits)
        x = randrange(p)
        g = randrange(2, p)
        y = pow(g,x,p)
        return p, g, x, y

    def sign(p, g, x, m):
        k = randrange(p)
        r = pow(g, k, p)
        inv_k = pow(k, -1, p - 1)
        s = ((m - x * r) * inv_k) % (p - 1)
        return r, s

    def verify(p, g, y, m, r, s):
        if r <= 0 or r >= p - 1 or s < 0 or s >= p - 1:
            return False
        return pow(g, m, p) == ((pow(y, r, p) * pow(r, s, p)) % p)

    print("Public key:")
    p, g, x, y = generate(2048)
    print(f"{p = }")
    print(f"{g = }")
    print(f"{y = }")

    try:
        print("Input a message.")
        m = int(input(">>> "))

        print("Input a signature. First, input r.")
        r = int(input(">>> "))

        print("Now, input s.")
        s = int(input(">>> "))

        if verify(p, g, y, m, r, s):
            print("Congratulations! The message and signature match. Here is your flag:")
            print(open("flag.txt").read())
        else:
            print("Better luck next time!")
    except:
        print("Please check your inputs!")
</details>

### Ý tưởng 
- Trước tiên, đây là một bài toán theo kiểu `ElGamal signature`, nôm na là nó sẽ có p, g, y là 3 public key, message m, với 2 private key r, s. 

- Đây là lý thuyết liên quan đến `ElGamal signature`:
    - Cách để tạo ra private key gốc của `ElGamal signature` là:
    - Chọn một số $1<k<p-1$  sao cho GCD(k,p-1)==1 (cốt là để tồn tại $k^{-1} \ (mod \ p-1)$.
    - Tính $r = g^k \pmod p$.
    - Tạo ra h = H(m), ta phải băm message ra thành h, **khúc này quan trọng** .
    - Tính $s = (h - xr) \cdot k^{-1} \pmod{p-1}$.
    - Và từ đó ta đã có s, r.
    - Cuối cùng ta được công thức tổng quát như sau:
    $$g^{H(m)} \equiv (y)^r (r)^s \pmod p$$

- Quay ngược lại bài toán, như ta thấy thì nó không yêu cầu phải băm message, và vì thế ta có thể chọn một message bất kì hay cụ thể ở đây mình sẽ chọn m = 0. Lúc này công thức tổng quát của ta sẽ thành:
$$1 \equiv g^{0} \equiv (y)^r (r)^s \pmod p$$
- Bây giờ việc duy nhất của ta là phải tìm r, s làm sao cho nó cũng = 1.
- Lúc này giữa trên cảm tình của mình (làm đại) thì ta nhận thấy rằng, mình sẽ cho s = r, lúc này vế phải sẽ thành:
$$(y*r)^r$$
- Giờ để cho nó = 1 thì dễ thấy nhất, r sẽ bằng pow(y,-1,p). Lúc này vế phải sẽ = 1. Hết bài.

### Code
```python
from pwn import *

c = remote("localhost", 4000)
c.recvuntil(b"p =")
p = int(c.recvline())
c.recvuntil(b"g =")
g = int(c.recvline())
c.recvuntil(b"y =")
y = int(c.recvline())

m = 0
r = pow(y,-1,p)
s = r 

c.recvuntil(b">>> ")
c.sendline(str(m).encode())
c.recvuntil(b">>> ")
c.sendline(str(r).encode())
c.recvuntil(b">>> ")
c.sendline(str(r).encode())
c.interactive()
```

## El Gamal Fait 2/2

### Đề bài
<details>
    <summary> <strong>el-gamal-fait-2.py</strong></summary>

    from Crypto.Random.random import randrange
    from Crypto.Util.number import getPrime

    def generate(bits):
        p = 2
        while p % 4 != 1:
            p = getPrime(bits)
        x = randrange(p)
        g = 2
        y = pow(g, x, p)
        return p, g, x, y

    def sign(p, g, x, m):
        k = randrange(p)
        r = pow(g, k, p)
        inv_k = pow(k, -1, p - 1)
        s = ((m - x * r) * inv_k) % (p - 1)
        return r, s

    def verify(p, g, y, m, r, s):
        if r <= 0 or r >= p or s < 0 or s >= p - 1:
            return False
        return pow(g, m, p) == ((pow(y, r, p) * pow(r, s, p)) % p)

    print("Public key:")
    p, g, x, y = generate(2048)
    print(f"{p = }")
    print(f"{g = }")
    print(f"{y = }")

    try:
        m = randrange(p)
        print(f"Your task is to sign the following message m = {m}")

        print("Input a signature. First, input r.")
        r = int(input(">>> "))

        print("Now, input s.")
        s = int(input(">>> "))

        if verify(p, g, y, m, r, s):
            print("Congratulations! The message and signature match. Here is your flag:")
            print(open("flag.txt").read())
        else:
            print("Better luck next time!")
    except:
        print("Please check your inputs!")
</details>

### Ý tưởng
- Bài này khác với bài trước là ta sẽ không được tự ý nhập m, tuy nhiên g luôn cố định = 2. Và lúc này công thức tổng quát của ta sẽ là:
$$2^m \equiv y^r*r^s \pmod p$$
- Vấn đề lúc này thì bản thân mình nghĩ sẽ phụ thuộc phần lớn vào kĩ năng biến đổi và sự nhạy bén với số liệu của mỗi người cũng như là "ăn hên". 
- Với mình thì ban đầu mình nghĩ rằng s phải liên quan đến m hoặc -m hoặc đại loại gì đó và cũng như ta phải tìm một cách nào đó để $y^r$ thành 1 hoặc làm sao để nó không ảnh hưởng tới phần cơ số vì cơ số luôn cố dịnh = 2. Và lúc đó mình đã nghĩ tới định lý fermat nhỏ: $y^{p-1} \equiv 1 \ (mod \ p)$, tức là r = p-1. Nhưng sau một lúc biến đổi thì nó lại thành ra như này:
$$2^m \equiv 1*(p-1)^m \pmod p$$
=>
$$2^m \equiv (-1)^m \pmod p$$
- Lúc này chắc chắn là nó sai, và trong lúc đó thì mình vô tình rùa ra bằng cách để ý rằng $p \equiv 1 \ (mod \ 4)$, và lúc này mình nghĩ tới công thức $(x/p) \equiv x^{(p-1)/2} \ (mod \ p)$ của `Legendre Symbol` 
> cái này hình như là rùa vì thực chất dữ kiện đó của p hình như giúp cho nó không rơi vào safe prime chứ chẳng liên quan gì tới cái này hết thì phải =)))
- Lúc này mình đã biến đổi như sau:
$$2^m \equiv y^{(p-1)/2}* (\frac{p-1}{2})^{-m} \pmod p$$
=>
$$2^m \equiv y^{(p-1)/2}* ((\frac{-1}{2})^{-1})^m \pmod p$$
=> 
$$2^m \equiv y^{(p-1)/2}* (-2)^m \pmod p$$
- Thì cái này sẽ đúng trong cụ thể 2 trường hợp: 1 là khi $y^{(p-1)/2}$ = 1 và m là số chẵn, 2 là khi $y^{(p-1)/2}$ = -1 và m là số lẻ.
- Lúc này gần như là hết bài trừ việc ta phải biến đổi một chút làm sao cho thỏa điều kiện của hàm verify trong source code.

<details>
<summary><strong>raw.jpeg</strong></summary>

<img src="img/learning/wco4-dsa/raw.jpeg" width="450">

</details>

### Code
```python
from pwn import *

while True:
    c = remote("localhost", 4000)
    c.recvuntil(b"p = ")
    p = int(c.recvline())
    c.recvuntil(b"g = ")
    g = int(c.recvline())
    c.recvuntil(b"y = ")
    y = int(c.recvline())
    c.recvuntil(b"m = ")
    m = int(c.recvline())

    r = (p-1)//2
    s = -m % (p-1)

    c.recvuntil(b">>> ")
    c.sendline(str(r).encode())
    c.recvuntil(b">>> ")
    c.sendline(str(s).encode())

    response = c.recvall()
    if(b"FCSC" in response):
        print(response.decode())
        break
```
