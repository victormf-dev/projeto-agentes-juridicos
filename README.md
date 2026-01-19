# 🏛️ Infraestrutura Cloud - Agentes de IA Jurídicos

Este repositório documenta a configuração e deploy da infraestrutura para o projeto de Automação Jurídica e Agentes de IA (n8n + Chatwoot + Docker + integrações).

## 📋 Stack Tecnológica
- **Cloud Provider:** Microsoft Azure
- **OS:** Ubuntu Server 24.04 LTS
- **Containerization:** Docker & Docker Compose
- **Orquestração:** Portainer CE
- **Banco de Dados:** Postgres (com extensão pgvector)
- **Cache & Filas:** Redis
- **Gateway:** Nginx Proxy Manager

## 🚀 Passo a Passo da Configuração Inicial

### 1. Provisionamento da VM (Azure)
Foi utilizada a máquina **Standard B2als v2** (2 vCPUs, 4GB RAM) na região **Canada Central** para otimização de custos e performance.
- **Rede:** Configuração de IP Público Estático reservado na Azure para garantir estabilidade nos apontamentos DNS.

### 2. Acesso ao Servidor (SSH)
O acesso é feito via terminal seguro utilizando chaves RSA:

    ssh -i nome_da_chave.pem azureuser@IP_PUBLICO

### 3. Preparação do Sistema (Linux)
Atualização dos pacotes do Ubuntu e instalação de dependências básicas:

    sudo apt update && sudo apt upgrade -y

### 4. Instalação do Docker Engine
Utilizado o script oficial de instalação para garantir a versão mais recente e compatível:

    # 1. Baixar e executar o script de instalação
    curl -fsSL https://get.docker.com -o get-docker.sh
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

**Comando de Execução:**

    docker run -d -p 8000:8000 -p 9000:9000 --name portainer \
    --restart=always \
    -v /var/run/docker.sock:/var/run/docker.sock \
    -v portainer_data:/data \
    portainer/portainer-ce:latest  
    

### 7. Implementação do n8n (Automação)
Deploy da stack de automação utilizando Docker Compose:
- **n8n:** Motor de automação para os Agentes de IA.
- **Postgres:** Banco de dados relacional para persistência de fluxos e dados.

