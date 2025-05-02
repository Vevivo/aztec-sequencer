# Aztec Sequencer Node Kurulum Rehberi (Türkçe)

Bu rehber, Aztec Network Testnet üzerinde Sequencer Node kurulumunu baştan sona adım adım anlatır. Özellikle teknik bilgisi olmayan kullanıcılar için sadeleştirilmiş ve doğrulanmış komutlarla hazırlanmıştır.

## Sistem Gereksinimleri

### Donanım:
- **İşlemci:** 8 çekirdek
- **RAM:** 16 GB
- **Disk:** 100+ GB SSD

> VPS kullanıyorsanız minimum: 4 çekirdek CPU, 8GB RAM önerilir.

### Yazılım:
- Ubuntu 20.04 veya üzeri
- Docker yüklü olmalı (kurulum adımda anlatılacak)

## 1. Sistem Güncellemeleri ve Temel Paketler

```bash
sudo apt-get update && sudo apt-get upgrade -y
sudo apt install curl iptables build-essential git wget lz4 jq make gcc nano automake autoconf tmux htop nvme-cli libgbm1 pkg-config libssl-dev libleveldb-dev tar clang bsdmainutils ncdu unzip libleveldb-dev -y
```

## 2. Docker Kurulumu

```bash
for pkg in docker.io docker-doc docker-compose podman-docker containerd runc; do sudo apt-get remove $pkg; done
sudo apt-get update
sudo apt-get install ca-certificates curl gnupg -y
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y

# Docker kontrolü:
sudo docker run hello-world
sudo systemctl enable docker
sudo systemctl restart docker
```

## 3. Aztec Araçlarını Kur

```bash
bash -i <(curl -s https://install.aztec.network)
```
Kurulumdan sonra terminali kapatıp yeniden açın:
```bash
aztec
```

## 4. Testnet Güncellemesi (alpha-testnet)

```bash
aztec-up alpha-testnet
```

## 5. RPC ve Beacon URL'lerini Ayarla

Alchemy veya Chainstack gibi hizmetlerden alınabilir. Örnek:
- **RPC URL:** `https://eth-sepolia.g.alchemy.com/v2/YOUR_KEY`
- **BEACON URL:** `https://ethereum-sepolia.core.chainstack.com/beacon/YOUR_ID`

## 6. Cüzdan Oluştur

Gerekenler:
- Public Address
- Private Key

## 7. IP Adresini Al

```bash
curl ipv4.icanhazip.com
```

## 8. Güvenlik Duvarı Ayarları

```bash
sudo ufw allow 22
sudo ufw allow ssh
sudo ufw allow 40400
sudo ufw allow 8080
sudo ufw enable
```

## 9. Sequencer Node’u Başlat

```bash
screen -S aztec
```

```bash
aztec start --node --archiver --sequencer \
  --network alpha-testnet \
  --l1-rpc-urls https://eth-sepolia.g.alchemy.com/v2/YOUR_KEY \
  --l1-consensus-host-urls https://ethereum-sepolia.core.chainstack.com/beacon/YOUR_ID \
  --sequencer.validatorPrivateKey 0xPRIVATE_KEY \
  --sequencer.coinbase 0xPUBLIC_ADDRESS \
  --p2p.p2pIp SUNUCU_IP
```

Screen'den çıkmak için: `Ctrl + A` ardından `D`

## 10. Blok Numarası ve Discord Rolü

Blok numarası al:
```bash
curl -s -X POST -H 'Content-Type: application/json' \
-d '{"jsonrpc":"2.0","method":"node_getL2Tips","params":[],"id":67}' \
http://localhost:8080 | jq -r ".result.proven.number"
```

Sync proof oluştur:
```bash
curl -s -X POST -H 'Content-Type: application/json' \
-d '{"jsonrpc":"2.0","method":"node_getArchiveSiblingPath","params":["BLOCK_NUMBER","BLOCK_NUMBER"],"id":67}' \
http://localhost:8080 | jq -r ".result"
```

Discord'a gir: https://discord.gg/aztec  
Kanal: `#operators | start-here`  
Komut: `/operator start`

## 11. Validator Kaydı (opsiyonel)

```bash
aztec add-l1-validator \
  --l1-rpc-urls https://eth-sepolia.g.alchemy.com/v2/YOUR_KEY \
  --private-key 0xPRIVATE_KEY \
  --attester 0xPUBLIC_ADDRESS \
  --proposer-eoa 0xPUBLIC_ADDRESS \
  --staking-asset-handler 0xF739D03e98e23A7B65940848aBA8921fF3bAc4b2 \
  --l1-chain-id 11155111
```

## Lisans

MIT

## Hazırlayan

@senin-github-kullanıcın
