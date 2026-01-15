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

    ssh -i nome_da_chave.pem azureuser@IP_PUBLICO

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

### 6. Gestão Visual com Portainer
Instalação do Portainer Community Edition para gerenciamento visual de containers e volumes.
- Mapeamento de portas internas seguindo padrões de segurança.
- Persistência de dados configurada via volumes Docker para segurança das stacks.

### 7. Implementação do n8n & Postgres
Deploy da stack de automação utilizando Docker Compose:
- **n8n:** Motor de automação para os Agentes de IA.
- **Postgres:** Banco de dados relacional para persistência de fluxos e dados.

### 8. Segurança e Proxy Reverso (HTTPS)
Configuração do **Nginx Proxy Manager** como único ponto de entrada do servidor:
- Criação de rede interna isolada (`infra-public`) para comunicação segura entre containers.
- Emissão de certificados SSL via Let's Encrypt para os subdomínios:
  - `n8n.seudominio.com.br`
  - `painel.seudominio.com.br`
  - `proxy.seudominio.com.br`

### 9. Blindagem de Rede (Hardening)
Aplicação do princípio de privilégio mínimo no **Network Security Group (NSG)** da Azure:
- **Portas Públicas:** 22 (SSH), 80 (HTTP) e 443 (HTTPS) para tráfego web seguro.
- **Isolamento de Aplicações:** Todas as portas de gerenciamento e serviços (n8n, Portainer, Proxy Dashboard) foram restritas no firewall, tornando-as inacessíveis externamente.
Toda a comunicação externa é obrigatoriamente mediada pelo Proxy Reverso com criptografia SSL, eliminando vetores de ataque diretos aos serviços.

## 📝 Próximos Passos
- [ ] Configuração de IP Estático na Azure.
- [ ] Instalação do Chatwoot (Omnichannel) e Redis.
- [ ] Implementação de lógica de RAG (Busca em documentos jurídicos).
- [ ] Desenho das regras de negócio para triagem de leads.

---
*Projeto desenvolvido como parte do portfólio de Engenharia de Software / ADS.*
