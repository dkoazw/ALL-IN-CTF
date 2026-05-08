# Modular Arithmetic

## Greatest Common Divisor

### Đề bài 
![1](/img/learning/wco4-math/1.png)

### Ý tưởng
- Ta sẽ sử dụng `euclid's algorithm` để giải.

### Code
```python=
def gcd(a,b):
    while b>0:
        b, a = a%b, b
    return a

a=66528
b=52920
print(gcd(a,b))
```

## Extended GCD

### Đề bài
![2](/img/learning/wco4-math/2.png)

### Ý tưởng
- Mục tiêu của bài này là tìm `egcd` của 2 số, nó được biểu diễn bằng công thức sau:

```math
a \cdot p + b \cdot q = \gcd(a, b)
```
- Và để tìm `egcd` thì mình có một test case nhỏ như sau: Cho a = 26, b =15. Từ đó ta sẽ lập được 1 bảng egcd:

| nums | p   | q   | Note    |
| ---- | --- | --- | --- |
| 26   | 1   | 0   |  dòng này là dòng setup, chỉ đơn giản p ứng với a = 26  |
| 15   | 0   | 1   |    dòng này tương tự như trên |
| 11   | 1   | -1  |   11 = 26 - 15 \| lúc này  p = 1, q = -1  |
| 4    | -1  | 2   |   4 = 15 - 11 = 15 - (26 - 15) = -26 + 2*15 \| ta có thể bắt đầu hình dung thuật toán là p[i] = p[i-2] - p[i-1] * (nums[i-2]//nums[i-1]|.
| 3    | 3   | -5  |  2 dòng cuối tương tự như trên    |
| 1    | -4  | 7   |   ...  |

Để setup thuật toán, ta chỉ việc tạo ra các list như trên, và điều kiện dừng là nums[i]!=0.

Note: egcd chinh là thuật toán cốt lõi để có thể tinh được $d^{-1} \ (mod \ n)$ trong RSA và cũng như rất nhiều hệ mật khác như Affine. Tuy nhiên python lại hỗ trợ rất tốt trong việc tìm nghịch đảo khiến trở nên dễ hơn rất nhiều (nhưng vẫn có một số ngoại lệ).
### Code 
```python=
def egcd(a,b):
    num = [a,b]
    p = [1,0]
    q = [0,1]
    while num[-1] != 0:
        p_temp = p[-2] - p[-1]*(num[-2]//num[-1])
        p.append(p_temp)
        q_temp = q[-2] - q[-1]*(num[-2]//num[-1])
        q.append(q_temp)
        num_temp = num[-2] % num[-1]
        num.append(num_temp)
    return (num[-2], p[-2], q[-2])
p=26513
q=32321

print(egcd(p,q))
```

## Modular Arithmetic 1

### Đề bài 
![3](/img/learning/wco4-math/3.png)

### Ý tưởng
- Như đề.

### Code
```python=
a = 11%6
b = 8146798528947%17
print(min(a,b)) 
```

## Modular Arithmetic 2

### Đề bài
![4](/img/learning/wco4-math/4.png)

### Ý tưởng
Theo `Fermat's little theorem` - Định lý fermat nhỏ thì:

```math
a^{p-1} \equiv 1 \pmod{p}
```

Và cũng như là:

```math
a^{p} \equiv a \pmod{p}
```

### Code 
- Không có.

## Modular Inverting

### Đề bài 
![5](/img/learning/wco4-math/5.png)

### Ý tưởng
- Như đã đề cập ở bài trên thì egcd chính là thuật toán cốt lõi để giải bài này, và mình sẽ chứng minh như sau:
 Ta có $a*b \ \equiv 1 \ (mod \ c)$, từ đó => $a*b - 1 \ ⋮ \ c$
Lúc này ta về lại với thuật toán egcd, ta có rằng $a*p + c*q = 1$, với a,c là 2 số nguyên tố cùng nhau. Từ đó suy ra $a*p - 1 \ = \ q*c$ và cũng như $a*p - 1 \ ⋮ \ c$
- Vậy lúc này ta có thể suy ra rằng $b = p$.

Note: Tuy nhiên trong python ta có thể sử dụng hàm pow để giải quyết bài toán này: b = pow(a,-1,c).


### Code
```python=
print(pow(3,-1,13)) 
```

## Quadratic Residues

### Đề bài
![6](/img/learning/wco4-math/6.png)

### Ý tưởng
- Ta gọi x là `Quadratic Residue` - lượng dư bậc 2 khi tồn tại a<29 (vì trong đề đang là 29):

  ```math
  a^2 \equiv x \pmod{p}
  ```
- Tuy nhiên ta chỉ cần tìm a trong khoảng p/2 vì ví dụ khi a = 1, $a^2 = 1$., còn khi a = 28, lúc này nó tương đương a = -1 và $a^2 = 1$.

- Về lại bài toán, ta chỉ việc bruteforce để ra được 2 số và tìm số nhỏ nhất.

### Code
```python=
p = 29
ints = [14,6,11]
for i in range(29):
    if pow(i,2,p) in ints :
        print(i) 
```

## Legendre Symbol

### Đề bài
![7](/img/learning/wco4-math/7.png)
<details>
    <summary> <strong>output.txt</strong></summary>

    p = 101524035174539890485408575671085261788758965189060164484385690801466167356667036677932998889725476582421738788500738738503134356158197247473850273565349249573867251280253564698939768700489401960767007716413932851838937641880157263936985954881657889497583485535527613578457628399173971810541670838543309159139

    ints = [25081841204695904475894082974192007718642931811040324543182130088804239047149283334700530600468528298920930150221871666297194395061462592781551275161695411167049544771049769000895119729307495913024360169904315078028798025169985966732789207320203861858234048872508633514498384390497048416012928086480326832803, 45471765180330439060504647480621449634904192839383897212809808339619841633826534856109999027962620381874878086991125854247108359699799913776917227058286090426484548349388138935504299609200377899052716663351188664096302672712078508601311725863678223874157861163196340391008634419348573975841578359355931590555, 17364140182001694956465593533200623738590196990236340894554145562517924989208719245429557645254953527658049246737589538280332010533027062477684237933221198639948938784244510469138826808187365678322547992099715229218615475923754896960363138890331502811292427146595752813297603265829581292183917027983351121325, 14388109104985808487337749876058284426747816961971581447380608277949200244660381570568531129775053684256071819837294436069133592772543582735985855506250660938574234958754211349215293281645205354069970790155237033436065434572020652955666855773232074749487007626050323967496732359278657193580493324467258802863, 4379499308310772821004090447650785095356643590411706358119239166662089428685562719233435615196994728767593223519226235062647670077854687031681041462632566890129595506430188602238753450337691441293042716909901692570971955078924699306873191983953501093343423248482960643055943413031768521782634679536276233318, 85256449776780591202928235662805033201684571648990042997557084658000067050672130152734911919581661523957075992761662315262685030115255938352540032297113615687815976039390537716707854569980516690246592112936796917504034711418465442893323439490171095447109457355598873230115172636184525449905022174536414781771, 50576597458517451578431293746926099486388286246142012476814190030935689430726042810458344828563913001012415702876199708216875020997112089693759638454900092580746638631062117961876611545851157613835724635005253792316142379239047654392970415343694657580353333217547079551304961116837545648785312490665576832987, 96868738830341112368094632337476840272563704408573054404213766500407517251810212494515862176356916912627172280446141202661640191237336568731069327906100896178776245311689857997012187599140875912026589672629935267844696976980890380730867520071059572350667913710344648377601017758188404474812654737363275994871, 4881261656846638800623549662943393234361061827128610120046315649707078244180313661063004390750821317096754282796876479695558644108492317407662131441224257537276274962372021273583478509416358764706098471849536036184924640593888902859441388472856822541452041181244337124767666161645827145408781917658423571721, 18237936726367556664171427575475596460727369368246286138804284742124256700367133250078608537129877968287885457417957868580553371999414227484737603688992620953200143688061024092623556471053006464123205133894607923801371986027458274343737860395496260538663183193877539815179246700525865152165600985105257601565]
</details>


### Ý tưởng
- Trước hết, để biết một số có phải là lượng dư bâc 2 không thì ta có `Legendre Symbol`, có công thức như sau:

```math
(x/p) \equiv x^{(p-1)/2} \pmod{p}
```

Khi (x/p) = 1 thì nó là lượng dư bậc 2, các trường hợp còn lại có thể xem ảnh.

- Và bình thường ta không thể cứ bruteforce để tính lượng dư bậc 2 được, từ đó ta sẽ chia ra được 2 trường hợp để tìm lượng dư bậc 2, và trường hợp dễ trong đó là khi $p \equiv 3 \pmod{4}$. Ta có công thức như sau:

```math
a \equiv x^{(p+1)/4} \pmod{p}
```
(Vì $p+1 \ ⋮ \ 4$ nên phép tính trên mới thỏa mãn)
- Mình sẽ chứng minh theo chiều nghịch như sau:
```math
a \equiv x^{(p+1)/4} \pmod{p}
```

=>

```math
a^2 \equiv x^{(p+1)/2} \pmod{p}
```

Mà $x^{(p-1)/2} \equiv 1 \pmod{p}$ nên ta có:

```math
a^2 \equiv x^{(p+1)/2} \cdot x^{(p-1)/2} \pmod{p}
```

Điều này tương đương:

```math
a^2 \equiv x^p \equiv x \pmod{p}
```
Và định lý fermat nhỏ cũng chính là mấu chốt của phép tính này.

### Code
```python=
p = 101524035174539890485408575671085261788758965189060164484385690801466167356667036677932998889725476582421738788500738738503134356158197247473850273565349249573867251280253564698939768700489401960767007716413932851838937641880157263936985954881657889497583485535527613578457628399173971810541670838543309159139

ints = [25081841204695904475894082974192007718642931811040324543182130088804239047149283334700530600468528298920930150221871666297194395061462592781551275161695411167049544771049769000895119729307495913024360169904315078028798025169985966732789207320203861858234048872508633514498384390497048416012928086480326832803, 45471765180330439060504647480621449634904192839383897212809808339619841633826534856109999027962620381874878086991125854247108359699799913776917227058286090426484548349388138935504299609200377899052716663351188664096302672712078508601311725863678223874157861163196340391008634419348573975841578359355931590555, 17364140182001694956465593533200623738590196990236340894554145562517924989208719245429557645254953527658049246737589538280332010533027062477684237933221198639948938784244510469138826808187365678322547992099715229218615475923754896960363138890331502811292427146595752813297603265829581292183917027983351121325, 14388109104985808487337749876058284426747816961971581447380608277949200244660381570568531129775053684256071819837294436069133592772543582735985855506250660938574234958754211349215293281645205354069970790155237033436065434572020652955666855773232074749487007626050323967496732359278657193580493324467258802863, 4379499308310772821004090447650785095356643590411706358119239166662089428685562719233435615196994728767593223519226235062647670077854687031681041462632566890129595506430188602238753450337691441293042716909901692570971955078924699306873191983953501093343423248482960643055943413031768521782634679536276233318, 85256449776780591202928235662805033201684571648990042997557084658000067050672130152734911919581661523957075992761662315262685030115255938352540032297113615687815976039390537716707854569980516690246592112936796917504034711418465442893323439490171095447109457355598873230115172636184525449905022174536414781771, 50576597458517451578431293746926099486388286246142012476814190030935689430726042810458344828563913001012415702876199708216875020997112089693759638454900092580746638631062117961876611545851157613835724635005253792316142379239047654392970415343694657580353333217547079551304961116837545648785312490665576832987, 96868738830341112368094632337476840272563704408573054404213766500407517251810212494515862176356916912627172280446141202661640191237336568731069327906100896178776245311689857997012187599140875912026589672629935267844696976980890380730867520071059572350667913710344648377601017758188404474812654737363275994871, 4881261656846638800623549662943393234361061827128610120046315649707078244180313661063004390750821317096754282796876479695558644108492317407662131441224257537276274962372021273583478509416358764706098471849536036184924640593888902859441388472856822541452041181244337124767666161645827145408781917658423571721, 18237936726367556664171427575475596460727369368246286138804284742124256700367133250078608537129877968287885457417957868580553371999414227484737603688992620953200143688061024092623556471053006464123205133894607923801371986027458274343737860395496260538663183193877539815179246700525865152165600985105257601565]


for i in ints:
    if pow(i, (p - 1) // 2, p) == 1:
        a = i
        print(pow(a, (p + 1) // 4, p))
        break 
```

## Modular Square Root

### Đề bài 
![8](/img/learning/wco4-math/8.png)

### Ý tưởng
- Như đề đã đề cập, bài này ta sẽ sử dụng `Tonelli-Shanks algorithm` để giải và bản thân mình nhận thấy rằng việc xài sage sẽ tốt hơn trong trường hợp này, tuy nhiên mình vẫn sẽ đề cập sơ qua thuật toán ở dưới đây:
- 1. Mình sẽ tính `phi` của `p` bằng cách $phi = p - 1$. Sau đó mình sẽ chuyển `phi` thành dạng $phi = QS^2$, với Q là một số lẻ.
- 2. Mình sẽ tìm một bất thặng dư bậc 2 của `p` bằng cách thử từng số nguyên tố i nhỏ, bởi vì những số bất thặng dư bậc 2 chiếm số lượng rất lớn, nên bạn có thể tạo một mảng chứa khoảng 50-100 số nếu sợ.
- 3. Mình sẽ khởi tạo biến như sau:
```math
\begin{align}
M &= S \\
C &\equiv n^Q \pmod{p} \\
t &\equiv a^q \pmod{p} \\
R &\equiv a^{(q+1)/2} \pmod{p}
\end{align}
```
- 4. Mình sẽ cho chạy vòng lặp tới khi $t=1$, và vòng lặp như sau:
```math
\begin{align}
t^{2^i}  &\equiv 1 \pmod{p} \\
b &\equiv C^{2^{M-i-1}} \pmod{p}
\end{align}
```

Với i là một số mà mình sẽ phải chạy vòng lặp để tìm nó.
Giờ ta sẽ cập nhật:

```math
\begin{align}
M &= i\\
C  &\equiv b^2 \pmod{p} \\
t &\equiv t \cdot b \pmod{p} \\
R &\equiv R \cdot b \pmod{p}
\end{align}
```
Đó là cách để tìm ra thặng dư bậc 2 với số $a \equiv 3 \ (mod \ 4)$
- Còn đối với trường hợp sử dụng sage thì nó đơn giản hơn rât nhiều bằng cách tạo một trường module p, và tinh sqrt(x) để tìm ra a.

### Code
```python=
from sage.all import *
a = 8479994658316772151941616510097127087554541274812435112009425778595495359700244470400642403747058566807127814165396640215844192327900454116257979487432016769329970767046735091249898678088061634796559556704959846424131820416048436501387617211770124292793308079214153179977624440438616958575058361193975686620046439877308339989295604537867493683872778843921771307305602776398786978353866231661453376056771972069776398999013769588936194859344941268223184197231368887060609212875507518936172060702209557124430477137421847130682601666968691651447236917018634902407704797328509461854842432015009878011354022108661461024768
p = 30531851861994333252675935111487950694414332763909083514133769861350960895076504687261369815735742549428789138300843082086550059082835141454526618160634109969195486322015775943030060449557090064811940139431735209185996454739163555910726493597222646855506445602953689527405362207926990442391705014604777038685880527537489845359101552442292804398472642356609304810680731556542002301547846635101455995732584071355903010856718680732337369128498655255277003643669031694516851390505923416710601212618443109844041514942401969629158975457079026906304328749039997262960301209158175920051890620947063936347307238412281568760161

R = Integers(p)
r1 = R(a).sqrt()
r2 = -r1
print(min(r1,r2)) 
```

## Chinese Remainder Theorem

### Đề bài
![9](/img/learning/wco4-math/9.png)

### Ý tưởng
- Bài này giới thiệu `Chinese Remainder Theorem` - định lý thặng dư trung hoa. Để giải thích một cách đơn giản nhất thì khi giải hệ phương trình ở toán bình thường, ta có rất nhiêu cách để làm khác nhau như thế..., còn để giải hệ phương trình đồng dư thì ta sẽ sử dụng cách này.
- Hồi đó mình từng học toán đồng dư ở [đây](https://youtube.com/playlist?list=PL3WwdyzXMhuYNSYhQX-r0jKW-tuj-GZoJ&si=ga01fdhw70BdcncO)(mặc dù tác giả đôi lúc có sai nhẹ =)))
- Tuy nhiên, hiện tại thì mình sẽ sử dụng sage để giải bài này cho nhanh.

### Code 
```python=
from sage.all import *
x = crt([2, 3, 5], [5, 11, 17])
print(x%935) 
```


## Adrien's Signs

### Đề bài
![10](/img/learning/wco4-math/10.png)


### Ý tưởng
- Khi đọc code thì mình đã thử test một tí và nhận ra là:
```math
(a/p) = 1
```
Mà ta lại có:
```math
(n/p) \equiv (a^e/p) \equiv (a/p)
```
Vì a là một thặng dư bậc 2 nên mọi $a^e$ cũng là thặng dư bậc 2 và n cũng vậy. Và vì thế nếu n là thặng dư bậc 2 thì mình append 1, nếu không phải thì append 0.

### Code 
```python=
from Crypto.Util.number import long_to_bytes
a = 288260533169915
p = 1007621497415251

ints = [67594220461269, 501237540280788, 718316769824518, 296304224247167, 48290626940198, 30829701196032, 521453693392074, 840985324383794, 770420008897119, 745131486581197, 729163531979577, 334563813238599, 289746215495432, 538664937794468, 894085795317163, 983410189487558, 863330928724430, 996272871140947, 352175210511707, 306237700811584, 631393408838583, 589243747914057, 538776819034934, 365364592128161, 454970171810424, 986711310037393, 657756453404881, 388329936724352, 90991447679370, 714742162831112, 62293519842555, 653941126489711, 448552658212336, 970169071154259, 339472870407614, 406225588145372, 205721593331090, 926225022409823, 904451547059845, 789074084078342, 886420071481685, 796827329208633, 433047156347276, 21271315846750, 719248860593631, 534059295222748, 879864647580512, 918055794962142, 635545050939893, 319549343320339, 93008646178282, 926080110625306, 385476640825005, 483740420173050, 866208659796189, 883359067574584, 913405110264883, 898864873510337, 208598541987988, 23412800024088, 911541450703474, 57446699305445, 513296484586451, 180356843554043, 756391301483653, 823695939808936, 452898981558365, 383286682802447, 381394258915860, 385482809649632, 357950424436020, 212891024562585, 906036654538589, 706766032862393, 500658491083279, 134746243085697, 240386541491998, 850341345692155, 826490944132718, 329513332018620, 41046816597282, 396581286424992, 488863267297267, 92023040998362, 529684488438507, 925328511390026, 524897846090435, 413156582909097, 840524616502482, 325719016994120, 402494835113608, 145033960690364, 43932113323388, 683561775499473, 434510534220939, 92584300328516, 763767269974656, 289837041593468, 11468527450938, 628247946152943, 8844724571683, 813851806959975, 72001988637120, 875394575395153, 70667866716476, 75304931994100, 226809172374264, 767059176444181, 45462007920789, 472607315695803, 325973946551448, 64200767729194, 534886246409921, 950408390792175, 492288777130394, 226746605380806, 944479111810431, 776057001143579, 658971626589122, 231918349590349, 699710172246548, 122457405264610, 643115611310737, 999072890586878, 203230862786955, 348112034218733, 240143417330886, 927148962961842, 661569511006072, 190334725550806, 763365444730995, 516228913786395, 846501182194443, 741210200995504, 511935604454925, 687689993302203, 631038090127480, 961606522916414, 138550017953034, 932105540686829, 215285284639233, 772628158955819, 496858298527292, 730971468815108, 896733219370353, 967083685727881, 607660822695530, 650953466617730, 133773994258132, 623283311953090, 436380836970128, 237114930094468, 115451711811481, 674593269112948, 140400921371770, 659335660634071, 536749311958781, 854645598266824, 303305169095255, 91430489108219, 573739385205188, 400604977158702, 728593782212529, 807432219147040, 893541884126828, 183964371201281, 422680633277230, 218817645778789, 313025293025224, 657253930848472, 747562211812373, 83456701182914, 470417289614736, 641146659305859, 468130225316006, 46960547227850, 875638267674897, 662661765336441, 186533085001285, 743250648436106, 451414956181714, 527954145201673, 922589993405001, 242119479617901, 865476357142231, 988987578447349, 430198555146088, 477890180119931, 844464003254807, 503374203275928, 775374254241792, 346653210679737, 789242808338116, 48503976498612, 604300186163323, 475930096252359, 860836853339514, 994513691290102, 591343659366796, 944852018048514, 82396968629164, 152776642436549, 916070996204621, 305574094667054, 981194179562189, 126174175810273, 55636640522694, 44670495393401, 74724541586529, 988608465654705, 870533906709633, 374564052429787, 486493568142979, 469485372072295, 221153171135022, 289713227465073, 952450431038075, 107298466441025, 938262809228861, 253919870663003, 835790485199226, 655456538877798, 595464842927075, 191621819564547]

binary =""
for i in ints:
    if(pow(i, (p - 1) // 2, p) == 1):
        binary += "1"
    else:
        binary += "0"
flag = b""
for i in range(0,len(binary),8):
    byte = binary[i:i+8]
    flag+=long_to_bytes(int(byte,2))
print(flag)
```

## Modular Binomials

### Đề bài
![11](/img/learning/wco4-math/11.png)


### Ý tưởng
Note: thành thật mà nói thì mình đã giải lại bài này lần thứ 4 hay 5 rồi =))) và lần đầu tiên giải thì mình có sử dụng llm. Nên mình sẽ không có flow tư duy để giải ra bài này mà chỉ đơn thuần là kinh nghiệm từ những lần giải trước.

