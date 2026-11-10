# Bases de dados

| Pasta | Conteúdo |
|---|---|
| `./` (raiz) | Base sintética **tratada** |
| `m2/` | Base **real** da M2 Locadora de Motos, anonimizada |

## Base sintética tratada

As colunas são as mesmas dos arquivos originais (a versão com erros está no commit inicial). Datas ficam em `dd/mm/aaaa`, números usam ponto decimal e as categorias foram padronizadas.

Principais decisões:
- **Removidos:** linhas duplicadas e locações/manutenções cuja moto ou cliente não existe no cadastro.
- **Cliente cadastrado duas vezes** (mesmo CPF e nome): os cadastros foram unificados e as locações passaram para o ID mais antigo.
- **Erros de digitação:** valores negativos voltaram a ser positivos. Valores multiplicados por 100 foram divididos por 100. `999999`, `N/D`, CPF `000.000.000-00` e telefone `99999-9999` viraram nulos.
- **Regra de negócio** confirmada nos próprios dados: `valor_total = valor_diaria × dias locados`. Ela foi usada para recalcular totais vazios ou incoerentes.
- **Custos de manutenção extremos** (acima de Q3 + 3·IQR do mesmo tipo de serviço) foram trocados pela mediana daquele tipo.
- **Outros nulos** foram preenchidos só quando havia uma regra defensável. Os demais continuam nulos.

## Base M2 (dados reais)

O layout é o mesmo da base sintética, mais o arquivo extra `pagamentos.csv`, com cada cobrança recebida.

**Fontes:** CRLVs das motos, contratos de locação, extratos Asaas/Stone (05/2025 a 04/2026) e boletins de ocorrência.

**Anonimização (LGPD):**
- O nome fica só com o primeiro nome, seguido de `XXX`.
- CPF, data de nascimento e telefone foram substituídos por `XXX`.
- Do e-mail, só o domínio foi mantido.
- Cidade/UF e as placas foram mantidas. As placas pertencem à empresa (pessoa jurídica).

**Limitações** (a informação não existe nos documentos, então o campo ficou vazio):
- `valor_aquisicao`, `km_atual`, `km_retirada` e `km_devolucao`.
- Em `manutencoes`, não há `id_moto`, `tipo`, `km_moto` nem `dias_parada`: o extrato bancário só informa a oficina e o valor.

**Regras usadas:**
- `data_devolucao_real` é a primeira destas datas: furto/roubo, início do contrato seguinte com a mesma moto, ou último pagamento + 7 dias (quando os pagamentos param bem antes do fim dos extratos).
- `valor_total = valor_diaria × dias locados`, com a mesma regra da base sintética. Para locações em andamento, conta-se até 10/11/2026. A receita efetivamente recebida está em `pagamentos.csv`.
- `status_pagamento` compara o que foi recebido com o que era devido (caução + semanas) no período coberto pelos extratos. Para contratos posteriores a 03/2026, o valor é `Não informado`.
