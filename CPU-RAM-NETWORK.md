# 🔥 Workshop: จาก Prometheus → PromQL → Grafana Dashboard

## 🟢 Step 1: เตรียม Metric ที่เราจะใช้

ถ้าเรามี `node_exporter` ติดตั้งไว้แล้ว Metric ที่สำคัญคือ:

* **CPU** → `node_cpu_seconds_total`
* **Memory** → `node_memory_MemTotal_bytes`, `node_memory_MemAvailable_bytes`
* **Disk** → `node_filesystem_size_bytes`, `node_filesystem_free_bytes`
* **Network** → `node_network_receive_bytes_total`, `node_network_transmit_bytes_total`

---

## 🟢 Step 2: Query CPU Usage

เริ่มจาก **CPU Usage ต่อ instance**

```promql
100 - (avg by (instance)(rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)
```

➡ ได้ค่า % CPU usage ของแต่ละ instance

---

## 🟢 Step 3: Query Memory Usage

```promql
(node_memory_MemTotal_bytes - node_memory_MemAvailable_bytes) 
/ node_memory_MemTotal_bytes * 100
```

➡ ได้ค่า % Memory Usage ต่อ instance

---

## 🟢 Step 4: Query Disk Usage

```promql
(node_filesystem_size_bytes{fstype!="tmpfs"} - node_filesystem_free_bytes{fstype!="tmpfs"})
/ node_filesystem_size_bytes{fstype!="tmpfs"} * 100
```

➡ ได้ค่า % Disk Usage (ไม่เอา tmpfs, overlay ที่ไม่ใช่ physical disk)

---

## 🟢 Step 5: Query Network Traffic

### Incoming (Receive)

```promql
rate(node_network_receive_bytes_total{device!="lo"}[5m])
```

### Outgoing (Transmit)

```promql
rate(node_network_transmit_bytes_total{device!="lo"}[5m])
```

➡ ได้ค่า bytes/sec ของ network (ตัด loopback `lo` ทิ้ง)

---

## 🟢 Step 6: ไปที่ Grafana → สร้าง Dashboard

1. เข้า Grafana → **Dashboards → New → Add Panel**
2. เลือก **Prometheus datasource**
3. ใส่ Query ที่เราทำในแต่ละ Step
4. ตั้งชื่อ Legend ให้สวย เช่น:

   ```promql
   100 - (avg by (instance)(rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)
   ```

   ➡ Legend = `CPU Usage - {{instance}}`

---

## 🟢 Step 7: สร้าง Panel พื้นฐาน

* Panel 1: CPU Usage (Graph / Gauge)
* Panel 2: Memory Usage (Gauge)
* Panel 3: Disk Usage (Bar)
* Panel 4: Network Traffic (Line, Incoming vs Outgoing)

---

## 🟢 Step 8: รวมเป็น Dashboard

* **Row 1** → CPU & Memory
* **Row 2** → Disk
* **Row 3** → Network

เวลาเปิด Dashboard เราจะเห็นภาพรวมเครื่องทั้งหมดแบบ Real-time

---

👉 หลังจากนี้คุณสามารถเพิ่ม Alert ได้ เช่น:

```promql
100 - (avg by (instance)(rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) > 80
```

➡ ตั้ง Alert ว่า CPU เกิน 80%


