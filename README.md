# Teritori Chain Testnet-2 Türkçe Kurulum Rehberi

![Banner!](assets/banner.png)

[![Discord](https://badgen.net/badge/icon/discord?icon=discord&label)](https://discord.gg/PKd938HbpR)
[![Go Report
Card](https://goreportcard.com/badge/github.com/TERITORI/teritori-chain?style=flat-square)](https://goreportcard.com/report/github.com/TERITORI/teritori-chain)
[![Version](https://img.shields.io/github/tag/TERITORI/teritori-chain.svg?style=flat-square)](https://github.com/TERITORI/teritori-chain/releases/latest)
[![Lines Of
Code](https://img.shields.io/tokei/lines/github/TERITORI/teritori-chain?style=flat-square)](https://github.com/TERITORI/teritori-chain)


## Sistem Gereksinimleri
- 2CPU
- 2GB RAM
- 80GB SSD
- Ubuntu 18.04 LTS ve üzeri.

## Sistemi Güncelleme
```shell
apt update && apt upgrade -y 
```  

## Gerekli Kütüphanelerin Kurulması
```shell
apt install build-essential git curl gcc make jq -y
```

## Go Kurulumu  
```shell
wget -c https://go.dev/dl/go1.18.3.linux-amd64.tar.gz && rm -rf /usr/local/go && tar -C /usr/local -xzf go1.18.3.linux-amd64.tar.gz && rm -rf go1.18.3.linux-amd64.tar.gz
echo 'export GOROOT=/usr/local/go' >> $HOME/.bash_profile
echo 'export GOPATH=$HOME/go' >> $HOME/.bash_profile
echo 'export GO111MODULE=on' >> $HOME/.bash_profile
echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> $HOME/.bash_profile && . $HOME/.bash_profile
go version
```  

Yukarıdaki son kodun çıktısı aşağıdaki gibiyse işlem tamamdır.
```shell
go version go1.18.3 linux/amd64
``` 

## Değişkenleri Yükleme
* Aşağıda değiştirmeniz gereken yerler belirtilmiştir.
 * `$NODENAME` validator adınız
 * `$WALLET` cüzdan adınız
```shell
echo "export NODENAME=$NODENAME"  >> $HOME/.bash_profile
echo "export WALLET=$WALLET" >> $HOME/.bash_profile
echo "export CHAIN_ID=teritori-testnet-v2" >> $HOME/.bash_profile
```

## Teritori Chain Kurulumu 
```shell
git clone https://github.com/TERITORI/teritori-chain && cd teritori-chain && git checkout teritori-testnet-v2 && make install
```  

## Kurulumu Teyit Etme
```shell
teritorid version
```
Yukarıdaki kodun çıktısı aşağıdaki gibiyse kurulum gerçekleşmiştir.
```shell
teritori-testnet-v2-0f4e5cb1d529fa18971664891a9e8e4c114456c6
```  

## Uygulamayı Başlatma
```shell
teritorid init $NODENAME --chain-id $CHAIN_ID
```  

## PEERS Ekleme
```shell
sed -i.bak 's/persistent_peers =.*/persistent_peers = "0b42fd287d3bb0a20230e30d54b4b8facc412c53@176.9.149.15:26656,2371b28f366a61637ac76c2577264f79f0965447@176.9.19.162:26656,2f394edda96be07bf92b0b503d8be13d1b9cc39f@5.9.40.222:26656"/' $HOME/.teritorid/config/config.toml
```  

## Genesis ve Addrbook Dosyalarının İndirilmesi
```shell
wget -O $HOME/.teritorid/config/genesis.json https://raw.githubusercontent.com/TERITORI/teritori-chain/main/testnet/teritori-testnet-v2/genesis.json
wget -O $HOME/.teritorid/config/addrbook.json https://raw.githubusercontent.com/StakeTake/guidecosmos/main/teritori/teritori-testnet-v2/addrbook.json
```  

## Servis Dosyası Oluşturma
```shell
tee <<EOF >/dev/null /etc/systemd/system/teritorid.service
[Unit]
Description=Teritori Cosmos daemon
After=network-online.target

[Service]
User=$USER
ExecStart=/$USER/go/bin/teritorid start
Restart=on-failure
RestartSec=3
LimitNOFILE=4096

[Install]
WantedBy=multi-user.target
EOF
```  

## Servisi Başlatma
```shell
systemctl enable teritorid
systemctl daemon-reload
systemctl restart teritorid
```  

## Logları Kontrol Etme
```shell
journalctl -u teritorid.service -f -n 100
```  

## Cüzdan Oluşturma

### Yeni Cüzdan Oluşturma
`$WALLET` bölümünü değiştirmiyoruz kurulumun başında cüzdanımıza isim belirledik.
```shell 
teritorid keys add $WALLET
```  

### Var Olan Cüzdanı İçeri Aktarma
```shell
teritorid keys add $WALLET --recover
```  

BU AŞAMADAN SONRA NODE'UMUZUN EŞLEŞMESİNİ BEKLİYORUZ.

## Faucet / Musluk
Test token almak için Discord'da [#faucet](https://discord.gg/PKd938HbpR) kanalından şu şekilde `$request CUZDAN_ADRESINIZ` mesaj atıyoruz.

## Cüzdan Bakiyesini Kontrol Etme
```shell
teritorid query bank balances CUZDAN_ADRESINIZ --chain-id $CHAIN_ID
```  

## Senkronizasyonu Kontrol Etme
`false` çıktısı almaldıkça bir sonraki yani validator oluşturma adımına geçmiyoruz.
```shell
teritorid status 2>&1 | jq .SyncInfo
```

## Validator Oluşturma
* Aşağıdaki komutta aşağıda berlittiğim yerler dışında bir değişikli yapmanız gerekmez;
  * identity : buraya `https://keybase.io/` sitesine üye olarak size verilen kimlik numaranızı yazıyorsunuz.
  * details : kendiniz hakkında bilgiler verebilir ya da `Rues Community Supporter` yazabilirsiniz.
  * website : Varsa bir siteniz yazabilirsiniz ya da `https://forum.rues.info` olarak bırakabilirsiniz.
  * security-contact : E-posta adresiniz.
```shell 
teritorid tx staking create-validator \
 --commission-max-change-rate=0.01 \
 --commission-max-rate=0.2 \
 --commission-rate=0.05 \
 --amount 1000000utori \
 --pubkey=$(teritorid tendermint show-validator) \
 --moniker=$NODENAME \
 --chain-id=$CHAIN_ID \
 --details="Rues Community Supporter" \
 --security-contact="E-POSTANIZ" \
 --website="https://forum.rues.info" \
 --identity="XXXX1111XXXX1111" \
 --min-self-delegation=1000000 \
 --from=$WALLET
 ```  

## Validator Linkinizi Paylaşma
Teritori Discord [#role-request](https://discord.gg/PKd938HbpR) kanalından validatorumuze ait [explorer](https://teritori.explorers.guru/Teritori) linkini gönderiyoruz.

## Exproler
https://teritori.explorers.guru/


## DAHA FAZLA SORUNUZ VARSA TERITORI TÜRKİYE TELEGRAM GRUBU:

[Teritori Türkiye Telegram Sayfası](https://t.me/TeritoriTurkish)

## FAYDALI KOMUTLAR

### Logları Kontrol Etme 
```shell
journalctl -fu teritorid -o cat
```

### Sistemi Başlatma
```shell
systemctl start teritorid
```

### Sistemi Durdurma
```shell
systemctl stop teritorid
```

### Sistemi Yeniden Başlatma
```shell
systemctl restart teritorid
```

### Node Senkronizasyon Durumu
```shell
teritorid status 2>&1 | jq .SyncInfo
```

### Validator Bilgileri
```shell
teritorid status 2>&1 | jq .ValidatorInfo
```

### Node Bilgileri
```shell
teritorid status 2>&1 | jq .NodeInfo
```

### Node ID Öğrenme
```shell
teritorid tendermint show-node-id
```

### Node IP Adresini Öğrenme
```shell
curl icanhazip.com
```

### Peer Adresinizi Öğrenme
```shell
echo "$(teritorid tendermint show-node-id)@$(curl ifconfig.me):26656"
```

### Cüzdanların Listesine Bakma
```shell
teritorid keys list
```

### Cüzdanı İçeri Aktarma
```shell
teritorid keys add $WALLET --recover
```

### Cüzdanı Silme
```shell
teritorid keys delete CUZDAN_ADI
```

### Cüzdan Bakiyesine Bakma
```shell
teritorid query bank balances CUZDAN_ADRESI
```

### Bir Cüzdandan Diğer Bir Cüzdana Transfer Yapma
```shell
teritorid tx bank send CUZDAN_ADRESI GONDERILECEK_CUZDAN_ADRESI 100000000utori
```

### Proposal Oylamasına Katılma
```shell
teritorid tx gov vote 1 yes --from $WALLET --chain-id=CHAIN_ID 
```

### Validatore Stake Etme / Delegate Etme
```shell
teritorid tx staking delegate $VALOPER_ADDRESS 100000000utori --from=$WALLET --chain-id=C$HAIN_ID  --gas=auto
```

### Mevcut Validatorden Diğer Validatore Stake Etme / Redelegate Etme
```shell
teritorid tx staking redelegate <MevcutValidatorAdresi> <StakeEdilecekYeniValidatorAdresi> 100000000utori --from=WALLET --chain-id=CHAIN_ID  --gas=auto
```

### Ödülleri Çekme
```shell
teritorid tx distribution withdraw-all-rewards --from=$WALLET --chain-id=CHAIN_ID  --gas=auto
```

### Komisyon Ödüllerini Çekme
```shell
teritorid tx distribution withdraw-rewards VALIDATOR_ADRESI --from=$WALLET --commission --chain-id=CHAIN_ID 
```

### Validator İsmini Değiştirme
```shell
teritorid tx staking edit-validator \
--moniker=YENI_NODE_ADI \
--chain-id=$CHAIN_ID  \
--from=WALLET
```

### Validatoru Jail Durumundan Kurtarma 
```shell
teritorid tx slashing unjail \
  --broadcast-mode=block \
  --from=$WALLET \
  --chain-id=$CHAIN_ID  \
  --gas=auto
```

### Node'u Tamamen Silme 
```shell
sudo systemctl stop teritorid && \
sudo systemctl disable teritorid && \
rm /etc/systemd/system/teritorid.service && \
sudo systemctl daemon-reload && \
cd $HOME && \
rm -rf .teritori teritori && \
rm -rf $(which teritorid)
```


### Hesaplar:

[Linktree](https://linktr.ee/mehmetkoltigin)

[Twitter](https://twitter.commehmetkoltigin)

### Komunite 
[Forum Rues](https://forum.rues.info/index.php)

[Telegram Rues Announcement](https://t.me/RuesAnnouncement)

[Telegram Rues Chat](https://t.me/RuesChat)

[Telegram Rues Node](https://t.me/RuesNode)

[Telegram Rues Node Chat](https://t.me/RuesNodeChat)

