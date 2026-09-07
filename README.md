# 🏫 Gestão de Infraestrutura de Racks - Bloco B (UNESC)

<p align="center">
  <img src="https://img.shields.io/badge/MySQL-005C84?style=for-the-badge&logo=mysql&logoColor=white" alt="MySQL"/>
  <img src="https://img.shields.io/badge/Database_Modeling-3FN-success?style=for-the-badge" alt="3FN"/>
  <img src="https://img.shields.io/badge/Status-Concluído-success?style=for-the-badge" alt="Status"/>
</p>

## 📖 Sobre o Projeto
Este repositório contém a modelagem, script SQL e documentação do banco de dados desenvolvido como **1ª Avaliação da disciplina de Desenvolvimento Back-End** do curso de Sistemas de Informação do UNESC.

A abstração escolhida para o trabalho foca no gerenciamento da infraestrutura física e lógica de rede do **Bloco B**, especificamente no rastreamento e manutenção dos **Racks de Rede** e dos ativos de TI neles contidos (Switches, Roteadores, Patch Panels).

## 👥 Equipe Desenvolvedora
* **Alhysson Francisco Saqueto**
* **José Roberto de Moura Vieira**
* **Luiz Paulo Chiaato**

---

## ⚙️ Arquitetura e Modelagem do Banco de Dados
Para garantir a máxima integridade dos dados e evitar anomalias de escrita, o modelo relacional foi construído seguindo os preceitos da **3ª Forma Normal (3FN)**, totalizando **7 tabelas**.

### Dicionário de Dados Resumido
1. `ambientes`: Mapeia a estrutura física do prédio (Salas, Corredores, Laboratórios).
2. `racks`: Os gabinetes físicos propriamente ditos distribuídos no bloco.
3. `categorias_equipamento`: Catálogo normalizado para os tipos de equipamento (Switch, Roteador).
4. `fabricantes`: Tabela normalizada para as marcas (Cisco, TP-Link, Furukawa).
5. `equipamentos_rack`: Dispositivos hospedados dentro de cada Rack.
6. `tecnicos`: Profissionais de TI da instituição responsáveis pela manutenção.
7. `manutencoes`: Histórico de intervenções e serviços executados na infraestrutura.

## 🚀 Como Executar
O script completo de criação das tabelas e inserção de dados fictícios está disponível neste repositório.
1. Baixe ou clone o projeto.
2. Importe o arquivo `script_racks_bloco_b.sql` em qualquer cliente MySQL (ex: DBeaver, MySQL Workbench, phpMyAdmin).
3. Execute o script para gerar a base `gestao_racks_bloco_b`.

---
*Trabalho acadêmico desenvolvido para a disciplina de Desenvolvimento Back-End - 2026.*
