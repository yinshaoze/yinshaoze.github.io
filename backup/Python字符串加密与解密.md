在生产中会遇到很多情况是需要加密字符串的（如访问或存储密码）这些需求造就了需要字符串加密，以及反解密的问题，推荐两种方法来实现，下附代码：

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-
def encrypt(key, s):
    b = bytearray(str(s).encode("utf-8"))
    n = len(b)
    c = bytearray(n*2)
    j = 0
    for i in range(0, n):
        b1 = b[i]
        b2 = b1 ^ key
        c1 = b2 % 19
        c2 = b2 // 19
        c1 = c1 + 46
        c2 = c2 + 46
        c[j] = c1
        c[j+1] = c2
        j = j+2
    return c.decode("utf-8")


def decrypt(ksa, s):
    c = bytearray(str(s).encode("utf-8"))
    n = len(c)
    if n % 2 != 0:
        return ""
    n = n // 2
    b = bytearray(n)
    j = 0
    for i in range(0, n):
        c1 = c[j]
        c2 = c[j + 1]
        j = j + 2
        c1 = c1 - 46
        c2 = c2 - 46
        b2 = c2 * 19 + c1
        b1 = b2 ^ ksa
        b[i] = b1
    return b.decode("utf-8")

print(encrypt(11, 'password123'))
print(decrypt(11, '43<303>3/1.1@041'))
```
第二种：
```python
from Crypto.Cipher import AES
from binascii import b2a_hex, a2b_hex


class EncryptStr(object):
    def __init__(self, key):
        self.key = key
        self.mode = AES.MODE_CBC

    def encrypt(self, text):
        cryptor = AES.new(self.key, self.mode, self.key)
        length = 16
        count = len(text)
        if (count % length != 0):
            add = length - (count % length)
        else:
            add = 0
        text = text + ('\0' * add)
        self.ciphertext = cryptor.encrypt(text)
        return b2a_hex(self.ciphertext)

    # 解密后，去掉补足的空格用strip() 去掉
    def decrypt(self, text):
        cryptor = AES.new(self.key, self.mode, self.key)
        plain_text = cryptor.decrypt(a2b_hex(text))
        return plain_text.decode('utf-8').strip('\0')

if __name__ == '__main__':
    pc = EncryptStr('keyskeyskeyskeys')  # 初始化密钥
    e = pc.encrypt("passwd123")
    d = pc.decrypt("0e0dbd0509f9eaaafd420b8a2c72cbde")
    print(e, d)
```