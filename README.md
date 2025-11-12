# proxy


1. Proxy si ProxyChains

1.1. Ce este un Proxy?

Un server prox este un server intermediar care actioneaza ca un punct de legatura intre intre un client ( dispozitivul nostru ) si un server destinatie ( site-ul sau resura dorita )

Cum functioneaza un Proxy?

Traficul nostru este re-rutat:

1. Cazul fara Proxy: Clientul (IP: 1.1.1.1) solicita site-ul web (IP: 9.9.9.9). Site-ul vede request-ul IP-ului real: 1.1.1.1
<img width="1071" height="426" alt="image" src="https://github.com/user-attachments/assets/fe97425b-a729-48d7-bc0f-be1b6a5274a0" />

2. Cazul cu Proxy: Clientul (IP: 1.1.1.1) solicita Proxy-ul (IP: 2.2.2.2). Proxy-ul ulterior, folosind propria sa adresa (2.2.2.2), solicita site-ul web (9.9.9.9). Astfel, site-ul web inregistreaza Proxy-ul (2.2.2.2) ca sursa a cererii, iar IP-ul real al clientului (1.1.1.1) ramane mascat.


1.2. Ce este un ProxyChain?

Acum ca am inteles cum functioneaza un singur Proxy, putem sa trecem la o notiune putin mai complexa.
Un lant de proxy-uri, reprezinta o succesiune de servere Proxy, prin care trece traficul de la sursa la destinatie.


2. Tipuri de Proxy-uri si lanturi.

2.1. Tipuri de Proxy-uri:
| Tip | Nivel OSI | Nivel de anonimitate |
|-----| --------- | -------------------- |
| `HTTP` | Nivelul 7 (Aplicatie) | Scazut |
| `HTTPS` | Nivelul 7 (Aplicatie) | Scazut-Mediu |
| `SOCKS4` | Nivelul 5 (Sesiune) | Mediu-Ridicat |
| `SOCKS5` | Nivelul 5 (Sesiune) | Ridicat |


`HTTP` - Fiind la nivel de aplicatie, acesta este obligat sa citeasca si sa manipuleze datele (headerele HTTP) pentru a usura comunicarea si a indeplini functii precum caching-ul. Aceasta manipulare de headere permite inserarea accidentala sau intentionata a unor campuri precum: "X-Forwarded-For", care dezvaluie adresa IP reala a clientului serverului destinatie.

`HTTPS`- Acesta stabileste un tunel criptat (SSL/TLS) intre client si proxy. Chiar daca traficul este securizat pe acest segmen, el opereaza asemenea HTTP la nivelul 7 (Aplicatie), avand un risc similar de a trada identitatea prin headere de identificare (X-Forwarded-For).

`SOCKS4` - Fiind la nivel de sesiune, Proxy-ul SOCKS nu este conceput pentru a intelege sau a inspecta protocoalele de nivel superior (precum HTTP). La nivelul 5, SOCKS se ocupa doar de stabilirea si terminarea sesiunii de comunicare. Acesta nu se intereseaza de continutul pachetelor precum HTTP, asadar nu are nici un motiv sa citeasca sau sa modifice headerele HTTP, ceea ce contribuie la o anonimitate mai mare.

`SOCKS5` - In timp ce SOCKS4 se limiteaza doar la conexiunile bazate pe TCP si nu suporta autentificarea sau pachetele UDP, SOCKS5 este versiunea avansata ce permite autentificarea utilizatorilor, suporta si UDP (esential pentru prevenirea DNS Leaks) si are suport pentru IPv6.

2.2. Tipuri de lanturi (ProxyChains)

