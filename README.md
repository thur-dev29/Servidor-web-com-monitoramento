# Projeto com a finalidade de desenvolver e testar habilidades em Linux, AWS e automação de processos através da configuração de um ambiente  de servidor web monitorado.

# Etapa 1 - Configurando uma VPC na AWS com Sub-redes Públicas e Privadas

A etapa 1 descreve o processo de criação de uma **VPC na AWS** com **2 sub-redes públicas**, **2 sub-redes privadas** e um **Internet Gateway** para conectar as sub-redes públicas à Internet. Além disso, abordamos a criação de uma **instância EC2** e seu acesso via SSH.

## 1. Criando a VPC
1. Acesse sua conta AWS e procure pelo serviço **VPC**.
2. No menu lateral, clique em **Suas VPCs** e depois em **Criar VPC**.
3. Configure os seguintes parâmetros:
   - **Nome da VPC**: `MyVPC`
   - **CIDR IPv4**: `10.0.0.0/16`
   - **Demais configurações**: Mantenha os valores padrão.
4. Clique em **Criar VPC**.

---

## 2. Criando as Sub-redes
1. No menu lateral da VPC, vá até **Sub-redes** e clique em **Criar Sub-rede**.
2. Configure os seguintes parâmetros:
   - **ID da VPC**: Selecione a VPC recém-criada (`MyVPC`).
   - **Nome da Sub-rede**: `Public-Subnet-1`.
   - **Zona de disponibilidade**: `us-east-1a` (ou a de sua preferência).
   - **Bloco CIDR IPv4 da Sub-rede**: `10.0.1.0/24`.
3. Clique em **Criar Sub-rede**.
4. Após criar a sub-rede, selecione-a, clique em **Ações** → **Editar configurações de sub-rede** e ative **Habilitar endereço IPv4 público de atribuição automática**. **(Somente para sub-redes públicas)**.

5. **Repita o processo para criar as demais sub-redes, alterando apenas os parâmetros abaixo:**

| Nome da Sub-rede  | Tipo     | Zona de Disponibilidade | Bloco CIDR IPv4 |
|-------------------|----------|------------------------|-----------------|
| Public-Subnet-2  | Pública  | us-east-1b            | 10.0.2.0/24     |
| Private-Subnet-1 | Privada  | us-east-1a            | 10.0.3.0/24     |
| Private-Subnet-2 | Privada  | us-east-1b            | 10.0.4.0/24     |

---

## 3. Criando e Associando um Internet Gateway
1. No serviço **VPC**, acesse **Gateways da Internet**.
2. Clique em **Criar Gateway da Internet**.
3. Defina os seguintes parâmetros:
   - **Nome**: `MeuIGW`
4. Clique em **Criar Gateway da Internet**.
5. Após criar, selecione o Internet Gateway e clique em **Ações** → **Associar a uma VPC**.
6. Selecione a **VPC** (`MyVPC`) e confirme.

### 3.1 Criando a Tabela de Rotas
1. Vá até **Tabelas de Rotas** e clique em **Criar Tabela de Rotas**.
2. Configure:
   - **Nome**: `Public-Route-Table`
   - **VPC**: Selecione `MyVPC`.
3. Clique em **Criar Tabela de Rotas**.
4. Selecione a tabela criada e vá até a aba **Rotas**.
5. Clique em **Editar Rotas** → **Adicionar Rotas**.
   - **Destino**: `0.0.0.0/0`
   - **Alvo**: Selecione o **Internet Gateway (`MeuIGW`)**.
6. Salve as alterações.

### 3.2 Associando as Sub-redes Públicas à Tabela de Rotas
1. No menu **Sub-redes**, selecione **Public-Subnet-1**.
2. Clique em **Tabela de Rotas** → **Editar Associação da Tabela de Rotas**.
3. Em **ID da Tabela de Rotas**, selecione `Public-Route-Table` e salve.
4. **Repita o processo para `Public-Subnet-2`**.

---

## 4. Criando uma Instância EC2
1. No **AWS Console**, procure por **EC2**.
2. No menu lateral, clique em **Instâncias** → **Executar Instância**.
3. Configure os seguintes parâmetros:
   - **Nome**: `MeuServidor`
   - **Imagem de Sistema Operacional (AMI)**: `Debian` (ou outra de sua escolha).
   - **Par de chaves**: Selecione um já existente ou clique em **Criar Novo Par de Chaves**.
   - **Tipo de Instância**: `t2.micro` (para uso gratuito).
   - **Sub-rede**: Selecione uma das **sub-redes públicas**.
   - **Grupo de Segurança**: Crie ou selecione um que contenha:
     - **Permitir tráfego SSH (porta 22)** de `0.0.0.0/0`.
     - **Permitir tráfego HTTP (porta 80)** de `0.0.0.0/0`.
