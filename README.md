# curso-aws

---

## ☁️ AWS IAM (Identity and Access Management)

O **IAM** é o serviço da **AWS** responsável por **gerenciar identidades, permissões e credenciais de acesso** de forma segura.  
Com ele, você controla **quem** pode fazer **o quê** e **em qual recurso** dentro da AWS.

---

### 🔹 Principais Conceitos

- **Usuários (Users)** → Entidades individuais com credenciais próprias.
- **Grupos (Groups)** → Coleção de usuários para aplicar permissões de forma centralizada.
- **Funções (Roles)** → Identidades temporárias usadas por serviços, aplicações ou contas externas.
- **Políticas (Policies)** → Documentos JSON que definem permissões (Allow / Deny).
- **Credenciais** →  
  - **Acesso via Console**: login com senha.  
  - **Acesso Programático**: Access Key ID e Secret Access Key.

---

### 🛠️ Ações Comuns

#### Criar um usuário no Console AWS
1. Vá em **IAM** → **Users** → **Add users**.
2. Defina **nome**, tipo de acesso (console / programático).
3. Atribua permissões:
   - **Add user to group**
   - **Attach policies directly**
   - **Copy permissions from another user**
4. Finalize e guarde as credenciais.

---

### 📜 Exemplo de Política (JSON)

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:ListBucket",
                "s3:GetObject"
            ],
            "Resource": [
                "arn:aws:s3:::meu-bucket",
                "arn:aws:s3:::meu-bucket/*"
            ]
        }
    ]
}
```
Essa política permite **listar** e **baixar arquivos** de um bucket específico no S3.

---

### 🔒 Boas Práticas

- **Princípio do menor privilégio**: conceda apenas as permissões necessárias.
- Use **MFA (autenticação multifator)** para usuários sensíveis.
- Não use o usuário **root** da conta no dia a dia.
- Faça **rotação periódica** das chaves de acesso.
- Prefira **Roles** para serviços e aplicações (em vez de chaves fixas).

---

### 🧪 Testar Permissões

Você pode simular políticas no **IAM Policy Simulator**:  
<https://policysim.aws.amazon.com>

---

### 🔁 Comandos AWS CLI para IAM

```bash
# Criar um usuário
aws iam create-user --user-name Joao

# Criar um grupo
aws iam create-group --group-name Desenvolvedores

# Adicionar usuário ao grupo
aws iam add-user-to-group --user-name Joao --group-name Desenvolvedores

# Anexar política a um grupo
aws iam attach-group-policy --group-name Desenvolvedores \
    --policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess

# Criar credenciais de acesso
aws iam create-access-key --user-name Joao
```

---
---

## 🛡️ Criar Políticas no IAM

As **políticas** (policies) definem permissões na AWS em formato **JSON**, especificando:
- **Ação (Action)** → O que pode ser feito (ex.: `s3:ListBucket`).
- **Recurso (Resource)** → Onde a ação pode ser executada (ex.: um bucket S3 específico).
- **Efeito (Effect)** → Permitir (`Allow`) ou negar (`Deny`).

---

### 📌 Criando Política pelo Console AWS

1. No Console AWS, acesse **IAM** → **Policies** → **Create policy**.
2. Escolha:
   - **Visual editor** → Interface guiada.
   - **JSON** → Escrever manualmente.
3. Defina as permissões.
4. Adicione nome e descrição.
5. Criar a política.

---

### 📜 Exemplo de Política JSON

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "ec2:StartInstances",
                "ec2:StopInstances"
            ],
            "Resource": [
                "arn:aws:ec2:us-east-1:123456789012:instance/i-0abcd1234efgh5678"
            ]
        }
    ]
}
```
Essa política **permite iniciar e parar** apenas uma instância EC2 específica.

---

### 🖥️ Criar Política via AWS CLI

```bash
# Criar política a partir de um arquivo JSON
aws iam create-policy \
    --policy-name EC2StartStopOnly \
    --policy-document file://minha-politica.json
```

---

### 📂 Estrutura de um Documento de Política

- **Version** → Versão do formato de política (use sempre `"2012-10-17"`).
- **Statement** → Lista de permissões.
  - **Effect**: `"Allow"` ou `"Deny"`.
  - **Action**: Lista de ações permitidas ou negadas.
  - **Resource**: ARN do recurso ou `"*"` para todos.
  - (Opcional) **Condition**: Restrições extras.

---

### 🔒 Boas Práticas ao Criar Políticas

