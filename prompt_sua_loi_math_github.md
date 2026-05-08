# Prompt sửa lỗi công thức toán học trong Markdown/GitHub README

Bạn hãy đóng vai một người chuyên sửa Markdown cho GitHub README, đặc biệt là các lỗi render công thức toán học LaTeX/MathJax.

---

## Mục tiêu

Tôi sẽ đưa cho bạn nội dung của một file `.md`.

Nhiệm vụ của bạn là sửa toàn bộ lỗi liên quan đến toán học để GitHub có thể render đúng.

Ưu tiên cao nhất:

- Sửa lỗi công thức không render.
- Sửa lỗi ma trận LaTeX.
- Sửa lỗi dùng `$$...$$` sai vị trí.
- Sửa lỗi công thức nằm trong bullet list.
- Sửa lỗi xuống dòng trong công thức.
- Sửa lỗi thiếu escape ký tự đặc biệt.
- Sửa lỗi công thức bị GitHub hiểu thành text thường.
- Sửa lỗi code block Markdown chưa đóng.
- Kiểm tra kích thước ma trận nếu có phép nhân ma trận.
- Giữ nguyên nội dung giải thích nhiều nhất có thể.
- Không tự ý đổi ý nghĩa toán học nếu không chắc.

---

## Quy tắc sửa

### 1. Ưu tiên dùng block `math` thay vì `$$...$$`

Với GitHub README, hãy ưu tiên đổi toàn bộ block math từ dạng:

```md
$$
...
$$
```

hoặc:

```md
$$...$$
```

sang dạng:

````md
```math
...
```
````

Ví dụ sai:

```md
$$
\begin{bmatrix}
1 & h \\
0 & q
\end{bmatrix}
$$
```

Ví dụ đúng:

````md
```math
\begin{bmatrix}
1 & h \\
0 & q
\end{bmatrix}
```
````

---

### 2. Inline math dùng `$...$`

Nếu công thức ngắn nằm trong một câu, có thể giữ dạng inline math:

```md
Ta có $h \cdot f \equiv g \pmod q$.
```

Nhưng nếu công thức dài, có ma trận, hoặc có nhiều dòng, phải chuyển sang block:

````md
```math
h \cdot f \equiv g \pmod q
```
````

---

### 3. Không viết `$$...$$` giữa đoạn văn dài

Không nên viết:

```md
Ta có công thức $$h \cdot f \equiv g \pmod q$$ nên suy ra...
```

Nên viết:

```md
Ta có công thức $h \cdot f \equiv g \pmod q$ nên suy ra...
```

hoặc nếu muốn tách rõ:

````md
Ta có:

```math
h \cdot f \equiv g \pmod q
```

Từ đó suy ra...
````

---

### 4. Công thức trong bullet list phải thụt dòng đúng

Sai:

````md
- Ta có:
```math
a + b = c
```
````

Đúng:

````md
- Ta có:

  ```math
  a + b = c
  ```
````

Nếu công thức nằm trong bullet list, block `math` phải được thụt vào 2 spaces hoặc 4 spaces.

---

### 5. Sửa ma trận plain text thành LaTeX matrix

Nếu thấy dạng:

```md
[ 1 h 0 q ]
```

hãy sửa thành:

````md
```math
\begin{bmatrix}
1 & h \\
0 & q
\end{bmatrix}
```
````

Nếu thấy dạng:

```md
[ f -k ] · [ 1 h ; 0 q ] = [ f g ]
```

hãy sửa thành:

````md
```math
\begin{bmatrix}
f & -k
\end{bmatrix}
\cdot
\begin{bmatrix}
1 & h \\
0 & q
\end{bmatrix}
=
\begin{bmatrix}
f & g
\end{bmatrix}
```
````

---

### 6. Kiểm tra kích thước ma trận

Khi sửa công thức ma trận, phải kiểm tra phép nhân có hợp lệ không.

Ví dụ sai kích thước:

```math
\begin{bmatrix}
a & b
\end{bmatrix}
\cdot
\begin{bmatrix}
x \\
y \\
z \\
t
\end{bmatrix}
```

Lỗi ở đây là ma trận `1x2` nhân với ma trận `4x1`, nên phép nhân không hợp lệ.

Nếu gặp lỗi như vậy, hãy ghi chú rõ:

```md
> Lưu ý: công thức gốc bị sai kích thước ma trận. Mình đã sửa theo ý tưởng hợp lý nhất bên dưới.
```

Sau đó sửa sang dạng đúng nếu có thể.

Ví dụ dạng đúng hơn cho bài lattice:

````md
```math
\begin{bmatrix}
f & -k
\end{bmatrix}
\cdot
\begin{bmatrix}
1 & h \\
0 & q
\end{bmatrix}
=
\begin{bmatrix}
f & g
\end{bmatrix}
```
````

---

### 7. Không để LaTeX bên trong code block thường

Nếu thấy công thức đang nằm trong code block thường như:

````md
```python
print("hello")
$$
a + b = c
$$
```
````

thì không sửa LaTeX bên trong đó, vì đó là code Python, không phải Markdown math.

Chỉ sửa khi công thức thật sự là nội dung Markdown bên ngoài code block.

---

### 8. Kiểm tra code block bị thiếu đóng

Nếu file có đoạn:

````md
```python
print("hello")
````

mà không có dòng đóng:

````md
```
````

thì phải thêm lại. Vì nếu quên đóng code block, toàn bộ phần dưới sẽ bị GitHub hiểu là code, làm công thức không render.

---

### 9. Chuẩn hóa các ký hiệu toán học

Hãy sửa các ký hiệu thường gặp như sau nếu chúng đang nằm trong công thức toán:

