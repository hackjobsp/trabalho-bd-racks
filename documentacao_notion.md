# Trabalho de Desenvolvimento Back-End: Gestão de Infraestrutura de Racks do Bloco B

---

## 1. Introdução e Contextualização do Problema

O **Bloco B do Centro Universitário do Espírito Santo (UNESC)** é uma estrutura física de grande importância, abrigando diversas dependências fundamentais para a rotina acadêmica, como salas de aula, laboratórios de informática, corredores de alto fluxo e espaços administrativos. Para que todos esses ambientes tenham conectividade à internet e acesso aos sistemas da instituição, existe uma complexa rede de equipamentos de TI distribuída pelo prédio.

A abstração escolhida para este trabalho foca especificamente no **gerenciamento físico e lógico da Infraestrutura de Rede**, mais especificamente no controle dos **Racks de Rede**. 

### O que é um Rack de Rede?
Um "Rack" é um gabinete metálico (podendo ser de piso ou fixado na parede) projetado para organizar, abrigar e proteger equipamentos de telecomunicação e conectividade, como switches, roteadores, patch panels, servidores e cabeamento. 

### O Desafio
Em instituições de grande porte, é comum que a equipe técnica perca o rastreio de onde cada equipamento está instalado, qual rack sofreu manutenção recentemente ou quais marcas de switch estão em determinado laboratório. Portanto, o objetivo deste banco de dados é **mapear exatamente onde os Racks estão fisicamente localizados no Bloco B**, quais equipamentos estão dentro deles e manter um registro rigoroso das manutenções realizadas pela equipe de TI.

---

## 2. Justificativa da Modelagem e Normalização

O banco de dados relacional foi projetado para garantir máxima integridade referencial, escalabilidade e evitar redundância de dados. O projeto final é composto por **7 tabelas**, superando o requisito mínimo de 5 tabelas, graças à aplicação de técnicas de **Normalização de Banco de Dados**.

### O Diagrama Entidade-Relacionamento (DER)

A modelagem visual (Diagrama Entidade-Relacionamento) foi desenvolvida utilizando a 3ª Forma Normal (3FN). Nela, mapeamos a estrutura física do Bloco B através das tabelas de `ambientes` e `racks`. Para evitar anomalias de atualização e garantir consistência na gestão dos ativos, os atributos dos equipamentos foram isolados em tabelas satélites (`categorias_equipamento` e `fabricantes`), que se relacionam de forma (1:N) com a tabela central `equipamentos_rack`. Por fim, implementamos a tabela de `manutencoes` para criar um histórico rastreável das intervenções feitas pelos `tecnicos` na infraestrutura.

### Por que 7 tabelas e não 5? (Normalização)
Inicialmente, poderíamos ter apenas uma tabela de "Equipamentos" onde o tipo (Switch, Roteador) e a marca (Cisco, Dell) seriam escritos manualmente em formato de texto. Isso geraria problemas conhecidos como anomalias de inserção e atualização (ex: um técnico escrever "TP-Link", outro escrever "TPLink" e outro "TP Link"). 

Para garantir a confiabilidade dos dados (aplicando as Formas Normais), nós extraímos essas informações repetitivas para tabelas próprias:
* A tabela **`categorias_equipamento`** funciona como um catálogo padronizado dos tipos de equipamentos.
* A tabela **`fabricantes`** armazena as marcas e seus contatos de suporte de forma única.

Dessa forma, a tabela principal `equipamentos_rack` apenas se relaciona com essas entidades menores usando **Chaves Estrangeiras (Foreign Keys)**, formando um banco de dados extremamente robusto e profissional.

---

## 3. Dicionário de Dados e Descrição Detalhada das Tabelas

O banco de dados foi nomeado como **`gestao_racks_bloco_b`** e sua estrutura está detalhada abaixo:

### 3.1. Tabela `ambientes`
Representa a arquitetura física do Bloco B. É a tabela base que mapeia os espaços físicos reais da instituição.
* **`id_ambiente` (INT, PK)**: Identificador único, gerado automaticamente (Auto Increment).
* **`nome` (VARCHAR)**: O nome amigável ou numeração do local (Exemplo: "Laboratório de Redes", "Corredor 2º Andar", "Sala 101"). Campo obrigatório (NOT NULL).
* **`tipo` (VARCHAR)**: Classifica a natureza do ambiente, facilitando relatórios (Ex: Sala, Laboratório, Corredor, Auditório).
* **`andar` (INT)**: Indica em qual pavimento o ambiente se encontra, otimizando a localização física para os técnicos.