- Usar **ARNs específicos** em vez de `"*"` para restringir escopo.
- Criar políticas **customizadas** apenas quando necessário — preferir **Managed Policies** da AWS.
- Adicionar **tags** para facilitar organização e auditoria.
- Testar no **IAM Policy Simulator** antes de aplicar.

---
---

## 🔐 Política de Senhas no IAM (AWS)

A **política de senhas** define regras para a criação e gerenciamento de senhas de usuários do IAM.  
Essa configuração **afeta todos os usuários IAM** da conta.

---

### 📌 Configurando pelo Console AWS

1. No Console AWS, acesse:
   **IAM** → **Account settings** → **Password policy**.
2. Clique em **Set password policy**.
3. Configure opções como:
   - Comprimento mínimo da senha.
   - Exigência de caracteres maiúsculos, minúsculos, números e símbolos.
   - Expiração automática de senha.
   - Histórico de senhas (impede repetição).
4. Salve a política.

---

### 📜 Exemplo de Política de Senha Recomendada

| Configuração | Valor Sugerido |
|--------------|----------------|
| Comprimento mínimo | 12 caracteres |
| Letra maiúscula obrigatória | Sim |
| Letra minúscula obrigatória | Sim |
| Número obrigatório | Sim |
| Caractere especial obrigatório | Sim |
| Expiração de senha | 90 dias |
| Histórico de senhas | Últimas 5 não podem ser reutilizadas |
| Permitir alteração própria | Sim |
| Exigir redefinição na criação | Sim |

---

### 🖥️ Configurando via AWS CLI

```bash
aws iam update-account-password-policy \
    --minimum-password-length 12 \
    --require-symbols \
    --require-numbers \
    --require-uppercase-characters \
    --require-lowercase-characters \
    --allow-users-to-change-password \
    --max-password-age 90 \
    --password-reuse-prevention 5
```

---

### 🔎 Verificando a Política Atual

```bash
aws iam get-account-password-policy
```

---

### 🔒 Boas Práticas

- Usar **12+ caracteres**.
- Exigir **caracteres mistos** (maiúsculo, minúsculo, número, símbolo).
- Aplicar **MFA** para maior segurança.
- Revisar periodicamente a política.
- Não permitir senhas fracas ou genéricas.

---

# ☁️ Amazon EC2

O **Amazon EC2 (Elastic Compute Cloud)** é um serviço da AWS que fornece capacidade de computação escalável na nuvem.  
Ele permite criar e gerenciar **instâncias (máquinas virtuais)** de acordo com a demanda.

---

## 🔑 Principais características

- Criação de servidores virtuais (instâncias) sob demanda.
- Diversos tipos de instâncias: otimizadas para CPU, memória, armazenamento ou GPU.
- Integração com outros serviços AWS (S3, RDS, CloudWatch, etc).
- Escalabilidade: ajuste de capacidade automaticamente com **Auto Scaling**.
- Segurança: uso de **Security Groups**, **VPC** e **Key Pairs**.

---

## 🛠️ Passos básicos para criar uma instância EC2

1. Acesse o **Console da AWS** e vá até **EC2**.
2. Clique em **Launch Instance**.
3. Escolha:
   - Uma **AMI (Amazon Machine Image)** → ex: Ubuntu, Amazon Linux, etc.
   - O **tipo da instância** → ex: `t2.micro` (free tier).
4. Configure:
   - **Key Pair** → necessário para login via SSH.
   - **Security Group** → defina portas abertas (ex: 22/SSH, 80/HTTP).
5. Revise e clique em **Launch**.
6. Sua instância estará disponível no painel.

---

## 💻 Conectar via SSH

No terminal:

```bash
ssh -i "sua-chave.pem" ubuntu@ip_publico_da_instancia
```

Exemplo:

```bash
ssh -i "minha-chave.pem" ec2-user@54.210.123.45
```

---

## 📦 Exemplos de uso

- Hospedar sites com **Nginx ou Apache**.
- Rodar containers com **Docker**.
- Servidores de banco de dados ou aplicações corporativas.
- Ambientes de testes e desenvolvimento.

---

> ✅ O EC2 é a base da AWS para servidores virtuais na nuvem, ideal para aprender infraestrutura e implantar aplicações.

# 💾 Amazon EBS e Snapshots

## 📌 O que é EBS?

O **Amazon EBS (Elastic Block Store)** é o serviço de armazenamento em bloco da AWS.  
Ele é usado principalmente para fornecer **volumes persistentes** que podem ser anexados a instâncias **EC2**.

