## Vigenere++

### 题目描述

这一次，我们采用了先进的仿射版维吉尼亚，抛弃了以往凯撒密码过于简单的弊端，也修复了可以直接使用现成工具的bug。所有，很高兴的告诉大家，可以手搓脚本啦。

### 解题思路

题目已经说了，就是维吉尼亚的仿射版本，所以说，其实做法基本和维吉尼亚相同，都是通过频率攻击去破解的。

首先，按照维吉尼亚的思路，是将常规的文本按照密钥长度拆分成组，暴力破解的思路则是尝试每一个可能的密钥长度结果，通过和计算得到的概率进行比较，相近的保留，从而筛选合格的部分。
那么这题也是如此，我们知道仿射密码虽然比常规的凯撒位移复杂，重复度低，但是其最大的密钥重复长度也就是a*b，题目中限制了两者乘机不超过100，所以有爆破的余地。

接着，按照对应的长度，常规的维吉尼亚进行具体偏移的尝试，从而得出具体密钥。
于此相近，改版通过具体的i，j的尝试，得出每一块最可能的密钥对，然后综合起来，从而得出密钥空间。

然后，我们去拿得到的密钥空间进行对明文的恢复，如果并非所有字符都是正常的，可读的英文文本内容，则将其进行一定的微调，从而最后得到正常的明文。

最后，我们拿着具体的文本，又知道flag在明文中，和flag的长度，考虑通过hash碰撞得出具体的flag（这一块其实是最先考虑的，才会采用上面的方式恢复明文）。

整体解题脚本如下——

脚本1：

