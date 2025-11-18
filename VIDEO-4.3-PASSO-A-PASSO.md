# 🎬 Vídeo 4.3 - FluxCD: GitOps Automático

**Aula**: 4 - GitOps  
**Vídeo**: 4.3  
**Temas**: FluxCD, Bootstrap, Image Automation, Comparação com ArgoCD  

---

## ⚠️ Pré-requisitos

- ✅ Cluster EKS `cicd-lab` criado (Aula 01)
- ✅ kubectl configurado
- ✅ Conhecimento de ArgoCD (Vídeos 4.1 e 4.2)

---

## 📚 Parte 1: Entendendo FluxCD

### O Que é FluxCD?

**FluxCD** é uma ferramenta GitOps que automatiza deployments no Kubernetes **sem interface gráfica**.

**Diferença Principal:**
```
ArgoCD = GitOps + UI Visual + Controle Manual
FluxCD = GitOps Puro + Automação Total + CLI
```

### Arquitetura Simplificada

```
┌─────────────────────────────────────────────────────────┐
│  FLUXCD - Como Funciona                                 │
└─────────────────────────────────────────────────────────┘

1. GIT REPOSITORY (GitHub)
   └─ Manifests YAML
          ↓
2. SOURCE CONTROLLER (monitora Git a cada 1min)
   └─ Detecta mudanças
          ↓
3. KUSTOMIZE CONTROLLER (aplica mudanças)
   └─ kubectl apply automático
          ↓
4. KUBERNETES CLUSTER
   └─ Pods atualizados

EXTRA: IMAGE AUTOMATION (opcional)
   └─ Monitora ECR → Atualiza Git → Loop reinicia
```

### Componentes do FluxCD

**5 Controllers principais:**

1. **Source Controller** 📥
   - Monitora: Git, Helm repos, S3 buckets
   - Intervalo: A cada 1 minuto (configurável)
   - Função: Detectar mudanças

2. **Kustomize Controller** 🔧
   - Aplica: Manifests Kustomize
   - Função: `kubectl apply` automático

3. **Helm Controller** ⎈
   - Gerencia: Helm charts
   - Função: Deploy de aplicações Helm

4. **Notification Controller** 📢
   - Envia: Alertas para Slack, Teams, Email
   - Função: Notificar sobre deployments

5. **Image Automation** 🤖
   - Monitora: ECR, Docker Hub, etc
   - Função: Atualizar tags automaticamente no Git

---

## 🆚 Parte 2: ArgoCD vs FluxCD

### Comparação Rápida

| Característica | ArgoCD | FluxCD |
|----------------|--------|--------|
| **Interface** | ✅ UI Web rica | ❌ Apenas CLI |
| **Facilidade** | ✅ Mais fácil (visual) | ⚠️ Requer CLI |
| **Multi-cluster** | ✅ Nativo | ⚠️ Requer config |
| **RBAC/SSO** | ✅ Integrado | ⚠️ Via Kubernetes |
| **Image Automation** | ❌ Não tem | ✅ Nativo |
| **Peso** | ⚠️ Mais pesado | ✅ Mais leve |
| **GitOps Puro** | ⚠️ Permite UI | ✅ 100% Git |
| **Modularidade** | ❌ Monolítico | ✅ Modular |

### Quando Usar Cada Um?

**Use ArgoCD se:**
- ✅ Você quer interface visual
- ✅ Equipe prefere UI para troubleshooting
- ✅ Precisa de multi-cluster fácil
- ✅ Quer SSO/RBAC integrado

**Use FluxCD se:**
- ✅ Você quer GitOps 100% puro
- ✅ Precisa de Image Automation
- ✅ Prefere abordagem modular
- ✅ Quer solução mais leve

**Nossa Recomendação para Iniciantes:** **ArgoCD** (mais visual e fácil de aprender)

---

## 🚀 Parte 3: Instalar FluxCD

### Passo 1: Instalar Flux CLI

**macOS:**
```bash
# Homebrew (recomendado)
brew install fluxcd/tap/flux

# Verificar
flux --version
```

**Linux:**
```bash
# Script oficial
curl -s https://fluxcd.io/install.sh | sudo bash

# Verificar
flux --version
```

**Windows (PowerShell):**
```powershell
# Chocolatey
choco install flux

# Verificar
flux --version
```

### Passo 2: Verificar Cluster

```bash
# Pre-check do Flux
flux check --pre

# Deve mostrar:
# ✔ Kubernetes 1.28.0 >=1.26.0
# ✔ prerequisites checks passed
```

---

## 🔧 Parte 4: Bootstrap - O Que É e Como Funciona

### O Que é Bootstrap?

**Bootstrap** é o processo de **instalar e configurar** o FluxCD no cluster de forma **automática e GitOps**.

**O que o Bootstrap faz:**

