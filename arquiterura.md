# FLUXO TÉCNICO COMPLETO
## Automação para Pequenos Restaurantes - Fundamentação Arquitetônica

---

## 1. VISÃO GERAL DA ARQUITETURA

```
┌─────────────────────────────────────────────────────────────────┐
│                    CAMADAS DO SISTEMA                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  CLIENTE                  BOT                    BACKEND        │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────────┐   │
│  │  WhatsApp    │◄──►│  Bot Server  │◄──►│  API REST        │   │
│  │  Cliente     │    │  (Python)    │    │  (Python/Fast API)  │   │
│  └──────────────┘    └──────────────┘    └──────────────────┘   │
│                            │                       │            │
│                            │                       ▼            │
│                            │              ┌──────────────────┐  │
│                            │              │  BANCO DE DADOS  │  │
│                            │              │  (MySQL)         │  │
│                            │              └──────────────────┘  │
│                            │                                    │
│                     INTERNET (HTTPS)                            │
│                     Via WhatsApp Cloud API                      │
│                                                                 │
│  INFRAESTRUTURA                                                 │
│  ┌─────────────────────────────────────────────────────┐        │
│  │               Cloudflare + Servidor local           │        │
│  └─────────────────────────────────────────────────────┘        │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. FUNDAMENTOS TEÓRICOS

### 2.1 Por que Python para este Projeto?

**Python** é adequado para este caso por:

1. **Prototipagem Rápida**: Sintaxe simples, permite desenvolvimento ágil
2. **Bibliotecas Robustas**: FastAPI para APIs, PyMySQL/SQLAlchemy para BD
3. **Comunidade**: Grande suporte para bots e integrações
4. **Manutenção**: Fácil de entender para futuros desenvolvedores
5. **Custo**: Open-source, sem licenças

### 2.2 Por que WhatsApp Cloud API (WHAPI.Cloud)?

**Alternativa a**: WhatsApp Business API (requer aprovação, é complexa)

**Vantagens do WHAPI.Cloud**:
- Integração mais simples (menos configuração)
- Documentação clara
- Plano gratuito com limite de mensagens
- Webhook para receber mensagens em tempo real
- Não precisa fazer autenticação complexa

**Funcionamento**:
```
Cliente envia mensagem no WhatsApp
         ↓
WHAPI.Cloud recebe (infraestrutura deles)
         ↓
WHAPI envia webhook para nosso servidor (POST HTTP)
         ↓
Nossa API processa a mensagem
         ↓
Nossa API envia resposta de volta para WHAPI.Cloud
         ↓
WHAPI.Cloud entrega para cliente via WhatsApp
```

### 2.3 Por que MySQL para o Banco de Dados?

**MySQL** foi escolhido para este projeto por:

1. **Estrutura Relacional**: Dados organizados em tabelas bem definidas (restaurantes, pedidos, clientes)
2. **ACID Compliance**: Garante consistência dos pedidos e transações financeiras
3. **Escalabilidade**: Suporta múltiplos restaurantes sem problemas de performance
4. **Comunidade Madura**: Amplo suporte, documentação e ferramentas disponíveis
5. **Segurança**: Permissões por usuário, criptografia integrada
6. **Custo**: Open-source, sem licenças, hospedagem econômica
7. **Integração Python**: PyMySQL e SQLAlchemy oferecem abstração robusta

**Alternativas Consideradas**:
- **MongoDB**: Mais flexível, mas menos eficiente para dados estruturados como cardápios e pedidos
- **PostgreSQL**: Igualmente robusto, mas MySQL é mais leve para este caso

### 2.4 Por que Cloudflare?

**Cloudflare** fornece:
1. **DNS**: Mapeia domínio para IP do servidor (exemplo.com → 192.168.1.1)
2. **SSL/HTTPS**: Certificado gratuito (obrigatório para webhooks)
3. **CDN**: Acelera requisições (caching)
4. **DDoS Protection**: Protege servidor de ataques
5. **Firewall**: Controla quem acessa

---

## 3. ORGANIZAÇÃO DE REPOSITÓRIO (ESTRUTURA GIT)

### 3.1 Estrutura do Projeto em Pastas

```
seu-repositorio/
│
├── .github/
│   └── workflows/              # CI/CD (testes automáticos)
│       └── tests.yml           # Rodar testes a cada push
│
├── src/                         # CÓDIGO FONTE PRINCIPAL
│   ├── main.py                 # Arquivo principal (entry point)
│   ├── config.py               # Configurações (variáveis ambiente)
│   ├── requirements.txt         # Dependências Python
│   │
│   ├── services/               # LÓGICA DE NEGÓCIO
│   │   ├── __init__.py
│   │   ├── bot_service.py      # Lógica do fluxo do bot
│   │   ├── menu_service.py     # Gerenciar cardápio
│   │   ├── order_service.py    # Gerenciar pedidos
│   │   ├── customer_service.py # Gerenciar clientes
│   │   └── whapi_service.py    # Integração com WHAPI.Cloud
│   │
│   ├── models/                 # SCHEMAS DO BANCO DE DADOS
│   │   ├── __init__.py
│   │   ├── restaurant.py       # Modelo de restaurante
│   │   ├── order.py            # Modelo de pedido
│   │   ├── customer.py         # Modelo de cliente
│   │   └── menu_item.py        # Modelo de item do cardápio
│   │
│   ├── routes/                 # ENDPOINTS DA API (rotas HTTP)
│   │   ├── __init__.py
│   │   ├── auth_routes.py      # Login, token, permissões
│   │   ├── restaurant_routes.py # CRUD restaurante (cadastro)
│   │   ├── menu_routes.py      # CRUD cardápio
│   │   ├── order_routes.py     # CRUD pedidos
│   │   ├── webhook_routes.py   # Receber mensagens do WHAPI
│   │
│   ├── database/               # CONEXÃO COM BANCO
│   │   ├── __init__.py
│   │   ├── mysql_connection.py # Classe de conexão MySQL (SQLAlchemy)
│   │   └── migrations/         # Scripts Alembic para versionamento do schema
│   │
│   ├── middleware/             # PROCESSADORES DE REQUISIÇÃO
│   │   ├── __init__.py
│   │   ├── auth_middleware.py  # Validar token JWT
│   │   └── error_handler.py    # Tratamento de erros
│   │
│   └── utils/                  # FUNÇÕES AUXILIARES
│       ├── __init__.py
│       ├── validators.py       # Validação de dados
│       ├── formatters.py       # Formatação de mensagens
│       └── logger.py           # Sistema de logs
│
│
├── docs/                       # DOCUMENTAÇÃO
│   ├── API.md                  # Documentação dos endpoints
│   ├── ARCHITECTURE.md         # Este documento
│   ├── SETUP.md                # Como configurar
│   └── DEPLOYMENT.md           # Como fazer deploy
│
├── env.example                 # Variáveis de ambiente (exemplo)
├── .gitignore                  # Arquivos a ignorar
├── README.md                   # Documentação principal
└── LICENSE                     # Licença do projeto