### 3.2. Tabela `racks`
Representa os gabinetes físicos propriamente ditos que estão instalados nos ambientes.
* **`id_rack` (INT, PK)**: Identificador único do rack.
* **`identificacao` (VARCHAR)**: Nome ou patrimônio que vem colado no rack físico (Ex: "Rack-Mini-Corr1"). 
* **`tamanho_us` (INT)**: Medida padrão internacional de altura de racks de TI, chamada de Unidade de Rack ("U"). Determina a capacidade de equipamentos que cabem nele.
* **`data_instalacao` (DATE)**: Registra quando a infraestrutura foi montada no Bloco B.
* **`id_ambiente` (INT, FK)**: Chave estrangeira que conecta este rack à tabela `ambientes`. Responde à pergunta: *"Onde este rack está instalado?"*

### 3.3. Tabela `categorias_equipamento`
Tabela auxiliar de normalização que define a taxonomia dos dispositivos de rede.
* **`id_categoria` (INT, PK)**: Chave primária.
* **`nome` (VARCHAR)**: Nome da categoria (Ex: "Switch", "Patch Panel", "Servidor").
* **`descricao` (VARCHAR)**: Breve explicação técnica sobre a função daquele tipo de equipamento na rede do UNESC.

### 3.4. Tabela `fabricantes`
Tabela auxiliar de normalização para padronizar as marcas compradas pela instituição.
* **`id_fabricante` (INT, PK)**: Chave primária.
* **`nome` (VARCHAR)**: Nome da empresa fabricante (Ex: "Cisco", "Intelbras", "Furukawa").
* **`suporte_contato` (VARCHAR)**: Telefone ou e-mail de contato do fabricante, muito útil para os técnicos acionarem garantias em caso de queima de equipamento.

### 3.5. Tabela `equipamentos_rack`
É o coração do sistema, mapeando todos os dispositivos eletrônicos distribuídos pelos racks.
* **`id_equipamento` (INT, PK)**: Identificador único do ativo de TI.
* **`id_rack` (INT, FK)**: Chave estrangeira apontando para o rack físico onde a peça foi parafusada.
* **`id_categoria` (INT, FK)**: Chave estrangeira que busca na tabela de categorias o que é este equipamento.
* **`id_fabricante` (INT, FK)**: Chave estrangeira que busca a marca na tabela de fabricantes.
* **`quantidade_portas` (INT)**: Informação vital para equipamentos de rede. Indica quantas conexões físicas (cabos) aquele equipamento suporta (Ex: Um switch de 24 ou 48 portas). Pode ser NULL para equipamentos que não possuem portas, como um no-break ou servidor simples.

### 3.6. Tabela `tecnicos`
Representa o recurso humano. Mapeia a equipe de infraestrutura de TI do UNESC.
* **`id_tecnico` (INT, PK)**: Identificador único do funcionário.
* **`nome` (VARCHAR)**: Nome completo do profissional.
* **`matricula` (VARCHAR)**: Registro institucional do funcionário. Possui restrição de unicidade (`UNIQUE`), garantindo que não existam dois funcionários cadastrados com a mesma matrícula.
* **`email` (VARCHAR)**: E-mail de contato corporativo do técnico.

### 3.7. Tabela `manutencoes`
Uma tabela de transações e histórico. Sempre que alguém for fisicamente até um rack no Bloco B fazer um conserto, uma linha é gerada aqui.
* **`id_manutencao` (INT, PK)**: Chave primária do chamado técnico.
* **`id_rack` (INT, FK)**: Qual rack apresentou problema ou recebeu melhoria.
* **`id_tecnico` (INT, FK)**: Qual funcionário (técnico) executou o serviço. Garante a rastreabilidade e responsabilidade da equipe.
* **`data_servico` (DATE)**: Data em que a intervenção física ocorreu.
* **`descricao` (TEXT)**: Relatório textual com detalhes do que foi feito (Ex: "Troca de ventoinha do rack, organização do cabeamento e crimpagem de novos patch cords").

---

## 4. Conclusão

A estrutura criada respeita todas as diretrizes do trabalho avaliativo. O modelo atinge alto grau de organização lógica, integridade relacional através do uso adequado de Chaves Primárias (PK) e Chaves Estrangeiras (FK), e representa com precisão um cenário absolutamente real da infraestrutura de tecnologia presente em blocos educacionais modernos como o Bloco B.

---

## 5. Script SQL do Projeto

Abaixo encontra-se o código-fonte SQL para geração completa da estrutura de dados e inserção de registros fictícios para homologação.