```
1. Cria namespace flux-system no cluster
2. Instala os 5 controllers do Flux
3. Cria manifests no seu repositório Git (pasta flux-system/)
4. Configura sync automático Git → Cluster
5. FluxCD passa a se auto-gerenciar via Git
```

**Por que é especial?**
- ✅ FluxCD se instala via GitOps
- ✅ FluxCD se atualiza via GitOps
- ✅ Tudo fica versionado no Git

### Passo 3: Criar Token GitHub

**Por que precisa?**
- FluxCD precisa ler e escrever no seu repositório
- Vai criar a pasta `flux-system/` automaticamente

**Como criar:**

1. Acesse: https://github.com/settings/tokens
2. Clique: **Generate new token** → **Classic**
3. Selecione: **repo** (todas as permissões)
4. Clique: **Generate token**
5. **Copie o token** (você não verá novamente!)

### Passo 4: Executar Bootstrap

**Linux / macOS:**
```bash
# Definir variáveis
export GITHUB_TOKEN=ghp_seu_token_aqui
export GITHUB_USER=seu_usuario

# Bootstrap FluxCD
flux bootstrap github \
  --owner=$GITHUB_USER \
  --repository=fiap-dclt-aula04 \
  --branch=main \
  --path=flux-system \
  --personal

# Aguardar (leva ~2 minutos)
```

**Windows (PowerShell):**
```powershell
# Definir variáveis
$env:GITHUB_TOKEN = "ghp_seu_token_aqui"
$env:GITHUB_USER = "seu_usuario"

# Bootstrap FluxCD
flux bootstrap github `
  --owner=$env:GITHUB_USER `
  --repository=fiap-dclt-aula04 `
  --branch=main `
  --path=flux-system `
  --personal
```

**O que acontece durante o Bootstrap:**

```
► Connecting to github.com
✔ repository cloned
► applying manifests
✔ source-controller: deployment ready
✔ kustomize-controller: deployment ready
✔ helm-controller: deployment ready
✔ notification-controller: deployment ready
✔ image-reflector-controller: deployment ready
✔ image-automation-controller: deployment ready
✔ all components are healthy
```

### Passo 5: Verificar Instalação

```bash
# Ver pods do Flux
kubectl get pods -n flux-system

# Deve mostrar:
# NAME                                       READY   STATUS
# source-controller-xxx                      1/1     Running
# kustomize-controller-xxx                   1/1     Running
# helm-controller-xxx                        1/1     Running
# notification-controller-xxx                1/1     Running
# image-reflector-controller-xxx             1/1     Running
# image-automation-controller-xxx            1/1     Running

# Ver o que Flux está monitorando
flux get sources git

# Ver kustomizations
flux get kustomizations
```

### Passo 6: Ver Mudanças no Git

```bash
# Pull das mudanças que Flux criou
git pull origin main

# Ver nova pasta
ls -la flux-system/

# Flux criou:
# flux-system/
# ├── gotk-components.yaml    # Todos os controllers
# ├── gotk-sync.yaml          # Configuração de sync
# └── kustomization.yaml      # Kustomize config
```

---

## 📦 Parte 5: Configurar Aplicação no FluxCD

### Passo 7: Criar Estrutura de Pastas

```bash
# Criar pasta para configs Flux
mkdir -p gitops-repo/clusters/production

# Verificar
ls -la gitops-repo/clusters/production/
```

### Passo 8: Criar GitRepository Source

**O que é GitRepository?**
- Define **qual repositório Git** o Flux deve monitorar
- Define **intervalo** de verificação (ex: 1 minuto)
- Define **qual pasta** monitorar

**⚠️ IMPORTANTE:** Substitua `SEU_USUARIO` pelo seu usuário GitHub!

```bash
# Criar GitRepository
cat > gitops-repo/clusters/production/fiap-todo-api-source.yaml << 'EOF'
apiVersion: source.toolkit.fluxcd.io/v1
kind: GitRepository
metadata:
  name: fiap-todo-api
  namespace: flux-system
