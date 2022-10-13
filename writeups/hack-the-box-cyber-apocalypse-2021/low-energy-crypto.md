# Low Energy Crypto

{% file src="../../.gitbook/assets/forensics_low_energy_crypto.zip" %}

<figure><img src="../../.gitbook/assets/116129688-965f6f80-a6f4-11eb-8794-622ad1df2033 (1).png" alt=""><figcaption></figcaption></figure>

* Tiếp tục với file `PCAPNG`. Lần đầu tiên gặp Protocol LE LL nên mình đã tìm tất cả các thông tin quan trọng.
* Đầu tiên mình phát hiện được 2 phần Key Public ở Packet 215 và 223

<figure><img src="../../.gitbook/assets/116129688-965f6f80-a6f4-11eb-8794-622ad1df2033.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/116130774-cfe4aa80-a6f5-11eb-93a1-d396f463906b.png" alt=""><figcaption></figcaption></figure>

```
-----BEGIN PUBLIC KEY-----
MGowDQYJKoZIhvcNAQEBBQADWQAwVgJBAKKPHxnmkWVC4fje7KMbWZf07zR10D0m
B9fjj4tlGekPOW+f8JGzgYJRWboekcnZfiQrLRhA3REn1lUKkRAnUqAkCEQDL/3Li
4l+RI2g0FqJvf3ff
-----END PUBLIC KEY-----
```

* Tiếp tục ở Packet 230 mình tìm thấy được đoạn strings

<figure><img src="../../.gitbook/assets/116130934-fd315880-a6f5-11eb-98d3-b2c0b21c2002.png" alt=""><figcaption></figcaption></figure>

```
┌──(kali㉿kali)-[~/Desktop/CTF/HTB/LowEnergyCrypto]
└─$ cat ciphertext    
9)i�▒�����^E����x�9��w��bm�@�9*����"�B���c��������pw��8                                                                                            

```

* Bài này dùng thuật toán RSA để mã hóa
* Ok, vậy chúng ta có `public key` và `ciphertext` rồi, mình đi tìm `privatekey`

```
┌──(kali㉿kali)-[~/Desktop/CTF/HTB/LowEnergyCrypto]
└─$ openssl rsa -in ./publickey -text -inform PEM -pubin
RSA Public-Key: (512 bit)
Modulus:
    00:a2:8f:1f:19:e6:91:65:42:e1:f8:de:ec:a3:1b:
    59:97:f4:ef:34:75:d0:3d:26:07:d7:e3:8f:8b:65:
    1a:43:ce:5b:e7:fc:24:6c:e0:60:94:56:6e:87:a4:
    72:76:5f:89:0a:cb:46:10:37:44:49:f5:95:42:a4:
    44:09:d4:a8:09
Exponent:
    00:cb:ff:72:e2:e2:5f:91:23:68:34:16:a2:6f:7f:
    77:df
writing RSA key
-----BEGIN PUBLIC KEY-----
MGowDQYJKoZIhvcNAQEBBQADWQAwVgJBAKKPHxnmkWVC4fje7KMbWZf07zR10D0m
B9fjj4tlGkPOW+f8JGzgYJRWboekcnZfiQrLRhA3REn1lUKkRAnUqAkCEQDL/3Li
4l+RI2g0FqJvf3ff
-----END PUBLIC KEY-----
```

* Tiếp theo có Modulus và Exponent rồi, lấy private key thôi

```
p = 92270847179792937622745249326651258492889546364106258880217519938223418249279

q = 92270847179792937622745249326651258492889546364106258880217519938223418258871

e = 271159649013582993327688821275872950239
```

* Mình dùng [rsatool](https://github.com/ius/rsatool/blob/master/rsatool.py) để lấy private key

```
──(kali㉿kali)-[~/Desktop/CTF/HTB/LowEnergyCrypto]
└─$ python3 rsatool.py -p 92270847179792937622745249326651258492889546364106258880217519938223418249279 -q 92270847179792937622745249326651258492889546364106258880217519938223418258871 -e 271159649013582993327688821275872950239 -o private 
Using (p, q) to initialise RSA instance

n =
a28f1f19e6916542e1f8deeca31b5997f4ef3475d03d2607d7e38f8b651a43ce5be7fc246ce06094
566e87a472765f890acb4610374449f59542a44409d4a809

e = 271159649013582993327688821275872950239 (0xcbff72e2e25f9123683416a26f7f77df)

d =
587f5ba09f76fa1f56ddb4bcbe27a2c280f1fe2b51347253eba6b8cf8c53b5a4f524c5a7f1b3c3b2
b0c0dd4f9541c7b7594522f1edc09f4d55914ca4a3c1828f

p =
cbff72e2e25f9123683416a26f7f77cb7199bbe424b9f138dc0dc130b3c2103f

q =
cbff72e2e25f9123683416a26f7f77cb7199bbe424b9f138dc0dc130b3c235b7

Saving PEM as private
                                                                                                                         
┌──(kali㉿kali)-[~/Desktop/CTF/HTB/LowEnergyCrypto]
└─$ cat private                            
-----BEGIN RSA PRIVATE KEY-----
MIIBRwIBAAJBAKKPHxnmkWVC4fje7KMbWZf07zR10D0mB9fjj4tlGkPOW+f8JGzgYJRWboekcnZf
iQrLRhA3REn1lUKkRAnUqAkCEQDL/3Li4l+RI2g0FqJvf3ffAkBYf1ugn3b6H1bdtLy+J6LCgPH+
K1E0clPrprjPjFO1pPUkxafxs8OysMDdT5VBx7dZRSLx7cCfTVWRTKSjwYKPAiEAy/9y4uJfkSNo
NBaib393y3GZu+QkufE43A3BMLPCED8CIQDL/3Li4l+RI2g0FqJvf3fLcZm75CS58TjcDcEws8I1
twIgJXpkF+inPgZETjVKdec6UGg75ZwW3WTPEoVANux3DscCIDjx+RSYECVaraeGG2O/v8iKe6dn
1GpMVGUuaKecISArAiA0QRYkZFB5D4BnOxGkMX3ihjn7NFPQ7+Jk/abWRRq6+w==
-----END RSA PRIVATE KEY-----
```

Có Private key rồi :D dùng Openssl tiếp để có flag

```
┌──(kali㉿kali)-[~/Desktop/CTF/HTB/LowEnergyCrypto]
└─$ openssl rsautl -decrypt -raw -inkey private -in ciphertext 
�5����p+wɹ�[R�.;���^�bs� �6yCHTB{5p34k_fr13nd_4nd_3n73r}
```

So we got the flag
