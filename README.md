# Setup : AUTOMATIC1111 / Stable Diffusion WebUI ver 1.6.0-RC for vast.ai (including SDXL 1.0)
* GPU sharing cloud service `Vast.ai` : https://cloud.vast.ai/
* Setup memo : `2023/08/28 ver` for AUTOMATIC1111/stable-diffusion-webui 1.6.0-RC
* Video explaining in Japanese (Youtube) : https://www.youtube.com/watch?v=U4HrpzkinP4

# Preparation
1) Setup your local PC(Windows, macOS, Linux,...)'s SSH client setting. Generate a `SSH keypair(private key & public key)` for remote access.
2) Create a new account for vast.ai `Client Account Type`.
3) Regist your SSH public key into vast.ai `Account` menu.
4) Add credit 10 USD with your credit card on `Billing` menu. I DON'T recommend `Auto charge` setting.

# GPU Instance configuration
* Base docker image : `nvidia/cuda:11.8.0-cudnn8-runtime-ubuntu22.04`
* Launch Type: `ssh`
* On-start script: `Not set`
* Disk Space To Allocate: `over ~60GB` (recommend)
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
apt-get install vim unzip libgl1-mesa-dev libcairo2-dev wget git -y
apt-get install python3 python3-venv python3-dev build-essential -y
adduser user1 --disabled-password --gecos ""
su user1
```

6.2 step2 (as user1 = not root user)
```sh
cd ~
#bash <(wget -qO- https://raw.githubusercontent.com/AUTOMATIC1111/stable-diffusion-webui/master/webui.sh)
wget https://github.com/AUTOMATIC1111/stable-diffusion-webui/archive/refs/tags/v1.6.0-RC.tar.gz
tar xvfz v1.6.0-RC.tar.gz
ln -s stable-diffusion-webui-1.6.0-RC stable-diffusion-webui
cd stable-diffusion-webui/
# launch AUTOMATIC1111 WebUI (with xformers)
./webui.sh --xformers
```

7.Access the stable diffusion by your local PC's web browser
   * http://localhost:7860
   * via SSH port forward by `-L 7860:localhost:7860`


# how to terminate stable diffusion and restart
* terminate by `Ctrl + C`
* restart : ./webui.sh
## restart from SSH connection :
```sh
su user1
cd ~
cd ~/stable-diffusion-webui/
./webui.sh
```

# how to install checkpoint model

Execute as user1
```sh
wget -P /home/user1/stable-diffusion-webui/models/Stable-diffusion/ http://example.com/HogeHogeModel.safetensors
```

* SDXL 1.0 (base + refiner)
```sh
wget -P /home/user1/stable-diffusion-webui/models/Stable-diffusion/ https://huggingface.co/stabilityai/stable-diffusion-xl-base-1.0/resolve/main/sd_xl_base_1.0.safetensors
wget -P /home/user1/stable-diffusion-webui/models/Stable-diffusion/ https://huggingface.co/stabilityai/stable-diffusion-xl-refiner-1.0/resolve/main/sd_xl_refiner_1.0.safetensors
```

# how to install VAE

Execute as user1
```sh
wget -P /home/user1/stable-diffusion-webui/models/VAE/ http://example.com/vae-ft-mse-840000-ema-pruned.safetensors
```
* SDXL 1.0 VAE
```sh
wget -P /home/user1/stable-diffusion-webui/models/VAE/ https://huggingface.co/stabilityai/sdxl-vae/resolve/main/sdxl_vae.safetensors
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
```
* as a `user1` user
```sh
cd
mkdir ~/public_html
chmod 711 $HOME
chmod 755 ~/public_html
ln -s ~/stable-diffusion-webui/outputs/ ~/public_html/outputs
```

* recoonect with port forwarding setting for web server
```sh
ssh -p XXXXX root@AAA.BBB.CCC.DDD -L 8080:localhost:80 -L 7860:localhost:7860
```

* access from local PC with web browser (via SSH port forwarding)
* http://localhost:8080/~user1/

# (Additional info) Multi GPU settings
* [1] start a instance which has 2 multi GPUs in vast.ai
* [2] connect the instance with SSH port forward
```sh
ssh -p XXXXX root@AAA.BBB.CCC.DDD -L 7860:localhost:7860 -L 7861:localhost:7861
```
* [3] start AUTOMATIC1111 webui. (1st time) and stop it.
* [4] clone configfile
```sh
cd ~/stable-diffusion-webui/
cp ui-config.json ui-config2.json
cp webui.sh webui2.sh
```
* [5] edit ui-config2.json
* [6] start two `webui`s with `&`
```sh
cd ~/stable-diffusion-webui/
./webui.sh --device-id 0 --listen --port 7860 --xformers --ui-settings-file ui-config.json &
./webui2.sh --device-id 1 --listen --port 7861 --xformers --ui-settings-file ui-config2.json &
```
* [7] access the two webui with 2 browser
   * http://127.0.0.1:7860
   * http://127.0.0.1:7861
* [8] change save directory name
   * Settings -> `Saving to a directory` -> change the name of `Directory name pattern` from `[data]` to `0_[date]` for GPU0 / `1_[date]` for GPU1

# Ref. ( &Special thanks)
* https://cloud.vast.ai
* https://github.com/AUTOMATIC1111/stable-diffusion-webui
* https://hub.docker.com/r/nvidia/cuda/tags
* https://pytorch.org/get-started/pytorch-2.0/
* https://github.com/CompVis/stable-diffusion
