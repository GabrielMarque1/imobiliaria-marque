# Imobiliária Marque — Análise de Dados

<img width="1280" height="720" alt="capa" src="https://github.com/user-attachments/assets/7461ec76-3593-42da-8a10-9fc7ba5e94e0" />


## Sobre o projeto

Projeto de análise de dados desenvolvido para explorar as informações comerciais de uma imobiliária fictícia em Passo Fundo, Rio Grande do Sul.
A proposta é usar ferramentas de análise de dados para organizar informações, acompanhar indicadores de desempenho e gerar insights que ajudem na tomada de decisões do negócio.

> ⚠️ A Imobiliária Marque é uma empresa fictícia. Todos os dados foram criados para fins de estudo e portfólio.

<!-- Dica: grave um GIF navegando pelo painel e coloque aqui -->
<!-- ![Demonstração](docs/demo.gif) -->

## Perguntas de negócio

| Pergunta | Página |
|---|---|
| Estamos vendendo mais do que no ano passado? Estamos batendo a meta? | Visão Geral |
| Quais corretores geram mais resultado e quais precisam de apoio? | Vendas e Corretores |
| Quanto tempo um imóvel leva para vender e quanto desconto damos na negociação? | Vendas e Corretores |
| Quanto vale o m² em cada região da cidade? | Imóveis e Aluguéis |
| Quais imóveis estão parados há muito tempo? | Imóveis e Aluguéis |
| Qual a ocupação da carteira de locação e quais contratos vencem em breve? | Imóveis e Aluguéis |
| O que fazer com essas informações? | Conclusões |

## Principais insights (jan–out/2026)

- Faturamento de **R$ 21,6 mi, +33,3%** sobre o mesmo período de 2025, puxado pelo volume (50 vendas contra 37); o ticket médio ficou estável.
- **102% da meta** acumulada atingida; janeiro é o mês mais fraco (62% da meta).
- **8 imóveis à venda há mais de 180 dias** (R$ 3,4 mi anunciados), enquanto um imóvel vendido leva em média 68 dias.
- **75% de ocupação** na locação; **6 contratos vencem em 90 dias** (R$ 12,1 mil/mês a renovar).

## Ferramentas utilizadas

- Microsoft Excel: criação e organização das bases de dados.
- Power BI Desktop (formato PBIP): modelo de dados, relatório e visualizações.
- Power Query (M): importação e tipagem dos dados.
- DAX: medidas, tabela calendário e inteligência de tempo.
- Git/GitHub: versionamento do projeto.

## Estrutura dos dados

O projeto usa oito tabelas, organizadas em **esquema estrela**: as tabelas de fatos registram os acontecimentos (vendas, aluguéis, metas) e as dimensões descrevem quem, o quê, onde e quando.

| Tabela no modelo | Tabela no Excel | Tipo | Descrição |
|---|---|---|---|
| fVendas | tabVendas | Fato | Vendas de imóveis: data, valor, comissão, corretor e comprador. |
| fAlugueis | tabAlugueis | Fato | Contratos de aluguel: início, fim, valor mensal, taxa de administração e status. |
| fMetas | tabMetas | Fato | Meta mensal de faturamento. |
| dImoveis | tabImoveis | Dimensão | Imóveis cadastrados e suas características. |
| dBairros | tabBairros | Dimensão | Bairros e regiões da cidade. |
| dClientes | tabClientes | Dimensão | Clientes: proprietários, compradores e inquilinos. |
| dCorretores | tabCorretores | Dimensão | Corretores da imobiliária. |
| dCalendario | (criada em DAX) | Dimensão | Tabela calendário marcada como tabela de datas. |

Todas as medidas ficam numa tabela separada, `_Medidas`.

## Origem e preparação dos dados

Os dados foram criados manualmente no Microsoft Excel, em tabelas com informações fictícias de uma imobiliária. A base cobre **jan/2025 a out/2026**, com 157 imóveis, 96 vendas, 51 contratos de aluguel, 280 clientes e 8 corretores.

Durante a revisão foram corrigidas algumas inconsistências da base:

- Vendas em que o comprador era o próprio proprietário do imóvel.
- Um contrato de aluguel em que o inquilino era o proprietário.
- Preço de venda e valor de aluguel na mesma coluna (`Valor_Anunciado`), o que distorcia a média de valor dos imóveis. O aluguel anunciado passou para a coluna `Valor_Aluguel_Anunciado`.
- Nome de coluna com espaço (`Tem _Area_Jogos` → `Tem_Area_Jogos`).

## Tratamento e transformação dos dados

No Power Query:

