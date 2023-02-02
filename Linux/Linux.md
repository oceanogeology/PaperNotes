# Linux骚操作集锦

1.   构造系统函数  vi ~/.bashrc  source ~/.bashrc

```shell
login-ftp(){ 
docker run  -v /mnt:/mnt -v /home:/home --rm -it duankui:cuda10.0-cudnn7-mxnet1.5-torch1.8-py3.8.5-cv2-ssh-req-gjiajia-htop  expect /home/duankui/config/login-ftp
}
```

2. 解决mv太多问题-bash: /bin/mv: Argument list too long

```
find sourcePath/ -name "*.txt"  -exec mv {} targetPath/  \;
#or
find sourcePath/ -type f  -exec mv {} targetPath/  \;
```

3. 查看进程信息

```
ls /proc/32270 -al
ps -aux | grep 进程id 
ps -ef | grep id

```

4. 杀死系列进程

```
ps -ef  | grep du | awk '{print $2}' | xargs -i kill -9 {}

ps -ef| grep python | grep -v grep | awk '{print $2}' | xargs kill -9

for ((i=1; i<=50; i ++))
do
	echo $i
    nohup tar -czf dir${i}.tgz dir${i} > ${i}.log 2>&1 &
done
```

git强制覆盖本地命令（单条执行）：
git fetch --all && git reset --hard origin/master && git pull



ffmpeg下视频

```
ffmpeg -i "http://video.hobby666.com/9e7fde73vodtranshk1308381747/38a9c366387702298400013731/adp.1228215.m3u8?t=6300e5ca&sign=cd6422980739146b54b26f62b45b1938" -c copy -bsf:a aac_adtstoasc 388837889134141440.mp4 -y
```

