# HW1 Description
請使用加密相關的library，實作加密檔案的程式(檔案需至少100MB)。需實作底下功能：

1. 使用AES-CBC mode加密

2. 使用AES-CTR mode (counter mode)加密

3. 使用ChaCha20加密

## 被加密大小
使用`100mb_file.py` 產生 `testfile.bin`，為**100MB**

## Key & IV 產生方式
Key為自己自訂義
IV使用`os.urandom( )`在每次執行產生不同的隨機值

## Execution description:

需在有**python & pip**的環境下執行，可利用以下指令檢查

python:
```console
python --version
```
>正常(最新版)應該會顯示 Python 3.10.X

pip:
```console
pip --version
```
>正常(最新版)應該會顯示 pip 23.X.X from ~ (python 3.10)

接著需安裝pycryptodome函式庫
```console
pip install pycryptodome
```
完成後即可開始執行以下程式了:

# 使用AES-CBC mode加密
規格為AES-128

檔案為`HW1_aes_CBC.py`，可直接執行
#### 程式碼如下
```python
from Crypto.Cipher import AES
from Crypto.Util.Padding import pad, unpad
import os
import time

# 加密函數，chunsize寫1024的話後面就沒有padding，然後unpad的時候就會發現後面沒有padding所以出錯?????
# if encrypt chunksize=1024 will get "ValueError: PKCS#7 padding is incorrect." when decrypt
# if chunksize is 1024 in decrypt, then encrypt chunksize needs to be 1008~1023

def encrypt(key, iv, in_filename, out_filename=None, chunksize=64*1024-1):
    
    cipher = AES.new(key, AES.MODE_CBC, iv=iv)

    if not out_filename:
            out_filename = in_filename + 'CBC.enc'

    with open(in_filename, 'rb') as infile:
        with open(out_filename, 'wb') as outfile:
            while True:
                # data = infile.read()
                
                data = infile.read(chunksize)
                if len(data) == 0:
                    break
                padded_data = pad(data, AES.block_size, style='pkcs7')
                ciphertext = cipher.encrypt(padded_data)
                outfile.write(ciphertext)

    return out_filename

def decrypt(key, iv, in_filename, out_filename=None, chunksize=64*1024):
    if not out_filename:
            out_filename = in_filename + '.dec'
    cipher = AES.new(key, AES.MODE_CBC, iv=iv)
    with open(in_filename, 'rb') as infile:
        with open(out_filename, 'wb') as outfile:
            while True:
                data = infile.read(chunksize)
                
                if len(data) == 0:
                    break
                decrypted_data = cipher.decrypt(data)
                # PKCS7
                unpadded_data = unpad(decrypted_data, AES.block_size, style='pkcs7')
                outfile.write(unpadded_data)
    return out_filename


key = b'ThisIsASecretKey'
iv = os.urandom(16)

in_filename = 'testfile.bin'
size = os.path.getsize(in_filename)
print('%s = %d bytes' % (in_filename, size))

# encrypt
start_time = time.time()
encrypted_filename = encrypt(key, iv, in_filename)
end_time = time.time()
print("Time taken to encrypt: ", end_time - start_time, " seconds.")

# decrypt
start_time = time.time()
decrypted_filename = decrypt(key, iv, encrypted_filename)
end_time = time.time()
print("Time taken to decrypt: ", end_time - start_time, " seconds.")

bytes_per_second = size / (end_time - start_time)
print("Bytes per second: ", bytes_per_second)

with open(in_filename, 'rb') as f1, open(decrypted_filename, 'rb') as f2:
    if f1.read() == f2.read():
        print('Correct！')
    else:
        print('Different！')
```
### AES-CBC執行結果:


