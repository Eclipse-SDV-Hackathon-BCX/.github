# About the JetRacer

The JetRacer is a robot car using a Jetson Nano DevKit as its compute unit. It is preconfigured with Ubuntu 18.04 LTS and the default software from Waveshare.
Additional information can be found at the Waveshare wiki site: https://www.waveshare.com/wiki/JetRacer_AI_Kit

The device exposes a JupyterLab Notebook interface. When connecting using the USB port, use your browser to connect to http://192.168.55.1:8888. Otherwise, connect it with the Ethernet cable and look at the IP address in the small display on the car. From the notebook it is possible to test all components. The interface also contains a terminal.

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

- Make sure that docker is installed and that the default user is added to the docker group (might require login in back)

```
docker --version

sudo usermod -aG docker $USER
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

[Eclipse KUKSA.val server](https://www.eclipse.org/kuksa/) can be installed as a docker container using the following command:

```
docker run -it --rm -v $HOME/kuksaval.config:/config  -p 127.0.0.1:8090:8090 -e LOG_LEVEL=ALL ghcr.io/eclipse/kuksa.val/kuksa-val:0.2.1-arm64
```

KUKSA.val server will start and be ready to accept commands using gRPC. To verify it is running, use the following docker command:

```
docker container list
```
The result is a list like this:

``` bash
CONTAINER ID   IMAGE                                             COMMAND                  CREATED              STATUS          PORTS                      NAMES
8444a8bdc70a   ghcr.io/eclipse/kuksa.val/kuksa-val:0.2.1-arm64   "/bin/sh -c /kuksa.v…"   About a minute ago   Up 58 seconds   127.0.0.1:8090->8090/tcp   tender_hodgkin
```

## Eclipse Muto
For [Eclipse Muto]([https://github.com/eclipse/chariott](https://eclipse-muto.github.io/docs/docs/muto-edge/getting-started/by-example)) on the JetRacer, follow these steps:

### ROS on JetRacer

Nvidia JetRacer with Jeson Nano 4GB uses JetPack 4.5.1 which is based on Ubuntu 18.02. This sets the ROS version to use to [ROS Melodic](http://wiki.ros.org/melodic/Installation/Ubuntu), see[ROS Distros here](http://wiki.ros.org/Distributions). You do not have to install ROS seperately, the following step will do it for you.


###  Installing Eclipse Muto on JetRacer

It is very easy to install Muto on the JetRacer:
* Complete the hardware and software installtion steps described in [JetRacer AI Kit Wiki Pages](https://www.waveshare.com/wiki/JetRacer_AI_Kit)
* Connect to you JetRacer

         ssh jetson@<YOUR JETRACER IP>

* Checkout this repository

        git clone https://github.com/composiv/jetracer_example.git

* Change into the jetracer_example folder and run the setup script

        cd jetracer_example
        ./setup.sh
        [sudo] password for jetson:

* When prompted for password enter `jetson` or your password
* Sit back and relax while ROS melodic and Muto is being installed and setup. This may take sometime depending on your network speek. Make sure the Jetracer is plugged in.

        REDACTED

        [ 38%] Built target muto_msgs_generate_messages_py
        [ 56%] Built target muto_msgs_generate_messages_nodejs
        [ 74%] Built target muto_msgs_generate_messages_cpp
        [ 93%] Built target muto_msgs_generate_messages_eus
        [ 96%] Built target jetson_camera_node
        [100%] Built target jetson_camera_nodelet
        [100%] Built target muto_msgs_generate_messages


### Start Eclipse Muto on JetRacer

The scripts for starting muto on the JetRace is provided. After completing  "Installing Muto on JetRacer".  
* Complete  "Installing Muto on JetRacer".  
* Connect to you JetRacer

         ssh jetson@<YOUR JETRACER IP>

* Change into the jetracer_example folder and edit the [launch/config/muto.yaml](https://github.com/composiv/jetracer_example/blob/main/launch/config/muto.yaml) configuration to change its name and connect to the Muto Twin server of your choice:

```diff title="launch/config/muto.yaml"
muto:
  stack_topic: /stack
  twin_topic: /twin
  type: simulator
