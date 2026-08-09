# GSP510---Manage-Kubernetes-in-Google-Cloud-Challenge-Lab

GKE & Artifact Registry Lab Cheat Sheet📋 Lab Setup & CredentialsParameterValueUsernamestudent-02-7ecb2c47ff86@qwiklabs.netPasswordbKGEJCOPxQz2Project IDqwiklabs-gcp-04-ed402c1f1a34Cluster Namehello-world-1dh2Regionus-west1Zoneus-west1-aNamespace Namegmp-bhfnRepo Namehello-repoService Namehelloweb-service-huhm🛠️ Environment Variables ConfigurationSalin dan tempel perintah berikut di Cloud Shell untuk menginisialisasi variabel proyek:bashexport PROJECT_ID="qwiklabs-gcp-04-ed402c1f1a34"
export CLUSTER_NAME="hello-world-1dh2"
export REGION="us-west1"
export ZONE="us-west1-a"
export NAMESPACE_NAME="gmp-bhfn"
export REPO_NAME="hello-repo"
export SERVICE_NAME="helloweb-service-huhm"
Gunakan kode dengan hati-hati.🚀 Task 1: Create a GKE ClusterJalankan perintah pembuatan cluster awal berikut:bashgcloud beta container clusters create hello-world-1dh2 \
    --num-nodes=3 \
    --min-nodes 2 \
    --max-nodes 6 \
    --zone $ZONE
Gunakan kode dengan hati-hati.Tunggu sekitar 4-5 menit hingga status berubah menjadi RUNNING.Lakukan pembaruan fitur (Autoscaling & Managed Prometheus) agar lulus penilaian lab:bashgcloud container clusters update hello-world-1dh2 \
    --enable-autoscaling \
    --enable-managed-prometheus \
    --min-nodes=2 \
    --max-nodes=6 \
    --zone=us-west1-a
Gunakan kode dengan hati-hati.📌 Check progress untuk Task 1📦 Task 2: Configure Prometheus MonitoringAmbil kredensial autentikasi cluster:bashgcloud container clusters get-credentials $CLUSTER_NAME --zone $ZONE
Gunakan kode dengan hati-hati.Buat Namespace baru:bashkubectl create ns gmp-bhfn
Gunakan kode dengan hati-hati.Unduh file konfigurasi aplikasi contoh:bashgcloud storage cp gs://spls/gsp510/prometheus-app.yaml .
Gunakan kode dengan hati-hati.Buka Cloud Shell Editor dan perbarui file prometheus-app.yaml pada baris 35-38 (ganti tag <todo>):yamlcontainers:
  - name: prometheus-test
    image: nilebox/prometheus-example-app:latest
    ports:
      - name: metrics
Gunakan kode dengan hati-hati.Deploy aplikasi ke cluster:bashkubectl -n gmp-bhfn apply -f prometheus-app.yaml
Gunakan kode dengan hati-hati.Unduh manifes pemantauan pod:bashgcloud storage cp gs://spls/gsp510/pod-monitoring.yaml .
Gunakan kode dengan hati-hati.Buka Editor dan perbarui file pod-monitoring.yaml pada baris 18-24 (ganti tag <todo>):yamlmetadata:
  name: prometheus-test
spec:
  selector:
    matchLabels:
      app: prometheus-test
  endpoints:
    - interval: 50s
