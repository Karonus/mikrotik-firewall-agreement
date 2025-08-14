**Универсальная логическая схема Firewall и конфигурация MikroTik**

Этот документ описывает универсальную логическую схему организации фаервола, включая входные/выходные интерфейсы, соглашения об именовании цепочек и стандарты комментирования. В заключение приведена полная конфигурация MikroTik RouterOS, соответствующая этим принципам.


**Универсальная логическая схема для организации фаервола**

Основной принцип этой универсальной схемы фаервола — **разделение правил по назначению и направлению трафика**. Мы будем использовать **зоны безопасности** и **направления потока трафика**.

**Ключевые компоненты:**

**Интерфейсы:**
    * **Входящие (In):** Интерфейсы, через которые трафик поступает в фаервол.
    * **Исходящие (Out):** Интерфейсы, через которые трафик покидает фаервол.
    * **Локальные:** Трафик, предназначенный для самого фаервола (управление, логирование, системные службы).

**Цепочки:** Наборы правил, применяемые к трафику, соответствующему определенным критериям.

**Зоны безопасности:** Логические группы интерфейсов и сетей с аналогичными уровнями доверия.
    * **LAN (Внутренняя):** Основная внутренняя корпоративная сеть.
    * **DMZ (Демилитаризованная зона):** Сеть для публичных сервисов (веб-серверы, почтовые серверы).
    * **WAN (Внешняя/Интернет):** Внешняя, недоверенная сеть (Интернет).
    * **SRV (Серверы):** Сеть для внутренних серверных приложений (AD, SQL, 1C и т.д.).
    * **VPN:** Сеть для VPN-клиентов.
    * **GUEST:** Гостевая сеть.

**Логический поток трафика и цепочки:**

Мы используем три основные встроенные цепочки (`INPUT`, `OUTPUT`, `FORWARD`) в большинстве систем фаерволов (таких как Linux Netfilter, на котором основан фаервол MikroTik). 
Внутри них мы будем создавать пользовательские цепочки для более гранулированной фильтрации.

  * **INPUT:** Трафик, предназначенный для самого фаервола.
  * **OUTPUT:** Трафик, исходящий от самого фаервола.
  * **FORWARD:** Транзитный трафик, проходящий через фаервол.

**Ключевые принципы:**

  * **Запрет по умолчанию (Default Deny):** Весь трафик, не разрешенный явно, неявно отклоняется.
  * **Установленные/связанные соединения (Established/Related):** Разрешать трафик для уже установленных соединений на ранних этапах цепочки для повышения производительности.
  * **Принцип наименьших привилегий:** Разрешать только необходимый трафик и службы.
  * **Изначально мы определяем трафик по двум критериям:** - входной interface list (он определяет название изначальной цепочки) и address-list с ip адресами сетей - далее мы переходим в цепочки согласно направлению траффика

      Для примера:
      Forward -> FWD_LAN -> FWD_LAN_WAN
      Forward -> FWD_LAN -> FWD_LAN_SRV
      Forward -> FWD_LAN -> FWD_LAN_VPN

---

**2. Соглашения об именовании цепочек**

Хорошее соглашение об именовании гарантирует, что ваша конфигурация фаервола будет **понятной, предсказуемой и масштабируемой**.

**Предлагаемый формат:**
<code>
[DIRECTION]_[ZONE_FROM]_[ZONE_TO]_[PURPOSE]
</code>

Где:

**`DIRECTION`**: Указывает направление трафика.
    * `IN`: Входящий трафик (к фаерволу).
    * `OUT`: Исходящий трафик (от фаервола).
    * `FWD`: Транзитный трафик (всегда в цепочке `FORWARD`).

**`ZONE_FROM`**: Исходная зона безопасности (например, `WAN`, `LAN`, `DMZ`, `MGMT`, `GUEST`, `VPN`, `SRV`). 

**`ZONE_TO`**: Целевая зона безопасности (например, `WAN`, `LAN`, `DMZ`, `MGMT`, `GUEST`, `VPN`, `SRV`). 

**`PURPOSE` (Необязательно):** Дополнительно уточняет назначение цепочки для более гранулированной фильтрации.
    * `SERVICES`: Общие службы.
    * `MGMT`: Управление.
    * `ICMP`: Трафик ICMP.
    * `DROP_INVALID`: Отклонение недействительных пакетов.
    * `HTTP`, `HTTPS`, `MAIL`, `DNS`, `NTP`, `AD`, `SQL`, `RDP`, `SMB`, `KSC`, `P2P`: Специфические протоколы приложений или службы.

