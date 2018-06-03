# Работа с блокчейном биткойна

Основной справочник по командам bitcoin:
https://en.bitcoin.it/wiki/Original_Bitcoin_client/API_calls_list

## Работа с локальным тестнетом - создание и отправка транзакций.

Для Windows описана работа через Git Bash.


Запустить демон:
#### Для Windows:
```
"c:\Program Files (x86)\Bitcoin\daemon\bitcoind.exe" -debug=1 -printtoconsole -regtest=1 -rpcport=8332 -rpcpassword=pass -listenonion=0
```

#### Для Linux/Mac:
```
bitcoind -debug=1 -printtoconsole -regtest=1 -rpcport=8332 -rpcpassword=pass -listenonion=0
```

#### Для Windows:
Создать файл btcclient.bat (чтобы каждый раз не набирать ключи командной строки) с кодом:
```
@echo off
"c:\Program Files (x86)\Bitcoin\daemon\bitcoin-cli.exe" -regtest=1 -rpcport=8332 -rpcpassword=pass %1 %2 %3 %4 %5
```
Для Linux/Mac создать alias:

`alias btcclient='bitcoin-cli -regtest=1 -rpcport=8332 -rpcpassword=pass'`

Далее выполнение команд осуществлять следующим образом:
Для Windows (находять в папке с созданным файлом btcclient.bat):

`./btcclient.bat <текст команды>`

Для Linux/Mac:

`btcclient <текст команды>`

Проверить баланс текущего аккаунта:

`./btcclient.bat getbalance`

пока 0.


Выведем список аккаунтов

`./btcclient.bat listaccounts`

Видим, что есть аккаунт "по-умолчанию" с пустым названием "".

Сгенерируем первый блок в тестовом блокчейне

`./btcclient.bat generate 1`


Проверим список транзакций:

`./btcclient.bat listtransactions`


Видим, что всего есть 1 транзакция создания 50 BTC (coinbase-транзакция).

Скопируем значения поля "txid" **&lt;txid&gt;** для использования в дальшейнем в примере (будет переводить эти 50 BTC).

*Пример: 56d8a69a250846b9acea9b4c7a5f9eed9d3ef4a254ba35247dbb75ad872695f4*

Еще раз проверить баланс текущего аккаунта:

`./btcclient.bat getbalance`

Все еще 0. Дело в том, что трананзакция создания 50 BTC еще недостаточно "старая".


В информации о кошельке видим immature balance:

`./btcclient.bat getwalletinfo`

В выводе видим, "immature_balance": 50.00000000


Создаем новый аккаунт и адрес в нем (будем на него переводить средства)

`./btcclient.bat getnewaddress "account2"`

Записываем адрес **&lt;address2&gt;**

*Пример: 2NFk5tWmRTvwo3gv9x2ENMB23ewPNR4aiG2*


Создаем для аккаунта аккаунт "по-умолчанию" адрес для "сдачи":

`./btcclient.bat getnewaddress ""`

Записываем **&lt;address3&gt;**

*Пример: 2MvMvWTADM7krpbG6BeZ4D1JgXKoNA1MKGe*


Проверим привязку адресов к аккаунтам:

`./btcclient.bat getaddressesbyaccount ""`

и

`./btcclient.batgetaddressesbyaccount "account2"`


Выводим список аккаунтов

`./btcclient.bat listaccounts`

На аккаунте "" (аккаунт "по-умолчанию") 0 BTC

На аккаунте "account2" 0 BTC


