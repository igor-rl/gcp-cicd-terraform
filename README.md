<!-- badges https://github.com/Ileriayo/markdown-badges -->
<!-- icons https://devicon.dev/ -->

![banner](https://github.com/igor-rl/assets/blob/main/img/github-projetcs-header.jpg)

<div class="center">

  # GKE - GIT HUB ACTIONS - TERRAFORM
  Um projeto com pipe line CI-CD para implantar projetos na Google Cloud Platform automatizado com Terraform.
  
  
  <p>

  ![Google Cloud](https://img.shields.io/badge/GoogleCloud-%234285F4.svg?style=for-the-badge&logo=google-cloud&logoColor=white)
  ![Terraform](https://img.shields.io/badge/terraform-%235835CC.svg?style=for-the-badge&logo=terraform&logoColor=white)
  ![kubernets](https://img.shields.io/badge/kubernetes-326ce5.svg?&style=for-the-badge&logo=kubernetes&logoColor=white)
  ![GitHub](https://img.shields.io/badge/github-%23121011.svg?style=for-the-badge&logo=github&logoColor=white)
  ![GitHub Actions](https://img.shields.io/badge/github%20actions-%232671E5.svg?style=for-the-badge&logo=githubactions&logoColor=white)
  <!-- ![Docker](https://img.shields.io/badge/docker-%230db7ed.svg?style=for-the-badge&logo=docker&logoColor=white)
  [Postgres](https://img.shields.io/badge/postgres-%23316192.svg?style=for-the-badge&logo=postgresql&logoColor=white)
  ![NodeJS](https://img.shields.io/badge/node.js-6DA55F?style=for-the-badge&logo=node.js&logoColor=white)
  ![NestJS](https://img.shields.io/badge/nestjs-%23E0234E.svg?style=for-the-badge&logo=nestjs&logoColor=white)
  ![NextJS](https://img.shields.io/badge/Next-black?style=for-the-badge&logo=next.js&logoColor=white)
  ![JavaScript](https://img.shields.io/badge/javascript-%23323330.svg?style=for-the-badge&logo=javascript&logoColor=%23F7DF1E)
  ![TypeScript](https://img.shields.io/badge/typescript-%23007ACC.svg?style=for-the-badge&logo=typescript&logoColor=white) -->
  </p>
</div>


<br>


<br>


## Índice
- [Sobre o projeto](#sobre-o-projeto)
- [1. Pré-requisitos](#1-pré-requisitos)
- [2. Criando o Cluster no GCP](#2-criando-o-cluster-no-gcp)
- [3. Cert-Manager](#3-cert-manager)
- [4. Ingress e Cluster Issuer](#4-ingress-e-cluster-issuer)
- [5. Pipeline CI-CD](#5-pipeline-ci-cd)


<br>


## Sobre o projeto
Esse projeto foi desenvolvido como base de estudo e aplicação prática voltada para produção. Neste projeto vamos (1) criar uma conta e um novo projeto no GCP, (2) criar um cluster kubernetes GKE usando o Terraform, (3) criar certificados tls e manipular o DNS e (4) criar um balanceador de carga para nossos serviços kubernetes, (5) implantar nosso projeto automaticatimente usando 'Git Actons'.

## 1. Pré-requisitos:

- Instale o [Terraform](https://developer.hashicorp.com/terraform/downloads) e inicialize-o em seu ambiente.
- Instale a [CLI gcloud](https://cloud.google.com/sdk/docs/install?hl=pt-br) e configure-o em sua máquina.
- [Construa a infraestrutura](https://developer.hashicorp.com/terraform/tutorials/gcp-get-started/google-cloud-platform-build) do projeto no GCP.
- Certifique-se de que você está está logado no gcloud CLI e conectado com seu cluster GKE.
```
gcloud init
gcloud auth login
gcloud container clusters get-credentials NOME_DO_CLUSTER --project=SEU_PROJETO --zone=SUA_ZONA
```

<br>


## 2. Criando o Cluster no GCP
Com o cluster e o projeto criados e selecionados no seu terminal, crie o novo cluster de forma automatizada usando Terraform:

```
cd gcp/terraform
terraform init
terraform apply
```
Digite "yes" para prosseguir com a criação do seu cluster.
Obs.: Isso pode demorar um pouco.

<br>

## 3. Cert-Manager
- Siga os passo do [guia de instalação do cert-manager usando o kubectl](https://cert-manager.io/docs/installation/kubectl/).

- Aguarde até que todos os pods do cert-manager estejam com o status "Running".
```
kubectl get pods --namespace cert-manager --watch
```

<br>

## 4. Ingress e Cluster Issuer
- Siga os passo do [guia de instalação do ingress no GCE-GKE](https://kubernetes.github.io/ingress-nginx/deploy/#gce-gke).

- Para veririfcar o estado dos serviços do ingress, execute o comando:
```
kubectl get service ingress-nginx-controller --namespace=ingress-nginx
```
- Insira o ip do ingress-nginx-controller no ipv4 do seu seu domínio. Caso precise de um gerenciador de domínios e DNS, utilize o [Cloudflare](https://www.cloudflare.com/pt-br/).

- Aplique o manifesto do cluster-issuer e ingress:
```
kubectl apply -f gcp/k8s/ingress/cluster-issuer.yaml
kubectl apply -f gcp/k8s/ingress/ingress.yaml
```
- Recuperar certificados e aguardar status = true:
```
kubectl get certificates --watch
```
- Detalhar certificado:
```
kubectl describe certificate letsencrypt-tls
```
<br>


## 5. Pipeline CI-CD
As pipe lines de CI-CD serão acionadas automaticamente quando um novo commit na branch 'master' dos respectivos projetos são enviados. Esse gatilho inicia um novo build da imagem docker no projeto e faz o apply no GKE com a nova versão do projeto.
