# Sistema de Monitoramento e Alerta de Estoque com Microsserviços e Kafka

Este projeto é uma implementação de um sistema de monitoramento de estoque utilizando uma arquitetura de microsserviços com coreografia baseada em eventos. A comunicação entre os serviços é realizada exclusivamente através do Apache Kafka, garantindo desacoplamento e escalabilidade.

## Arquitetura

O sistema é composto por três microsserviços independentes, cada um executando em seu próprio container Docker e conectado a uma rede isolada.

1.  **Data Ingestion Service**: Simula a entrada de dados de estoque (vendas e reposições) e publica esses eventos no tópico `stock-updates` do Kafka.
2.  **Monitoring Service**: Consome os eventos de `stock-updates`, mantém o estado do estoque de cada produto e, ao detectar um nível crítico (estoque < 10), publica um alerta no tópico `stock-alerts`.
3.  **Notification Service**: Consome os alertas do tópico `stock-alerts` e simula o envio de uma notificação (exibindo um log detalhado no console).

Cada serviço possui um painel web em tempo real para visualização dos eventos que processa.

---

## Pré-requisitos

Para executar este projeto, você precisará ter instalados:

-   **Docker**: [https://www.docker.com/get-started](https://www.docker.com/get-started)
-   **Docker Compose**: Geralmente já vem incluído na instalação do Docker Desktop.

---

## Como Executar o Sistema

Siga os passos abaixo para iniciar a aplicação completa.

### 1. Construir e Iniciar os Serviços

O projeto utiliza uma imagem Docker base para compartilhar dependências e acelerar o processo de build. O `docker-compose` gerencia a construção dessa imagem automaticamente.

Para construir a imagem base e iniciar todos os serviços, abra um terminal na raiz do projeto e execute:

```bash
docker-compose up --build
```

-   **Como funciona o `--build`?**
    1.  Primeiro, o Docker construirá a imagem `estoque-ddi-base:latest` a partir do diretório `docker-base-image`, contendo todas as dependências compartilhadas. Essa camada será cacheada.
    2.  Em seguida, as imagens de cada microsserviço (`data-ingestion`, `monitoring`, `notification`) serão construídas sobre a imagem base.
-   O processo pode levar alguns minutos na primeira vez.
-   Aguarde até que os logs indiquem que os serviços se conectaram com sucesso ao Kafka.

### 2. Acessar os Painéis de Monitoramento

Após iniciar os containers, você pode acessar os painéis web de cada serviço em seu navegador:

-   **Painel de Ingestão de Dados**: [http://localhost:3001/panel](http://localhost:3001/panel)
    -   *Mostra os eventos de venda e reposição que estão sendo gerados e enviados ao Kafka.*

-   **Painel de Monitoramento**: [http://localhost:3002/panel](http://localhost:3002/panel)
    -   *Mostra tanto os eventos de estoque que está consumindo quanto os alertas que está gerando.*

-   **Painel de Notificação**: [http://localhost:3003/panel](http://localhost:3003/panel)
    -   *Mostra os alertas de estoque crítico recebidos do serviço de monitoramento.*

Os painéis se atualizam automaticamente a cada 2 segundos.

### 3. Acessar os Painéis via Terminal

Como alternativa aos painéis web, você pode visualizar os eventos de cada serviço diretamente no seu terminal. Para isso, execute os seguintes comandos em novos terminais:

-   **Painel de Ingestão de Dados:**
    ```bash
    docker-compose exec data-ingestion-service python src/panel.py
    ```

-   **Painel de Monitoramento:**
    ```bash
    docker-compose exec monitoring-service python src/panel.py
    ```

-   **Painel de Notificação:**
    ```bash
    docker-compose exec notification-service python src/panel.py
    ```

### 4. Verificar os Logs dos Serviços

Para acompanhar os logs de um serviço específico em tempo real, abra um novo terminal e use o comando `docker-compose logs -f`.

-   **Ver logs do serviço de ingestão:**
    ```bash
    docker-compose logs -f data-ingestion-service
    ```

-   **Ver logs do serviço de monitoramento:**
    ```bash
    docker-compose logs -f monitoring-service
    ```

-   **Ver logs do serviço de notificação (onde as "notificações" são impressas):**
    ```bash
    docker-compose logs -f notification-service
    ```

### 4. Parar a Aplicação

Para parar todos os containers, volte ao terminal onde o `docker-compose up` está executando e pressione `Ctrl + C`. Depois, para garantir que os containers e redes sejam removidos, execute:

```bash
docker-compose down
```

### 5. Limpar Volumes (Opcional)

Se você desejar remover completamente os dados, incluindo os volumes do Kafka (o que limpará todos os tópicos e mensagens), use o comando:

```bash
docker-compose down -v
```

---

## Estrutura do Projeto

```
/
├── docker-compose.yml
├── docker-base-image/
│   ├── Dockerfile
│   └── requirements.txt
├── data-ingestion-service/
│   ├── Dockerfile
│   └── src/
│       ├── service.py
│       ├── panel.py
│       └── templates/
│           └── panel.html
├── monitoring-service/
│   ├── ... (estrutura similar)
└── notification-service/
    └── ... (estrutura similar)
```