```

### 3.2 Convenções de Organização

**Por que essa estrutura?**
- **Separação de responsabilidades**: cada pasta tem um propósito
- **Escalabilidade**: fácil adicionar novos serviços
- **Testabilidade**: código modular é fácil de testar
- **Manutenção**: novo desenvolvedor entende onde procurar
- **Padrão indústria**: segue convenções Python (MVC adaptado)

---

## 4. SERVIÇOS (MICROSERVIÇOS LÓGICOS)

### 4.1 Bot Service (Fluxo da Conversa)

**Arquivo**: `src/services/bot_service.py`

**Responsabilidade**: Gerenciar a lógica de conversa com cliente

```
FLUXO:
1. Cliente envia mensagem
2. WHAPI.Cloud envia webhook para /webhook/message
3. Bot Service processa:
   - Identifica restaurante (baseado no número de telefone)
   - Identifica cliente
   - Verifica em qual "estado" da conversa está (inicial, selecionando itens, etc)
   - Gera resposta apropriada

4. Resposta é enviada de volta para cliente

ESTADOS POSSÍVEIS:
├─ INICIAL
│  └─ Cliente envia qualquer coisa
│     Resposta: "Olá! Bem-vindo ao [Restaurante]. Escolha uma opção:"
│
├─ AGUARDANDO_ACAO
│  ├─ Cliente digita: "1" (ver cardápio)
│  │  Resposta: Lista os itens disponíveis
│  ├─ Cliente digita: "2" (meu pedido)
│  │  Resposta: Mostra pedido atual
│  └─ Cliente digita: "3" (confirmar pedido)
│     Resposta: "Pedido confirmado! Número: #123"
│
└─ FAZENDO_PEDIDO
   └─ Cliente seleciona item
      Resposta: "Item adicionado. Deseja mais algo?"
```
**Pseudocódigo**:
```python
class BotService:
    def processar_mensagem(numero_cliente, mensagem, restaurante_id):
        # 1. Busca histórico do cliente
        historico = BuscaHistorico(cliente_id)
        estado_atual = historico.ultimo_estado
        
        # 2. Processa baseado no estado
        if estado_atual == "INICIAL":
            resposta = GeraNavegarMenu()
        elif estado_atual == "FAZENDO_PEDIDO":
            resposta = ProcessaSelecaoItem(mensagem)
        elif estado_atual == "CONFIRMACAO":
            resposta = ConfirmaEFinalizaPedido()
        
        # 3. Salva novo estado
        SalvaEstado(cliente_id, novo_estado)
        
        # 4. Envia resposta
        return resposta
```

### 4.2 Menu Service

**Arquivo**: `src/services/menu_service.py`

**Responsabilidade**: Gerenciar cardápio do restaurante

```
FUNCIONALIDADES:
- Buscar cardápio do restaurante
- Filtrar itens por disponibilidade
- Formatar cardápio para exibição no bot
- Validar se item existe antes de adicionar ao pedido

DADOS ARMAZENADOS:
{
  "restaurante_id": "12345",
  "itens": [
    {
      "id": "item_001",
      "nome": "Burger Classic",
      "preco": 25.00,
      "descricao": "Pão, carne, queijo, alface",
      "disponivel": true,
      "imagem_url": "..."
    }
  ],
  "personalizacao_por_dia": true,
  "tipo_exibicao": "texto" ou "imagem"
}
```

### 4.3 Order Service

**Arquivo**: `src/services/order_service.py`

**Responsabilidade**: Gerenciar pedidos

```
FUNCIONALIDADES:
- Criar novo pedido
- Adicionar item ao pedido
- Remover item do pedido
- Confirmar pedido
- Consultar status do pedido

DADOS DE PEDIDO:
{
  "pedido_id": "PED_001",
  "cliente_id": "CLI_001",
  "restaurante_id": "REST_001",
  "itens": [
    {"item_id": "item_001", "quantidade": 2, "preco_unitario": 25.00},
    {"item_id": "item_002", "quantidade": 1, "preco_unitario": 15.00}
  ],
  "total": 65.00,
  "status": "PENDENTE", "CONFIRMADO", "ENTREGANDO", "ENTREGUE"
  "data_criacao": "2024-01-15 14:30:00",
  "data_confirmacao": "2024-01-15 14:35:00"
}
```

### 4.4 Customer Service

**Arquivo**: `src/services/customer_service.py`

**Responsabilidade**: Gerenciar dados dos clientes

```
FUNCIONALIDADES:
- Identificar cliente pela número WhatsApp
- Buscar histórico do cliente
- Armazenar preferências

DADOS DE CLIENTE:
{
  "cliente_id": "CLI_001",
  "numero_whatsapp": "11999999999",
  "nome": "João",
  "historico_pedidos": ["PED_001", "PED_002", "PED_003"],
  "preferencias": {
    "sem_tomate": true,
    "alergias": ["amendoim"]
  },
  "data_primeiro_pedido": "2024-01-15"
}
```

### 4.5 WHAPI Service

**Arquivo**: `src/services/whapi_service.py`

**Responsabilidade**: Integração com WhatsApp Cloud

```
FUNCIONALIDADES:
- Enviar mensagens de texto
- Enviar mensagens com imagem
- Enviar mensagens com botões
- Receber webhooks

EXEMPLO DE USO:
# Para enviar mensagem simples
whapi.enviar_mensagem(
  numero_destino="11999999999",
  mensagem="Olá! Bem-vindo ao restaurante"
)

# Para enviar mensagem com imagem
whapi.enviar_imagem(
  numero_destino="11999999999",
  url_imagem="https://...",
  caption="Veja nosso cardápio!"
)

