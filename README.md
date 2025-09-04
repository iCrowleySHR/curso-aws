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
<br>
<img width="991" height="678" alt="image" src="https://github.com/user-attachments/assets/ec8a7393-c7f1-40b5-a5de-97e64c84da65" />


# 🔄 Auto Scaling Group (ASG)

## 📌 O que é um Auto Scaling Group?

Um **Auto Scaling Group** é um recurso da AWS que gerencia automaticamente a **quantidade de instâncias EC2** em execução.  
Ele garante que sua aplicação esteja sempre disponível, ajustando os recursos conforme a demanda e substituindo instâncias com falha.

---

## 🔑 Principais conceitos

- **Launch Template/Configuration** → Define como as instâncias serão criadas (AMI, tipo de instância, rede, etc).  
- **Desired Capacity** → Número de instâncias desejadas em execução.  
- **Min/Max Size** → Limites mínimo e máximo de instâncias no grupo.  
- **Scaling Policies** → Regras que aumentam ou reduzem instâncias com base em métricas (ex: CPU, memória, requisições).  
- **Health Checks** → Monitoram instâncias; se uma falhar, o ASG a substitui automaticamente.  
- **Distribuição em AZs** → O ASG pode criar instâncias em várias Zonas de Disponibilidade, aumentando a resiliência.  

---

## 🛠️ Como funciona na prática

1. Você define um **modelo de instância** (Launch Template).  
2. O **ASG** inicia as instâncias com base nesse modelo.  
3. O grupo mantém sempre a **quantidade desejada** de instâncias:  
   - Se uma instância falhar → é substituída.  
   - Se a demanda aumentar → novas instâncias são criadas.  
   - Se a demanda cair → instâncias são encerradas.  

---

## ✅ Benefícios

- **Alta disponibilidade** → sempre mantém instâncias ativas e saudáveis.  
- **Escalabilidade automática** → responde a mudanças de tráfego ou carga.  
- **Otimização de custos** → apenas os recursos necessários são usados.  
- **Integração com Load Balancers** → distribui o tráfego entre todas as instâncias do grupo.  

---

## 🔗 Fluxo simplificado

```
Cliente → Load Balancer → Auto Scaling Group → EC2 Instances
```

# ☁️ Amazon S3 (Simple Storage Service)

## 📌 O que é o Amazon S3?

O **Amazon S3** é um serviço de armazenamento de objetos na nuvem, altamente escalável e durável, usado para guardar **arquivos, backups, imagens, vídeos e dados de aplicações**.

- Cada arquivo é chamado de **objeto**.  
- Cada objeto é armazenado em um **bucket**, que funciona como uma pasta ou repositório.  
- O S3 é acessível via **AWS Management Console, CLI, SDKs** ou APIs.

---

## 🔑 Características principais

- **Durabilidade de 99,999999999%** dos dados (11 9’s).  
- **Escalabilidade automática** → suporta qualquer quantidade de dados e acessos.  
- **Controle de acesso** → via políticas, IAM e ACLs.  
- **Versionamento** → mantém diferentes versões de um objeto.  
- **Integração com outros serviços AWS** → Lambda, CloudFront, EC2, Athena, etc.  
- **Suporte a criptografia** → SSE-S3, SSE-KMS, SSE-C.

---

## 🛠️ Exemplos de uso

- Hospedagem de arquivos estáticos para sites (HTML, CSS, JS).  
- Backup e arquivamento de dados.  
- Armazenamento de logs e dados de aplicações.  
- Integração com pipelines de dados (ETL, analytics).  
- Distribuição de conteúdo com **CloudFront**.

---

## ✅ Benefícios

- **Alta disponibilidade e durabilidade**.  
- **Fácil gerenciamento e escalabilidade**.  
- **Segurança e controle de acesso granulares**.  
- **Custo eficiente** → paga apenas pelo que usa.  

---

## 🔗 Fluxo simplificado

```
Usuário/Aplicação → Bucket S3 → Objetos (arquivos)
```

# 🔁 S3 Replication: CRR e SRR

## 📌 Visão geral
O **Amazon S3 Replication** copia objetos automaticamente de um **bucket de origem** para um **bucket de destino** conforme regras de replicação.  
Existem dois modos:
- **CRR (Cross-Region Replication):** replica para **outra região** (ex.: `us-east-1` ➜ `sa-east-1`).
- **SRR (Same-Region Replication):** replica **dentro da mesma região** (ex.: `us-east-1` ➜ `us-east-1`).

> A replicação é **assíncrona** e, por padrão, aplica-se a **novos objetos** (ou a existentes via **S3 Batch Replication**).

---

## ✅ Requisitos
- **Versionamento** **habilitado** no bucket **origem e destino**.
- **Permissões IAM** para o S3 replicar em nome da conta/role.
- (Opcional) **Criptografia KMS:** a role precisa de permissão para **usar a CMK** (origem e destino).

---

## 🧩 Quando usar cada um
**CRR (entre regiões)**
- **DR/BCP** (recuperação de desastre).
- **Latência menor** para usuários de outra região.
- **Compliance** que exige cópia geograficamente separada.

**SRR (mesma região)**
- **Separação de ambientes** (prod ➜ analytics).
- **Contabilidade/auditoria** com cópia imutável (WORM/S3 Object Lock).
- **Fluxos de dados** (ex.: origens diferentes centralizando em um bucket).

---

## 🔎 Como funciona (alto nível)
1. Você cria **regras de replicação** por **prefixo** e/ou **tags**.
2. O S3 detecta **novas gravações/versões** e replica para o destino.
3. Itens excluídos podem replicar **delete markers** (conforme a regra).
4. Metadados e ACLs/Ownership podem ser ajustados (ex.: **Change object ownership to destination**).

---

## ⚙️ Opções importantes na regra
- **Filtro por prefixo** (ex.: `logs/`) e/ou **tags** (ex.: `env=prod`).
- **RTC (Replication Time Control):** SLA de replicação (custo extra).
- **Replica delete markers / replica delete** (controla propagação de exclusões).
- **Replica metadados e objetos criptografados por KMS** (exige permissões KMS).
- **Alterar proprietário no destino** (útil para replicar entre **contas**).

---

## 💰 Custos (resumo)
- **Requests** de replicação + **dados transferidos** (no CRR há **data transfer inter-region**).
- **Armazenamento** no destino.
- **RTC** (se habilitado) cobra adicional.

---

## ⚠️ Limitações e pegadinhas
- Replicação é **unidirecional** por regra.
- Por padrão **não replica objetos antigos** — use **S3 Batch Replication** para retroagir.
- **Server-side encryption (SSE-KMS)** requer permissões **kms:Encrypt/Decrypt/ReEncrypt** adequadas.
- **Object Ownership/ACLs**: confira se o destino receberá a **propriedade** correta (evita “ownership issues”).
- **Ciclos**: evite loop (origem ➜ destino ➜ origem); use filtros e **replication time**/ID para prevenir.

---

## 🧭 Passos (Console/alto nível)
1. Habilite **Versioning** nos dois buckets.
2. Em **Management → Replication**, crie uma **regra**:
   - Escolha **SRR** ou **CRR** e o **bucket destino** (e conta, se diferente).
   - Defina **prefix/tag filters**.
   - (Opcional) Habilite **RTC**.
   - Se usar **KMS**, selecione chaves e permita o uso pela role.
3. Salve a regra e **valide** com upload de teste.
4. (Opcional) Rode **S3 Batch Replication** para objetos pré-existentes.

---

## 🧪 Casos de uso comuns
- **CRR:** Prod (US) ➜ DR (BR) com RTC para RTO/RPO agressivos.
- **SRR:** Prod ➜ Data Lake (mesma região) para analytics com políticas distintas.
- **Entre contas:** Origem (Conta A) ➜ Destino (Conta B) com **ownership** no destino.

---

## 🧷 Checklist rápido
- [ ] Versioning ON em origem e destino  
- [ ] Role de replicação com permissões S3 e (se preciso) KMS  
- [ ] Filtro por **prefix/tags** definido  
- [ ] Decisão sobre **delete markers** e **RTC**  
- [ ] Teste de upload e verificação de **status de replicação** no objeto  
- [ ] (Se necessário) **Batch Replication** para históricos

---

# 🗂️ Classes de Armazenamento do Amazon S3

## S3 Standard
- **Uso:** Dados acessados com frequência.  
- **Tempo de recuperação:** Milissegundos.  
- **Custo relativo:** Alto.  
- **Exemplo:** Sites, APIs, conteúdo ativo.

---

## S3 Standard-IA (Infrequent Access)
- **Uso:** Dados acessados raramente, mas que precisam de recuperação rápida.  
- **Tempo de recuperação:** Milissegundos.  
- **Custo relativo:** Moderado (menor que Standard; cobrança por recuperação).  
- **Exemplo:** Backups e dados de DR acessados ocasionalmente.

---

## S3 One Zone-IA
- **Uso:** Dados infrequentes armazenados em **uma única** Zona de Disponibilidade.  
- **Tempo de recuperação:** Milissegundos.  
- **Custo relativo:** Mais baixo que Standard-IA.  
- **Exemplo:** Cópias secundárias ou dados que podem ser recriados.

---

## S3 Intelligent-Tiering
- **Uso:** Dados com padrão de acesso imprevisível.  
- **Tempo de recuperação:** Milissegundos (camadas de acesso automático).  
- **Custo relativo:** Variável — otimizado automaticamente entre camadas.  
- **Exemplo:** Coleções com acesso irregular que você não quer classificar manualmente.

---

