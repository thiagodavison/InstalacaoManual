INSTALAÇÃO PLATAFORMA WHATICKET SAAS COM BANCO POSTGRES

--------------------------------------------------------------------------------------------------------</br>
LEGENDAS PARA SUBSTITUIÇÃO</br>
%Substituir% - Substituir todos os conteúdos entre % </br>
%%%%%%%%% Comando %%%%%%%%% - Editar o arquivo com os dados abaixo do comando em destaque.</br>
app.seudomínio.com.br - Substituir com o domínio ou subdomínio do FRONTEND</br>
api.seudomínio.com.br - Substituir com o domínio ou subdomínio do BACKEND</br>
--------------------------------------------------------------------------------------------------------</br>
Obs: Sempre observar o que é para ser executado com usuario root e usuario deploy</br>

@@@@@@@@@@@@@############## INSTALAÇÃO COM USUARIO ROOT ##############@@@@@@@@@@@@@</br>
ATUALIZAR A VPS
```bash
	sudo apt update && sudo apt -y upgrade
```
INSTALAR O NODE E POSTGRES
```bash 
	sudo su root
	curl -fsSL https://deb.nodesource.com/setup_16.x | sudo -E bash -
	apt-get install -y nodejs
	npm install -g npm@latest
	sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
	wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
	sudo apt-get update -y && sudo apt-get -y install postgresql
	sudo timedatectl set-timezone America/%SeuTimezone%
```
	
INTALAR O DOCKER
```bash	
	apt install -y apt-transport-https ca-certificates curl software-properties-common
	curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -
	add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu bionic stable"
	apt install -y docker-ce
```
INSTALAR O PUPPETEER
```bash	
	apt-get install -y libxshmfence-dev libgbm-dev wget unzip fontconfig locales gconf-service libasound2   libatk1.0-0 libc6 libcairo2 libcups2 libdbus-1-3  	 libexpat1 libfontconfig1 libgcc1 libgconf-2-4 libgdk-pixbuf2.0-0 libglib2.0-0 libgtk-3-0 libnspr4 libpango-1.0-0 libpangocairo-1.0-0 libstdc++6        libx11-6 	libx11-xcb1 libxcb1 libxcomposite1 libxcursor1 libxdamage1 libxext6 libxfixes3 libxi6 libxrandr2 libxrender1 libxss1 libxtst6 ca-certificates  fonts-liberation 	libappindicator1 libnss3 lsb-release xdg-utils
```
	
INSTALAR O PM2
```bash
	npm install -g pm2
```

INSTALAR O SNAPD
```bash
	apt install -y snapd
	snap install core
	snap refresh core
```

INSTALAR O NGINX</br>
```bash
	apt install -y nginx
	rm /etc/nginx/sites-enabled/default
		service nginx restart
	%%%%%%%%%    sudo nano /etc/nginx/nginx.conf    %%%%%%%%%
		client_max_body_size 100M;
```

INSTALAR O CERTBOT</br>
```bash
	apt-get remove certbot
	snap install --classic certbot
	ln -s /snap/bin/certbot /usr/bin/certbot
```	
	
CRIAR USUARIO DEPLOY</br>
```bash
	adduser deploy
	usermod -aG sudo deploy
```

CLONAR O REPOSITORIO</br>
```bash
	su deploy
	sudo apt install -y git && git clone https://github.com/%SeuRepositório%.git  /home/deploy/%instancia%
```
	
@@@@@@@@@@@@@##############-- BACKEND --##############@@@@@@@@@@@@@</br>

CRIAR O REDIS E BANCO POSTGRES</br>
```bash	
	sudo su root
	usermod -aG docker deploy
	docker run --name redis-%NomeRedis% -p 5000:6379 --restart always --detach redis redis-server --requirepass %SenhaRedis%
	sudo su - postgres
	createdb %Nomebanco%;
	psql
	CREATE USER %Userbanco% SUPERUSER INHERIT CREATEDB CREATEROLE;
	ALTER USER %Userbanco% PASSWORD '%SenhaUser%';
	\q
	exit
```	
	
CRIAR VARIAVEL DE AMBIENTE</br>
	----Utilizando usuario deploy no diretótio backend----</br>
```bash	
	cd /home/deploy/%instancia%/backend
```
```bash	
	%%%%%%%%% sudo nano .env %%%%%%%%%
------------Editar o arquivo com os dados abaixo--------------
NODE_ENV=  
BACKEND_URL=https://api.seudomínio.com.br
FRONTEND_URL=https://app.seudomínio.com.br
PORT=4000  
PROXY_PORT=443
CHROME_BIN=/usr/bin/google-chrome-stable

DB_DIALECT=postgres  
DB_HOST=localhost  
DB_TIMEZONE=-04:00  
DB_USER= %Userdb% 
DB_PASS= %Passdb%
DB_NAME= %Nomedb%

rootPath=/var/www/backend

USER_LIMIT=30  
CONNECTIONS_LIMIT=2

JWT_SECRET=5g1yk7pD9q3YL0isdutyi
JWT_REFRESH_SECRET=F2c8gag5nvqQkasadr

REDIS_URI=redis://:%SenhaRedis%@127.0.0.1:5000
REDIS_OPT_LIMITER_MAX=1
REDIS_OPT_LIMITER_DURATION=3000


SKIP_PREFLIGHT_CHECK=true
```
	
