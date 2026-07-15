# 📊 Projeto Analytics E-Commerce Olist: Logística, Clientes & Simulador de SLA

![Badge em Desenvolvimento](http://img.shields.io/static/v1?label=STATUS&message=CONCLUÍDO&color=GREEN&style=for-the-badge)
![Power BI](https://img.shields.io/badge/PowerBI-F2C811?style=for-the-badge&logo=Power%20BI&logoColor=black)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-316192?style=for-the-badge&logo=postgresql&logoColor=white)
![DAX](https://img.shields.io/badge/DAX-Advanced-purple?style=for-the-badge)

## 📝 Sobre o Projeto
Este projeto foi desenvolvido unindo a minha bagagem prática de mais de 6 anos na área de Logística e Administração com técnicas avançadas de Análise de Dados. O objetivo foi transformar a base de dados pública de e-commerce da **Olist (2017-2018)** em um ecossistema analítico estratégico para tomada de decisão executiva.

Em vez de criar apenas um painel de visualização estático, desenvolvi um **Simulador de Cenários Multivariável (What-If)**, permitindo que gestores simulem metas futuras de eficiência logística e enxerguem o impacto real no indicador global da companhia.

---

## 🛠️ Stack Técnica
*   **Banco de Dados:** PostgreSQL (Engenharia de Dados, queries de junção, limpeza e modelagem inicial).
*   **Ferramenta de BI:** Power BI Desktop.
*   **Linguagem de Modelagem:** DAX Avançado (Uso de variáveis `VAR`, funções de contexto `CALCULATE`, e tratamento seguro de parâmetros com `SELECTEDVALUE`).
*   **Design de Interface:** Figma (Desenho de backgrounds personalizados focados em UX/UI minimalista).

---

## 🎯 Problema de Negócio & Desafios Logísticos
A operação de grandes e-commerces enfrenta desafios diários com prazos de entrega, fretes abusivos em regiões periféricas e baixas taxas de fidelização. As principais perguntas que este projeto buscou responder foram:
1. Como a performance de entrega afeta a retenção e recompra de clientes?
2. Onde estão os gargalos de SLA que puxam a média nacional para baixo?
3. Como prever o impacto global de melhorias logísticas regionais antes de investir recursos?

---

## 📈 Insights Estratégicos Extraídos
*   **O Paradoxo de São Paulo:** SP concentra mais de 64% do faturamento da empresa (R$ 8,7 Mi), mas seu SLA de entrega operacional patina em **91%**, ficando abaixo da média nacional (92%). Por ter um peso massivo, a ineficiência de SP prejudica o resultado do Brasil inteiro.
*   **O Gargalo da Expansão:** Regiões Norte e Nordeste registram faturamentos baixíssimos porque o valor médio do frete ultrapassa **35% do Ticket Médio** do produto, inviabilizando a conversão.
*   **O Balde Furado da Retenção:** Apesar do faturamento bruto expressivo (R$ 13,54 Mi), a taxa de recorrência de clientes é de apenas **3,02%**. O cruzamento de dados prova que a demora na entrega (média de 12 dias) e de despacho (3 dias) sabota a fidelização.

---

## 🎛️ Diferencial Técnico: O Simulador Multivariável (What-If)
O grande destaque do projeto é a tela de simulação. Utilizando parâmetros numéricos decimais e lógica DAX ponderada, o painel permite que o usuário movimente sliders de SLA para **São Paulo, Rio de Janeiro e Minas Gerais** de forma independente. 

A fórmula recalcula dinamicamente a média ponderada nacional, provando matematicamente para a diretoria que focar esforços de melhoria em estados menores gera pouco resultado se o gargalo de São Paulo não for resolvido.

```dax
// Exemplo da lógica utilizada para a consolidação ponderada dos parâmetros
SLA_Geral_Simulado_Multivariavel = 
VAR PedidosTotal = [Total Pedidos]
VAR PedidosSP = CALCULATE([Total Pedidos], dim_clientes[customer_state] = "SP")
VAR PedidosRJ = CALCULATE([Total Pedidos], dim_clientes[customer_state] = "RJ")
VAR PedidosMG = CALCULATE([Total Pedidos], dim_clientes[customer_state] = "MG")

VAR PesoSP = DIVIDE(PedidosSP, PedidosTotal, 0)
VAR PesoRJ = DIVIDE(PedidosRJ, PedidosTotal, 0)
VAR PesoMG = DIVIDE(PedidosMG, PedidosTotal, 0)
VAR PesoResto = 1 - (PesoSP + PesoRJ + PesoMG)

VAR SLARestoBrasil = 
    CALCULATE(
        [% SLA Entrega], 
        dim_clientes[customer_state] <> "SP" && 
        dim_clientes[customer_state] <> "RJ" && 
        dim_clientes[customer_state] <> "MG"
    )

VAR SLA_SP_Simulado = SELECTEDVALUE('Meta SLA SP'[Meta SLA SP], 0.91)
VAR SLA_RJ_Simulado = SELECTEDVALUE('Meta SLA RJ'[Meta SLA RJ], 0.90)
VAR SLA_MG_Simulado = SELECTEDVALUE('Meta SLA MG'[Meta SLA MG], 0.92)

RETURN
    (SLA_SP_Simulado * PesoSP) + 
    (SLA_RJ_Simulado * PesoRJ) + 
    (SLA_MG_Simulado * PesoMG) + 
    (SLARestoBrasil * PesoResto)