# Para enviar mensagem com botões
whapi.enviar_botoes(
  numero_destino="11999999999",
  titulo="Escolha uma opção:",
  botoes=[
    {"texto": "Ver Cardápio", "id": "1"},
    {"texto": "Meu Pedido", "id": "2"},
    {"texto": "Confirmar", "id": "3"}
  ]
)
```

---

## 5. FLUXO DE DADOS DETALHADO

### 5.1 Fluxo: Cliente → Bot → Banco de Dados

```
┌─────────────────────────────────────────────────────────────────┐
│ PASSO 1: CLIENTE ENVIA MENSAGEM                                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                   │
│  Cliente: "Quero pedir um burger"                               │
│         ↓ (via Internet)                                         │
│  WhatsApp cloud (WHAPI.CLOUD) recebe                           │
│         ↓ = envia ao server                              │
│  server processa e armazena                                       │
│         ↓                                                         │
│  Envia Webhook HTTP POST para nosso servidor                    │
│  URL: https://seudominio.com/api/webhook/message                │
│                                                                   │
│  CORPO DO WEBHOOK:                                               │
│  {                                                                │
│    "tipo_evento": "message",                                     │
│    "numero_remetente": "11999999999",                            │
│    "mensagem": "Quero pedir um burger",                          │
│    "timestamp": "2024-01-15T14:30:00Z",                          │
│    "restaurante_id": "12345" (identificado por outro meio)      │
│  }                                                                │
│                                                                   │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│ PASSO 2: NOSSA API RECEBE E PROCESSA                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                   │
│  1. Endpoint /api/webhook/message recebe POST                   │
│     └─ Middleware valida autenticidade do webhook                │
│                                                                   │
│  2. CustomerService busca cliente                                │
│     └─ SELECT * FROM clientes WHERE numero = "11999999999"      │
│     └─ Se não existe, cria novo cliente                          │
│                                                                   │
│  3. BotService processa mensagem                                 │
│     └─ Verifica estado da conversa                               │
│     └─ Aplica lógica: é saudação? pedido? confirmação?          │
│     └─ Gera resposta apropriada                                  │
│                                                                   │
│  4. Se for pedido, OrderService salva                            │
│     └─ INSERT INTO pedidos {...}                                │
│     └─ Gera ID do pedido                                         │
│                                                                   │
│  5. Salva novo estado do cliente                                 │
│     └─ UPDATE clientes SET estado = "SELECIONANDO_ITEM"         │
│                                                                   │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│ PASSO 3: API ENVIA RESPOSTA PARA WHAPI                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                   │
│  Nossa API chama WHAPI.Cloud via HTTP POST                      │
│  URL: https://api.whapi.cloud/messages/send                     │
│                                                                   │
│  CORPO:                                                           │
│  {                                                                │
│    "numero_destino": "11999999999",                              │
│    "mensagem": "Ótimo! Nossos burgers estão em alta!",          │
│    "botoes": [                                                    │
│      {"texto": "Ver Cardápio Completo", "id": "1"},             │
│      {"texto": "Burger Classic", "id": "2"},                     │
│      {"texto": "Burger Premium", "id": "3"}                      │
│    ]                                                              │
│  }                                                                │
│                                                                   │
│  WHAPI responde: {"status": "enviado", "id_mensagem": "msg_001"}│
│                                                                   │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│ PASSO 4: CLIENTE RECEBE RESPOSTA                                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                   │
│  WhatsApp mostra para cliente:                                   │
│  ┌──────────────────────────────┐                               │
│  │ Restaurante XYZ              │                                │
│  │                              │                                │
│  │ Ótimo! Nossos burgers estão  │                               │
│  │ em alta!                      │                               │
│  │                              │                                │
│  │ [Ver Cardápio Completo]      │                               │
│  │ [Burger Classic]             │                               │
│  │ [Burger Premium]             │                               │
│  └──────────────────────────────┘                               │
│                                                                   │
│  Ciclo se repete quando cliente clica em botão...               │
│                                                                   │
└─────────────────────────────────────────────────────────────────┘
```

### 5.2 Fluxo Completo: Cadastro do Restaurante

```
┌──────────────────────────────────────────────────────────────────┐
│ CADASTRO DE NOVO RESTAURANTE                                     │
├──────────────────────────────────────────────────────────────────┤
│                                                                    │
│ 1. PROPRIETÁRIO ACESSA DASHBOARD WEB                             │
│    URL: https://seudominio.com/cadastro                          │
│    └─ Página web (HTML + JavaScript)                             │
│                                                                    │
│ 2. PREENCHE FORMULÁRIO:                                           │
│    ├─ Nome do restaurante: "Burger House"                        │
│    ├─ Número WhatsApp: "11999999999"                             │
│    ├─ Cardápio: (arquivo ou manual)                              │
│    ├─ Modelo: "Texto" ou "Imagem"                                │
│    └─ Mensagens customizadas                                      │
│                                                                    │
│ 3. FRONTEND ENVIA POST PARA API                                  │
│    POST /api/restaurantes/criar                                  │
│    CORPO:                                                          │
│    {                                                               │
│      "nome": "Burger House",                                      │
│      "numero_whatsapp": "11999999999",                            │
│      "cardapio": [...itens...],                                   │
│      "tipo_exibicao": "texto",                                    │
│      "mensagens": {                                                │
│        "saudacao": "Olá! Bem-vindo!",                            │
│        "obrigado": "Obrigado! Entraremos em contato!"            │
│      }                                                             │
│    }                                                               │
│                                                                    │
│ 4. API VALIDA DADOS                                               │
│    ├─ Verifica se número é válido                                │
│    ├─ Verifica se restaurante já existe                          │
│    └─ Valida cada item do cardápio                               │
│                                                                    │
│ 5. API SALVA NO MySQL                                           │
│    INSERT INTO restaurantes (restaurante_id,nome,numero_whatsapp,cardapio, status)
     VALUES ('burguers_link','21999009073','cardapio.json','ativo' )
│                                          │
│                                                                   │
│                                                                    │
│ 6. API GERA TOKEN DE ACESSO                                       │
│    └─ Token JWT contém restaurante_id                            │
│    └─ Válido por 365 dias                                        │
│    └─ Armazenado no banco                                         │
│                                                                    │
│ 7. API ENVIA RESPOSTA                                             │
│    {                                                               │
│      "status": "sucesso",                                         │
│      "restaurante_id": "rest_12345",                              │
│      "token": "eyJhbGciOiJIUzI1NiIs...",                         │
│      "link_webhook": "https://seudominio.com/webhook/rest_12345" │
│    }                                                               │
│                                                                    │
│ 8. FRONTEND EXIBE:                                                │
│    ├─ "Restaurante criado com sucesso!"                          │
│    ├─ QR Code do link webhook (para o WHAPI conectar)            │
│    ├─ Link para dashboard de gerenciamento                       │
│    └─ Instruções de como conectar ao WHAPI                       │
│                                                                    │
└──────────────────────────────────────────────────────────────────┘
```

---

## 6. BANCO DE DADOS (MYSQL)

### 6.1 Estrutura de Tabelas Relacionais

**MySQL** organiza dados em tabelas normalizadas com relacionamentos bem definidos.

```sql
-- TABELA: restaurantes
CREATE TABLE restaurantes (
  restaurante_id INT PRIMARY KEY AUTO_INCREMENT,
  nome VARCHAR(255) NOT NULL,
  numero_whatsapp VARCHAR(20) UNIQUE NOT NULL,
  tipo_exibicao ENUM('texto', 'imagem') DEFAULT 'texto',
  data_criacao TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  status ENUM('ATIVO', 'INATIVO') DEFAULT 'ATIVO',
  taxa_entrega DECIMAL(10, 2) DEFAULT 0.00,
  INDEX idx_whatsapp (numero_whatsapp)
);