CREATE DATABASE IF NOT EXISTS gestao_racks_bloco_b;
USE gestao_racks_bloco_b;

CREATE TABLE ambientes (
    id_ambiente INT AUTO_INCREMENT PRIMARY KEY,
    nome VARCHAR(100) NOT NULL,
    tipo VARCHAR(50) NOT NULL,
    andar INT NOT NULL
);

CREATE TABLE racks (
    id_rack INT AUTO_INCREMENT PRIMARY KEY,
    identificacao VARCHAR(50) NOT NULL,
    tamanho_us INT NOT NULL,
    data_instalacao DATE,
    id_ambiente INT NOT NULL,
    FOREIGN KEY (id_ambiente) REFERENCES ambientes(id_ambiente)
);

CREATE TABLE categorias_equipamento (
    id_categoria INT AUTO_INCREMENT PRIMARY KEY,
    nome VARCHAR(50) NOT NULL,
    descricao VARCHAR(255)
);

CREATE TABLE fabricantes (
    id_fabricante INT AUTO_INCREMENT PRIMARY KEY,
    nome VARCHAR(100) NOT NULL,
    suporte_contato VARCHAR(100)
);

CREATE TABLE equipamentos_rack (
    id_equipamento INT AUTO_INCREMENT PRIMARY KEY,
    id_rack INT NOT NULL,
    id_categoria INT NOT NULL,
    id_fabricante INT NOT NULL,
    quantidade_portas INT,
    FOREIGN KEY (id_rack) REFERENCES racks(id_rack),
    FOREIGN KEY (id_categoria) REFERENCES categorias_equipamento(id_categoria),
    FOREIGN KEY (id_fabricante) REFERENCES fabricantes(id_fabricante)
);

CREATE TABLE tecnicos (
    id_tecnico INT AUTO_INCREMENT PRIMARY KEY,
    nome VARCHAR(100) NOT NULL,
    matricula VARCHAR(20) UNIQUE NOT NULL,
    email VARCHAR(100)
);

CREATE TABLE manutencoes (
    id_manutencao INT AUTO_INCREMENT PRIMARY KEY,
    id_rack INT NOT NULL,
    id_tecnico INT NOT NULL,
    data_servico DATE NOT NULL,
    descricao TEXT NOT NULL,
    FOREIGN KEY (id_rack) REFERENCES racks(id_rack),
    FOREIGN KEY (id_tecnico) REFERENCES tecnicos(id_tecnico)
);

INSERT INTO ambientes (nome, tipo, andar) VALUES 
('Corredor Principal', 'Corredor', 1),
('Laboratório de Redes', 'Laboratório', 2),
('Sala dos Servidores', 'Sala', 1);

INSERT INTO racks (identificacao, tamanho_us, data_instalacao, id_ambiente) VALUES 
('Rack-Mini-Corr1', 5, '2023-05-10', 1),
('Rack-Piso-LabRedes', 44, '2022-02-15', 2),
('Rack-Core-Servidores', 44, '2021-10-01', 3);

INSERT INTO categorias_equipamento (nome, descricao) VALUES 
('Switch', 'Equipamento de interconexão de rede local'),
('Patch Panel', 'Painel de conexão para organização de cabos'),
('Roteador', 'Equipamento de roteamento entre redes'),
('Servidor', 'Computador de alto desempenho para serviços');

INSERT INTO fabricantes (nome, suporte_contato) VALUES 
('Cisco', 'suporte@cisco.com'),
('Furukawa', '0800-furukawa'),
('MikroTik', 'support@mikrotik.com'),
('TP-Link', 'suporte.br@tp-link.com'),
('Dell', '0800-dell');

INSERT INTO equipamentos_rack (id_rack, id_categoria, id_fabricante, quantidade_portas) VALUES 
(1, 1, 1, 24), 
(1, 2, 2, 24), 
(2, 3, 3, 10), 
(2, 1, 4, 48), 
(3, 4, 5, NULL);

INSERT INTO tecnicos (nome, matricula, email) VALUES 
('Carlos Silva', 'TI-1001', 'carlos.silva@unesc.br'),
('Ana Paula', 'TI-1002', 'ana.paula@unesc.br');

INSERT INTO manutencoes (id_rack, id_tecnico, data_servico, descricao) VALUES 
(1, 1, '2024-01-15', 'Organização de cabos e troca de patch cord defeituoso.'),
(2, 2, '2024-03-20', 'Atualização de firmware do switch principal.'),
(3, 1, '2024-08-05', 'Limpeza preventiva dos coolers do servidor.');

