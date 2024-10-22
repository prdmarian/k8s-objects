Documentație completă pentru Configurarea Clusterului Kubernetes cu Prometheus, Grafana, și Aplicațiile tale
Această documentație acoperă configurațiile care au fost efectuate în clusterul Kubernetes pentru monitorizarea și gestionarea resurselor, împreună cu aplicațiile care rulează în mod intern și extern, folosind Helm, Prometheus, Grafana, și alte componente. De asemenea, se oferă detalii despre conexiunile între aplicații și sugestii pentru extinderi viitoare.

1. Structura Generală a Proiectului
Proiectul a fost structurat astfel încât fiecare aplicație să fie gestionată printr-un Helm Chart specific, stocat în directoare dedicate, cu configurații de tip ArgoCD Application pentru automatizarea deployment-ului. De asemenea, toate valorile utilizate pentru aplicații sunt gestionate separat într-un repository Git extern.

Structura proiectului:
kotlin
Copy code
├── applications/
│   ├── letsencrypt.yaml
│   ├── prometheus.yaml
│   ├── grafana.yaml
│   ├── custom-exporter.yaml
│   ├── git-cache.yaml
│   ├── helm-cache.yaml
│   ├── ldap.yaml
│   ├── web-apps.yaml
│   ├── ingress-proxy-internal.yaml
│   ├── ingress-proxy-external.yaml
├── argocd/
│   ├── values.yaml
├── monitoring/
│   ├── prometheus/
│   │   ├── values.yaml
│   ├── grafana/
│   │   ├── values.yaml
├── letsencrypt/
│   ├── values.yaml
├── tools/
│   ├── ldap/
│   │   ├── values.yaml
│   ├── git-cache/
│   │   ├── values.yaml
│   ├── helm-cache/
│   │   ├── values.yaml
├── web-apps/
│   ├── values.yaml
├── ingress-proxy-internal/
│   ├── values.yaml
├── ingress-proxy-external/
│   ├── values.yaml
2. Scopul fiecărei componente și motivele configurării
A. Prometheus și Grafana (Namespace: monitoring)
Prometheus este utilizat pentru monitorizarea întregului cluster Kubernetes, colectând date din noduri, namespace-uri, poduri și alte resurse. Grafana preia aceste date și le vizualizează în dashboard-uri predefinite.

De ce a fost adăugat Prometheus:
Monitorizează toate resursele din cluster (noduri, namespace-uri, poduri, storage).
Oferă vizibilitate asupra stării de sănătate și performanței clusterului.
De ce a fost adăugat Grafana:
Vizualizează metricile preluate de Prometheus.
Folosește dashboard-uri predefinite pentru monitorizarea Kubernetes.
Conexiunea între Prometheus și Grafana:
Prometheus colectează metrici și le expune către Grafana, care este configurat să utilizeze Prometheus ca sursă de date. Astfel, Grafana creează dashboard-uri interactive pentru vizualizarea performanței clusterului.
Ce a fost configurat:
Ephemeral Storage a fost configurat pentru Prometheus pentru stocarea temporară a datelor. Există o configurare comentată pentru Persistent Volume, în caz că vrei să stochezi datele pe termen lung în viitor.
În Grafana, a fost configurat Prometheus ca sursă de date, și s-au adăugat dashboard-uri preconfigurate din Grafana.com pentru monitorizarea Kubernetes.
Ce poți adăuga în viitor:
Persistent Volume pentru Prometheus, pentru a păstra datele de monitorizare mai mult timp.
Alte dashboard-uri personalizate pentru aplicații specifice (ex: monitorizarea aplicațiilor web).
B. ArgoCD (Namespace: argocd)
ArgoCD este folosit pentru automatizarea deployment-urilor în cluster, gestionând aplicațiile prin intermediul fișierelor de tip Application. Acesta permite o sincronizare automată a configurațiilor din Git, asigurând că toate aplicațiile sunt mereu în starea dorită.