### 🔑 Características do EBS

- Armazenamento **persistente**: os dados permanecem mesmo após desligar a instância.
- Pode ser anexado/desanexado de uma instância EC2.
- Tipos de volumes: otimizados para **desempenho** (SSD) ou **custo/armazenamento** (HDD).
- Criptografia nativa com **KMS**.
- Permite **resize** (aumentar tamanho e mudar tipo de volume sem parar instância).

---

## 📌 O que são Snapshots?

Um **Snapshot** é uma **cópia pontual (backup)** de um volume EBS.  
Eles são armazenados no **Amazon S3** (internamente) e podem ser usados para:

- Restaurar volumes em caso de falhas.
- Criar novos volumes a partir de um snapshot existente.
- Migrar volumes entre **regiões** ou **contas AWS**.

---

## 🛠️ Exemplos de uso

### ➕ Criar um volume EBS e anexar a uma instância

```bash
# Criar volume de 20 GB na zona us-east-1a
aws ec2 create-volume \
  --size 20 \
  --availability-zone us-east-1a \
  --volume-type gp2

# Anexar o volume criado a uma instância
aws ec2 attach-volume \
  --volume-id vol-1234567890abcdef0 \
  --instance-id i-1234567890abcdef0 \
  --device /dev/sdf
```

---

### 📸 Criar um Snapshot

```bash
aws ec2 create-snapshot \
  --volume-id vol-1234567890abcdef0 \
  --description "Backup do volume de dados"
```

---

### 🔄 Criar volume a partir de um Snapshot

```bash
aws ec2 create-volume \
  --snapshot-id snap-1234567890abcdef0 \
  --availability-zone us-east-1a \
  --volume-type gp2
```

---

## ✅ Resumindo

- **EBS** → Armazenamento em bloco para EC2.  
- **Snapshot** → Backup/cópia de segurança de um volume EBS.  

Combinando os dois, você garante **persistência, recuperação e escalabilidade** para os dados em sua infraestrutura na AWS.
# 🖼️ Amazon Machine Image (AMI)

## 📌 O que é uma AMI?

A **Amazon Machine Image (AMI)** é uma **imagem pré-configurada** usada para lançar instâncias no **Amazon EC2**.  
Ela contém todas as informações necessárias para iniciar uma máquina virtual na AWS, incluindo:

- Um **sistema operacional** (Linux, Windows, etc.)
- **Configurações do sistema**
- Softwares e pacotes pré-instalados
- Permissões de acesso (quem pode usar a AMI)

---

## 🔑 Tipos de AMIs

1. **AMIs da AWS**  
   - Fornecidas oficialmente pela Amazon (ex: Amazon Linux 2).  

2. **AMIs da Comunidade**  
   - Criadas e disponibilizadas por outros usuários da AWS.  

3. **AMIs Personalizadas**  
   - Criadas a partir de instâncias existentes, com seus softwares e configurações específicas.  

4. **AMIs do Marketplace**  
   - Imagens pagas ou gratuitas fornecidas por terceiros (ex: Ubuntu Pro, Red Hat, Windows Server, softwares prontos).  

---

## 🛠️ Criar uma AMI personalizada

1. Configure sua instância EC2 (instale pacotes, configure serviços, etc.).
2. No console da AWS, selecione a instância.
3. Clique em **Actions → Image and templates → Create image**.
4. Defina nome e descrição.
5. A AWS criará a AMI e ela ficará disponível para lançar novas instâncias.

---

## 💻 Exemplo com AWS CLI

Criar uma AMI a partir de uma instância em execução:

```bash
aws ec2 create-image \
  --instance-id i-1234567890abcdef0 \
  --name "minha-ami-personalizada" \
  --description "AMI com Nginx instalado"
```

---

## ✅ Resumindo

- A **AMI** é como um **template de servidor**.  
- Serve para padronizar ambientes, acelerar deploys e facilitar escalabilidade.  
- Você pode usar AMIs prontas (da AWS ou comunidade) ou criar as suas personalizadas.

# 🏗️ EC2 Image Builder

## 📌 O que é o EC2 Image Builder?

O **EC2 Image Builder** é um serviço da AWS que automatiza a **criação, teste e atualização de AMIs** (Amazon Machine Images).  
Ele permite manter imagens sempre atualizadas, seguras e consistentes para suas instâncias EC2.

---

## 🔑 Principais funcionalidades

