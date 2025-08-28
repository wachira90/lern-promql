# Step by Step ใน Grafana UI 

สร้าง Dashboard ขึ้นมาใหม่ แล้วเพิ่ม Panel

---

# 🟢 Workshop: สร้าง Grafana Dashboard ทีละ Panel

## Step 0: เข้า Grafana

1. Login เข้า Grafana
2. ไปที่เมนู **Dashboards → New → New Dashboard**
3. กด **Add a new panel**

---

## Step 1: สร้าง CPU Usage Panel

1. เลือก **Prometheus datasource**
2. ใส่ Query:

   ```promql
   100 - (avg by (instance)(rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)
   ```
3. เลือก Visualization → **Gauge** หรือ **Time series**
4. ที่ **Legend** ใส่:

   ```
   CPU - {{instance}}
   ```
5. ตั้งชื่อ Panel: **CPU Usage**

---

## Step 2: สร้าง Memory Usage Panel

1. กด **Add Panel**
2. ใส่ Query:

   ```promql
   (node_memory_MemTotal_bytes - node_memory_MemAvailable_bytes) 
   / node_memory_MemTotal_bytes * 100
   ```
3. Visualization → **Gauge**
4. Legend:

   ```
   Memory - {{instance}}
   ```
5. ชื่อ Panel: **Memory Usage**

---

## Step 3: สร้าง Disk Usage Panel

1. กด **Add Panel**
2. Query:

   ```promql
   (node_filesystem_size_bytes{fstype!="tmpfs",fstype!="overlay"} - node_filesystem_free_bytes{fstype!="tmpfs",fstype!="overlay"})
   / node_filesystem_size_bytes{fstype!="tmpfs",fstype!="overlay"} * 100
   ```
3. Visualization → **Bar Gauge** หรือ **Table**
4. Legend:

   ```
   Disk - {{instance}} {{mountpoint}}
   ```
5. ชื่อ Panel: **Disk Usage**

---

## Step 4: สร้าง Network Traffic Panel

### Incoming

1. Add Panel
2. Query:

   ```promql
   rate(node_network_receive_bytes_total{device!="lo"}[5m])
   ```
3. Visualization → **Time series**
4. Legend:

   ```
   RX - {{instance}} {{device}}
   ```
5. Panel Title: **Network In**

### Outgoing

1. Duplicate Panel (ขวาบน → Duplicate)
2. Query:

   ```promql
   rate(node_network_transmit_bytes_total{device!="lo"}[5m])
   ```
3. Legend:

   ```
   TX - {{instance}} {{device}}
   ```
4. Panel Title: **Network Out**

---

## Step 5: จัด Layout Dashboard

* แบ่งเป็น **Rows**

  * Row 1 → CPU & Memory
  * Row 2 → Disk
  * Row 3 → Network In/Out
* Save Dashboard ตั้งชื่อว่า `Node Exporter Overview`

---

## Step 6: ใส่ Thresholds

ในแต่ละ Panel เราสามารถใส่สีเตือน เช่น:

* CPU > 80% = สีแดง
* Memory > 80% = สีส้ม
* Disk > 85% = สีแดง
* Network traffic → แค่ดู trend ก็พอ

---

## Step 7: เพิ่ม Alert Rule (ถ้าต้องการ)

ตัวอย่าง Alert CPU > 80%:

```promql
100 - (avg by (instance)(rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) > 80
```

ใน Grafana:

* ไปที่ Panel → Alert → Add Alert Rule
* Condition: เมื่อค่าเกิน 80% → Alert
* ส่ง Alert ไป Slack/Email/LINE ได้


