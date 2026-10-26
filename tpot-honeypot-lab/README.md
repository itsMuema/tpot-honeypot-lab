# T-Pot Honeypot Lab

A cloud honeypot deployed on Vultr to observe real-world internet attack traffic, then analyzed with Kibana and Suricata from a SOC analyst's perspective.

**Stack:** Vultr VPS · Ubuntu 26.04 LTS · T-Pot (Mini edition) · Docker · Elastic Stack (Elasticsearch, Logstash, Kibana) · Suricata

\---

## 1\. Goal

Build and secure an internet-facing honeypot, collect real attack data, and practice the analyst workflow: separate background noise from meaningful activity, document findings, and report them clearly.

## 2\. Environment

|Item|Detail|
|-|-|
|Provider|Vultr (cloud VPS)|
|OS|Ubuntu 26.04.1 LTS|
|Resources|8 GB RAM (about 7.2 GiB usable), 150 GB disk|
|Honeypot platform|T-Pot, Mini edition|
|Monitoring|Kibana dashboards, T-Pot Attack Map, Suricata IDS|
|Exposure window|24.10.2026/2130hrs to 25.10.2026/2200hrs 24hours 30Minutes|

**Why the Mini edition:** the standard Hive install lists 16 GB RAM as its requirement and the Sensor edition needs a separate Hive server to send logs to, so Mini was the best fit for an 8 GB single server.

## 3\. Security design

The server is deliberately vulnerable-looking, so the management side was locked down:

* **SSH key authentication** (Ed25519 key pair) instead of password login for the server.
* **Non-root admin user** with sudo for the T-Pot installation.
* **Provider firewall (Vultr firewall group):**

  * Management ports (SSH 64295, web dashboard 64297, install-time SSH 22) restricted to a single trusted IP (/32).
  * Honeypot port range (TCP/UDP 1-64000) opened to the internet only after the management side was verified.
  * Ports above 64000 left closed to the public.
* **Verified listening services** with `ss -tulpn`: honeypots run in Docker, real SSH on 64295, and Kibana, Elasticsearch and Logstash bound to localhost behind an nginx reverse proxy.
![DPS](screenshots/dps.jpg)

* **Isolation:** no personal data, credentials or other infrastructure were stored on or reachable from the server.

![Firewall rules](screenshots/firewall-rules.jpg)



|Time|Event|
|-|-|
|24.10.2026/2150hrs|Server deployed, T-Pot installed, management access verified|
|24.10.2026/2200hrs|Honeypot port range opened to the internet|
|+1 hour|43 events recorded on the Attack Map|
|25.10.2026/1551hrs|Analysis and evidence capture|
|25.10.2026/2200hrs|Server destroyed to stop charges|

![Attack Map, 24 hours](screenshots/attack-map-first-24hours.jpg)



5\. Findings

**Overview**

* Total events in 24 hours: 44346 events

**Top source countries:** Iran · United States · France· The Netherlands· Romania

**Top targeted services / ports:**  SMB 445, SSH 22, RDP 3389, HTTP 80, SQL Server 1433

Top source IPs:217.218.34.63,77.239.124.152,80.94.95.109

**Credentials attempted (top usernames / passwords):** Usernames:hadoop.Administrator,Admin.steam,devop Password:password,demo,123456,Admin@123,admin123

**Malware or file download attempts:** No

![Kibana dashboard](screenshots/kibana-overview.jpg)



### Suricata alerts

* Alerts began shortly after the ports were opened and stayed steady around the clock, which is typical of automated internet-wide scanning.
* The most common category was **Generic Protocol Command Decode**, mostly malformed or unexpected traffic aimed at open ports.
* The top Suricata alerts were protocol anomalies and informational events, consistent with automated scanning.
* SSH was the most visible service in the alerts, through port 22 probing and invalid banners.
* No exploit or malware signatures appeared in the top 10.



![Suricata alert categories](screenshots/suricata-histogram.jpg)

## 6\. Analysis

* **Noise vs signal:** most traffic came from mass scanners and previously reported IPs, so it was background noise rather than targeted attacks.
* **Interesting events:** No single source dominated the spike.- **Credential guessing:** attempts used default and common credentials (`Administrator`, `admin`, `123456`, `admin123`), consistent with automated brute-force tools targeting Windows and generic admin accounts.- **Service targeting:** the most probed services were SMB (445), SSH (22), RDP (3389) and SQL Server (1433), the usual remote-access and database targets.
* What did not happen:- **No malware downloads or exploit signatures were observed** in the 24-hour window.

* **Caveat:** Suricata alerts are detections, not confirmed compromises.

## 7\. Lessons learned

* **Firewall scoping mistake:** I first entered the server's IP in the firewall allow-list instead of my own, which caused SSH timeouts. Management rules need the *client's* public IP.
* **Installer failure:** the T-Pot installer failed at its first step because of a rejected sudo password. I diagnosed it from the "Sorry, try again" message in the output and retried.
* **Choosing an edition:** matched the T-Pot edition to the available RAM and avoided the Sensor edition.
* **Browser URL:** reaching the dashboard requires port 64297. Without it the browser lands on a honeypot web service.

## 8\. Next steps

* Write detection ideas based on the observed signatures.
* Try Zeek or Wazuh for comparison.
* Add a short incident-style report for one spike.

\---

*All IP addresses of my own systems, usernames and credentials have been removed or blurred. Attacker IPs shown come from public internet scanning.*