## S3 Glacier Instant Retrieval
- **Uso:** Arquivamento de longo prazo **com necessidade de acesso imediato** quando solicitado.  
- **Tempo de recuperação:** Milissegundos (acesso instantâneo).  
- **Custo relativo:** Baixo (mais barato que Standard-IA para armazenamento; custo de recuperação também baixo).  
- **Exemplo:** Arquivos raramente acessados que, quando necessários, devem estar disponíveis imediatamente (ex.: ativos digitais grandes, certos catálogos).

---

## S3 Glacier Flexible Retrieval (antes Glacier)
- **Uso:** Arquivamento de longo prazo com recuperação flexível.  
- **Tempo de recuperação:** Minutos a horas (opções de prioridade).  
- **Custo relativo:** Muito baixo para armazenamento; custo de recuperação varia conforme a opção.  
- **Exemplo:** Arquivos de compliance ou arquivamento onde latência de minutos/hours é aceitável.

---

## S3 Glacier Deep Archive
- **Uso:** Arquivamento de muito longo prazo (anos) para dados raramente acessados.  
- **Tempo de recuperação:** Horas (pode chegar a 12+ horas dependendo da opção).  
- **Custo relativo:** O mais baixo entre as classes S3.  
- **Exemplo:** Retenção legal, arquivamento regulatório de longo prazo.

---

## Tabela resumo (comparativa rápida)

| Classe                          | Uso principal                            | Recuperação         | Custo relativo |
|---------------------------------|------------------------------------------|---------------------|----------------|
| S3 Standard                     | Acesso frequente                         | Milissegundos       | Alto           |
| S3 Standard-IA                  | Acesso infrequente, rápido               | Milissegundos       | Moderado       |
| S3 One Zone-IA                  | Infrequente, 1 AZ                        | Milissegundos       | Baixo          |
| S3 Intelligent-Tiering          | Padrão imprevisível (auto-tiering)       | Milissegundos       | Variável       |
| **S3 Glacier Instant Retrieval**| Arquivamento com necessidade instantânea | Milissegundos       | Baixo          |
| S3 Glacier Flexible Retrieval   | Arquivamento (recuperação flexível)     | Minutos → Horas     | Muito baixo    |
| S3 Glacier Deep Archive         | Arquivamento extremo (anos)             | Horas               | Mínimo         |

---

**Observação:** custos reais dependem de: armazenamento, requests, transferência de dados e taxas de recuperação. Ao planejar, verifique o modelo de preços da AWS para cada classe.

# ❄️ AWS Snowball

## 📌 O que é o AWS Snowball?

O **AWS Snowball** é um serviço da AWS que permite **transferir grandes volumes de dados** entre sua infraestrutura local e a nuvem de forma rápida e segura, usando **dispositivos físicos transportáveis**.  
Ele é indicado quando a **transferência pela internet não é prática** devido ao tamanho dos dados ou limitações de banda.

---

## 🔑 Principais características

- **Dispositivo físico seguro** (criptografia AES 256-bit).  
- **Capacidade**: de 50 TB a 80 TB por dispositivo (dependendo do modelo).  
- **Alta durabilidade**: resistente a falhas físicas e ambientais.  
- **Integração com AWS S3 e Glacier**: dados carregados no Snowball podem ser transferidos diretamente para buckets na nuvem.  
- **Reduz tempo e custo** para transferências massivas de dados em comparação com internet pública.

---

## 🔧 Como funciona (alto nível)

1. Solicita o dispositivo Snowball pelo **AWS Management Console**.  
2. AWS envia o dispositivo físico para sua empresa.  
3. Você conecta o dispositivo à sua rede local e copia os dados desejados.  
4. Depois de copiar, o dispositivo é devolvido para a AWS.  
5. AWS transfere os dados para o **S3, Glacier ou buckets configurados**.

---

## 🧩 Modelos disponíveis

- **Snowball Edge Storage Optimized** → foco em transferência de dados e armazenamento em S3.  
- **Snowball Edge Compute Optimized** → inclui capacidade de processamento com **EC2 e Lambda** embarcados.  
- **Snowcone** → versão menor, portátil, até 8 TB, ideal para locais remotos ou IoT.  

---

## ✅ Casos de uso

- Migração de **grandes volumes de dados** para AWS.  
- Backup e arquivamento offline de **dados críticos**.  
- Coleta de dados em locais remotos sem boa conectividade.  
- Processamento de dados local com **computação embutida** (Snowball Edge Compute).  

---

## ⚠️ Observações

- Dispositivos são **transportáveis**, mas a segurança física e criptografia garantem confidencialidade.  
- Útil quando transferir dados pela internet seria **lento ou caro**.  
- Pode ser usado em conjunto com **AWS DataSync** para cargas menores ou contínuas.


<img width="1069" height="464" alt="image" src="https://github.com/user-attachments/assets/9442844b-b835-4ec6-b28e-aa8f83ea05cb" />

# 🛡️ Amazon Storage Gateway

## 📌 O que é o Amazon Storage Gateway?

O **Amazon Storage Gateway** é um serviço que conecta **ambientes on-premises** (locais) à **nuvem AWS**, permitindo que empresas utilizem o **armazenamento na nuvem** como extensão de seus sistemas locais.  
Ele combina **armazenamento local e em nuvem** para oferecer desempenho rápido com durabilidade e escalabilidade da AWS.

---

## 🔑 Principais características

- **Integração híbrida:** conecta servidores locais à AWS.  
- **Copia dados para S3, Glacier ou Glacier Deep Archive** automaticamente.  
- **Baixa latência local:** mantém uma cópia local em cache para acesso rápido.  
- **Segurança:** criptografia de dados em trânsito e em repouso.  
- **Compatível com aplicações existentes:** funciona como **NAS (file), iSCSI (block)** ou **VTL (virtual tape)**.

---

## 🧩 Tipos de Storage Gateway

1. **File Gateway**  
   - Exposição via **NFS ou SMB**.  
   - Armazena arquivos diretamente em **S3**.  
   - Ideal para compartilhamento de arquivos e backup baseado em arquivos.

2. **Volume Gateway**  
   - Armazenamento em blocos via **iSCSI**.  
   - Pode ser **Cached Volume** (dados principais na nuvem, cache local) ou **Stored Volume** (dados principais locais, backup na nuvem).  
   - Ideal para backups e recuperação de desastres.

3. **Tape Gateway**  
   - Simula uma **biblioteca de fitas virtual (VTL)**.  
   - Integra com softwares de backup existentes.  
   - Armazena os backups em **S3 ou Glacier**.  

---

## ✅ Casos de uso

- Extensão de armazenamento local para a nuvem.  
- Backup e recuperação de desastres sem precisar migrar todo o data center.  
- Arquivamento de dados históricos em S3 ou Glacier.  
- Integração com sistemas legados que exigem armazenamento em bloco ou fita.  

---

## ⚠️ Observações

- Requer **conectividade com a AWS**.  
- Permite **reduzir custos de armazenamento on-premises** e aumentar durabilidade.  
- Pode ser combinado com **AWS Snowball** para transferências iniciais de grandes volumes de dados.

# 🗄️ Amazon RDS (Relational Database Service)

## 📌 O que é o Amazon RDS?

O **Amazon Relational Database Service (RDS)** é um serviço gerenciado da AWS que facilita a **criação, operação e escalabilidade** de bancos de dados relacionais na nuvem.  
Ele cuida de tarefas administrativas como **provisionamento, backup, patching, replicação e monitoramento**, permitindo que você foque apenas no uso do banco.

---

## 🔑 Principais características

- **Bancos suportados:**  
  - Amazon Aurora  
  - MySQL  
  - MariaDB  
  - PostgreSQL  
  - Oracle  
  - Microsoft SQL Server  

- **Alta disponibilidade:**  
  - Opção de **Multi-AZ** (replicação automática entre zonas).  
  - Failover automático em caso de falha.

- **Escalabilidade:**  
  - Ajuste de **CPU, memória e armazenamento** sob demanda.  
  - Suporte a **Read Replicas** para leitura escalável.

- **Segurança:**  
  - Criptografia em repouso (KMS) e em trânsito (SSL/TLS).  
  - Integração com **IAM** para controle de acesso.  

- **Backups automáticos:**  
  - Snapshots manuais e automáticos.  
  - Recuperação point-in-time.  

---

## ✅ Casos de uso

- Aplicações **web e móveis** que precisam de banco relacional.  
- Sistemas de ERP, CRM e aplicações corporativas.  
- Análises que requerem dados transacionais consistentes.  
- Substituição de bancos locais por uma solução **gerenciada e escalável**.  

---

## 🧩 Benefícios principais

- Menor esforço operacional (AWS gerencia updates, patches, backups).  
- Confiabilidade com **99,95% de SLA**.  
- Fácil integração com outros serviços da AWS (EC2, Lambda, Elastic Beanstalk).  
- Redução de custos com administração de banco.  

---

## ⚠️ Observações

- Apesar de gerenciado, você **não tem acesso root ao sistema operacional** subjacente.  
- Cobrança baseada em **instância + armazenamento + I/O**.  
- Para workloads mais críticos, usar **Multi-AZ** ou **Aurora** é recomendado.

# 🤔 RDS vs Banco em EC2

Quando você precisa de um banco de dados na AWS, existem duas opções principais:  
1. **Gerenciar manualmente em uma instância EC2**.  
2. **Usar um serviço gerenciado como o Amazon RDS**.  

---

## 🔧 Banco de dados em EC2 (autogerenciado)

