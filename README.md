# Kiçik Ofis Şəbəkəsi Topologiyası (Cisco Packet Tracer)

## Məqsəd

Cisco Packet Tracer istifadə edərək, 3 şöbədən ibarət (Management, HR, IT)
kiçik ofis şəbəkəsi simulyasiyası qurmaq. Layihə VLAN seqmentasiyası, inter-VLAN
routing, iki router arasında dinamik marşrutlama (OSPF) və ACL ilə şöbələr arası
giriş nəzarətini əhatə edir.

## Topologiya

- **Router1** — Router-on-a-Stick metodu ilə 3 VLAN-ı marşrutlayır
- **Switch1** — 3 VLAN-a bölünmüş (Access portlar) + Router1-ə trunk bağlantı
- **Router2** — Router1 ilə OSPF vasitəsilə bağlı, ayrı bir filial/binanı təmsil edir
- 9 PC (3 VLAN-da 2-şər + Router2 tərəfində 3)

## IP Ünvan Planı

| VLAN/Şəbəkə | Ad | Şəbəkə | Gateway |
|---|---|---|---|
| VLAN 10 | Management | 192.168.10.0/24 | 192.168.10.1 |
| VLAN 20 | HR | 192.168.20.0/24 | 192.168.20.1 |
| VLAN 30 | IT | 192.168.30.0/24 | 192.168.30.1 |
| Router1↔Router2 | Link | 10.0.0.0/30 | — |
| Router2 tərəfi | Filial | 192.168.100.0/24 | 192.168.100.1 |

Filial tərəfindəki 3 PC-nin IP-ləri: `192.168.100.10`, `192.168.100.11`, `192.168.100.12` (gateway: `192.168.100.1`)

## Konfiqurasiya Addımları

### 1. Switch-də VLAN yaradılması
```
vlan 10
 name Management
vlan 20
 name HR
vlan 30
 name IT
```

### 2. Switch portlarının VLAN-lara təyin edilməsi
PC-lərə bağlı portlar **access mode**-da müvafiq VLAN-a təyin edildi.
Router1-ə bağlı port **trunk mode**-a keçirildi ki, bütün VLAN-ların
trafiki tək kabellə ötürülə bilsin.

### 3. Router-on-a-Stick (Inter-VLAN Routing)
Router1-in fiziki interfeysi üzərində hər VLAN üçün subinterfeys yaradıldı
(`dot1Q` encapsulation ilə), hər birinə gateway IP-si verildi.

### 4. Router1 ↔ Router2 bağlantısı və OSPF
İki router arasında fiziki link qurulub IP ünvanlar verildi. Hər iki
routerda OSPF (area 0) konfiqurasiya olunaraq, bütün şəbəkələr (3 VLAN +
filial şəbəkəsi) dinamik olaraq bir-birinə tanıdıldı.

### 5. ACL ilə giriş nəzarəti
Router1 üzərində extended ACL yaradıldı:
- HR (VLAN 20) ↔ IT (VLAN 30) arası trafik **qadağan edildi**
- Management (VLAN 10) hər iki VLAN-a sərbəst giriş imkanına **sahib qaldı**
- Bütün VLAN-lar filial şəbəkəsinə (Router2) sərbəst çata bilir

ACL, VLAN 20 və VLAN 30-un subinterfeyslərinə `in` istiqamətində tətbiq
olundu.

## Test Nəticələri

| Test | Mənbə | Hədəf | Nəticə |
|---|---|---|---|
| Inter-VLAN routing | VLAN 10 PC | VLAN 20 PC | ✅ Uğurlu ping |
| OSPF marşrutlama | VLAN 10 PC | Filial (192.168.100.10) | ✅ Uğurlu ping |
| ACL — qadağan | VLAN 20 PC | VLAN 30 PC | ✅ Bloklandı (gözlənilən nəticə) |
| ACL — icazə | Management (VLAN 10) PC | VLAN 20 və VLAN 30 | ✅ Hər ikisi əlçatan |
| ACL yan-təsiri yoxlaması | VLAN 20 PC | Filial (192.168.100.10) | ✅ Əlçatan (ACL yalnız 20↔30 arasını bloklayır) |

## Öyrənilən Əsas Anlayışlar

- VLAN seqmentasiyası və access/trunk port fərqi
- Router-on-a-Stick metodu ilə tək fiziki interfeys üzərindən çoxlu VLAN-ın
  marşrutlanması
- Wildcard mask məntiqi (subnet mask-ın OSPF/ACL-də tərs istifadəsi)
- OSPF-in şəbəkələri necə avtomatik elan edib öyrəndiyi
- Extended ACL-in mənbə/hədəf əsasında trafiki necə süzdüyü və
  "implicit deny" prinsipi (ACL-in sonunda `permit any any` olmasa,
  bütün digər trafik avtomatik bloklanır)

## Real Həyatda Tətbiq Məntiqi

Bu topologiya, real bir ofisdə tez-tez rast gəlinən bir təhlükəsizlik
tələbini simulyasiya edir: həssas işçi məlumatlarına (maaş, şəxsi
məlumatlar) sahib olan HR şöbəsi ilə texniki idarəetmə girişinə sahib
IT şöbəsi arasında birbaşa şəbəkə əlaqəsinin kəsilməsi, halbuki
rəhbərliyin (Management) hər iki şöbəyə nəzarət imkanının qorunması.
