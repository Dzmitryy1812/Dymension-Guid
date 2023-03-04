# Dymension-Guid

![demynsion](https://avatars.mds.yandex.net/get-images-cbir/1693524/sihuzT45M8j3i8RLiFz3kA7350/ocr)

Dymension позволяет легко развертывать свои собственные блокчейны, называемые RollApps
Dymension Hub является расчетным уровнем протокола и действует как децентрализованный источник истины для сети. Dymension Hub - это блокчейн Proof of Stake, который обеспечивает консенсус, безопасность и ликвидность для RollApps
[Docs] (https://docs.dymension.xyz/) \
[WEBSITE](https://dymension.xyz/) \
[EXPLORER](https://exp.utsa.tech/dymension-test)
- **Требования**:
-250 GB SSD storage
-Quad core CPU less than 10 years old if self hosting
-Dual core CPU works if hosted with newer Xeons / EPYC
-16 GB of ram,  4+ GB of virtual memory recommended
-Hosting: 8 GB RAM + 8 GB Virtual Memory

# Подача Gentx  
Устанавливаем бинарные файлы 
```bash
git clone https://github.com/dymensionxyz/dymension.git --branch v0.2.0-beta
cd dymension
make install
dymd version --long | grep -e version -e commit
```
 v0.2.0-beta
 commit: 987e33407911c0578251f3ace95d2382be7e661d

Инициализируем ноду, чтобы создать необходимые файлы конфигурации
```bash
dymd init <name_node> --chain-id 35-C
dymd config chain-id 35-C
```
Создаем или восстанавливаем кошелек и сохраняем вывод
```bash
dymd keys add <key-name>
dymd keys add <key-name> --recover
```
Добавляем учетную запись в свой локальный файл genesis с заданной суммой и ключом, которые вы только что создали
```bash
dymd add-genesis-account <address_or_key_name> 600000000000udym --keyring-backend os
```
Создаем Gentx (предварительно меняем под себя к примеру в блокноте)
```bash
dymd gentx <name_wallet> 500000000000udym \
  --chain-id 35-C \
  --commission-rate="0.05" \
  --commission-max-rate="0.10" \
  --commission-max-change-rate="0.01" \
  --pubkey $(dymd tendermint show-validator) \
  --moniker="<moniker-name>" \
  --website=<your-node-website> \
  --details=<your-node-details> \
  --min-self-delegation="1"
  ```

*Не забываем сохранить priv_validator_key

# Новая установка ноды  
Устанавливаем бинарники
```bash
git clone https://github.com/dymensionxyz/dymension && cd dymension
git checkout v0.2.0-beta
make install
dymd version --long | grep -e version -e commit
```
Инициализируем ноду, чтобы создать необходимые файлы конфигурации
```bash
dymd init <name_moniker> --chain-id 35-C
```
Скачиваем Genesis
```bash
wget -O $HOME/.dymension/config/genesis.json "https://raw.githubusercontent.com/dymensionxyz/testnets/main/dymension-hub/35-C/genesis.json"

 Проверим генезис
sha256sum ~/.dymension/config/genesis.json
cf20e3b15d089ceeaaa9bb2abcd48a50f98e9f2274f4320aeae534d6972c4ee2
```

# Настраиваем конфигурацию ноды

 правим конфиг, благодаря чему мы можем больше не использовать флаг chain-id для каждой команды CLI в client.toml
```bash
dymd config chain-id 35-C
```
при необходимости настраиваем keyring-backend в client.toml 
```bash  dymd config keyring-backend os ``` 

настраиваем минимальную цену за газ в app.toml
```bash
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.025udym\"/;" ~/.dymension/config/app.toml
```
 добавляем seeds/bpeers/peers в config.toml
```bash
external_address=$(wget -qO- eth0.me)
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.dymension/config/config.toml
peers="ebc272824924ea1a27ea3183dd0b9ba713494f83@dymension-testnet-peer.autostake.net:27086,9111fd409e5521470b9b33a46009f5e53c646a0d@178.62.81.245:45656,f8a0d7c7db90c53a989e2341746b88433f47f980@209.182.238.30:30657,1bffcd1690806b5796415ff72f02157ce048bcdd@144.76.67.53:2580,c17a4bcba59a0cbb10b91cd2cee0940c610d26ee@95.217.144.107:20556,e6ea3444ac85302c336000ac036f4d86b97b3d3e@38.146.3.199:20556,b473a649e58b49bc62b557e94d35a2c8c0ee9375@95.214.53.46:36656,db0264c412618949ce3a63cb07328d027e433372@146.19.24.101:26646,281190aa44ca82fb47afe60ba1a8902bae469b2a@dymension.peers.stavr.tech:17806,d8b1bcfc123e63b24d0ebf86ab674a0fc5cb3b06@51.159.97.212:26656,55f233c7c4bea21a47d266921ca5fce657f3adf7@168.119.240.200:26656,139340424dddf85e54e0a54179d06875013e1e39@65.109.87.88:24656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.dymension/config/config.toml

bpeers=""
sed -i.bak -e "s/^bootstrap-peers *=.*/bootstrap-peers = \"$bpeers\"/" $HOME/.dymension/config/config.toml

seeds="f97a75fb69d3a5fe893dca7c8d238ccc0bd66a8f@dymension-testnet.seed.brocha.in:30584"
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.dymension/config/config.toml
```bash
# при необходимости увеличиваем количество входящих и исходящих пиров для подключения, за исключением постоянных пиров в config.toml
# может помочь при падении ноды, но увеличивает нагрузку
```bash
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 50/g' $HOME/.dymension/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 25/g' $HOME/.dymension/config/config.toml
```

(ОПЦИОНАЛЬНО) Настраиваем прунинг одной командой вapp.toml

```bash
pruning="custom"
pruning_keep_recent="1000"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.dymension/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.dymension/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.dymension/config/app.toml
```

Создаем сервисный файл
```bash
tee /etc/systemd/system/dymd.service > /dev/null <<EOF
[Unit]
Description=dymd
After=network-online.target

[Service]
User=$USER
ExecStart=$(which dymd) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF

systemctl daemon-reload
systemctl enable dymd
systemctl restart dymd && journalctl -u dymd -f -o cat
```
# Создаем или восстанавливаем кошелек и сохраняем вывод
 создать кошелек
```bash
dymd keys add <name_wallet> --keyring-backend os
```
восстановить кошелек (после команды вставить seed)
```bash
dymd keys add <name_wallet> --recover --keyring-backend os
```
 восстановить кошелек для EVM сетей
```bash
dymd keys add <name_wallet> --recover --coin-type 118 --algo secp256k1
```
# Не забываем сохранить MNEMONIC!
Создаем валидатора 
```bashdymd tx staking create-validator \
--chain-id 35-C \
--commission-rate 0.05 \
--commission-max-rate 0.2 \
--commission-max-change-rate 0.1 \
--min-self-delegation "1000000" \
--amount 1000000udym \
--pubkey $(dymd tendermint show-validator) \
--moniker "<name_moniker>" \
--from <name_wallet> \
--fees 5000udym
dymd tx staking create-validator \
--chain-id 35-C \
--commission-rate 0.05 \
--commission-max-rate 0.2 \
--commission-max-change-rate 0.1 \
--min-self-delegation "1000000" \
--amount 1000000udym \
--pubkey $(dymd tendermint show-validator) \
--moniker "<name_moniker>" \
--from <name_wallet> \
--fees 5000udym
```
проверить блоки
```bash
dymd status 2>&1 | jq ."SyncInfo"."latest_block_height"
```
 проверить логи
```bash
journalctl -u dymd -f -o cat
```
 проверить статус
```bash
curl localhost:26657/status
```
 проверить баланс
```bash
dymd q bank balances <address>
```