- Conferência e definição dos tipos de dados (texto, número inteiro, decimal e data).
- Parâmetro **`CaminhoExcel`**: todas as consultas leem o arquivo a partir dele, então trocar a fonte exige mudar um único campo.
- Tabelas renomeadas no modelo para o padrão de fatos (`f`) e dimensões (`d`).

## Modelo de dados e relacionamentos

<img width="1600" height="900" alt="modelo-de-dados" src="https://github.com/user-attachments/assets/99fea2b4-a78b-4840-86fd-490cd0fb6a4d" />


| Tabela de origem | Coluna de origem | Tabela relacionada | Coluna relacionada | Status |
|---|---|---|---|---|
| fVendas | ID_Imovel | dImoveis | ID_Imovel | Ativo |
| fVendas | ID_Corretor | dCorretores | ID_Corretor | Ativo |
| fVendas | ID_Cliente | dClientes | ID_Cliente | Ativo |
| fVendas | Data_Venda | dCalendario | Data | Ativo |
| fAlugueis | ID_Imovel | dImoveis | ID_Imovel | Ativo |
| fAlugueis | ID_Corretor | dCorretores | ID_Corretor | Ativo |
| fAlugueis | ID_Cliente | dClientes | ID_Cliente | Ativo |
| fAlugueis | Data_Inicio | dCalendario | Data | Ativo |
| fMetas | Mes_Referencia | dCalendario | Data | Ativo |
| dImoveis | ID_Bairro | dBairros | ID_Bairro | Ativo |
| fAlugueis | Data_Fim | dCalendario | Data | Inativo |
| dImoveis | ID_Proprietario | dClientes | ID_Cliente | Inativo |
| dImoveis | Data_Cadastro | dCalendario | Data | Inativo |
| dClientes | Data_Cadastro | dCalendario | Data | Inativo |
| dCorretores | Data_Contratacao | dCalendario | Data | Inativo |

### Configurações dos relacionamentos

- Cardinalidade: muitos para um (*:1).
- Direção de filtro: única.
- Os relacionamentos inativos são ativados nas medidas que precisam deles com `USERELATIONSHIP` (por exemplo, contratos encerrados por data de fim e novos clientes por data de cadastro).
- A data/hora automática do Power BI foi desativada; todas as análises de tempo usam a `dCalendario`.

## Destaques técnicos (DAX)

- **Comparação justa com o ano anterior:** `SAMEPERIODLASTYEAR` limitado à data de referência, para não comparar jan–out de um ano com o ano anterior inteiro.
- **Data de referência:** os indicadores de situação são calculados no último mês com dados, então o painel não "envelhece".
- **Rótulos e cores dinâmicos:** medidas de texto (`▲ 33,3% vs 2025`) e medidas que retornam a cor usada na formatação condicional.
- **Métricas de negócio:** tempo médio de venda, desconto na negociação, preço do m² (sem terrenos), taxa de ocupação e receita a renovar.

## Documentação técnica

- [Medidas DAX](medidas-dax.md): fórmulas e descrições das medidas, por pasta.
- [Dashboards](dashboards.md): páginas, indicadores e visualizações do relatório.

## Como abrir o projeto

1. Instale o Power BI Desktop (versão recente, com suporte a projetos PBIP).
2. Baixe ou clone este repositório.
3. Abra `powerbi/Imobiliaria Marque.pbip`.
4. Em **Página Inicial → Transformar dados → Editar parâmetros**, informe em `CaminhoExcel` o caminho completo do arquivo `dados/Imobiliaria Marque.xlsx` no seu computador.
5. Clique em **Atualizar**.

## Estrutura do repositório

```
powerbi/
  Imobiliaria Marque.pbip              → abre o projeto no Power BI Desktop
  Imobiliaria Marque.Report/           → páginas e visuais (formato PBIR)
  Imobiliaria Marque.SemanticModel/    → modelo, medidas DAX e consultas (formato TMDL)
dados/
  Imobiliaria Marque.xlsx              → base de dados fictícia
docs/                                  → imagens usadas na documentação
medidas-dax.md                         → documentação das medidas
dashboards.md                          → documentação das páginas
```

No formato PBIP o projeto fica salvo em arquivos de texto, então é possível ler aqui no GitHub o código de cada medida e ver o histórico de alterações.

## Status do projeto

Versão 2 concluída: modelo em esquema estrela, tabela calendário, comparações com o ano anterior, metas e página de conclusões.

Próximos passos:
- Layout para celular.
- Filtros por bairro, tipo de imóvel e corretor.
- Publicação no Power BI Service.

## Autor

**Gabriel Marque** · [LinkedIn](https://www.linkedin.com/in/gabriel-oliveira-de-marque/)
