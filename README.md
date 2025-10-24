# 🛍 Online Boutique - Kubernetes with ArgoCD

Este é um projeto com objetivo de colocar em prática meus conhecimentos sobre **Kubernetes e Gitops com ArgoCD**.

O arquivo nomeado como **online-boutique.yaml** é responsável por subir toda a insfraestrura no Kubernetes, o arquivo original está disponível no seguinte repositório:

> [GoogleCloudPlatform/microservices-demo](https://github.com/GoogleCloudPlatform/microservices-demo)

O arquivo pode ser localizado em **release/kubernetes_manifests.yaml**

## 🧰 Preparando o Ambiente

A princípio este projeto será executado de forma local, então será necessário instalar alguma ferramenta para trabalhar com o Kubernetes, por exemplo minikube, Docker Desktop ou Rancher Desktop.

## Argo CD Cluster

O Argo CD é uma ferramenta de GitOps que permite que o cluster Kubernetes esteja sincronizado com o Github realizando Deploy automatico quando novos commits forem efetuados e validados.

A instalação será realizada dentro de um cluster no Kubernetes, então o primeiro passo será criar um namespace para o argoCD com o seguinte comando.

```bash
kubectl create namespace argocd
```

Com o namespace criado o seguinte comando vai instalar o ArgoCD dentro do **namespace argoCD**.

```bash
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

Pronto com isso o ArgoCD já vai estar instalado, podemos então realizar um Port Forwarding para acessar o dashboard pelo navegador na porta 8888, basta executar o seguinte comando:

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

### Dashboard do ArgoCD (Tela de Login)

<img width="1867" height="974" alt="image" src="https://github.com/user-attachments/assets/abf3f515-d2a9-437f-b68b-8e5d7daa649c" />

### Como realizar no Login no Argo CD

Por padrão o ArgoCD gerá um senha para o usuário admin na hora da criação, para recuperar essa senha execute o seguinte comando:

```bash
argocd admin initial-password -n argocd
```

Daí basta colocar o usuário **admin** e colocar a senha para acessar o dashboard.

<img width="1867" height="974" alt="image" src="https://github.com/user-attachments/assets/4b16f0ca-6550-40d8-9473-8d440ac1c62b" />

### Criando um APP no ArgoCD

Para que o ArgoCD comece de fato a obsvervar um repositório git e realizar o deploy automatico devemos criar um **app** dentro do ArgoCD.

Primeiro crie um namespace para o App, com o comando abaixo:

```bash
kubectl create namespace online-boutique
```

Então acesse o Dashboard do ArgoCD e clique em "+ NEW APP".

<img width="1032" height="112" alt="image" src="https://github.com/user-attachments/assets/47356491-9d9a-4508-8fcf-690a6586fcf4" />

Na seção "General" dê um nome ao App em ApplicationName, por exemplo "Online-Boutique", em Project marque a opção "default" e em SyncPolice deixe "Manual", as demais opções não precisa mudar.

Na seção "Source" cole a URL do repositório adicionado .git no final do link.

```text
https://github.com/DellGarcia/gitops-kubernetes-microservices/
```

Em Revision selecione **HEAD** e em Path coloque "k8s".

<img width="1447" height="449" alt="image" src="https://github.com/user-attachments/assets/a3e9c750-5e84-41c0-b019-b21dd57c49b9" />

Na Seção "Destiantion" em Cluster URL coloque o seguinte:

```text
https://kubernetes.default.svc
```

No campo namespace coloque o nome do namespace criado anteriormente.

<img width="1443" height="330" alt="image" src="https://github.com/user-attachments/assets/e3871d7b-33ec-42d8-88ab-21b106b659b6" />

