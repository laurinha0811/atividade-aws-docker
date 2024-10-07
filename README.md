# Passo 1: Instalação e configuração do Docker no host EC2
Passo a passo para instalar e configurar Docker ou Containerd em uma instância EC2 na AWS.

## Lançar uma instância EC2
Para criar uma instância EC2 no console da AWS é necessário seguir os seguintes passos: 
1. Vá para o AWS Management Console.
2. Navegue até o serviço EC2.
3. Clique em Launch Instance.
4. Selecione uma Amazon Machine Image (AMI). Para este tutorial, selecione Amazon Linux 2 ou Ubuntu.
5. Escolha o tipo de instância. A t2.micro (elegível para o nível gratuito) é suficiente para esse caso.
6. Em Configure Instance Details, verifique se sua instância está dentro de uma VPC (rede privada).
7. No campo User Data, adicione um script para automatizar a instalação do Docker.
8. Continue a configuração e clique em Launch.

## Instalação do Docker via User Data Script
No passo anterior foi mencionado a necessidade de usar o campo User Data para rodar scripts automaticamente na inicialização da instância. Abaixo está um exemplo de script user_data.sh para instalar e configurar o Docker.
- Script User Data para Amazon Linux 2:
No campo "User Data" da instância EC2, adicione o seguinte script:

```bash
#!/bin/bash
# Atualizar o sistema
sudo yum update -y

# Instalar o Docker
sudo amazon-linux-extras install docker -y

# Iniciar o serviço Docker
sudo service docker start

# Adicionar o usuário 'ec2-user' ao grupo docker
sudo usermod -aG docker ec2-user

# Ativar Docker para iniciar na inicialização
sudo systemctl enable docker

# Instalar o Docker Compose
sudo curl -L "https://github.com/docker/compose/releases/download/v2.10.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose

# Testar se o Docker Compose foi instalado corretamente
docker-compose --version
```

Na prática, ficará assim:

![image](https://github.com/user-attachments/assets/183b6129-6154-418d-8650-861a3b37e04b)

Este script é responsável por:
- Atualizar o sistema.
- Instalar o Docker e iniciar o serviço.
- Adicionar o usuário padrão ec2-user ao grupo Docker para permitir rodar comandos sem sudo.
- Instalar o Docker Compose.

## Conectar-se a instância EC2:
Depois de lançar a instância, siga os passos abaixo para se conectar:
- Vá até o painel de instâncias EC2.
- Selecione a instância que você criou e clique em Connect.
- Siga as instruções para conectar via SSH. Exemplo de comando:

```bash
ssh -i "C:\Users\laura\Downloads\meu-par-de-chaves.pem" ec2-user@ec2-34-204-73-16.compute-1.amazonaws.com
```

Após a execução do comando vocêw verá uma tela semelhante a essa: 
![image](https://github.com/user-attachments/assets/2eefc45c-a6ea-4855-9f4a-0148ce10b2c0)

## Verificar a instalação do Docker:
Após conectar-se via SSH, verifique se o Docker foi intalado corretamente usando o seguinte comando:
```bash
# Verificar a versão do Docker
docker --version

# Testar se o Docker está rodando corretamente
sudo docker run hello-world
```
Após a execução do comando "sudo docker run hello-world" você verá uma informação semelhante a essa, indicando que o Docker está funcionando corretamente.

![image](https://github.com/user-attachments/assets/3f1884b7-c994-45f4-ba67-538fee16f7d6)

# Passo 2: Efetuar Deploy de uma aplicação WordPress
Agora vamos criar uma aplicação WordPress usando conteiners Docker e conectar ao banco de dados MySQL hospedado no RDS.

## Criar um Banco de Dados RDS (MySQL)
Vamos precisar de um banco de dados MySQL no Amazon RDS para armazenar os dados do WordPress. Para isso, basta seguir os seguintes passos: 
- Acesse o console da AWS RDS e clique em "Create Database".
- Escolha MySQL como o mecanismo do banco de dados.
- Engine: Escolha a versão do MySQL.
- Templates: Free Tier.
- DB instance class: db.t3.micro
- Storage: Defina a capacidade de armazenamento -  20 GB.

Defina um nome de usuário (por exemplo: admin) e uma senha segura.

Configurações de rede:
- Certifique-se de configurar o RDS para estar na mesma VPC que a instância EC2.

Criar a instância RDS: Clique em "Create Database" e aguarde até o banco de dados MySQL ser iniciado. Após a criação do banco de dados você verá uma tela semelhante a essa: 

![image](https://github.com/user-attachments/assets/f88d0556-4e99-467f-9302-769416068545)

## Conectar o WordPress ao RDS (MySQL)
Agora que temos o RDS MySQL pronto, o própximo passo será concectar a aplicação WordPress ao banco de dados durante o deploy.

Vamos usar Docker Compose para a aplicação WordPress, para isso será necessário:
- Criar um arquivo `docker-compose.yml`. Em resumo, esse arquivo ficará responsável por definir como o Docker deve iniciar os containers para o WordPress e contectar ao banco MySQL
```bash
services:
  wordpress:
    image: wordpress:latest
    restart: always
    ports:
      - "80:80"
    environment:
      WORDPRESS_DB_HOST: database-2.cxsmgakqaa8w.us-east-1.rds.amazonaws.com:3306
      WORDPRESS_DB_USER: laura
      WORDPRESS_DB_PASSWORD: 1337LBG!
      WORDPRESS_DB_NAME: wordpress_db
    volumes:
      - wordpress-data:/var/www/html

volumes:
  wordpress-data:
```

Explicação dos parâmetros utilizados:

- `WORDPRESS_DB_HOST`: serve para inserir o endpoint do RDS MySQL.
- `WORDPRESS_DB_USER`: é o nome de usuário do banco de dados (que você configurou no RDS).
- `WORDPRESS_DB_PASSWORD`: é a senha do banco de dados MySQL (também configurada no RDS).
- `WORDPRESS_DB_NAME`: é o nome do banco de dados MySQL que foi configurado.
- `ports`: é a configuração 80:80 faz o container WordPress rodar na porta 80 instância EC2, tornando o site acessível via HTTP.

Feito isso, podemos inicializar o conteiner WordPress:
- Para isso, vamos ao diretório onde foi criado o arquivo docker-compose.yml, e lá vamos executar o seguinte comando:
```bash
docker-compose up -d
```
Em resumo, isso irá iniciar o container WordPress e conectar-se automaticamente ao banco de dados RDS MySQL:
![image](https://github.com/user-attachments/assets/82f0a69e-dc5b-4e9b-9c86-86237011f0e2)

## Acesso ao aplicativo WordPress:
Feito a implantação, vamos conseguir acessar o Wordpress na porta 80 da nossa instância EC2. Para isso, vamos usar o URL associado à instância.
Para confirmar que está funcionando:
- Vamos acessar o IP público da instância EC2.
- A página de instalação do WordPress deve aparecer.
- Em seguida, é só concluir a instalação configurando o nome do site e credenciais de administrador do WordPress.

![Captura de tela 2024-10-03 120450](https://github.com/user-attachments/assets/1224d295-0b74-4022-9b75-95c506f579cf)

![image](https://github.com/user-attachments/assets/f7717c97-7035-480f-a8b2-b5f5a3b1c56b)

# Passo 3: Configuração do EFS (Elastic File System)

Primeiramente, vamos criar o AWS EFS, para isso vamos até o console da AWS, pesquisando por "ESF" e vamos acessar o Elastic File System. A partir dai é só seguir os seguintes passos: 
- Clique em "Create File System".
- Selecione a VPC onde sua instância EC2 está localizada.
- Escolha as sub-redes nas zonas de disponibilidade, nesse caso as mesmas em que a EC2 esta rodando.
- Selecione os grupos de segurança, lembre de garantir   que ela tenha acesso de inbound NFS (porta 2049).
- Feito as configurações, basta clicar em Create.

Agora, vamos conectar à instância EC2:
- Para montar o EFS, instale os pacotes necessários:
  ```bash
  sudo yum install -y amazon-efs-utils
    ```
- Crie um diretório onde o EFS será montado:
  ```bash
  sudo mkdir /mnt/efs
    ```
- Monte o EFS:
  ```bash
  sudo mount -t efs fs-xxxxxxxx:/ /mnt/efs
  ```
Lembre-se de substituir o fs-xxxxxxxx pelo ID do seu próprio EFS.
- Verifique se a montagem foi bem sucedida:
 ```bash
df -h
 ```
Na prática, você receberá resultados semelhantes a esses:
![image](https://github.com/user-attachments/assets/d2bd1a3d-aaec-4320-9b55-684134a9cc56)

Após atualizar o arquivo `docker-compose.yml`, é necessário reinicializar o Docker para aplicar as alterações:
Feito isso, vamos integrar o EFS ao conteiner WordPress para armazenar os arquivos estáticos, para isso será necessário: 
- Atualizar o arquivo `docker-compose.yml` para que o volume do ESF seja usado pelo WordPress:
![image](https://github.com/user-attachments/assets/0ce4e73f-087d-4287-a943-c667c6e1b104)

- Depois de atualizar o arquivo `docker-compose.yml`, reinicie o Docker Compose para aplicar as alterações:
![image](https://github.com/user-attachments/assets/3659febd-6d3e-4119-bb4c-ba0ae3533629)

## Agora vamos verificar o funcionamento
Tela de login WordPress (http://54.91.39.246/wp-login.php):
  ![image](https://github.com/user-attachments/assets/cedb0533-459e-48b7-a9b7-61e2ba87c86f)

# Passo 4: Load Balancer para a aplicação WordPress
Agora vamos configurar um Load Balancer Classic no AWS para gerenciar o tráfego de entrada da nossa aplicação WordPress e distribuir entre as instâncias EC2. Para isso será necessário seguir os seguintes passos:
- Acessar o Console da AWS.
- No menu de serviços, selecione EC2.
- No painel esquerdo, desça até a seção Load Balancers.
- Criar um Novo Load Balancer
- No painel de Load Balancers, clique no botão Create Load Balancer.
- Selecione a opção Application Load Balancer e clique em Continue.
- Escolha a VPC na qual suas instâncias EC2 estão executando.
- Escolha Internet-facing para que o Load Balancer seja acessível da Internet.

Listeners:
- Protocolo: Selecione HTTP.
- Porta: porta padrão 80.

Availability Zones:
- Zonas de disponibilidade onde suas instâncias EC2 estão rodando.
- Marque as sub-redes associadas a essas zonas.
- Clique em Next: Assign Security Groups.

Agora vamos configurar Grupos de Segurança
- Assign a Security Group: o grupo de segurança deve permitir tráfego de entrada na porta 80.

Configurar Verificação de Saúde (Health Checks)
- Ping Protocol: Selecione HTTP.
- Ping Port: Defina como 80.
- Ping Path: Insira / para verificar a resposta da página principal do WordPress.
Advanced Details:
- Response Timeout: 5 seconds.
- Interval: 30 seconds.
- Unhealthy Threshold: 2.
- Healthy Threshold: 2.
- Clique em Next: Add EC2 Instances e adicione a instâncias EC2 ao Load Balancer.
E pronto, nosso Load balancer foi criado:
![image](https://github.com/user-attachments/assets/a5dac8ff-732d-49da-97e7-a6af34be82d1)