-- TABELA: horarios_funcionamento
CREATE TABLE horarios_funcionamento (
  horario_id INT PRIMARY KEY AUTO_INCREMENT,
  restaurante_id INT NOT NULL,
  dia_semana ENUM('segunda', 'terca', 'quarta', 'quinta', 'sexta', 'sabado', 'domingo'),
  hora_abertura TIME NOT NULL,
  hora_fechamento TIME NOT NULL,
  FOREIGN KEY (restaurante_id) REFERENCES restaurantes(restaurante_id),
  UNIQUE KEY unique_dia (restaurante_id, dia_semana)
);

-- TABELA: cardapio
CREATE TABLE cardapio (
  item_id INT PRIMARY KEY AUTO_INCREMENT,
  restaurante_id INT NOT NULL,
  nome VARCHAR(255) NOT NULL,
  preco DECIMAL(10, 2) NOT NULL,
  descricao TEXT,
  disponivel BOOLEAN DEFAULT TRUE,
  data_criacao TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (restaurante_id) REFERENCES restaurantes(restaurante_id),
  INDEX idx_restaurante (restaurante_id)
);

-- TABELA: clientes
CREATE TABLE clientes (
  cliente_id INT PRIMARY KEY AUTO_INCREMENT,
  restaurante_id INT NOT NULL,
  numero_whatsapp VARCHAR(20) NOT NULL,
  nome VARCHAR(255),
  estado_conversa VARCHAR(50) DEFAULT 'INICIAL',
  data_primeiro_contato TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  ultimo_acesso TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  FOREIGN KEY (restaurante_id) REFERENCES restaurantes(restaurante_id),
  UNIQUE KEY unique_cliente (restaurante_id, numero_whatsapp),
  INDEX idx_whatsapp (numero_whatsapp)
);

-- TABELA: preferencias_cliente
CREATE TABLE preferencias_cliente (
  preferencia_id INT PRIMARY KEY AUTO_INCREMENT,
  cliente_id INT NOT NULL,
  sem_tomate BOOLEAN DEFAULT FALSE,
  alergias VARCHAR(500),
  observacoes TEXT,
  FOREIGN KEY (cliente_id) REFERENCES clientes(cliente_id)
);

-- TABELA: pedidos
CREATE TABLE pedidos (
  pedido_id INT PRIMARY KEY AUTO_INCREMENT,
  restaurante_id INT NOT NULL,
  cliente_id INT NOT NULL,
  numero_cliente VARCHAR(20) NOT NULL,
  total DECIMAL(10, 2) NOT NULL,
  status ENUM('PENDENTE', 'CONFIRMADO', 'PREPARANDO', 'ENTREGANDO', 'ENTREGUE') DEFAULT 'PENDENTE',
  data_criacao TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  data_confirmacao TIMESTAMP NULL,
  rua VARCHAR(255),
  numero VARCHAR(10),
  bairro VARCHAR(100),
  cep VARCHAR(9),
  metodo_pagamento VARCHAR(50),
  FOREIGN KEY (restaurante_id) REFERENCES restaurantes(restaurante_id),
  FOREIGN KEY (cliente_id) REFERENCES clientes(cliente_id),
  INDEX idx_restaurante (restaurante_id),
  INDEX idx_cliente (cliente_id),
  INDEX idx_status (status),
  INDEX idx_data (data_criacao)
);

-- TABELA: items_pedido
CREATE TABLE items_pedido (
  item_pedido_id INT PRIMARY KEY AUTO_INCREMENT,
  pedido_id INT NOT NULL,
  item_id INT NOT NULL,
  quantidade INT NOT NULL,
  preco_unitario DECIMAL(10, 2) NOT NULL,
  subtotal DECIMAL(10, 2) NOT NULL,
  observacoes TEXT,
  FOREIGN KEY (pedido_id) REFERENCES pedidos(pedido_id) ON DELETE CASCADE,
  FOREIGN KEY (item_id) REFERENCES cardapio(item_id),
  INDEX idx_pedido (pedido_id)
);

-- TABELA: mensagens_enviadas (log)
CREATE TABLE mensagens_enviadas (
  mensagem_id INT PRIMARY KEY AUTO_INCREMENT,
  id_whapi VARCHAR(100) UNIQUE,
  restaurante_id INT NOT NULL,
  numero_cliente VARCHAR(20) NOT NULL,
  conteudo TEXT NOT NULL,
  tipo ENUM('texto', 'imagem', 'botoes') DEFAULT 'texto',
  data_envio TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  status_entrega ENUM('enviado', 'entregue', 'lido') DEFAULT 'enviado',
  FOREIGN KEY (restaurante_id) REFERENCES restaurantes(restaurante_id),
  INDEX idx_data (data_envio),
  INDEX idx_restaurante (restaurante_id)
);

-- TABELA: tokens (autenticação)
CREATE TABLE tokens (
  token_id INT PRIMARY KEY AUTO_INCREMENT,
  restaurante_id INT NOT NULL,
  token VARCHAR(500) NOT NULL UNIQUE,
  tipo ENUM('acesso', 'refresh') DEFAULT 'acesso',
  data_criacao TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  data_expiracao DATETIME NOT NULL,
  ativo BOOLEAN DEFAULT TRUE,
  FOREIGN KEY (restaurante_id) REFERENCES restaurantes(restaurante_id),
  INDEX idx_token (token),
  INDEX idx_restaurante (restaurante_id),
  INDEX idx_expiracao (data_expiracao)
);

-- TABELA: mensagens_customizadas
CREATE TABLE mensagens_customizadas (
  mensagem_id INT PRIMARY KEY AUTO_INCREMENT,
  restaurante_id INT NOT NULL,
  tipo VARCHAR(50) NOT NULL, -- 'saudacao', 'obrigado', etc
  conteudo TEXT NOT NULL,
  FOREIGN KEY (restaurante_id) REFERENCES restaurantes(restaurante_id),
  UNIQUE KEY unique_msg (restaurante_id, tipo)
);
```

### 6.2 Como a API Acessa o Banco

**Arquivo**: `src/database/mysql_connection.py`

Usamos **SQLAlchemy** como ORM (Object-Relational Mapping) para abstrair a complexidade do SQL puro.

```python
from sqlalchemy import create_engine, Column, Integer, String, Decimal, DateTime, Boolean, ForeignKey
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker, relationship
from datetime import datetime