Сгенерируем еще 100 блоков, чтобы разблокировать средства в первом блоке (не меньше, см. https://bitcoin.org/en/developer-examples#regtest-mode)

`./btcclient.bat generate 100`


Выводим список аккаунтов

`./btcclient.bat listaccounts`

На аккаунте "" (аккаунт "по-умолчанию") 50 BTC

На аккаунте "account2" 0 BTC


Сделаем перевод между адресами аккаунта по-умолчанию и аккаунта 2.

Создаем транзакцию перевода 10 BTC с непотраченного output coinbase-транзакции самого первого блока на **&lt;address2&gt;** со сдачей на **&lt;address3&gt;**:

`./btcclient.bat createrawtransaction "[{\"txid\": \"<txid>\",\"vout\":0}]" "{\"<address2>\": 10, \"<address3>\": 39.9999 }"`

Получаем hex транзакции **&lt;raw transaction hex&gt;**

*Пример: 0200000001f4952687ad75bb7d2435ba54a2f43e9ded9e5f7a4c9beaacb94608259aa6d8560000000000ffffffff0200ca9a3b0000000017a914f6c811abd649d89dc6b857a8bbc17731ea4815c887f0006bee0000000017a914222cb297c0fc1dfcc94ab94bf6714dc83fafbe538700000000*

*Пример*
```
./btcclient.bat createrawtransaction "[{\"txid\": \"4fa6b0361b416dd1cb3f468975100e5a3553984d6d6016a0910def3049c3820f\",\"vout\":0}]" "{\"2MznUekxqCqGEKc5dbBmJ22d3vkUphb6FE8\": 10, \"2NDoToJGp2PWnqscBc7z2tpxm6drP1TDZva\": 39.9999 }"
```


Подписываем транзакцию:

`./btcclient.bat signrawtransaction <raw transaction hex>`

Получаем **&lt;signed raw transaction hex&gt;**

Ожидаем "complete": true

*Пример: 0200000001f4952687ad75bb7d2435ba54a2f43e9ded9e5f7a4c9beaacb94608259aa6d8560000000049483045022100ad6d8d4ff8cfe8d0507af9e6f72298ec3f17520789b8f85ff481b6f46f2d6dcb0220449bfdbaff8b41da13608f1ff2ecda72ec58a8b24e369f811f38590a004749b701ffffffff0200ca9a3b0000000017a914f6c811abd649d89dc6b857a8bbc17731ea4815c887f0006bee0000000017a914222cb297c0fc1dfcc94ab94bf6714dc83fafbe538700000000*


Проверяем, что транзакции нет в мемпуле (она еще не отправлена):

`./btcclient.bat getrawmempool`

Получаем: пустой массив (мемпул пуст)


Отправляем транзакцию:

`./btcclient.bat sendrawtransaction <signed raw transaction hex>`

В случае успеха выводится хеш нашей транзакции перевода  **&lt;transaction hash&gt;**

*Пример: 32e9c13d47ee939fabb954ea1df76b2463ca6e9b1679a13b848683d656c239f5*


Можем посмотреть содержание транзакции:

`./btcclient.bat gettransaction <transaction hash>`


Проверяем, что транзакция появилась в мемпуле (она еще не включена в блок):

`./btcclient.bat getrawmempool`

В пуле появилась транзакция **&lt;transaction hash&gt;**.


Генерируем еще 1 блок, чтобы транзакция из mempool попала в блок. При этом из состава immature в состав mature передет еще несколько блоков, которые пополнят баланс первого аккаунта "":

`./btcclient.bat generate 1`


Проверяем mempool:

`./btcclient.bat getrawmempool`

Транзакций нет, так как наша транзакция перевода замайнена (включена в блок).


Смотрим балансы:

`./btcclient.bat listaccounts`

Видим, что на аккаунте по-умолчанию 89.9999 (50 было из блока №1, минус 10 перевод, плюс 50 из блока №2 перешло в категорию "mature" и теперь отображается в балансе).
На аккаунте 2 теперь 10 BTC.


Для сброса всего тестового блокчейна достаточно удалить папку regtest и запустить блокчейн заново

Windows: `C:\Users\<username>\AppData\Roaming\Bitcoin\regtest`

Linux: `/home/<username>/.bitcoin/regtest`

Mac OSX: `/Users/<username>/Library/Application Support/Bitcoin/regtest`               



Дополнительные материалы:

https://en.bitcoin.it/wiki/Script

https://en.bitcoin.it/wiki/Contract

http://siminchen.github.io/bitcoinIDE/build/editor.html

https://bitcoin.stackexchange.com/questions/30543/can-i-transfer-bitcoins-generated-in-regtest-mode-to-a-friend


### Задание 1
1.1. Объясните, почему при вызове getbalance отображается 99.99990000 BTC? Где еще 0.0001 BTC?

1.2. Что будет с балансами аккаунтов при выполнении generate 1 (создании еще одного блока)?


---
## Запись произвольных данных в блокчейн биткойна

### Разворачиваем ноду тестнета блокчейна
Для тестирования используем публичный тестнет биткойна.

Для этого требуется подключиться и синхронизировать ноду с тестовым блокчейном (около 14 Gb). Для синхронизации необходимо в каталоге пользователя создать файл bitcoin.conf (расположение файла на разных платформах указано тут https://bitcoin.stackexchange.com/a/11210) со следующим содержимым (**&lt;имя_пользователя&gt;** и **&lt;пароль&gt;** вы выбираете самостоятельно):
```
testnet=1
server=1
gen=0
rpcuser=<имя_пользователя>
rpcpassword=<пароль>
rpcport=18332
```

Так же в файл можно добавить параметр
```
datadir=<путь_к_папке_с_блокчейном>
```
В которую будет загружен блокчейн тестнета (14Gb+). Настоятельно рекомендуется использовать SSD, чтобы сократить время синхронизации.

Далее запустить синхронизацию командой 

`bitcoind -dbcache=1024 -printtoconsole -onlynet=ipv4`

**** Hint ****
Можно использовать команду
`bitcoind -dbcache=1024 -printtoconsole -onlynet=ipv4 -prune=<size_mb>`
Указание параметра prune со значением &lt;size_mb&gt; позволит уменьшить объем хранимого на диске блокчейна, но не сократит время синхронизации (фактически, синхронизация будет выполнена полностью, но будут отброшена информация тех транзакций, output которых на текущеую дату потрачен).

Также можно добавить параметр prune=&lt;size_mb&gt; в файл bitcoin.conf.


Скачать блокчейн, синхрониированный с параметром prune=1024, в виде архива testnet-prune1024.zip с папками blockchain и chainstate (которые нужно поместить в вашу папку bitcoin/testnet3) можно через ipfs:
- через ipfs.io: http://gateway.ipfs.io/ipfs/QmWPLELTjUe2RPBSVzHFkCk5SXT4GZ3f3ko3vszaxEagSu
- (лучше) запустив ipfs daemon локально и затем скачать по адресу http://localhost:8080/ipfs/QmWPLELTjUe2RPBSVzHFkCk5SXT4GZ3f3ko3vszaxEagSu
- (еще лучше) командой ipfs get QmWPLELTjUe2RPBSVzHFkCk5SXT4GZ3f3ko3vszaxEagSu

Архив в google drive https://goo.gl/EFmZLD

Помещать содержимое архива в вашу папку bitcoin/testnet3 нужно следующим образом:
1) запустить демон с ключами как указано выше (он создаст папку testnet3)
2) остановить демон, удалить папки blocks и chainstate из папки testnet3
3) разархивировать скаченный архив в папку testnet3 (в ней должны полявиться папки blocks и chainstate с содержимом из архива)



