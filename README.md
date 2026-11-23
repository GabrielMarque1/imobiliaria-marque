# Imobiliária Marque — Análise de Dados

## Sobre o projeto

Projeto de análise de dados desenvolvido com o objetivo de explorar informações comerciais de uma imobiliária fictícia localizada em Passo Fundo, Rio Grande do Sul.
A proposta é utilizar ferramentas de análise de dados para organizar informações, acompanhar indicadores de desempenho e gerar insights que possam auxiliar na tomada de decisões do negócio.

## Objetivos

- Analisar o desempenho comercial da imobiliária.
- Acompanhar as vendas de imóveis.
- Avaliar o desempenho dos corretores.
- Identificar indicadores relevantes para o negócio.
- Desenvolver dashboards interativos para visualização dos resultados.

## Ferramentas utilizadas

- Microsoft Excel
- SQL
- Power BI
- DAX

## Estrutura dos Dados

O projeto utiliza seis tabelas principais para organizar as informações da imobiliária:

| Tabela | Descrição |
|---|---|
| tabImoveis | Informações e características dos imóveis cadastrados. |
| tabBairros | Identificação dos bairros e regiões da cidade. |
| tabClientes | Cadastro e informações dos clientes. |
| tabCorretores | Cadastro e informações dos corretores. |
| tabVendas | Registro das transações de venda dos imóveis. |
| tabAlugueis | Registro das operações de aluguel dos imóveis. |

As tabelas estão relacionadas no modelo de dados do Power BI, permitindo cruzar informações e desenvolver análises sobre as operações da imobiliária.

## Origem e Preparação dos Dados

Os dados utilizados no projeto foram desenvolvidos manualmente no Microsoft Excel, por meio da criação de seis tabelas com informações fictícias de uma imobiliária.

Após a estruturação das tabelas, os arquivos foram importados para o Power BI, onde foi desenvolvido o modelo de dados e estabelecidos os relacionamentos entre as tabelas.

### Ferramentas utilizadas nesta etapa

- Microsoft Excel: criação e organização das bases de dados.
- Power BI: importação dos dados e construção do modelo relacional.

### Natureza dos dados

Todas as informações utilizadas são fictícias e foram criadas exclusivamente para fins de estudo, prática e desenvolvimento do portfólio.

## Modelo de Dados e Relacionamentos

O modelo de dados foi desenvolvido no Power BI, conectando as seis tabelas por meio de seus respectivos identificadores.

### Relacionamentos

| Tabela de origem | Coluna de origem | Tabela relacionada | Coluna relacionada | Status |
|---|---|---|---|---|
| tabAlugueis | ID_Cliente | tabClientes | ID_Cliente | Inativo |
| tabAlugueis | ID_Corretor | tabCorretores | ID_Corretor | Ativo |
| tabAlugueis | ID_Imovel | tabImoveis | ID_Imovel | Ativo |
| tabImoveis | ID_Bairro | tabBairros | ID_Bairro | Ativo |
| tabImoveis | ID_Proprietario | tabClientes | ID_Cliente | Ativo |
| tabVendas | ID_Cliente | tabClientes | ID_Cliente | Inativo |
| tabVendas | ID_Corretor | tabCorretores | ID_Corretor | Ativo |
| tabVendas | ID_Imovel | tabImoveis | ID_Imovel | Ativo |

### Configurações dos relacionamentos

- Cardinalidade: muitos para um (*:1).
- Direção de filtro: única.
- Relacionamentos ativos: 6.
- Relacionamentos inativos: 2.

Os relacionamentos permitem conectar informações de imóveis, clientes, bairros e corretores às operações de venda e aluguel.

## Status do projeto
Em desenvolvimento.
