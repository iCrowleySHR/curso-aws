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

