# stremio-ocihosted  
**Stremio Stack con Mammamia, MediaFlow Proxy e altro ancora**

Questo repository contiene istruzioni, configurazioni e suggerimenti per il self-hosting **Oracle Cloud Free Tier** di un'intera istanza privata di Stremio, con plugin **Mammamia**, **MediaFlow Proxy**, **StreamV**, **AIOStreams** e altri componenti opzionali.

---

## 📢 Disclaimer (con un piccolo rant) 

> Questo progetto è a scopo puramente educativo.  
> L'utilizzo improprio di componenti che accedono a contenuti protetti da copyright potrebbe violare le leggi del tuo paese.  
> **Usa questi strumenti solo per contenuti legalmente ottenuti.**  
> L’autore non si assume responsabilità per eventuali usi illeciti.

> ### 📣 Un pensiero personale:
> Se stai usando Stremio con mille plugin e Real-Debrid, sappilo: non sei un pirata.
> 
>  **News flash: non sei un pirata, sei un leacher da salotto con le crocs ai piedi.**
> 
> Il pirata vero seedava, uploada, si faceva il port forwarding da solo e sniffava i peer con Wireshark. Tu clicchi e guardi. Comodo, eh? Ma zero gloria.
> Stai "guardando gratis" sì, ma stai succhiando banda da server altrui senza restituire niente.

> **Non dai nulla, non condividi nulla.**  

> **Zero upload, zero sharing, zero rispetto per chi ci mette storage, tempo e skill.**
> La tua banda in upload è più vuota della cartella “Download” su eMule nel 2025.
> 
> 💰 E poi ci sono quelli che bypassano la pubblicità sui siti di streaming…
> Ma lo sai che quei 2 banner schifosi sono l’unica cosa che tiene in piedi quei siti?
> Se li togli pure quelli, poi piangi perché non trovi più il film russo del 2003 sottotitolato in polacco.

> 💀 Se sei uno che si è mai lamentato per la qualità di uno stream pirata...ti meriti il buffering perpetuo.

> Se proprio vuoi vivere ai margini del sistema, almeno fallo con un po’ di dignità.
> Usa i torrent. Condividi. Seeda. Rompiti la testa sui port forwarding.
> E soprattutto: non fare il figo con gli script di qualcun altro.

> “Steal with style. Share like it’s 2006. Respect the swarm.”

---

## 🎁 Vantaggi del Free Tier di Oracle Cloud

Oracle Cloud offre un piano gratuito **senza scadenza** con una serie di servizi utilizzabili a costo zero. I principali vantaggi includono:

- ✅ **fino a 2 istanze Ampere A1 (ARM)** fino a 4 OCPU e 24 GB RAM **totali**
- ✅ **2 istanza AMD (x86)** VM.Standard.E2.1.Micro
- ✅ **200 GB totale di storage block** da da utilizzare per le Compute Istances
- ✅ **10 GB di Object Storage**
- ✅ **Rete virtuale (VCN) gratuita** con indirizzo IP pubblico
- ✅ **Accesso a strumenti avanzati** come Load Balancer, Monitoring, CLI, SDK, e Terraform
- ✅ **Zero costi permanenti**, finché resti nel Free Tier
- ✅ **Prestazioni elevate**, paragonabili a servizi cloud a pagamento

⚠️ **Importante**: nessun addebito viene effettuato a meno che non si passi manualmente al piano **"Pay As You Go"**.

---

## ✅ Requisiti  

Hai deciso di configurare una tua istanza privata di Mammamia e Media Flow Proxy, senza spendere un centesimo? Ecco cosa ti serve:

