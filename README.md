# lern-promql
lerning promql query

การใช้งาน **PromQL (Prometheus Query Language)** แบบทีละขั้นตอน โดยใช้ตัวอย่างจริง และค่อย ๆ ขยายจากพื้นฐานไปสู่การใช้งานที่ซับซ้อนขึ้น

---

## 🔹 ขั้นตอนที่ 1: เข้าใจ Concept ของ PromQL

PromQL เอาไว้ใช้ **query metrics** จาก Prometheus เช่น CPU, Memory, Network, Application Metrics
ใน Prometheus ข้อมูลจะแบ่งเป็น 2 อย่างหลัก ๆ:

1. **Metric Name** → ชื่อ metric เช่น `node_cpu_seconds_total`
2. **Labels** → key=value ที่บอก context เช่น `{mode="idle", instance="server01"}`

---

## 🔹 ขั้นตอนที่ 2: Query พื้นฐาน

### 2.1 ดึงค่า metric ตรง ๆ

```promql
node_cpu_seconds_total
```

* ได้ค่าของทุก instance + ทุก mode (user, system, idle ฯลฯ)

### 2.2 Filter ด้วย label

```promql
node_cpu_seconds_total{mode="idle", instance="server01:9100"}
```

* เลือกเฉพาะ **instance server01** ที่เป็น **idle mode**

---

## 🔹 ขั้นตอนที่ 3: ใช้ Aggregation

### 3.1 รวมค่าด้วย `sum`

```promql
sum(node_cpu_seconds_total) by (mode)
```

* รวมค่า CPU ของทุก instance แล้วแยกตาม mode

### 3.2 หาค่าเฉลี่ยด้วย `avg`

```promql
avg(node_load1) by (instance)
```

* หาค่า load average (1 นาที) ของแต่ละ instance

---

## 🔹 ขั้นตอนที่ 4: คำนวณ Rate (สำคัญมาก)

Metric ที่เป็น **counter** ต้องใช้ `rate()` เพื่อดูการเปลี่ยนแปลงต่อเวลา

```promql
rate(node_cpu_seconds_total{mode!="idle"}[5m])
```

* ดูการเปลี่ยนแปลงของ CPU ที่ไม่ใช่ idle ในช่วง 5 นาทีล่าสุด

---

## 🔹 ขั้นตอนที่ 5: รวมหลาย Instance

```promql
sum(rate(node_cpu_seconds_total{mode!="idle"}[5m])) by (instance)
```

* แสดง CPU Usage (ที่ไม่ idle) ของแต่ละ instance

---

## 🔹 ขั้นตอนที่ 6: คิด % การใช้งาน CPU

```promql
100 - (avg by (instance)(rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)
```

* คำนวณ CPU Usage เป็นเปอร์เซ็นต์

---

## 🔹 ขั้นตอนที่ 7: Memory Usage

```promql
(node_memory_MemTotal_bytes - node_memory_MemAvailable_bytes) 
/ node_memory_MemTotal_bytes * 100
```

* ดู % การใช้งาน Memory

---

## 🔹 ขั้นตอนที่ 8: Network Traffic

```promql
rate(node_network_receive_bytes_total[5m])
```

* ดู incoming traffic (bytes/sec)

```promql
rate(node_network_transmit_bytes_total[5m])
```

* ดู outgoing traffic (bytes/sec)

---

## 🔹 ขั้นตอนที่ 9: Combine Metrics

```promql
(sum by (instance)(rate(node_network_receive_bytes_total[5m]))) 
+ (sum by (instance)(rate(node_network_transmit_bytes_total[5m])))
```

* รวม incoming + outgoing traffic ต่อ instance

---

## 🔹 ขั้นตอนที่ 10: ใช้ใน Grafana

* PromQL ใช้เป็น datasource query ใน **Grafana panel**
* สามารถใส่ `legend` เพื่อเปลี่ยนชื่อ series เช่น:

```promql
sum(rate(node_cpu_seconds_total{mode!="idle"}[5m])) by (instance)
```

แล้วตั้ง legend เป็น `CPU - {{instance}}`

---

👉 แนะนำว่าเราควรเริ่มจาก **metric พื้นฐาน** (CPU, Memory, Disk, Network) แล้วค่อย ๆ เล่นกับ `rate()`, `sum()`, `avg()`, `by()` เพื่อเข้าใจ pattern ก่อน

---

คุณอยากให้ผมสอนแบบ **Workshop** เลยมั้ยครับ? เช่น เริ่มจาก query CPU → Memory → Disk → Network แล้วทำให้เห็นเป็น Dashboard ใน Grafana ทีละขั้นตอน?