```python
from Crypto.Util.number import *
from tqdm import trange

f = open(r'cipher.md','r')
c = f.read()
f.close()
# 我们已经知道，维吉尼亚密码可以被分解为若干组平移密码来破译，
# 而一个明文足够长的平移密码的重合指数接近0.0687。
# 换言之，如果我们选取某个l值，使得分组后的密文的重合指数接近0.065，
# 则说明选取的t值与密钥的长度是一致的。
best_index = 0.065
sum = 0
dic_index = {'a': 0.08167,'b': 0.01492,'c': 0.02782,'d':0.04253,'e': 0.12702,'f':0.02228,'g': 0.02015,'h':0.06094,'i':0.06966,'j':0.00153,'k':0.00772,'l':0.04025,'m':0.02406,'n':0.06749,'o':0.07507,'p':0.01929,'q':0.00095,'r':0.05987,'s':0.06327,'t':0.09056,'u':0.02758,'v':0.00978,'w':0.02360,'x':0.00150,'y':0.01974,'z':0.00074}

def IndCo(s):
    # 计算字符串的重合指数（所有字母出现频率的平方和）
    # 输入 s 
    # 输出 重合指数
    alpha = 'abcdefghijklmnopqrstuvwxyz'
    freq:dict[str,int] = {}
    for i in alpha:
        freq[i] = 0
    for i in s:
        freq[i] =  freq[i] + 1
    index = 0
    for i in alpha:
        index = index + (freq[i]*(freq[i] - 1 )) / (len(s) * (len(s) - 1 ))
    return index

def IndCo_m(s):
    # 计算明文s中的各字母频率和英文字母中的频率吻合程度
    # 输入：明文s
    # 输出：吻合程度
    alpha = 'abcdefghijklmnopqrstuvwxyz'
    freq = {}
    for i in alpha:
        freq[i] = 0
    for i in s:
        freq[i] += 1
    index = 0
    for i in alpha:
        index += freq[i] / len(s) * dic_index[i]
    return index

def get_keylen(c):
    # 求出最符合统计学的m,n的最小公倍数，方法通过爆破足够大的周期样本，观察成倍出现的周期
    # 计算方法是解出每一个子密文段的重合指数然后求平均值 在和最佳重合指数相减 误差小于0.01
    # 输入：密文
    # 输出：公共周期列表
    keylen = []
    for i in range(1,100):
        average_index = 0
        for j in range(i):
            s = ''.join(c[j+i*x] for x in range(0,len(c)//i))
            index = IndCo(s)
            average_index += index
        average_index = average_index / i - best_index
        if abs(average_index) < 0.01:
            keylen.append(i)
    return keylen

keylen = get_keylen(c)
print(f'{keylen=}')

#____________________________得到keylen_____________________________#
#____________________________爆破flag———————————————————————————————#

def decrypt(c,i,j):
    alpha = 'abcdefghijklmnopqrstuvwxyz'
    m = ''
    for x in c:
        m += alpha[((alpha.index(x)-j)*inverse(i,26))%26]
    return m 

def get_key(c):
    # 得到一个密文段的单个字符key .i .j
    # 暴力枚举，找到最符合的
    # 输入：密文段
    # 输出：i,j
    for i in range(26):
        if GCD(i,26)!= 1 :
            continue
        for j in range(26):
            m = decrypt(c,i,j)
            index = IndCo_m(m)
            if abs(index-0.065)<0.01:
                return (i,j)

def get_all_key(s,keylen):
    # 得到一个周期内所有密文段的key
    # 输入：原密文，周期
    # 输出：无
    keypairs = []
    for i in range(keylen):
        temps = ''.join([s[i+x*keylen] for x in range(0,len(s)//keylen)])
        keypair = get_key(temps)
        if keypair == None:
            return None
        keypairs.append(keypair)
    return keypairs


k1=[]
k2=[]
for i in trange(len(keylen)):
    keypairs = get_all_key(c,keylen[i])
    if keypairs is not None:
        k1,k2 =zip(*keypairs)
        print(f'{k1=}')
        print(f'{k2=}')

#____________________________MD5爆破_____________________________#

from hashlib import *

alpha='abcdefghijklmnopqrstuvwxyz'

while True:
    plaintext = ''
    k1 = [int(i) for i in input('k1 (sep with comma)>').split(",")]
    k2 = [int(i) for i in input('k2 (sep with comma)>').split(",")]
    
    if (input("are you sure?(Y/else)") not in ("Y","y")):
        continue

    l1 = len(k1)
    l2 = len(k2)
    if not (l1*l2//GCD(l1,l2) in keylen):
        print("faulty l1 and l2!!!")
        continue
    for i in range(len(c)):
        plaintext+=alpha[((alpha.index(c[i])-k2[i%l2])*inverse(k1[i%l1],26))%26]
    with open('generated_text.md','w') as f:
        f.write(plaintext)
        f.close()
    if (input("are you sure again?(Y/else)") in ("Y","y")):
        break

Pad = b'Cryptography_is_so_interesting'
MD = 'c58e96cc46568136c322888ee0f0b832'
for j in trange(30):
    for i in range(len(plaintext)-j):
        key = plaintext[i:i+j]
        if md5(key.encode()+Pad).hexdigest() == MD:
            print(key)
            print('r00t2025{'+key+'}')
            break
```

脚本2：

```python
from hashlib import *
from Crypto.Util.number import *
from tqdm import trange

alpha='abcdefghijklmnopqrstuvwxyz'
plaintext = ''

k1 = [25, 1, 7, 23, 11, 9, 15]
k2 = [12, 3, 25, 6, 13, 3, 8, 21, 9, 19, 8]

l1 = len(k1)
l2 = len(k2)
with open(r'cipher.md','r') as f:
    c = f.read()
for i in range(len(c)):
    plaintext+=alpha[((alpha.index(c[i])-k2[i%l2])*inverse(k1[i%l1],26))%26]
with open('generated_text.md','w') as f:
    f.write(plaintext)
    f.close()
input()
Pad = b'Cryptography_is_so_interesting'
MD = 'c58e96cc46568136c322888ee0f0b832'
for j in trange(30):
    for i in range(len(plaintext)-j):
        key = plaintext[i:i+j]
        if md5(key.encode()+Pad).hexdigest() == MD:
            print(key)
            print('r00t2025{'+key+'}')
            break
```



### 出题思路

我一直以来最喜欢的题目（也的确不是我原创的）。感觉堪称古典密码学在ctf实践中最后的非misc题目。

不仅要求对维吉尼亚的完全掌握和超越，还要进行hash碰撞，可谓是很有难度了。

不过这题出题难度是真的高，想要找到合适的文本以及正确的，符合预期的参数需要进行严谨的调试，真的不是这么一个随机数就可以解决的事情。。。