### 🔐 Account Oracle Cloud
- Vai su [Oracle Cloud Free Tier](https://www.oracle.com/cloud/free/)
- Completa la registrazione inserendo i tuoi dati personali
- Inserisci una carta di credito per la verifica (**non verranno effettuati addebiti** se resti nel Free Tier)

### 🖥️ Shape istanze e sistema operativo
- Puoi creare **una istanza AMD** (architettura x86), che per questo progetto è adeguata.
- In alternativa puoi creare fino a **2 istanze ARM (Ampere A1)** con massimo **4 OCPU e 24 GB RAM totali**, ottime per progetti più intensivi o specializzati.
  > ℹ️ **Nota**: Le istanze ARM sono spesso **non disponibili** nella regione `Italy-Milan`, quindi si consiglia di usare l'istanza AMD per iniziare e riservare l'uso delle istanze ARM a progetti specifici o regioni con disponibilità (es. `Germany-Frankfurt`, `US-Ashburn`).
- Sistema Operativo Ubuntu Server Minimal 22.04.  
  > *(Le istruzioni sono per Ubuntu, ma facilmente adattabili ad altre distro.)*

### 🌍 Accesso remoto con IP dinamico + Modifica regole Firewall su Oracle Cloud Platform
> ℹ️ In realtà Oracle Cloud Platform nel piano Free Tier permette di avere fino a 2 indirizzi ip pubblici riservati da assegnare alle istanze dopo la creazione. Ovviamente questi IP essendo riservati e assegnati staticamente alle vostre istanze non cambieranno mai nel corso del tempo. Il mio consiglio è di utilizzare questi ip per progetti più importanti... tanto la soluzione c'è

Poiché poichè l'ip assegnato alle istanze create su OCP è dinamico, è necessario un sistema per mantenere accessibile il tuo server anche quando l’IP cambia.
- Un **IP pubblico** (va bene anche se dinamico).
- Un account gratuito su [**duckdns.org**](https://www.duckdns.org) per creare **hostname statici** che puntano sempre al tuo NAS.
- Normalmente l'accesso alle istanze OCP è permesso solo sulla porta 22 (SSH) è necessario aprire l'accesso anche le porte:
  - Porta **80** (HTTP) 
  - Porta **443** (HTTPS)
  - Porta **8080** (Admin Panel di Nginx Proxy Manager)

> 🔁 Questo setup è fondamentale per permettere a Nginx Proxy Manager di ottenere e rinnovare automaticamente i certificati SSL tramite Let’s Encrypt.


### 🔐 Creazione degli hostname su DuckDns

Per accedere alle tue applicazioni da remoto, devi creare 1 hostname pubblico gratuito su [**duckdns.org**](https://www.duckdns.org).

> ⚠️ GL'hostname deve essere univoco. Il mio consiglio è quello di aggiungere un identificativo personale (es. il tuo nome o una sigla) per evitare conflitti.

#### Esempi di hostname personalizzato:
- `stremio-mario.duckdns.org`

Puoi ovviamente scegliere qualsiasi nome, purché sia disponibile e facile da ricordare.

>ℹ️ Poiché **DuckDNS gestisce wildcard per ogni sottodominio**, non appena creiamo uno hostname come **stremio-mario.duckdns.org** possiamo—utilizzando Nginx e generando un certificato valido per *.stremio-mario.duckdns.org—ospitare sulla nostra macchina più servizi sotto domini diversi. 

#### Esempi di hostname che andremo a creare su Nginx:
- `mammamia.stremio-mario.duckdns.org`
- `mfp.stremio-mario.duckdns.org`
- `streamv.stremio-mario.duckdns.org`
- `aiostreams.stremio-mario.duckdns.org`

Questi hostname punteranno sempre al tuo NAS anche se il tuo IP cambia.  
Il tutto è possibile installando un piccolo agente (Dynamic DNS client) che aggiorna automaticamente il record DNS.

> ℹ️ In realtà Oracle Cloud Platform nel piano Free Tier permette di avere fino a 2 indirizzi ip pubblici riservati da assegnare alle istanze dopo la creazione. Ovviamente questi IP essendo riservati e assegnati staticamente alle vostre istanze non cambieranno mai nel corso del tempo. Il mio consiglio è di utilizzare questi ip per progetti più importanti... tanto la soluzione c'è

### 🍴 Consigliato: fai un fork del repository

> ✨ E' consigliabile creare un **fork personale** di questo repository su GitHub, in modo da poterlo modificare facilmente secondo le tue esigenze.
> Per farlo ti servirà anche un account su GitHub

Per fare ciò:
1. Vai sulla pagina del repository originale
2. Clicca su **"Fork"** (in alto a destra)
3. Clona il tuo fork sul NAS:

```bash
git clone https://github.com/<il-tuo-utente>/<nome-repo>.git
cd <nome-repo>
```
---

## 🔧 Componenti del progetto

| Servizio           | Nome Servizio Docker | Porta interna | Descrizione                              |
|--------------------|----------------------|---------------|------------------------------------------|
| **[Mammamia](https://github.com/UrloMythus/MammaMia)**|mammamia       | 8080(*)          | Plugin personalizzato per Stremio        |
| **[MediaFlow Proxy (MFP)](https://github.com/mhdzumair/mediaflow-proxy)**|mediaflow_proxy | 8888(*)   | Proxy per streaming video                |
| **[StreamV](https://github.com/qwertyuiop8899/StreamV)**|steamv        | 7860(*)          | Web player personalizzato (opzionale)    |
| **[Nginx Proxy Manager](https://github.com/NginxProxyManager/nginx-proxy-manager)**|npm | 80/443/8080 | Reverse proxy + certificati Let's Encrypt |
| **[docker-duckdns](https://github.com/linuxserver/docker-duckdns)** |duckdns-updater |—         | Aggiorna il DNS dinamicamente            |
| **[AIOStreams](https://github.com/Viren070/AIOStreams)** |aiostreams |3000(*)        | multipli Stremio addons e servizi debrid in un solo plugin|

>ℹ️ (*)Le **porte elencate (tranne quelle di Nginx Proxy Manager)** sono **interne alla rete Docker** e **non sono esposte direttamente** sulla macchina host.
Questo significa che i servizi **non sono accessibili dall’esterno se non tramite Nginx Proxy Manager**, che funge da gateway sicuro con supporto a **HTTPS e Let's Encrypt**.

---

## 📝 Passaggi per creare un'istanza

### 1. 🔐 Registrazione

- Vai su [Oracle Cloud Free Tier](https://www.oracle.com/cloud/free/)
- Completa la registrazione inserendo i tuoi dati personali
- Inserisci una carta di credito per la verifica (**non verranno effettuati addebiti** se resti nel Free Tier)
  ![image](https://github.com/user-attachments/assets/38a9c92f-63ec-49c5-b6a0-98781976205b)

### 2. ☁️ Creazione dell'istanza

- Dopo l'accesso, vai su **Navigation Menu** > **Compute** > **Instances**
- Clicca su **Create Instance**
  ![image](https://github.com/user-attachments/assets/ea6646e6-fe61-460d-bf33-0e8828a8c090)

### 3. 📋 Configurazione di base

- Nella sezione **Basic Information**:
  - Inserisci un **nome** per la tua istanza
  - Clicca su **Change Image**
    - Seleziona **Ubuntu Server Minimal 22.04**
    - Scegli l'**architettura** in base al tipo di istanza che desideri (es. AMD64 o ARM)
    - La **Shape** (configurazione hardware) verrà aggiornata automaticamente
  ![image](https://github.com/user-attachments/assets/37ce9740-0fa8-40a9-94d9-695e457b3f13)
  ![image](https://github.com/user-attachments/assets/929359db-aa05-4298-8c97-8ff74003cda4)

### 4. ⚙️ Configurazione della Shape

- **Per istanze AMD (x86)**: lascia la **Shape** predefinita
- **Per istanze ARM (Ampere A1)**:
  - Clicca su **Change Shape**
    - Configura:
      - Numero di OCPU: max **4 totali** per account Free Tier
      - Quantità di RAM: max **24 GB totali** per account Free Tier
    ![image](https://github.com/user-attachments/assets/0d68b154-6a4a-47d3-b119-f01abba63baf)
    ![image](https://github.com/user-attachments/assets/49378b98-312c-436a-a77c-0bff337ebc3d)

### 5. 🔐 Security (opzionale)

- Salta la sezione **Add SSH Keys for Root Compartment**
  - Non modificare nulla
  ![image](https://github.com/user-attachments/assets/802a82e1-a3bb-455e-a5fc-ec59f8bdc425)

### 6. 🌐 Networking

- Espandi la sezione **Advanced Options**
- Inserisci un **hostname** per la tua istanza
- In **Add SSH Keys**:
  - Seleziona **Generate SSH Key Pair**
  - Scarica e **conserva in modo sicuro** la chiave privata (`.pem`) e pubblica
  ![image](https://github.com/user-attachments/assets/b69a7447-2546-4ea3-bfd9-c3f6c68512d2)
  ![image](https://github.com/user-attachments/assets/1ff4df24-0dfd-4708-9a75-5c9d6a573afa)

### 7. 💾 Storage

- Salta la sezione **Boot Volume and Storage**
  - Le impostazioni predefinite sono sufficienti
  ![image](https://github.com/user-attachments/assets/37ce66dd-d3c4-455b-b26a-0196fdc58f36)

### 8. 🔍 Review e Creazione

- Rivedi le impostazioni nella sezione **Review**
- Se tutto è corretto, clicca su **Create**

Dopo pochi minuti l'istanza sarà attiva e pronta all'uso.
Puoi connetterti via SSH utilizzando la chiave privata che hai scaricato

### 🔗 Risorse utili

- [Documentazione ufficiale OCI](https://docs.oracle.com/en-us/iaas/Content/home.htm)
- [Come connettersi via SSH](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/accessinginstance.htm)

---

## 🔧 Configurazione hostname statici con DuckDns

Gli IP pubblici assegnati alle istanze Oracle Cloud normalmente sono dinamici, No-IP ti permette di associare un hostname che si aggiorna automaticamente ogni volta che l'IP della tua istanza cambia. Ecco come fare:

### 1. Registrazione e login

- Vai su [https://www.duckdns.org/](https://www.duckdns.org/)
- Clicca su **Sign In With GitHub/Google/QuelloCheVipare** e crea un account gratuito.

### 2. Creazione del Sottodominio (Dynamic DNS)

- Dopo il login, vi ritroverete direttamente nella **Dashboard** di DuckDns. Notate nella sezione in alto il vostro token identificatvo che utilizzeremo nel .env del container duckdns.
- Inserisci il nome host, ad esempio:

  - `stremio-mario`
  
  Scegli un nome unico che ti permetta di riconoscerlo facilmente.
- Premi **Add Domain**.
- Nel campo **Current IP**, vedrai il tuo IP pubblico attuale (se errato, correggilo con quello giusto).

![image](https://github.com/user-attachments/assets/7825a4dc-021e-49c7-908d-cf9d43cba0e0)

---

## 🌐 Configurazione DNS globale (Cloudflare, Quad9, ecc.)

Se il tuo sistema utilizza systemd-resolved (come avviene per default in Ubuntu e derivati), non modificare direttamente il file /etc/resolv.conf, perché viene gestito automaticamente.

1. Apri il file di configurazione:

```bash
sudo nano /etc/systemd/resolved.conf
```

2. Cerca la sezione [Resolve] e modifica come segue:

```text
[Resolve]
DNS=1.1.1.1 1.0.0.1
FallbackDNS=9.9.9.9
DNSStubListener=yes
```

* 1.1.1.1 = Cloudflare
* 9.9.9.9 = Quad9 (opzionale fallback)

3. Riavvia il servizio:

```bash
sudo systemctl restart systemd-resolved
```

4. Verifica che i DNS siano attivi:

```bash
resolvectl status
```

---

## 📦 Docker + Docker Compose

> Questo progetto usa **Docker** per semplificare l’installazione e l’isolamento dei servizi.

### 📥 Installazione Docker

```bash
# 🔁 Rimuovi eventuali versioni precedenti
sudo apt remove docker docker-engine docker.io containerd runc

# 🔄 Aggiorna l’elenco dei pacchetti
sudo apt update

# 📦 Installa i pacchetti richiesti per aggiungere il repository Docker
sudo apt install -y ca-certificates curl gnupg lsb-release

# 🗝️ Aggiungi la chiave GPG ufficiale di Docker
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/$(. /etc/os-release; echo "$ID")/gpg | \
  sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

# 📥 Aggiungi il repository Docker alle fonti APT
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/$(. /etc/os-release; echo "$ID") \
  $(lsb_release -cs) stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# 🐳 Installa Docker, Docker Compose e altri componenti
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# 👤 Aggiungi il tuo utente al gruppo docker
sudo usermod -aG docker $USER

# ⚠️ Per applicare il cambiamento, esegui il logout/login oppure:
newgrp

# ✅ Verifica che Docker funzioni correttamente
docker run hello-world
```

---

## 🚀 Avvio del progetto dal repository GitHub

Il progetto è contenuto in un repository GitHub che include un file `docker-compose.yml` preconfigurato. Alcuni servizi costruiranno automaticamente le immagini Docker a partire da Dockerfile remoti ospitati su GitHub.

### 🔧 Prerequisiti

Assicurati di avere:
- Docker e Docker Compose installati (vedi sezione precedente)
- Git installato (`sudo apt install git` se non lo hai)


### 📥 Clona il repository

```bash
cd ~
git clone https://github.com/tuo-utente/tuo-repo.git
cd tuo-repo
```

### 🌐 Crea la rete Docker esterna proxy
Se non l'hai già fatto, crea la rete che verrà utilizzata da Nginx Proxy Manager e dagli altri container per comunicare tra loro:

```bash
docker network create proxy
```
>🔁 Questo comando va eseguito una sola volta. Se la rete esiste già, Docker mostrerà un errore che puoi ignorare in sicurezza.

### 🛠️ Creazione dei file .env per MammaMia, MediaFlow Proxy, StreamV, AIOStreams e docker-duckdns
In ogni sotto cartella di questo progetto è presente un file .env_example con tutte le chiavi necessarie per il corretto funzionamento dei vari moduli.
Per ogni modulo copiare e rinominare il file .env_example in .env. I vari .env dovranno essere modificati in base alle vostre specifiche configurazioni.

**1. .env per MammaMia**
Per configurare il plugin MammaMia è necessario configurare il relativo file .env. Vi rimando al repo del progetto per i dettagli.

📄 Esempio: ./mammamia/.env
```text
# File .env per il plugin mammamia
TMDB_KEY=xxxxxxxxxxxxxxxx
PROXY=["http://xxxxxxx-rotate:xxxxxxxxx@p.webshare.io:80"]
FORWARDPROXY=http://xxxxxxx-rotate:xxxxxxxx@p.webshare.io:80/
```

**2. .env per MediaFlow Proxy**
Per configurare il modulo Media Flow Proxy è necessario configurare il relativo file .env. Vi rimando al repo del progetto per i dettagli.

📄 Esempio: ./mfp/.env
```text
API_PASSWORD=password
TRANSPORT_ROUTES={"all://*.ichigotv.net": {"verify_ssl": false}, "all://ichigotv.net": {"verify_ssl": false}}
```

**3. .env per StreamV**
Per configurare il plugin StreamV è necessario configurare il relativo file .env. Vi rimando al repo del progetto per i dettagli.

📄 Esempio: ./streamv/.env
```text
TMDB_API_KEY="xxxxxxxxxxxxxxxx"
MFP_PSW="xxxxxxxxx"
MFP_URL="https://mfp.stremio-mario.duckdns.org"
BOTHLINK=true
```

**4. .env per AIOStreams**
Per configurare il plugin AIOStreams è necessario configurare il relativo file .env. Vi rimando al repo del progetto per i dettagli.

📄 Esempio: ./AIOStreams/.env
```text
#queste sono le impostazioni minime per il corretto funzionamento del plugin
ADDON_ID="aiostreams.stremio-mario.duckdns.org"
BASE_URL=https://aiostreams.stremio-mario.duckdns.org
SECRET_KEY=36148382b90f80430d69075df9848eee87032d16fc4c03fe9ca7ce53b7028973  (può essere generata con openssl rand -hex 32)
ADDON_PASSWORD=password_a_scelta
```

**5. .env per DuckDNS Updater**
Per configurare correttamente il client DDNS, è necessario un file .env contenente le credenziali e il sottodominio associati al tuo account DuckDns.

📄 Esempio: ./duckdns-updater/.env
```text
# File .env per il client DDNS
SUBDOMAINS=stremio-mario
TOKEN=IL_TUO_TOKEN
TZ=Europe/Rome
```

🛑 Attenzione alla sicurezza: imposta i permessi del file .env in modo che sia leggibile solo dal tuo utente, ad esempio:

```bash
chmod 600 ./duckdns-updater/.env
```

🔁 Ricorda di sostituire:
IL_TUO_TOKEN → con il token visibile sulla Dashboard di DuckDns.
SUBDOMAINS → con il sottodominio specifico (senza .duckdns.org).


### 🏗️ Build delle immagini e avvio dei container

Per buildare le immagini (se definite tramite build: con URL GitHub) e avviare tutto in background:

```bash
docker compose up -d --build
```
> 🧱 Il flag --build forza Docker a scaricare i Dockerfile remoti ed eseguire la build, anche se l'immagine esiste già localmente.

### 🔍 Verifica che tutto sia partito correttamente

```bash
docker compose ps
```

Puoi anche consultare i log con:

```bash
docker compose logs -f
```

🔁 Aggiornare il repository e ricostruire tutto (quando aggiorni da GitHub)
```bash
git pull
docker compose down
docker compose up -d --build
```

---

## 🔐 Configurazione Proxy Hosts e gestione SSL con Nginx Proxy Manager (NPM)

Per rendere accessibili le tue applicazioni web da internet in modo sicuro, useremo **Nginx Proxy Manager (NPM)**. Questo tool semplifica la gestione dei proxy inversi e automatizza l’ottenimento dei certificati SSL con Let’s Encrypt.

### 1. Creazione del sottodominio su DuckDns

Assicurati di aver creato il sottodominio/hostname statico su [**duckdns.org**](https://www.duckdns.org) e che punti al tuo IP pubblico (anche se dinamico, aggiornato tramite l’agent docker-duckdns)::

- `stremio-<tuo-id>.duckdns.org`

> 🔔 **Suggerimento:** Usa un identificativo unico (`<tuo-id>`) per evitare conflitti con altri utenti DuckDns.

### 2. 🔧 Modifica delle regole di firewall per abilitare le porte 80, 443, 8080 su Oracle Cloud Platform

1. Vai su **Navigation Menu** > **Networking** > **Virtual Cloud Networks** ![image](https://github.com/user-attachments/assets/e0aa996c-8577-4dd6-ab4b-03798384e68b)

2. Seleziona la **VCN** associata alla tua istanza
3. Vai al tab **Security** e clicca su **Default Security List for vcn-XXXXXXX**
4. Passa al tab **Security Rules**
5. In **Ingress Rules**, clicca su **Add Ingress Rules**
   - **Source CIDR**: `0.0.0.0/0`
   - **IP Protocol**: `TCP`
   - **Source Port Range**: `All`
   - **Destination Port Range**: `80, 443, 8080`
   ![image](https://github.com/user-attachments/assets/2fcb6879-9337-43d9-b4cc-970dd1828e80)

Salva le modifiche. Ora la tua istanza potrà ricevere connessioni su HTTP/HTTPS.
![image](https://github.com/user-attachments/assets/b021f0d7-af41-4a84-8175-2d432c9cc066)

> Questo consente a Let’s Encrypt di verificare il dominio e rilasciare i certificati.

### 3. Configurazione dei proxy host in Nginx Proxy Manager

>🛑 Attenzione!!! Prima di iniziare assicuratevi che http://stremio-mario.duckdns.org/ vi riporti alla welcome page di Nginx Proxy Manager.

#### Creazione Certificato di tipo Wildcard
Andremo a generare un certificato rilasciato da Let’s Encrypt, che Nginx Proxy Manager (NPM) rinnoverà automaticamente prima della scadenza. Useremo un singolo certificato di tipo wildcard, valido per *.stremio-mario.duckdns.org, sfruttando la challenge DNS con le API di DuckDNS.

- **accedi ad http://<ip-tuo-server>:8080** (al primo accesso le credenziali di default sono **Email: admin@example.com Password: changeme**. Vi verrà chiesto di modificarle)
- Dalla barra di menu **SSL Certificates** → **Add SSL Certificate**
- Inserisci i due domini:
  - stremio-mario.duckdns.org
  - *.stremio-mario.duckdns.org
- Seleziona Use a DNS Challenge, scegli DuckDNS come provider e inserisci il tuo Token API.
- Inserisci un indirizzo email valido per la registrazione SSL
- Imposta Propagation Seconds su 300 per sicurezza.
- Accetta i Termini di Let’s Encrypt e clicca su Save.

![image](https://github.com/user-attachments/assets/876629f3-8ac3-41cb-b493-20646f6209a8)

>⚠️ NPM + DuckDNS a volte fallisce: alcuni utenti segnalano errori nel DNS challenge su DuckDNS . Tuttavia, molte guide e utenti confermano che con i settaggi corretti funziona.. a me ha funzionato... quindi provate, provate e provate :)

Se il certificato viene correttamente generato dovreste vedere lo stato dello stesso a **Disabled**, questo perchè non risulta ancora associato a nessun Proxy Host.

![image](https://github.com/user-attachments/assets/524c3b4a-cc57-42c1-a615-04fa69d57cce)

#### Creazione Proxy Host
Per ogni applicazione, crea un nuovo **Proxy Host** in NPM seguendo questi passi:

- **Dalla barra di menu selezionate **Hosts** → **Proxy Hosts** → **Add New Proxy**
- **Domain Names:** inserisci l’hostname corrispondente (es. `mammamia.stremio-<tuo id>.duckdns.org`)
- **Scheme:** `http`
- **Forward Hostname / IP:** il mome del servizio cosi come configurato nel docker-compose ovvero mammmia, mediaflow_proxy e streamv
- **Forward Port:** la porta interna dove l’app è in ascolto (es. `8080` per Mammamia, `8888` per mediaflow_proxy, `7860` per streamv e `3000` per aiostreams)
- Abilita le seguenti opzioni:
  - **Block Common Exploits**
  - **Websockets Support** (se necessario)
  - **Enable HSTS** (opzionale, aumenta la sicurezza)
  ![image](https://github.com/user-attachments/assets/4c86d022-ce56-4f6c-b6b1-3d7a53aad9de)

- **SSL tab:** seleziona:
  - **Enable SSL**
  - **Force SSL**
  - **HTTP/2 Support**
  - In **SSL certificate** il certificato creato in precedenza
  ![image](https://github.com/user-attachments/assets/bfab6476-200f-455e-9213-c013ce89ad78)

- **Nel tab Advanced** aggiungete queste configurazioni :

  ```text
  proxy_set_header Host $host;
  proxy_set_header X-Real-IP $remote_addr;
  proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
  proxy_set_header X-Forwarded-Proto $scheme;
  proxy_pass_request_headers on;
  ```
  ![image](https://github.com/user-attachments/assets/92fea31d-bb8c-49c5-a887-f3d5486c9f7f)

Ripeti questa configurazione per ciascuno dei tre hostname con la rispettiva porta (ad esempio, `mfp.stremio-<tuo-id>.duckdns.org` → porta `8888`, ecc.).

### 4. Verifica e manutenzione

- Dopo aver configurato i proxy host, prova ad accedere agli URL pubblici via browser.
- NPM gestirà automaticamente il rinnovo dei certificati SSL.
- Assicurati che il tuo router sia sempre configurato correttamente per il port forwarding, specialmente dopo eventuali riavvii o aggiornamenti firmware.

---


