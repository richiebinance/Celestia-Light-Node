![228786938-4112e207-17e4-43c4-bec6-ec93e137df79](https://user-images.githubusercontent.com/82761397/229158545-aa7c5b7e-58b4-4529-a98b-03bdd5242536.jpg)


# Celestia-ITN-Light-Node

# Tutorial Running Light Node Celestia

Celestia Light Node
Light nodes memastikan ketersediaan data. 
Ini adalah cara paling umum untuk berinteraksi dengan jaringan Celestia.

Persyaratan perangkat keras

· Memori: 2 GB RAM

· CPU: Inti tunggal

· Disk: Penyimpanan SSD 5 GB

Perbarui & instal dependensi yang diperlukan
```
sudo apt update && sudo apt upgrade -y

sudo apt install curl tar wget dentang pkg-config libssl-dev jq build-essential git make ncdu -y
```
# Instal GO 1.19.1
ver="1.19.1"
```
cd $HOME

wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"

sudo rm -rf /usr/local/go

sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"

rm "go$ver.linux-amd64.tar.gz"
```
# Tambahkan direktori /usr/local/go/bin ke $PATH:
```
echo "ekspor PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile
source $HOME/.bash_profile
```
# Menginstal node Celestia
```
cd $HOME
rm -rf celestia-node
git clone https://github.com/celestiaorg/celestia-node.git
cd celestia-node/
git checkout tags/v0.6.0
make install
make cel-key
```
# Inisiasi Light Node
```
celestia light init --p2p.network blockspacerace
```
# Simpan Phase
```
./cel-key list --node.type light --p2p.network blockspacerace
```
Buat Service
```
sudo tee <<EOF >/dev/null /etc/systemd/system/celestia-lightd.service
[Unit]
Description=celestia-lightd Light Node
After=network-online.target

[Service]
User=$USER
ExecStart=/usr/local/bin/celestia light start --core.ip https://rpc-blockspacerace.pops.one --core.rpc.port 26657 --core.grpc.port 9090 --keyring.accname my_celes_key --metrics.tls=false --metrics --metrics.endpoint otel.celestia.tools:4318 --gateway --gateway.addr localhost --gateway.port 26659 --p2p.network blockspacerace
Restart=on-failure
RestartSec=3
LimitNOFILE=4096

[Install]
WantedBy=multi-user.target
EOF
```
# Mulai Service Celestia
```
systemctl enable celestia-lightd
systemctl start celestia-lightd
```
# Periksa Log
```
journalctl -u celestia-lightd.service -f
```
# Ambil Node ID celestia
```
AUTH_TOKEN=$(celestia light auth admin --p2p.network blockspacerace)

curl -X POST \
     -H "Otorisasi: Pembawa $AUTH_TOKEN" \
     -H 'Tipe-Konten: aplikasi/json' \
     -d '{"jsonrpc":"2.0","id":0,"method":"p2p.Info","params":[]}' \
     http://localhost:26658
```
ID Anda 12D3xxxxx Simpan

Cek ID di Explorer : https://tiascan.com/light-nodes

# Jika Anda mendapatkan pesan header: not found, silakan coba langkah berikut:
1. hentikan simpul cahayamu
2. buka direktori .celestia-light-blockspacerace-0
3. hapus folder /data, pastikan Anda TIDAK menghapus folder /key
4. init light node Anda lagi
5. start kembali light node

# Hapus Node
```
systemctl stop celestia-lightd
rm -rf /etc/systemd/system/celestia-lightd.service
rm -rf celestia-node
rm -rf .celestia-aplikasi
rm -rf .celestia-light-blockspacerace-0
rm -f $(yang celestia-lightd)
rm -rf $HOME/.celestia-lightd
rm -rf $HOME/celestia-lightd
```
