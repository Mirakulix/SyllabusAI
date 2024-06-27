`docker-compose.yml`
```yaml
version: '3.7'

services:
  db:
    image: postgres
    environment:
      POSTGRES_DB: mydatabase
      POSTGRES_USER: myuser
      POSTGRES_PASSWORD: mypassword
    volumes:
      - postgres_data:/var/lib/postgresql/data

  web:
    build: .
    command: bash -c "python manage.py migrate && python manage.py runserver 0.0.0.0:8000"
    volumes:
      - .:/app
    ports:
      - "8000:8000"
    environment:
      SCRAPERAPI_KEY: your_scraperapi_key
      OPENAI_API_KEY: your_openai_api_key
      DATABASE_URL: postgres://myuser:mypassword@db:5432/mydatabase
    depends_on:
      - db

volumes:
  postgres_data:



Um das Projekt in ein strukturiertes Design-Dokument zu überführen, lassen Sie uns zunächst die einzelnen Bereiche nacheinander besprechen und sicherstellen, dass wir alle notwendigen Anforderungen und Designentscheidungen abgedeckt haben.

Beginnen wir mit dem Hintergrund des Projekts.

Hintergrund

Dieses Projekt zielt darauf ab, eine Django-basierte Webanwendung zu erstellen, die sowohl ein Frontend als auch ein Backend umfasst. Die Anwendung ermöglicht es Benutzern, URLs von Webseiten einzugeben, deren Texte extrahiert und mithilfe der OpenAI-API zusammengefasst werden. Die Anwendung bewertet die Textähnlichkeit und speichert die Ergebnisse in einer PostgreSQL-Datenbank. Das gesamte System wird auf einem Kubernetes-Cluster bereitgestellt.

Könnten Sie mir bitte bestätigen, ob dieser Hintergrund genau den Zweck des Projekts beschreibt oder ob Anpassungen erforderlich sind?

Fast, es sollen Lehrinhalte von den online Beschreibungen von Lehrveranstaltungen (LV) an fremden Universitäten abgerufen und deren Texte heruntergeladen werden. Studierende können so einen Antrag stellen, in dem können sie für eine offene LV mehrere abgeschlossene LVs anrechnen aber nicht umgekehrt. Eine abgeschlossene LV kann nicht für zwei offene LVs angerechnet werden. Die Bearbeitung der Anträge selbst, werden durch MitarbeiterInnen der Hochschule geprüft und sollen für die Studierenden grundsätzlich nicht erreichbar sein. Die KI soll dann anhand der Beschreibungen der Lehrstoffinhalte der LVs vergleichen bzw. zuerst zusammenfassen und dann vergleichen und einen Score für die Anerkennung in Prozent und eine Begründung für diese Einschätzung liefern. Ein Mensch überprüft dann ob das Ergebnis der KI akzeptiert oder abgelehn werden soll oder ob dieses fehlerhaft war.

Verstanden, das ergibt ein klares Bild des Projekts. Lassen Sie uns den Hintergrund entsprechend anpassen.

### Hintergrund

Dieses Projekt zielt darauf ab, eine Django-basierte Webanwendung zu erstellen, die sowohl ein Frontend als auch ein Backend umfasst. Die Anwendung ermöglicht es Studierenden, Anträge für die Anerkennung von Lehrveranstaltungen (LV) zu stellen. Studierende geben URLs von Beschreibungen der Lehrveranstaltungen anderer Universitäten ein, deren Texte extrahiert und mithilfe der OpenAI-API zusammengefasst werden. Die Anwendung vergleicht die Inhalte der Lehrveranstaltungen und berechnet einen Anerkennungs-Score in Prozent sowie eine Begründung für diese Einschätzung. Ein Mitarbeiter der Hochschule prüft die Ergebnisse der KI und entscheidet, ob diese akzeptiert oder abgelehnt werden. Eine abgeschlossene LV kann nur für eine offene LV angerechnet werden, jedoch nicht umgekehrt. 

**Könnten Sie bitte bestätigen, ob dieser Hintergrund den Zweck des Projekts nun korrekt beschreibt, oder ob Anpassungen erforderlich sind?**

Großartig! Als nächstes werde ich die Anforderungen des Projekts zusammenstellen. Wir verwenden das MoSCoW-Prinzip (Must have, Should have, Could have, Won't have) zur Priorisierung.

### Anforderungen

**Must have:**
- Studierende müssen URLs von Beschreibungen der Lehrveranstaltungen anderer Universitäten eingeben können.
- Die Anwendung muss die Texte von diesen URLs extrahieren.
- Die Texte müssen mittels der OpenAI-API zusammengefasst werden.
- Die Anwendung muss die zusammengefassten Texte vergleichen und einen Anerkennungs-Score sowie eine Begründung liefern.
- Mitarbeiter der Hochschule müssen die Ergebnisse der KI überprüfen und eine Entscheidung treffen können (akzeptieren, ablehnen, Fehler melden).
- Eine abgeschlossene Lehrveranstaltung kann nur für eine offene Lehrveranstaltung angerechnet werden.
- PostgreSQL-Datenbank für die Speicherung aller relevanten Daten.
- Bereitstellung der Anwendung auf einem Kubernetes-Cluster.

**Should have:**
- Benutzerfreundliches Frontend für die Eingabe der URLs und Anzeige der Ergebnisse.
- Administrationsoberfläche für die Mitarbeiter zur Verwaltung der Anträge.
- Authentifizierung und Autorisierung der Benutzer (Studierende und Mitarbeiter).
- Logging und Monitoring der Anträge und Entscheidungen.

**Could have:**
- E-Mail-Benachrichtigungen für Statusänderungen der Anträge.
- Historie der Anträge und Entscheidungen.
- Unterstützung für mehrere Sprachen.

**Won't have:**
- Mobile App-Version der Anwendung.
- Offline-Unterstützung.

**Könnten Sie bitte bestätigen, ob diese Anforderungen korrekt sind oder ob Anpassungen notwendig sind?**




Perfekt. Nun, da wir den Hintergrund und die Anforderungen geklärt haben, gehen wir zum technischen Lösungsansatz über. Hier werde ich das Architekturdesign und die technischen Details beschreiben, die erforderlich sind, um die Anforderungen zu erfüllen.

### Method

#### Architekturdesign

1. **Django Backend:**
   - Verwaltung der Benutzer, Anträge und Lehrveranstaltungen.
   - Interaktion mit der PostgreSQL-Datenbank zur Speicherung und Abfrage von Daten.
   - Integration mit der OpenAI-API zur Zusammenfassung und Bewertung der Texte.
   - Bereitstellung von REST-APIs für das Frontend.

2. **Frontend:**
   - Eingabeformular für die Studierenden zur Eingabe der URLs.
   - Anzeige der Ergebnisse (zusammengefasste Texte, Anerkennungs-Score, Begründung).
   - Administrationsoberfläche für Mitarbeiter zur Überprüfung und Entscheidung der Anträge.

3. **Datenbank (PostgreSQL):**
   - Speicherung der Benutzerinformationen (Studierende und Mitarbeiter).
   - Speicherung der Anträge und zugehörigen Lehrveranstaltungen.
   - Speicherung der extrahierten und zusammengefassten Texte, sowie der Bewertungen.

4. **Kubernetes-Cluster:**
   - Bereitstellung und Skalierung der Django-Anwendung.
   - Verwaltung der Container für die verschiedenen Komponenten (Django-App, PostgreSQL-Datenbank).

#### Datenbank-Schema

```sql
CREATE TABLE applicant (
    id SERIAL PRIMARY KEY,
    name VARCHAR(200),
    email VARCHAR(200) UNIQUE,
    matrikelnummer VARCHAR(20) UNIQUE
);

