# Imobiliária Marque — Medidas DAX

Este documento reúne as medidas DAX do modelo. Todas ficam na tabela `_Medidas`, organizadas nas mesmas pastas abaixo.
O código é o mesmo do arquivo `powerbi/Imobiliaria Marque.SemanticModel/definition/tables/_Medidas.tmdl`.

> **Data de referência:** os indicadores de situação (contratos ativos, contratos a vencer, estoque) são calculados no último mês com movimentação, para que o painel não mude sozinho com o passar do tempo.

## Sumário

- [0. Geral](#0-geral)
- [1. Vendas](#1-vendas)
- [2. Aluguéis](#2-aluguéis)
- [3. Imóveis](#3-imóveis)
- [4. Corretores e Clientes](#4-corretores-e-clientes)
- [5. Comparativos](#5-comparativos)
- [6. Rótulos e Cores](#6-rótulos-e-cores)

## 0. Geral

Referência de data usada pelos indicadores de situação.

### Data de Referência
```dax
Data de Referência =
    VAR _UltimaVenda = CALCULATE ( MAX ( fVendas[Data_Venda] ), REMOVEFILTERS () )
    VAR _UltimoContrato = CALCULATE ( MAX ( fAlugueis[Data_Inicio] ), REMOVEFILTERS () )
    RETURN
        EOMONTH ( MAX ( _UltimaVenda, _UltimoContrato ), 0 )
```

## 1. Vendas

Indicadores comerciais de vendas e metas.

### Faturamento
```dax
Faturamento =
    SUM ( fVendas[Valor_Venda] )
```

### Qtd Vendas
```dax
Qtd Vendas =
    COUNTROWS ( fVendas )
```

### Ticket Médio
```dax
Ticket Médio =
    DIVIDE ( [Faturamento], [Qtd Vendas] )
```

### Comissões
```dax
Comissões =
    SUM ( fVendas[Valor_Comissao] )
```

### Taxa de Comissão Efetiva
```dax
Taxa de Comissão Efetiva =
    DIVIDE ( [Comissões], [Faturamento] )
```

### Desconto Médio na Negociação
```dax
Desconto Médio na Negociação =
    VAR _Anunciado = SUMX ( fVendas, RELATED ( dImoveis[Valor_Anunciado] ) )
    RETURN
        IF ( _Anunciado > 0, 1 - DIVIDE ( [Faturamento], _Anunciado ) )
```

### Tempo Médio de Venda (dias)
```dax
Tempo Médio de Venda (dias) =
    AVERAGEX (
        fVendas,
        DATEDIFF ( RELATED ( dImoveis[Data_Cadastro] ), fVendas[Data_Venda], DAY )
    )
```

### Meta de Faturamento
```dax
Meta de Faturamento =
    VAR _Ref = [Data de Referência]
    RETURN
        CALCULATE ( SUM ( fMetas[Meta_Faturamento] ), KEEPFILTERS ( dCalendario[Data] <= _Ref ) )
```

### % da Meta Atingida
```dax
% da Meta Atingida =
    DIVIDE ( [Faturamento], [Meta de Faturamento] )
```

### Preço Médio do m²
```dax
Preço Médio do m² =
    CALCULATE (
        DIVIDE ( [Faturamento], SUMX ( fVendas, RELATED ( dImoveis[Area_M2] ) ) ),
        dImoveis[Tipo_Imovel] <> "Terreno"
    )
```

### Participação no Faturamento
```dax
Participação no Faturamento =
    DIVIDE ( [Faturamento], CALCULATE ( [Faturamento], REMOVEFILTERS ( dCorretores ) ) )
```

## 2. Aluguéis

Indicadores da carteira de locação (situação na data de referência).

### Contratos Ativos
```dax
Contratos Ativos =
    CALCULATE ( COUNTROWS ( fAlugueis ), fAlugueis[Status] = "Ativo", REMOVEFILTERS ( dCalendario ) )
```

### Receita Mensal de Aluguéis
```dax
Receita Mensal de Aluguéis =
    CALCULATE (
        SUM ( fAlugueis[Valor_Mensal] ),
        fAlugueis[Status] = "Ativo",
        REMOVEFILTERS ( dCalendario )
    )
```

### Taxa de Administração Mensal
```dax
Taxa de Administração Mensal =
    CALCULATE (
        SUMX ( fAlugueis, fAlugueis[Valor_Mensal] * fAlugueis[Taxa_Administracao] ),
        fAlugueis[Status] = "Ativo",
        REMOVEFILTERS ( dCalendario )
    )
```

### Repasse aos Proprietários
```dax
Repasse aos Proprietários =
    [Receita Mensal de Aluguéis] - [Taxa de Administração Mensal]
```

### Aluguel Médio
```dax
Aluguel Médio =
    DIVIDE ( [Receita Mensal de Aluguéis], [Contratos Ativos] )
```

### Novos Contratos
```dax
Novos Contratos =
    COUNTROWS ( fAlugueis )
```

### Imóveis para Locação
```dax
Imóveis para Locação =
    CALCULATE (
        COUNTROWS ( dImoveis ),
        dImoveis[Finalidade] IN { "Aluguel", "Venda e Aluguel" },
        dImoveis[Status] <> "Vendido"
    )
```

### Taxa de Ocupação
```dax
Taxa de Ocupação =
    DIVIDE ( [Imóveis Alugados], [Imóveis para Locação] )
```

### Contratos a Vencer (90 dias)
```dax
Contratos a Vencer (90 dias) =
    VAR _Ref = [Data de Referência]
    RETURN
        CALCULATE (
            COUNTROWS ( fAlugueis ),
            fAlugueis[Status] = "Ativo",
            fAlugueis[Data_Fim] > _Ref,
            fAlugueis[Data_Fim] <= _Ref + 90,
            REMOVEFILTERS ( dCalendario )
        ) + 0
```

### Receita a Vencer (90 dias)
```dax
Receita a Vencer (90 dias) =
    VAR _Ref = [Data de Referência]
    RETURN
        CALCULATE (
            SUM ( fAlugueis[Valor_Mensal] ),
            fAlugueis[Status] = "Ativo",
            fAlugueis[Data_Fim] > _Ref,
            fAlugueis[Data_Fim] <= _Ref + 90,
            REMOVEFILTERS ( dCalendario )
        )
```

### Dias para Vencer
```dax
Dias para Vencer =
    VAR _Ref = [Data de Referência]
    VAR _Fim = MAX ( fAlugueis[Data_Fim] )
    RETURN
        IF (
            HASONEVALUE ( fAlugueis[ID_Aluguel] )
                && SELECTEDVALUE ( fAlugueis[Status] ) = "Ativo"
                && _Fim > _Ref && _Fim <= _Ref + 90,
            DATEDIFF ( _Ref, _Fim, DAY )
        )
```

### Data de Vencimento
```dax
Data de Vencimento =
    IF ( NOT ISBLANK ( [Dias para Vencer] ), MAX ( fAlugueis[Data_Fim] ) )
```

### Aluguel Mensal do Contrato
```dax
Aluguel Mensal do Contrato =
    IF ( NOT ISBLANK ( [Dias para Vencer] ), SUM ( fAlugueis[Valor_Mensal] ) )
```

## 3. Imóveis

Indicadores da carteira e do estoque de imóveis.

### Imóveis Cadastrados
```dax
Imóveis Cadastrados =
    COUNTROWS ( dImoveis )
```

### Imóveis em Carteira
```dax
Imóveis em Carteira =
    CALCULATE ( COUNTROWS ( dImoveis ), dImoveis[Status] <> "Vendido" )
```

### Imóveis Disponíveis
```dax
Imóveis Disponíveis =
    CALCULATE ( COUNTROWS ( dImoveis ), dImoveis[Status] = "Disponível" )
```

### Imóveis Alugados
```dax
Imóveis Alugados =
    CALCULATE ( COUNTROWS ( dImoveis ), dImoveis[Status] = "Alugado" )
```

### Imóveis Vendidos
```dax
Imóveis Vendidos =
    CALCULATE ( DISTINCTCOUNT ( fVendas[ID_Imovel] ), REMOVEFILTERS ( dCalendario ) )
```

### Imóveis à Venda
```dax
Imóveis à Venda =
    CALCULATE (
        COUNTROWS ( dImoveis ),
        dImoveis[Status] = "Disponível",
        dImoveis[Finalidade] IN { "Venda", "Venda e Aluguel" }
    )
```

### Valor Médio Anunciado
```dax
Valor Médio Anunciado =
    AVERAGE ( dImoveis[Valor_Anunciado] )
```

### Dias em Estoque
```dax
Dias em Estoque =
    VAR _Ref = [Data de Referência]
    RETURN
        AVERAGEX (
            FILTER (
                dImoveis,
                dImoveis[Status] = "Disponível"
                    && dImoveis[Finalidade] IN { "Venda", "Venda e Aluguel" }
            ),
            DATEDIFF ( dImoveis[Data_Cadastro], _Ref, DAY )
        )
```

### Imóveis Parados +180 dias
```dax
Imóveis Parados +180 dias =
    VAR _Ref = [Data de Referência]
    RETURN
        COUNTROWS (
            FILTER (
                dImoveis,
                dImoveis[Status] = "Disponível"
                    && dImoveis[Finalidade] IN { "Venda", "Venda e Aluguel" }
                    && DATEDIFF ( dImoveis[Data_Cadastro], _Ref, DAY ) > 180
            )
        ) + 0
```

### Valor Anunciado em Estoque
```dax
Valor Anunciado em Estoque =
    IF (
        HASONEVALUE ( dImoveis[ID_Imovel] ) && NOT ISBLANK ( [Dias em Estoque] ),
        SUM ( dImoveis[Valor_Anunciado] )
    )
```

### Imóveis Cadastrados no Período
```dax
Imóveis Cadastrados no Período =
    CALCULATE ( [Imóveis Cadastrados], USERELATIONSHIP ( dImoveis[Data_Cadastro], dCalendario[Data] ) )
```

## 4. Corretores e Clientes

Indicadores de equipe e base de clientes.

### Corretores Ativos
```dax
Corretores Ativos =
    CALCULATE ( COUNTROWS ( dCorretores ), dCorretores[Status] = "Ativo" )
```

### Novos Clientes
```dax
Novos Clientes =
    CALCULATE ( COUNTROWS ( dClientes ), USERELATIONSHIP ( dClientes[Data_Cadastro], dCalendario[Data] ) )
```

## 5. Comparativos

Comparação com o mesmo período do ano anterior. Cada indicador tem a versão "AA" (ano anterior) e a variação percentual; abaixo, o padrão usado, com Faturamento como exemplo.

### Faturamento AA
```dax
Faturamento AA =
    VAR _Ref = [Data de Referência]
    RETURN
        IF (
            HASONEVALUE ( dCalendario[Ano] ),
            IF (
                MIN ( dCalendario[Data] ) > _Ref,
                CALCULATE ( [Faturamento], SAMEPERIODLASTYEAR ( dCalendario[Data] ) ),
                CALCULATE ( [Faturamento], SAMEPERIODLASTYEAR ( dCalendario[Data] ), dCalendario[Data] <= EDATE ( _Ref, -12 ) )
            )
        )
```

### Var % Faturamento
```dax
Var % Faturamento =
    VAR _Atual = [Faturamento]
    VAR _Anterior = [Faturamento AA]
    RETURN
        IF ( NOT ISBLANK ( _Anterior ) && NOT ISBLANK ( _Atual ), DIVIDE ( _Atual - _Anterior, _Anterior ) )
```

## 6. Rótulos e Cores

Medidas de texto e cor usadas nos cartões (setas ▲▼ e formatação condicional). Abaixo, os exemplos principais.

### Rótulo Var Faturamento
```dax
Rótulo Var Faturamento =
    VAR _Var = [Var % Faturamento]
    RETURN
        SWITCH (
            TRUE (),
            NOT HASONEVALUE ( dCalendario[Ano] ), "Selecione um ano para comparar",
            ISBLANK ( _Var ), "Sem base no ano anterior",
            IF ( _Var >= 0, "▲ ", "▼ " ) & FORMAT ( ABS ( _Var ), "0.0%" ) & " vs " & ( SELECTEDVALUE ( dCalendario[Ano] ) - 1 )
        )
```

### Cor Var Faturamento
```dax
Cor Var Faturamento =
    VAR _Var = [Var % Faturamento]
    RETURN
        SWITCH ( TRUE (), ISBLANK ( _Var ), "#8A9AB8", _Var >= 0, "#1E9E5A", "#D64545" )
```

### Rótulo Atualização
```dax
Rótulo Atualização =
    "Dados até " & FORMAT ( [Data de Referência], "dd/mm/yyyy" )
```

### Rótulo Meta
```dax
Rótulo Meta =
    VAR _Meta = [Meta de Faturamento]
    RETURN
        IF ( ISBLANK ( _Meta ), "Sem meta cadastrada", "Meta: R$ " & FORMAT ( _Meta / 1000000, "#,0.0" ) & " mi" )
```
