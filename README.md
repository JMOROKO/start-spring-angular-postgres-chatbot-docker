<h1>Au niveau du back end</h1>
<p>
<h1>1. Il faut d'abord clean le projet</h1>
    <pre>
    mvn clean 
    </pre>
    <h1>2. Il faut ensuite générer le fichier .jar</h1>
    <pre>
    mvn package -DskipTests
    </pre>
    <h1>3. Création du fichier Dockerfile dans le back end et ajouter les instructions suivantes :</h1>
    <pre>
    FROM openjdk:21-oracle
    VOLUME /tmp
    COPY target/backend*.jar app.jar
    EXPOSE 8074
    ENTRYPOINT ["java", "-jar", "app.jar"]
    </pre>
    <h1>4. Creer l'image docker</h1>
    <pre>
        docker build -t nom_image:tage location_dockerFile
    </pre>
    <pre>
        docker build -t back:v1 .
    </pre>
</p>
<h1>Pour le front end</h1>
<p>
    <h1>1. Creer le dossier environnement</h1>
    <pre>
        ng g c environment
    </pre>
    <h1>
        2. Creer le build
    </h1>
    <pre>
        ng build --configuration=developement 
        #development pour utiliser le fichier d'environnement de development
        #production pour utiliser le fichier d'environnement de production
    </pre>
    <h1>
        3. Creer le fichier Dockefile
    </h1>
    <pre>
        FROM nginx:alpine
        COPY ./nginx.conf /etc/nginx/conf.d/default.conf
        COPY dist/frontend/browser /usr/share/nginx/html
        EXPOSE 80
    </pre>
    <h1>4. Creation du fichier de configuration de nginx pour modifier la configuration du serveur</h1>
    <pre>
        server {
          listen 80;
          sendfile on;
          default_type application/octet-stream;
        
            gzip on;
            gzip_http_version 1.1;
            gzip_disable      "MSIE [1-6]\.";
            gzip_min_length   256;
            gzip_vary         on;
            gzip_proxied      expired no-cache no-store private auth;
            gzip_types        text/plain text/css application/json application/javascript application/x-javascript text/xml application/xml application/xml+rss text/javascript;
            gzip_comp_level   9;
            
            root /usr/share/nginx/html;
            
            location / {
                    try_files $uri $uri/ /index.html =404;
                }
            }
    </pre>
    
</pre>
<h1>5. Configurer le fichier docker compose de la racine afin qu'il puisse comuniquer avec les autres fichier dockerFile creer pour démarrer le projet</h1>
<pre>
services:
  pgvector:
    container_name: pgdb-store
    image: 'pgvector/pgvector:pg16'
    environment:
      - 'POSTGRES_DB=dbCryptoCurrency'
      - 'POSTGRES_PASSWORD=1234'
      - 'POSTGRES_USER=admin'
    volumes:
      - chatbot_data:/var/lib/postgresql/data
    ports:
      - '5432:5432'
    networks: chatbot-net
  chatbot-back:
      build: ./backend
      container_name: chatbot-back
      ports:
        - '8074:8074'
      networks: chatbot-net
      depends_on: pgvector #dependance entre service, permet de demander à docker de demarrer le service de la base de données avant de démarrer le backend
  chatbot-front:
    build: ./frontend
    container_name: chatbot-front
    ports:
      - '8086:80'
    networks: chatbot-net
volumes:
  chatbot_data:
networks:
  chatbot-net
</pre>
<h1>6. Démarrer le fichier docker compose</h1>
<pre>
    docker compose up --build
</pre>