De ce a fost adăugat ArgoCD:
Automatizează deployment-ul aplicațiilor din Git.
Sincronizează automat configurările și permite gestionarea aplicațiilor dintr-o interfață centralizată.
Conexiunea între aplicații și ArgoCD:
Fiecare aplicație (ex: Prometheus, Grafana, LDAP, etc.) are propriul fișier de tip Application, iar ArgoCD sincronizează automat fișierele de configurare din Git și deployment-urile în Kubernetes.
Ce poți adăuga în viitor:
Alte aplicații care să fie gestionate prin ArgoCD.
Politici de sincronizare avansate pentru rollback și deployment-uri canar.
C. Ingress-Proxy (Namespaces: ingress-proxy-internal și ingress-proxy-external)
Sunt două tipuri de ingress configurate:

Ingress Proxy External: folosit pentru aplicațiile care vor fi accesibile extern, cum ar fi aplicațiile din web-apps.
Ingress Proxy Internal: folosit pentru aplicațiile care vor fi accesate doar intern, cum ar fi Prometheus, Grafana, LDAP și ArgoCD.
De ce a fost adăugat Ingress:
Permite accesul la aplicații interne și externe prin URL-uri DNS configurate corespunzător.
Conexiunea între ingress și aplicații:
Aplicațiile interne (Prometheus, Grafana, LDAP) sunt expuse doar intern prin intermediul Ingress Proxy Internal.
Aplicațiile externe (cum ar fi viitoarele aplicații din web-apps) sunt expuse prin Ingress Proxy External pentru a fi accesibile din afara clusterului.
Ce poți adăuga în viitor:
Alte namespace-uri externe, cum ar fi web-apps2, pot fi expuse prin Ingress Proxy External.
Alte certificate gestionate automat prin Let’s Encrypt.
D. Cert-Manager (Let’s Encrypt) (Namespace: letsencrypt)
Cert-Manager este utilizat pentru gestionarea automată a certificatelor SSL, asigurând că toate aplicațiile expuse extern au un certificat valid.

De ce a fost adăugat Cert-Manager:
Automatizează generarea și reînnoirea certificatelor SSL pentru aplicațiile externe.
Conexiunea între Cert-Manager și Ingress Proxy:
Cert-Manager lucrează împreună cu Ingress Proxy External pentru a genera certificate SSL pentru aplicațiile accesibile din exterior.
Ce poți adăuga în viitor:
Alte servicii externe care necesită SSL, cum ar fi alte web-app-uri sau servicii publice.
E. LDAP, Git-Cache și Helm-Cache (Namespace: tools)
Aceste aplicații sunt instrumente de suport pentru dezvoltare și gestionarea identităților.

LDAP: Servește pentru autentificarea și gestionarea utilizatorilor.
Git-Cache: Optimizează gestionarea repository-urilor Git pentru deployment-uri rapide.
Helm-Cache: Cache pentru gestionarea pachetelor Helm.
Ce poți adăuga în viitor:
ElasticSearch și Logstash pentru gestionarea și analiza log-urilor.
3. Recomandări pentru viitor
Pe viitor, poți extinde această infrastructură pentru a include:

Persistent Volume pentru Prometheus: Stocarea pe termen lung a datelor de monitorizare pentru o analiză detaliată.
ElasticSearch + Logstash: O soluție puternică de colectare și analiză a log-urilor pentru aplicațiile din cluster.
Extinderea aplicațiilor expuse extern: Dacă vei adăuga mai multe aplicații în namespace-uri noi (ex: web-apps2), acestea pot fi expuse prin Ingress Proxy External și pot beneficia de certificate SSL generate automat de Let’s Encrypt.
Alte aplicații gestionate prin ArgoCD: Orice altă aplicație sau componentă poate fi adăugată și gestionată automat cu ArgoCD.
Aceasta este documentația completă a ceea ce s-a făcut în proiect, explicând de ce au fost realizate anumite configurări și cum sunt conectate componentele între ele