![image](https://user-images.githubusercontent.com/60705979/229025389-412dcc1b-ea11-4da3-8b50-7a16dbfa47c0.png)


可以看到加密時間為 : **0.14876341819763184  seconds.**

而解密時間為 : **0.20273160934448242 seconds.**

加密速度為 : **704861459.0227884 Byte/s**

最後驗證解密後與原始文件兩者內容相同

# 使用AES-CTR (counter mode)加密
規格為AES-128

檔案為`HW1_aes_CTR.py`，可直接執行
#### 程式碼如下
```python
from Crypto.Cipher import AES
from Crypto.Util import Counter
import os
import time

def encrypt(key, in_filename, out_filename=None, chunksize=64 * 1024):
    iv = os.urandom(16)
    ctr = Counter.new(128, initial_value=int.from_bytes(iv, byteorder='big'))
    cipher = AES.new(key, AES.MODE_CTR, counter=ctr)
    
    if not out_filename:
        out_filename = in_filename + 'CTR.enc'

    with open(in_filename, 'rb') as infile:
        with open(out_filename, 'wb') as outfile:
            while True:
                chunk = infile.read(chunksize)
                if len(chunk) == 0:
                    break
                ciphertext = cipher.encrypt(chunk)
                outfile.write(ciphertext)
    return (iv,out_filename)

def decrypt(key, iv, in_filename, out_filename=None, chunksize=64 * 1024):    
    ctr = Counter.new(128, initial_value=int.from_bytes(iv, byteorder='big'))
    cipher = AES.new(key, AES.MODE_CTR, counter=ctr)
    
    if not out_filename:
        out_filename = in_filename + '.dec'

    with open(in_filename, 'rb') as infile:
        with open(out_filename, 'wb') as outfile:
            while True:
                chunk = infile.read(chunksize)
                if len(chunk) == 0:
                    break
                plaintext = cipher.decrypt(chunk)
                outfile.write(plaintext)
    return out_filename

key = b'ThisIsASecretKey'

in_filename = 'testfile.bin'
size = os.path.getsize(in_filename)
print('%s = %d bytes' % (in_filename, size))

# encrypt
start_time1 = time.time()
iv,encrypted_filename = encrypt(key, in_filename)
end_time1 = time.time()
print("Time taken to encrypt: ", end_time1 - start_time1, " seconds.")

# decrypt
start_time = time.time()
decrypted_filename = decrypt(key, iv, encrypted_filename)
end_time = time.time()
print("Time taken to decrypt: ", end_time - start_time, " seconds.")

bytes_per_second = size / (end_time1 - start_time1)
print("Bytes per second: ", bytes_per_second)

with open(in_filename, 'rb') as f1, open(decrypted_filename, 'rb') as f2:
    if f1.read() == f2.read():
        print('Correct！')
    else:
        print('Different！')
```
### AES-CTR執行結果:
![image](https://user-images.githubusercontent.com/60705979/229020381-2eda1ae4-6f21-4674-94e3-b4fe0c6fc264.png)

可以看到加密時間為 : **0.06496357917785645  seconds.**

而解密時間為 : **0.065185546875  seconds.**

加密速度為 : **1614098258.239778 Byte/s**

最後驗證解密後與原始文件兩者內容相同

# 使用ChaCha20加密
檔案為`HW1_ChaCha20.py`，可直接執行
#### 程式碼如下
```python
from Crypto.Cipher import ChaCha20
import time
import os

def encrypt_ChaCha20(in_filename, key, nonce,out_filename=None,chunksize = 64 * 1024):
    
    chacha20 = ChaCha20.new(key=key, nonce=nonce)
    if not out_filename:
        out_filename = in_filename + 'ChaCha.enc'
        
    with open(in_filename, 'rb') as infile:
        with open(out_filename, 'wb') as outfile:
            while True:
                chunk = infile.read(chunksize)
                if len(chunk) == 0:
                    break
                ciphertext = chacha20.encrypt(chunk)
                outfile.write(ciphertext)
    return out_filename

def decrypt_ChaCha20(in_filename, key, nonce,out_filename=None,chunksize = 64 * 1024):
    chacha20 = ChaCha20.new(key=key, nonce=nonce)
    
    if not out_filename:
        out_filename = in_filename + '.dec'
        
    with open(in_filename, 'rb') as infile:
        with open(out_filename, 'wb') as outfile:
            while True:
                chunk = infile.read(chunksize)
                if len(chunk) == 0:
                    break
                plaintext = chacha20.decrypt(chunk)
                outfile.write(plaintext)
    return out_filename


key = b'ThisIsASecretKeyThisIsASecretKey' # 32 bytes長度的密鑰
nonce = os.urandom(12)

in_filename = 'testfile.bin'
size = os.path.getsize(in_filename)
print('%s = %d bytes' % (in_filename, size))

# encrypt
start_time1 = time.time()
encrypted_filename = encrypt_ChaCha20(in_filename, key, nonce)
end_time1 = time.time()
print("Time taken to encrypt: ", end_time1 - start_time1, " seconds.")

# decrypt
start_time = time.time()
decrypted_filename = decrypt_ChaCha20(encrypted_filename, key, nonce)
end_time = time.time()
print("Time taken to decrypt: ", end_time - start_time, " seconds.")

bytes_per_second = size / (end_time1 - start_time1)
print("Bytes per second: ", bytes_per_second)

with open(in_filename, 'rb') as f1, open(decrypted_filename, 'rb') as f2:
    if f1.read() == f2.read():
        print('Correct！')
    else:
        print('Different！')
```
### ChaCha20執行結果:
![image](https://user-images.githubusercontent.com/60705979/229020718-8f236539-e358-43dd-8b11-9da68c4d9736.png)

可以看到加密時間為 : **0.2567024230957031  seconds.**

而解密時間為 : **0.2102365493774414  seconds.**

加密速度為 : **408479198.347525 Byte/s**

最後驗證解密後與原始文件兩者內容相同

# 結論

從上面三種結果匯成表格如下
||Ecrypt time(s)|Decrypt time(s) |Speed(Byte/s)|
|-----|--------|-----|--------|
|AES-CBC|0.14876341819763184      |0.20273160934448242|704,861,459|
|AES-CTR|**0.06496357917785645** |**0.065185546875**|**1,614,098,258**|
|ChaCha20|0.2567024230957031|0.2102365493774414|408,479,198|

可以看到AES-CTR為全部加解密速度最快的，而AES-CBC次之，ChaCha20為最後