- Você instala e configura o banco de dados (MySQL, PostgreSQL, etc.) dentro de uma **instância EC2**.  
- Você é responsável por **todas as tarefas administrativas**:  
  - Patching e atualizações de versão.  
  - Backups.  
  - Alta disponibilidade e replicação.  
  - Monitoramento.  
  - Tuning de performance.  
- Maior controle, mas também **maior complexidade e risco de erros**.  
- Útil apenas quando você precisa de **customizações profundas** que o RDS não suporta.

---

## 🗄️ Banco de dados com Amazon RDS (gerenciado)

- AWS gerencia quase todo o **trabalho operacional**.  
- Benefícios:  
  - **Backups automáticos**.  
  - **Replicação Multi-AZ** e **Read Replicas** com poucos cliques.  
  - **Patching automático** do banco.  
  - Integração nativa com **IAM, CloudWatch, KMS**.  
  - **Failover automático** em caso de falha.  
- Permite escalar verticalmente (mais CPU/RAM) ou horizontalmente (réplicas de leitura).  
- Você foca no **uso do banco e nos dados**, não na infraestrutura.

---

## ⚖️ Comparação

| Aspecto                  | Banco em EC2 (autogerenciado)            | Amazon RDS (gerenciado)                  |
|---------------------------|------------------------------------------|------------------------------------------|
| Instalação e setup        | Manual (você cuida de tudo)             | Automática (poucos cliques)              |
| Patching/updates          | Você gerencia                           | AWS gerencia                             |
| Backups                   | Você implementa                         | Automáticos e configuráveis              |
| Alta disponibilidade      | Você configura (replicação/manual)      | Multi-AZ integrado + failover automático |
| Segurança                 | Você configura tudo                     | Criptografia, IAM e segurança integrados |
| Escalabilidade            | Complexa, exige scripts/configuração     | Simples (troca de tamanho, read replicas)|
| Custo de tempo operacional| Alto                                    | Baixo                                    |
| Flexibilidade             | Máxima (customizações no SO e DB)       | Limitada (restrições do serviço)         |

---

## ✅ Por que usar RDS em vez de EC2?

- Reduz **carga operacional** (você não perde tempo com patch, backup, replicação).  
- Menor risco de erro humano em tarefas administrativas.  
- Melhor **resiliência e disponibilidade** (Multi-AZ, failover automático).  
- Escalabilidade muito mais fácil.  
- Integração nativa com o ecossistema AWS.  

👉 Use **EC2 com banco autogerenciado** somente quando:  
- Precisa de **customizações no SO ou no banco** que o RDS não suporta.  
- Precisa de um **sistema legado** que não roda em RDS.  
- Tem requisitos específicos de configuração avançada.

Na maioria dos casos, **Amazon RDS é a escolha ideal**: menos esforço, mais segurança, mais confiabilidade.

# ⚡ Amazon Aurora

## 📌 O que é o Amazon Aurora?

O **Amazon Aurora** é um banco de dados relacional compatível com **MySQL** e **PostgreSQL**, totalmente gerenciado pela AWS, projetado para ser **mais rápido, mais escalável e mais resiliente** do que bancos tradicionais.  
Ele faz parte do **Amazon RDS**, mas foi desenvolvido pela AWS para **tirar o máximo proveito da nuvem**.

---

## 🔑 Principais características

- **Compatibilidade:** funciona com aplicações MySQL e PostgreSQL já existentes.  
- **Performance:** até **5x mais rápido que MySQL** e **3x mais rápido que PostgreSQL** em hardware equivalente.  
- **Escalabilidade automática:** cresce de 10 GB até **128 TB por banco** sem precisar de intervenção.  
- **Alta disponibilidade:**  
  - Armazena **6 cópias de dados em 3 AZs diferentes**.  
  - Failover automático em segundos.  
- **Replicação:** até 15 réplicas de leitura com baixa latência.  
- **Integração com outros serviços AWS:** IAM, KMS, CloudWatch, Lambda, etc.  
- **Segurança:** criptografia em trânsito (SSL/TLS) e em repouso (KMS).  

---

## ✅ Casos de uso

- Aplicações **mission-critical** que precisam de **alta disponibilidade** e **resiliência**.  
- Sistemas financeiros, e-commerce e apps com **grande volume de transações**.  
- Ambientes onde é necessário **crescimento automático do banco** sem downtime.  
- Substituição de bancos MySQL/PostgreSQL locais com ganho de performance.  

---

## ⚖️ Comparação rápida: Aurora vs RDS

| Aspecto                   | Amazon RDS (MySQL/Postgres)            | Amazon Aurora                          |
|----------------------------|----------------------------------------|----------------------------------------|
| Performance                | Boa (gerenciada pela AWS)              | Excelente (3x–5x mais rápido)          |
| Escalabilidade             | Manual (alterar instância ou storage)  | Automática até 128 TB                  |
| Replicação                 | Algumas réplicas (menos eficientes)    | Até 15 réplicas com baixa latência     |
| Alta disponibilidade       | Multi-AZ com failover                  | 6 cópias de dados em 3 AZs             |
| Tempo de failover          | Minutos                                | Segundos                               |
| Custo                      | Mais barato                            | Mais caro (compensa pela performance)  |
| Compatibilidade            | Engines variados (MySQL, MariaDB etc.) | Apenas MySQL e PostgreSQL              |

---

## ⚠️ Observações

- O custo do Aurora é **maior que RDS "puro"**, mas pode sair mais barato se considerar a **eficiência/performance**.  
- Se você já usa MySQL/PostgreSQL, a migração é simples.  
- Aurora é pensado para quem precisa de **escalabilidade sem dor de cabeça** e alta confiabilidade.  

# 🌐 Amazon Aurora Serverless

## 📌 O que é o Aurora Serverless?

O **Aurora Serverless** é uma versão **sob demanda** do Amazon Aurora que **escala automaticamente a capacidade do banco de dados** conforme a carga da aplicação.  
Você **não precisa escolher o tamanho da instância**: o banco cresce ou diminui sozinho, e você só paga pelos recursos que realmente usar.

---

## 🔑 Principais características

- **Escalabilidade automática:** ajusta a capacidade de processamento e memória conforme a demanda.  
- **Sob demanda:** paga apenas pelos segundos de uso (não precisa de instância fixa 24/7).  
- **Alta disponibilidade:** integrado ao Aurora, com dados replicados em múltiplas zonas (Multi-AZ).  
- **Compatibilidade:** funciona com MySQL e PostgreSQL.  
- **Pause/Resume automático:** o banco pode "hibernar" quando não há uso, reduzindo custos.  

---

## ✅ Casos de uso

- **Aplicações intermitentes ou imprevisíveis:** apps que têm períodos de uso intenso e depois ficam ociosos.  
- **Ambientes de desenvolvimento e teste:** só consome recursos quando realmente usado.  
- **Sistemas sazonais:** como e-commerce em época de promoções ou aplicações com picos em horários específicos.  

---

## ⚖️ Aurora Provisioned vs Aurora Serverless

| Aspecto                  | Aurora Provisioned                     | Aurora Serverless                      |
|---------------------------|----------------------------------------|----------------------------------------|
| Capacidade                | Fixa (você escolhe o tamanho da instância) | Dinâmica (escala automaticamente)      |
| Custos                    | Pago pela instância, mesmo ociosa       | Pago por uso em segundos               |
| Performance               | Constante, previsível                  | Variável (depende do ajuste automático)|
| Disponibilidade           | Alta (Multi-AZ, réplicas, failover)    | Alta (com failover automático)         |
| Casos de uso              | Workloads contínuos e estáveis         | Workloads variáveis ou imprevisíveis   |

---

## ⚠️ Observações

- **Latência no scale-up/down:** pode levar alguns segundos para ajustar a capacidade.  
- Nem todos os recursos do Aurora estão disponíveis no **Serverless**.  
- Ideal para workloads variáveis; para cargas sempre constantes, o **Aurora provisionado** pode sair mais barato.  

# ⚡ Amazon ElastiCache

## 📌 O que é o Amazon ElastiCache?

O **Amazon ElastiCache** é um serviço de **cache em memória totalmente gerenciado** da AWS, compatível com os mecanismos **Redis** e **Memcached**.  
Ele é usado para melhorar a **performance e escalabilidade** de aplicações, armazenando dados frequentemente acessados na memória, o que é muito mais rápido do que buscar em um banco de dados tradicional.

---

## 🔑 Principais características

- **Alto desempenho:** baixa latência, resposta em microssegundos.  
- **Compatibilidade:** suporta **Redis** (mais avançado, com persistência e replicação) e **Memcached** (mais simples e distribuído).  
- **Gerenciado pela AWS:** sem necessidade de administrar servidores manualmente.  
- **Escalabilidade:** fácil de aumentar ou reduzir clusters conforme a demanda.  
- **Alta disponibilidade:** suporte a **Multi-AZ** com failover automático.  
- **Segurança:** integração com **IAM, VPC, KMS e TLS**.  

---

## ✅ Casos de uso

- **Cache de consultas de banco de dados:** reduz a carga em bancos relacionais como RDS ou Aurora.  
- **Armazenamento de sessões de usuários:** muito usado em aplicações web escaláveis.  
- **Ranking em tempo real e contadores:** jogos online, analytics e e-commerce.  
- **Filas, Pub/Sub e mensagens em tempo real** (no caso do Redis).  
- **Caching de páginas e conteúdo dinâmico** para reduzir latência em aplicações.  

---

## ⚖️ Redis vs Memcached