**Примеры именования цепочек:**

* **Основные цепочки:**
    * `IN`: Входящий трафик, предназначенный для самого фаервола (аналогично `INPUT`).
    * `OUT`: Исходящий трафик, генерируемый самим фаерволом (аналогично `OUTPUT`).
    * `FWD`: Основная цепочка для всего транзитного трафика (аналогично `FORWARD`).

**Зональные цепочки перенаправления:**
    * `FWD_LAN_WAN`: Трафик из LAN в WAN.
    * `FWD_WAN_DMZ`: Трафик из WAN в DMZ.
    * `FWD_LAN_SRV`: Трафик из LAN в SRV.
    * `FWD_VPN_SRV`: Трафик из VPN в SRV.

**Детализированные цепочки (вызываются из вышеуказанных):**
    * `FWD_WAN_DMZ_HTTP`: Разрешает HTTP из WAN в DMZ.
    * `IN_LAN_MGMT`: Разрешает доступ к управлению фаерволом.

---

**3. Стандарты комментирования **

Четкие комментарии имеют решающее значение для поддержания конфигураций фаервола. Они должны быть лаконичными, явными и не содержать специальных символов, которые могут вызвать проблемы в средах CLI.

**Общие принципы:**

  * **Язык:** Используйте **английский** для комментариев, чтобы обеспечить универсальность.
  * **Ясность и краткость:** Комментарии должны быть короткими и по существу, четко описывая назначение правила.
  * **Без специальных символов:** Используйте только **буквы (A-Z, a-z), цифры (0-9), дефисы (`-`), нижние подчеркивания (`_`) и пробелы**. Избегайте всех других символов (например, `!`, `@`, `#`, `(`, `)`, `'`, `"`).
  * **Заглавные буквы:** Начинайте комментарии с заглавной буквы.
  * **Без знаков препинания:** Не заканчивайте комментарии точкой.

**Структура комментариев:**

1.  **Заголовки секций:** Используются для логической группировки правил (например, `--- JUMP RULES ---`).
    * *Пример:* `add action=passthrough comment="--- JUMP RULES ---"`

2.  **Правила перехода (Jump Rules):** Указывают источник, назначение и цель перехода.
    * *Формат:* `JUMP: Traffic from [ZONE_FROM] to [ZONE_TO]`
    * *Пример:* `comment="JUMP: Traffic from LAN to WAN"`

3.  **Правила пользовательских цепочек:** Описывают конкретное действие и службу.
    * *Формат:* `[CHAIN_NAME]: Allow/Drop [SERVICE_OR_PROTOCOL_NAME]`
    * *Пример:* `comment="FWD_LAN_WAN: Allow HTTP HTTPS"`

4.  **Глобальные правила:** Описывают широкие, основополагающие правила.
    * *Пример:* `comment="Allow ESTABLISHED RELATED connections globally for FORWARD"`

5.  **Правила запрета по умолчанию (Default Deny Rules):** Четко указывают, что весь неразрешенный трафик отбрасывается.
    * *Пример:* `comment="FWD_LAN_WAN: DROP all other"`

---

**Конфигурация MikroTik RouterOS**

Этот раздел содержит полную конфигурацию MikroTik CLI, включающую логическую схему, соглашения об именовании и стандарты комментирования.

**Перед применением:**
  * **Замените все заполнители:** `[interface_name_for_ZONE]`, `[IP_addresses_for_ZONE]` и `[1C_server_port]` на актуальные данные вашей сети.
  * **Создайте резервную копию текущей конфигурации:** `/export file=backup-firewall-before-rules`
  * **Убедитесь, что у вас есть доступ к консоли/SSH** на случай неправильной конфигурации.

<code>

#Create Interface Lists
/interface list add name=LAN
/interface list add name=WAN
/interface list add name=SRV
/interface list add name=VPN
/interface list add name=DMZ
/interface list add name=GUEST

#Add your actual interfaces to the lists
/interface list member add list=LAN interface=[interface_name_for_LAN]
/interface list member add list=WAN interface=[interface_name_for_WAN]
/interface list member add list=SRV interface=[interface_name_for_SRV]
/interface list member add list=VPN interface=[interface_name_for_VPN]
/interface list member add list=DMZ interface=[interface_name_for_DMZ]
/interface list member add list=GUEST interface=[interface_name_for_GUEST]

