# 🎬 Vídeo 4.2 - Pipeline GitOps Automatizado

**Aula**: 4 - GitOps  
**Vídeo**: 4.2  
**Temas**: CI/CD + GitOps; Update manifests; ArgoCD Sync; Automation  

---

## ⚠️ Pré-requisitos

**IMPORTANTE**: Este vídeo assume que você já tem:
- ✅ Cluster EKS `cicd-lab` criado na **Aula 01** (repositório da Aula 01)
- ✅ ArgoCD instalado e configurado (Vídeo 4.1)
- ✅ kubectl configurado com acesso ao cluster

**Se ainda não criou o cluster:**
- Volte ao **repositório da Aula 01** e crie o cluster EKS
- O cluster deve ter o nome: `cicd-lab`
- Depois retorne ao **Vídeo 4.1** para instalar o ArgoCD

---

## 📚 Parte 1: Conceito CI/CD + GitOps

### Passo 1: Fluxo Completo CI/CD + GitOps

```mermaid
graph TB
    subgraph "❌ Pipeline Tradicional"
        A1[Code] --> A2[Build]
        A2 --> A3[Test]
        A3 --> A4[Deploy]
        A4 -->|kubectl apply| A5[Cluster]
    end
    
    subgraph "✅ Pipeline GitOps"
        B1[Code] --> B2[Build]
        B2 --> B3[Test]
        B3 --> B4[Push Image]
        B4 --> B5[Update Git Manifests]
        B5 --> B6[Git Repository]
        B7[ArgoCD] -->|pull| B6
        B7 -->|sync| B8[Cluster]
    end
```

**Separação de responsabilidades:**

```mermaid
graph LR
    A[CI Pipeline] -->|1. Build & Test| B[Container Image]
    B -->|2. Push| C[Registry ECR]
    C -->|3. Update tag| D[Git Manifests]
    D -->|4. Source of Truth| E[ArgoCD]
    E -->|5. Deploy & Sync| F[Cluster]
```

| Componente | Responsabilidade |
|------------|------------------|
| **CI Pipeline** | Build, test, push image |
| **Git Repository** | Source of truth para manifests |
| **ArgoCD** | Deploy e sync automático |
| **Cluster** | Executar aplicações |

---

## � Parte 2: Configurar Secrets no GitHub

### Passo 2: Adicionar AWS Credentials como Secrets

**⚠️ IMPORTANTE**: Configure os secrets ANTES de criar os workflows!

**Onde configurar:**
1. Acesse seu repositório no GitHub
2. **Settings** → **Secrets and variables** → **Actions**
3. Clique em **New repository secret**

**Secrets necessários:**

| Secret Name | Descrição | Como obter |
|-------------|-----------|------------|
| `AWS_ACCESS_KEY_ID` | Access Key ID da AWS | AWS Learner Lab → AWS Details → Show |
| `AWS_SECRET_ACCESS_KEY` | Secret Access Key da AWS | AWS Learner Lab → AWS Details → Show |
| `AWS_SESSION_TOKEN` | Session Token (AWS Learner Lab) | AWS Learner Lab → AWS Details → Show |
| `ARGOCD_PASSWORD` | Senha do ArgoCD admin | `kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" \| base64 -d` |

**Passo a passo para adicionar cada secret:**
```
1. Clique em "New repository secret"
2. Name: AWS_ACCESS_KEY_ID
3. Secret: <cole o valor do AWS Learner Lab>
4. Clique em "Add secret"
5. Repita para os outros 3 secrets
```

**⚠️ Nota sobre AWS Learner Lab:**
- Os secrets AWS expiram quando a sessão do Learner Lab termina
- Você precisará atualizar os secrets a cada nova sessão
- Sempre inicie o Learner Lab antes de executar workflows

---

## 🔄 Parte 3: Criar Workflows GitHub Actions

### Passo 3: Criar Estrutura de Diretórios

```bash
# Navegar para o diretório do projeto
cd fiap-dclt-aula04

# Criar estrutura de diretórios
mkdir -p .github/workflows

# Verificar
ls -la .github/workflows/
```

---

## 📦 Parte 4: Criar Pipeline GitOps Unificado

### Passo 4: Entender a Estrutura do Pipeline

**⚠️ Nova Abordagem:**
- **1 único arquivo** em vez de 3 separados
- **4 jobs sequenciais** que executam em ordem
- **Compartilhamento de dados** entre jobs usando `outputs`

**Fluxo do Pipeline:**
```
┌──────────────────────────────────────────────────────────┐
│ Push em app/ → Trigger Pipeline                          │
└──────────────────────────────────────────────────────────┘
                        ↓
┌──────────────────────────────────────────────────────────┐
│ JOB 1: build-and-push                                    │
│ 🐳 Build Docker image e push para ECR                    │
│ Output: image-tag, ecr-registry                          │
└──────────────────────────────────────────────────────────┘
                        ↓ needs: build-and-push
┌──────────────────────────────────────────────────────────┐
│ JOB 2: update-gitops                                     │
│ 📝 Atualiza kustomization.yaml com nova tag              │
│ Faz commit e push das mudanças                           │
└──────────────────────────────────────────────────────────┘
                        ↓ needs: update-gitops
┌──────────────────────────────────────────────────────────┐
│ JOB 3: validate-manifests                                │
│ ✅ Valida manifests com kustomize build                  │
└──────────────────────────────────────────────────────────┘
                        ↓ needs: all
┌──────────────────────────────────────────────────────────┐
│ JOB 4: pipeline-summary                                  │
│ 📊 Mostra resumo completo do pipeline                    │
└──────────────────────────────────────────────────────────┘
```

