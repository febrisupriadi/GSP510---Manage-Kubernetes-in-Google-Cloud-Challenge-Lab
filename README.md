# GSP510---Manage-Kubernetes-in-Google-Cloud-Challenge-Lab

### GKE & Artifact Registry

### 📋 Lab Setup & Credentials

ParameterValue
****
**Username**: student-02-7ecb2c47ff86@qwiklabs.net
****
**Password**: bKGEJCOPxQz2
****
**Project ID**: qwiklabs-gcp-04-ed402c1f1a34
****
**Cluster Name**: hello-world-1dh2
****
**Region**: us-west1
****
**Zone**: us-west1-a
****
**Namespace Name**: gmp-bhfn
****
**Repo Name**: hello-repo
****
**Service Name**: helloweb-service-huhm

### 🛠️ Environment Variables Configuration

Salin dan tempel perintah berikut di Cloud Shell untuk menginisialisasi variabel proyek: 

    ```bash
    export PROJECT_ID="qwiklabs-gcp-04-ed402c1f1a34"
    export CLUSTER_NAME="hello-world-1dh2"
    export REGION="us-west1"
    export ZONE="us-west1-a"
    export NAMESPACE_NAME="gmp-bhfn"
    export REPO_NAME="hello-repo"
    export SERVICE_NAME="helloweb-service-huhm"
    ```



### 🚀 Task 1: Create a GKE Cluster

1. Jalankan perintah pembuatan cluster awal berikut: 

    ```bash    
    gcloud beta container clusters create hello-world-1dh2 \
        --num-nodes=3 \
        --min-nodes 2 \
        --max-nodes 6 \
        --zone $ZONE
    ```

*Tunggu sekitar 4-5 menit hingga status berubah menjadi RUNNING.*

2. Lakukan pembaruan fitur (Autoscaling & Managed Prometheus) agar lulus penilaian lab: 

bash

gcloud container clusters update hello-world-1dh2 \
    --enable-autoscaling \
    --enable-managed-prometheus \
    --min-nodes=2 \
    --max-nodes=6 \
    --zone=us-west1-a

Gunakan kode dengan hati-hati.

📌 **Check progress untuk Task 1** 

### 📦 Task 2: Configure Prometheus Monitoring

1. Ambil kredensial autentikasi cluster: 

bash

gcloud container clusters get-credentials $CLUSTER_NAME --zone $ZONE

Gunakan kode dengan hati-hati.
2. Buat Namespace baru: 

bash

kubectl create ns gmp-bhfn

Gunakan kode dengan hati-hati.
3. Unduh file konfigurasi aplikasi contoh: 

bash

gcloud storage cp gs://spls/gsp510/prometheus-app.yaml .

Gunakan kode dengan hati-hati.
4. Buka Cloud Shell Editor dan perbarui file prometheus-app.yaml pada baris **35-38** (ganti tag <todo>): 

yaml

containers:
  - name: prometheus-test
    image: nilebox/prometheus-example-app:latest
    ports:
      - name: metrics

Gunakan kode dengan hati-hati.
5. Deploy aplikasi ke cluster: 

bash

kubectl -n gmp-bhfn apply -f prometheus-app.yaml

Gunakan kode dengan hati-hati.
6. Unduh manifes pemantauan pod: 

bash

gcloud storage cp gs://spls/gsp510/pod-monitoring.yaml .

Gunakan kode dengan hati-hati.
7. Buka Editor dan perbarui file pod-monitoring.yaml pada baris **18-24** (ganti tag <todo>): 

yaml

metadata:
  name: prometheus-test
spec:
  selector:
    matchLabels:
      app: prometheus-test
  endpoints:
    - interval: 50s

Gunakan kode dengan hati-hati.
8. Terapkan manifes pemantauan pod: 

bash

kubectl -n gmp-bhfn apply -f pod-monitoring.yaml

Gunakan kode dengan hati-hati.

📌 **Check progress untuk Task 2** 

### 📂 Task 3: Download Demo Application

1. Unduh repositori aplikasi demo ke direktori lokal: 

bash

gcloud storage cp -r gs://spls/gsp510/hello-app/ .

Gunakan kode dengan hati-hati.
2. Pindah ke direktori manifests: 

bash

cd ~/hello-app/manifests

Gunakan kode dengan hati-hati.
3. Deploy aplikasi demo awal: 

bash

kubectl -n gmp-bhfn apply -f helloweb-deployment.yaml

Gunakan kode dengan hati-hati.

📌 **Check progress untuk Task 3** 

### 📊 Task 4: Operations Suite (Logging & Alerting)

### Part A: Create Log-Based Metric

