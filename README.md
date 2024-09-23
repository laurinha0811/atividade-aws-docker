# Passo 1: Instalação e configuração do Docker/Containerd no host EC2
Passo a passo para instalar e configurar Docker ou Containerd em uma instância EC2 na AWS.

## 1. Lançar uma instância EC2
Para criar uma instância EC2 no console da AWS é necessário seguir os seguintes passos: 
1. Vá para o AWS Management Console.
2. Navegue até o serviço EC2.
3. Clique em Launch Instance.
4. Selecione uma Amazon Machine Image (AMI). Para este tutorial, selecione Amazon Linux 2 ou Ubuntu.
5. Escolha o tipo de instância. A t2.micro (elegível para o nível gratuito) é suficiente para esse caso.
6. Em Configure Instance Details, verifique se sua instância está dentro de uma VPC (rede privada).
7. No campo User Data, adicione um script para automatizar a instalação do Docker.
8. Continue a configuração e clique em Launch.

# Passo 2: Instalação do Docker via User Data Script
No passo anterior foi mencionado a necessidade de usar o campo User Data para rodar scripts automaticamente na inicialização da instância. Abaixo está um exemplo de script user_data.sh para instalar e configurar o Docker.

- Script User Data para Amazon Linux 2:
No campo "User Data" da instância EC2, adicione o seguinte script:
'''
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
'''

![image](https://github.com/user-attachments/assets/183b6129-6154-418d-8650-861a3b37e04b)
