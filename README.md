# 🛍 Online Boutique - Kubernetes with ArgoCD

Este é um projeto com objetivo de colocar em prática meus conhecimentos sobre **Kubernetes e Gitops com ArgoCD**.

O arquivo nomeado como **online-boutique.yaml** é responsável por subir toda a insfraestrura no Kubernetes, o arquivo original está disponível no seguinte repositório:

> [GoogleCloudPlatform/microservices-demo](https://github.com/GoogleCloudPlatform/microservices-demo)

O arquivo pode ser localizado em **release/kubernetes_manifests.yaml**

## 🧰 Preparando o Ambiente

Para executar esse projeto será necessário um ambiente Kubernetes. Existem diversas formas de usar o Kubernetes pode ser criado um cluster local usando Minikube ou Rancher Desktop, ou até usar algum serviço de Cloud como EKS ou AKS. Para este projeto utilizei o Rancher Desktop que permite criar um Cluster local e oferece uma interface gráfica bem interessante. Abaixo deixo os links dessas ferramentas caso deseje conferir:

* [Minikube](https://minikube.sigs.k8s.io/docs/start/?arch=%2Flinux%2Fx86-64%2Fstable%2Fbinary+download)
* [Rancher Desktop](https://docs.rancherdesktop.io/getting-started/installation)
* [EKS](https://docs.aws.amazon.com/eks/latest/userguide/quickstart.html)
* [AKS](https://learn.microsoft.com/en-us/azure/aks/get-started-aks)

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

Na Seção "Destination" em Cluster URL coloque o seguinte:

```text
https://kubernetes.default.svc
```

No campo namespace coloque o nome do namespace criado anteriormente.

<img width="1443" height="330" alt="image" src="https://github.com/user-attachments/assets/e3871d7b-33ec-42d8-88ab-21b106b659b6" />

Então clique em create no topo da página, com isso obterá o seguinte resultado:

<img width="1857" height="326" alt="image" src="https://github.com/user-attachments/assets/a68b42c4-ea89-483e-a2ab-975ccd75de4c" />

Para que de fato o ArgoCD comece a criação dos recursos clique em "Sync" e depois em "Syncronize".

<img width="1861" height="342" alt="image" src="https://github.com/user-attachments/assets/4f1de4c6-99a0-4f75-b209-60a795ae305a" />

Com isso o ArgoCD vai automaticamente criar os recursos no namespace solicitado. Se tudo ocorrer como esperado vai aparecer a mensagem "Healthy" no campo APP HEALTH.

<img width="1861" height="342" alt="image" src="https://github.com/user-attachments/assets/f9d6ebe7-d817-455c-88a4-eaf5d7129a6e" />

## Visualizado página WEB da aplicação

Para visualizar a página será necessário realizar um port forwarding assim como foi foi para o ArgoCD, basta executar o seguinte comando:

```bash
kubectl port-forward -n online-boutique svc/frontend 3333:80
```

Com isso o site estará acessível em http://localhost:3333.

<img width="1853" height="970" alt="image" src="https://github.com/user-attachments/assets/e82d0a65-97d4-4814-86da-12935f518555" />

## Configurando Auto Sync

O Auto Sync faz com que o ArgoCD cheque periodicamente se houve novos commits no repositório alvo e automaticamente realizar os ajustes necessário no deploy caso necessário.
Para ativar basta ir na aba "Details" e na seção "Sync Policy" clicar em "Enable Auto Sync".

<img width="1425" height="183" alt="image" src="https://github.com/user-attachments/assets/239bda92-4259-448b-a159-0e434b135cad" />

## Testando Auto Sync
Realizar esse teste é bem simples basta alterar algo no online-boutique.yaml e realizar um commit, por exemplo pode pesquisar no arquivo pela palavra réplica no loadgenerator e alterar de 1 para 2.

Loadgenerator antes do commit:
<img width="1126" height="161" alt="image" src="https://github.com/user-attachments/assets/55c0c923-4776-4022-8a27-9dcceb58aad8" />

Alguns minutos depois:
<img width="1126" height="161" alt="image" src="https://github.com/user-attachments/assets/d7c4eaab-2356-4989-92f4-a03081dfb364" />


## Aplicando ArgoCD em repositórios privados

Em um caso real normalmente as aplicações não costumam ficar com o código fonte exposto ao público, geralmente isso fica protegido em um repositório privado. Então surge a dúvida como posso aplicar o uso do ArgoCD se meu repositório esta privado? Testei duas formas de resolver esse problema, segue a abaixo como aplicá-las:

### 1 - Criar um Github PAT
Github Pat permite criar um token de acesso temporário* a um repositório. Ele é bem interessante pois permite dar acesso apenas aos recursos necessários, podendo configurar até se é permitido somente leitura ou leitura e escrita, e ainda é bem simples de se usar uma vez gerado o token basta usá-lo em uma URL como a seguinte:

```text
https://<PAT>@github.com/<USUARIO>/<REPOSITORIO>.git
```

Montando a URL acima basta adicioná-la em um APP no ArgoCD e ele vai conseguir acessar o repositório normalmente.

Obs: É recomendavel que esse token tenha um tempo de expiração por motivos de segurança, mas é possível criar um que nunca expire.

### 2 - Configurar o acesso SSH
O ArgoCD permite adicionar repositórios privados com acesso via SSH, para isso acesse "Settings" e então clique em "Repositories", então clique em "+ CONNECT REPO".

Selecione a opção "Via SSH", e então preencha os campos solicitados inclusive qual a chave SSH para acessar o repositório.

<img width="1853" height="932" alt="image" src="https://github.com/user-attachments/assets/4fe51f69-60c7-4361-a7a9-1c2024ed8d5b" />

Caso não saiba como criar uma chave SSH para o github o link abaixo explica como fazer isso:

[https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent)
