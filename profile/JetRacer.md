# About the JetRacer

The JetRacer is a robot car using a Jetson Nano DevKit as its compute unit. It is preconfigured with Ubuntu 18.04 LTS and the default software from Waveshare.
Additional information can be found at the Waveshare wiki site: https://www.waveshare.com/wiki/JetRacer_AI_Kit

The device exposes a JupyterLab Notebook interface. When connecting using the USB port, use your browser to connect to http://192.168.55.1:8888. From the notebook it is possible to test all components. The interface also contains a terminal.

# WiFi configuration

To configure Wifi: 

``` bash
sudo nmcli device wifi list
sudo nmcli device wifi connect <ssid_name> password <password>
```

The devices will be configured to use the BCX unrestricted network

# Installing SDV Components

## Support tools

- gRPCurl enables developers to test Eclipse SDV components that use gRCP such as Chariot and Kuksa. The following commands create a directory, download the latest binaries and create a symbolic link

```
sudo mkdir /usr/bin/grpcurl.d

curl -sSL https://github.com/fullstorydev/grpcurl/releases/download/v1.8.7/grpcurl_1.8.7_linux_arm64.tar.gz | sudo tar -xvz --directory /usr/bin/grpcurl.d

sudo ln /usr/bin/grpcurl.d/grpcurl /usr/bin/grpcurl
```

- Make sure that docker is installed

```
docker --version
```

## Eclipse Chariott

[Eclipse Chariott](https://github.com/eclipse/chariott) has the following preconditions that need to be installed

- Rust compiler: the recommended approach is to download and execute the install script.
```
curl https://sh.rustup.rs -sSf | sh
```

- Protobuf development kit: it can be installed using apt-get directly

``` bash
sudo apt-get install  libprotobuf-dev protobuf-compiler
```

- dotnet (only when using the dog mode)

After installing it is possible to check out the code from https://github.com/eclipse/chariott, compile and run the examples. It is not necessary to use a dev container. 

On the folder execute
```
cargo build --workspace
```

And then follow the intructions to run the [Key-Value Store Application](https://github.com/eclipse/chariott/blob/main/examples/applications/kv-app/README.md)

## Eclipse KUKSA.val

[Eclipse KUKSA.val](https://www.eclipse.org/kuksa/) can be installed as a docker container 