Далее можно выполнять команды, подключаясь к запущенному демону bitcoin:

`bitcoin-cli -rpcport=18332 -rpcuser=<имя_пользователя> -rpcpassword=<пароль> <команда>`

Для Windows создать файл btctestnet.bat (чтобы каждый раз не набирать ключи командной строки) с кодом:
```
@echo off
"c:\Program Files (x86)\Bitcoin\daemon\bitcoin-cli.exe" -rpcport=18332 -rpcuser=<имя_пользователя> -rpcpassword=<пароль> %1 %2 %3 %4 %5
```

Для Linux/Mac создать alias:

`alias btctestnet='bitcoin-cli -rpcport=18332 -rpcuser=<имя_пользователя> -rpcpassword=<пароль>'`


Попробуем подключиться к демону и запросить статус блокчейна:

`./btctestnet.bat getblockchaininfo`


Количество blocks должно быть равно максимальному номеру блока тут https://live.blockcypher.com/btc-testnet/
Если она меньше, то нужно дождаться завершения синхронизации.

### Запрашиваем тестовый BTC через faucet

Создаем адрес для получения BTC:

`./btctestnet.bat getnewaddress ""`

Получаем адрес **&lt;testnet_address1&gt;**.


Далее идем сюда https://testnet.manu.backend.hamburg/faucet и запрашиваем получение BTC в тестнете на этот адрес **&lt;testnet_address1&gt;**.


В случае успеха сревис выводит хеш транзакции.


Проверяем, что имеем входящую транзакцию:

`./btctestnet.bat listtransactions`


Копируем **&lt;txid&gt;** из входящей транзакции для создания новой.


Создаем транзакцию перевода средств со своего адреса на другой свой с одновременным указанием произвольных данных в транзакции:
```
btcclient.bat createrawtransaction "[{\"txid\": \"<txid>\",\"vout\":0}]" "{\"<testnet_address1>\": 0.2735, \"data\":\"deadbeef\"}"
```

Вместо deadbeef можно указать произвольные данные в hex-формате (до 80 байт, то есть до 140 символов в формате hex). Преобразовать текстовую строку в hex можно тут http://www.swingnote.com/tools/texttohex.php.

После создания транзакции подписываем ее `signrawtransaction` и отправляем через `sendrawtransaction`.

В реультате отправки получаем хеш (txid) новой транзакции и идем смотреть ее в блокчейн-эксплорере тестнета биткойна https://www.blocktrail.com/tBTC


Пример успешно отправленной транзакции:

https://www.blocktrail.com/tBTC/tx/aff1f506b1cad762c95c729822f529b77c983eef6924565c64c2ecdaa106b38e


Среди outputs видим операцию OP_RETURN, нажимаем 'view decoded message', видим текст '6a04deadbeef', где

- 6a - код операции OP_RETURN
- 04 - длина текста в байтах
- deadbeef - hex коды 4 байт: DE AD BE EF

### Задание 2
2.1. Easy. Запишите свой текст в тестовый блокчейн биткойна с использованием техники, описанной выше.

2.2. Hard. Нода bitcoin поддерживает JSON RPC, что позволяет формировать транзакции практически из любого языка программирования. С использованием библиотек наподобии этой https://www.npmjs.com/package/bitcoind-rpc это становится еще проще. Разработайте программу записи в блокчейн биткойна произвольного текста из командной строки node.js.


Полезные ссылки:
https://en.bitcoin.it/wiki/Running_Bitcoin