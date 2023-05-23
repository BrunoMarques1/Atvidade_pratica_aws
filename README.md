# Requisitos:

### AWS:
- Gerar uma chave pública;
- Criar 1 instância EC2 com o sistema operacional Amazon Linux 2 (Família t3.small, 16 GB SSD);
- Gerar 1 elastic IP e anexar à instância EC2;
- Liberar as portas de comunicação para acesso público: (22/TCP, 111/TCP e UDP, 2049/TCP/UDP, 80/TCP, 443/TCP).

### Linux:
- Configurar o NFS entregue;
- Criar um diretorio dentro do filesystem do NFS com seu nome;
- Subir um apache no servidor - o apache deve estar online e rodando;
- Criar um script que valide se o serviço esta online e envie o resultado da validação para o seu diretorio no nfs;
- O script deve conter - Data HORA + nome do serviço + Status + mensagem personalizada de ONLINE ou offline;
- O script deve gerar 2 arquivos de saida: 1 para o serviço online e 1 para o serviço OFFLINE;
- Preparar a execução automatizada do script a cada 5 minutos.
- Fazer o versionamento da atividade;
- Fazer a documentação explicando o processo de instalação do Linux.

<br>

---
# AWS:
### Criando um usuário IAM
- Ir no serviço IAM da AWS;
- No menu da esquerda clicar em "Usuários" e depois "Adicionar usuários";
- Escolher um nome para o usuário e nas opções de permissões selecionar "Anexar políticas diretamente";
- Escolher a política "Administrator Access";
- Após seguir esses passos, criar uma chave de acesso para o usuário criado, para usar no terminal com o comando `aws configure`, como no exemplo abaixo:
<div align="center">
  