#Create Address Lists
/ip firewall address-list add list=LAN_NET address=192.168.0.0/16
/ip firewall address-list add list=WAN_NET address=0.0.0.0/0
/ip firewall address-list add list=SRV_NET address=192.168.0.0/16
/ip firewall address-list add list=VPN_NET address=10.0.0.0/8
/ip firewall address-list add list=DMZ_NET address=198.51.100.0/24
/ip firewall address-list add list=GUEST_NET address=192.168.89.0/24

#Main 'forward' Chain Rules (Jumps to User-Defined Chains)
/ip firewall filter
#General rules for FORWARD chain
add chain=forward connection-state=established,related action=accept comment="Allow ESTABLISHED RELATED connections globally for FORWARD"
add action=accept chain=input comment="Allow ESTABLISHED RELATED connections globally for IINPUT" connection-state=established,related

#JUMP rules for specific traffic flows
add action=jump chain=forward comment="JUMP: Traffic from LAN to WAN" dst-address-list=!BOGON in-interface-list=LAN jump-target=FWD_LAN_WAN out-interface-list=WAN src-address-list=LAN_NET
add action=jump chain=forward comment="JUMP: Traffic from LAN to SRV" dst-address-list=SRV_NET in-interface-list=LAN jump-target=FWD_LAN_SRV out-interface-list=SRV src-address-list=LAN_NET
add action=jump chain=forward comment="JUMP: Traffic from LAN to VPN" dst-address-list=VPN_NET in-interface-list=LAN jump-target=FWD_LAN_VPN out-interface-list=VPN src-address-list=LAN_NET
add action=jump chain=forward comment="JUMP: Traffic from VPN to LAN" dst-address-list=LAN_NET in-interface-list=VPN jump-target=FWD_VPN_LAN out-interface-list=LAN src-address-list=VPN_NET
add action=jump chain=forward comment="JUMP: Traffic from VPN to SRV" dst-address-list=SRV_NET in-interface-list=VPN jump-target=FWD_VPN_SRV out-interface-list=SRV src-address-list=VPN_NET
add action=jump chain=forward comment="JUMP: Traffic from LAN to DMZ" dst-address-list=DMZ_NET in-interface-list=LAN jump-target=FWD_LAN_DMZ out-interface-list=DMZ src-address-list=LAN_NET
add action=jump chain=forward comment="JUMP: Traffic from DMZ to LAN" dst-address-list=LAN_NET in-interface-list=DMZ jump-target=FWD_DMZ_LAN out-interface-list=LAN src-address-list=DMZ_NET
add action=jump chain=forward comment="JUMP: Traffic from GUEST to LAN" dst-address-list=LAN_NET in-interface-list=GUEST jump-target=FWD_GUEST_LAN out-interface-list=LAN src-address-list=GUEST_NET
add action=jump chain=forward comment="JUMP: Traffic from SRV to LAN" dst-address-list=LAN_NET in-interface-list=SRV jump-target=FWD_SRV_LAN out-interface-list=LAN src-address-list=SRV_NET

#JUMP rules for specific traffic flows INPUT
add action=jump chain=input comment="JUMP: Traffic from WAN to LOCAL" in-interface-list=WAN jump-target=IN_WAN
add action=jump chain=input comment="JUMP: Traffic from LAN to LOCAL" in-interface-list=LAN jump-target=IN_LAN src-address-list=LAN_NET
add action=jump chain=input comment="JUMP: Traffic from GUEST to LOCAL" in-interface-list=GUEST jump-target=IN_GUEST src-address-list=GUEST_NET
add action=jump chain=input comment="JUMP: Traffic from VPN to LOCAL" in-interface-list=VPN jump-target=IN_VPN src-address-list=VPN_NET
add action=jump chain=input comment="JUMP: Traffic from SRV to LOCAL" in-interface-list=SRV jump-target=IN_SRV src-address-list=SRV_NET
add action=jump chain=input comment="JUMP: Traffic from DMZ to LOCAL" in-interface-list=DMZ jump-target=IN_SRV src-address-list=DMZ_NET

#Default DROP for all INPUT traffic not explicitly allowed by JUMP rules
add chain=forward action=input comment="Default DROP for all FORWARD traffic" connection-nat-state=!dstnat
#Default DROP for all FORWARD traffic not explicitly allowed by JUMP rules
add chain=forward action=drop comment="Default DROP for all FORWARD traffic" connection-nat-state=!dstnat

