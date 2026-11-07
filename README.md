# NavegaAM — Sistema de Venda de Passagens Fluviais

> **Trabalho em Equipe — Requisitos e Arquitetura de um Sistema**
> Disciplina: Projetos de Sistemas · Centro Universitário de Ensino Superior do Amazonas (CIESA)

| Papel | Integrante |
|---|---|
| Analista de requisitos | _(nome)_ |
| Analista de requisitos | _(nome)_ |
| Arquiteto (diagramas C4) | _(nome)_ |
| Responsável por decisões e comunicação (ADRs, HTTP, HTTPS) | _(nome)_ |
| Revisor de rastreabilidade | _(nome)_ |

## Sumário

- [Parte 1 — Requisitos](#parte-1--requisitos)
  - [1.1 Descrição do sistema](#11-descrição-do-sistema)
  - [1.2 Requisitos funcionais (RF)](#12-requisitos-funcionais-rf)
  - [1.3 Requisitos não funcionais (RNF)](#13-requisitos-não-funcionais-rnf)
  - [1.4 Regras de negócio (RN)](#14-regras-de-negócio-rn)
  - [1.5 Resumo das quantidades](#15-resumo-das-quantidades)
- [Parte 2 — Documento de arquitetura](#parte-2--documento-de-arquitetura)
  - [2.1 Diagramas C4 (níveis 1 a 3)](#21-diagramas-c4-níveis-1-a-3)
  - [2.2 Registros de decisão (ADRs)](#22-registros-de-decisão-adrs)
  - [2.3 Comunicação cliente-servidor](#23-comunicação-cliente-servidor)
  - [2.4 Rastreabilidade](#24-rastreabilidade)

---

# Parte 1 — Requisitos

## 1.1 Descrição do sistema

O **NavegaAM** é o sistema de venda de passagens da **Expresso Rio Negro Navegação** (empresa fictícia), que opera lanchas expressas e barcos regionais saindo do Porto de Manaus para municípios do interior do Amazonas, como Manacapuru, Itacoatiara, Parintins, Coari e Tefé. Hoje as passagens são vendidas apenas nos guichês do porto, com controle de lotação feito em planilhas e listas de papel, o que causa filas nas vésperas de feriados e do Festival de Parintins, risco de vender acima da lotação e dificuldade para quem mora no interior comprar a passagem de volta. O NavegaAM permite comprar passagens pela internet, vender no guichê, validar o embarque por QR Code e gerenciar embarcações, viagens, preços e relatórios. O sistema tem quatro tipos de usuário:

- **Passageiro** — pessoa que procura viagens, compra, consulta e cancela passagens pelo celular ou computador.
- **Atendente de guichê** — funcionário que vende passagens no porto, inclusive para quem não tem conta ou paga em dinheiro.
- **Fiscal de embarque** — funcionário que confere os bilhetes na rampa de embarque lendo o QR Code com o celular.
- **Administrador** — gerente que cadastra embarcações e viagens, define preços e acompanha vendas e ocupação.

## 1.2 Requisitos funcionais (RF)

| Código | Tipo | Descrição | Prioridade |
|---|---|---|---|
| RF01 | Funcional | O sistema deve permitir que o passageiro crie uma conta informando nome, CPF, data de nascimento, e-mail, telefone e senha. | Alta |
| RF02 | Funcional | O sistema deve permitir que o usuário (passageiro, atendente, fiscal ou administrador) entre no sistema com e-mail e senha. | Alta |
| RF03 | Funcional | O sistema deve permitir que o passageiro consulte as viagens disponíveis informando origem, destino e data. | Alta |
| RF04 | Funcional | O sistema deve permitir que o passageiro escolha o tipo de acomodação (poltrona, rede ou camarote) em uma viagem com vagas. | Alta |
| RF05 | Funcional | O sistema deve permitir que o passageiro compre uma passagem pagando com Pix ou cartão de crédito. | Alta |
| RF06 | Funcional | O sistema deve permitir que o passageiro receba o bilhete eletrônico com QR Code por e-mail e SMS após a confirmação do pagamento. | Alta |
| RF07 | Funcional | O sistema deve permitir que o passageiro cancele uma passagem e solicite o reembolso. | Média |
| RF08 | Funcional | O sistema deve permitir que o passageiro consulte o histórico das suas passagens. | Baixa |
| RF09 | Funcional | O sistema deve permitir que o atendente venda uma passagem no guichê, com pagamento em dinheiro, Pix ou cartão, para um passageiro sem conta. | Alta |
| RF10 | Funcional | O sistema deve permitir que o fiscal de embarque valide o bilhete lendo o QR Code com a câmera do celular. | Alta |
| RF11 | Funcional | O sistema deve permitir que o administrador cadastre embarcações com nome, inscrição na Capitania dos Portos e capacidade por tipo de acomodação. | Média |
| RF12 | Funcional | O sistema deve permitir que o administrador cadastre e altere viagens (embarcação, rota, data, horário de partida, chegada prevista e preços). | Alta |
| RF13 | Funcional | O sistema deve permitir que o administrador emita relatórios de vendas e de ocupação por viagem e por período. | Média |

## 1.3 Requisitos não funcionais (RNF)

Todos os RNF têm uma medida verificável (tempo, porcentagem, quantidade ou norma).

| Código | Tipo | Categoria | Descrição | Prioridade |
|---|---|---|---|---|
| RNF01 | Não funcional | Desempenho | A consulta de viagens (RF03) deve responder em até **2 segundos** no percentil 95, com até **500 usuários simultâneos**. | Alta |
| RNF02 | Não funcional | Desempenho | A reserva com geração da cobrança (RF04/RF05) deve ser concluída em até **3 segundos** no percentil 95 no horário de pico (sextas-feiras, das 16h às 20h). | Alta |
| RNF03 | Não funcional | Desempenho | A primeira carga da aplicação do passageiro deve transferir no máximo **500 KB** comprimidos e ficar utilizável em até **5 segundos** em conexão 3G (1,6 Mbps), comum no interior do estado. | Média |
| RNF04 | Não funcional | Segurança | Toda comunicação entre clientes, servidor e serviços externos deve usar **HTTPS com TLS 1.2 ou superior**; requisições em HTTP devem ser redirecionadas para HTTPS (HSTS de **1 ano**). | Alta |
| RNF05 | Não funcional | Segurança | Senhas devem ser armazenadas com **bcrypt de custo ≥ 12**, e **5 tentativas** de login erradas em **15 minutos** devem bloquear a conta por **15 minutos**. | Alta |
| RNF06 | Não funcional | Segurança | A credencial de acesso deve expirar em no máximo **15 minutos**; a sessão deve exigir novo login após **7 dias** (passageiro) ou **12 horas** (atendente, fiscal e administrador). | Alta |
| RNF07 | Não funcional | Segurança | Os dados pessoais devem ser tratados conforme a **LGPD (Lei 13.709/2018)**: CPF e número de documento cifrados em repouso com **AES-256**, e pedidos de exclusão de dados atendidos em até **15 dias**. | Alta |
| RNF08 | Não funcional | Disponibilidade | O sistema deve ter disponibilidade mínima de **99,5% ao mês** (no máximo 3 h 39 min fora do ar por mês), medida por monitor externo a cada 1 minuto. | Alta |
| RNF09 | Não funcional | Disponibilidade | Em caso de falha do banco de dados, a perda máxima de dados deve ser de **5 minutos (RPO)** e o serviço deve ser restaurado em até **1 hora (RTO)**. | Média |
| RNF10 | Não funcional | Acessibilidade | A aplicação do passageiro deve atender à **WCAG 2.1 nível AA**, com **zero violações** graves ou críticas na verificação automática com axe-core. | Média |
| RNF11 | Não funcional | Usabilidade | Em teste com no mínimo **10 pessoas**, **90%** dos passageiros devem concluir a primeira compra em até **3 minutos**, sem ajuda. | Média |
| RNF12 | Não funcional | Compatibilidade | O sistema deve funcionar nas **2 últimas versões** de Chrome, Firefox, Safari e Edge, em celulares com **Android 9+** e **iOS 15+**, e em telas a partir de **360 px** de largura. | Alta |

Categorias cobertas: desempenho, segurança, disponibilidade, usabilidade/acessibilidade e compatibilidade (5 de 5).

## 1.4 Regras de negócio (RN)

As regras abaixo valeriam mesmo se a empresa continuasse vendendo passagens só no papel.

| Código | Tipo | Descrição | Prioridade |
|---|---|---|---|
| RN01 | Regra de negócio | Criança de até 5 anos completos não paga passagem, desde que viaje no colo do responsável, sem ocupar poltrona, rede ou camarote próprio. | Média |
| RN02 | Regra de negócio | Menor de 16 anos só viaja sem os pais ou o responsável legal se apresentar autorização escrita com firma reconhecida (ECA, art. 83). | Alta |
| RN03 | Regra de negócio | Pessoa idosa (60 anos ou mais) com renda de até 2 salários mínimos tem direito a 2 vagas gratuitas por viagem; esgotadas essas vagas, paga no máximo 50% da tarifa. | Média |
| RN04 | Regra de negócio | O número de passageiros a bordo não pode ultrapassar a lotação máxima da embarcação autorizada pela Capitania Fluvial da Amazônia Ocidental (Marinha do Brasil). | Alta |
| RN05 | Regra de negócio | A passagem só pode ser cancelada até 24 horas antes da partida, com devolução de 95% do valor pago (5% de taxa administrativa); depois desse prazo não há cancelamento nem reembolso. | Média |
| RN06 | Regra de negócio | Para embarcar, o passageiro deve apresentar documento oficial com foto cujo nome seja igual ao do bilhete. | Alta |
| RN07 | Regra de negócio | O embarque encerra 30 minutos antes do horário de partida; quem não embarcar até lá perde a passagem. | Média |
| RN08 | Regra de negócio | Cada passageiro tem direito a até 20 kg de bagagem gratuita; o excedente é cobrado por quilo conforme a tabela da empresa. | Baixa |

## 1.5 Resumo das quantidades

| Tipo | Quantidade | Mínimo exigido |
|---|---|---|
| Requisitos funcionais | 13 | 10 |
| Requisitos não funcionais | 12 (5 categorias) | 6 (4 categorias) |
| Regras de negócio | 8 | 5 |

---

# Parte 2 — Documento de arquitetura

A arquitetura é **cliente-servidor**: dois clientes web (passageiro e equipe interna) conversam por HTTPS/JSON com uma API REST, que é a única parte com acesso ao banco de dados e aos sistemas externos. Toda decisão abaixo aponta para um requisito da Parte 1 (ver [2.4 Rastreabilidade](#24-rastreabilidade)).

## 2.1 Diagramas C4 (níveis 1 a 3)

Os diagramas seguem o modelo C4 e as cores usuais da notação (pessoa em azul-escuro, sistema em foco em azul, contêiner em azul-médio, componente em azul-claro, sistema externo em cinza). Foram escritos com `flowchart` do Mermaid para que cada diagrama tenha legenda própria.

### Nível 1 — Contexto

O NavegaAM aparece como uma única caixa, com os quatro tipos de usuário e os três sistemas externos. Este nível não mostra banco de dados nem tecnologia.

```mermaid
---
title: "C4 Nível 1 — Contexto do sistema NavegaAM"
---
flowchart TB
    passageiro["<b>Passageiro</b><br/>[Pessoa]<br/>Compra, consulta e cancela passagens"]
    atendente["<b>Atendente de guichê</b><br/>[Pessoa]<br/>Vende passagens no porto"]
    fiscal["<b>Fiscal de embarque</b><br/>[Pessoa]<br/>Confere bilhetes na rampa de embarque"]
    admin["<b>Administrador</b><br/>[Pessoa]<br/>Gerencia frota, viagens e preços"]

    navega["<b>NavegaAM</b><br/>[Sistema de software]<br/>Venda de passagens fluviais, controle de embarque e gestão de viagens"]

    gateway["<b>Gateway de pagamento</b><br/>[Sistema externo]<br/>Processa Pix e cartão de crédito"]
    email["<b>Serviço de e-mail</b><br/>[Sistema externo]<br/>Entrega e-mails transacionais"]
    sms["<b>Serviço de SMS</b><br/>[Sistema externo]<br/>Entrega mensagens SMS"]

    passageiro -->|"Busca viagens, compra e cancela passagens"| navega
    atendente -->|"Registra vendas feitas no guichê"| navega
    fiscal -->|"Envia o código lido do QR Code do bilhete"| navega
    admin -->|"Cadastra embarcações e viagens, pede relatórios"| navega
    passageiro -->|"Informa dados do cartão ou paga o Pix"| gateway
    navega -->|"Pede cobranças e estornos"| gateway
    gateway -->|"Avisa que o pagamento foi confirmado"| navega
    navega -->|"Envia bilhete com QR Code e avisos"| email
    navega -->|"Envia código do bilhete e avisos"| sms
    email -->|"Entrega o e-mail"| passageiro
    sms -->|"Entrega o SMS"| passageiro

    subgraph legenda["Legenda"]
        direction LR
        lp["Pessoa"]:::pessoa
        ls["Sistema em foco"]:::sistema
        le["Sistema externo"]:::externo
        la["Origem"]:::neutro -->|"o que trafega"| lb["Destino"]:::neutro
    end

    class passageiro,atendente,fiscal,admin pessoa
    class navega sistema
    class gateway,email,sms externo
    classDef pessoa fill:#08427b,stroke:#052e56,color:#ffffff
    classDef sistema fill:#1168bd,stroke:#0b4884,color:#ffffff
    classDef externo fill:#8a8a8a,stroke:#6b6b6b,color:#ffffff
    classDef neutro fill:#ffffff,stroke:#999999,color:#333333
```

### Nível 2 — Contêineres

Abre a caixa do NavegaAM e mostra as partes que rodam separadas, cada uma com a tecnologia escolhida e cada seta com o protocolo.

```mermaid
---
title: "C4 Nível 2 — Contêineres do NavegaAM"
---
flowchart TB
    passageiro["<b>Passageiro</b><br/>[Pessoa]"]
    atendente["<b>Atendente de guichê</b><br/>[Pessoa]"]
    fiscal["<b>Fiscal de embarque</b><br/>[Pessoa]"]
    admin["<b>Administrador</b><br/>[Pessoa]"]

    subgraph navega["NavegaAM — Sistema de software"]
        spa["<b>Aplicação do Passageiro</b><br/>[Contêiner: React 18 + TypeScript, PWA servida pelo Amazon CloudFront]<br/>Busca viagens, compra, mostra e cancela passagens"]
        painel["<b>Painel de Operação</b><br/>[Contêiner: React 18 + TypeScript, PWA servida pelo Amazon CloudFront]<br/>Venda no guichê, leitura de QR Code, cadastros e relatórios"]
        api["<b>API NavegaAM</b><br/>[Contêiner: Node.js 20 + NestJS, API REST, 2+ instâncias no AWS ECS Fargate atrás de balanceador]<br/>Autenticação, regras de negócio, reservas, pagamentos e embarque"]
        worker["<b>Worker de Notificações</b><br/>[Contêiner: Node.js 20 + BullMQ]<br/>Gera o bilhete com QR Code e envia e-mail e SMS"]
        redis[("<b>Cache e Fila</b><br/>[Contêiner: Redis 7 no Amazon ElastiCache]<br/>Cache das buscas de viagens, tentativas de login e fila de notificações")]
        db[("<b>Banco de Dados</b><br/>[Contêiner: PostgreSQL 16 no Amazon RDS Multi-AZ]<br/>Usuários, embarcações, viagens, passagens e pagamentos")]
    end

    gateway["<b>Gateway de pagamento</b><br/>[Sistema externo]"]
    email["<b>Serviço de e-mail</b><br/>[Sistema externo]"]
    sms["<b>Serviço de SMS</b><br/>[Sistema externo]"]

    passageiro -->|"HTTPS"| spa
    atendente -->|"HTTPS"| painel
    fiscal -->|"HTTPS"| painel
    admin -->|"HTTPS"| painel
    spa -->|"HTTPS/JSON (REST)"| api
    painel -->|"HTTPS/JSON (REST)"| api
    spa -->|"HTTPS/JSON (tokenização do cartão)"| gateway
    api -->|"SQL sobre TLS (TCP 5432)"| db
    api -->|"RESP sobre TLS (TCP 6379)"| redis
    api -->|"HTTPS/JSON (cria cobrança e estorno)"| gateway
    gateway -->|"HTTPS/JSON (webhook assinado)"| api
    worker -->|"RESP sobre TLS (consome a fila)"| redis
    worker -->|"SQL sobre TLS (TCP 5432)"| db
    worker -->|"HTTPS/JSON (API de envio)"| email
    worker -->|"HTTPS/JSON (API de envio)"| sms

    subgraph legenda["Legenda"]
        direction LR
        lp["Pessoa"]:::pessoa
        lc["Contêiner"]:::container
        lb[("Armazenamento")]:::container
        le["Sistema externo"]:::externo
        la["Origem"]:::neutro -->|"protocolo"| ld["Destino"]:::neutro
    end

    class passageiro,atendente,fiscal,admin pessoa
    class spa,painel,api,worker,redis,db container
    class gateway,email,sms externo
    classDef pessoa fill:#08427b,stroke:#052e56,color:#ffffff
    classDef container fill:#438dd5,stroke:#2e6295,color:#ffffff
    classDef externo fill:#8a8a8a,stroke:#6b6b6b,color:#ffffff
    classDef neutro fill:#ffffff,stroke:#999999,color:#333333
    style navega fill:none,stroke:#1168bd,stroke-width:2px,stroke-dasharray:6 4
```

A borda tracejada marca a fronteira do sistema NavegaAM; tudo fora dela é pessoa ou sistema externo, os mesmos do nível 1.

### Nível 3 — Componentes da API NavegaAM

Abre o contêiner **API NavegaAM** e mostra os módulos internos separados nas camadas de apresentação, negócio e dados.

```mermaid
---
title: "C4 Nível 3 — Componentes da API NavegaAM"
---
flowchart TB
    spa["<b>Aplicação do Passageiro</b><br/>[Contêiner: React PWA]"]
    painel["<b>Painel de Operação</b><br/>[Contêiner: React PWA]"]

    subgraph api["API NavegaAM — Contêiner: Node.js 20 + NestJS"]
        direction TB
        subgraph apresentacao["Camada de apresentação"]
            guarda["<b>Guarda de Autenticação</b><br/>[Componente: NestJS Guard + Passport JWT]<br/>Valida o token e o perfil em toda requisição"]
            ctrlAuth["<b>AuthController</b><br/>[Componente: controller REST]<br/>/auth"]
            ctrlViagens["<b>ViagensController</b><br/>[Componente: controller REST]<br/>/viagens"]
            ctrlReservas["<b>ReservasController</b><br/>[Componente: controller REST]<br/>/reservas e /passagens"]
            ctrlEmbarque["<b>EmbarqueController</b><br/>[Componente: controller REST]<br/>/embarques"]
            ctrlAdmin["<b>AdminController</b><br/>[Componente: controller REST]<br/>/admin"]
            ctrlWebhook["<b>WebhookController</b><br/>[Componente: controller REST]<br/>/webhooks/pagamentos"]
        end
        subgraph negocio["Camada de negócio"]
            svcAuth["<b>Serviço de Autenticação</b><br/>[Componente: NestJS Service]<br/>bcrypt, bloqueio por tentativas, emissão de JWT e refresh token"]
            svcViagens["<b>Serviço de Viagens</b><br/>[Componente: NestJS Service]<br/>Busca com cache, cadastro de viagens e embarcações"]
            svcReservas["<b>Serviço de Reservas</b><br/>[Componente: NestJS Service]<br/>Reserva com controle de lotação, tarifas e cancelamento"]
            regras["<b>Motor de Regras de Negócio</b><br/>[Componente: módulo TypeScript]<br/>Gratuidades, descontos, lotação, prazos e documentos"]
            svcPagamentos["<b>Serviço de Pagamentos</b><br/>[Componente: NestJS Service]<br/>Cria cobrança, confirma pagamento e faz estorno"]
            svcEmbarque["<b>Serviço de Embarque</b><br/>[Componente: NestJS Service]<br/>Confere assinatura do QR Code e registra o embarque"]
            svcRelatorios["<b>Serviço de Relatórios</b><br/>[Componente: NestJS Service]<br/>Vendas e ocupação por viagem e período"]
        end
        subgraph dados["Camada de dados"]
            repos["<b>Repositórios</b><br/>[Componente: Prisma ORM]<br/>Acesso a usuários, viagens, passagens e pagamentos, cifra CPF e documento"]
            cacheFila["<b>Adaptador de Cache e Fila</b><br/>[Componente: ioredis + BullMQ]<br/>Lê e grava cache, publica jobs de notificação"]
            clienteGateway["<b>Cliente do Gateway</b><br/>[Componente: cliente HTTP]<br/>Chama a API do gateway de pagamento"]
        end
    end

    db[("<b>Banco de Dados</b><br/>[Contêiner: PostgreSQL 16]")]
    redis[("<b>Cache e Fila</b><br/>[Contêiner: Redis 7]")]
    gateway["<b>Gateway de pagamento</b><br/>[Sistema externo]"]

    spa -->|"HTTPS/JSON"| guarda
    painel -->|"HTTPS/JSON"| guarda
    guarda -->|"repassa requisição"| ctrlAuth
    guarda -->|"repassa requisição"| ctrlViagens
    guarda -->|"repassa requisição autenticada"| ctrlReservas
    guarda -->|"repassa requisição do fiscal"| ctrlEmbarque
    guarda -->|"repassa requisição do administrador"| ctrlAdmin
    gateway -->|"HTTPS/JSON (webhook assinado)"| ctrlWebhook

    ctrlAuth -->|"chama"| svcAuth
    ctrlViagens -->|"chama"| svcViagens
    ctrlReservas -->|"chama"| svcReservas
    ctrlEmbarque -->|"chama"| svcEmbarque
    ctrlAdmin -->|"chama"| svcViagens
    ctrlAdmin -->|"chama"| svcRelatorios
    ctrlWebhook -->|"chama"| svcPagamentos

    svcReservas -->|"valida tarifa, lotação e prazo"| regras
    svcEmbarque -->|"valida documento e horário"| regras
    svcReservas -->|"pede cobrança ou estorno"| svcPagamentos

    svcAuth -->|"lê e grava usuários"| repos
    svcAuth -->|"conta tentativas"| cacheFila
    svcViagens -->|"lê e grava viagens"| repos
    svcViagens -->|"lê e grava cache"| cacheFila
    svcReservas -->|"transação de reserva"| repos
    svcPagamentos -->|"atualiza pagamento"| repos
    svcPagamentos -->|"enfileira envio do bilhete"| cacheFila
    svcPagamentos -->|"usa"| clienteGateway
    svcEmbarque -->|"registra embarque"| repos
    svcRelatorios -->|"consulta"| repos

    repos -->|"SQL sobre TLS"| db
    cacheFila -->|"RESP sobre TLS"| redis
    clienteGateway -->|"HTTPS/JSON"| gateway

    subgraph legenda["Legenda"]
        direction LR
        lk["Componente"]:::componente
        lc["Contêiner"]:::container
        le["Sistema externo"]:::externo
        la["Origem"]:::neutro -->|"chamada ou protocolo"| ld["Destino"]:::neutro
    end

    class guarda,ctrlAuth,ctrlViagens,ctrlReservas,ctrlEmbarque,ctrlAdmin,ctrlWebhook,svcAuth,svcViagens,svcReservas,regras,svcPagamentos,svcEmbarque,svcRelatorios,repos,cacheFila,clienteGateway componente
    class spa,painel,db,redis container
    class gateway externo
    classDef componente fill:#85bbf0,stroke:#5d82a8,color:#000000
    classDef container fill:#438dd5,stroke:#2e6295,color:#ffffff
    classDef externo fill:#8a8a8a,stroke:#6b6b6b,color:#ffffff
    classDef neutro fill:#ffffff,stroke:#999999,color:#333333
    style api fill:none,stroke:#438dd5,stroke-width:2px,stroke-dasharray:6 4
```

| Camada | Componentes | Responsabilidade |
|---|---|---|
| Apresentação | Guarda de Autenticação e controllers | Recebe HTTP, valida token e formato do JSON, devolve o código de status. Não tem regra de negócio. |
| Negócio | Serviços e Motor de Regras | Aplica RF e RN (lotação, gratuidades, prazos). Não conhece HTTP nem SQL. |
| Dados | Repositórios, Adaptador de Cache e Fila, Cliente do Gateway | Fala com PostgreSQL, Redis e gateway. Só esta camada sai do contêiner. |

## 2.2 Registros de decisão (ADRs)

### ADR-001 — Usar aplicação web progressiva (PWA) como cliente, em vez de aplicativos nativos

**Status:** aceita
**Data:** 2026-11-06

#### Contexto
O cliente precisa funcionar em computadores do guichê e em celulares Android e iOS de passageiros e fiscais (**RNF12**), carregar rápido em conexões 3G do interior (**RNF03**), ser acessível (**RNF10**) e permitir a primeira compra em até 3 minutos (**RNF11**). O fiscal precisa ler o QR Code com a câmera do celular (**RF10**) e o atendente usa o computador do guichê (**RF09**).

#### Alternativas consideradas
##### Alternativa A — Aplicativos nativos (Kotlin para Android, Swift para iOS) e um site para o guichê
A favor: melhor acesso à câmera e ao modo offline; notificações push completas; presença nas lojas de aplicativos.
Contra: três bases de código; publicação depende da revisão das lojas; download de 30 a 80 MB é pesado no 3G (fere RNF03); o passageiro precisa instalar antes de comprar (prejudica RNF11).

##### Alternativa B — Aplicativo multiplataforma (Flutter) para celular e web
A favor: uma base de código para Android, iOS e web; bom desempenho no celular.
Contra: no celular ainda exige instalação pela loja; a versão web do Flutter gera um pacote inicial acima de 2 MB, o que fere RNF03; acessibilidade na web é mais limitada (RNF10).

##### Alternativa C — PWA em React + TypeScript para passageiros e para a equipe
A favor: roda em qualquer navegador moderno sem instalação (RNF12, RNF11); pode ser adicionada à tela inicial; câmera acessível pelo navegador (`getUserMedia`) para ler QR Code; divisão do código em partes mantém a primeira carga abaixo de 500 KB (RNF03); HTML semântico facilita a WCAG (RNF10).
Contra: recursos nativos limitados no iOS; sem presença nas lojas; precisa de testes em vários navegadores.

#### Decisão
Escolhemos a **Alternativa C (PWA em React)**, porque é a única que atende ao mesmo tempo RNF03 (carga leve em 3G), RNF11 (comprar sem instalar nada) e RNF12 (mesmo produto em desktop, Android e iOS), além de dar acesso à câmera para RF10.

#### Consequências
**Positivas:** uma só tecnologia para os dois clientes; atualizações chegam na hora, sem passar pelas lojas; o passageiro compra direto pelo link.
**Negativas:** notificações push no iOS só funcionam com a PWA instalada (iOS 16.4+), por isso o bilhete depende de e-mail e SMS (RF06), o que aumenta o custo com SMS; a leitura de QR Code em celulares antigos é mais lenta que em app nativo; sem vitrine nas lojas, a empresa precisa divulgar o endereço; cada versão exige testes em quatro navegadores.

---

### ADR-002 — Usar banco relacional PostgreSQL com transações para controlar a lotação

**Status:** aceita
**Data:** 2026-11-06

#### Contexto
A mesma viagem é vendida ao mesmo tempo pela internet (**RF04**, **RF05**) e no guichê (**RF09**), e a lotação autorizada pela Capitania nunca pode ser ultrapassada (**RN04**). Duas vendas simultâneas não podem ficar com a última vaga. Os relatórios cruzam vendas, viagens e embarcações (**RF13**). Uma falha no banco pode perder no máximo 5 minutos de dados e deve ser resolvida em até 1 hora (**RNF09**), e os dados pessoais precisam ser cifrados (**RNF07**).

#### Alternativas consideradas
##### Alternativa A — Banco de documentos (MongoDB Atlas)
A favor: esquema flexível; escala horizontal simples; bom para catálogos que mudam muito.
Contra: transações com vários documentos existem, mas são mais caras e menos naturais; relatórios que juntam vendas, viagens e embarcações exigem pipelines de agregação mais complexos; a equipe tem menos experiência.

##### Alternativa B — Banco relacional (PostgreSQL 16 no Amazon RDS Multi-AZ)
A favor: transações ACID com `SELECT ... FOR UPDATE` e restrição `CHECK (vagas >= 0)` impedem vender acima da lotação (RN04); SQL com `JOIN` e `GROUP BY` resolve os relatórios (RF13); o RDS Multi-AZ com recuperação para um ponto no tempo atende RNF09; criptografia em repouso gerenciada pelo AWS KMS ajuda no RNF07.
Contra: o esquema é rígido e exige migrações; escalar escrita depende de máquina maior (escala vertical).

#### Decisão
Escolhemos a **Alternativa B (PostgreSQL)**, porque a garantia de não vender acima da lotação (RN04) fica no próprio banco, por transação e restrição, e não depende de cada parte do código lembrar de checar.

#### Consequências
**Positivas:** é impossível vender acima da lotação mesmo com internet e guichê vendendo juntos; relatórios saem em SQL simples; backup e failover automáticos atendem RNF09.
**Negativas:** o bloqueio de linha cria fila de espera em viagens muito disputadas (véspera do Festival de Parintins), por isso as transações precisam ser curtas e a chamada ao gateway fica fora delas; toda mudança de esquema exige migração versionada (Prisma Migrate); o Multi-AZ custa cerca de duas vezes uma instância simples; crescer em escrita exige trocar a instância por uma maior.

---

### ADR-003 — Autenticar com JWT de curta duração e refresh token rotativo, em vez de sessão no servidor

**Status:** aceita
**Data:** 2026-11-06

#### Contexto
Os quatro tipos de usuário entram pelo mesmo login (**RF02**), com permissões diferentes. A credencial de acesso deve expirar em até 15 minutos e a sessão em 7 dias ou 12 horas, conforme o perfil (**RNF06**). Para chegar a 99,5% de disponibilidade (**RNF08**), a API roda em duas ou mais instâncias atrás de um balanceador, e qualquer instância precisa atender qualquer requisição.

#### Alternativas consideradas
##### Alternativa A — Sessão no servidor (cookie de sessão com dados guardados no Redis)
A favor: revogação imediata (basta apagar a sessão); modelo simples e conhecido; o cookie `HttpOnly` não fica exposto ao JavaScript.
Contra: toda requisição consulta o Redis, que vira ponto único de falha do login e soma latência (RNF01, RNF08); exige proteção contra CSRF em todas as rotas que alteram dados.

##### Alternativa B — JWT de acesso assinado (15 min) + refresh token opaco e rotativo
O token de acesso vai no cabeçalho `Authorization: Bearer`. O refresh token fica num cookie `HttpOnly`, `Secure`, `SameSite=Strict`, e o banco guarda só o hash dele.
A favor: a API não guarda estado, e cada instância valida o token só com a chave pública (RNF08); o perfil do usuário vai dentro do token; os prazos de RNF06 são configuração direta.
Contra: o token de acesso não pode ser revogado antes de expirar; mais complexidade (rotação do refresh token, detecção de reuso, rotação das chaves de assinatura).

#### Decisão
Escolhemos a **Alternativa B (JWT + refresh token rotativo)**, porque deixa a API sem estado, o que permite várias instâncias atrás do balanceador sem dependência extra no login (RNF08), e implementa diretamente os prazos de RNF06.

#### Consequências
**Positivas:** o balanceador não precisa de sessão fixa; uma instância pode cair sem deslogar ninguém; as permissões por perfil ficam no token e são checadas pela Guarda de Autenticação.
**Negativas:** um token de acesso roubado continua válido por até 15 minutos; ao desligar um funcionário, o acesso só termina de vez quando o token atual expira (mitigação: revogar os refresh tokens na hora); é preciso guardar e trocar periodicamente as chaves de assinatura no AWS Secrets Manager; os clientes precisam de código extra para renovar o token.

## 2.3 Comunicação cliente-servidor

### Fluxo de requisição e resposta — Comprar passagem com Pix (RF04, RF05, RF06)

```mermaid
sequenceDiagram
    title Comprar passagem com Pix (RF04, RF05, RF06)
    actor P as Passageiro
    participant SPA as Aplicação do Passageiro (React PWA)
    participant API as API NavegaAM (NestJS)
    participant DB as Banco de Dados (PostgreSQL)
    participant GW as Gateway de pagamento
    participant Q as Cache e Fila (Redis)
    participant W as Worker de Notificações
    participant N as Serviços de e-mail e SMS

    Note over P,N: Legenda: seta cheia = chamada síncrona, seta tracejada = resposta, seta aberta = mensagem assíncrona

    P->>SPA: Escolhe a viagem, a rede e confirma
    SPA->>API: POST /api/v1/reservas (Bearer JWT, JSON)
    API->>API: Valida o token e aplica RN01 a RN03 (tarifas e gratuidades)
    API->>DB: BEGIN e SELECT vagas ... FOR UPDATE
    DB-->>API: vagas disponíveis da acomodação
    alt Sem vagas (RN04)
        API->>DB: ROLLBACK
        API-->>SPA: 409 Conflict com erro SEM_VAGAS
        SPA-->>P: Mostra "Acomodação esgotada"
    else Com vagas
        API->>DB: INSERT reserva AGUARDANDO_PAGAMENTO e UPDATE vagas, COMMIT
        DB-->>API: OK
        API->>GW: POST /cobrancas (valor, txid)
        GW-->>API: 201 Created com código Pix copia e cola
        API-->>SPA: 201 Created com código da reserva e Pix
        SPA-->>P: Mostra o QR Code do Pix
    end

    Note over P,GW: O passageiro paga o Pix no aplicativo do banco
    GW-)API: POST /api/v1/webhooks/pagamentos (assinado com HMAC)
    API->>DB: UPDATE passagem SET status = EMITIDA
    DB-->>API: OK
    API-)Q: Enfileira o job enviar-bilhete
    API-->>GW: 200 OK
    Q-)W: Entrega o job
    W->>DB: SELECT dados do bilhete
    DB-->>W: Dados do passageiro e da viagem
    W->>N: POST envio de e-mail e SMS com o QR Code assinado
    N-)P: Bilhete por e-mail e SMS
```

O envio do bilhete fica fora do caminho da compra (worker e fila), e a chamada ao gateway fica fora da transação. Assim a resposta ao passageiro não espera e-mail nem SMS (RNF02) e a linha da viagem fica bloqueada o menor tempo possível (ADR-002). Uma reserva não paga em 15 minutos expira e devolve a vaga.

### Requisições HTTP

Base da API: `https://api.navegaam.com.br/api/v1`. Todas as respostas usam `Content-Type: application/json; charset=utf-8`.

#### Requisição 1 — Consultar viagens (RF03)

```http
GET /api/v1/viagens?origem=MAO&destino=PIN&data=2026-12-10 HTTP/1.1
Host: api.navegaam.com.br
Accept: application/json
```

Resposta: **`200 OK`**

```json
{
  "origem": { "codigo": "MAO", "nome": "Manaus" },
  "destino": { "codigo": "PIN", "nome": "Parintins" },
  "data": "2026-12-10",
  "viagens": [
    {
      "id": 4821,
      "embarcacao": "Lancha Rio Negro Expresso II",
      "tipo": "LANCHA",
      "partida": "2026-12-10T06:00:00-04:00",
      "chegadaPrevista": "2026-12-10T17:00:00-04:00",
      "acomodacoes": [
        { "tipo": "POLTRONA", "preco": 260.00, "vagas": 34 }
      ]
    },
    {
      "id": 4830,
      "embarcacao": "B/M Amazonas Star",
      "tipo": "BARCO",
      "partida": "2026-12-10T12:00:00-04:00",
      "chegadaPrevista": "2026-12-11T08:00:00-04:00",
      "acomodacoes": [
        { "tipo": "REDE", "preco": 180.00, "vagas": 112 },
        { "tipo": "CAMAROTE", "preco": 650.00, "vagas": 3 }
      ]
    }
  ]
}
```

#### Requisição 2 — Reservar e comprar passagem (RF04, RF05)

Passageira adulta em rede, com o filho de 4 anos no colo (gratuidade da RN01).

```http
POST /api/v1/reservas HTTP/1.1
Host: api.navegaam.com.br
Authorization: Bearer eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...
Content-Type: application/json

{
  "viagemId": 4830,
  "formaPagamento": "PIX",
  "passageiros": [
    { "nome": "Ana Souza", "dataNascimento": "1990-03-15", "documento": "RG 1234567-8", "acomodacao": "REDE" },
    { "nome": "Pedro Souza", "dataNascimento": "2022-05-02", "documento": "Certidão 0012345", "acomodacao": "COLO" }
  ]
}
```

Resposta: **`201 Created`** (cabeçalho `Location: /api/v1/reservas/NAV-7Q2K9`)

```json
{
  "codigo": "NAV-7Q2K9",
  "status": "AGUARDANDO_PAGAMENTO",
  "viagemId": 4830,
  "itens": [
    { "passageiro": "Ana Souza", "acomodacao": "REDE", "tarifa": "INTEIRA", "valor": 180.00 },
    { "passageiro": "Pedro Souza", "acomodacao": "COLO", "tarifa": "GRATUIDADE_ATE_5_ANOS", "regra": "RN01", "valor": 0.00 }
  ],
  "valorTotal": 180.00,
  "pagamento": {
    "forma": "PIX",
    "pixCopiaECola": "00020126580014br.gov.bcb.pix0136a1b2c3d4-e5f6-7890-abcd-ef1234567890520400005303986540518000",
    "expiraEm": "2026-11-06T20:15:00-04:00"
  }
}
```

**Caso de erro** — mesma requisição quando não há mais redes na viagem: **`409 Conflict`**

```json
{
  "status": 409,
  "erro": "SEM_VAGAS",
  "mensagem": "Não há mais redes disponíveis nesta viagem.",
  "regra": "RN04"
}
```

#### Requisição 3 — Alterar viagem (RF12)

O administrador muda o horário de partida. Como é `PUT`, o corpo traz a representação completa da viagem.

```http
PUT /api/v1/admin/viagens/4830 HTTP/1.1
Host: api.navegaam.com.br
Authorization: Bearer eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...
Content-Type: application/json

{
  "embarcacaoId": 12,
  "origem": "MAO",
  "destino": "PIN",
  "partida": "2026-12-10T14:00:00-04:00",
  "chegadaPrevista": "2026-12-11T10:00:00-04:00",
  "precos": { "REDE": 180.00, "CAMAROTE": 650.00 }
}
```

Resposta: **`200 OK`**

```json
{
  "id": 4830,
  "embarcacaoId": 12,
  "embarcacao": "B/M Amazonas Star",
  "origem": "MAO",
  "destino": "PIN",
  "partida": "2026-12-10T14:00:00-04:00",
  "chegadaPrevista": "2026-12-11T10:00:00-04:00",
  "precos": { "REDE": 180.00, "CAMAROTE": 650.00 },
  "atualizadoEm": "2026-11-06T18:42:10-04:00"
}
```

#### Requisição 4 — Cancelar passagem (RF07)

```http
DELETE /api/v1/passagens/NAV-7Q2K9 HTTP/1.1
Host: api.navegaam.com.br
Authorization: Bearer eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...
```

Resposta (cancelada 9 dias antes da partida): **`200 OK`**

```json
{
  "codigo": "NAV-7Q2K9",
  "status": "CANCELADA",
  "canceladaEm": "2026-12-01T10:12:00-04:00",
  "reembolso": {
    "valorPago": 180.00,
    "taxaAdministrativa": 9.00,
    "valorDevolvido": 171.00,
    "forma": "PIX",
    "prazoDias": 7,
    "regra": "RN05"
  }
}
```

**Caso de erro** — pedido feito a menos de 24 horas da partida: **`422 Unprocessable Content`**

```json
{
  "status": 422,
  "erro": "PRAZO_CANCELAMENTO_EXPIRADO",
  "mensagem": "A passagem só pode ser cancelada até 24 horas antes da partida.",
  "regra": "RN05"
}
```

#### Resumo dos códigos de status usados

| Código | Quando |
|---|---|
| `200 OK` | Consulta, alteração ou cancelamento bem-sucedidos |
| `201 Created` | Reserva criada (com `Location`) |
| `401 Unauthorized` | Token ausente, inválido ou expirado (RNF06) |
| `403 Forbidden` | Perfil sem permissão (ex.: passageiro chamando `/admin`) |
| `409 Conflict` | Conflito com o estado atual, como falta de vagas (RN04) |
| `422 Unprocessable Content` | Pedido bem formado que viola uma regra de negócio, como o prazo da RN05 |

### HTTPS

**Dados do NavegaAM que precisam de proteção em trânsito:**

| Dado | Onde trafega | Risco sem proteção |
|---|---|---|
| E-mail, senha, token JWT e refresh token | Login e todas as chamadas autenticadas | Roubo de conta (alguém compra ou cancela em nome do passageiro) |
| Dados pessoais: nome, CPF, data de nascimento, documento, telefone | Cadastro (RF01), reserva (RF04) e venda no guichê (RF09) | Vazamento de dados pessoais, infração à LGPD (RNF07) |
| Dados de pagamento: número do cartão e código Pix | Tokenização do cartão no gateway e resposta da reserva | Fraude; o número do cartão nem passa pela API, vai direto ao gateway |
| Código do bilhete e QR Code | Envio do bilhete (RF06) e validação no embarque (RF10) | Bilhete copiado e usado por outra pessoa |
| Webhook de pagamento | Gateway para a API | Aviso falso de "pago" para emitir passagem sem pagamento |
| Ligação da API com PostgreSQL e Redis | Rede interna da nuvem | Leitura dos dados por quem tiver acesso à rede |

**O que o TLS garante:**

1. **Confidencialidade** — o conteúdo é cifrado; quem intercepta a conexão (por exemplo, num Wi-Fi público do porto) não lê senhas, CPF nem tokens.
2. **Integridade** — qualquer alteração dos dados no caminho é detectada e a conexão é derrubada; ninguém troca o valor da passagem ou o código Pix no meio do caminho.
3. **Autenticidade do servidor** — o certificado assinado por uma autoridade certificadora prova que o cliente está falando com `api.navegaam.com.br`, e não com um site falso.

**O que o TLS não garante** (e por isso existem outras medidas): não protege os dados guardados no banco (cifragem em repouso, RNF07); não prova quem é o cliente (login com JWT, ADR-003); não impede um webhook falso vindo de outro servidor (assinatura HMAC conferida pela API); não impede que um bilhete seja copiado depois de entregue (QR Code assinado e marcado como usado no primeiro embarque).

**Configuração adotada:** TLS 1.2 ou 1.3 apenas, com certificados do AWS Certificate Manager no balanceador e no CloudFront; redirecionamento de HTTP para HTTPS e cabeçalho `Strict-Transport-Security: max-age=31536000; includeSubDomains` (RNF04); TLS obrigatório também nas conexões internas com PostgreSQL e Redis.

## 2.4 Rastreabilidade

### RNF → decisão de arquitetura

| RNF | Decisão que o atende (ADR ou elemento do C4) |
|---|---|
| RNF01 — consulta em até 2 s com 500 usuários | **C4 N2:** Cache e Fila (Redis) guarda as buscas de viagens; API com 2+ instâncias. **C4 N3:** Serviço de Viagens lê o cache antes do banco. |
| RNF02 — reserva em até 3 s no pico | **ADR-002:** transação curta, sem a chamada ao gateway dentro dela. **C4 N2:** Worker de Notificações tira o envio de e-mail e SMS do caminho da compra. |
| RNF03 — até 500 KB e 5 s em 3G | **ADR-001:** PWA em React com divisão do código. **C4 N2:** Aplicação do Passageiro servida pelo CloudFront (CDN). |
| RNF04 — HTTPS com TLS 1.2+ | **C4 N2:** todas as setas usam HTTPS ou TLS. **Seção 2.3 HTTPS:** TLS 1.2/1.3, HSTS e redirecionamento. |
| RNF05 — bcrypt custo ≥ 12 e bloqueio após 5 tentativas | **C4 N3:** Serviço de Autenticação (bcrypt) e Adaptador de Cache e Fila (contador de tentativas no Redis). |
| RNF06 — acesso expira em 15 min, sessão em 7 dias ou 12 h | **ADR-003:** JWT de 15 minutos e refresh token rotativo com prazo por perfil. |
| RNF07 — LGPD e AES-256 em repouso | **ADR-002:** RDS com criptografia em repouso pelo AWS KMS. **C4 N3:** Repositórios cifram CPF e documento antes de gravar. |
| RNF08 — 99,5% ao mês | **C4 N2:** API com 2+ instâncias no ECS Fargate atrás de balanceador; PostgreSQL em RDS Multi-AZ. **ADR-003:** API sem estado, sem sessão fixa. |
| RNF09 — RPO 5 min e RTO 1 h | **ADR-002:** RDS Multi-AZ com failover automático e recuperação para um ponto no tempo. |
| RNF10 — WCAG 2.1 AA | **ADR-001:** cliente web com HTML semântico, verificado pelo axe-core. **C4 N2:** Aplicação do Passageiro. |
| RNF11 — 90% compram em até 3 min | **ADR-001:** compra direto no navegador, sem instalar aplicativo. **C4 N2:** Aplicação do Passageiro. |
| RNF12 — navegadores, Android 9+, iOS 15+, 360 px | **ADR-001:** PWA responsiva em vez de aplicativos nativos. |

### Requisição HTTP → RF

| Requisição | RF atendido | RN aplicada |
|---|---|---|
| `GET /api/v1/viagens` | RF03 | — |
| `POST /api/v1/reservas` (201 e 409) | RF04, RF05 | RN01 (gratuidade), RN04 (lotação) |
| `PUT /api/v1/admin/viagens/{id}` | RF12 | — |
| `DELETE /api/v1/passagens/{codigo}` (200 e 422) | RF07 | RN05 (prazo e reembolso) |

### ADR → requisitos

| ADR | Requisitos que motivaram a decisão |
|---|---|
| ADR-001 — PWA em vez de apps nativos | RNF03, RNF10, RNF11, RNF12, RF09, RF10 |
| ADR-002 — PostgreSQL com transações | RN04, RF04, RF05, RF09, RF13, RNF07, RNF09 |
| ADR-003 — JWT + refresh token rotativo | RF02, RNF06, RNF08 |

Resultado da revisão: os 12 RNF têm decisão, as 4 requisições HTTP têm RF correspondente e os 3 ADRs apontam para requisitos declarados.
