# Passo 1: Instalação e configuração do Docker/Containerd no host EC2
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