![image](https://github.com/BrunoMarques1/Atvidade_pratica_aws/assets/127341401/bc029516-1680-45a1-a45c-b0ab36f5b1d8)
  
</div>
<br>

### Criando par de chaves
- Já dentro do serviço EC2 da AWS, clicar em "Pares de Chaves", na parte de "Rede e segurança";
- Clicar em "Criar par de chaves";
- Escolher um nome, o tipo e o formato para a chave. No meu caso o nome inserido foi "ChaveC", o tipo escolhido foi RSA e o formato foi .pem;
- Salvar o arquivo da chave.

<br>

### Configurando security group
- Após a criação do par de chaves, ainda na parte de Rede e segurança, clicar em "Security Groups";
- Clicar em "Criar grupo de segurança";
- Escolher um nome para o security group, no meu o nome será "SGa1"
- As regras de entrada serão as seguintes:
<div align="center">
 
Tipo | Protocolo | Intervalo de Portas | Origem
---|---|---|---
SSH  | TCP | 22 | 0.0.0.0/0
TCP Personalizado | TCP | 111 | 0.0.0.0/0
UDP Personalizado | UDP | 111 | 0.0.0.0/0
NFS | TCP | 2049 | 0.0.0.0/0
UDP Personalizado | UDP | 2049 | 0.0.0.0/0
HTTP | TCP | 80 | 0.0.0.0/0
HTTPS | TCP | 443 | 0.0.0.0/0

</div>
<br>

### Criação da instância EC2:
- Para criar a instância EC2, ir no terminal da máquina usada anteriormente para logar no usuário IAM criado, e usar o seguinte comando (fazendo alterações necessárias na parte das tags): 
```

aws ec2 run-instances --image-id "ami-06a0cd9728546d178" --count 1 --instance-type "t3.small" --key-name "ChaveC" --security-groups "SGa1" --block-device-mappings '[{"DeviceName":"/dev/xvda","Ebs":{"VolumeSize":16,"VolumeType":"gp2"}}]' --tag-specifications "ResourceType=instance,Tags=[{Key=Name,Value=Name},{Key=Key,Value=Value},{Key=Key,Value=Value}]"  "ResourceType=volume,Tags=[{Key=Name,Value=Name},{Key=Key,Value=Value},{Key=Key,Value=Value}]"  

```
- O que faz cada parte do comando:
  - É usado para criar uma instância EC2 na AWS:
    - ` aws ec2 run-instaces ` 
  - Especifica a ID da imagem AMI a ser usada para criar a instância EC2 (Nesse exemplo temos a seguinte imagem: Amazon Linux 2):
    - ` --image-id "ami-06a0cd9728546d178" ` 
  - Indica o número de instâncias EC2 que serão criadas (Nesse exemplo foi criada apenas uma instância):
    - `--count 1`
  - Define o tipo de instância EC2 que será criada (Nesse exemplo foi escolhido o seguinte tipo de instância: t3.small):
    - ` --instance-type "t3.small" ` 
  - Especifica o nome do par de chaves EC2 que será associado à instância (Nesse exemplo a chave escolhida foi a mesma criada anteriormente, ChaveC):
    - ` --key-name "ChaveC" ` 
  - Define o nome do grupo de segurança EC2 a ser associado à instância (Nesse exemplo o Security group escolhido foi o mesmo criado anteriormente, SGa1):
    - ` --security-groups "SGa1" ` 
  - Permite especificar a configuração de armazenamento da instância (Nesse exemplo criamos um volume de 16gb do tipo gp2):
    - ` --block-device-mappings '[{"DeviceName":"/dev/xvda","Ebs":{"VolumeSize":16,"VolumeType":"gp2"}}]' `
  - Define as tags a serem aplicadas à instância EC2 e/ou outros recursos associados (Nesse exmplo criamos tags do tipo instancia e volume, para usar é necessário mudar os valores dentro de "Key" e "Value"):
    - `--tag-specifications "ResourceType=instance,Tags=[{Key=Name,Value=Name},{Key=Key,Value=Value},{Key=Key,Value=Value}]"  "ResourceType=volume,Tags=[{Key=Name,Value=Name},{Key=Key,Value=Value},{Key=Key,Value=Value}]"`

<br>

### Gerando um IP elástico e alocando na instância EC2:
- Ir no serviço EC2 da aws e clicar em "IPs elásticos", presente na parte de "Rede e Segurança";
- Clicar em "Alocar endereço IP elástico" e depois em "Alocar";
- Selecionar o IP alocado e clicar em "Associar endereço IP elástico";
- Selecionar a instância criada anteriormente e clicar em "Associar"

<br>

---
# Linux:
### Configurando o NFS entregue:
- No console AWS procurar pelo serviço EFS;
- Clicar em "Criar sistema de arquivos";
- Escolher um nome e manter a mesma VPC de sua instância EC2, depois clicar em "Criar";
- Na instância EC2, criar um novo diretório com o comando `sudo mkdir /mnt/nfs`;
- No serviço de EFS clique no sistema de arquivos recém criado e vá na parte de "Rede";
- Mude todos os Security Groups para o mesmo usado na instância criada anteriromente, no meu caso usarei o SGa1;
- Volte e clique em "Anexar";
- Copie o código em baixo do seguinte enunciado: "Usando o cliente do NFS", e mude apenas o caminho final para o diretório criado anteriormente;
- Cole no terminal da sua instância EC2 e tecle ENTER;
- Para não precisar rodar esse comando toda vez que reiniciar a máquina, adcione a seguinte linha para o arquivo `/etc/fstab`:
``` 
IP_OU_DNS_DO_NFS:/ /mnt/nfs nfs defaults 0 0 
```
- Após salvar o arquivo, criar um diretório com seu nome no caminho `/mnt/nfs`, por exemplo: `sudo mkdir /mnt/nfs/Bruno`;

<br>

### Configurando o Apache:
- Usar o comando sudo `yum update -y` para atualizar o sistema;
- Usar o comando sudo `yum install httpd -y` para instalar o apache;
- Usar o comando sudo `systemctl start httpd` para iniciar o apache;
- Usar o comando sudo `systemctl enable httpd` para habilitar o apache para iniciar automaticamente;
- Usar o comando `sudo systemctl status httpd` para verificar o status do apache.

<br>

### Criando script de validação do serviço:
<details>
<summary>Configurar horario/data</summary>
 
- Antes de criar o script, vamos configurar a data da máquina, use o comando `date`, se o horário estiver de acordo com o de sua localização, pode pular essa parte;
- No terminal da sua instância EC2, use o comando `timedatectl list-timezones`, ele irá listar os fusos horários disponíveis;
- Após achar o seu fuso horário, use o comando `sudo timedatectl set-timezone <nome_do_fuso_horario>`;
- Use o comando `timedatectl` para ver se o fuso horário foi configurado corretamente.

 </details>
  
 - Crie o arquivo que será o script com o seguinte comando `vi script.sh`;
 - Dentro do arquivo, use o seguite script: 
```bash
#!/bin/bash

DATA=$(date +%d/%m/%Y)
HORA=$(date +%H:%M:%S)
SERVICO="httpd"
STATUS=$(systemctl is-active $SERVICO)

if [ $STATUS == "active" ]; then
    MENSAGEM="O serviço $SERVICO está online"
    echo "$MENSAGEM - Data: $DATA - Hora: $HORA" >> /mnt/nfs/Bruno/servico_ON.txt
else
    MENSAGEM="O serviço $SERVICO está offline"
    echo "$MENSAGEM - Data: $DATA - Hora: $HORA" >> /mnt/nfs/Bruno/servico_OFF.txt
fi
 ```

<br>
  
### Preparando execução automatizada do script a cada 5 minutos:
  - Usar o comando `crontab -e`;
  - Adicionar a seguinte linha no arquvio que foi aberto (Substituindo a parte escrita "CAMINHO" pelo caminho correto até seu script):
  ```
  */5 * * * * /CAMINHO/script.sh
  ```

  
