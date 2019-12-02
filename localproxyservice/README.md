# Local Proxy Service for Robots

This is a local proxy service which needs to run within the wifi network that the robots are using. The service needs to be able to expose an external API with accessible IP/DNS to OpenShift services.
It maps ROBOT names to local IP destinations. 

It takes JSON input as below:
http://10.0.0.10/command
{
    "robotName": "PODMAN",
    "cmdString": "speedRight 50"
}

and will pass this to the ROBOT api located within the local network to 
http://192.168.0.5/rpc/Robot.Cmd
{ "cmd" : "speedright 50" }

The service can be run in the quarkus:dev mode, java jar runner or native executable that can also be containerised.

Notes:
1. Testing locally there were some issues testing the service depending on the binding address and having the service call its mock test service with localhost did not work unless it was bound to certain ips. Need to investigate this issue.

# Add Chinese documents for "How to use localproxyservice"
# Add specical description for localproxyservice support Donkeycar project car.

1. localproxyservice 是为了将 OpenShift 云端下发命令转译并转发给本地局域网的小车。

2. localproxyservice 需要使用 ngork 来暴露一个公网地址，以便让OpenShift可以找到本地服务的位置。

https://dashboard.ngrok.com/get-started
从这里下载ngrok 然后注册一个免费账户。 在本账户获得一个 tocken 。按照网站上的4步骤启动并指定到需要转发的端口。例如 localproxyservice 使用8080 启动时就可以用如下命令
./ngrok http 8080 

启动后本机屏幕上会显示如下结果

ngrok by @inconshreveable                (Ctrl+C to quit)      
Session Status                online       
Account                       zhaoxiyi (Plan: Free)    
Version                       2.3.35    
Region                        United States (us)     
Web Interface                 http://127.0.0.1:4040    
Forwarding                    http://8cc883a6.ngrok.io -> http://localhost:8080   
Forwarding                    https://8cc883a6.ngrok.io -> http://localhost:8080                                                                    
Connections                   ttl     opn     rt1     rt5     p50     p90                         
                              0       0       0.00    0.00    0.00    0.00                        

                                                                                                                                                                     
出现以上内容 意味着本地8080端口可以通过 http://8cc883a6.ngrok.io 访问到

https://dashboard.ngrok.com/auth
网址可以看到ngrok在自己账户下的使用情况

3. localproxyservcie 本地的编译
localproxy 使用了 quarks
因此使用 quarks 的 mvnw 就可以完成编译
./mvnw package

[INFO] --- maven-jar-plugin:2.4:jar (default-jar) @ localproxyservice ---
[INFO] Building jar: /Users/xiyzhao/workspace/DancingRobot/DancingRobot/localproxyservice/target/localproxyservice-1.0-SNAPSHOT.jar
[INFO] 
[INFO] --- quarkus-maven-plugin:0.23.2:build (default) @ localproxyservice ---
[INFO] [io.quarkus.deployment.QuarkusAugmentor] Beginning quarkus augmentation
[INFO] [org.jboss.threads] JBoss Threads version 3.0.0.Final
[INFO] [io.quarkus.deployment.QuarkusAugmentor] Quarkus augmentation completed in 1830ms
[INFO] [io.quarkus.creator.phase.runnerjar.RunnerJarPhase] Building jar: /Users/xiyzhao/workspace/DancingRobot/DancingRobot/localproxyservice/target/localproxyservice-1.0-SNAPSHOT-runner.jar
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  9.424 s
[INFO] Finished at: 2019-12-02T11:36:00+08:00
[INFO] ------------------------------------------------------------------------

当出现 build success 就可以在 target目录使用java 执行了
java -jar ./target/localproxyservice-1.0-SNAPSHOT-runner.jar

接可以打开本地 localproxyservice 服务。

4. 配置 Jetson Nano 连接wifi。
在第一次连接Donkeycar小车时 可以使用 miniusb线 连接Nvidia jetson nano板的miniusb口。连接后可以直接通过 192.168.55.1 访问操作系统（操作系统为ubantu）
目前Donkeycar小车有一个 CLB用户，可以通过ssh直接登陆
ssh CLB@192.168.55.1
密码为 12345678
登陆后使用 nmtui 界面可以配置 wifi 网络
需要注意的是，目前已知的问题，当自动 wifi 路由其使用混杂信道时，Donkeycar时无法自动设置信道id的因此一定要将路由器设置为固定信道id

5. 配置启动 Donkeycar 小车
Donkeycar 小车自己拥有一套完整的操作系统和相应的操作界面。可以参考以下网址全面了解 Donkeycar 项目
https://www.donkeycar.com
https://github.com/autorope/donkeycar

当前小车已配置完成，在用户目录下，例如目前使用 CLB
可以直接在用户目录 /home/CLB 下找到 mycar目录
在mycar目录下直接执行
sudo python3 managed.py drive
即可启动小车的接受程序
远程访问小车目前获得的局域网地址即可直接操作
http://<donkeycarIP>:8887

6. 发送给Donckeycar json指令
donkeycar 可以接受json指令 json格式如下：

{"angle":0,"throttle":0,"drive_mode":"user","recording":true}
 {"angle":-0.008147321428571426,"throttle":0.017633928571428575,"drive_mode":"user","recording":true}
 {"angle":-0.058147321428571444,"throttle":0.10039062499999998,"drive_mode":"user","recording":true}
{"angle":-0.09905133928571427,"throttle":0.15864955357142857,"drive_mode":"user","recording":true}

注意 donkeycar 每接受一个json才会改变上一个状态，也就是说必须接受{"angle":0,"throttle":0,"drive_mode":"user","recording":true}才会停止，否则始终保持上一个接受的状态。