#User-Defined Chains Rules

#FWD_LAN_WAN Chain (Traffic from LAN to WAN)
add chain=FWD_LAN_WAN protocol=tcp dst-port=80,443 action=accept comment="FWD_LAN_WAN: Allow HTTP HTTPS"
add chain=FWD_LAN_WAN protocol=tcp dst-port=25,465,110,995,143,993 action=accept comment="FWD_LAN_WAN: Allow Mail"
add chain=FWD_LAN_WAN protocol=udp dst-port=53 action=accept comment="FWD_LAN_WAN: Allow DNS UDP"
add chain=FWD_LAN_WAN protocol=tcp dst-port=53 action=accept comment="FWD_LAN_WAN: Allow DNS TCP"
add chain=FWD_LAN_WAN protocol=udp dst-port=123 action=accept comment="FWD_LAN_WAN: Allow NTP"
add chain=FWD_LAN_WAN protocol=icmp action=accept comment="FWD_LAN_WAN: Allow ICMP"
add chain=FWD_LAN_WAN protocol=tcp dst-port=6881-6999 action=accept comment="FWD_LAN_WAN: Allow P2P TCP (Example be cautious!)"
add chain=FWD_LAN_WAN protocol=udp dst-port=6881-6999 action=accept comment="FWD_LAN_WAN: Allow P2P UDP (Example be cautious!)"
add chain=FWD_LAN_WAN action=drop comment="FWD_LAN_WAN: DROP all other"

#FWD_LAN_SRV Chain (Traffic from LAN to SRV)
add chain=FWD_LAN_SRV protocol=tcp dst-port=80,443 action=accept comment="FWD_LAN_SRV: Allow internal WEB"
add chain=FWD_LAN_SRV protocol=tcp dst-port=389,636,88,445 action=accept comment="FWD_LAN_SRV: Allow AD LDAP Kerberos SMB"
add chain=FWD_LAN_SRV protocol=udp dst-port=88,389 action=accept comment="FWD_LAN_SRV: Allow AD Kerberos LDAP UDP"
add chain=FWD_LAN_SRV protocol=tcp dst-port=1540,1541,475,1560-1591 action=accept comment="FWD_LAN_SRV: Allow 1C Server"
add chain=FWD_LAN_SRV protocol=udp dst-port=475 action=accept comment="FWD_LAN_SRV: Allow 1C Server"
add chain=FWD_LAN_SRV protocol=tcp dst-port=1433,3306,5432 action=accept comment="FWD_LAN_SRV: Allow SQL Servers"
add chain=FWD_LAN_SRV protocol=tcp dst-port=3389 action=accept comment="FWD_LAN_SRV: Allow RDP"
add chain=FWD_LAN_SRV protocol=tcp dst-port=139,445 action=accept comment="FWD_LAN_SRV: Allow SMB TCP"
add chain=FWD_LAN_SRV protocol=udp dst-port=137,138 action=accept comment="FWD_LAN_SRV: Allow SMB UDP"
add chain=FWD_LAN_SRV protocol=tcp dst-port=13000,14000,17000 action=accept comment="FWD_LAN_SRV: Allow Kaspersky KSC"
add chain=FWD_LAN_SRV protocol=udp dst-port=53 action=accept comment="FWD_LAN_SRV: Allow DNS UDP"
add chain=FWD_LAN_SRV protocol=tcp dst-port=53 action=accept comment="FWD_LAN_SRV: Allow DNS TCP"
add chain=FWD_LAN_SRV protocol=icmp action=accept comment="FWD_LAN_SRV: Allow ICMP"
add chain=FWD_LAN_SRV action=drop comment="FWD_LAN_SRV: DROP all other"

#FWD_LAN_VPN Chain (Traffic from LAN to VPN)
add chain=FWD_LAN_VPN action=accept comment="FWD_LAN_VPN: Allow All traffic from LAN to VPN clients"
#If more granular policy needed, uncomment and modify:
add chain=FWD_LAN_VPN protocol=tcp dst-port=3389 action=accept comment="FWD_LAN_VPN: Allow RDP to VPN clients"
add chain=FWD_LAN_VPN action=drop comment="FWD_LAN_VPN: DROP all other"

