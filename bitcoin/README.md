## Работа с блокчейном биткойна

Основной справочник по командам bitcoin:
https://en.bitcoin.it/wiki/Original_Bitcoin_client/API_calls_list

Запустить демон:
```
"c:\Program Files (x86)\Bitcoin\daemon\bitcoind.exe" -debug=1 -printtoconsole -regtest=1 -rpcport=8332 -rpcpassword=pass -listenonion=0
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
Скопируем значения поля "txid" <txid> для использования в дальшейнем в примере (будет переводить эти 50 BTC).
Пример: 56d8a69a250846b9acea9b4c7a5f9eed9d3ef4a254ba35247dbb75ad872695f4

Еще раз проверить баланс текущего аккаунта:
`./btcclient.bat getbalance`
Все еще 0. Дело в том, что трананзакция создания 50 BTC еще недостаточно "старая".

В информации о кошельке видим immature balance:
`./btcclient.bat getwalletinfo`
В выводе видим, "immature_balance": 50.00000000

Создаем новый аккаунт и адрес в нем (будем на него переводить средства)
`./btcclient.bat getnewaddress "account2"`
Записываем адрес **<address2>**
Пример: 2NFk5tWmRTvwo3gv9x2ENMB23ewPNR4aiG2

Создаем для аккаунта аккаунт "по-умолчанию" адрес для "сдачи":
`./btcclient.bat getnewaddress ""`
Записываем **<address3>**
Пример: 2MvMvWTADM7krpbG6BeZ4D1JgXKoNA1MKGe

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
Создаем транзакцию перевода 10 BTC с непотраченного output coinbase-транзакции самого первого блока на **<address2>** со сдачей на **<address3>**:
`./btcclient.bat createrawtransaction "[{\"txid\": \"<txid>\",\"vout\":0}]" "{\"<address2>\": 10, \"<address3>\": 39.9999 }"`
Получаем hex транзакции **<raw transaction hex>**
Пример: 0200000001f4952687ad75bb7d2435ba54a2f43e9ded9e5f7a4c9beaacb94608259aa6d8560000000000ffffffff0200ca9a3b0000000017a914f6c811abd649d89dc6b857a8bbc17731ea4815c887f0006bee0000000017a914222cb297c0fc1dfcc94ab94bf6714dc83fafbe538700000000

Пример
```
./btcclient.bat createrawtransaction "[{\"txid\": \"4fa6b0361b416dd1cb3f468975100e5a3553984d6d6016a0910def3049c3820f\",\"vout\":0}]" "{\"2MznUekxqCqGEKc5dbBmJ22d3vkUphb6FE8\": 10, \"2NDoToJGp2PWnqscBc7z2tpxm6drP1TDZva\": 39.9999 }"
```

Подписываем транзакцию:
`./btcclient.bat signrawtransaction <raw transaction hex>`
Получаем **<signed raw transaction hex>**
Ожидаем "complete": true
Пример: 0200000001f4952687ad75bb7d2435ba54a2f43e9ded9e5f7a4c9beaacb94608259aa6d8560000000049483045022100ad6d8d4ff8cfe8d0507af9e6f72298ec3f17520789b8f85ff481b6f46f2d6dcb0220449bfdbaff8b41da13608f1ff2ecda72ec58a8b24e369f811f38590a004749b701ffffffff0200ca9a3b0000000017a914f6c811abd649d89dc6b857a8bbc17731ea4815c887f0006bee0000000017a914222cb297c0fc1dfcc94ab94bf6714dc83fafbe538700000000

Проверяем, что транзакции нет в мемпуле (она еще не отправлена):
`./btcclient.bat getrawmempool`
Получаем: пустой массив (мемпул пуст)

Отправляем транзакцию:
`./btcclient.bat sendrawtransaction <signed raw transaction hex>`
В случае успеха выводится хеш нашей транзакции перевода  **<transaction hash>**
Пример: 32e9c13d47ee939fabb954ea1df76b2463ca6e9b1679a13b848683d656c239f5

Можем посмотреть содержание транзакции:
`./btcclient.bat gettransaction <transaction hash>`

Проверяем, что транзакция появилась в мемпуле (она еще не включена в блок):
`./btcclient.bat getrawmempool`
В пуле появилась транзакция **<transaction hash>**.

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
C:\Users\Asus233\AppData\Roaming\Bitcoin\regtest


Дополнительные материалы:
https://en.bitcoin.it/wiki/Script
https://en.bitcoin.it/wiki/Contract
http://siminchen.github.io/bitcoinIDE/build/editor.html
https://bitcoin.stackexchange.com/questions/30543/can-i-transfer-bitcoins-generated-in-regtest-mode-to-a-friend

## Задание.
1. Объясните, почему при вызове getbalance отображается 99.99990000 BTC?
2. Что будет с балансами аккаунтов при выполнении generate 1 (создании еще одного блока)?
