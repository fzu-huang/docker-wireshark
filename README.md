# docker-wireshark
An image which can provide wireshar in html5. 

# Thanks
This repository cames from https://github.com/ffeldhaus/docker-wireshark and all its bases:
- https://github.com/ffeldhaus/docker-xpra-html5-minimal
- https://github.com/ffeldhaus/docker-xpra-minimal

Thank you ffeldhaus.

# Why fork?

When I saw this amazing project, my first thought was: I can use it to capture packages in the container!
You might think that common container network solutions are implemented in veth, and the network packets of the container can also be captured on the machine, but this is not always the case in my usage scenario. Some containers will use sriov vf for acceleration. I can only Capture packets for vf; for veth scenarios, directly entering the container to capture packets can also eliminate a lot of noise.

The original project has been very complete, but when I'm starting the wireshark container, it cannot be controlled by commands to enter the network namespace of a certain container, for example:

```
$ docker run -it -d --name hyshark --net host --user=0 --pid=host --ipc=host --cap-add NET_ADMIN --privileged  ffeldhaus/wireshark nsenter -t 11916 -n wireshark --fullscreen

$ docker logs  a5ba76e513aa
Warning: using '/run/user/1000' as XDG_RUNTIME_DIR
...
2021-04-28 09:04:13,025 created tcp socket '0.0.0.0:14500'
...
2021-04-28 09:04:16,234 started command 'nsenter -t 11916 -n wireshark --fullscreen' with pid 8615
nsenter: cannot open /proc/11916/ns/net: Permission denied
2021-04-28 09:04:16,236 Warning: cannot watch for application menu changes without pyinotify:
...
2021-04-28 09:04:16,457 child 'nsenter -t 11916 -n wireshark --fullscreen' with pid 8615 has terminated
2021-04-28 09:04:16,466 all children have exited and --exit-with-children was specified, exiting
2021-04-28 09:04:16,476 xpra GTK3 X11 server is terminating
2021-04-28 09:04:16,531 waiting for initialization thread to complete
2021-04-28 09:04:16,581  uid=1000 (xpra), gid=106 (xpra)
2021-04-28 09:04:16,581  running with pid 8567 on Linux unknown unknown unknown
2021-04-28 09:04:17,083 1.9GB of system memory
2021-04-28 09:04:18,244 OpenGL is supported on display ':0'
2021-04-28 09:04:18,245  using 'llvmpipe (LLVM 11.0.0, 256 bits)' renderer
2021-04-28 09:04:18,248 killing xvfb with pid 8590
2021-04-28 09:04:18,248 closing tcp socket '0.0.0.0:14500'
2021-04-28 09:04:18,248 removing unix domain socket '/run/user/1000/xpra/cil2-0'
Gdk-Message: 09:04:18.258: Xpra: Fatal IO error 0 (Success) on X server :0.

```

# Changelog

## cancel user `xpra` and use `root` to run all commands in container.

In order to ensure security, the startup user is specified in the base image `ffeldhaus/xpra-minimal`, which means that no matter how I configure it during `docker run`, the user who runs xpra in the created container is always `xpra` and it cannot execute `nsenter`. So I adjusted the script: `docker-entrypoint.sh` in the base image.

# Build

```
docker build  --network=host -f   ./Dockerfile.docker-xpra-minimal  -t fzu-huang/docker-xpra-minimal .

docker build --network=host -f ./Dockerfile.docker-xpra-html5-minimal  -t fzu-huang/docker-xpra-html5-minimal .

docker build --network=host -f ./Dockerfile.wireshark  -t fzu-huang/wireshark . 

```

# Usage

## Step 1. Get pod's Sandbox Pid

Example:

```
# docker inspect   1244c1396fab -f "{{.State.Pid}}"
12742
```

## Step 2. Start wireshark

```
docker run -it -d --name hyshark --net host --pid=host --ipc=host --cap-add NET_ADMIN --privileged hub.c.163.com/monsterhy/wireshark nsenter -t 11916 -n wireshark --fullscreen
```

## Step 3. Visit wireshark in your browser

![image](https://user-images.githubusercontent.com/5361248/116384508-7e016900-a84a-11eb-8354-e5d8167835ef.png)

you can get more info in https://github.com/ffeldhaus/docker-wireshark
