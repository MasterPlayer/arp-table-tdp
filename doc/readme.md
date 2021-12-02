# arp_table_tdp

Параметризуемая память для использования в качестве таблицы, которая хранит ARP. Основана на true-dual port BRAM. Количество элементов задается через параметризацию. В таблице реализована работа под разными клоковыми доменами. Это самый простой вариант реализации таблицы для хранения параметров IP, MAC, PORT.


## Структура 

![arp_table_struct][logo1]

[logo1]:https://github.com/MasterPlayer/xilinx-vhdl/blob/master/eth_parts/arp_tables/arp_table_tdp/documentation/arp_table_tdp.png

## generic-параметры
Название | Тип | Диапазон значений | Описание
---------|-----|-------------------|---------
N_ELEMENTS | integer | >0 | Количество элементов
ADDR_WIDTH | integer | >0 | Разрядность шины адреса
ASYNC | boolean | true или false | Асинхронность портов памяти

## порты 

Компонент основан на двухпортовом примитиве памяти с разделением портов на PORTA и PORTB

### PORTA

Предназначен для конфигурации таблицы из-под внешнего интерфейса

Название | направление | Разрядность | Назначение
---------|-------------|-------------|-----------
CLKA             | вход | 1 | сигнал тактирования
RSTA             | вход | 1 | сигнал сброса порта A
CFG_ADDRA_IN     | вход | ADDR_WIDTH | сигнал адреса
CFG_DV_IN        | вход | 1 | сигнал разрешения команды
CFG_CMD_IN       | вход | 1 | сигнал команды. 0 - чтение, 1 - запись в таблицу
CFG_DST_MAC_IN   | вход | 48 | сигнал данных на запись MAC-адреса назначения на запись. При чтении не имеет эффекта
CFG_DST_IP_IN    | вход | 32 | сигнал данных на запись IP-адреса назначения. При чтении не имеет эффекта
CFG_DST_PORT_IN  | вход | 16 | сигнал данных на запись порта назначения. При чтении не имеет эффекта
CFG_SRC_IP_IN    | вход | 32 | сигнал данных на запись IP-адреса источника. При чтении не имеет эффекта
CFG_SRC_PORT_IN  | вход | 16 | сигнал данных на запись порта источника. При чтении не имеет эффекта
CFG_DST_MAC_OUT  | выход | 48 | сигнал данных на чтение MAC-адреса назначения. При записи должен игнорироваться
CFG_DST_IP_OUT   | выход | 32 | сигнал данных на чтение IP-адреса назначения. При записи должен игнорироваться
CFG_DST_PORT_OUT | выход | 16 | сигнал данных на чтение порта назначения. При записи должен игнорироваться
CFG_SRC_IP_OUT   | выход | 32 | сигнал данных на чтение IP-адреса источника. При записи должен игнорироваться
CFG_SRC_PORT_OUT | выход | 16 | сигнал данных на чтение порта ичточника. При записи должен игнорироваться
CFG_DV_OUT       | выход | 1 | сигнал валидности для операции чтения

### PORTB

Предназначен для запросов данных по адресу из под внешнего компонента(например, упаковщика трафика) 

Название | направление | Разрядность | Назначение
---------|-------------|-------------|-----------
CLKB           | вход | 1 | сигнал тактирвоания порта B памяти
RSTB           | вход | 1 | сигнал сброса для порта B
ADDRB_IN       | вход | ADDR_WIDTH | сигнал адреса для порта B
ADDRB_IN_VALID | вход | 1 | сигнал разрешения команды чтения для порта B
DST_MAC_OUT    | выход | 48 | сигнал чтения MAC-адреса назначения. Валиден только при `DVO = 1`
DST_IP_OUT     | выход | 32 | сигнал чтения IP-адреса назначения. Валиден только при `DVO = 1`
DST_PORT_OUT   | выход | 16 | сигнал чтения порта назначения. Валиден только при `DVO = 1`
SRC_IP_OUT     | выход | 32 | сигнал чтения ip-адреса источника. Валиден только при `DVO = 1`
SRC_PORT_OUT   | выход | 16 | сигнал чтения порта источника. Валиден только при `DVO = 1`
DVO            | выход | 1 | сигнал валидных данных. 


## Некоторые принципы работы компонента
- Порт A предназначается для конфигурации таблицы. Имеет возможность записи и чтения
- Порт B предназначен только для выполнения операции чтения
- ПОрт А и Порт B имеют возможность работать в асинхронном режиме. 
- Задержки на чтение являются конфигурируемыми для каждого порта и задаются константами READ_LATENCY_A - для порта A, и READ_LATENCY_B - для порта B
- Если запись в таблице по какому либо адресу отсутствует, но на шину будет все равно выдан валидный какой-либо результат, который хранится в адресе BRAM.


## Необходимые внешние компоненты:
Название компонента | Описание
--------------------|---------
[tdpram_xpm](https://github.com/MasterPlayer/xilinx-vhdl/blob/master/ram_parametrized/tdpram_xpm/tdpram_xpm.vhd) | Примитив двухпортовой памяти

## Лог изменений

**1. 14.02.2020 v1.0 - первая версия**