INSTALAÇÃO DAS DEPENDENCIAS</br>
	----Utilizando usuario deploy no diretótio backend----</br>
```bash	
	cd /home/deploy/%instancia%/backend
	npm install
```
COMPILANDO O CÓDIGO DO BACKEND</br>
	----Utilizando usuario deploy no diretótio backend----</br>
```bash	
	cd /home/deploy/%instancia%/backend
	npm run build
```

CRIANDO AS TABELAS NO BANCO</br>
	----Utilizando usuario deploy no diretótio backend----</br>
```bash	
	cd /home/deploy/%instancia%/backend
	npx sequelize db:migrate
```

PUPULANDO AS TABELAS DO BANCO</br>
	----Utilizando usuario deploy no diretótio backend----</br>
```bash	
	cd /home/deploy/%instancia%/backend
	npx sequelize db:seed:all
```	
	
INICIANDO O SERVIÇO PM2</br>
	----Utilizando usuario deploy no diretótio backend----</br>
```bash	
	cd /home/deploy/%instancia%/backend
	pm2 start dist/server.js --name %instancia%-backend
```

CONFIGURAÇÃO DO NGINX</br>
	----Utilizando usuario root no diretótio etc----</br>
```bash	
	%%%%%%%%%    sudo nano /etc/nginx/sites-available/%instancia%-backend    %%%%%%%%% 

	server {
  server_name api.seudomínio.com.br;
  location / {
    proxy_pass http://127.0.0.1:4000;
    proxy_http_version 1.1;
    proxy_set_header Upgrade \$http_upgrade;
    proxy_set_header Connection 'upgrade';
    proxy_set_header Host \$host;
    proxy_set_header X-Real-IP \$remote_addr;
    proxy_set_header X-Forwarded-Proto \$scheme;
    proxy_set_header X-Forwarded-For \$proxy_add_x_forwarded_for;
    proxy_cache_bypass \$http_upgrade;
  }
}
```
	----Gerando link simbólicocd----
	
```bash	
	ln -s /etc/nginx/sites-available/%instancia%-backend /etc/nginx/sites-enabled
```

@@@@@@@@@@@@#######--- FRONTEND ---########@@@@@@@@@@</br>

CRIAR VARIAVEL DE AMBIENTE</br>
	----Utilizando usuario deploy no diretótio frontend----</br>
```bash	
	cd /home/deploy/%instancia%/frontend	
	%%%%%%%%% sudo nano .env %%%%%%%%%
		REACT_APP_BACKEND_URL=https://api.seudomínio.com.br
		REACT_APP_HOURS_CLOSE_TICKETS_AUTO = 24
```		

INSTALANDO DEPENDENCIAS DO FRONTEND</br>
	----Utilizando usuario deploy no diretótio frontend----</br>
```bash	
	cd /home/deploy/%instancia%/frontend
	npm install
```
	
COMPILANDO O CODIGO DO FRONTEND</br>
	----Utilizando usuario deploy no diretótio frontend----</br>
```bash	
	cd /home/deploy/%instancia%/frontend
	npm run build
```

INICIANDO O PM2</br>
	----Utilizando usuario deploy no diretótio frontend----</br>
```bash	
	cd /home/deploy/%instancia%/frontend
	pm2 start server.js --name %instancia%-frontend
	pm2 save
```	
	----Configuração para o PM2 iniciar com o Ubuntu----</br>
```bash	
	pm2 startup
	sudo env PATH=$PATH:/usr/bin /usr/lib/node_modules/pm2/bin/pm2 startup systemd -u deploy --hp /home/deploy
```

CONFIGURAÇÃO DO NGINX</br>
	----Utilizando usuario root no diretótio etc----</br>
```bash
	%%%%%%%%%    sudo nano /etc/nginx/sites-available/%instancia%-frontend    %%%%%%%%%
	
	server {
  server_name app.seudomíno.com.br;
  location / {
    proxy_pass http://127.0.0.1:3000;
    proxy_http_version 1.1;
    proxy_set_header Upgrade \$http_upgrade;
    proxy_set_header Connection 'upgrade';
    proxy_set_header Host \$host;
    proxy_set_header X-Real-IP \$remote_addr;
    proxy_set_header X-Forwarded-Proto \$scheme;
    proxy_set_header X-Forwarded-For \$proxy_add_x_forwarded_for;
    proxy_cache_bypass \$http_upgrade;
  }
}
```	
	----Gerando link simbólico----</br>
```bash	
	sudo ln -s /etc/nginx/sites-available/%instancia%-frontend /etc/nginx/sites-enabled
```

RODAR O CERTBOT PARA GERAR O SSL</br>
```bash
	certbot -m %SeuEmail% --nginx --agree-tos --non-interactive --domains app.seudomínio.com.br,api.seudomínio.com.br
```