| Aspecto            | Redis                                     | Memcached                         |
|---------------------|-------------------------------------------|-----------------------------------|
| Persistência        | Suporta persistência em disco             | Não suporta (somente em memória)  |
| Estruturas de dados | Avançadas (listas, sets, hashes, streams) | Apenas chave/valor simples        |
| Replicação          | Suporta replicação e failover automático  | Não tem replicação nativa         |
| Escalabilidade      | Boa, mas mais complexa                    | Muito simples e horizontal        |
| Casos de uso        | Sessões, filas, contadores, cache robusto | Cache simples e distribuído       |

---

## ⚠️ Observações

- Não substitui um banco de dados relacional ou NoSQL — é um **complemento** para performance.  
- Ideal para workloads onde a **latência é crítica**.  
- Escolha **Redis** quando precisar de **mais funcionalidades e resiliência**; use **Memcached** se precisar apenas de cache básico e distribuído.  

# ⚡ Amazon DynamoDB

## 📌 O que é o DynamoDB?

O **Amazon DynamoDB** é um **banco de dados NoSQL totalmente gerenciado**, com alta performance e baixa latência.  
Ele é **serverless**, escalável automaticamente e ideal para aplicações que precisam lidar com **grande volume de dados** e acessos imprevisíveis.

---

## 🔑 Principais características

- **Modelo NoSQL:** baseado em **tabelas, itens e atributos** (não relacional).  
- **Escalabilidade automática:** ajusta throughput conforme a demanda sem downtime.  
- **Performance previsível:** latência em **milissegundos de um dígito**, mesmo com alto volume.  
- **Serverless:** não é necessário provisionar instâncias.  
- **Alta disponibilidade e durabilidade:** replicado automaticamente em **3 AZs**.  
- **Segurança:** integração com **IAM, KMS e VPC**, criptografia de dados em repouso e trânsito.  
- **Streams:** permite capturar alterações nos dados para **eventos ou replicação**.

---

## ✅ Casos de uso

- **Aplicações web e móveis de grande escala** (ex.: apps de rede social, e-commerce).  
- **Sistemas que exigem leitura/escrita rápida e imprevisível**.  
- **Gaming:** rankings, contadores, inventário de jogadores.  
- **IoT:** armazenamento de grandes volumes de dados de dispositivos.  
- **Aplicações serverless** com Lambda para processar eventos do DynamoDB Streams.

---

## ⚖️ DynamoDB vs RDS / Aurora

| Aspecto                  | RDS / Aurora                           | DynamoDB                       |
|---------------------------|----------------------------------------|--------------------------------|
| Modelo de dados           | Relacional                              | NoSQL (chave-valor / documento)|
| Escalabilidade            | Vertical ou Read Replicas               | Horizontal automática (serverless)|
| Latência                  | Milissegundos a dezenas de ms           | Milissegundos de 1 dígito     |
| Consistência              | Forte (ACID)                            | Configurável: eventual ou forte|
| Serverless                | Não (instâncias provisionadas)

# 📊 Amazon Redshift

## 📌 O que é o Amazon Redshift?

O **Amazon Redshift** é um serviço de **data warehouse totalmente gerenciado** pela AWS.  
Ele é otimizado para **consultas analíticas em grandes volumes de dados** (em petabytes), permitindo que empresas façam **Business Intelligence (BI)**, relatórios e análises complexas com alta performance.

---

## 🔑 Principais características

- **Armazenamento em colunas (columnar):** otimizado para consultas analíticas, não para transações.  
- **Processamento massivamente paralelo (MPP):** distribui a execução de queries entre vários nós.  
- **Integração nativa com BI e ML:** suporta Amazon QuickSight, Tableau, Power BI, SageMaker.  
- **Escalabilidade:** pode crescer para **petabytes de dados**.  
- **Backup e snapshots automáticos** para o Amazon S3.  
- **Segurança:** criptografia em trânsito e repouso (KMS, HSM), VPC e integração com IAM.  
- **Redshift Spectrum:** permite consultar dados diretamente no **Amazon S3** sem precisar carregar no Redshift.  

---

## ✅ Casos de uso

- **Business Intelligence (BI)** e relatórios corporativos.  
- **Análises em tempo quase real** de grandes volumes de dados.  
- **Integração de múltiplas fontes de dados** (bancos, logs, IoT, apps).  
- **Data Lakes:** análise de dados armazenados no **Amazon S3**.  

---

## ⚖️ Comparação Redshift vs RDS / DynamoDB

| Aspecto                  | RDS / Aurora                           | DynamoDB                           | Redshift                           |
|---------------------------|----------------------------------------|------------------------------------|------------------------------------|
| Modelo de dados           | Relacional (OLTP)                      | NoSQL (chave-valor/documento)       | Data Warehouse (OLAP)               |
| Otimização                | Transações (muitas escritas/leitura leve)| Leitura/escrita ultrarrápida e escala| Consultas analíticas complexas      |
| Volume de dados           | De GB a alguns TBs                     | Escala massiva horizontal           | Até petabytes                       |
| Latência                  | Baixa, para apps transacionais         | Milissegundos                       | Segundos a minutos (para queries)   |
| Escalabilidade            | Vertical + réplicas                    | Serverless, horizontal automática   | Clusters elásticos                  |
| Casos de uso              | ERP, e-commerce, sistemas transacionais| Apps web/mobile, IoT, gaming        | BI, relatórios, big data analytics  |

---

## ⚠️ Observações

- **Redshift não é para transações** (inserções rápidas e frequentes). Ele é feito para **consultas analíticas pesadas**.  
- Trabalha muito bem junto com **S3 (Data Lake)** e **Glue (ETL)**.  
- Se o objetivo é **processamento transacional (OLTP)** → use **RDS/Aurora/DynamoDB**.  
- Se o objetivo é **análise e relatórios (OLAP)** → use **Redshift**.  

# 🔥 Amazon EMR (Elastic MapReduce)

## 📌 O que é o Amazon EMR?

O **Amazon EMR (Elastic MapReduce)** é um serviço gerenciado da AWS para **processamento de grandes volumes de dados** usando frameworks como **Apache Hadoop, Spark, Hive, HBase, Flink e Presto**.  
Ele permite analisar, transformar e processar dados em **clusters escaláveis** de forma rápida e com custo reduzido.

---

## 🔑 Principais características

- **Big Data como serviço:** executa workloads de análise em escala massiva.  
- **Frameworks suportados:** Hadoop, Spark, Hive, HBase, Presto, Flink, entre outros.  
- **Integração com S3:** pode usar o **Amazon S3 como data lake**.  
- **Escalabilidade elástica:** ajusta automaticamente o número de nós do cluster.  
- **Custo otimizado:** pode usar **Spot Instances** para reduzir custos.  
- **Segurança:** integração com **IAM, KMS, VPC, Kerberos** e criptografia.  
- **Gerenciamento facilitado:** a AWS provisiona, configura e gerencia os clusters.  

---

## ✅ Casos de uso

- **Processamento de grandes volumes de dados (Big Data).**  
- **ETL (Extract, Transform, Load):** preparação de dados para análise.  
- **Machine Learning:** treinar modelos com Spark MLlib ou TensorFlow em escala.  
- **Data Warehousing:** análises em conjunto com **Amazon Redshift**.  
- **Data Lakes:** processamento direto em dados armazenados no S3.  

---

## ⚖️ Comparação EMR vs Redshift vs Glue

| Aspecto                  | EMR                                   | Redshift                              | Glue                             |
|---------------------------|---------------------------------------|---------------------------------------|----------------------------------|
| Tipo de serviço           | Big Data (Hadoop, Spark, etc.)        | Data Warehouse (OLAP)                 | ETL (serverless)                 |
| Modelo de dados           | Flexível (estruturado e não estruturado)| Estruturado/semiestruturado (SQL)     | Dados em S3, Catálogo de dados   |
| Escalabilidade            | Cluster escalável elástico            | Clusters de Data Warehouse            | Totalmente serverless            |
| Casos de uso              | Processamento massivo, ML, ETL        | BI, relatórios, análise de dados       | ETL, preparação e catalogação    |
| Custos                    | Paga pelos clusters (EC2 + storage)   | Paga pelos nós/armazenamento          | Paga por execução (sob demanda)  |

---

## ⚠️ Observações

- O **EMR é ideal para workloads complexos de Big Data**, mas exige mais configuração do que Redshift ou Glue.  
- Normalmente, é usado em conjunto com **S3 (Data Lake)** e **Athena/Redshift** para análises.  
- Permite controle detalhado sobre o cluster, o que dá flexibilidade, mas também responsabilidade maior de configuração.

# 🔎 Amazon Athena

## 📌 O que é o Amazon Athena?

O **Amazon Athena** é um serviço **serverless de consulta interativa** que permite executar **queries SQL diretamente em dados armazenados no Amazon S3**.  
Não é necessário provisionar servidores ou clusters: você paga apenas pelas consultas realizadas.

Dica: Analíse um S3 usando serverless SQL. USE ATHENA

---

## 🔑 Principais características

- **Serverless:** sem necessidade de gerenciar infraestrutura.  
- **Consulta em S3:** acessa dados diretamente em buckets do Amazon S3.  
- **Compatibilidade:** suporta **ANSI SQL** e integra com **AWS Glue Data Catalog**.  
- **Formatos de dados:** funciona com **CSV, JSON, Parquet, ORC, Avro**, entre outros.  
- **Integrações:** com Amazon QuickSight, Redshift, Glue e outros serviços de análise.  
- **Custo otimizado:** você paga somente pelos dados lidos pela query.  

---

## ✅ Casos de uso