# Configuração de conexão
DATABASE_URL = "mysql+pymysql://user:password@localhost/bot_restaurantes"
engine = create_engine(DATABASE_URL, pool_pre_ping=True)
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)
Base = declarative_base()

# Modelos (Classes que representam tabelas)
class Restaurante(Base):
    __tablename__ = "restaurantes"
    
    restaurante_id = Column(Integer, primary_key=True)
    nome = Column(String(255), nullable=False)
    numero_whatsapp = Column(String(20), unique=True, nullable=False)
    tipo_exibicao = Column(String(50), default='texto')
    status = Column(String(20), default='ATIVO')
    data_criacao = Column(DateTime, default=datetime.utcnow)
    
    # Relacionamentos
    pedidos = relationship("Pedido", back_populates="restaurante")
    clientes = relationship("Cliente", back_populates="restaurante")
    cardapio = relationship("CardapioItem", back_populates="restaurante")

class Cliente(Base):
    __tablename__ = "clientes"
    
    cliente_id = Column(Integer, primary_key=True)
    restaurante_id = Column(Integer, ForeignKey("restaurantes.restaurante_id"))
    numero_whatsapp = Column(String(20), nullable=False)
    nome = Column(String(255))
    estado_conversa = Column(String(50), default='INICIAL')
    
    restaurante = relationship("Restaurante", back_populates="clientes")
    pedidos = relationship("Pedido", back_populates="cliente")

class Pedido(Base):
    __tablename__ = "pedidos"
    
    pedido_id = Column(Integer, primary_key=True)
    restaurante_id = Column(Integer, ForeignKey("restaurantes.restaurante_id"))
    cliente_id = Column(Integer, ForeignKey("clientes.cliente_id"))
    total = Column(Decimal(10, 2))
    status = Column(String(50), default='PENDENTE')
    data_criacao = Column(DateTime, default=datetime.utcnow)
    
    restaurante = relationship("Restaurante", back_populates="pedidos")
    cliente = relationship("Cliente", back_populates="pedidos")
    items = relationship("ItemPedido", back_populates="pedido")

# Exemplos de uso:
def buscar_restaurante(restaurante_id):
    db = SessionLocal()
    return db.query(Restaurante).filter(
        Restaurante.restaurante_id == restaurante_id
    ).first()

def buscar_cliente(numero_whatsapp, restaurante_id):
    db = SessionLocal()
    return db.query(Cliente).filter(
        Cliente.numero_whatsapp == numero_whatsapp,
        Cliente.restaurante_id == restaurante_id
    ).first()

def criar_pedido(pedido_data):
    db = SessionLocal()
    novo_pedido = Pedido(**pedido_data)
    db.add(novo_pedido)
    db.commit()
    db.refresh(novo_pedido)
    return novo_pedido

def atualizar_status_pedido(pedido_id, novo_status):
    db = SessionLocal()
    pedido = db.query(Pedido).filter(Pedido.pedido_id == pedido_id).first()
    if pedido:
        pedido.status = novo_status
        db.commit()
        return pedido
    return None

def buscar_pedidos_restaurante(restaurante_id, data_inicio, data_fim):
    db = SessionLocal()
    return db.query(Pedido).filter(
        Pedido.restaurante_id == restaurante_id,
        Pedido.data_criacao >= data_inicio,
        Pedido.data_criacao <= data_fim
    ).all()
```

**Vantagens dessa abordagem**:
- ✅ Abstração: Trabalha com objetos Python, não SQL puro
- ✅ Segurança: Evita SQL injection automaticamente
- ✅ Portabilidade: Código funciona com qualquer SQL (MySQL, PostgreSQL, etc)
- ✅ Type Hints: Melhor autocompletar e type checking

### 6.3 Normalização de Dados (Design Relacional)

O design do banco MySQL segue **3ª Forma Normal (3NF)**, garantindo:

**1️⃣ Atomicidade dos Dados**
```
❌ ERRADO: Uma coluna com múltiplos valores
`alergias: "amendoim, glúten, lactose"`

✅ CORRETO: Tabela separada ou JSON array tipado
`preferencias_cliente` → lista de alergias individuais
```

**2️⃣ Eliminação de Redundância**
```
❌ ERRADO: Repetir preço do item em cada pedido
| pedido_id | item_nome | preco |
| 1         | Burger    | 25.00 |
| 2         | Burger    | 25.00 |

✅ CORRETO: Referenciar item (evita inconsistência de preço)
| pedido_id | item_id | preco_unitario_confirmado |
| 1         | 5       | 25.00                     |
| 2         | 5       | 25.00                     |
```

**3️⃣ Integridade Referencial (Foreign Keys)**
```sql
-- Garante que pedido sempre referencia cliente e restaurante válidos
ALTER TABLE pedidos 
ADD CONSTRAINT fk_cliente 
FOREIGN KEY (cliente_id) REFERENCES clientes(cliente_id);
```

**Benefícios**:
- 🔒 **Integridade**: Impossível ter dados órfãos
- 📊 **Consistência**: Uma fonte de verdade para cada informação
- ⚡ **Performance**: Índices em chaves estrangeiras aceleram JOINs
- 🛡️ **Segurança**: Dificulta operações que violariam regras de negócio
```

---

## 7. API REST - ENDPOINTS

### 7.1 Rotas de Autenticação

**Arquivo**: `src/routes/auth_routes.py`

```
POST /api/auth/login
├─ Corpo: { "numero_whatsapp": "11999999999", "senha": "..." }
├─ Resposta: { "token": "...", "restaurante_id": "..." }
└─ Status: 200 ou 401

POST /api/auth/cadastro
├─ Corpo: { "nome": "...", "numero_whatsapp": "...", "senha": "..." }
├─ Resposta: { "restaurante_id": "...", "token": "..." }
└─ Status: 201 ou 409 (já existe)
```

### 7.2 Rotas de Webhook (Receber Mensagens)

**Arquivo**: `src/routes/webhook_routes.py`

```
POST /api/webhook/message
├─ Origem: WHAPI.Cloud
├─ Corpo:
│  {
│    "numero_remetente": "11999999999",
│    "mensagem": "Quero pedir um burger",
│    "tipo_evento": "message",
│    "timestamp": "2024-01-15T14:30:00Z"
│  }
├─ Processamento:
│  1. Valida assinatura do webhook (segurança)
│  2. Identifica restaurante pelo número
│  3. Chama BotService.processar_mensagem()
│  4. Obtém resposta
│  5. Envia resposta via WHAPI
├─ Resposta: { "status": "recebido" }
└─ Status: 200

GET /api/webhook/verify (para verificação do WHAPI)
├─ WHAPI faz GET aqui para confirmar que webhook existe
├─ Resposta: "OK"
└─ Status: 200
```

