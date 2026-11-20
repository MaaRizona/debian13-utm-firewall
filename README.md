# Debian 13 (Trixie) UTM Firewall & Router Gateway

Prototyp bramy sieciowej klasy UTM oparty na systemie **Debian GNU/Linux 13 (Trixie)**, realizujący funkcje routingu, translacji adresów NAT, lokalnych usług DHCP/DNS oraz stanowej filtracji pakietów z punktem integracji dla silnika Suricata IPS i centralnego logowania Grafana/Loki.

## Architektura sieciowa (Hyper-V)

| Interfejs | Strefa | Adresacja IP | Rola w systemie |
| :--- | :--- | :--- | :--- |
| `eth0` | WAN (External) | `10.1.1.100/24` (DHCP) | Wyjście do sieci zewnętrznej, maskarada NAT, dostęp administracyjny SSH przez VPN |
| `eth1` | LAN (Private) | `192.168.10.1/24` (Static) | Brama domyślna dla sieci lokalnej, serwer DHCP (`192.168.10.50–150`) oraz DNS |

## Struktura repozytorium

* `interfaces` (`/etc/network/interfaces`) — konfiguracja statycznej adresacji interfejsu wewnętrznego `eth1`.
* `99-utm-security.conf` (`/etc/sysctl.d/99-utm-security.conf`) — włączenie przekazywania pakietów (`ip_forward = 1`) oraz utwardzanie stosu sieciowego jądra (`rp_filter`, `tcp_syncookies`, blokada ICMP Redirect).
* `utm-dhcp-dns.conf` (`/etc/dnsmasq.d/utm-dhcp-dns.conf`) — serwer DHCP i resolver DNS powiązany wyłącznie z interfejsem `eth1` (`bind-interfaces`).
* `nftables.conf` (`/etc/nftables.conf`) — stanowa zapora sieciowa (`policy drop`), maskarada SNAT na `eth0`, logowanie odrzuconych pakietów (`NFT-INPUT-DROP`, `NFT-FORWARD-DROP`) oraz kolejka `NFQUEUE 0 bypass` dla modułu Suricata IPS.
