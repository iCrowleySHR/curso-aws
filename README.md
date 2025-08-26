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