spec:
  interval: 1m                    # Verifica a cada 1 minuto
  url: https://github.com/SEU_USUARIO/fiap-dclt-aula04
  ref:
    branch: main
  ignore: |
    /*                            # Ignora tudo
    !/gitops-repo/applications/   # Exceto esta pasta
EOF

echo "✅ GitRepository criado!"
```

**Explicação:**
- `interval: 1m` → Flux verifica Git a cada 1 minuto
- `url` → Seu repositório GitHub
- `ignore` → Monitora apenas `gitops-repo/applications/`

### Passo 9: Criar Namespace da Aplicação

**⚠️ IMPORTANTE:** Precisamos criar o namespace antes!

```bash
# Criar namespace
kubectl create namespace fiap-todo-prod

# Verificar
kubectl get namespaces | grep fiap-todo
```

### Passo 10: Criar Kustomization

**O que é Kustomization (Flux)?**
- Define **o que aplicar** no cluster
- Define **onde estão** os manifests
- Define **namespace** de destino

```bash
# Criar Kustomization
cat > gitops-repo/clusters/production/fiap-todo-api-kustomization.yaml << 'EOF'
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: fiap-todo-api
  namespace: flux-system
spec:
  interval: 5m                    # Reconcilia a cada 5 minutos
  sourceRef:
    kind: GitRepository
    name: fiap-todo-api           # Referência ao GitRepository acima
  path: ./gitops-repo/applications/fiap-todo-api/overlays/production
  prune: true                     # Remove recursos deletados do Git
  wait: true                      # Aguarda recursos ficarem prontos
  timeout: 2m
  targetNamespace: fiap-todo-prod # ⚠️ Namespace onde aplicar os recursos
EOF

echo "✅ Kustomization criado!"
```

**Explicação:**
- `sourceRef` → Usa o GitRepository criado antes
- `path` → Pasta com os manifests Kustomize
- `prune: true` → Se deletar do Git, deleta do cluster
- `wait: true` → Aguarda pods ficarem ready

### Passo 11: Aplicar Configurações

```bash
# Aplicar GitRepository e Kustomization
kubectl apply -f gitops-repo/clusters/production/

# Ver status
flux get sources git
flux get kustomizations

# Ver reconciliação em tempo real
flux logs --follow
```

### Passo 12: Fazer Commit

```bash
# Commit das configurações Flux
git add gitops-repo/clusters/
git commit -m "feat: configurar FluxCD para fiap-todo-api"
git push origin main

# Flux vai detectar e aplicar automaticamente!
```

---

## 🤖 Parte 6: Image Automation - Automação Total

### O Que é Image Automation?

**Image Automation** permite que FluxCD:
1. **Monitore** seu ECR (ou Docker Hub)
2. **Detecte** novas tags de imagem
3. **Atualize** automaticamente o Git
4. **Deploy** automático da nova versão

**Fluxo Completo:**
```
Push código → GitHub Actions builda → Push ECR
    ↓
FluxCD detecta nova tag no ECR
    ↓
FluxCD atualiza kustomization.yaml no Git
    ↓
FluxCD detecta mudança no Git
    ↓
FluxCD aplica no cluster
    ↓
Deploy automático! 🎉
```

### Passo 13: Criar ImageRepository

**O que faz:** Monitora o ECR para novas imagens

```bash
cat > gitops-repo/clusters/production/fiap-todo-api-imagerepository.yaml << 'EOF'
apiVersion: image.toolkit.fluxcd.io/v1beta2
kind: ImageRepository
metadata:
  name: fiap-todo-api
  namespace: flux-system
spec:
  image: 123456789012.dkr.ecr.us-east-1.amazonaws.com/fiap-todo-api
  interval: 1m                    # Verifica ECR a cada 1 minuto
  secretRef:
    name: ecr-credentials         # Credenciais AWS (criar depois)
EOF
```

### Passo 14: Criar ImagePolicy

**O que faz:** Define qual tag usar (ex: sempre a mais recente)

```bash
cat > gitops-repo/clusters/production/fiap-todo-api-imagepolicy.yaml << 'EOF'
apiVersion: image.toolkit.fluxcd.io/v1beta2
kind: ImagePolicy
metadata:
  name: fiap-todo-api
  namespace: flux-system
spec:
  imageRepositoryRef:
    name: fiap-todo-api
  policy:
    semver:
      range: '>=1.0.0'            # Usa tags semver >= 1.0.0
    # OU para usar sempre latest:
    # alphabetical:
    #   order: asc
EOF
```

### Passo 15: Criar ImageUpdateAutomation

**O que faz:** Atualiza o Git automaticamente

```bash
cat > gitops-repo/clusters/production/fiap-todo-api-imageupdateautomation.yaml << 'EOF'
apiVersion: image.toolkit.fluxcd.io/v1beta1
kind: ImageUpdateAutomation
metadata:
  name: fiap-todo-api
  namespace: flux-system
spec:
  interval: 1m
  sourceRef:
    kind: GitRepository
    name: fiap-todo-api
  git:
    checkout:
      ref:
        branch: main
    commit:
      author:
        name: fluxcdbot
        email: flux@users.noreply.github.com
      messageTemplate: '🤖 Update image {{range .Updated.Images}}{{println .}}{{end}}'
    push:
      branch: main
  update:
    path: ./gitops-repo/applications/fiap-todo-api/overlays/production
    strategy: Setters                # Usa markers no YAML
EOF
```

### Passo 16: Adicionar Marker no Kustomization

**Editar:** `gitops-repo/applications/fiap-todo-api/overlays/production/kustomization.yaml`

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - ../../base

namespace: fiap-todo-prod

images:
  - name: fiap-todo-api
    newName: 123456789012.dkr.ecr.us-east-1.amazonaws.com/fiap-todo-api
    newTag: v1.0.0  # {"$imagepolicy": "flux-system:fiap-todo-api"}
```

**Explicação do marker:**
- `# {"$imagepolicy": "flux-system:fiap-todo-api"}` 
- FluxCD vai substituir `v1.0.0` pela tag mais recente do ECR
- Commit automático no Git

### Passo 17: Criar Secret para ECR

```bash
# Criar secret com credenciais AWS
kubectl create secret generic ecr-credentials \
  --from-literal=AWS_ACCESS_KEY_ID=$AWS_ACCESS_KEY_ID \
  --from-literal=AWS_SECRET_ACCESS_KEY=$AWS_SECRET_ACCESS_KEY \
  --from-literal=AWS_SESSION_TOKEN=$AWS_SESSION_TOKEN \
  -n flux-system
```

### Passo 18: Aplicar Image Automation

```bash
# Aplicar todas as configs
kubectl apply -f gitops-repo/clusters/production/

# Ver status
flux get image repository
flux get image policy
flux get image update

# Ver logs
flux logs --follow
```

---

## 🎯 Parte 7: Testar Fluxo Completo

### Passo 19: Fazer Deploy de Nova Versão

```bash
# 1. Fazer mudança no código
echo "// New feature" >> app/src/app.js

# 2. Commit e push
git add app/
git commit -m "feat: nova funcionalidade"
git push origin main

# 3. GitHub Actions vai:
#    - Build imagem
#    - Push para ECR com tag (ex: v1.1.0)

# 4. FluxCD vai (automático):
#    - Detectar nova tag no ECR
#    - Atualizar kustomization.yaml no Git
#    - Fazer commit automático
#    - Aplicar no cluster

# 5. Ver logs
flux logs --follow
```

### Passo 20: Verificar Deployment

```bash
# Ver pods sendo atualizados
kubectl get pods -n fiap-todo-prod -w

# Ver commits automáticos do Flux
git pull origin main
git log --oneline

# Deve ver commit do fluxcdbot:
# 🤖 Update image 123456789012.dkr.ecr.us-east-1.amazonaws.com/fiap-todo-api:v1.1.0
```

---

## 📊 Parte 8: Resumo e Comparação Final

### FluxCD - Prós e Contras

**✅ Vantagens:**
- GitOps 100% puro (tudo via Git)
- Image Automation nativa
- Mais leve que ArgoCD
- Modular (instala só o que precisa)
- Auto-gerenciamento via GitOps

**❌ Desvantagens:**
- Sem UI visual (apenas CLI)
- Curva de aprendizado maior
- Troubleshooting mais difícil
- Multi-cluster requer config manual

### ArgoCD - Prós e Contras

**✅ Vantagens:**
- UI visual rica e intuitiva
- Fácil de aprender e usar
- Multi-cluster nativo
- RBAC/SSO integrado
- Troubleshooting visual

**❌ Desvantagens:**
- Sem Image Automation nativa
- Mais pesado
- Permite bypass do GitOps via UI

### Nossa Recomendação

**Para Produção:**
- **Pequenas equipes / Iniciantes**: ArgoCD (UI facilita)
- **Equipes DevOps maduras**: FluxCD (GitOps puro)
- **Precisa Image Automation**: FluxCD
- **Multi-cluster complexo**: ArgoCD

**Para Aprendizado:**
- **Comece com ArgoCD** (mais visual)
- **Depois experimente FluxCD** (entender GitOps puro)

---

## 🧹 Parte 9: Limpeza (Opcional)

### Desinstalar FluxCD

```bash
# Desinstalar Flux
flux uninstall --silent

# Deletar namespace
kubectl delete namespace flux-system

# Remover pasta do Git
rm -rf flux-system/
git add .
git commit -m "chore: remover FluxCD"
git push origin main
```

---

## 🎓 Conclusão da Aula 4

### O Que Aprendemos

✅ **Vídeo 4.1**: ArgoCD - Instalação e conceitos
✅ **Vídeo 4.2**: GitHub Actions + GitOps Pipeline
✅ **Vídeo 4.3**: FluxCD + Image Automation

### Conceitos Principais

1. **GitOps**: Git como fonte única da verdade
2. **Declarativo**: Descrever estado desejado, não passos
3. **Automação**: CI/CD totalmente automatizado
4. **Observabilidade**: Rastreamento de mudanças via Git

### Próximos Passos

- Escolher: ArgoCD ou FluxCD para seu projeto
- Implementar: Pipeline completo em produção
- Explorar: Helm charts com GitOps
- Avançar: Multi-cluster e multi-environment

---

**FIM DO VÍDEO 4.3** ✅

**FIM DA AULA 4 - GitOps** 🎓
