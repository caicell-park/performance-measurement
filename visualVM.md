
# VisualVM

An engineer who has used applications running by java has a requirement to analysis its performance. VisualVM is one of the performance measurment tools of java. 
This documents introduces the followings:

- [Run VisualVM](#01)
- TBA

<a id=01></a>

## Run VisualVM

### 1. Download [VisualVM](https://visualvm.github.io/download.html)

### 2. Configure an environment for VisualVM on a remote machine

#### 2.1. Find out an unused port on a remote machine

A linux command, lsof, monitors a list of used ports.
```shell
# Check a port
sudo lsof -i:$PORT
COMMAND   PID  USER   FD   TYPE             DEVICE SIZE/OFF NODE NAME
ruby    24670 kakao   11u  IPv4 0xf8c58feae0a4e281      0t0  TCP localhost:hbci (LISTEN)

# List up used ports
sudo lsof -i -P -n | grep LISTEN
...
ruby      24670           kakao   11u  IPv4 0xf8c58feae0a4e281      0t0    TCP 127.0.0.1:3000 (LISTEN)
node      24910           kakao   23u  IPv4 0xf8c58feae30f86d1      0t0    TCP *:3001 (LISTEN)
PulseSetu 79163           kakao    5u  IPv4 0xf8c58feae0a50b21      0t0    TCP 127.0.0.1:3380 (LISTEN)
java      79774           kakao   21u  IPv6 0xf8c58feac2833621      0t0    TCP [fe80:1::1]:62108 (LISTEN)
...
```

#### 2.2. Allow a port using for VisualVM

Run a rmregister server for a port

```shell
rmiregistry $PORT
```

Allow a permission for a VisualVM to access an information of java VM. 

```shell
cat > ${JAVA_HOME}/bin/tools.policy <<EOL
grant codebase "file:${JAVA_HOME}/../lib/tools.jar" {
   permission java.security.AllPermission;

};
EOL
```

Run a daemon, jstatd 

```shell
jstatd -p $PORT -J-Djava.security.policy=${JAVA_HOME}/bin/tools.policy
```

Allow to access JMX inforation to a java application 

> set JAVA_OPTS=%JAVA_OPTS% %LOGGING_CONFIG% -Dcom.sun.management.jmxremote.authenticate=false -Dcom.sun.management.jmxremote.ssl=false -Dcom.sun.management.jmxremote=true -Dcom.sun.management.jmxremote.port=9983

#### 2.3. Restrict the specific machine to access a machie that runs a java application

```shell
# IP and subnet mask example: 203.0.113.0/24
sudo iptables -A INPUT -p tcp -s $IP/$SUBNET_MASK --dport $PORT -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT
sudo iptables -A OUTPUT -p tcp --sport $PORT -m conntrack --ctstate ESTABLISHED -j ACCEPT
sudo iptables -A INPUT -p tcp --dport $PORT -j DROP
```
