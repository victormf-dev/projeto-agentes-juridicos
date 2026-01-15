# 🏛️ Infraestrutura Cloud - Agentes de IA Jurídicos

Este repositório documenta a configuração e deploy da infraestrutura para o projeto de Automação Jurídica e Agentes de IA (n8n + Docker + integrações).

## 📋 Stack Tecnológica
- **Cloud Provider:** Microsoft Azure
- **OS:** Ubuntu Server 24.04 LTS
- **Containerization:** Docker & Docker Compose
- **Gestão:** Portainer

## 🚀 Passo a Passo da Configuração Inicial

### 1. Provisionamento da VM (Azure)
Foi utilizada a máquina **Standard B2als v2** (2 vCPUs, 4GB RAM) na região **Canada Central** para otimização de custos e performance.

### 2. Acesso ao Servidor (SSH)
O acesso é feito via terminal seguro utilizando chaves RSA:

    ssh -i nome_da_chave.pem azureuser-vm@IP_PUBLICO

### 3. Preparação do Sistema (Linux)
Atualização dos pacotes do Ubuntu e instalação de dependências básicas:

    sudo apt update && sudo apt upgrade -y

### 4. Instalação do Docker Engine
Utilizado o script oficial de instalação para garantir a versão mais recente e compatível:

    # 1. Baixar e executar o script de instalação
    curl -fsSL [https://get.docker.com](https://get.docker.com) -o get-docker.sh
    sudo sh get-docker.sh

    # 2. Configuração de permissões (Para não precisar usar sudo)
    sudo usermod -aG docker $USER
    newgrp docker

### 5. Validação
Confirmação que o Docker está rodando corretamente:

    docker version

## 📝 Próximos Passos
- [ ] Instalação do Portainer (Interface Visual)
- [ ] Criação do Docker Compose (n8n + Chatwoot + Postgres)
- [ ] Configuração de Proxy Reverso (HTTPS/SSL)

---
*Projeto desenvolvido como parte do portfólio de Engenharia de Software / ADS.*