CREATE TABLE course (
    id SERIAL PRIMARY KEY,
    lv_url TEXT,
    institution VARCHAR(200),
    title VARCHAR(200),
    ects FLOAT,
    sws FLOAT,
    description TEXT
);

CREATE TABLE antrag (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    urls TEXT,
    texts TEXT,
    openai_texts TEXT,
    similarity_score FLOAT,
    explanation TEXT,
    status VARCHAR(20),
    comment TEXT,
    applicant_id INTEGER REFERENCES applicant(id)
);

CREATE TABLE antrag_courses (
    antrag_id UUID REFERENCES antrag(id),
    course_id INTEGER REFERENCES course(id),
    PRIMARY KEY (antrag_id, course_id)
);
```

#### PlantUML Klassendiagramm

```plantuml
@startuml
class Applicant {
  +id: Integer
  +name: CharField
  +email: EmailField
  +matrikelnummer: CharField
}

class Course {
  +id: Integer
  +lv_url: URLField
  +institution: CharField
  +title: CharField
  +ects: FloatField
  +sws: FloatField
  +description: TextField
}

class Antrag {
  +id: UUIDField
  +urls: TextField
  +texts: TextField
  +openai_texts: TextField
  +similarity_score: FloatField
  +explanation: TextField
  +status: CharField
  +comment: TextField
}

