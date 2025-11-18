# 📚 Aula 04 - GitOps com ArgoCD e FluxCD

## 🎯 Objetivos

- Compreender os **princípios do GitOps** e diferenças entre push e pull model
- Instalar e configurar **ArgoCD** para deploy automático
- Instalar e configurar **FluxCD** como alternativa GitOps
- Implementar **pipeline CI/CD completo** com GitOps
- Comparar **ArgoCD vs FluxCD** e escolher a ferramenta adequada

## 📹 Vídeos

| Vídeo | Título | Temas | Tempo |
|-------|--------|-------|-------|
| 4.1 | Introdução ao GitOps e sua Filosofia | GitOps; ArgoCD; Continuous Deployment; Sync | 25 min |
| 4.2 | Pipeline GitOps Automatizado | CI/CD + GitOps; Update manifests; ArgoCD Sync; Automation | 20 min |
| 4.3 | FluxCD e Comparação | FluxCD; Image Automation; ArgoCD vs FluxCD; Escolha | 20 min |

**Tempo total**: 65 minutos

## 🚀 Como Usar

### 1. Fork e Clone

```bash
git clone https://github.com/SEU_USUARIO/fiap-dclt-aula04.git
cd fiap-dclt-aula04
```

### 2. Pré-requisitos

**⚠️ IMPORTANTE**: Esta aula depende de aulas anteriores:

#### 📦 Cluster EKS (Aula 01)
O cluster EKS `cicd-lab` deve ter sido criado na **Aula 01**.

Se você ainda não criou o cluster:
1. Volte ao **repositório da Aula 01**
2. Siga os passos de criação do cluster EKS
3. O cluster deve ter o nome: `cicd-lab`
4. Região: `us-east-1`
5. Profile AWS: `fiapaws`

**Verificar se o cluster existe:**
```bash
aws eks describe-cluster --name cicd-lab --region us-east-1 --profile fiapaws
```

#### 🐳 Aplicação Docker
A aplicação de exemplo (fiap-todo-api) **já está incluída** neste repositório:
- Código fonte em: `app/`
- Dockerfile na raiz do repositório
- Pronta para build e deploy

### 3. Seguir Vídeos em Ordem

- [VIDEO-4.1-PASSO-A-PASSO.md](VIDEO-4.1-PASSO-A-PASSO.md) - GitOps com ArgoCD
- [VIDEO-4.2-PASSO-A-PASSO.md](VIDEO-4.2-PASSO-A-PASSO.md) - Pipeline GitOps Automatizado
- [VIDEO-4.3-PASSO-A-PASSO.md](VIDEO-4.3-PASSO-A-PASSO.md) - FluxCD e Comparação

## 📁 Estrutura do Projeto

```
aula-04/
├── README.md                          # Este arquivo
├── Dockerfile                         # Dockerfile da aplicação
├── VIDEO-4.1-PASSO-A-PASSO.md         # Vídeo 1: GitOps com ArgoCD
├── VIDEO-4.2-PASSO-A-PASSO.md         # Vídeo 2: Pipeline GitOps
├── VIDEO-4.3-PASSO-A-PASSO.md         # Vídeo 3: FluxCD
├── app/                               # Código fonte da aplicação
│   ├── package.json                   # Dependências Node.js
│   └── src/                           # Código da aplicação
├── .github/workflows/                 # GitHub Actions
│   ├── docker-build.yml               # Build e push de imagens
│   ├── update-image.yml               # Update de manifests
│   └── argocd-sync.yml                # Sync com ArgoCD
└── gitops-repo/                       # Repositório GitOps
    ├── applications/                  # Definições de aplicações
    │   ├── fiap-todo-api/            # Manifests da aplicação
    │   │   ├── base/                 # Manifests base (comum)
    │   │   └── overlays/             # Overlays por ambiente
    │   └── fiap-todo-api-app.yaml    # ArgoCD Application
    └── clusters/                      # Configurações FluxCD
        └── production/                # Cluster de produção
```

## ✅ Checklist de Aprendizado

### Vídeo 4.1 - GitOps com ArgoCD
- [ ] Entender diferença entre **Push vs Pull model**
- [ ] Compreender **Git como source of truth**
- [ ] Instalar e configurar **ArgoCD**
- [ ] Criar **Application** no ArgoCD
- [ ] Testar **auto-sync** e **self-healing**
- [ ] Explorar **ArgoCD UI**

