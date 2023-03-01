### spcn04 Revert Proxy

จากเดิมที่สร้าง 3 node (manager, worker1, worker2)

**revert Proxy >> ทำให้สามารถใช้ service ( บริการ )ได้ใน port ที่ติดต่อจากภายนอกเข้ามาได้ใน port เดียวกัน(ภายใน) สามารถแชร์ใช้ stcks ร่วมกันได้**

## Trefik Proxy เป็นตัวรับการร้องขอข้อมูลจาก client 
* deploy revert Proxy, swrampit, hello-world

1. Prepare host file (Client)
สิ่งที่ต้องมี ประกอบด้วย
(Update file)
    * portainer
    * tracfik
    * swarmpit
    * lab...(work other)

* edit file for (เมื่อแก้ไขใน file นี้)
- Windows C:\Windows\System32\drivers\etc
- Linux/Mac /etc/

    ```
    
    ```

* Trcefiklabs
Step
    - สร้าง network ที่จะแชร์กับ trcefik (ลงที่ manager)
    ```
    docker network create --driver=overlay tracfik-public
    ``` 
    - สร้าง tag ในไฟล์ใบรับรอง public ที่โหนดนี้
    ```
    export NODE_ID=$(docker info -f '{{.Swarm.NodeID}}')
    echo $NODE_ID
    ```
    ```
    docker node update --label-add tracefix-public.tracefix-public-cerificates(certificates=ture $NODE_IDS)
    ```
    - เพิ่ม lable
    
