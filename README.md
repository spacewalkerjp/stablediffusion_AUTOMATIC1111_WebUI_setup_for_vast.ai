# Setup : AUTOMATIC1111 / Stable Diffusion WebUI ver 1.1.0 (torch2.0.0) for vast.ai
* GPU sharing cloud service `Vast.ai` : https://cloud.vast.ai/
* Setup memo : `2023/05/01 ver` for AUTOMATIC1111/stable-diffusion-webui 1.1.0(2023/05/01)

# Preparation
1) Setup your local PC(Windows, macOS, Linux,...)'s SSH client setting. Generate a SSH keypair for remote access.
2) Create a new account for vast.ai `Client account type`.
3) Regist your SSH public key into vast.ai `Account` menu.
4) Add credit 10 USD with your credit card on `Billing` menu.

# GPU Instance configuration
* Base docker image : `nvidia/cuda:11.8.0-cudnn8-runtime-ubuntu22.04`
* Launch Type: `ssh`
* On-start script: `Not set`
* Disk Space To Allocate: `over ~50GB` (recommend)
* Launch mode : 
   * `Run interactive shell server, SSH`. This will allow you to connect and run commands using an SSH client.
   * `Checked` : Use direct SSH connection - faster than proxy, but limited to machines with open ports. Proxy ssh still available as backup.
* GPU Type :
   *  `Interruptible` and `On-Demand` : both OK
   *  #GPUs : `1X` (if the instance has multi GPUs, the current version of Stable Diffusion uses only 1X GPU.) 

# Setup procedures
1. Start a instance based on the above `Instance configuration`.
2. Wait until the instance loads the docker image and starts up. (5 minutes at the fastest, 20 minutes at the longest)
3. Push the `>_ CONNECT` button and copy the `Direct ssh connect` command.
* ex) 
```sh
ssh -p XXXXX root@AAA.BBB.CCC.DDD -L 8080:localhost:8080
```
4. Paste the command into your terminal/command prompt and add `-L 7860:localhost:7860` for browser access (SSH local port fowarding)
* ex)
```sh
ssh -p XXXXX root@AAA.BBB.CCC.DDD -L 7860:localhost:7860
```
5. Connect the instance via SSH with the above `4.` ssh command.
6. Install the AUTOMATIC1111 / stable-diffusion-webui (v1.1.0)

6.1 step1 (as ROOT)
```sh
apt-get install vim unzip libgl1-mesa-dev wget git -y
apt-get install python3 python3-venv -y
adduser user1 --disabled-password --gecos ""
su user1
```

6.2 step2 (as user1 = not root user)
```sh
cd ~
bash <(wget -qO- https://raw.githubusercontent.com/AUTOMATIC1111/stable-diffusion-webui/master/webui.sh)

# after launch, once terminate the webui with `ctrl+c`

# important symbolic link!!
cd ~/stable-diffusion-webui/venv/lib/python3.10/site-packages/torch/lib
ln -s libnvrtc-672ee683.so.11.2 libnvrtc.so
cd ~/stable-diffusion-webui/

chmod +x webui.sh
./webui.sh -opt-sdp-attention
```

7.Access the stable diffusion by your local PC's web browser
   * http://localhost:7860
   * via SSH port forward by `-L 7860:localhost:7860`


# how to terminate stable diffusion and restart
* terminate : `Ctrl + C`
* restart : ./webui.sh -opt-sdp-attention
* restart from SSH connection :
```sh
su user1
cd ~
cd ~/stable-diffusion-webui/
./webui.sh -opt-sdp-attention
```

# how to install checkpoint model

Execute as user1
```sh
wget -P /home/user1/stable-diffusion-webui/models/Stable-diffusion/ http://example.com/HogeHogeModel.safetensors
```

# how to install VAE

Execute as user1
```sh
wget -P /home/user1/stable-diffusion-webui/models/VAE/ http://example.com/vae-ft-mse-840000-ema-pruned.safetensors
```

# how to install LoRA 

Execute as user1
```sh
wget -P /home/user1/stable-diffusion-webui/models/Lora/ http://example.com/HogeHogeLora.safetensors
```

# how to install embeddings

Execute as user1
```sh
wget -P /home/user1/stable-diffusion-webui/embeddings/ http://example.com/EasyNegative.safetensors
```

# Download the outputs from the instance to local PC.

Execute from your Local PC's terminal
```sh
scp -r -P XXXXX root@AAA.BBB.CCC.DDD:/home/user1/stable-diffusion-webui/outputs ./outputs/
```

# (Option) Apache2 for output directory
* as a root user
```sh
apt install apache2 -y
a2enmod userdir
/etc/init.d/apache2 restart
su user1
cd
mkdir ~/public_html
chmod 711 $HOME
chmod 755 ~/public_html
ln -s /home/user1/stable-diffusion-webui/outputs/ ~/public_html/outputs
```

* recoonect with port forwarding setting for web server
```sh
ssh -p XXXXX root@AAA.BBB.CCC.DDD -L 8080:localhost:80 -L 7860:localhost:7860
```

* access from local PC with web browser (via SSH port forwarding)
* http://localhost:8080/~user1/




# Ref. ( &Special thanks)
* https://cloud.vast.ai
* https://github.com/AUTOMATIC1111/stable-diffusion-webui
* https://hub.docker.com/r/nvidia/cuda/tags
* https://pytorch.org/get-started/pytorch-2.0/
* https://github.com/CompVis/stable-diffusion