- **Exploração e análise de dados em data lakes (S3).**  
- **Ad-hoc queries:** consultas rápidas em grandes volumes de dados.  
- **Auditoria e logs:** análise de CloudTrail, ELB, VPC Flow Logs, etc.  
- **ETL simplificado:** transformar dados sem precisar rodar clusters (com integração Glue).  
- **Integração BI:** conectar com ferramentas como **Amazon QuickSight, Tableau e Power BI**.  

---

## ⚖️ Comparação Athena vs Redshift vs EMR

| Aspecto                  | Athena                                | Redshift                              | EMR                                      |
|---------------------------|---------------------------------------|---------------------------------------|------------------------------------------|
| Tipo de serviço           | Consulta SQL serverless em S3         | Data Warehouse (OLAP)                 | Big Data (Hadoop, Spark, Hive, etc.)      |
| Infraestrutura            | Nenhuma (serverless)                  | Cluster provisionado pela AWS         | Clusters gerenciados pelo usuário         |
| Casos de uso              | Consultas rápidas em data lakes       | BI, relatórios complexos, alta escala | Processamento massivo, ETL, ML, pipelines |
| Custos                    | Pago por query (dados lidos em S3)    | Pago por nó/armazenamento             | Pago por cluster EC2 + tempo de execução  |

---

## ⚠️ Observações

- **Ideal para consultas rápidas** em grandes datasets sem precisar de cluster.  
- Se o workload é **análise repetitiva em petabytes de dados estruturados**, Redshift pode ser mais eficiente.  
- Para **ETL complexo ou machine learning**, o EMR é mais adequado.  
- Athena brilha em arquiteturas de **Data Lake** junto com **S3 e Glue**.  

# 📊 Amazon QuickSight

## 📌 O que é o Amazon QuickSight?

O **Amazon QuickSight** é o serviço de **Business Intelligence (BI) da AWS**, totalmente gerenciado e **serverless**, que permite criar **dashboards interativos, relatórios e visualizações de dados**.  
Ele conecta-se a diversas fontes (como **S3, Athena, Redshift, RDS, DynamoDB** e até bancos externos) para transformar dados em **insights visuais**.

---

## 🔑 Principais características

- **Serverless:** não há necessidade de provisionar infraestrutura.  
- **Visualizações interativas:** gráficos, tabelas dinâmicas, mapas e dashboards responsivos.  
- **Integrações nativas:** conecta facilmente com **Athena, Redshift, RDS, S3** e serviços externos (via JDBC/ODBC).  
- **Machine Learning embutido:** insights automáticos, detecção de anomalias, previsão de tendências.  
- **Acesso multiplataforma:** disponível em **navegador e aplicativo mobile**.  
- **Escalabilidade:** pode atender desde um time pequeno até milhares de usuários.  
- **Custo otimizado:** baseado em usuários ativos ou capacidade de uso.  

---

## ✅ Casos de uso

- **Dashboards executivos** para monitorar KPIs.  
- **Relatórios de vendas e marketing.**  
- **Monitoramento em tempo quase real** (quando conectado a fontes como Redshift ou Kinesis).  
- **Visualização de dados de Data Lakes (S3 + Athena).**  
- **Análise financeira, de operações e suporte a decisões estratégicas.**  

---

## ⚖️ Comparação QuickSight vs outras ferramentas BI

| Aspecto                  | QuickSight                           | Tableau / Power BI                    |
|---------------------------|--------------------------------------|---------------------------------------|
| Modelo de uso             | 100% gerenciado e serverless (AWS)   | Precisa instalar/gerenciar servidores ou SaaS |
| Integração AWS            | Nativa (Athena, Redshift, S3, RDS)  | Integração via conectores adicionais  |
| Machine Learning          | Integrado (previsões, anomalias)    | Necessário configurar separadamente   |
| Custos                    | Baseado em uso (pay-per-session)    | Licenças fixas por usuário            |
| Escalabilidade            | Automática                          | Limitada à capacidade do servidor     |

---

## ⚠️ Observações

- Ideal quando os **dados já estão na AWS** (S3, Athena, Redshift, RDS).  
- Fácil de usar para **dashboards rápidos e análises visuais** sem precisar de muita configuração.  
- Pode substituir ou complementar ferramentas como Tableau ou Power BI, especialmente em **ambientes 100% AWS**.  


# 📚 Amazon DocumentDB

## 📌 O que é o Amazon DocumentDB?

O **Amazon DocumentDB** é um banco de dados **NoSQL gerenciado pela AWS** compatível com o **MongoDB**.  
Ele foi projetado para armazenar, consultar e indexar **documentos JSON** de forma escalável, segura e altamente disponível, sem a complexidade de gerenciar servidores de banco.

---

## 🔑 Principais características

- **Compatibilidade com MongoDB:** drivers e APIs do MongoDB funcionam no DocumentDB.  
- **Escalabilidade:** aumenta leitura e armazenamento sob demanda.  
- **Alta disponibilidade:** arquitetura com replicação automática em **3 zonas de disponibilidade (AZs)**.  
- **Backups automáticos:** snapshots contínuos e recuperação point-in-time.  
- **Segurança:** criptografia em trânsito (TLS) e em repouso (KMS), integração com **IAM e VPC**.  
- **Gerenciamento automático:** a AWS cuida de patching, failover e monitoramento.  

---

## ✅ Casos de uso

- **Aplicações web e móveis** que armazenam dados semiestruturados (JSON).  
- **Catálogos de produtos** em e-commerce.  
- **Perfis de usuários** em redes sociais ou apps.  
- **Sistemas de gerenciamento de conteúdo (CMS).**  
- **Logs e eventos** para análises rápidas.  

---

# 🕸️ Amazon Neptune

## 📌 O que é o Amazon Neptune?

O **Amazon Neptune** é um **banco de dados de grafos totalmente gerenciado** pela AWS, projetado para trabalhar com relacionamentos altamente conectados entre dados.  
Ele é otimizado para **consultas de grafos complexos** e suporta os principais **motores e linguagens de grafos**:  
- **Property Graph (Apache TinkerPop Gremlin)**  
- **RDF (Resource Description Framework) com SPARQL**

---

## 🔑 Principais características

- **Banco orientado a grafos:** ideal para armazenar e consultar relações complexas.  
- **Compatibilidade:** suporta **Gremlin** e **SPARQL**.  
- **Alto desempenho:** consultas em grafos de bilhões de relações com latência baixa.  
- **Alta disponibilidade:** replicação em **3 zonas de disponibilidade (AZs)**.  
- **Backups automáticos:** snapshots e recuperação point-in-time.  
- **Segurança:** criptografia em repouso (KMS) e em trânsito (TLS), integração com **IAM e VPC**.  
- **Gerenciamento automático:** a AWS cuida de patching, failover e monitoramento.  

---

## ✅ Casos de uso

- **Redes sociais:** encontrar conexões entre pessoas e interesses.  
- **Sistemas de recomendação:** sugerir produtos, filmes ou músicas com base em relacionamentos.  
- **Detecção de fraudes:** identificar padrões suspeitos em redes de transações.  
- **Gerenciamento de conhecimento:** modelar relações complexas em bases de dados semânticos.  
- **IoT:** modelar interações entre dispositivos conectados.  

---
# ⏱️ Amazon Timestream

## 📌 O que é o Amazon Timestream?

O **Amazon Timestream** é um banco de dados **serverless, totalmente gerenciado e otimizado para séries temporais**.  
Ele foi projetado para lidar com **dados que variam ao longo do tempo** (como métricas, logs, eventos e dados de IoT), permitindo ingestão, armazenamento e consulta em grande escala com baixa latência.

---

## 🔑 Principais características

- **Especializado em séries temporais:** otimizado para dados que possuem carimbo de data/hora.  
- **Serverless:** escalabilidade automática, sem necessidade de gerenciar servidores.  
- **Armazenamento em camadas:** separa automaticamente dados **recentes (quentes)** e **históricos (frios)** para otimizar custo/performance.  
- **Consultas SQL:** usa linguagem SQL estendida para funções de séries temporais.  
- **Alto desempenho:** ingestão de milhões de eventos por segundo.  
- **Integrações nativas:** funciona com **IoT Core, Kinesis, CloudWatch, Lambda e Grafana**.  
- **Segurança:** criptografia com KMS e controle de acesso via IAM.  

---

## ✅ Casos de uso

- **IoT:** monitoramento de dispositivos conectados em tempo real.  
- **DevOps / Observabilidade:** análise de métricas de aplicações, logs de sistemas e dados de sensores.  
- **Monitoramento industrial:** leitura de máquinas, sensores e equipamentos.  
- **Aplicações financeiras:** séries temporais de preços e transações.  
- **Análises em tempo real:** dashboards de performance e alertas automáticos.  

---

# 🔗 Amazon Managed Blockchain

## 📌 O que é o Amazon Managed Blockchain?

O **Amazon Managed Blockchain** é um serviço **totalmente gerenciado da AWS** que permite **criar e gerenciar redes de blockchain escaláveis** usando frameworks populares como **Hyperledger Fabric** e **Ethereum** (em versões anteriores).  

Ele elimina a complexidade de configurar manualmente infraestrutura de blockchain, permitindo que empresas construam **aplicações descentralizadas (dApps)**, **redes privadas entre parceiros** e sistemas de **transações imutáveis**.

---

## 🔑 Principais características