#FWD_VPN_LAN Chain (Traffic from VPN to LAN)
add chain=FWD_VPN_LAN action=accept comment="FWD_VPN_LAN: Allow All traffic from VPN clients to LAN"
#If more granular policy needed, uncomment and modify:
add chain=FWD_VPN_LAN protocol=tcp dst-port=3389 dst-address-list=LAN_NET action=accept comment="FWD_VPN_LAN: Allow RDP to LAN from VPN"
add chain=FWD_VPN_LAN action=drop comment="FWD_VPN_LAN: DROP all other"

#FWD_VPN_SRV Chain (Traffic from VPN to SRV)
add chain=FWD_VPN_SRV protocol=tcp dst-port=80,443 action=accept comment="FWD_VPN_SRV: Allow internal WEB"
add chain=FWD_VPN_SRV protocol=tcp dst-port=389,636,88,445 action=accept comment="FWD_VPN_SRV: Allow AD LDAP Kerberos SMB"
add chain=FWD_VPN_SRV protocol=udp dst-port=88,389 action=accept comment="FWD_VPN_SRV: Allow AD Kerberos LDAP UDP"
add chain=FWD_LAN_SRV protocol=tcp dst-port=1540,1541,475,1560-1591 action=accept comment="FWD_LAN_SRV: Allow 1C Server"
add chain=FWD_LAN_SRV protocol=udp dst-port=475 action=accept comment="FWD_LAN_SRV: Allow 1C Server"
add chain=FWD_VPN_SRV protocol=tcp dst-port=1433,3306,5432 action=accept comment="FWD_VPN_SRV: Allow SQL Servers"
add chain=FWD_VPN_SRV protocol=tcp dst-port=3389 action=accept comment="FWD_VPN_SRV: Allow RDP"
add chain=FWD_VPN_SRV protocol=tcp dst-port=139,445 action=accept comment="FWD_VPN_SRV: Allow SMB TCP"
add chain=FWD_VPN_SRV protocol=udp dst-port=137,138 action=accept comment="FWD_VPN_SRV: Allow SMB UDP"
add chain=FWD_VPN_SRV protocol=tcp dst-port=13000,14000,17000 action=accept comment="FWD_VPN_SRV: Allow Kaspersky KSC"
add chain=FWD_VPN_SRV protocol=udp dst-port=53 action=accept comment="FWD_VPN_SRV: Allow DNS UDP"
add chain=FWD_VPN_SRV protocol=tcp dst-port=53 action=accept comment="FWD_VPN_SRV: Allow DNS TCP"
add chain=FWD_VPN_SRV protocol=icmp action=accept comment="FWD_VPN_SRV: Allow ICMP"
add chain=FWD_VPN_SRV action=drop comment="FWD_VPN_SRV: DROP all other"

#FWD_LAN_DMZ Chain (Traffic from LAN to DMZ)
add chain=FWD_LAN_DMZ protocol=tcp dst-port=80,443 action=accept comment="FWD_LAN_DMZ: Allow WEB to DMZ"
add chain=FWD_LAN_DMZ protocol=tcp dst-port=25,465,110,995,143,993 action=accept comment="FWD_LAN_DMZ: Allow Mail to DMZ"
add chain=FWD_LAN_DMZ protocol=icmp action=accept comment="FWD_LAN_DMZ: Allow ICMP"
add chain=FWD_LAN_DMZ action=drop comment="FWD_LAN_DMZ: DROP all other"

#FWD_DMZ_LAN Chain (Traffic from DMZ to LAN) - Very strict rules!
#Only ESTABLISHED/RELATED is allowed globally. Add specific rules below ONLY if critical.
#add chain=FWD_DMZ_LAN protocol=tcp dst-port=1433 dst-address=[IP_of_LAN_SQL_SERVER] action=accept comment="FWD_DMZ_LAN: Allow DMZ Web to LAN SQL (USE CAUTION!)"
add chain=FWD_DMZ_LAN action=drop comment="FWD_DMZ_LAN: DROP all other (Default Deny)"

#FWD_GUEST_LAN Chain (Traffic from GUEST to LAN) - Strictly forbidden!
add chain=FWD_GUEST_LAN action=drop comment="FWD_GUEST_LAN: DROP all traffic from GUEST to LAN"

#FWD_SRV_LAN Chain (Traffic from SRV to LAN)
#If specific services are needed from SRV to LAN:
#add chain=FWD_SRV_LAN protocol=tcp src-port=17000 action=accept comment="FWD_SRV_LAN: Allow KSC Server to Agents in LAN"
add chain=FWD_SRV_LAN action=drop comment="FWD_SRV_LAN: DROP all other"

</code>
