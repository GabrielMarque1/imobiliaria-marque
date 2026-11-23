# Imobiliária Marque — Medidas DAX

Este documento reúne as medidas DAX desenvolvidas no Power BI para analisar os indicadores comerciais, imobiliários e financeiros da Imobiliária Marque.

## 1. Indicadores de Aluguéis

### Contratos Ativos
```dax
Contratos Ativos =
    CALCULATE(
        COUNTROWS(tabAlugueis),
        tabAlugueis[Status] = "Ativo"
    )
```

### Receita Mensal de Aluguéis
```dax
Receita Mensal de Aluguéis =
    CALCULATE(
        SUM(tabAlugueis[Valor_Mensal]),
        tabAlugueis[Status] = "Ativo"
    )
```

### Receita Anual Projetada de Aluguéis
```dax
Receita Anual Projetada de Aluguéis =
    [Receita Mensal de Aluguéis] * 12
```

### Taxa de Administração Mensal
```dax
Taxa de Administração Mensal =
    CALCULATE(
        SUMX(
            tabAlugueis,
            tabAlugueis[Valor_Mensal] * tabAlugueis[Taxa_Administracao]
        ),
        tabAlugueis[Status] = "Ativo"
    )
```

### Receita Mensal Líquida de Administração
```dax
Receita Mensal Líquida de Administração =
    [Taxa de Administração Mensal]
```

### Receita Mensal Total da Imobiliaria
```dax
Receita Mensal Total da Imobiliaria =
    [Taxa de Administração Mensal]
```

### Taxa Média de Administração
```dax
Taxa Média de Administração =
    AVERAGE(tabAlugueis[Taxa_Administracao])
```

### Total de Contratos
```dax
Total de Contratos =
    COUNTROWS(tabAlugueis)
```

## 2. Indicadores de Clientes e Corretores

### Total de Clientes Cadastrados
```dax
Total de Clientes Cadastrados =
    DISTINCTCOUNT(tabClientes[ID_Cliente])
```

### Total de Corretores Ativos
```dax
Total de Corretores Ativos =
    CALCULATE(
        COUNTROWS(tabCorretores),
        tabCorretores[Status] = "Ativo"
    )
```

## 3. Indicadores de Imóveis

### Total de Imóveis Cadastrados
```dax
Total de Imóveis Cadastrados =
    COUNTROWS(tabImoveis)
```

### Imóveis Alugados
```dax
Imóveis Alugados =
    CALCULATE(
        COUNTROWS(tabImoveis),
        tabImoveis[Status] = "Alugado"
    )
```

### Imóveis Disponíveis
```dax
Imóveis Disponíveis =
    CALCULATE(
        COUNTROWS(tabImoveis),
        tabImoveis[Status] = "Disponível"
    )
```

### Imóveis Vendidos
```dax
Imóveis Vendidos =
    CALCULATE(
        COUNTROWS(tabImoveis),
        tabImoveis[Status] = "Vendido"
    )
```

### Percentual de Imóveis Disponíveis
```dax
Percentual de Imóveis Disponíveis =
    DIVIDE(
        [Imóveis Disponíveis],
        [Total de Imóveis Cadastrados],
        0
    )
```

### Percentual de Imóveis Vendidos
```dax
Percentual de Imóveis Vendidos =
    DIVIDE(
        [Imóveis Vendidos],
        [Total de Imóveis Cadastrados],
        0
    )
```

### Área Média dos Imóveis
```dax
Área Média dos Imóveis =
    AVERAGE(tabImoveis[Area_M2])
```

### Média de Quartos por Imóvel
```dax
Média de Quartos por Imóvel =
    AVERAGE(tabImoveis[Quartos])
```

### Valor Médio dos Imóveis
```dax
Valor Médio dos Imóveis =
    AVERAGE(tabImoveis[Valor_Anunciado])
```

## 4. Indicadores de Vendas

### Faturamento de Vendas
```dax
Faturamento de Vendas =
    SUM(tabVendas[Valor_Venda])
```

### Total de Vendas
```dax
Total de Vendas =
    COUNTROWS(tabVendas)
```

### Comissões Totais
```dax
Comissões Totais =
    SUM(tabVendas[Valor_Comissao])
```

### Receita Total de Comissões
```dax
Receita Total de Comissões =
    SUM(tabVendas[Valor_Comissao])
```

### Percentual Médio de Comissão
```dax
Percentual Médio de Comissão =
    AVERAGE(tabVendas[Comissao_Percentual])
```

### Taxa de Comissão Efetiva
```dax
Taxa de Comissão Efetiva =
    DIVIDE(
        [Comissões Totais],
        [Faturamento de Vendas],
        0
    )
```

### Ticket Médio de Venda
```dax
Ticket Médio de Venda =
    DIVIDE(
        [Faturamento de Vendas],
        [Total de Vendas],
        0
    )
```

### Valor Médio de Venda
```dax
Valor Médio de Venda =
    AVERAGE(tabVendas[Valor_Venda])
```

### Vendas por Corretor
```dax
Vendas por Corretor =
    COUNT(tabVendas[ID_Venda])
```

---

## Observações

As fórmulas acima representam as medidas desenvolvidas durante o projeto. Seus resultados dependem dos dados cadastrados e dos filtros aplicados no Power BI.