- **Suporte a frameworks de blockchain:** Hyperledger Fabric (Ethereum já foi suportado, mas não está mais disponível em versões recentes).  
- **Gerenciamento automatizado:** provisionamento de nós, atualização de software, monitoramento e escalabilidade.  
- **Alta disponibilidade:** infraestrutura tolerante a falhas em múltiplas zonas da AWS.  
- **Integração com AWS:** IAM para controle de acesso, CloudWatch para monitoramento, S3 para armazenar dados off-chain.  
- **Segurança:** identidade de participantes gerenciada, criptografia em repouso e em trânsito.  
- **Escalabilidade:** fácil adição ou remoção de membros e nós validadores.  

---

## ✅ Casos de uso

- **Supply Chain:** rastreamento de produtos e insumos de ponta a ponta.  
- **Serviços financeiros:** liquidação de transações, auditoria e compliance.  
- **Governança corporativa:** votação eletrônica segura e transparente.  
- **Saúde:** compartilhamento seguro de registros médicos entre organizações.  
- **Identidade digital:** sistemas de autenticação descentralizados.  

---

# 🧩 AWS Glue

## 📌 O que é o AWS Glue?

O **AWS Glue** é um serviço **serverless de ETL (Extract, Transform, Load)** totalmente gerenciado, usado para **preparar e transformar dados** para análise.  
Ele facilita a integração de dados de múltiplas fontes e sua disponibilização em **data lakes, data warehouses e sistemas de análise**.  

---

## 🔑 Principais características

- **Serverless:** não há necessidade de gerenciar servidores ou clusters.  
- **ETL simplificado:** cria, transforma e carrega dados automaticamente.  
- **Glue Data Catalog:** catálogo centralizado para metadados de todos os dados armazenados na AWS.  
- **Integração nativa:** funciona diretamente com **S3, Redshift, RDS, DynamoDB, Athena, Lake Formation** e outros.  
- **Transformações de dados:** suporta **PySpark**, **Scala** e **Python** para processar grandes volumes de dados.  
- **Glue Studio:** interface visual para criar pipelines ETL sem precisar de muito código.  
- **Glue DataBrew:** interface para preparação de dados sem código (no-code).  
- **Glue Crawlers:** detecta automaticamente esquemas e popula o Data Catalog.  

---

## ✅ Casos de uso

- **Data Lakes:** organizar dados no **Amazon S3** em formato otimizado (Parquet, ORC, Avro).  
- **Integração de dados:** unificação de dados espalhados em **bancos relacionais, DynamoDB, S3, logs e APIs**.  
- **Data Warehousing:** preparar dados antes de enviá-los para **Amazon Redshift**.  
- **Análise ad hoc:** combinar dados de múltiplas fontes para consulta via **Athena ou QuickSight**.  
- **Machine Learning:** preparação de dados para modelos de ML no **SageMaker**.  

---

# 🔄 AWS Database Migration Service (DMS)

## 📌 O que é o AWS DMS?

O **AWS DMS (Database Migration Service)** é um serviço **totalmente gerenciado** que facilita a **migração de bancos de dados para a AWS** de forma **rápida, segura e com mínimo downtime**.  
Ele suporta tanto **migração homogênea** (mesmo mecanismo de banco, ex: Oracle → Oracle) quanto **heterogênea** (diferentes mecanismos, ex: Oracle → Aurora, MySQL → PostgreSQL).  

---

## 🔑 Principais características

- **Migração com baixo downtime:** mantém o banco de origem e destino sincronizados até a troca final.  
- **Flexível:** suporta bancos relacionais, NoSQL e data warehouses.  
- **Homogêneo e heterogêneo:** funciona entre bancos iguais ou diferentes.  
- **Escalável e gerenciado:** AWS gerencia patching, replicação e monitoramento.  
- **Transformação de esquema:** com a ajuda do **AWS Schema Conversion Tool (SCT)**.  
- **Segurança:** criptografia em trânsito (TLS) e em repouso (KMS).  
- **Replicação contínua:** pode ser usado para sincronizar dados em tempo quase real.  

---

## ✅ Casos de uso

- **Migração para AWS:** mover bancos locais ou em outras clouds para **RDS, Aurora, DynamoDB ou Redshift**.  
- **Atualização de bancos:** migração de versões antigas para novas.  
- **Conversão de mecanismos:** ex: Oracle → Aurora PostgreSQL, SQL Server → MySQL.  
- **Replicação contínua:** manter dois bancos sincronizados durante uma transição ou integração.  
- **Híbrido:** replicação de dados entre **on-premises** e **AWS** para testes e análises.  

---

## ⚖️ Comparação DMS vs SCT vs Ferramentas manuais

| Aspecto                  | AWS DMS                               | AWS SCT (Schema Conversion Tool)    | Migração Manual                    |
|---------------------------|---------------------------------------|-------------------------------------|-------------------------------------|
| Foco                      | Migrar dados                          | Converter esquema e objetos          | Totalmente customizável             |
| Suporte a heterogêneo     | ✅ Sim                                | ✅ Sim                               | Depende do DBA                      |
| Downtime                  | Mínimo (replicação contínua)          | N/A                                 | Geralmente maior                    |
| Complexidade              | Baixa (gerenciado)                   | Média (ajuda na conversão)           | Alta (precisa de scripts manuais)   |
| Melhor uso                | Migração rápida e segura              | Suporte a DMS em conversões          | Casos muito específicos             |

---

## ⚠️ Observações

- O **DMS move dados**, mas não converte estruturas complexas (views, procedures, funções) — para isso existe o **SCT**.  
- É amplamente usado em **projetos de modernização**, por exemplo, migrar Oracle/SQL Server on-premises para **Aurora PostgreSQL** ou **MySQL**.  
- Cobra apenas pelo tempo de execução das instâncias de replicação usadas.  

# 🐳 Amazon ECS, ECR e Fargate

## 📌 O que é o Amazon ECS?

O **Amazon ECS (Elastic Container Service)** é um **serviço gerenciado de orquestração de contêineres** da AWS.  
Ele permite executar, gerenciar e escalar **aplicações em contêineres Docker** de forma simples e segura.

---

## 🔑 Principais características do ECS

- **Orquestração de contêineres:** gerencia execução, escalabilidade e balanceamento de cargas.  
- **Integração com AWS:** funciona nativamente com **VPC, IAM, CloudWatch e ALB**.  
- **Suporte a clusters:** permite agrupar instâncias EC2 para rodar contêineres.  
- **Flexibilidade de deploy:** funciona tanto com EC2 quanto com Fargate (serverless).  
- **Segurança:** controle de acesso granular via IAM, networking seguro e roles para contêineres.  

---

## 📌 O que é o Amazon ECR?

O **Amazon ECR (Elastic Container Registry)** é um **repositório gerenciado de imagens Docker**.  
Ele facilita **armazenar, versionar e distribuir imagens de contêineres** usadas pelo ECS, Fargate ou Kubernetes (EKS).

---

## 🔑 Principais características do ECR

- **Armazenamento seguro de imagens:** integrado com **IAM para controle de acesso**.  
- **Alta disponibilidade:** imagens replicadas automaticamente.  
- **Integração com ECS, Fargate e EKS:** facilita CI/CD e deploy contínuo.  
- **Compatibilidade Docker:** qualquer ferramenta Docker pode enviar e buscar imagens.  
- **Versionamento automático:** mantém histórico de imagens e tags.  

---

## 📌 O que é o AWS Fargate?

O **AWS Fargate** é uma **opção serverless do ECS** que permite executar contêineres **sem gerenciar servidores ou clusters EC2**.  
Você só precisa definir CPU, memória e container, e a AWS gerencia o resto.

---

## 🔑 Principais características do Fargate

- **Serverless:** elimina a necessidade de gerenciar instâncias EC2.  
- **Escalabilidade automática:** ajusta recursos conforme a demanda de contêineres.  
- **Integração com ECS e EKS:** deploy simples de aplicações containerizadas.  
- **Segurança:** isolamento de contêineres e integração com IAM e VPC.  
- **Cobrança por recurso usado:** paga apenas por CPU/memória consumidos pelo contêiner.  

---

## ✅ Casos de uso ECS + ECR + Fargate

- Deploy de **aplicações microservices** com contêineres.  
- **CI/CD:** pipelines integrados com CodePipeline e CodeBuild.  
- **Aplicações web e APIs** escaláveis.  
- **Execução de workloads temporários ou batch** sem gerenciar servidores.  
- **Desenvolvimento e teste de contêineres** com deploy rápido.  

---

## ⚖️ Comparação ECS + EC2 vs Fargate

| Aspecto                  | ECS + EC2                              | Fargate                               |
|---------------------------|----------------------------------------|---------------------------------------|
| Gerenciamento de servidores | Usuário gerencia instâncias EC2        | Serverless, AWS gerencia tudo         |
| Escalabilidade            | Manual ou com auto-scaling EC2         | Automática conforme contêineres       |
| Custo                     | Paga pelas instâncias EC2 provisionadas| Paga apenas por CPU/memória usada     |
| Flexibilidade             | Total controle sobre instâncias        | Menos controle, mais simplicidade     |
| Casos de uso              | Apps complexas, customização de infra  | Microservices, batch, apps serverless |

---

## ⚠️ Observações

- Use **ECR** sempre que precisar de um repositório seguro para imagens Docker.  
- Use **Fargate** quando quiser simplicidade e não quiser gerenciar servidores.  
- Use **ECS + EC2** quando precisar de maior controle sobre a infraestrutura ou customização de instâncias.  

# ☸️ Amazon EKS

## 📌 O que é o Amazon EKS?

O **Amazon EKS (Elastic Kubernetes Service)** é um **serviço gerenciado de Kubernetes** na AWS.  
Ele permite executar aplicações **containerizadas** com toda a orquestração, escalabilidade e gerenciamento nativos do **Kubernetes**, sem precisar instalar ou operar o plano de controle manualmente.

