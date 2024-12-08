# Banco de Dados: db_unaspizzas

Este banco de dados foi criado para gerenciar um sistema de pedidos de pizzas. Ele abrange tabelas para usuários, categorias de pizzas, pizzas, status de pedidos, pedidos e itens dos pedidos, com relações bem definidas entre elas.

---

## Estrutura do Banco de Dados

### 1. **Tabela `users`**
Armazena os dados dos clientes, como nome, e-mail e endereço.

### 2. **Tabela `pizza_category`**
Classifica as pizzas em categorias (ex.: Tradicional, Gourmet, Doce).

### 3. **Tabela `pizza`**
Armazena as informações das pizzas, como nome, descrição, preço e a categoria a que pertencem.

- **Relação**: Cada pizza pertence a uma categoria por meio de `id_pizza_category`.

### 4. **Tabela `status_order`**
Define os possíveis estados de um pedido, como "Pendente", "Preparando", e "Finalizado".

### 5. **Tabela `order_app`**
Representa os pedidos realizados pelos usuários, incluindo o cliente (`id_user`), o status do pedido (`id_status_order`), o preço total, e as datas de criação e atualização.

- **Relação**: 
  - Um pedido pertence a um usuário (`id_user` → `users`).
  - Um pedido tem um status associado (`id_status_order` → `status_order`).

### 6. **Tabela `order_pizza`**
Relaciona os pedidos com as pizzas e suas quantidades.

- **Relações**:
  - Cada registro associa um pedido (`id_order` → `order_app`) com uma pizza (`id_pizza` → `pizza`).

---

## Relações Entre Tabelas

- **Usuários (`users`)** podem fazer **pedidos (`order_app`)**.
- **Pedidos (`order_app`)** têm um **status (`status_order`)** e contêm uma ou mais **pizzas (`pizza`)** por meio da tabela intermediária **`order_pizza`**.
- **Pizzas (`pizza`)** são categorizadas por **categorias de pizzas (`pizza_category`)**.

---

## Como Executar o Script

1. Abra o cliente MySQL e crie o banco de dados:
   ```sql
   CREATE DATABASE db_unaspizzas;
   ```

2. Execute o script .sql para criar as tabelas e inserir os dados iniciais:
  ```sql

  -- Tabela para armazenar informações dos usuários
  CREATE TABLE users (
  id_user INT NOT NULL AUTO_INCREMENT, -- Identificador único do usuário
  name VARCHAR(100) NOT NULL,          -- Nome do usuário
  email VARCHAR(100) NOT NULL,         -- E-mail do usuário
  address VARCHAR(200) NOT NULL,       -- Endereço do usuário

  PRIMARY KEY (id_user)                -- Chave primária
  );

  -- Tabela para categorias de pizza
  CREATE TABLE pizza_category (
  id_pizza_category INT NOT NULL AUTO_INCREMENT, -- Identificador único da categoria de pizza
  name_pizza_category VARCHAR(100) NOT NULL,     -- Nome da categoria de pizza

  PRIMARY KEY (id_pizza_category)                -- Chave primária
  );

  -- Inserção de categorias de pizza
  INSERT INTO pizza_category (name_pizza_category) VALUES
  ('Tradicional'), -- Pizzas tradicionais
  ('Gourmet'),     -- Pizzas sofisticadas
  ('Doce');        -- Pizzas doces

  -- Tabela para armazenar informações das pizzas
  CREATE TABLE pizza (
  id_pizza INT NOT NULL AUTO_INCREMENT,               -- Identificador único da pizza
  id_pizza_category INT NOT NULL,                     -- Chave estrangeira para a categoria da pizza
  name VARCHAR(100) NOT NULL,                         -- Nome da pizza
  description VARCHAR(200) NOT NULL,                  -- Descrição da pizza
  image LONGTEXT NOT NULL,                            -- Imagem da pizza (armazenada como base64)
  price FLOAT NOT NULL,                               -- Preço da pizza

  PRIMARY KEY (id_pizza),                             -- Chave primária
  KEY fk_pizza_pizza_category (id_pizza_category),    -- Índice para chave estrangeira

  CONSTRAINT fk_pizza_pizza_category FOREIGN KEY (id_pizza_category)
    REFERENCES pizza_category (id_pizza_category)     -- Restrição de integridade referencial
  );

  -- Tabela para os diferentes status de pedidos
  CREATE TABLE status_order (
  id_status_order INT NOT NULL AUTO_INCREMENT, -- Identificador único do status
  name_status VARCHAR(100) NOT NULL,           -- Nome do status
  PRIMARY KEY (id_status_order)                -- Chave primária
  );

  -- Inserção de status de pedidos
  INSERT INTO status_order (name_status) VALUES
  ('Pendente'),          -- Pedido criado, aguardando confirmação
  ('Preparando'),        -- Pedido em preparação
  ('Aguardando entrega'),-- Pedido pronto e aguardando ser aceito para entrega
  ('Em rota de entrega'),-- Pedido em rota para o cliente
  ('Finalizado');        -- Pedido entregue e finalizado

  -- Tabela principal para armazenar os pedidos realizados
  CREATE TABLE order_app (
  id_order INT NOT NULL AUTO_INCREMENT,       -- Identificador único do pedido
  id_user INT DEFAULT NULL,                   -- Chave estrangeira para o usuário que fez o pedido
  id_status_order INT DEFAULT NULL,           -- Chave estrangeira para o status do pedido
  total_price DECIMAL(10,2) NOT NULL,         -- Preço total do pedido
  created_at DATE NOT NULL,                   -- Data de criação do pedido
  updated_at DATE DEFAULT NULL,               -- Data de última atualização do pedido

  PRIMARY KEY (id_order),                     -- Chave primária
  KEY fk_order_user (id_user),                -- Índice para chave estrangeira do usuário
  KEY fk_order_status (id_status_order),      -- Índice para chave estrangeira do status

  CONSTRAINT fk_order_status FOREIGN KEY (id_status_order)
    REFERENCES status_order (id_status_order),-- Restrição de integridade para status
  CONSTRAINT fk_order_user FOREIGN KEY (id_user)
    REFERENCES users (id_user)                -- Restrição de integridade para usuários
  );

  -- Tabela intermediária para relacionar pedidos e pizzas
  CREATE TABLE order_pizza (
  id_order INT NOT NULL,                      -- Chave estrangeira para o pedido
  id_pizza INT NOT NULL,                      -- Chave estrangeira para a pizza
  quantity INT NOT NULL,                      -- Quantidade da pizza no pedido

  PRIMARY KEY (id_order, id_pizza),           -- Chave primária composta
  KEY fk_order_pizza_pizza (id_pizza),        -- Índice para chave estrangeira da pizza

  CONSTRAINT fk_order_pizza_order FOREIGN KEY (id_order)
    REFERENCES order_app (id_order),          -- Restrição de integridade para pedidos
  CONSTRAINT fk_order_pizza_pizza FOREIGN KEY (id_pizza)
    REFERENCES pizza (id_pizza)               -- Restrição de integridade para pizzas
  );

  ```

3. Verifique se todas as tabelas foram criadas:

  ```sql
  SHOW TABLES;
  ```

### Observações
- O modelo foi projetado para permitir pedidos com múltiplas pizzas e diferentes quantidades.
- As imagens das pizzas são armazenadas no formato Base64.
- O sistema está normalizado para evitar redundância de dados.