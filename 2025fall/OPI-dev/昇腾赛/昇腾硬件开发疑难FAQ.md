# 11月13日

1. 千万不要忘记在烧镜像的时候解压缩
2. 有的时候板子上hdmi口会分 HDMIO以及HDM1， 有的时候必须要HDMIO才能启动
3. route -n可以查看板子的网关
	1. 注意名称：USB0不知道是什么
	2. Metric越小优先级越高
	3. ping的发出网关要是wlan0对应的才能联网，进行wget和git拉取等等操作
	4. 所以连不到网很可能是wlan0优先级不够，这个时候先看看他的优先级
	5. 暂时禁用有线网或者USB0可以用nmcli con show看看NAME一栏，然后执行 sudo nmcli con down "usb0-static"指令即可暂时禁用此网关
4. 注意UART是串口连接，暂时不清楚这个与USB0有什么联系，但是感觉没关系，因为拔出串口之后USB0没有消失
5. 注意OrangepiAIPro 20T有两个网口，如果修改一个网口没有用的话可以用nmcli con show指令看一下，Wired connection 1， Wired connection 2都尝试修改一下ip
6. 注意ip不能乱搞，香橙派有线连接应该只支持192.168.137.\_的网关
7. 如果一直黑屏的话可以使用串口+MobaXterm调出终端调试，不用束手无策
8. **注意更换ip之后jupyter的配置ip也要一起更改**
	1. vi start_notebook.sh
	2. ```bash
	    . /usr/local/Ascend/ascend-toolkit/set_env.sh
		export PYTHONPATH=/usr/local/Ascend/thirdpart/aarch64/acllite:$PYTHONPATH
	    if [ $# -eq 1 ];then
			jupyter lab --ip $1 --allow-root --no-browser
		else
			jupyter lab --ip 192.168.3.16 --allow-root --no-browser
		fi
	   ```


#### 这样终于把所有的乱七八糟的配置搞定了
1. 我找到了目前的案例的路径/home/HwHiAiUser/orange-pi-mindspore/Online/
2. 然后直接依照README里面的指导执行操作（今天肝不动了


# 11月14日

这是README.md, 记录了一些很好的案例
[[samples指导文件]]

这里是qwen0.5 1.5b模型的指导
[[samples指导文件#模型在线推理]]

/home/HwHiAiUser/orange-pi-mindspore 里面直接有start.sh，可以打开jupyter

mindspore生态的缓存与hugginggace的标准不太一样，他需要直接指定到快照版本的内部，直接能够到核心.json文件才能正常运行
所以从windowsPC端复制之后还要多一步cp扁平化文档

VNC服务要先杀vncserver
```bash
vncserver -kill :1
vncserver -geometry 2400x1600 :1
```
如果vnc登录的是sh-5 # 这种，就需要使用
```bash
su - HwHiAiUser
```
切换用户，这样才会加载conda和其他配置