---

## 🔑 Principais características

- **Kubernetes gerenciado:** AWS provisiona e gerencia os **control planes** (master nodes).  
- **Execução de containers:** suporta Docker e outros runtimes compatíveis com Kubernetes.  
- **Escalabilidade automática:** integra com **Cluster Autoscaler** e HPA (Horizontal Pod Autoscaler).  
- **Segurança:** integração com **IAM, VPC, Security Groups, RBAC** e criptografia de dados.  
- **Integração com AWS:** funciona com **ECR, Fargate, CloudWatch, CloudTrail** e outros serviços.  
- **Alta disponibilidade:** múltiplas zonas de disponibilidade para os nós do cluster.  
- **Suporte híbrido:** conecta clusters on-premises com **EKS Anywhere**.  

---

## ✅ Casos de uso

- Deploy de **microservices complexos** com Kubernetes.  
- **Workloads em containers** que precisam de escalabilidade automática.  
- Aplicações que **já usam Kubernetes** e querem migrar para AWS sem refazer infraestrutura.  
- Integração com **CI/CD** usando CodePipeline, CodeBuild ou Jenkins.  
- Orquestração de **batch jobs ou pipelines de ML** em containers.  

---

## ⚖️ Comparação EKS vs ECS + Fargate

| Aspecto                  | Amazon EKS                          | ECS + Fargate                          |
|---------------------------|-------------------------------------|----------------------------------------|
| Orquestração             | Kubernetes (padrão open-source)     | ECS (AWS native)                        |
| Gerenciamento do plano   | AWS gerencia o control plane        | N/A                                     |
| Flexibilidade            | Total, padrão Kubernetes            | Limitada a funcionalidades ECS         |
| Complexidade             | Média-alta, curva de aprendizado    | Baixa, mais simples de usar             |
| Escalabilidade            | Automática via HPA/Cluster Autoscaler | Automática por contêineres Fargate    |
| Casos de uso              | Microservices complexos, multi-cloud | Microservices simples, serverless      |

---

## ⚠️ Observações

- Use **EKS** quando precisar de **compatibilidade total com Kubernetes** ou multi-cloud.  
- Use **ECS + Fargate** quando quiser **simplicidade**, sem gerenciar clusters complexos.  
- EKS pode ser integrado com **Fargate**, permitindo rodar pods sem se preocupar com instâncias EC2.  

# 🌐 Amazon API Gateway

## 📌 O que é o Amazon API Gateway?

O **Amazon API Gateway** é um serviço totalmente gerenciado da AWS que permite **criar, publicar, manter, monitorar e proteger APIs** RESTful e WebSocket em qualquer escala.  
Ele funciona como a **porta de entrada para aplicações e microservices**, conectando clientes a serviços back-end, sejam eles **Lambda, EC2, ECS, Fargate, EKS ou outros**.

---

## 🔑 Principais características

- **Gerenciamento de APIs:** criação, versionamento e deploy de APIs REST e WebSocket.  
- **Escalabilidade automática:** suporta milhões de chamadas simultâneas sem necessidade de gerenciar servidores.  
- **Integração nativa:** conecta facilmente com **AWS Lambda, ECS, Fargate, EKS, DynamoDB e outros serviços**.  
- **Segurança:** suporte a **IAM, Cognito, API keys, JWT, throttling e WAF**.  
- **Monitoramento e logging:** integração com **CloudWatch** para métricas e logs detalhados.  
- **Transformação de requests/responses:** manipulação de payloads sem alterar o back-end.  
- **Caching:** suporte a cache de respostas para melhorar performance e reduzir custos.  

---

## ✅ Casos de uso

- **Microservices:** expor endpoints de serviços back-end em arquiteturas serverless.  
- **APIs públicas ou privadas:** fornecer dados e funcionalidades para clientes, parceiros ou aplicações internas.  
- **Backends sem servidor:** integração com **AWS Lambda** para execução de código on-demand.  
- **Gateway para aplicações móveis ou web:** unificação de chamadas e controle de tráfego.  
- **Monitoramento e controle de acesso:** aplicar quotas, throttling e autenticação.  

---

## ⚖️ Comparação API Gateway vs Load Balancer vs App Runner

| Aspecto                  | API Gateway                        | Load Balancer (ALB/NLB)              | App Runner                           |
|---------------------------|-----------------------------------|-------------------------------------|--------------------------------------|
| Foco                      | Gerenciar APIs REST/WebSocket      | Distribuição de tráfego HTTP/TCP     | Deploy de apps e serviços web        |
| Escalabilidade            | Automática, serverless             | Automática, baseada em instâncias    | Automática, serverless               |
| Integração com back-end   | Lambda, ECS, Fargate, EKS, HTTP   | EC2, ECS, Fargate                    | Código ou contêiner                   |
| Segurança                 | IAM, Cognito, JWT, throttling      | Security Groups, WAF                 | IAM + TLS                             |
| Casos de uso              | APIs públicas ou internas          | Distribuição de tráfego e HA         | Deploy rápido de apps web/container  |

---

## ⚠️ Observações

- **API Gateway** é ideal para arquiteturas **serverless** e **microservices**.  
- Para simplesmente distribuir tráfego HTTP entre instâncias EC2, um **Load Balancer** pode ser suficiente.  
- Combine **API Gateway + Lambda** para criar backends totalmente serverless, escaláveis e seguros.  

<img width="1060" height="370" alt="image" src="https://github.com/user-attachments/assets/a9c54f91-9c1e-44d0-956f-4ab407b47bd6" />

# ⚙️ AWS Batch

## 📌 O que é o AWS Batch?

O **AWS Batch** é um serviço **totalmente gerenciado** que permite executar **jobs de computação em lote** de forma escalável na AWS.  
Ele provisiona automaticamente os recursos de **EC2 ou Fargate** necessários para processar jobs, gerencia filas e otimiza a execução para reduzir custos.

---

## 🔑 Principais características

- **Execução de jobs em lote:** ideal para processamento de grandes volumes de dados.  
- **Gerenciamento automático de recursos:** provisiona e escala **EC2 ou Fargate** conforme demanda.  
- **Filas de jobs:** organiza jobs por prioridade, dependência e grupo.  
- **Integração com AWS:** funciona com **S3, DynamoDB, RDS, Lambda, CloudWatch**.  
- **Suporte a contêineres:** executa jobs como contêineres Docker.  
- **Custo otimizado:** paga apenas pelos recursos consumidos durante a execução dos jobs.  
- **Monitoramento:** integra com CloudWatch para métricas, logs e alertas.  

---

## ✅ Casos de uso

- **Processamento de dados em lote:** ETL, análise de logs ou transformação de arquivos grandes.  
- **Renderização de mídia:** imagens, vídeos e animações.  
- **Simulações científicas e financeiras:** cálculos complexos que exigem alta computação.  
- **Treinamento de modelos ML:** executar múltiplos experimentos ou pipelines de ML.  
- **Automação de pipelines:** jobs agendados ou acionados por eventos do S3 ou Lambda.  

---

# AWS Lightsail

O **Amazon Lightsail** é um serviço de nuvem simplificado da AWS, voltado para quem precisa criar aplicações ou sites rapidamente sem a complexidade dos serviços mais avançados da AWS.

## Características principais
- 💡 **Facilidade de uso**: interface simples, voltada para iniciantes ou pequenos projetos.
- ⚡ **Servidores virtuais (VPS)**: você pode criar instâncias pré-configuradas com sistemas operacionais ou aplicações (WordPress, LAMP, Node.js, etc.).
- 🌍 **Rede e DNS**: gerenciamento simplificado de domínios, IPs estáticos e DNS.
- 📊 **Preço previsível**: planos fixos de cobrança mensal, diferente do modelo sob demanda do EC2.
- 🔗 **Integração com AWS**: pode se conectar com outros serviços AWS conforme sua aplicação cresce.

## Casos de uso
- Hospedagem de sites e blogs (ex.: WordPress).
- Aplicações web simples.
- Ambientes de teste e desenvolvimento.
- Pequenos bancos de dados e aplicações internas.
- Alternativa simplificada ao **EC2** para quem não precisa de alta escalabilidade.

## Comparação com EC2
- **Lightsail**: mais simples, preços previsíveis, ideal para iniciantes e pequenos projetos.
- **EC2**: mais flexível, escalável e integrado, mas mais complexo e com cobrança sob demanda.

---
👉 Em resumo, o **AWS Lightsail** é uma forma prática e econômica de usar a nuvem da AWS sem precisar lidar com toda a complexidade do EC2 e da VPC.

# AWS CloudFormation

O **AWS CloudFormation** é um serviço que permite **modelar e provisionar recursos da AWS de forma automática**, usando arquivos de configuração no formato **YAML ou JSON**, conhecidos como *templates*.  

Ele facilita a **infraestrutura como código (IaC)**, garantindo que o ambiente seja criado, atualizado e excluído de maneira previsível e controlada.

---

## Principais Características
- **Infraestrutura como Código (IaC)**: define recursos (EC2, RDS, S3, VPC etc.) em arquivos versionáveis.
- **Automatização**: cria e gerencia pilhas (*stacks*) de forma automática.
- **Reprodutibilidade**: garante que ambientes sejam criados de forma idêntica.
- **Integração**: funciona junto com serviços como IAM, CloudTrail, AWS Config e CodePipeline.

---

## Conceitos-Chave
- **Template**: arquivo YAML/JSON que descreve os recursos.
- **Stack (Pilha)**: conjunto de recursos criados a partir de um template.
- **Change Set**: visualização das alterações antes de aplicá-las.
- **Drift Detection**: detecta se houve mudanças manuais na infraestrutura fora do CloudFormation.