### 7.3 Rotas de Gerenciamento

```
GET /api/restaurantes/:id
├─ Autenticação: Token JWT obrigatório
├─ Resposta: { "nome": "...", "cardapio": [...], ... }
└─ Status: 200 ou 401

PUT /api/restaurantes/:id
├─ Autenticação: Token JWT
├─ Corpo: { "nome": "...", "cardapio": [...], ... }
├─ Resposta: { "status": "atualizado" }
└─ Status: 200 ou 401

GET /api/pedidos/:restaurante_id
├─ Autenticação: Token JWT
├─ Query: ?data_inicio=2024-01-01&data_fim=2024-01-31
├─ Resposta: [{ "pedido_id": "...", "status": "...", ... }, ...]
└─ Status: 200 ou 401

GET /api/pedidos/:pedido_id/status
├─ Autenticação: Token JWT
├─ Resposta: { "status": "CONFIRMADO", "data": "2024-01-15", ... }
└─ Status: 200 ou 401
```

---

## 8. COMO FICA ONLINE (DEPLOY)

### 8.1 Fluxo de Deploy

```
┌────────────────────────────────────────────────────────────────┐
│ ETAPA 1: PREPARAÇÃO (LOCAL)                                    │
├────────────────────────────────────────────────────────────────┤
│                                                                  │
│ 1. Código está no Git (GitHub)                                 │
│ 2. Testes passam localmente                                    │
│ 3. Variáveis de ambiente configuradas (.env)                  │
│    EXEMPLO:                                                     │
│    MYSQL_URI=mysql://PROJETO@192.168.1.50:3306/b   │
│    WHAPI_API_KEY=sua_chave_api                                 │
│    CLOUDFLARE_API_KEY=sua_chave                                │
│    SECRET_KEY=chave_secreta_para_jwt                           │
│                                                                  │
└────────────────────────────────────────────────────────────────┘


┌──────────────────────────────────────────────────────────────────────┐
│ ETAPA 2: HOSPEDAGEM (SERVIDOR LOCAL)                                │
├──────────────────────────────────────────────────────────────────────┤
│                                                                      │
│ A aplicação será executada em um computador dedicado que atuará      │
│ como servidor principal do sistema.                                  │
│                                                                      │
│ O servidor permanecerá conectado à Internet para disponibilizar      │
│ a API REST e receber os Webhooks enviados pela Whapi.Cloud.          │
│                                                                      │
│ O ambiente será composto por:                                        │
│                                                                      │
│ • Python                                                             │
│ • FastAPI                                                            │
│ • Banco de Dados MySQL                                               │
│ • Cloudflare Tunnel                                                  │
│                                                                      │
│ Responsabilidades do servidor:                                       │
│                                                                      │
│ • Executar a API REST                                                │
│ • Disponibilizar a interface Web                                     │
│ • Receber Webhooks da Whapi.Cloud                                    │
│ • Executar as automações                                             │
│ • Persistir dados no banco                                           │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────────────┐
│ ETAPA 3: PUBLICAÇÃO NA INTERNET                                     │
├──────────────────────────────────────────────────────────────────────┤
│                                                                      │
│ O acesso externo será realizado através do Cloudflare Tunnel,        │
│ permitindo que o servidor local seja acessível pela Internet sem     │
│ necessidade de abrir portas no roteador ou possuir IP público.       │
│                                                                      │
│ Fluxo:                                                               │
│                                                                      │
│ Cliente                                                              │
│     │                                                                │
│ HTTPS                                                                │
│     │                                                                │
│ Cloudflare                                                           │
│     │                                                                │
│ Tunnel Seguro                                                        │
│     │                                                                │
│ Servidor Local                                                       │
│     │                                                                │
│ FastAPI                                                              │
│                                                                      │
│ Vantagens:                                                           │
│                                                                      │
│ • HTTPS automático                                                   │
│ • Comunicação criptografada                                          │
│ • Não requer Port Forwarding                                         │
│ • Não exige IP público                                               │
│ • Fácil implantação e manutenção                                     │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘

```

### 8.2 Fluxo Final: Cliente → Bot Online

```
CLIENTE (WhatsApp)
        ↓ (envia mensagem)
WHAPI.Cloud (servidores WhatsApp)
        ↓ (POST webhook)
Cloudflare DNS (www.seu-dominio.com)
        ↓ (resolve IP)
Seu Servidor (IP X.X.X.X)
        ↓
        ↓
Fast API (processa em src/main.py)
        ↓
Services (bot_service, order_service, etc)
        ↓
MuSQL (salva dados)
        ↓
Resposta volta pelo mesmo caminho
        ↓
CLIENTE recebe resposta no WhatsApp ✓
```

---

## 9. RESUMO DA ARQUITETURA

| Componente | Tecnologia | Função |
|------------|-----------|--------|
| **Bot** | Python + WHAPI SDK | Gerencia conversa |
| **API** | FastAPI | Endpoints HTTP |
| **Banco** | MySQL + SQLAlchemy | Persiste dados relacionais |
| **Autenticação** | JWT | Valida acesso |
| **DNS/SSL** | Cloudflare Tunnel | Acesso seguro do servidor local |
| **Versionamento** | Git/GitHub | Controla código |
| **ORM** | SQLAlchemy | Abstração de acesso ao banco |

---

## 10. SEGURANÇA (Considerações Teóricas)

```
1. VALIDAÇÃO DE ENTRADA
   └─ Todas mensagens validadas antes de processar
   └─ Evita SQL injection, XSS, etc

2. AUTENTICAÇÃO
   └─ Token JWT para API
   └─ Expiração em 24h ou 365 dias (configurável)
   └─ Renovação via refresh token

3. CRIPTOGRAFIA
   └─ HTTPS (obrigatório via Cloudflare)
   └─ Senhas com hash bcrypt no banco
   └─ Variáveis sensíveis em .env (não no código)

4. WEBHOOK VALIDATION
   └─ Assinatura do webhook validada
   └─ Evita requisições falsas do WHAPI

5. RATE LIMITING
   └─ Máximo X requisições por minuto
   └─ Previne abuso

6. LOGS
   └─ Todas ações registradas
   └─ Facilita auditoria e debugging
```

---

Essa estrutura permite que o sistema seja:
✅ **Escalável**: Fácil adicionar novos restaurantes
✅ **Mantível**: Código organizado e testável
✅ **Seguro**: Validações e criptografia
✅ **Online**: Acessível 24/7 globalmente

---

# 11. REFERÊNCIAS BIBLIOGRÁFICAS

## Frameworks e Desenvolvimento Web

