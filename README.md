# Open-Source UTM Firewall (Debian 13)

Projekt inżynierski realizowany na kierunku **Cyberbezpieczeństwo**. Celem pracy jest zaprojektowanie, wdrożenie oraz weryfikacja wielostrefowej zapory sieciowej klasy UTM (*Unified Threat Management*) opartej na systemie **Debian 13 (Trixie)**.

## Architektura i Topologia Sieciowa

System pełni funkcję routera brzegowego oraz stanowej zapory sieciowej, zapewniając izolację ruchu pomiędzy strefami bezpieczeństwa oraz szyfrowany dostęp administracyjny w modelu **Zero Trust**.

| Interfejs | Strefa | Adresacja IP / Pula DHCP | Rola w systemie |
| :--- | :--- | :--- | :--- |
| `enp0s3` | **WAN** | DHCP (NAT Hypervisora) | Interfejs zewnętrzny (`policy drop`, otwarty wyłącznie port `51820/UDP` dla WireGuard) |
| `enp0s8` | **LAN** | `192.168.10.1/24` (DHCP: `.50–.150`) | Zaufana sieć wewnętrzna (stacje robocze) |
| `enp0s9` | **DMZ** | `10.0.100.1/24` (DHCP: `.50–.150`) | Izolowana strefa zdemilitaryzowana (brak możliwości inicjowania połączeń do LAN) |
| `wg0` | **VPN** | `10.8.0.1/24` | Szyfrowany tunel administracyjny WireGuard (jedyny punkt wejścia SSH z zewnątrz) |

## Struktura Konfiguracji (Infrastructure as Code)

* `interfaces` – statyczna i dynamiczna adresacja interfejsów sieciowych (`WAN`, `LAN`, `DMZ`).
* `99-utm-security.conf` – parametry utwardzania (*hardeningu*) stosu sieciowego jądra Linux (`ip_forward`, `rp_filter`, `tcp_syncookies`, blokada `ICMP redirects`).
* `nftables.conf` – stanowa filtracja pakietów w przestrzeni `inet`, translacja adresów NAT (`masquerade`), izolacja strefowa oraz przekierowanie ruchu do kolejki `NFQUEUE 0` (z flagą `bypass`) na potrzeby silnika IPS.
* `utm-dhcp-dns.conf` – konfiguracja lokalnego serwera DHCP oraz resolvera DNS (`dnsmasq`), powiązanego wyłącznie z interfejsami wewnętrznymi.

## Instrukcja Uruchomienia Środowiska (dla zespołu)

1. Zaimportuj obraz bazowy `UTM-Gateway-Debian13-Base.ova` z dysku zespołowego OneDrive do Oracle VirtualBox.
2. W ustawieniach sieciowych maszyny wirtualnej upewnij się, że:
   * **Karta 1 (`enp0s3`):** tryb *NAT* z przekierowaniem portu **UDP `51820` -> `51820`**.
   * **Karta 2 (`enp0s8`):** *Sieć wewnętrzna* (`net-lan`).
   * **Karta 3 (`enp0s9`):** *Sieć wewnętrzna* (`net-dmz`).
3. Zaimportuj dostarczony profil klienta (`client.conf`) do aplikacji WireGuard na hoście i aktywuj tunel.
4. Połącz się z bramą przez SSH: `ssh trixie@10.8.0.1`.

## Moduły Projektu i Podział Prac

* **Core Networking, Firewall (`nftables`) & Zero Trust VPN (`WireGuard`):** Wdrożone w obrazie bazowym v1.0.
* **Detekcja i Blokowanie Włamań (`Suricata IDS/IPS`):** Integracja z kolejką `NFQUEUE 0` *(w toku)*.
* **Centralizacja Logów i Wizualizacja (`Grafana` + `Loki`):** Zbieranie zdarzeń jądra oraz alertów IPS *(w toku)*.