**Comando de Execução**

    version: '3.8'

    services:
      postgres:
        image: postgres:16-alpine
        restart: always
        environment:
          - POSTGRES_USER=n8n_user
          - POSTGRES_PASSWORD=SUA_SENHA_AQUI
          - POSTGRES_DB=n8n_db
        volumes:
          - n8n_db_data:/var/lib/postgresql/data
        networks:
          - infra-public

      n8n:
        image: n8nio/n8n:latest
        restart: always
        environment:
          - DB_TYPE=postgresdb
          - DB_POSTGRESDB_HOST=postgres
          - DB_POSTGRESDB_DATABASE=n8n_db
          - DB_POSTGRESDB_USER=n8n_user
          - DB_POSTGRESDB_PASSWORD=SUA_SENHA_AQUI
          - N8N_HOST=n8n.seudominio.com.br
          - WEBHOOK_URL=[https://n8n.seudominio.com.br/](https://n8n.seudominio.com.br/)
        networks:
          - infra-public
        volumes:
          - n8n_data:/home/node/.n8n

    networks:
      infra-public:
        external: true      

### 8. Implementação do Chatwoot (Omnichannel)
Deploy da plataforma de atendimento utilizando uma arquitetura de microsserviços robusta:
- **Chatwoot Web & Worker:** Separação entre front-end e processamento de tarefas em background.
- **Redis:** Implementado para gerenciamento de cache rápido e filas de mensagens (Sidekiq), garantindo alta performance.
- **Postgres (pgvector):** Banco de dados configurado com a extensão `pgvector`, preparando a infraestrutura para futuras implementações de RAG (Busca Vetorial).

**Comando de Execução**

    version: '3'

    services:
      base: &base
        image: chatwoot/chatwoot:latest
        env_file: .env
        volumes:
          - /data/storage:/app/storage
        networks:
          - infra-public

      web:
        <<: *base
        command: bundle exec rails s -p 3000 -b 0.0.0.0
        ports:
          - "3000:3000"
        depends_on:
          - postgres
          - redis

      worker:
        <<: *base
        command: bundle exec sidekiq -C config/sidekiq.yml
        depends_on:
          - postgres
          - redis

      postgres:
        image: pgvector/pgvector:pg16
        restart: always
        environment:
          - POSTGRES_PASSWORD=SUA_SENHA_DB
          - POSTGRES_USER=chatwoot
          - POSTGRES_DB=chatwoot_production
        volumes:
          - postgres_data:/var/lib/postgresql/data
        networks:
          - infra-public

      redis:
        image: redis:alpine
        restart: always
        volumes:
          - redis_data:/data
        networks:
          - infra-public

    networks:
      infra-public:
        external: true    

### 9. Segurança e Proxy Reverso (HTTPS)
Configuração do **Nginx Proxy Manager** como único ponto de entrada do servidor:

**Comando de Execução**

    version: '3.8'

    services:
      app:
        image: 'jc21/nginx-proxy-manager:latest'
        restart: unless-stopped
        ports:
          - '80:80'   # HTTP Public
          - '443:443' # HTTPS Public
          - '81:81'   # Admin Panel
        volumes:
          - ./data:/data
          - ./letsencrypt:/etc/letsencrypt
        networks:
          - infra-public

    networks:
      infra-public:
        external: true
        
**Configurações Realizadas**

- Criação de rede interna isolada (`infra-public`) para comunicação segura entre containers.
- Emissão de certificados SSL via Let's Encrypt para os subdomínios:
  - `n8n.seudominio.com.br`
  - `chat.seudominio.com.br`
  - `painel.seudominio.com.br`
  - `proxy.seudominio.com.br`

### 10. Blindagem de Rede (Hardening)
Aplicação do princípio de privilégio mínimo no **Network Security Group (NSG)** da Azure:
- **Portas Públicas:** 22 (SSH), 80 (HTTP) e 443 (HTTPS) para tráfego web seguro.
- **Isolamento de Aplicações:** Todas as portas de gerenciamento e serviços (n8n, Chatwoot, Portainer, Banco de Dados) foram restritas no firewall, tornando-as inacessíveis externamente.
Toda a comunicação externa é obrigatoriamente mediada pelo Proxy Reverso com criptografia SSL, eliminando vetores de ataque diretos aos serviços.

------


## 🛠️ Desafios Superados e Otimizações (Troubleshooting)

Durante o processo de infraestrutura e o deploy, enfrentamos e solucionamos desafios técnicos críticos:

### 🔴 OOM Kill (Gerenciamento de Memória)
**O Problema:** A VM de 4GB de RAM encerrava processos (crash) ao tentar subir as stacks do Chatwoot e n8n simultaneamente devido ao alto consumo do **Ruby (Chatwoot) e Node.js**.
**A Solução:** Criação de um arquivo de **Swap de 4GB**, totalizando 8GB de memória disponível (4GB Física + 4GB Virtual) para evitar o estouro de memória.

### Comandos executados
    sudo fallocate -l 4G /swapfile
    sudo chmod 600 /swapfile
    sudo mkswap /swapfile
    sudo swapon /swapfile
### Configuração persistida no /etc/fstab


### 🔴 Persistência de Dados e Autenticação (Gerenciamento de Memória)
**O Problema:** Erros recorrentes de password authentication failed no n8n e no banco de dados. **A Causa:** O Docker preserva volumes antigos mesmo quando o docker-compose.yml é alterado. A senha antiga permanecia gravada no volume do Postgres.
**A Solução:** Limpeza profunda de volumes orfãos e recriação das stacks garantindo nomes de serviços únicos e volumes novos.

### Comandos executados
    docker volume prune -f
    docker volume rm nome_do_volume_antigo




## 📝 Próximos Passos
- [ ] Implementação de lógica de RAG (Busca em documentos jurídicos).
- [ ] Desenho das regras de negócio para triagem de leads.

---
*Projeto desenvolvido como parte do portfólio de Engenharia de Software / ADS.*