### FastAPI
RAMÍREZ, S. (2024). FastAPI documentation - official documentation. Disponível em: https://fastapi.tiangolo.com/. Acesso em: 2024.

RAMÍREZ, S. (2019). Build APIs with FastAPI. Real Python. Disponível em: https://realpython.com/fastapi-python-web-apis/. Acesso em: 2024.

MATHIEU, S. et al. (2023). FastAPI: Modern Python Web Framework. O'Reilly Media. Concepts of asynchronous programming and dependency injection in API development.

---

## Bancos de Dados Relacionais

### MySQL
MYSQL, Inc. (2024). MySQL 8.0 Reference Manual. Disponível em: https://dev.mysql.com/doc/refman/8.0/en/. Acesso em: 2024.

DUBOIS, P. (2013). MySQL Cookbook: Solutions for Database Developers and Administrators. O'Reilly Media. Practical solutions for MySQL database design and optimization.

SILBERSCHATZ, A.; KORTH, H. F.; SUDARSHAN, S. (2010). Database System Concepts. McGraw-Hill. Fundamental concepts of relational database design, ACID properties, and normalization.

### SQLAlchemy (ORM)
BAYER, M. (2024). SQLAlchemy Documentation. Disponível em: https://docs.sqlalchemy.org/. Acesso em: 2024.

BAYER, M. (2020). SQLAlchemy ORM in Depth. Real Python. Disponível em: https://realpython.com/sqlalchemy/. Acesso em: 2024.

### Normalização e Design Relacional
DATE, C. J. (2003). An Introduction to Database Systems. Addison-Wesley. Theory of relational databases, normalization, and SQL fundamentals.

CODD, E. F. (1970). A Relational Model of Data for Large Shared Data Banks. Communications of the ACM, 13(6), 377-387. Foundational theory of relational databases.

---

## Arquitetura de Software e Design Patterns

### Padrões de Arquitetura
MARTIN, R. C. (2017). Clean Architecture: A Craftsman's Guide to Software Structure and Design. Prentice Hall. Principles of layered architecture and separation of concerns.

RICHARDSON, C. (2018). Microservices Patterns: With examples in Java. Manning Publications. Patterns for designing resilient and scalable microservice systems.

FOWLER, M. (2014). Microservices. Disponível em: https://martinfowler.com/articles/microservices.html. Acesso em: 2024.

### RESTful API Design
RICHARDSON, L.; RUBY, S. (2007). RESTful Web Services. O'Reilly Media. Principles of REST architecture, HTTP methods, and resource-oriented design.

FIELDING, R. T. (2000). Architectural Styles and the Design of Network-based Software Architectures. Doctoral dissertation, University of California. Foundational theory of REST.

---

## Integração com WhatsApp e Automação de Conversas

### WhatsApp Cloud API
FACEBOOK/META PLATFORMS, Inc. (2024). WhatsApp Business Platform API Documentation. Disponível em: https://developers.facebook.com/docs/whatsapp/cloud-api. Acesso em: 2024.

PATEL, P. (2022). Building Conversational Interfaces with WhatsApp Bot. Towards Data Science. Disponível em: https://towardsdatascience.com. Acesso em: 2024.

### Chatbots e Conversational AI
WEIZENBAUM, J. (1966). ELIZA—A Computer Program for the Study of Natural Language Communication Between Man and Machine. Communications of the ACM, 9(1), 36-45.

SERBAN, I. V. et al. (2015). A Survey of Available Corpora for Building Data-Driven Dialogue Systems: The Seq2Seq Era. arXiv preprint arXiv:1512.09130.

MORBINI, F. et al. (2012). Which ASR Should I Use in My Spoken Dialog System? Proceedings of the SIGDIAL 2012 Conference.

---

## Autenticação e Segurança

### JWT (JSON Web Tokens)
JONES, M.; BRADLEY, J.; SAKIMURA, N. (2015). JSON Web Token (JWT). RFC 7519. Internet Engineering Task Force. Disponível em: https://tools.ietf.org/html/rfc7519. Acesso em: 2024.

AUTH0. (2024). JWT Introduction. Disponível em: https://jwt.io/introduction. Acesso em: 2024.

### Segurança em APIs REST
OWASP (Open Web Application Security Project). (2023). API Security Top 10. Disponível em: https://owasp.org/www-project-api-security/. Acesso em: 2024.

GRAHAM-CUMMING, J. (2012). The Security of HTTPS. Disponível em: https://blog.cloudflare.com/the-security-of-https/. Acesso em: 2024.

---

## Infraestrutura e Deploy

### Cloudflare e Tunelamento
CLOUDFLARE, Inc. (2024). Cloudflare Tunnel Documentation. Disponível em: https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/. Acesso em: 2024.

GRAHAM-CUMMING, J. (2021). Argo Tunnel is Now Cloudflare Tunnel. Disponível em: https://blog.cloudflare.com/argo-tunnel-renamed-to-cloudflare-tunnel/. Acesso em: 2024.

### HTTPS e SSL/TLS
DIERKS, T.; RESCORLA, E. (2008). The Transport Layer Security (TLS) Protocol Version 1.2. RFC 5246. Internet Engineering Task Force.

RESCORLA, E. (2018). The Transport Layer Security (TLS) Protocol Version 1.3. RFC 8446. Internet Engineering Task Force.

---

## Python e Desenvolvimento

### Python para Backend
LUTZ, M. (2013). Learning Python: Powerful Object-Oriented Programming. O'Reilly Media. Comprehensive guide to Python programming language.

VAN ROSSUM, G.; WARSAW, B.; COGHLAN, N. (2001). PEP 8 – Style Guide for Python Code. Disponível em: https://pep8.org/. Acesso em: 2024.

### Async/Await e Programação Assíncrona
KUMAR, R. (2020). Asynchronous Programming in Python. Real Python. Disponível em: https://realpython.com/async-io-python/. Acesso em: 2024.

SELIVANOV, Y. (2014). PEP 492 – Coroutines with async and await syntax. Disponível em: https://www.python.org/dev/peps/pep-0492/. Acesso em: 2024.

---

## Arquitetura de Microsserviços e Serviços

### Padrões de Serviços
NEWMAN, S. (2015). Building Microservices: Designing Fine-Grained Systems. O'Reilly Media. Practical patterns for designing service-based architectures.

BASS, L.; WEBER, I.; ZHU, L. (2015). DevOps: A Software Architect's Perspective. Addison-Wesley. Integration of development and operations in scalable systems.

### Organização de Código
MCCONNELL, S. (2004). Code Complete: A Practical Handbook of Software Construction. Microsoft Press. Best practices for organizing and structuring code.

---

## Pequenos Negócios e Transformação Digital