| Tip | Mod de functionare | Configurare | Plusuri (+) | Minusuri (-) |
|-----| --------- | -------------------- |
| `Dynamic` | Foloseste toate proxy-urile functionale din lista, sarind automat peste cele care nu raspund | `dynamic_chain` | Foarte optim. Nu esueaza din cauza unui proxy inactiv. Are viteza buna. | Anonimitate usor mai scazut decat `strict_chain` |
| `Strict` | Foloseste toate proxy-urile in ordinea exacta specificata in lista | `strict_chain` | Anonimitate maxima garantata prin lungimea lantului (in conditiile in care fiecare proxy functioneaza | Risc mare de esec. Daca un singur proxy nu functioneaza, intreaga conexiune esueaza |
| `Random` | Alege aleatoriu un anumit numar de proxy-uri din lista pentru fiecare conexiune noua | `random_chain` | Ideal pentru a evita detectia bazata pe o sursa IP fixa (foarte bun pentru penetration testing) | Nu garanteaza lungimea lantului. Poate folosi un singur proxy (daca chain_len nu este setat) |


3. Utilizare practica in Kali Linux

3.1. Configurare ProxyChains4

ProxyChains4 este un tool pre-instalat pe Kali Linux. Ce trebuie sa facem pentru a utiliza proxy-uri, sunt urmatorii pasi:

1. Localizare proxychains4:
<img width="266" height="54" alt="image" src="https://github.com/user-attachments/assets/481c6744-52a3-4c13-a4ed-3d9957461a0c" />

2. Configurarea proxychains4:

Dupa ce am deshis proxychains4.conf putem observa urmatoare setari by default:
<img width="113" height="23" alt="image" src="https://github.com/user-attachments/assets/d87b1df0-a229-4148-8e54-fe7e470b42af" />
Tipul de lant folosit by default este `strict_chain`, dar daca vrem sa folosim alt tip putem doar sa comentam linia respectiva pentru a nu mai fi activ.
<img width="499" height="155" alt="image" src="https://github.com/user-attachments/assets/1b41aaf1-fd21-43d3-a948-2bb63645f577" />
Spre exemplu, am ales sa folosesc lantul de `tip dynamic_chain`

<img width="90" height="24" alt="image" src="https://github.com/user-attachments/assets/aff2042e-a73f-4626-ac61-f726cfd88d25" />
Vedem ca proxy_dns nu este comentat, asa dar este activ.

Ce face proxy_dns?

`proxy_dns` - Forteaza toate cererile de rezolvare DNS (Domain Name System) sa treaca prin lantul de proxy-uri inainte de a ajunge la serverele DNS externe. In cazul in care am dezactiva `proxy_dns`:
- ISP-ul ar vedea exact ce nume de domeniu incercam sa accesam
- ISP-ul ar vedea adresa noastra IP reala ca sursa a cererii
Chiar daca traficul ulterior rece prin ProxyChains, actiunea initiala de cautare a DNS compromite anonimitate. Asadar, pentru o securitate buna, `proxy_dns` trebuie sa fie activat.

----
<img width="257" height="64" alt="image" src="https://github.com/user-attachments/assets/753893d9-4efb-4cb3-9e61-7fb838cbca23" />

`tcp_read_time_out` (15 secunde) - Reprezinta timpul maxim pe care lantul nostru proxy il asteapta pentru a primi date de la un proxy activ.
`tcp_connect_time_out` (8 secunde) - Reprezinta timpul maxim pe care lantul nostru proxy il asteapta pentru a stabili o conexiune (handshake-ul initial).

Timpii de asteptare (`tcp_read_time_out` si `tcp_connect_time_out`) ar trebui ajustati in functie de calitatea si performanta serverelor proxy folosite.

- Proxy-uri publice (Performanta scazuta) - Proxy-urile gratuite sunt adesea foarte lente si instabile, asadar valori mari sunt necesare (Exemplu: 8000ms-150000ms). Acestea ofera serverelor suficient timp pentru a raspunde, crescand astfel sansa ca lantul `dynamic_chain` sa gasesasca o ruta pe care o putem utiliza.

- Proxy-uri Private/Platite (Performanta ridicata) - Proxy-urile private/platite, fata de cele gratuite, ar trebui sa raspunda rapid, asadar este recomandata reducerea valorilor (Exemplu: Sub 8000ms). Setarea unui *timeout* scurt permite lantului `dynamic_chain` sa detecteze si sa sara peste proxy-urile inactive mai rapid, optimizand astfel viteza generala a unui lant.





<img width="191" height="90" alt="image" src="https://github.com/user-attachments/assets/538be262-e943-4d10-ad89-5f8600724083" />

Ultimul lucru setat by default este `ProxyList`. Acesta este locul unde trebuie sa adaugam lista de proxy-uri pe care le vom folosi pentru a construi lantul. Linia afisata in imagine `socks4 127.0.0.1 9050` este pentru a folosi nodul local al retelei Tor, daca acesta ruleaza pe sistemul nostru.

<img width="673" height="306" alt="image" src="https://github.com/user-attachments/assets/671de5ec-c15d-4270-8538-7865fc8c2439" />

Aceasta sectiune din fisierul de configurare proxychains4.conf explica formatul corect de adaugare a unui proxy in lista. Pentru a adauga conform exemplelor trebuie sa avem: type IP port [user] [pass]

type - Protocolul proxy (http,https,socks4,socks5).
IP - Adresa IPv4 sau IPv6 (doar in cazul in care folosim protocolul `SOCKS5`) a serverului proxy.
port - Portul pe care asculta (ruleaza activ si monitorizeaza acel port pe adresa IP a serverului proxy) serverul proxy.
[user] & [pass] (Optional) - Datele de autentificare necesare daca proxy-ul este privat.



I. Metoda lantului Tor:
Pentru a testa un lant proxy, o sa folosesc reteaua Tor (The Onion Router) ca prim nod al lantului pentru o baza buna de anonimitate.

<img width="196" height="21" alt="image" src="https://github.com/user-attachments/assets/e10feb5a-4c5e-44f4-a5a4-98af35f4a147" />

1. Am schimbat protocolul default SOCKS4, in SOCKS5 si am salvat modificarile.
2. Instalam si rulam: Dupa ce Tor este instalat pe sistemul nostru, el trebuie pornit: 
<img width="249" height="46" alt="image" src="https://github.com/user-attachments/assets/ef0fd842-079c-4c6d-a1c6-4fda8e48fa83" />

3. Acum ca este pornit, putem folosi rutarea pe 3 nivele: Traficul trimis catre acest nod local este preluat de Tor, criptat si rutat aleatoriu prin cel putin 3 noduri din reteaua Tor (Guard, Middle, Exit) inainte de a ajunge la internetul public.

Nodul de intrare (Guard):
- Rol: Acesta este primul nod al circuitului Tor cu internetul.
- Anonimitate: Nodul Guard este singurul din circuit care cunoaste adresa noastra IP reala.
- Date vazute: Vede IP-ul nostru IP-ul nodului urmator (Middle Node)

Nodul Intermediar (Middle Node):
- Rol: Acesta este nodul din mijlocul circuitului, care are rolul de a "sparge" legatura directa dintre sursa (Guard) si destinatie (Exit).
- Anonimitate: Nu cunoaste IP-ul nostru real, nici al destinatiei finale. Stie doar IP-urile nodurilor Guard si Exit.
- Date vazute: Vede IP-ul nodului Guard si IP-ul nodului Exit.

Nodul de iesire (Exit Node):
- Rol: Acesta este ultimul nod al circuitului Tor. Nodul Exit este cel care decripteaza ultimul strat al pachetului si trimite cererea catre serverul destinatie (site-ul web)
- Anonimitate - Serverul destinatie vede IP-ul nodului Exit ca fiind sursa cererii.
- Date vazute: Vede IP-ul nodului Middle si IP-ul serverului destinatie.


Ce face Tor diferit fata de un ProxyChain pe care l-am crea noi?

Lantul creat de noi:
- Protectie: Este bazat pe incredere in operatorul proxy-ului.
- Risc de interceptare: Ridicat, deoarece traficul este clar intre noduri.

Lantul Tor:
- Protectie: Este bazat pe criptografie.
- Risc de interceptare: Scazut, deoarece traficul este criptat intre noduri.

In timp ce un lant Tor ofera o protectie superioara prin criptarea garantata pe straturi si separarea logica a cunostintelor intre cele trei noduri ale circuitului, lantul de proxy-uri pe care il cream ofera anonimitate doar prin complexitatea urmaririi (numarul de *hops*), asta fiind reprezentata de numarul de proxy-uri pe care il folosim in lista.

4. Verificam daca Tor ruleaza.
<img width="687" height="167" alt="image" src="https://github.com/user-attachments/assets/dc3212c9-a823-4c20-a248-fb19af257c42" />

Dupa cum putem observa, `Active: active`, semn ca suntem gata de urmatorul pas.

5.Verificarea finala a lantului
Am folosit comanda `proxychains4 curl ifconfig.me`, pentru a face o cerere HTTP folosind curl, la serviciul web `ifconfig.me` ce are rolul unic de a returna adresa IP a sursei care a facut cererea.
<img width="649" height="131" alt="image" src="https://github.com/user-attachments/assets/6ff80b17-6097-4a94-ad04-0af5687da702" />

<img width="844" height="605" alt="image" src="https://github.com/user-attachments/assets/ab78befe-26ef-42c1-a72c-bc4a033f2358" />

Din cate putem observa, a functionat! Verificarea IP-ului de iesire `37.114.50.27` pe un serviciu de geolocatie confirma ca traficul este emis pe internet de nodul de Exit Tor dintr-o locatie diferita de sursa reala.


II. Metoda lantului creat de noi.

1. Pentru a crea un lant nou de proxy-uri, o sa cautam cateva servicii gratuite de proxy-uri. Am ales proxy-uri de tip SOCKS4 furnizate de site-ul sys.one si apoi le-am verificat utilizand checker-ul online hide.mn

<img width="775" height="385" alt="image" src="https://github.com/user-attachments/assets/0563d38b-9171-426c-8768-722e5a33a8f3" />

<img width="816" height="556" alt="image" src="https://github.com/user-attachments/assets/466882ac-01fe-42b9-aec9-79c881862e09" />



Am adaugat proxy-urile functionale in fisierul de configurare `proxychains4.conf` respectand formatul `type ip port`:
<img width="250" height="75" alt="image" src="https://github.com/user-attachments/assets/ffbc1c7a-827a-4681-8f6b-db6239c38c5e" />

Am rulat testul pentru a verifica functionalitatea lantului: `proxychains4 curl ifconfig.me`
<img width="1189" height="138" alt="image" src="https://github.com/user-attachments/assets/c02eb78e-c3c4-4eb8-b54f-aa8f5a628743" />

Output-ul final demonstreaza ca lantul nostru este functional!