---

# AWS CloudFormation

O **AWS CloudFormation** é um serviço que permite **modelar e provisionar recursos da AWS de forma automática**, usando arquivos de configuração no formato **YAML ou JSON**, conhecidos como *templates*.  

Ele facilita a **infraestrutura como código (IaC)**, garantindo que o ambiente seja criado, atualizado e excluído de maneira previsível e controlada.

---

## Principais Características
- **Infraestrutura como Código (IaC)**: define recursos (EC2, RDS, S3, VPC etc.) em arquivos versionáveis.
- **Automatização**: cria e gerencia pilhas (*stacks*) de forma automática.
- **Reprodutibilidade**: garante que ambientes sejam criados de forma idêntica.
- **Integração**: funciona junto com serviços como IAM, CloudTrail, AWS Config e CodePipeline.

---

## Conceitos-Chave
- **Template**: arquivo YAML/JSON que descreve os recursos.
- **Stack (Pilha)**: conjunto de recursos criados a partir de um template.
- **Change Set**: visualização das alterações antes de aplicá-las.
- **Drift Detection**: detecta se houve mudanças manuais na infraestrutura fora do CloudFormation.

---

# AWS Elastic Beanstalk

## O que é?
O **AWS Elastic Beanstalk** é um serviço de *Platform as a Service (PaaS)* que facilita a implantação, gerenciamento e escalabilidade de aplicações na nuvem.  
Ele cuida automaticamente de **infraestrutura**, como provisionamento de servidores, balanceamento de carga, escalabilidade automática e monitoramento, enquanto você foca no **código da aplicação**.

---

## Como funciona?
1. O desenvolvedor faz o upload do código da aplicação (em linguagens suportadas).
2. O Beanstalk provisiona automaticamente:
   - **Instâncias EC2**
   - **Load Balancer**
   - **Auto Scaling**
   - **Banco de dados (opcional)**
3. Ele faz o deploy e gerencia as atualizações da aplicação.
4. O ambiente é monitorado pelo **Amazon CloudWatch**.

---

## Tecnologias suportadas
- **Java**
- **.NET**
- **Node.js**
- **Python**
- **Ruby**
- **Go**
- **PHP**
- **Docker**

---

## Benefícios
- Deploy rápido e simplificado.
- Escalabilidade automática.
- Integração com outros serviços AWS.
- Gerenciamento simplificado de ambientes.
- Sem custo adicional além dos recursos provisionados (EC2, RDS, etc.).

---

## Caso de uso
- Hospedagem de aplicações web em produção.
- APIs REST simples.
- Ambientes de testes e desenvolvimento.
- Aplicações que precisam de escalabilidade automática.

---

# AWS CodeDeploy

## O que é?
O **AWS CodeDeploy** é um serviço de **deploy automatizado** que facilita a atualização de aplicações em instâncias **Amazon EC2**, **Lambda** ou servidores **on-premises**, reduzindo erros manuais e diminuindo o tempo de inatividade.

---

## Principais Recursos
- **Automação** do processo de deploy (zero-downtime quando configurado com load balancers).  
- **Suporte a diferentes plataformas**: EC2, ECS, Lambda e até servidores locais.  
- **Estratégias de implantação**: in-place, blue/green, canary e linear.  
- **Monitoramento e rollback automático** em caso de falha.  
- **Integração com CI/CD**: funciona com CodePipeline, CodeBuild, GitHub, Bitbucket, Jenkins etc.  

---

## Tipos de Deploy
- **In-Place Deployment**: substitui a versão da aplicação diretamente na instância.  
- **Blue/Green Deployment**: cria um novo ambiente com a nova versão e depois redireciona o tráfego.  

---

## Benefícios
- Evita indisponibilidade durante atualizações.  
- Padroniza o processo de deploy.  
- Fácil integração com pipelines de entrega contínua.  
- Funciona tanto em nuvem quanto em servidores locais.  

---

## Casos de Uso
- Deploy de APIs em EC2.  
- Deploy de microsserviços em containers ECS.  
- Deploy de funções serverless (AWS Lambda).  
- Atualização de aplicações corporativas em servidores físicos/VMs.  

---

✅ **Resumo**: O **CodeDeploy** ajuda a automatizar e padronizar o processo de deploy, reduzindo falhas e garantindo maior disponibilidade das aplicações.

# AWS CodeCommit

## O que é?
O **AWS CodeCommit** é um serviço de **repositório Git totalmente gerenciado** na nuvem AWS.  
Ele permite armazenar, versionar e colaborar em código-fonte de forma segura e escalável, sem a necessidade de hospedar servidores Git por conta própria.

---

## Principais Recursos
- **Repositórios privados Git** gerenciados pela AWS.  
- **Integração nativa com IAM** para controle de permissões.  
- **Criptografia automática** em repouso (AWS KMS) e em trânsito (HTTPS/SSH).  
- **Alta disponibilidade**, rodando em múltiplas zonas de disponibilidade.  
- **Escalável**: suporta qualquer tamanho de repositório.  
- **Notificações e triggers** via SNS, Lambda e CloudWatch Events.  

---

## Benefícios
- Elimina a necessidade de hospedar e gerenciar servidores Git.  
- Controle de acesso seguro usando políticas do IAM.  
- Repositórios ilimitados sem cobrança por usuário.  
- Integração direta com **CodeBuild**, **CodePipeline** e **CodeDeploy**.  

---

## Casos de Uso
- Hospedagem centralizada de código-fonte em equipes.  
- Versionamento seguro de infraestrutura como código (CloudFormation, Terraform).  
- Integração em pipelines de CI/CD.  
- Alternativa privada ao GitHub/GitLab/Bitbucket para projetos críticos.  

---

✅ **Resumo**: O **CodeCommit** é um repositório Git privado, seguro e totalmente gerenciado pela AWS, ideal para equipes que já usam outros serviços AWS e querem integração nativa no fluxo de desenvolvimento.

# AWS CodeBuild

## O que é?
O **AWS CodeBuild** é um serviço de **integração contínua (CI)** totalmente gerenciado.  
Ele compila o código-fonte, executa testes automatizados e gera artefatos prontos para implantação, sem precisar configurar ou manter servidores de build.

---

## Principais Recursos
- **Build totalmente gerenciado**: a AWS cuida da infraestrutura.  
- **Escalabilidade automática**: cada build roda em um contêiner isolado.  
- **Pagamentos sob demanda**: só paga pelo tempo de build usado.  
- **Buildspec.yml**: arquivo de configuração que define instruções de build e testes.  
- **Logs em tempo real** via Amazon CloudWatch.  
- **Integração nativa** com CodeCommit, CodePipeline, CodeDeploy, GitHub e Bitbucket.  

---

## Benefícios
- Elimina a necessidade de manter servidores próprios de CI.  
- Fácil integração em pipelines de CI/CD.  
- Escalável e sob demanda, sem filas de espera.  
- Seguro, com permissões gerenciadas pelo IAM.  

---

## Casos de Uso
- Compilar aplicações Java, Node.js, Python, Go, .NET e muitas outras.  
- Executar testes automatizados.  
- Gerar imagens Docker e enviar para o Amazon ECR.  
- Preparar artefatos de deploy para o CodeDeploy ou S3.  

---

# AWS CodePipeline

## O que é?
O **AWS CodePipeline** é um serviço de **integração e entrega contínua (CI/CD)** totalmente gerenciado.  
Ele automatiza o processo de **build, teste e deploy** de aplicações sempre que há uma mudança no código, garantindo entregas rápidas e consistentes.

---

## Como funciona?
O pipeline é dividido em **estágios**. Cada estágio contém **ações** que podem executar builds, testes ou deploys.  
Um fluxo típico pode ser:
1. **Source** → Obter código (ex.: CodeCommit, GitHub, S3).  
2. **Build** → Compilar e testar (ex.: CodeBuild).  
3. **Deploy** → Implantar aplicação (ex.: CodeDeploy, Elastic Beanstalk, ECS).  

---

## Principais Recursos
- **Integração nativa** com CodeCommit, CodeBuild, CodeDeploy e CodeStar.  
- **Compatível com provedores externos** como GitHub, GitLab, Jenkins e Bitbucket.  
- **Execução automatizada**: dispara pipelines em cada alteração de código.  
- **Paralelismo**: permite múltiplas ações rodando em paralelo.  
- **Controle de aprovação manual** antes de determinados estágios.  
- **Monitoramento** com CloudWatch e AWS CloudTrail.  

---

## Benefícios
- Automação completa do ciclo de vida da aplicação.  
- Reduz erros humanos durante o deploy.  
- Acelera o tempo de entrega de novas versões.  
- Flexível e personalizável com integrações externas.  

---

## Casos de Uso
- Automação de **CI/CD** para microsserviços em **ECS/EKS**.  
- Deploy contínuo de aplicações em **EC2** ou **Elastic Beanstalk**.  
- Criação de pipelines híbridos (AWS + Jenkins/GitHub Actions).  
- Aprovação manual antes de ir para produção.  

---

## Exemplo de Pipeline
- **Source**: CodeCommit (recebe push de novo código).  
- **Build**: CodeBuild (compila e roda testes).  
- **Deploy**: CodeDeploy (envia para instâncias EC2).  

---

✅ **Resumo**: O **CodePipeline** conecta serviços como **CodeCommit**, **CodeBuild** e **CodeDeploy**, permitindo a automação completa do fluxo de **CI/CD** na AWS.