Applicant "1" -- "0..*" Antrag
Antrag "1" -- "0..*" Course
@enduml
```

#### Sequenzdiagramm

```plantuml
@startuml
actor Student
actor Mitarbeiter
participant Form
participant View
participant Model
participant OpenAI
participant Scraper

Student -> Form: submit URLs
Form -> View: validate and send URLs
View -> Model: create Antrag
View -> Scraper: get_text_from_url(url)
Scraper -> View: return text
View -> OpenAI: get_openai_text(text)
OpenAI -> View: return summarized text
View -> Model: save texts and summaries
View -> Student: show results
Mitarbeiter -> View: check Antrag
View -> Model: update status
View -> Mitarbeiter: show final decision
@enduml
```

**Passt dieser technische Ansatz und die Architekturbeschreibung zu Ihren Vorstellungen?**

project.wsgi:application"]
```

`docker-compose.yml`
```yaml
version: '3.7'

services:
  db:
    image: postgres
    environment:
      POSTGRES_DB: mydatabase
      POSTGRES_USER: myuser
      POSTGRES_PASSWORD: mypassword
    volumes:
      - postgres_data:/var/lib/postgresql/data

  web:
    build: .
    command: bash -c "python manage.py migrate && python manage.py runserver 0.0.0.0:8000"
    volumes:
      - .:/app
    ports:
      - "8000:8000"
    environment:
      SCRAPERAPI_KEY: your_scraperapi_key
      OPENAI_API_KEY: your_openai_api_key
      DATABASE_URL: postgres://myuser:mypassword@db:5432/mydatabase
    depends_on:
      - db

volumes:
  postgres_data:
```

`deployment.yaml`
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: django-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: django
  template:
    metadata:
      labels:
        app: django
    spec:
      containers:
      - name: django
        image: your_docker_image
        ports:
        - containerPort: 8000
        env:
        - name: SCRAPERAPI_KEY
          valueFrom:
            secretKeyRef:
              name: scraperapi-secret
              key: SCRAPERAPI_KEY
        - name: OPENAI_API_KEY
          valueFrom:
            secretKeyRef:
              name: openai-secret
              key: OPENAI_API_KEY
```

`service.yaml`
```yaml
apiVersion: v1
kind: Service
metadata:
  name: django-service
spec:
  selector:
    app: django
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8000
  type: LoadBalancer
```

`ingress.yaml`
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: django-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: syllabusmatcherai.its.ingress.wu.ac.at
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: django-service
            port:
              number: 80
```

#### 10. Start-Skript

`start.sh`
```bash
#!/bin/bash
python manage.py test website_api
python manage.py migrate
python manage.py runserver 0.0.0.0:8000
```

**Passt dieser Implementierungsplan zu Ihren Vorstellungen und Anforderungen?**


Um die Anwendung produktiv zu setzen, folgen wir diesen Schritten:

### 1. Docker-Image erstellen und in ein Container-Registry pushen

1. **Erstellen Sie ein Docker-Image:**
   ```bash
   docker build -t your_dockerhub_username/django_project:latest .
   ```

2. **Melden Sie sich bei Docker Hub an:**
   ```bash
   docker login
   ```

3. **Pushen Sie das Image in die Docker-Registry:**
   ```bash
   docker push your_dockerhub_username/django_project:latest
   ```

### 2. Kubernetes Cluster einrichten

1. **Kubernetes Cluster erstellen:**
   - Sie können einen Kubernetes-Cluster bei einem Cloud-Anbieter wie Google Kubernetes Engine (GKE), Amazon EKS, Azure Kubernetes Service (AKS) oder lokal mit Minikube erstellen.
   - Hier ist ein Beispiel für GKE:
     ```bash
     gcloud container clusters create django-cluster --num-nodes=3
     gcloud container clusters get-credentials django-cluster
     ```

2. **Erstellen Sie Kubernetes-Secrets für API-Schlüssel:**
   ```bash
   kubectl create secret generic scraperapi-secret --from-literal=SCRAPERAPI_KEY=your_scraperapi_key
   kubectl create secret generic openai-secret --from-literal=OPENAI_API_KEY=your_openai_api_key
   ```