| Ký hiệu thô | Nên sửa thành |
|---|---|
| `*` | `\cdot` nếu là phép nhân toán học |
| `mod` | `\pmod q` hoặc `\bmod q` tùy ngữ cảnh |
| `<=` | `\le` |
| `>=` | `\ge` |
| `!=` | `\ne` |
| `inf` | `\infty` |
| `gcd` | `\gcd` |
| `sqrt` | `\sqrt{}` |
| `phi` | `\varphi` hoặc `\phi` |

Ví dụ sai:

```md
h * f mod q = g
```

Ví dụ đúng:

````md
```math
h \cdot f \equiv g \pmod q
```
````

---

### 10. Với bài lattice/LLL, giữ công thức rõ ràng

Nếu nội dung liên quan đến lattice, nên viết dạng:

````md
Ta có:

```math
h \cdot f - k \cdot q = g
```

hay:

```math
h \cdot f \equiv g \pmod q
```

Dựng lattice:

```math
L =
\begin{bmatrix}
1 & h \\
0 & q
\end{bmatrix}
```

Khi đó:

```math
\begin{bmatrix}
f & -k
\end{bmatrix}
L
=
\begin{bmatrix}
f & f h - kq
\end{bmatrix}
=
\begin{bmatrix}
f & g
\end{bmatrix}
```
````

Giải thích ngắn gọn:

```md
Vì `f`, `g` đều nhỏ so với `q`, vector `[f, g]` sẽ là một vector ngắn trong lattice. Do đó ta có thể dùng LLL để tìm lại `f`, `g`.
```

---

## Các lỗi GitHub Markdown math thường gặp

### Lỗi 1: `$$abc$$` không render

Nguyên nhân thường gặp:

- Đặt `$$...$$` giữa đoạn văn dài.
- Đặt `$$...$$` trong bullet list nhưng không thụt dòng.
- Có code block phía trên bị quên đóng.
- File không phải `.md`.
- Công thức nằm trong code block thường.
- Có dấu `$` lẻ trong cùng đoạn.

Cách sửa an toàn nhất:

````md
```math
abc
```
````

---

### Lỗi 2: Ma trận hiện thành text thường

Sai:

```md
[ 1 h 0 q ]
```

Đúng:

````md
```math
\begin{bmatrix}
1 & h \\
0 & q
\end{bmatrix}
```
````

---

### Lỗi 3: Công thức trong list bị vỡ

Sai:

````md
- Công thức:
```math
a+b=c
```
````

Đúng:

````md
- Công thức:

  ```math
  a + b = c
  ```
````

---

### Lỗi 4: Quên đóng code block

Sai:

````md
```python
def solve():
    pass

Sau đó ta có:

```math
a+b=c
```
````

Đúng:

````md
```python
def solve():
    pass
```

Sau đó ta có:

```math
a + b = c
```
````

---

## Template prompt sử dụng ngay

Copy đoạn dưới đây rồi dán vào chat khác/Codex/AI khác cùng với nội dung file `.md` cần sửa.

```md
Bạn hãy đóng vai một người chuyên sửa Markdown cho GitHub README, đặc biệt là lỗi render công thức toán học LaTeX/MathJax.

Tôi sẽ đưa cho bạn nội dung một file `.md`. Hãy sửa toàn bộ lỗi liên quan đến toán học để GitHub render đúng.

Yêu cầu bắt buộc:

1. Ưu tiên đổi block math `$$...$$` sang code block `math`:

   ````md
   ```math
   ...
   ```
   ````

2. Inline math ngắn có thể giữ dạng `$...$`.
3. Không để block math nằm cùng dòng với text thường.
4. Nếu công thức nằm trong bullet list, hãy thụt block `math` vào đúng 2 hoặc 4 spaces.
5. Sửa ma trận plain text như `[ 1 h 0 q ]` thành `\begin{bmatrix} ... \end{bmatrix}`.
6. Kiểm tra kích thước ma trận nếu có phép nhân ma trận.
7. Nếu công thức gốc sai logic toán hoặc sai kích thước ma trận, hãy ghi chú rõ bằng blockquote `> Lưu ý: ...`.
8. Không sửa LaTeX bên trong code block thường như `python`, `cpp`, `bash`, trừ khi code block đó bị mở nhầm và cần đóng lại.
9. Kiểm tra và sửa code block Markdown bị thiếu dòng đóng ```.
10. Giữ nguyên nội dung giải thích nhiều nhất có thể, không tự ý đổi biến toán học.

Sau khi sửa, hãy trả về đúng 2 phần:

## 1. Bản Markdown đã sửa

Trả về toàn bộ file đã sửa trong một code block `md`.

## 2. Các lỗi đã sửa

Liệt kê ngắn gọn các lỗi đã sửa.

Dưới đây là nội dung file cần sửa:

```md
<dán toàn bộ nội dung file cần sửa vào đây>
```
```

---

## Output mong muốn sau khi AI sửa

AI phải trả về dạng:

````md
## 1. Bản Markdown đã sửa

```md
<nội dung file đã sửa>
```

## 2. Các lỗi đã sửa

- Đổi `$$...$$` sang block ```math để GitHub render ổn định hơn.
- Sửa ma trận plain text thành `bmatrix`.
- Sửa công thức trong bullet list bằng cách thụt dòng đúng.
- Sửa lỗi code block chưa đóng.
- Sửa phép nhân ma trận sai kích thước nếu có.
````

---

## Ghi nhớ ngắn gọn

Cách viết math ổn định nhất trên GitHub README:

````md
```math
\begin{bmatrix}
1 & h \\
0 & q
\end{bmatrix}
```
````

Không nên viết công thức dài hoặc ma trận bằng `$$...$$` trong README nếu bạn đang gặp lỗi render.