Ta có:
```math
\begin{align}
c_1 &\equiv (2p + 3q)^{e_1} \pmod{N} \\
c_2 &\equiv (5p + 7q)^{e_2} \pmod{N}
\end{align}
```

Từ đó suy ra:

```math
\begin{align}
A = c_1^{e_2} &\equiv (2p + 3q)^{e_1 \cdot e_2} \pmod{N} \\
B = c_2^{e_1} &\equiv (5p + 7q)^{e_2 \cdot e_1} \pmod{N}
\end{align}
```
Và ta có thể thực hiện một chút biến đổi như sau:
```math
\begin{align}
A = c_1^{e_2} &\equiv (2p)^{e_1 \cdot e_2} \pmod{q} \\
B = c_2^{e_1} &\equiv (5p)^{e_2 \cdot e_1} \pmod{q}
\end{align}
```
Từ đó ta có thể lập ra biểu thức: 
```math
A \cdot 5^{e_2 \cdot e_1} - B \cdot 2^{e_1 \cdot e_2} \equiv 0 \pmod{q} \quad (*)
```
(p đã bị triệt tiêu)
Từ biểu thức trên ta có thể suy ra rằng GCD(*,N) = q. Và ta có thể dễ dàng tính được p và q.

[Resources](https://www.ctfrecipes.com/cryptography/general-knowledge/maths/modular-arithmetic/modular-binomial)

Note: chứng minh vì sao * không bao giờ chia hết cho p. Vì * có dạng k*q => để nó chia hết cho p thì k phải có dạng p * m. Với m = 1 thì * = N (vô lý). Với m > 1 thì lại càng vô lý vì * đang được xử lý trong trường hữu hạn q.

### Code
```python=
from math import gcd
n = 14905562257842714057932724129575002825405393502650869767115942606408600343380327866258982402447992564988466588305174271674657844352454543958847568190372446723549627752274442789184236490768272313187410077124234699854724907039770193680822495470532218905083459730998003622926152590597710213127952141056029516116785229504645179830037937222022291571738973603920664929150436463632305664687903244972880062028301085749434688159905768052041207513149370212313943117665914802379158613359049957688563885391972151218676545972118494969247440489763431359679770422939441710783575668679693678435669541781490217731619224470152467768073
e1 = 12886657667389660800780796462970504910193928992888518978200029826975978624718627799215564700096007849924866627154987365059524315097631111242449314835868137
e2 = 12110586673991788415780355139635579057920926864887110308343229256046868242179445444897790171351302575188607117081580121488253540215781625598048021161675697
c1 = 14010729418703228234352465883041270611113735889838753433295478495763409056136734155612156934673988344882629541204985909650433819205298939877837314145082403528055884752079219150739849992921393509593620449489882380176216648401057401569934043087087362272303101549800941212057354903559653373299153430753882035233354304783275982332995766778499425529570008008029401325668301144188970480975565215953953985078281395545902102245755862663621187438677596628109967066418993851632543137353041712721919291521767262678140115188735994447949166616101182806820741928292882642234238450207472914232596747755261325098225968268926580993051
c2 = 14386997138637978860748278986945098648507142864584111124202580365103793165811666987664851210230009375267398957979494066880296418013345006977654742303441030008490816239306394492168516278328851513359596253775965916326353050138738183351643338294802012193721879700283088378587949921991198231956871429805847767716137817313612304833733918657887480468724409753522369325138502059408241232155633806496752350562284794715321835226991147547651155287812485862794935695241612676255374480132722940682140395725089329445356434489384831036205387293760789976615210310436732813848937666608611803196199865435145094486231635966885932646519

q1 = pow(c1, e2, n)
q2 = pow(c2, e1, n)

d = pow(5, e1 * e2, n) * q1 - pow(2, e1 * e2, n) * q2

q = gcd(d, n)
p = n // q

print(p) 
print(q) 
```

# General

## ASCII

### Đề bài 
![12](/img/learning/wco4-math/12.png)

### Ý tưởng
- Như đề đề cập.

### Code
```python=
a = [99, 114, 121, 112, 116, 111, 123, 65, 83, 67, 73, 73, 95, 112, 114, 49, 110, 116, 52, 98, 108, 51, 125]
print("".join(chr(i) for i in a))
```

## Hex 

### Đề bài
![13](/img/learning/wco4-math/13.png)

### Ý tưởng
- Như đề đề cập.
### Code
```python=
a = "63727970746f7b596f755f77696c6c5f62655f776f726b696e675f776974685f6865785f737472696e67735f615f6c6f747d"
b = bytes.fromhex(a)
print(b.decode())
```

## Base64 

### Đề bài
![14](/img/learning/wco4-math/14.png)

### Ý tưởng
- Như đề đề cập.

### Code
```python=
from base64 import b64encode
a = "72bca9b68fc16ac7beeb8f849dca1d8a783e8acf9679bf9269f7bf"
b = bytes.fromhex(a)
c = b64encode(b)
print(c.decode())
```

## Bytes and Big Integers

### Đề bài
![15](/img/learning/wco4-math/15.png)

### Ý tưởng
- Như đề.

### Code 
```python=
from Crypto.Util.number import long_to_bytes
a = 11515195063862318899931685488813747395775516287289682636499965282714637259206269
print(long_to_bytes(a))
```

## Encoding Challenge

### Đề bài
![16](/img/learning/wco4-math/16.png)

### Ý tưởng
- Như đề.

### Code
```python=
from pwn import * # pip install pwntools
import json
import base64
import codecs

r = remote('socket.cryptohack.org', 13377, level = 'debug')

def json_recv():
    line = r.recvline()
    return json.loads(line.decode())

def json_send(hsh):
    request = json.dumps(hsh).encode()
    r.sendline(request)

received = json_recv()

while True:

    if "flag" in received:
        break

    received_str = received["type"]
    received_encoded = received["encoded"]
    decoded_str = ""

    if received_str == "base64":
        decoded_str = base64.b64decode(received_encoded).decode()
    elif received_str == "hex":
        decoded_str = bytes.fromhex(received_encoded).decode()
    elif received_str == "rot13":
        decoded_str = codecs.decode(received_encoded, 'rot_13')
    elif received_str == "bigint":
        n = int(received_encoded, 16)
        decoded_str = n.to_bytes((n.bit_length() + 7) // 8, 'big').decode()
    elif received_str == "utf-8":
        decoded_str = bytes(received_encoded).decode()
    
    to_send = {
        "decoded": decoded_str
    }
    
    json_send(to_send)

    received = json_recv() 
```

## XOR Starter

### Đề bài
![17](/img/learning/wco4-math/17.png)

### Ý tưởng
- Bài này giới thiệu cho ta hiểu về phép tính xor.
- Nhiệm vụ của ta là phải xor(`label`,13). Về bản chất, ta phải chuyển 2 thằng về dạng binary sau đó xor từng bit với nhau. 

### Code
```python=
from pwn import *  

print(xor(b'label',13))
```

## XOR Properties

### Đề bài 
![18](/img/learning/wco4-math/18.png)

### Ý tưởng
- Bài này cho ta biết tính chất của xor rằng:
```math
A \oplus A = 0
```

### Code 
```python=
from pwn import *

KEY1_hex = 'a6c8b6733c9b22de7bc0253266a3867df55acde8635e19c73313'
KEY2_KEY1_hex = '37dcb292030faa90d07eec17e3b1c6d8daf94c35d4c9191a5e1e'
KEY2_KEY3_hex = 'c1545756687e7573db23aa1c3452a098b71a7fbf0fddddde5fc1'
FLAG_KEY1_KEY3_KEY2_hex = '04ee9855208a2cd59091d04767ae47963170d1660df7f56f5faf'


KEY1 = bytes.fromhex(KEY1_hex)
KEY2_KEY1 = bytes.fromhex(KEY2_KEY1_hex)
KEY2_KEY3 = bytes.fromhex(KEY2_KEY3_hex)
FLAG_KEY1_KEY3_KEY2 = bytes.fromhex(FLAG_KEY1_KEY3_KEY2_hex)

a = xor(KEY1, KEY2_KEY3)  
b = xor(a,FLAG_KEY1_KEY3_KEY2)
print(b)
```

## Favourite byte

### Đề bài
![19](/img/learning/wco4-math/19.png)

### Ý tưởng
- Như đề.

### Code 
```python=
from pwn import *

a_hex = '73626960647f6b206821204f21254f7d694f7624662065622127234f726927756d'
a = bytes.fromhex(a_hex)

for i in range(256):
    b = xor(a,i)
    if b'crypto' in b:
        print(b)
        break
```
## You either know, XOR you don't

### Đề bài
![20](/img/learning/wco4-math/20.png)

### Ý tưởng
- Như đề.

### Code
```python=
from pwn import *

a_hex = '0e0b213f26041e480b26217f27342e175d0e070a3c5b103e2526217f27342e175d0e077e263451150104'
a = bytes.fromhex(a_hex)

b = xor(a,b'crypto{') 
print(b)

key = b'myXORkey'
print(xor(a, key))
```

## Lemur XOR

### Đề bài
![21](/img/learning/wco4-math/21.png)

### Ý tưởng
- Bài này đề cập tới xor rgb, minh truy cập vào [đây](https://stackoverflow.com/questions/8504882/searching-for-a-way-to-do-bitwise-xor-on-images) để giải.

## Privacy-Enhanced Mail?

### Đề bài 
![image](https://hackmd.io/_uploads/rk43V8dMZl.png)

### Ý tưởng
- Bài này giới thiệu cho ta pem, là một dạng văn bản được mã hóa bằng base64, và nó chứa rất nhiều dữ liệu...

### Code
```python=
from Crypto.PublicKey import RSA
with open("privacy_enhanced_mail.pem", "rb") as f:
    private_key = f.read()
mykey = RSA.import_key(private_key)
print(mykey.d)
```

## CERTainly not

### Đề bài
![image](https://hackmd.io/_uploads/SJ2EnI_GZg.png)

### Ý tưởng
- Bài này giới thiệu về der, mình không đọc được file này, tuy nhiên ta có thể dùng lệnh sau để chuyên từ der sang pem:
`openssl x509 -inform der -in 2048b-rsa-example-cert.der -out certificate.pem`
- Tuy nhiên dù không đọc được thì ta vẫn có thể trích xuất n ra trong python, code như bài trên.

### Code
```python=
from Crypto.PublicKey import RSA
with open("2048b-rsa-example-cert.der", "rb") as f:
    public_key = f.read()
mykey = RSA.import_key(public_key)
print(mykey.n)
```

## SSH Keys 

### Đề bài
![image](https://hackmd.io/_uploads/Bykwbv_f-l.png)


### Ý tưởng
- Để giải thích đơn giản thì giao thức ssh sẽ sử dụng một bộ khóa bí mật và công khai để xác thực người dùng và từ đó có thể truy cập vào trang web. Khóa bí mật được lưu tại máy của chính người đó, còn bản sao của khóa công khai chính là file pub trong đề, nó có chút khác so với pem thông thường. Nhiệm vụ của ta là tìm n.

### Code
```python=
from Crypto.PublicKey import RSA
with open("bruce_rsa.pub", "rb") as f:
    public_key = f.read()
mykey = RSA.import_key(public_key)
print(mykey.n)
```

## Transparency

### Đề bài 
![image](https://hackmd.io/_uploads/Sy1ziwdfbl.png)

### Ý tưởng
- Bài này yêu cầu ta tìm subdomain của cryptohack.org, và vì thế mình đã chọn xài trick lỏ bằng tool [này](https://pentest-tools.com/information-gathering/find-subdomains-of-domain) để giải.
- Nhưng sau đó mình đã biết intended solution, bước đầu là trích xuất n, sau dó tim SHA256 fingerprint bằng cách chuyển nó thành file der và hash nó. Sau đó dùng tool crt.sh để search.

# Lo-Hi Card Game

## Đề bài
![image](https://hackmd.io/_uploads/r1Qlumvf-l.png)

## Ý tưởng
- Trước hết nói về logic và cách chiến thắng trò chơi: nó sẽ phát cho ta một lá bài và yêu cầu ta so sánh với lá bài kế tiếp, và ở ván đấu sau lá bài kế tiếp sẽ là của ta và tiếp tục vòng lặp. Khi thắng thì ta dược 1d, thua thì trừ 2d. Sau 200 vòng ta được >=130d thì thắng.
- Cứ sau khoảng 10/11 lá bài thì nó sẽ xáo bài 1 lần. Ta gọi 10/11 lá bài đó là 1 `state`. Nhìn vào đoạn code sau (chỉ cắt phần cốt lõi):
```python=
def rebase(self, n, b=52):
    if n < b:
        return [n]
    else:
        return [n % b] + self.rebase(n//b, b)
```
Ta có thể thấy rằng `n` ở đây tương ứng với giá trị `state` và nó đang xử lý state thành một list số nhỏ trong trường hữu hạn 52. Và nó sẽ được bỏ vào 1 stack - vào sau thì ra trước. Và đồng thời ta nhìn vào đoạn code này:
```python=
def deal_card(self):
    index = self.deals.pop()
    if self.deals == []:
        self.num_deals = self.shuffle()

    return self.deck[index]
```
Những số trong stack đó ứng với index ở đây, và index là số đại diện cho các quân bài 
```python=         
self.deck = [Card(value, suit) for suit in SUITS for value in VALUES]
```
Vì thế ta biết đuợc rằng khi ta biết được state thì ta hoàn toàn có thể dự đoán được toàn bộ bài của state đó. 
- State được tính bằng công thức sau:
```python=
def __init__(self, seed):
    self.state = seed

def next(self):
    self.state = (self.state * self.mul + self.inc) % self.mod
    return self.state
```
Giải thích đơn giản thì yêu cầu cốt lõi là ta phải tinh được mul với inc, và phương trình này hiện tại đang có 2 ẩn, mul, inc. Và hệ thống cũng đã cung cấp cho ta đủ một lượng điểm để ta có thể đoán bậy trong 30/33 lượt đầu và giải hệ phương trình trong trường hữu hạn mod. Khi ta tính đươc mul và inc thì end bài.
## Code
```python=
from pwn import *
import json
VALUES = ['Ace', 'Two', 'Three', 'Four', 'Five', 'Six',
          'Seven', 'Eight', 'Nine', 'Ten', 'Jack', 'Queen', 'King']
SUITS = ['Clubs', 'Hearts', 'Diamonds', 'Spades']

deck = list(enumerate((v, s) for s in SUITS for v in VALUES))
card_lookup = {card: i for i, card in deck}

def cal_next_state(state, mul, inc):
    return (state * mul + inc) % (2**61 - 1)

def cal_state(collected):
    reversed_collected = collected[::-1]
    state = 0
    for i, index_val in enumerate(reversed_collected):
        state += index_val * (52 ** i)
    return state

def rebase(n, b=52):
    if n < b:
        return [n]
    else:
        return [n % b] + rebase(n//b, b)

r = remote('socket.cryptohack.org', 13383, level='debug')


state_list = []
collected = []

for i in range(33):
    data = json.loads(r.recvline())
    my_hand = data["hand"]
    val, suit = my_hand.split(" of ")
    index = SUITS.index(suit) * 13 + VALUES.index(val)
    collected.append(index)
    if (len(collected) == 11):
        state = cal_state(collected)
        state_list.append(state)
        collected = []
    r.sendline('{"choice": "l"}')

mul = (state_list[2] - state_list[1]) * pow(state_list[1] - state_list[0], -1, 2**61 - 1) % (2**61 - 1)
inc = (state_list[2] - state_list[1] * mul) % (2**61 - 1) 


collect = []
state = cal_next_state(state_list[-1], mul, inc)
collect = rebase(state)

while True:
    data = json.loads(r.recvline())

    if len(collect) != 1:
        if collect[-1]%13 < collect[-2]%13:
            r.sendline('{"choice": "h"}')
        else:
            r.sendline('{"choice": "l"}')
        del collect[-1]

    if len(collect) == 1:
        state = cal_next_state(state, mul, inc)
        collect += rebase(state)
        if collect[0]%13 < collect[-1]%13:
            r.sendline('{"choice": "h"}')
        else:
            r.sendline('{"choice": "l"}')
        del collect[0]
```