4. Clique em **Executar Instância**.

---

## 5. Acessando a Instância via SSH
Após a instância estar rodando, siga os passos abaixo para acessá-la:

1. Vá para **EC2** → **Instâncias**.
2. Selecione a instância e copie o **Endereço IPv4 Público**.
3. No seu terminal (Linux/macOS) ou PowerShell (Windows), execute:

```bash
ssh -i "caminho/para/sua-chave.pem" admin@seu-ip-publico
```

- Substitua `caminho/para/sua-chave.pem` pelo caminho real do arquivo `.pem` da chave privada.
- Substitua `seu-ip-publico` pelo IP da instância.

---

## Conclusão
Agora você configurou uma **VPC na AWS** com **sub-redes públicas e privadas**, criou um **Internet Gateway** e configurou uma **instância EC2** acessível via SSH.

# Etapa 2 - Configuração do Servidor Web

A etapa 2 descreve o processo de configuração de um servidor web em uma instância EC2 na AWS.

## 1. Instalando o Nginx

Agora que está conectado via SSH à sua instância, execute os seguintes comandos:

```bash
sudo apt update && sudo apt install -y nginx
```

Isso irá baixar e instalar o Nginx.

### O que é o Nginx?
O **Nginx** é um servidor web leve e eficiente, usado para hospedar sites, fazer balanceamento de carga e funcionar como proxy reverso. Ele é conhecido por sua alta performance, baixo consumo de recursos e capacidade de lidar com milhares de conexões simultâneas. Além disso, é muito usado para melhorar a segurança e otimizar o tráfego em aplicações web.

Após a instalação, verifique se o serviço está rodando:

```bash
sudo systemctl status nginx
```

Se aparecer **"active (running)"**, o Nginx está funcionando. Caso precise iniciar o serviço manualmente, use:

```bash
sudo systemctl start nginx
```

## 2. Criando uma Página HTML
Agora é preciso criar uma página HTML para poder exibir no navegador.

Acesse o diretório padrão do Nginx:

```bash
cd /var/www/html
```

Crie um arquivo HTML:

```bash
sudo nano index.html
```

Adicione algum conteúdo ao arquivo, depois salve e saia do editor (**Ctrl + X**, depois **Y** e **Enter**).

## 3. Configurar o Nginx para Servir a Página

Reinicie o Nginx para carregar as mudanças:

```bash
sudo systemctl restart nginx
```

Verifique se o serviço está rodando corretamente:

```bash
sudo systemctl status nginx
```

Agora, teste no navegador. Pegue o **IP Público** da sua instância e acesse no navegador:

```
http://SEU_IP_PUBLICO
```

Se estiver tudo certo, a página será exibida.

## 4. Criar um Serviço systemd para Garantir que o Nginx Reinicie Automaticamente

O Nginx já vem configurado para iniciar automaticamente, mas vamos garantir que ele reinicie caso falhe.

Ative o Nginx para iniciar com o sistema:

```bash
sudo systemctl enable nginx
```

Para garantir que o Nginx reinicie automaticamente em caso de falha, crie um serviço systemd:

```bash
sudo nano /etc/systemd/system/nginx.service
```

Adicione estas linhas:

```
[Unit]
Description=Nginx Web Server
After=network.target

[Service]
ExecStart=/usr/sbin/nginx -g 'daemon off;'
Restart=Always
RestartSec=5s

[Install]
WantedBy=multi-user.target
```

Salve (**Ctrl + X**, depois **Y** e **Enter**) e ative o novo serviço:

```bash
sudo systemctl daemon-reload
sudo systemctl restart nginx
```

```bash
sudo systemctl daemon-reload
```

Este comando informa ao sistema para recarregar as definições dos serviços. Isso é útil quando você faz alterações em arquivos de configuração de serviços, o daemon-reload é necessário para que o systemd reconheça essas alterações.

```bash
sudo systemctl restart nginx
```

Este comando reinicia o serviço do Nginx. Quando você reinicia o Nginx, ele para o serviço em execução e o inicia novamente. Isso é útil para aplicar alterações de configuração ou para resolver problemas com o serviço.