Gunakan kode dengan hati-hati.Terapkan manifes pemantauan pod:bashkubectl -n gmp-bhfn apply -f pod-monitoring.yaml
Gunakan kode dengan hati-hati.📌 Check progress untuk Task 2📂 Task 3: Download Demo ApplicationUnduh repositori aplikasi demo ke direktori lokal:bashgcloud storage cp -r gs://spls/gsp510/hello-app/ .
Gunakan kode dengan hati-hati.Pindah ke direktori manifests:bashcd ~/hello-app/manifests
Gunakan kode dengan hati-hati.Deploy aplikasi demo awal:bashkubectl -n gmp-bhfn apply -f helloweb-deployment.yaml
Gunakan kode dengan hati-hati.📌 Check progress untuk Task 3📊 Task 4: Operations Suite (Logging & Alerting)Part A: Create Log-Based MetricBuka Logs Explorer di Google Cloud Console.Pada panel Resource, pilih Kubernetes Pod -> Pilih Region -> Pilih Cluster hello-world-1dh2 -> Klik Apply.Pada filter tingkat keparahan (Severities), klik All severities lalu ubah ke Warning and higher.Klik tombol Actions di kanan atas -> Pilih Create Metric.Isi konfigurasi log berikut:Metric Type: CounterLog Metric Name: pod-image-errorsKlik Create Metric.Part B: Create Alerting PolicyBuka menu navigasi sebelah kiri -> Log-based Metrics.Cari metrik pod-image-errors, klik ikon titik tiga (...) di sebelah kanan -> Pilih Create alert from metric.Konfigurasikan kebijakan waspada (Alerting Policy):Rolling Window: 10 minRolling window function: countTime series aggregation: sumKlik Next.Condition type: ThresholdAlert trigger: Any time series violatesThreshold position: Above thresholdThreshold value: 0Klik Next.Use notification channel: ⚠️ Matikan / Disable (Uncheck!)Alert policy name: Pod Error AlertKlik Create Policy.📌 Check progress untuk Task 4🔄 Task 5: Update Manifest with Stable ImageBuka file ~/hello-app/manifests/helloweb-deployment.yaml menggunakan Editor.Ganti tag <todo> pada bagian kontainer gambar dengan baris berikut:yamlimage: us-docker.pkg.dev/google-samples/containers/gke/hello-app:1.0
Gunakan kode dengan hati-hati.Hapus deployment lama dari cluster untuk menghindari konflik data:bashkubectl delete deployment helloweb -n gmp-bhfn
Gunakan kode dengan hati-hati.Deploy ulang manifes yang telah diperbarui:bashkubectl -n gmp-bhfn apply -f helloweb-deployment.yaml
Gunakan kode dengan hati-hati.Pastikan Pod baru telah berjalan dengan normal:bashkubectl get pods -n gmp-bhfn
Gunakan kode dengan hati-hati.📌 Check progress untuk Task 5🏷️ Task 6: Containerize and Deploy Version 2.0.0Buka file ~/hello-app/main.go di Editor, pergi ke baris 49, dan ubah teks versinya menjadi:goVersion: 2.0.0
Gunakan kode dengan hati-hati.Jalankan otentikasi Docker untuk Artifact Registry di Cloud Shell:bashgcloud auth configure-docker us-west1-docker.pkg.dev
Gunakan kode dengan hati-hati.Masuk ke direktori utama aplikasi dan lakukan kompilasi (build) image v2:bashcd ~/hello-app
docker build -t us-west1-docker.pkg.dev/$PROJECT_ID/hello-repo/hello-app:v2 .
Gunakan kode dengan hati-hati.Unggah (push) kontainer baru Anda ke Artifact Registry:bashdocker push us-west1-docker.pkg.dev/$PROJECT_ID/hello-repo/hello-app:v2
Gunakan kode dengan hati-hati.Buka kembali file ~/hello-app/manifests/helloweb-deployment.yaml di Editor, ubah baris image ke alamat registri baru Anda:yamlimage: us-west1-docker.pkg.dev/qwiklabs-gcp-04-ed402c1f1a34/hello-repo/hello-app:v2
Gunakan kode dengan hati-hati.Terapkan perubahan manifes ke cluster:bashcd ~/hello-app/manifests
kubectl -n gmp-bhfn apply -f helloweb-deployment.yaml
Gunakan kode dengan hati-hati.Buka akses deployment ke publik dengan membuat service berjenis LoadBalancer:bashkubectl expose deployment helloweb \
    --name=helloweb-service-huhm \
    --type=LoadBalancer \
    --port=8080 \
    --target-port=8080 \
    -n gmp-bhfn
Gunakan kode dengan hati-hati.Periksa status IP publik secara berkala hingga kolom EXTERNAL-IP memunculkan alamat IP:bashkubectl get service helloweb-service-huhm -n gmp-bhfn
Gunakan kode dengan hati-hati.📌 Check progress untuk Task 6 & Selesai!