1. Buka **Logs Explorer** di Google Cloud Console.
2. Pada panel Resource, pilih **Kubernetes Pod** -> Pilih Region -> Pilih Cluster hello-world-1dh2 -> Klik **Apply**.
3. Pada filter tingkat keparahan (*Severities*), klik **All severities** lalu ubah ke **Warning and higher**.
4. Klik tombol **Actions** di kanan atas -> Pilih **Create Metric**.
5. Isi konfigurasi log berikut: 

  * **Metric Type**: Counter
  * **Log Metric Name**: pod-image-errors
6. Klik **Create Metric**.

### Part B: Create Alerting Policy

1. Buka menu navigasi sebelah kiri -> **Log-based Metrics**.
2. Cari metrik pod-image-errors, klik ikon **titik tiga (...)** di sebelah kanan -> Pilih **Create alert from metric**.
3. Konfigurasikan kebijakan waspada (*Alerting Policy*): 

  * **Rolling Window**: 10 min
  * **Rolling window function**: count
  * **Time series aggregation**: sum
  * *Klik Next.*
  * **Condition type**: Threshold
  * **Alert trigger**: Any time series violates
  * **Threshold position**: Above threshold
  * **Threshold value**: 0
  * *Klik Next.*
  * **Use notification channel**: ⚠️ **Matikan / Disable (Uncheck!)**
  * **Alert policy name**: Pod Error Alert
4. Klik **Create Policy**.

📌 **Check progress untuk Task 4** 

### 🔄 Task 5: Update Manifest with Stable Image

1. Buka file ~/hello-app/manifests/helloweb-deployment.yaml menggunakan Editor.
2. Ganti tag <todo> pada bagian kontainer gambar dengan baris berikut: 

yaml

image: us-docker.pkg.dev/google-samples/containers/gke/hello-app:1.0

Gunakan kode dengan hati-hati.
3. Hapus deployment lama dari cluster untuk menghindari konflik data: 

bash

kubectl delete deployment helloweb -n gmp-bhfn

Gunakan kode dengan hati-hati.
4. Deploy ulang manifes yang telah diperbarui: 

bash

kubectl -n gmp-bhfn apply -f helloweb-deployment.yaml

Gunakan kode dengan hati-hati.
5. Pastikan Pod baru telah berjalan dengan normal: 

bash

kubectl get pods -n gmp-bhfn

Gunakan kode dengan hati-hati.

📌 **Check progress untuk Task 5** 

### 🏷️ Task 6: Containerize and Deploy Version 2.0.0

1. Buka file ~/hello-app/main.go di Editor, pergi ke **baris 49**, dan ubah teks versinya menjadi: 

go

Version: 2.0.0

Gunakan kode dengan hati-hati.
2. Jalankan otentikasi Docker untuk Artifact Registry di Cloud Shell: 

bash

gcloud auth configure-docker us-west1-docker.pkg.dev

Gunakan kode dengan hati-hati.
3. Masuk ke direktori utama aplikasi dan lakukan kompilasi (*build*) image v2: 

bash

cd ~/hello-app
docker build -t us-west1-docker.pkg.dev/$PROJECT_ID/hello-repo/hello-app:v2 .

Gunakan kode dengan hati-hati.
4. Unggah (*push*) kontainer baru Anda ke Artifact Registry: 

bash

docker push us-west1-docker.pkg.dev/$PROJECT_ID/hello-repo/hello-app:v2

Gunakan kode dengan hati-hati.
5. Buka kembali file ~/hello-app/manifests/helloweb-deployment.yaml di Editor, ubah baris image ke alamat registri baru Anda: 

yaml

image: us-west1-docker.pkg.dev/qwiklabs-gcp-04-ed402c1f1a34/hello-repo/hello-app:v2

Gunakan kode dengan hati-hati.
6. Terapkan perubahan manifes ke cluster: 

bash

cd ~/hello-app/manifests
kubectl -n gmp-bhfn apply -f helloweb-deployment.yaml

Gunakan kode dengan hati-hati.
7. Buka akses deployment ke publik dengan membuat service berjenis LoadBalancer: 

bash

kubectl expose deployment helloweb \
    --name=helloweb-service-huhm \
    --type=LoadBalancer \
    --port=8080 \
    --target-port=8080 \
    -n gmp-bhfn

Gunakan kode dengan hati-hati.
8. Periksa status IP publik secara berkala hingga kolom EXTERNAL-IP memunculkan alamat IP: 

bash

kubectl get service helloweb-service-huhm -n gmp-bhfn

Gunakan kode dengan hati-hati.

📌 **Check progress untuk Task 6 & Selesai!**
