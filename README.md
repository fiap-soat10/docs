# Sistema de Autoatendimento da Lanchonete

## Objetivo

Descrever a arquitetura do sistema de autoatendimento, que visa melhorar a eficiência do atendimento ao cliente e a
gestão de pedidos na lanchonete, com foco em escalabilidade, segurança e organização do código.
Como parte da fase 4 de desenvolvimento deste projeto, a arquitetura foi segregada em microservices.

![](assets/c4.png)

## Escopo

* Sistema de autoatendimento para clientes.
* Painel administrativo para a gestão de pedidos e produtos.
* Infraestrutura baseada em Kubernetes.
* Implementação de Clean Architecture nas APIs.
* Base de dados própria para cada container

## Componentes do sistema

* **Microservice de Identificação:**: Opção de identificação via CPF, nome ou não identificação.
* **Microservice de Carrinho:** Recebe a requisição do totem de autoatendimento contendo o combo escolhido pelo cliente, 
envia uma requisição para criação do pedido e envia um evento de pagamento para o microservice de pagamento.
* **Microservice de Produto:** Adição, edição e remoção de produtos do catálogo.
* **Microservice de Pedidos:** Operacionaliza a esteira do restaurante, com a criação de pedidos,
transição de seus status e acompanhamento do andamento. Interage com o microservice de produtos.
* **Microservice de Pagamento:** Processa o pagamento com o sistema externo do Mercado Pago e interage com o pedido para
atualizar seu status.

### Componentes auxiliares

Segregamos alguns serviços para melhor gerenciamento e buscando também: 
* **Separação de responsabilidades:** o foco da aplicação é a lógica do negócio, enquanto no projeto de infra o objetivo é 
provisionar e gerenciar recursos na nuvem.
* **Ciclos de vida diferentes:** possibilita atualizar a aplicação quantas vezes necessário sem ter que mudar a infra com a 
mesma frequência. A infra de um projeto é alterada com muito menos frequência.
* **Segurança e redução de possíveis impactos:** um bug na main do projeto poderia afetar a infraestrutura e derrubar um
ambiente inteiro. Dessa forma, isolamos os riscos.
* **Pipelines CI/CD simples e específicas:** os pipelines da aplicação e da infra tem propósitos diferentes. Tentar misturar 
as lógicas desses pipelines pode torná-la muito complexa, frágil e de difícil manutenção.
* **Reutilização de código:** uma estrutura de microservices é a que mais necessita de uma infra separada. Sem ela, cada 
services teria sua própria infra, e muito código seria repetido. Com a infra separada, todos os projetos e equipes podem
utilizar esse módulo, garantindo padronização e acelerando o provisionamento de novos serviços.

Nossos serviços auxiliares são
* **infra:** responsável por gerenciar os recursos em nuvem, como VPC, Security Groups, ECR, EKS e tabelas do AWS DynamoDB.
* **bd:** responsável por gerenciar a infraestrutura do banco de dados SQL
* **docs:** ponto focal de documentação, onde fica nosso README principal


## Requisitos funcionais

* O sistema deve permitir a personalização de pedidos.
* O sistema deve oferecer checkout do pedido com opção de pagamento através QRCode gerado no Mercado Pago.
* O sistema deve permitir acompanhamento em tempo real do status do pedido.
* O sistema deve oferecer opção para acompanhamento do status de pagamento do pedido.

## Requisitos Não Funcionais

* Separação da arquitetura em microservices com base dados própria.
* Necessário implementar pelo menos um exemplo de cada tipo de banco de dados (NoSQL e SQL).
* Deve conter testes unitários com cobertura mínima de 80%.
* Pelo menos uma operação de cada serviço deve ter testes BDD implementados.
* Branchs main devem ser protegidas, com pull requests validados (qualidade de código).
* Todos os repositórios precisam ter CI/CD executando corretamente para o deploy.

### Desempenho

* **Kubernetes:** Utilizar Kubernetes para orquestração de containers, permitindo a escalabilidade e gestão eficiente
dos serviços.

### Escalabilidade

* **HPA (Horizontal Pod Autoscaler):** Utilização do HPA para cada Deployment que utiliza métricas de CPU e memória para
escalar automaticamente o número de Pods baseando-se na demanda.

### Segurança

* **ConfigMap e Secrets:** Utilização de ConfigMaps para armazenar configurações não sensíveis e Secrets para dados
sensíveis (como credenciais de banco de dados e chaves de API), garantindo que informações confidenciais não sejam
expostas no código-fonte.

## Decisões de Design

Optamos por aplicar um pouco de cada tipo de arquitetura que aprendemos até aqui. 
A arquitetura dos microservices é baseada na Clean Architecture e também na Arquitetura Hexagonal.
Ambas nos permitiram criar sistemas robustos, escaláveis e de fácil manutenção.
> ⓘ Para mais detalhes, consulte o README e/ou o arquivo de docs/Arch Haiku de cada projeto.
Nesses arquivos você também encontrará detalhes sobre a organização de camadas de cada aplicação.

### Tecnologias utilizadas

* **Linguagem e Framework:** .NET 8 e Java 21 (Spring Boot 3), linguagens com versões LTS recentes.
* **Banco de dados:**
  * SQL: PostgreSQL, um banco open-source, que fornece escalabilidade, extensividade e segurança;
  * NoSQL: AWS DynamoDB, que possui alta escalabilidade e desempenho, que proporciona também uma
  flexibilidade perfeita para microservices por ser livre de esquema definido. O gerenciamento da AWS também
  elimina a necessidade de provisionamento de servidores e a criação de uma estrutura para segurança e escalonamento.
* **Docker e Docker Compose:** Empacota a aplicação e seus serviços dependentes em containers que podem ser orquestrados
localmente, garantindo um ambiente de desenvolvimento idêntico para todos os desenvolvedores e fornecendo portabilidade
(capacidade de uma imagem gerada ser executada em qualquer lugar que tenha suporte Docker).
* **Kubernetes:** Orquestra os containers em produção, gerenciando o deploy, a escalabilidade e a resiliência da
aplicação.
* **CI/CD:** Implementa o GitHub Actions, que automatiza os pipelines de CI/CD. Permite entregas mais rápidas e seguras,
pois possui etapas de verificação de qualidade (testes unitários, integração e análise de cobertura) antes de ser
integrado a base de código principal.
* **Teste e qualidade:** Utilizamos o LocalStack para instâncias containers Docker que simulam a
integração com a AWS.

