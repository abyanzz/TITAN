#!/bin/bash

# Periksa apakah skrip dijalankan sebagai pengguna root
if [ "$(id -u)" != "0" ]; then
    echo "Skrip ini harus dijalankan dengan izin pengguna root."
    echo "Silakan coba gunakan perintah 'sudo -i' untuk beralih ke pengguna root, lalu jalankan skrip ini lagi."
    exit 1
fi

echo "Saya Hanya Translate Source @y95277777"
echo "======================Titan Node============================="

# Set nilai default
id="7512F2B2-CE08-4715-8673-6512F0C8FB60"
container_count=1
storage_gb=5
custom_storage_path=""
start_rpc_port=$((10000 + RANDOM % 10000))

apt update

# Centang jika Docker diinstal 
if ! command -v docker &> /dev/null
then
    echo "Docker tidak terdeteksi, menginstal..."
    apt-get install ca-certificates curl gnupg lsb-release -y
    
    # Instal Docker versi terbaru
    apt-get install docker.io -y
else
    echo "Docker telah diinstal."
fi

# Tarik gambar Docker
docker pull nezha123/titan-edge

# Buat sejumlah kontainer yang ditentukan pengguna
for ((i=1; i<=container_count; i++))
do
    current_rpc_port=$((start_rpc_port + i - 1))

    # Tentukan jalur penyimpanan
    if [ -z "$custom_storage_path" ]; then
        storage_path="$PWD/titan_storage_$i"
    else
        storage_path="$custom_storage_path"
    fi

    # Pastikan jalur penyimpanan ada
    mkdir -p "$storage_path"

    # Jalankan container dan setel kebijakan mulai ulang ke selalu 
    container_id=$(docker run -d --restart always -v "$storage_path:/root/.titanedge/storage" --name "titan$i" --net=host nezha123/titan-edge:1.4)

    echo "Node titan$i telah memulai ID kontainer $container_id"

    sleep 30

    # Ubah file config.toml host untuk mengatur nilai StorageGB dan port
    docker exec $container_id bash -c "\
        sed -i 's/^[[:space:]]*#StorageGB = .*/StorageGB = $storage_gb/' /root/.titanedge/config.toml && \
        sed -i 's/^[[:space:]]*#ListenAddress = \"0.0.0.0:1234\"/ListenAddress = \"0.0.0.0:$current_rpc_port\"/' /root/.titanedge/config.toml && \
        echo 'Ruang penyimpanan titan$i disetel ke $storage_gb GB, RPC disetel ke $current_rpc_port'"

    # Mulai ulang wadah agar pengaturan diterapkan 
    docker restart $container_id

    # Masuk ke container dan lakukan pengikatan Order
    docker exec $container_id bash -c "\
        titan-edge bind --hash=$id https://api-test1.container1.titannet.io/api/v2/device/binding"
    echo "Node titan$i terikat."
done

echo "===========================Semua node telah disiapkan dan dimulai==========================="
