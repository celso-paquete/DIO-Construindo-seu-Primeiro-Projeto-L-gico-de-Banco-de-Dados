# DIO-Construindo-seu-Primeiro-Projeto-L-gico-de-Banco-de-Dados
 projeto prático focado na criação de um esquema lógico de banco de dados para o setor de e-commerce. scripts SQL detalhados, integrando elementos como chaves primárias, estrangeiras e refinamentos específicos para clientes, pagamentos e entregas.

```sql
-- =========================================================
-- 1. CRIAÇÃO DO BANCO DE DADOS
-- =========================================================

CREATE DATABASE IF NOT EXISTS ecommerce_db;
USE ecommerce_db;

-- =========================================================
-- 2. CRIAÇÃO DAS TABELAS
-- =========================================================

-- Tabela: clientes
CREATE TABLE clientes (
    id_cliente INT PRIMARY KEY AUTO_INCREMENT,
    tipo_cliente ENUM('PF', 'PJ') NOT NULL,

    -- Dados PF
    cpf VARCHAR(14) NULL UNIQUE,
    nome VARCHAR(100) NULL,
    data_nascimento DATE NULL,

    -- Dados PJ
    cnpj VARCHAR(18) NULL UNIQUE,
    razao_social VARCHAR(150) NULL,
    nome_fantasia VARCHAR(100) NULL,

    -- Dados comuns
    email VARCHAR(100) NOT NULL UNIQUE,
    telefone VARCHAR(20) NOT NULL,
    ativo BOOLEAN DEFAULT TRUE,
    data_cadastro TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

    CONSTRAINT chk_cliente_pj_pf CHECK (
        (tipo_cliente = 'PF' AND cpf IS NOT NULL AND cnpj IS NULL) OR
        (tipo_cliente = 'PJ' AND cnpj IS NOT NULL AND cpf IS NULL)
    )
);

-- Tabela: fornecedores
CREATE TABLE fornecedores (
    id_fornecedor INT PRIMARY KEY AUTO_INCREMENT,
    razao_social VARCHAR(150) NOT NULL,
    nome_fantasia VARCHAR(100),
    cnpj VARCHAR(18) NOT NULL UNIQUE,
    email VARCHAR(100) NOT NULL,
    telefone VARCHAR(20) NOT NULL,
    ativo BOOLEAN DEFAULT TRUE
);

-- Tabela: produtos
CREATE TABLE produtos (
    id_produto INT PRIMARY KEY AUTO_INCREMENT,
    id_fornecedor INT,
    nome VARCHAR(150) NOT NULL,
    sku VARCHAR(50) NOT NULL UNIQUE,
    preco DECIMAL(10, 2) NOT NULL,
    preco_custo DECIMAL(10, 2),
    categoria VARCHAR(80),
    marca VARCHAR(50),
    ativo BOOLEAN DEFAULT TRUE,

    FOREIGN KEY (id_fornecedor)
        REFERENCES fornecedores(id_fornecedor)
        ON DELETE SET NULL
        ON UPDATE CASCADE
);

-- Tabela: estoques
CREATE TABLE estoques (
    id_estoque INT PRIMARY KEY AUTO_INCREMENT,
    localizacao VARCHAR(100) NOT NULL,
    capacidade INT,
    ativo BOOLEAN DEFAULT TRUE
);

-- Tabela: produtos_estoque
CREATE TABLE produtos_estoque (
    id_produto INT NOT NULL,
    id_estoque INT NOT NULL,
    quantidade INT NOT NULL DEFAULT 0,

    PRIMARY KEY (id_produto, id_estoque),

    FOREIGN KEY (id_produto)
        REFERENCES produtos(id_produto)
        ON DELETE CASCADE
        ON UPDATE CASCADE,

    FOREIGN KEY (id_estoque)
        REFERENCES estoques(id_estoque)
        ON DELETE CASCADE
        ON UPDATE CASCADE
);

-- Tabela: pedidos
CREATE TABLE pedidos (
    id_pedido INT PRIMARY KEY AUTO_INCREMENT,
    id_cliente INT NOT NULL,
    status_pedido ENUM('pendente','confirmado','enviado','entregue','cancelado')
        DEFAULT 'pendente',
    data_pedido TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    valor_total DECIMAL(10, 2) NOT NULL,

    FOREIGN KEY (id_cliente)
        REFERENCES clientes(id_cliente)
        ON DELETE RESTRICT
        ON UPDATE CASCADE
);

-- Tabela: produtos_pedido
CREATE TABLE produtos_pedido (
    id_pedido INT NOT NULL,
    id_produto INT NOT NULL,
    quantidade INT NOT NULL DEFAULT 1,
    preco_unitario DECIMAL(10, 2) NOT NULL,

    PRIMARY KEY (id_pedido, id_produto),

    FOREIGN KEY (id_pedido)
        REFERENCES pedidos(id_pedido)
        ON DELETE CASCADE
        ON UPDATE CASCADE,

    FOREIGN KEY (id_produto)
        REFERENCES produtos(id_produto)
        ON DELETE RESTRICT
        ON UPDATE CASCADE
);

-- Tabela: pagamentos
CREATE TABLE pagamentos (
    id_pagamento INT PRIMARY KEY AUTO_INCREMENT,
    id_pedido INT NOT NULL,
    forma_pagamento ENUM('cartao_credito','boleto','pix','transferencia') NOT NULL,
    parcelas INT DEFAULT 1,
    valor DECIMAL(10, 2) NOT NULL,
    status_pagamento ENUM('pendente','aprovado','recusado') DEFAULT 'pendente',
    codigo_transacao VARCHAR(100),

    FOREIGN KEY (id_pedido)
        REFERENCES pedidos(id_pedido)
        ON DELETE CASCADE
        ON UPDATE CASCADE
);

-- Tabela: entregas
CREATE TABLE entregas (
    id_entrega INT PRIMARY KEY AUTO_INCREMENT,
    id_pedido INT NOT NULL UNIQUE,
    status_entrega ENUM('nao_iniciado','em_transito','entregue')
        DEFAULT 'nao_iniciado',
    codigo_rastreio VARCHAR(50) UNIQUE,
    transportadora VARCHAR(100),
    data_envio DATE,
    data_entrega DATE,

    FOREIGN KEY (id_pedido)
        REFERENCES pedidos(id_pedido)
        ON DELETE RESTRICT
        ON UPDATE CASCADE
);

-- =========================================================
-- 3. INSERTS (PERSISTÊNCIA DE DADOS)
-- =========================================================

INSERT INTO clientes (tipo_cliente, cpf, nome, data_nascimento, email, telefone) VALUES
('PF', '123.456.789-00', 'João Silva', '1985-03-15', 'joao@email.com', '(11) 99999-1111'),
('PF', '987.654.321-00', 'Maria Santos', '1990-07-22', 'maria@email.com', '(11) 98888-2222'),
('PF', '456.789.123-00', 'Pedro Oliveira', '1978-11-08', 'pedro@email.com', '(21) 97777-3333');

INSERT INTO clientes (tipo_cliente, cnpj, razao_social, nome_fantasia, email, telefone) VALUES
('PJ', '12.345.678/0001-99', 'Tech Solutions Ltda', 'Tech Solutions', 'tech@empresa.com', '(11) 3333-1111'),
('PJ', '98.765.432/0001-00', 'ABC Atacadista S/A', 'ABC Atacado', 'abc@empresa.com', '(11) 3333-2222');

INSERT INTO fornecedores (razao_social, nome_fantasia, cnpj, email, telefone) VALUES
('Eletrônica Brasil Ltda', 'EletroBrasil', '55.666.777/0001-88', 'vendas@eletrobrasil.com', '(11) 4444-1111'),
('Moveis Arte e Estilo', 'Móveis Arte', '66.777.888/0001-99', 'contato@moveisarteeestilo.com', '(11) 4444-2222'),
('Fashion Importações', 'Fashion Import', '77.888.999/0001-11', 'compras@fashionimport.com', '(21) 4444-3333');

INSERT INTO estoques (localizacao, capacidade) VALUES
('CD São Paulo', 10000),
('CD Rio de Janeiro', 8000);

INSERT INTO produtos (id_fornecedor, nome, sku, preco, preco_custo, categoria, marca) VALUES
(1, 'Smartphone Galaxy', 'ELE-SM-001', 2500.00, 1500.00, 'Eletrônicos', 'Samsung'),
(1, 'Notebook Dell', 'ELE-NT-001', 4500.00, 2800.00, 'Eletrônicos', 'Dell'),
(2, 'Sofá 3 Lugares', 'MOV-SF-001', 1800.00, 800.00, 'Móveis', 'Copenhagen'),
(2, 'Mesa de Jantar', 'MOV-MJ-001', 1200.00, 500.00, 'Móveis', 'Rome'),
(3, 'Jaqueta Couro', 'FAS-JC-001', 600.00, 250.00, 'Moda', 'Premium'),
(3, 'Tênis Nike', 'FAS-TN-001', 450.00, 200.00, 'Esportes', 'Nike');

INSERT INTO produtos_estoque (id_produto, id_estoque, quantidade) VALUES
(1, 1, 100), (2, 1, 50), (3, 1, 30), (4, 1, 25), (5, 1, 80), (6, 1, 120),
(1, 2, 40), (2, 2, 20), (3, 2, 15), (4, 2, 10), (5, 2, 30), (6, 2, 50);

-- =========================================================
-- 4. CONSULTAS
-- =========================================================

SELECT 
    c.id_cliente,
    IFNULL(c.nome, c.nome_fantasia) AS nome_cliente,
    COUNT(p.id_pedido) AS total_pedidos,
    ROUND(SUM(p.valor_total), 2) AS valor_total_gasto
FROM clientes c
LEFT JOIN pedidos p ON c.id_cliente = p.id_cliente
GROUP BY c.id_cliente, nome_cliente
ORDER BY total_pedidos DESC;

SELECT 
    p.id_produto,
    p.nome,
    SUM(pp.quantidade) AS total_vendido,
    ROUND(SUM(pp.quantidade * pp.preco_unitario), 2) AS receita
FROM produtos p
JOIN produtos_pedido pp ON p.id_produto = pp.id_produto
GROUP BY p.id_produto, p.nome
ORDER BY total_vendido DESC;

SELECT 
    nome,
    preco_custo AS custo,
    preco AS venda,
    ROUND(preco - preco_custo, 2) AS lucro_bruto,
    ROUND(((preco - preco_custo) / preco_custo * 100), 2) AS margem_percentual
FROM produtos
WHERE preco_custo IS NOT NULL
ORDER BY margem_percentual DESC;
```