+ twin_url: "http://ditto:ditto@sandbox.composiv.ai"
  commands:
      - name: ros/topic
        service: rostopic_list
        plugin: CommandPlugin
      - name: ros/topic/info
        service: rostopic_info
        plugin: CommandPlugin
      - name: ros/topic/echo
        service: rostopic_echo
        plugin: CommandPlugin
      - name: ros/node
        service: rosnode_list
        plugin: CommandPlugin
      - name: ros/node/info
        service: rosnode_info
        plugin: CommandPlugin
      - name: ros/param
        service: rosparam_list
        plugin: CommandPlugin
      - name: ros/param/get
        service: rosparam_get
        plugin: CommandPlugin
  pipelines:
      - name:  start
        pipeline:
          - sequence:
            - service: muto_compose
              plugin: ComposePlugin
            - service: muto_start_stack
              plugin: ComposePlugin
        compensation:
          - service: muto_kill_stack
            plugin: ComposePlugin   
      - name:  kill
        pipeline:
          - sequence:
            - service: muto_compose
              plugin: ComposePlugin
            - service: muto_kill_stack
              plugin: ComposePlugin
        compensation:
          - service: muto_kill_stack
            plugin: ComposePlugin 
      - name:  apply
        pipeline:
          - sequence:
            - service: muto_compose
              plugin: ComposePlugin
            - service: muto_apply_stack
              plugin: ComposePlugin
        compensation:
          - service: muto_kill_stack
            plugin: ComposePlugin 
  mqtt:
    host: sandbox.composiv.ai # subject to change
    port: 1883
    keep_alive: 60
    user: none
    password: none
  thing:
    namespace: org.eclipse.muto.sandbox # subject to change
    definition: org.eclipse.muto:EdgeDevice:0.0.1
    attributes:
      brand: jetracer
      model: ai-racer
    anonymous: False  # Use this for automatically generated id (uuid based)
    #   if anonymous is True or anynoymous param is missing, name/id will be auto generated
    # TODO: edit the name below
+   name: bcx-jetracer-muto

```
* Change into the jetracer_example folder and start Muto on the Jet Racer:

        cd jetracer_example
        ./start.sh

* Muto will start and rgister the Car with the twin server.

        ...
        REDACTED
        ...
        * /rosbridge_websocket/websocket_ping_timeout: 30
        * /rosdistro: melodic
        * /rosversion: 1.14.13

        NODES
        /
            composer_plugin (muto_composer/composer_plugin.py)
            launch_plugin (muto_composer/launch_plugin.py)
            muto_agent (muto_agent/muto_agent.py)
            muto_composer (muto_composer/muto_composer.py)
            rosapi (rosapi/rosapi_node)
            rosbridge_websocket (rosbridge_server/rosbridge_websocket)

        ROS_MASTER_URI=http://localhost:11311

        process[muto_agent-1]: started with pid [19100]
        process[muto_composer-2]: started with pid [19102]
        process[composer_plugin-3]: started with pid [19103]
        process[launch_plugin-4]: started with pid [19104]
        process[rosbridge_websocket-5]: started with pid [19106]
        process[rosapi-6]: started with pid [19109]
        ...
        Connected with result code Success
        ('Subscribed to: ', 'org.eclipse.muto.sandbox:bcx-sim-01aa/#')
        2022-11-06 23:26:39+0800 [-] [INFO] [1667748399.104261]: Client connected.  1 clients total.
        2022-11-06 23:26:43+0800 [-] [INFO] [1667748403.881240]: [Client 0] Subscribed to /main_camera/camera_info
        2022-11-06 23:26:43+0800 [-] [INFO] [1667748403.906542]: [Client 0] Subscribed to /main_camera/image_raw
        2022-11-06 23:26:43+0800 [-] [INFO] [1667748403.940345]: [Client 0] Subscribed to /drive
        2022-11-06 23:26:43+0800 [-] [INFO] [1667748403.982889]: [Client 0] Subscribed to /joy


### Start the Base JetRacer Stack from the Eclipse Muto Dashboard

New we can start a JetRacer `Stack` on the car to get access to the `Camera`, control the Car with the `Gamepad` and visualize with `Foxglove Studio`.  
*  Got to the [`Dashboard` at https:/dashboard.composiv.ai](https://dashboard.composiv.ai). 

<img src="https://github.com/composiv/jetracer_example/blob/main/assets/dashboard.gif?raw=true" height=512>

*  After you start the stack. You should see something similar to the following on your console:

        ...
        [ WARN] [1667748916.584222174]: [camera] does not match name raspicam_v2 in file /home/jetson/junk/src/jetracer/jetson_camera/camera_info/raspicam_v2.yaml
        [ WARN:0] global /home/nvidia/host/build_opencv/nv_opencv/modules/videoio/src/cap_gstreamer.cpp (933) open OpenCV | GStreamer warning: Cannot query video position: status=0, value=-1, duration=-1
        WARNING: Carrier board is not from a Jetson Developer Kit.
        WARNNIG: Jetson.GPIO library has not been verified with this carrier board,
        WARNING: and in fact is unlikely to work correctly.

* Press the buttons on your Gamepad to move the JetRacer around.  You can change how the gamepad controls the car by modifying the configuration parameters in [src/jetracer/jetracer_teleop/jetracer.yaml](https://github.com/composiv/jetracer_example/blob/main/src/jetracer/jetracer_teleop/jetracer.yaml): 

### Visualizing the JetRacer with Foxglove Studio

Navigate to foxglove studio webbsite. Click on Open Connection and change the Port ID from 9090 to your own port ID. In this example, we set the websoocket port to 7777, so set the data connection to ws://locahost:7777. Now we we can import the layout. These layouts will display predfined panels suitable for visualization oıf our example:

<img src="https://github.com/composiv/jetracer_example/blob/main/assets/foxglove.png?raw=true" height=512>