### Vídeo 4.2 - Pipeline GitOps Automatizado
- [ ] Integrar **CI/CD com GitOps**
- [ ] Criar workflow para **update de manifests**
- [ ] Implementar **sync automático** via ArgoCD
- [ ] Testar **fluxo end-to-end** completo
- [ ] Executar **rollback via Git**

### Vídeo 4.3 - FluxCD e Comparação
- [ ] Instalar e configurar **FluxCD**
- [ ] Entender **GitOps Toolkit** (componentes modulares)
- [ ] Configurar **Image Automation**
- [ ] Comparar **ArgoCD vs FluxCD**
- [ ] Escolher ferramenta adequada para cada cenário

## 🐛 Troubleshooting

### Erro: "ArgoCD pods não iniciam"
- **Causa**: Recursos insuficientes no cluster
- **Solução**: 
  ```bash
  # Verificar recursos do cluster
  kubectl top nodes
  
  # Aumentar nodes se necessário
  eksctl scale nodegroup --cluster=cicd-lab --nodes=3 workers --profile fiapaws
  ```

### Erro: "Application stuck in 'OutOfSync'"
- **Causa**: Manifests inválidos ou path incorreto no Git
- **Solução**:
  ```bash
  # Verificar logs do ArgoCD
  kubectl logs -n argocd deployment/argocd-application-controller
  
  # Validar manifests com kustomize
  cd gitops-repo/applications/fiap-todo-api/overlays/production
  kustomize build .
  ```

### Erro: "FluxCD não detecta mudanças no Git"
- **Causa**: Intervalo de polling muito longo ou credenciais inválidas
- **Solução**:
  ```bash
  # Forçar reconciliação
  flux reconcile source git fiap-todo-api
  
  # Verificar logs
  flux logs --follow
  ```

### Erro: "Self-healing não funciona"
- **Causa**: `selfHeal: false` ou `automated` não configurado
- **Solução**: Verificar `syncPolicy` no Application manifest
  ```yaml
  syncPolicy:
    automated:
      prune: true
      selfHeal: true  # Deve estar true
  ```

## 📚 Recursos Adicionais

- [ArgoCD Documentation](https://argo-cd.readthedocs.io/) - Documentação oficial completa
- [FluxCD Documentation](https://fluxcd.io/docs/) - Guia oficial do FluxCD
- [GitOps Principles](https://opengitops.dev/) - Princípios e boas práticas
- [Kustomize Tutorial](https://kubectl.docs.kubernetes.io/guides/introduction/kustomize/) - Gerenciamento de manifests

## ⚠️ Importante

### Custos AWS
- **EKS Cluster**: ~$0.10/hora ($2.40/dia)
- **EC2 Nodes (t3.medium x2)**: ~$0.08/hora ($1.92/dia)
- **Total estimado**: ~$4.32/dia para ambiente de testes
- **Créditos Learner Lab**: Geralmente $100 (suficiente para ~23 dias)

### Limpeza de Recursos
**IMPORTANTE**: Sempre deletar recursos após a aula para evitar custos!

```bash
# Deletar aplicações
kubectl delete -f gitops-repo/applications/fiap-todo-api-app.yaml

# Deletar ArgoCD
kubectl delete namespace argocd

# Deletar FluxCD
flux uninstall

# Deletar cluster EKS
eksctl delete cluster --name cicd-lab --region us-east-1 --profile fiapaws
```

### Secrets
- ❌ **NUNCA** commitar secrets no repositório Git
- ✅ Use **Sealed Secrets** ou **External Secrets Operator**
- ✅ Configure `.gitignore` para arquivos sensíveis

### Boas Práticas GitOps
- ✅ **Git como única fonte da verdade** - Todo estado no Git
- ✅ **Declarativo** - Descrever estado desejado, não comandos
- ✅ **Versionado** - Usar Git para versionamento e auditoria
- ✅ **Automatizado** - Sync automático, sem intervenção manual
- ✅ **Observável** - Monitorar estado e detectar drift

## 🔗 Links Relacionados

- **Aula 03**: CI/CD com GitHub Actions
- **Aula 05**: Infrastructure as Code com Terraform
- **Repositório Principal**: [FIAP DevOps & Cloud Tools](https://github.com/fiap-devops)
