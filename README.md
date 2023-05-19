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

---
# AWS:
### Criando um usuário IAM
- Ir no serviço IAM da AWS;
- No menu da esquerda clicar em "Usuários" e depois "Adicionar usuários";
- Escolher um nome para o usuário e nas opções de permissões selecionar Anexar políticas diretamente;
- Escolher a política Administrator Access;
- Após seguir esses passos, criar uma chave de acesso para o usuário criado, para usar no terminal com o comando `aws configure`, como no exemplo abaixo:
<div align="center">
  
![image](https://github.com/BrunoMarques1/Atvidade_pratica_aws/assets/127341401/bc029516-1680-45a1-a45c-b0ab36f5b1d8)
  
</div>
<br>

### Criando par de chaves
- Já dentro do serviço EC2 da AWS, clicar em "Pares de Chaves", na parte de Rede e segurança;
- Clicar em "Criar par de chaves";
- Escolher um nome, o tipo e o formato para a chave. No meu caso o nome inserido foi "ChaveC", o tipo escolhido foi RSA e o formato foi .pem;
- Salvar o arquivo da chave.
<br>

### Configurando security group
- Após a criação do par de chaves, ainda na parte de Rede e segurança, clicar em "Security Groups";
- Clicar em "Criar grupo de segurança";
- Escolher um nome para o security group, No meu o nome será "SGa1"
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
- Para criar a instância EC2 usar o seguinte comando via CLI: 
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
    - `--tag-specifications "ResourceType=instance,Tags=[{Key=Name,Value=Name},{Key=Key,Value=Value},{Key=Key,Value=Value}]"  "ResourceType=volume,Tags=[{Key=Name,Value=Name},{Key=Key,Value=Value},{Key=Key,Value=Value}]" `