- Criar **AMIs automatizadas** com softwares pré-instalados.
- Aplicar **patches de segurança** automaticamente.
- Testar imagens antes de disponibilizá-las.
- Integrar com **AWS Lambda, CloudWatch e S3** para pipelines automatizadas.
- Reduz erros e trabalho manual na manutenção de imagens.

---

## 🛠️ Como funciona

1. **Pipeline de imagens**  
   Define as etapas para criar e configurar a AMI.

2. **Componentes**  
   Scripts ou pacotes que serão instalados ou configurados na imagem.

3. **Distribuição**  
   Após a criação e teste, a AMI pode ser distribuída para regiões específicas.

---

## 💻 Exemplo de criação de AMI com AWS CLI

Criar um pipeline simples (exemplo resumido):

```bash
aws imagebuilder create-image-pipeline \
  --name "MeuPipelineAMI" \
  --image-recipe-arn arn:aws:imagebuilder:us-east-1:123456789012:image-recipe/meu-recipe/1.0.0 \
  --infrastructure-configuration-arn arn:aws:imagebuilder:us-east-1:123456789012:infrastructure-configuration/minha-infra-config \
  --distribution-configuration-arn arn:aws:imagebuilder:us-east-1:123456789012:distribution-configuration/minha-dist-config
```

> Observação:  
> - A `image-recipe` define qual SO, softwares e configurações a AMI terá.  
> - A `infrastructure-configuration` define como a imagem será construída (tipo de instância, VPC, sub-rede).  
> - A `distribution-configuration` define para quais regiões a AMI será disponibilizada.

---

## ✅ Resumindo

- O **EC2 Image Builder** automatiza a criação e atualização de AMIs.  
- Facilita manter imagens consistentes, seguras e prontas para deploy.  
- Ideal para ambientes corporativos que precisam de **padronização e compliance**.

# Amazon EFS (Elastic File System)

O **Amazon Elastic File System (EFS)** é um serviço de armazenamento de arquivos totalmente gerenciado e escalável, ideal para aplicações que precisam de acesso simultâneo a arquivos por múltiplas instâncias, como servidores web, containers e funções serverless.

---

## 💰 Preços do Amazon EFS

O custo varia conforme a classe de armazenamento e a região. Exemplo para **EUA Leste (N. Virgínia)**:

| Classe de armazenamento        | Preço por GB/mês |
|--------------------------------|----------------|
| EFS Standard                   | US$ 0,30       |
| EFS Infrequent Access (IA)    | US$ 0,025      |
| EFS One Zone                   | US$ 0,16       |
| EFS One Zone IA                | US$ 0,0133     |
| Provisioned Throughput         | US$ 6 por MB/s/mês |

> Além disso, podem haver cobranças adicionais por operações de leitura e escrita, especialmente nas classes IA e One Zone IA.

---

## 🇧🇷 Disponibilidade no Brasil

- Data centers localizados em São Paulo.  
- Faturamento em Reais (BRL) com emissão de nota fiscal eletrônica (NFS-e).  

---

## 📊 Comparativo: Google Drive vs. Amazon EFS

| Recurso                     | Google Drive                          | Amazon EFS                                  |
|------------------------------|--------------------------------------|---------------------------------------------|
| **Modelo de cobrança**        | Por usuário                           | Por GB armazenado e operações realizadas    |
| **Facilidade de uso**         | Alta (interface amigável)             | Requer configuração técnica                 |
| **Escalabilidade**            | Limitada                              | Altamente escalável                         |
| **Controle e personalização** | Baixo                                 | Alto                                        |
| **Integração com sistemas**   | Limitada                              | Total (ideal para aplicações empresariais) |
| **Preço por GB**              | Não aplicável                          | US$ 0,30/mês (EFS Standard)                |

> O Google Drive é mais indicado para uso pessoal ou pequenos grupos, enquanto o Amazon EFS é ideal para empresas que precisam de alta disponibilidade, escalabilidade e integração com outras soluções na nuvem.

# AWS Lambda

O **AWS Lambda** é um serviço de computação serverless da Amazon que permite executar código sem provisionar ou gerenciar servidores. Você paga apenas pelo tempo de execução do código e pelo número de solicitações. Ideal para automação, APIs, processamento de dados em tempo real e integração com outros serviços da AWS.

---

## 💡 Principais características do AWS Lambda