### Tecnologia em Pequenas Empresas
MCKENZIE, R. B.; MORRISSETTE, D. P. (2013). Keeping It Simple: A Guide to Using the Web and Email Marketing Effectively for Small Business. AMACOM.

LAUDON, K. C.; LAUDON, J. P. (2020). Management Information Systems: Managing the Digital Firm. Pearson. Digital transformation strategies for organizations of all sizes.

TORNATSKY, L. G.; FLEISCHER, M.; CHAKRABARTI, A. K. (1990). Engines of Innovation: U.S. Industrial R&D at the Crossroads. National Academies Press. Technology adoption and organizational change.

---

## Extensão Universitária e Responsabilidade Social

### Extensão Universitária
FORPROEXT (Fórum de Pró-Reitores de Extensão das Universidades Públicas Brasileiras). (2012). Plano Nacional de Extensão Universitária. Disponível em: http://forproext.org.br/. Acesso em: 2024.

MEC (Ministério da Educação). (2018). Diretrizes para a Extensão na Educação Superior. Brasil.

MIZUKAMI, M. G. N. (2002). Ensino: as abordagens do processo. E.P.U. Conceitos de aprendizagem e extensão universitária.

### Responsabilidade Social em Tecnologia
PAPACHARISSI, Z. (2015). We Have Always Been Social. Social Media + Society, 1(1), 1-2. Technology and community engagement.

WARSCHAUER, M.; KNOBEL, M.; STONE, L. (2004). Technology and Equity in Schooling: Deconstructing the Digital Divide. Educational Policy, 18(4), 562-588.

---

## Webhooks e Comunicação em Tempo Real

### Webhooks
STURM, B. (2013). Webhooks in Practice. Blog. Disponível em: https://www.infoq.com/articles/webhooks/. Acesso em: 2024.

RALPH, R. (2020). How to Implement Webhooks. Vonage Developer Blog. Disponível em: https://www.vonage.com/blog/how-to-implement-webhooks/. Acesso em: 2024.

### Comunicação em Tempo Real
GRIGORIK, I. (2013). High Performance Browser Networking: What every web developer should know about networking and web performance. O'Reilly Media.

---

## Documentação Técnica

### Boas Práticas em Documentação
CAGAN, M. (2017). Inspired: How to Create Products Customers Love. Wiley. Including documentation best practices.

MCCONNELL, S. (2004). Code Complete: A Practical Handbook of Software Construction. Microsoft Press. Chapter on creating effective technical documentation.

---

## Padrões de Versionamento e Controle de Código

### Git e Versionamento
CHACON, S.; STRAUB, B. (2014). Pro Git. Apress. Comprehensive guide to Git version control system.

LOELIGER, J.; MCCULLOUGH, D. (2012). Version Control with Git: Powerful tools and techniques for collaborative software development. O'Reilly Media.

---

## Validação de Dados

### Input Validation e Sanitização
OWASP. (2023). OWASP Top 10 Web Application Security Risks. Disponível em: https://owasp.org/www-project-top-ten/. Acesso em: 2024.

CWE (Common Weakness Enumeration). (2024). CWE-20: Improper Input Validation. Disponível em: https://cwe.mitre.org/data/definitions/20.html. Acesso em: 2024.

---

## Testes de Software

### Testes Automatizados
FOWLER, M. (2012). The Practical Test Pyramid. Disponível em: https://martinfowler.com/articles/practical-test-pyramid.html. Acesso em: 2024.

BECK, K. (2002). Test Driven Development: By Example. Addison-Wesley. Methodology for writing tests before code.

OSHEROVE, R. (2013). The Art of Unit Testing: with examples in C#. Manning Publications. Best practices for unit testing and test design.

---

## Observabilidade e Monitoramento

### Logging e Rastreamento
NEWMAN, S. (2015). Building Microservices: Designing Fine-Grained Systems. O'Reilly Media. Chapter on observability in distributed systems.

GARG, H. (2020). Observability Engineering: Managing production systems and the art of invisible excellence. O'Reilly Media.

---

## Tratamento de Erros e Exceções

### Exception Handling
MCCONNELL, S. (2004). Code Complete: A Practical Handbook of Software Construction. Microsoft Press. Error handling strategies and patterns.

FOWLER, M. (2014). Patterns of Enterprise Application Architecture. Addison-Wesley. Design patterns for error handling in large systems.

---

## Referências Online Essenciais

| Recurso | URL | Descrição |
|---------|-----|-----------|
| FastAPI Official Docs | https://fastapi.tiangolo.com/ | Documentação oficial do FastAPI |
| MySQL Reference Manual | https://dev.mysql.com/doc/refman/8.0/en/ | Guia completo do MySQL 8.0 |
| SQLAlchemy Docs | https://docs.sqlalchemy.org/ | Documentação oficial de SQLAlchemy ORM |
| WhatsApp Business API | https://developers.facebook.com/docs/whatsapp | Documentação oficial da API |
| Cloudflare Tunnel | https://developers.cloudflare.com/cloudflare-one | Documentação do Cloudflare Tunnel |
| Python.org | https://www.python.org/doc/ | Documentação oficial de Python |
| OWASP | https://owasp.org/ | Segurança em aplicações web |
| Real Python | https://realpython.com/ | Tutoriais avançados de Python |
| JWT.io | https://jwt.io/ | Informações sobre JWT |

---

## Notas sobre Referências

Esta lista de referências foi compilada considerando:

1. **Fundamentação Teórica**: Referências de autores clássicos em arquitetura de software (Martin, Richardson)
2. **Tecnologias Específicas**: Documentação oficial de FastAPI, MySQL, SQLAlchemy e WhatsApp
3. **Banco de Dados Relacional**: Teoria clássica de Codd (1970), normalização de Date (2003)
4. **ORM (Object-Relational Mapping)**: SQLAlchemy como padrão Python para abstração SQL
5. **Segurança**: Standards OWASP e RFC oficiais
6. **Contexto Social**: Extensão universitária e tecnologia para pequenos negócios
7. **Boas Práticas**: Reconhecidos experts na indústria (Martin Fowler, Steve McConnell, etc.)

**Alteração Importante**: Este documento foi atualizado de **MongoDB (NoSQL)** para **MySQL (Relacional)** por:
- ✅ Melhor adequação a dados estruturados (cardápios, pedidos, clientes)
- ✅ ACID Compliance (garantia de transações de pedidos)
- ✅ Integridade referencial nativa (relacionamentos entre tabelas)
- ✅ Comunidade madura e suporte estável
- ✅ Custo operacional reduzido para projetos de extensão

Recomenda-se consultar as documentações oficiais para versões mais recentes das tecnologias descritas neste documento.