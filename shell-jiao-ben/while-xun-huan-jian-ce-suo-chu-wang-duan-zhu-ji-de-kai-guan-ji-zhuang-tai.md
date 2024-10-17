# while 循环检测所处网段主机的开关机状态

```bash
#!/bin/bash
i=1
while [ $i -le 254 ]
do
 ping -c2 -i0.2 -W1 192.168.4.$i &>/dev/null
 if [ $? -eq 0 ] ;then
  echo -e "\e[32;1m192.168.4.$i is up[0m"
 else
  echo "192.168.4.$i is down"
 fi
 let i++
done
```