- **Execução sob demanda**: O código é executado apenas quando necessário.  
- **Escalabilidade automática**: O Lambda escala automaticamente com o volume de solicitações.  
- **Suporte a várias linguagens**: Python, Node.js, Java, Go, Ruby, .NET e custom runtimes.  
- **Integração com AWS**: Funciona com S3, DynamoDB, SNS, SQS, API Gateway, CloudWatch, entre outros.  
- **Modelo de cobrança**: Baseado em número de solicitações e duração da execução (GB-segundo).

---

## 💰 Preços do AWS Lambda

| Métrica                         | Limite Gratuito              | Preço após limite                           |
|---------------------------------|------------------------------|--------------------------------------------|
| **Solicitações**                 | 1 milhão/mês                | US$ 0,20 por 1 milhão de solicitações      |
| **Duração da execução**          | 400.000 GB-segundos/mês     | US$ 0,0000166667 por GB-segundo            |

> Exemplo: Uma função de 512 MB rodando por 1 segundo gera 0,5 GB-segundo de custo.

---

## 📊 Comparativo rápido: Lambda vs Servidores tradicionais

| Recurso                     | Servidor Tradicional         | AWS Lambda                     |
|------------------------------|----------------------------|--------------------------------|
| **Modelo de execução**        | Servidor sempre ligado      | Sob demanda (serverless)       |
| **Escalabilidade**            | Manual ou semi-automática   | Automática                     |
| **Custo**                     | Fixo (mesmo sem uso)        | Só paga pelo uso               |
| **Manutenção**                | Alta (SO, updates, patches) | Baixa (totalmente gerenciado) |
| **Integração com AWS**        | Limitada                   | Total                          |
| **Tempo de execução máximo**  | Ilimitado                  | 15 minutos por invocação       |

> O Lambda é ideal para workloads event-driven, enquanto servidores tradicionais podem ser melhores para aplicações persistentes ou de longa execução.
>
# ⚖️ Load Balancers

## 📌 O que é um Load Balancer?

Um **Load Balancer** distribui automaticamente o tráfego de rede ou de aplicações entre **múltiplos servidores ou instâncias**, garantindo:

- Alta disponibilidade
- Melhor desempenho
- Tolerância a falhas

No contexto da AWS, o serviço é chamado **Elastic Load Balancing (ELB)**.

---

## 🔑 Tipos de Load Balancers na AWS

1. **Application Load Balancer (ALB)**  
   - Trabalha na camada 7 (HTTP/HTTPS).  
   - Ideal para aplicações web, suporte a rotas baseadas em URL, host ou headers.

2. **Network Load Balancer (NLB)**  
   - Trabalha na camada 4 (TCP/UDP).  
   - Alta performance e baixa latência, ideal para cargas pesadas ou aplicações em tempo real.

3. **Classic Load Balancer (CLB)**  
   - Legado, funciona na camada 4 e 7, mas sem todos os recursos avançados do ALB e NLB.

---

## 🛠️ Como funciona

- O LB recebe requisições de clientes.  
- Ele distribui essas requisições para **instâncias saudáveis** do seu grupo de destino.  
- Realiza **health checks** periódicos para detectar instâncias com problemas e removê-las do pool temporariamente.

---

## 💻 Exemplo de criação de Load Balancer via AWS CLI

```bash
# Criar um Application Load Balancer
aws elbv2 create-load-balancer \
  --name meu-alb \
  --subnets subnet-12345 subnet-67890 \
  --security-groups sg-123456 \
  --scheme internet-facing \
  --type application
```

```bash
# Criar target group para o ALB
aws elbv2 create-target-group \
  --name meu-targets \
  --protocol HTTP \
  --port 80 \
  --vpc-id vpc-123456
```

```bash
# Registrar instâncias no target group
aws elbv2 register-targets \
  --target-group-arn arn:aws:elasticloadbalancing:...:targetgroup/meu-targets \
  --targets Id=i-123456 Id=i-789012
```

---

## ✅ Resumindo

- Load Balancer distribui tráfego entre várias instâncias, aumentando **disponibilidade e escalabilidade**.  
- Na AWS, você pode escolher ALB, NLB ou CLB dependendo do tipo de aplicação.  
- Integração com **Auto Scaling** garante que novas instâncias sejam automaticamente adicionadas ao balanceamento.

<img width="1139" height="567" alt="image" src="https://github.com/user-attachments/assets/c6c192f7-c645-421d-80c6-4d4923ae8d22" />

<img width="765" height="329" alt="image" src="https://github.com/user-attachments/assets/469c66ac-6db9-4e0a-8de1-e9a017d45e60" />