### 3. Kubernetes Ressourcen anwenden

1. **Deployment und Service anwenden:**
   ```bash
   kubectl apply -f deployment.yaml
   kubectl apply -f service.yaml
   ```

2. **Ingress anwenden (falls Ingress verwendet wird):**
   ```bash
   kubectl apply -f ingress.yaml
   ```

### 4. Datenbank konfigurieren und migrieren

1. **PostgreSQL-Datenbank bereitstellen:**
   - Sie können eine PostgreSQL-Datenbank über einen Cloud-Anbieter verwenden oder in Ihrem Kubernetes-Cluster bereitstellen.
   - Beispiel für PostgreSQL in Kubernetes:
     ```yaml
     apiVersion: v1
     kind: Service
     metadata:
       name: postgres
     spec:
       ports:
       - port: 5432
       selector:
         app: postgres
     ---
     apiVersion: apps/v1
     kind: Deployment
     metadata:
       name: postgres
     spec:
       selector:
         matchLabels:
           app: postgres
       template:
         metadata:
           labels:
             app: postgres
         spec:
           containers:
           - name: postgres
             image: postgres:12
             env:
             - name: POSTGRES_DB
               value: mydatabase
             - name: POSTGRES_USER
               value: myuser
             - name: POSTGRES_PASSWORD
               value: mypassword
             ports:
             - containerPort: 5432
     ```

2. **Umgebungsvariablen für die Django-App konfigurieren:**
   - Aktualisieren Sie `deployment.yaml`, um die Datenbankverbindungsinformationen bereitzustellen:
     ```yaml
     env:
     - name: DATABASE_URL
       value: postgres://myuser:mypassword@postgres:5432/mydatabase
     ```

3. **Datenbank-Migrationen ausführen:**
   - Nachdem die Anwendung und die Datenbank bereitgestellt wurden, führen Sie die Migrationen aus:
     ```bash
     kubectl exec -it <django_pod_name> -- python manage.py migrate
     ```

### 5. Überwachung und Logging einrichten

1. **Überwachungstools wie Prometheus und Grafana einrichten:**
   - Installieren Sie Prometheus und Grafana in Ihrem Cluster, um Metriken zu überwachen.
   - Beispiel mit Helm:
     ```bash
     helm install prometheus stable/prometheus
     helm install grafana stable/grafana
     ```

2. **Logging mit Elasticsearch, Fluentd und Kibana (EFK) einrichten:**
   - Installieren Sie das EFK-Stack, um Logs zu sammeln und zu visualisieren.
   - Beispiel mit Helm:
     ```bash
     helm install elasticsearch stable/elasticsearch
     helm install fluentd stable/fluentd
     helm install kibana stable/kibana
     ```

### 6. Load Balancer und Domain konfigurieren

1. **Domain und SSL/TLS einrichten:**
   - Richten Sie einen DNS-Eintrag für Ihre Anwendung ein, um den Traffic auf den Kubernetes-Load-Balancer zu leiten.
   - Nutzen Sie Let's Encrypt, um SSL/TLS-Zertifikate für Ihre Domain zu erhalten und zu konfigurieren:
     ```yaml
     apiVersion: cert-manager.io/v1
     kind: Certificate
     metadata:
       name: tls-cert
       namespace: default
     spec:
       secretName: tls-secret
       issuerRef:
         name: letsencrypt
         kind: ClusterIssuer
       commonName: syllabusmatcherai.its.ingress.wu.ac.at
       dnsNames:
       - syllabusmatcherai.its.ingress.wu.ac.at
     ```

**Zusammengefasst:**

1. Erstellen und pushen Sie das Docker-Image.
2. Richten Sie den Kubernetes-Cluster ein und erstellen Sie die erforderlichen Secrets.
3. Wenden Sie die Kubernetes-Ressourcen an (Deployment, Service, Ingress).
4. Stellen Sie sicher, dass die Datenbank bereitgestellt und konfiguriert ist.
5. Führen Sie die Datenbank-Migrationen aus.
6. Richten Sie Überwachung und Logging ein.
7. Konfigurieren Sie Domain und SSL/TLS.

Damit sollte Ihre Anwendung bereit sein, produktiv eingesetzt zu werden. Haben Sie noch weitere Fragen oder benötigen Sie Details zu einem bestimmten Schritt?

