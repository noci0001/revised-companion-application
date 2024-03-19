- Seat Adjuster
   - developed in DevContainer
   - import example app from SDK
   - Start runtime to test and debug:
      - `velocitas exec runtime-local up`
   - Run inside a container
      - `velocitas exec runtime-kanto up`
      - Once available, add the app:
         - `velocitas exec deployment-kanto build-vehicleapp`
         - `velocitas exec deployment-kanto deploy-vehicleapp`
      - Inside the Kanto local runtime we can test the application
      - shutdown: `velocitas runtime-kanto down`
   - Execute Pre-Commit Release Task
      - CI workflow Build Multiarch image
   - Generate Image to run in Leda
- QEMU and Leda
   - setup QEMU
      - `sudo apt-get update -y`
      - `sudo apt-get install -y xz-utils qemu-system-x86-64`
   - setup Leda on Linux
      - curl -fsSL https://cli.github.com/packages/githubcli-archive-keyring.gpg | sudo dd of=/usr/share/keyrings/githubcli-archive-keyring.gpg
      - `echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" | sudo tee /etc/apt/sources.list.d/github-cli.list > /dev/null`
      - `sudo apt update`
      - `sudo apt install gh`
      - `sudo auth login`
      - `mkdir leda && cd leda`
      - gh release download \
      --pattern '*.zip' \
      --pattern 'eclipse-leda-*' \
      --repo eclipse-leda/leda-distro
      - `./run-leda.sh`
      - login as root
      - (do not continue to device provisioning)
- Inside Qemu Leda
   - check containers with `kantui` or `kanto-cm list`
   - Disable (`kanto-cm list` and then `r` ot remove the container you are hovering on) interfering containers: `feedercan, seatservice-example, cloudconnector, hvacservice-example, sua, vum`.
   - Start container (2 ways)
      - Option A: with KANTO-CM:
         - kanto-cm create \
            --name seatadjuster-app \
            --e="SDV_SEATSERVICE_ADDRESS=grpc://seatservice-example:50051" \
            --e="SDV_MQTT_ADDRESS=mqtt://mosquitto:1883" \
            --e="SDV_VEHICLEDATABROKER_ADDRESS=grpc://databroker:55555" \
            --e="SDV_MIDDLEWARE_TYPE=native" \
            --hosts="databroker:container_databroker-host, mosquitto:host_ip, seatservice-example:container_seatservice-example-host" \
            ghcr.io/<YOUR_ORG>/seat-adjuster-app:latest
            - the ghcr.io string with the image inside your packages. e.g.:
            `docker pull ghcr.io/noci0001/my-seat-adjuster/seatadjuster:1`
         - `kanto-cm start --name seatadjuster-app`
         - `kanto-cm logs --name seatadjuster-app`
      - Option B: adding container manifest
         - `cd ~/../../`
         - `touch seat-adjuster.json`
         - `nano seat-adjuster.json`
         - Paste: 
            {
               "container_id": "seatadjuster-app",
               "container_name": "seatadjuster-app",
               "image": {
                  "name": "ghcr.io/<identifier-for-container>:<tag-for-container>"
               },
               "host_config": {
                  "devices": [],
                  "network_mode": "bridge",
                  "privileged": false,
                  "restart_policy": {
                        "maximum_retry_count": 0,
                        "retry_timeout": 0,
                        "type": "unless-stopped"
                  },
                  "runtime": "io.containerd.runc.v2",
                  "extra_hosts": [        
                           "mosquitto:host_ip",
                           "databroker:container_databroker-host",
                           "seatservice-example:container_seatservice-example-host"
                  ],
                  "port_mappings": [
                        {
                        "protocol": "tcp",
                        "container_port": 30151,
                        "host_ip": "localhost",
                        "host_port": 50151,
                        "host_port_end": 50151
                        }
                  ],
                  "log_config": {
                        "driver_config": {
                           "type": "json-file",
                           "max_files": 2,
                           "max_size": "1M",
                           "root_dir": ""
                        },
                        "mode_config": {
                           "mode": "blocking",
                           "max_buffer_size": ""
                        }
                  },
                  "resources": null
               },
               "config": {
                  "env": [
                     "SDV_SEATSERVICE_ADDRESS=grpc://seatservice-example:50051",
                     "SDV_VEHICLEDATABROKER_ADDRESS=grpc://databroker:55555",
                     "SDV_MQTT_ADDRESS=mqtt://mosquitto:1883",
                     "SDV_MIDDLEWARE_TYPE=native",
                     "RUST_LOG=info",
                     "vehicle_data_broker=info"
                  ],
                  "cmd": []
               }
            }
   - ECU not available? -> mock it
      - create container manifest of mockservice in /data/var/containers/manifest from boilerplate found [here](`https://sdv-blueprints.eclipse.dev/docs/companion-application/deploy-seat-adjuster/#seat-applicationjson`) in seat-application.json section
      - create the path /data/var/mock/mock.py and fill it with boilerplate provided in [here](`https://sdv-blueprints.eclipse.dev/docs/companion-application/deploy-seat-adjuster/#seat-applicationjson`) in the mock.py section
   - INTERACTING WITH SEAT-ADJUSTER
      - Work with two parallel terminals both  runing leda/QEMU. To do so you must be running QEMU and open the second terminal via ssh:
         - `ssh -p 2222 root@localhost`
      - In one terminal, subscribe to the MQTT topic:
         - `mosquitto_sub -t 'seatadjuster/#' -v`
      - To move the seat, publish the message in the other terminal:
         - `mosquitto_pub -t seatadjuster/setPosition/request -m '{"position": 1000, "requestId": "12345"}'`
- Run Kuksa Client:
   - kanto-cm create --i --t --network=host --name=kuksa-client ghcr.io/eclipse/kuksa.val/kuksa-client:0.3.0
   - kanto-cm start --name=kuksa-client
   sdv-ctr-exec -n kuksa-client /kuksa-client/bin/kuksa-client --port 30555 --protocol grpc --insecure