### Passo 5: Criar gitops-pipeline.yml

**📋 Arquivo Completo:**

O arquivo completo está disponível em `.github/workflows/gitops-pipeline.yml` no repositório.

**Vamos criar passo a passo:**

**Linux / macOS:**
```bash
# Navegar para o diretório do projeto
cd fiap-dclt-aula04

# Criar estrutura de diretórios
mkdir -p .github/workflows

# Copiar arquivo do repositório
cp .github/workflows/gitops-pipeline.yml .github/workflows/gitops-pipeline.yml

# OU criar manualmente (veja estrutura abaixo)
```


**Ver arquivo completo:** `.github/workflows/gitops-pipeline.yml`

**Principais seções do arquivo:**

#### 1. Configuração Inicial
```yaml
name: 🚀 GitOps Pipeline - Build, Update & Sync

permissions:
  contents: write  # Necessário para commit no Job 2

env:
  AWS_REGION: us-east-1
  ECR_REPOSITORY: fiap-todo-api
```

#### 2. Job 1 - Build and Push
```yaml
build-and-push:
  outputs:
    image-tag: ${{ steps.set-tag.outputs.tag }}
    ecr-registry: ${{ steps.login-ecr.outputs.registry }}
  steps:
    - Build Docker image
    - Push para ECR
    - Exporta tag e registry para próximos jobs
```

#### 3. Job 2 - Update GitOps
```yaml
update-gitops:
  needs: build-and-push  # Espera Job 1
  steps:
    - Setup Kustomize
    - Update kustomization.yaml com nova tag
    - Commit e push das mudanças
```

#### 4. Job 3 - Validate
```yaml
validate-manifests:
  needs: update-gitops  # Espera Job 2
  steps:
    - Checkout latest code
    - Validate com kustomize build
```

#### 5. Job 4 - Summary
```yaml
pipeline-summary:
  needs: [build-and-push, update-gitops, validate-manifests]
  if: always()  # Roda mesmo se houver falha
  steps:
    - Mostra resumo completo do pipeline
```

### Passo 6: Verificar Arquivo Criado

```bash
# Ver arquivo
cat .github/workflows/gitops-pipeline.yml

# Verificar estrutura
ls -la .github/workflows/

# Deve mostrar apenas:
# gitops-pipeline.yml
```

---

## 🧪 Parte 5: Testar o Pipeline

### Passo 7: Fazer Commit e Push

```bash
# Adicionar arquivo
git add .github/workflows/gitops-pipeline.yml

# Commit
git commit -m "feat: adicionar pipeline GitOps unificado"

# Push
git push origin main
```

### Passo 8: Testar Pipeline com Mudança no App

```bash
# Fazer mudança no código
echo "// Test pipeline" >> app/src/app.js

# Commit e push
git add app/
git commit -m "test: trigger pipeline"
git push origin main
```

### Passo 9: Acompanhar Execução

**No GitHub:**
1. Acesse: `https://github.com/SEU_USUARIO/fiap-dclt-aula04/actions`
2. Veja o workflow "🚀 GitOps Pipeline - Build, Update & Sync"
3. Clique para ver detalhes

**Você verá 4 jobs executando em sequência:**
```
✅ Job 1: 🐳 Build and Push Docker Image
   ↓
✅ Job 2: 📝 Update GitOps Manifests
   ↓
✅ Job 3: ✅ Validate Manifests
   ↓
✅ Job 4: 📊 Pipeline Summary
```

---

## 🎯 Parte 6: Entender o Fluxo Completo

### Fluxo End-to-End Automático

```
1. Developer faz push em app/
   ↓
2. GitHub Actions detecta mudança
   ↓
3. Job 1: Build image e push para ECR
   ↓
4. Job 2: Atualiza kustomization.yaml automaticamente
   ↓
5. Job 3: Valida manifests
   ↓
6. Job 4: Mostra resumo
   ↓
7. ArgoCD detecta mudança no Git (auto-sync ~3min)
   ↓
8. ArgoCD aplica mudanças no cluster
   ↓
9. Pods são atualizados com nova imagem
```

### Verificar Deployment

```bash
# Ver pods sendo atualizados
kubectl get pods -n fiap-todo-prod -w

# Ver status no ArgoCD
kubectl port-forward svc/argocd-server -n argocd 8080:443

# Abrir: http://localhost:8080
# Ver aplicação sincronizando automaticamente
```

---

## 📊 Parte 7: Resumo

### O Que Aprendemos

✅ **Pipeline Unificado**: 1 arquivo com 4 jobs sequenciais
✅ **Jobs com Dependências**: Uso de `needs:` para ordem de execução
✅ **Compartilhamento de Dados**: Outputs entre jobs
✅ **GitOps Automático**: Commit automático de manifests
✅ **Validação**: Kustomize build antes do deploy
✅ **Observabilidade**: Summaries em cada job

### Vantagens

- 🎯 **Fluxo claro**: Fácil de entender e debugar
- 🔄 **Totalmente automático**: Push → Deploy
- ✅ **Validação integrada**: Catch errors antes do deploy
- 📊 **Visibilidade**: Logs organizados por job
- 🛠️ **Manutenível**: 1 arquivo para gerenciar

### Próximos Passos

- **Vídeo 4.3**: FluxCD como alternativa ao ArgoCD
- **Comparação**: ArgoCD vs FluxCD
- **Escolha**: Quando usar cada ferramenta

