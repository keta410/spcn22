# spcn22
 About Dokcer and other

## spcn-pre lab
1. สร้าง vm/ct บน proxmox => เลือก OS Ubuntu 22.04

**ct => container

**vm => vistual machine

2. user-password (กรอกลงหน้า log in)
    * check ip =>``` ip a``` จากนั้น remote ผ่าน vs code แล้วทำการติดตั้ง docker, github, wakatime (เป็น Extenstion)
    * ปรับ timezone => ```timedatectl set-timezone Asia/Bangkok```
    * update package => apt update ; apt upgrade -y
3. ติดตั้ง Docker Engine บน Ubuntu
    ```
    apt-get update
    apt-get install \
        ca-certificates \
        curl \
        gnupg \
        lsb-release -y
    ```
    ```
    mkdir -m 0755 -p /etc/apt/keyrings
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | gpg --dearmor -o /etc/apt/keyrings/docker.gpg
    ```

    ```
    echo \
    "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
    $(lsb_release -cs) stable" |  tee /etc/apt/sources.list.d/docker.list > /dev/null
    
    ```
    ```
    apt-get update
    apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
    ```

หลังติดตั้งส่วนที่เป็น **Docker Engine** ให้ทำการ ```convert to template``` แล้วค่อย ```clone form template``` ตามลำดับ
//ทำเป็น Master Template

## spcn-03  
สร้าง vm 3 ตัวจาก vm master [สร้างเป็น manager, worker1, worker2]

ในการที่จะทำ swarm นั้น จะไม่สามารใช้ ct(container) ได้ เนื่องจากไม่สามารถใช้ฟังก์ชั่นบางตัวหรือพื้นที่เก็บข้อมูลอาจจะยังไม่พอ เลยต้องใช้ vm(virtual machine)

1. **Docker Swarm Rocks**
* การเตรียม node ใน ubuntu (3 node)
    ```
    sudo -i  //เข้าใช้งานสิทธิ์เป็น root
    hostnamectl set-hostname [name-change]  // name > spcn22-manager, spcn22-worker1, spcn22-worker2
    (ต้องทำการเปลี่ยน hostname ก่อน join ไม่งั้นเมื่อติดตั้ง swarm docker เข้าไปจะทำให้ตัว swarm เข้าใจว่าเป็น node ที่เข้ามาเพิ่มใหม่)
    ```

2. Clone to template

* reset machine-id (clone มาจาก master) เพราะเลข ip จะซ้ำกับตัวหลัก ทำให้เหมือนการทำงานชนกันเมื่อเรียกใช้งานตัว clone
    ```
    cp /dev/null /etc/machine-id
    rm /var/lib/dbus/machine-id
    ln -s /etc/machine-id /var/lib/dbus/machine-id
    init 0
    ```
* กำหนดสิทธิ์การใช้งาน user ให้สามารถใช้งาน docker ได้ (3 node)
    ```
    sudo usermod -aG docker $USER   //กำหนดการใช้งาน docker โดย root
    docker ps      //ทดสอบเรียกใช้งาน docker

    **ต้องสร้าง group ของ docker ก่อนหรือเปล่า??     รอบ 1 ไม่สามารถ docker ps ได้
    **ทดสอบ reboot สามารถใช้งานได้    รอบ 2
    ```

**Manager 1 node**

    ```
    docker swarm init
    (จากนั้นจะได้ url token มา แล้วเอาไป run บน Worker Node)

    ex. token
    docker swarm join --token (..token Manager Node..)
    ```

**Worker 2 node**

    ```
    docker swarm join (copy token form Manager Node) 172.31.1.(แล้วแต่เลข ip)
    ```

2. Set Up a New Portainer CE(on Linux) on **Manager Node**
    เป็นการติดตั้งผ่านไฟล์ .yml (Download file) จากนั้นนำมา run บน Manager Node

    ```
    curl -L https://downloads.portainer.io/ce2-17/portainer-agent-stack.yml -o portainer-agent-stack.yml
    docker stack deploy -c portainer-agent-stack.yml portainer  //อัพขึ้น Stack ชื่อ portainer
    ```  
จากนั้นลอง docker ps เป็นการตรวจสอบและนำ port ที่กำหนดไว้ภายใน docker-compose.yml มาทดลองใช้งานเว็บไซต์

### ส่วนที่เป็นเกี่ยวกับ Portainer
3. Website on school ทดลองเข้า portainer มาทดลองใช้
    
    3.1) กำหนดให้หน้า Environments => กรอก id ของ host นี้

    3.2) คลิก Swarm > คลิก Registries > (คลิก docker Hub ก่อนกด..) add registries 
//เพิ่มเติม เมื่อคลิก Swarm > clust   er visualizer จะแสดงสถานะ state แต่ละ node(3)


    Ex. 
    รายละเอียดเกี่ยวการตั้งค่าในหน้านี้ Create registry    
    
        Name : keta
        DockerHub username : docker-keta
        DockerHub access token : (กรอกข้อมูล token ของ keta, อิงจาก vdo ให้กรอกเป็น password ของ github โดยตรง)

จากนั้นกด Add registry => มีการเช็ค Stack, Service, Network และ Volumes

4. ทดลอง Deploy hello-world-noProxy

    4.1) ใช้ผ่าน GUI คลิกที่ Stacks บน Portainer ให้คลิก Add stack

        #สิ่งที่ต้องกำหนด#
        Name : (ตามที่ผู้ใช้ตั้ง)
        Web editor : (ข้อมูลนำมาจาก Stack ผ่าน Terminal )        
    
    
    จากนั้นคลิก deploy in progress..

    4.2) หน้า Stacks list ให้คลิกเลือกตัว stacks ที่พึ่งสร้างเสร็จ
    * การเข้าถึงของ os (ubuntu)
        - Add stack> กรอกชื่อ *e.g. mystack*
        - Web editor> *copy code for stcks you want to add in your stack*
        - คลิก Deploy stacks(ถ้าไม่มีอะไรผิดพลาด)
    * การเข้าถึง os เมื่อเรา deploy มาแล้ว
        - คลิก Stcks> *name of stack(ที่เราตั้งไว้ เช่น ubuntu-test etc.)*> เลื่อนมาตรง service แล้วคลิกชื่อ serrvice ที่มี> คลิก Exsc Console> คลิก Connect

Ref: 
>(https://github.com/pitimon/dockerswarm-inhoure/tree/main/ep03-hello-world-noRevProxy)

 