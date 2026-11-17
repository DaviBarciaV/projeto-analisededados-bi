# Fontes de dados, perguntas de negócio e indicadores

**Projeto:** Análise de dados e BI de uma locadora de motos (filiais João Pessoa e São Paulo)
**Grupo:** Ismael Victor, Davi Barcia, Daniel Rozendo, Slann Antonio
**Data de referência das consultas:** 10/11/2026

**Objetivo geral:** identificar, a partir dos dados de motos, locações, manutenções e clientes, onde a frota gera e onde perde resultado. Com isso, recomendar ações para aumentar a ocupação, reduzir custos de manutenção e diminuir as perdas de receita. O dashboard compara cada resultado com o mercado atual de locação e de motocicletas.

---

## 1. Identificação das fontes de dados

| # | Fonte | Tipo | Onde está no repositório | Uso no projeto |
|---|---|---|---|---|
| F1 | **Base hidratada com IA** (locadora simulada, 2 filiais) | Sintética, gerada por IA | `database/motos.csv`, `clientes.csv`, `locacoes.csv`, `manutencoes.csv` | Base principal do dashboard e das 5 primeiras perguntas |
| F2 | **M2 Locadora de Motos** (empresa real, São Paulo/SP) | Documentos internos da empresa | `database/m2/` (5 CSVs anonimizados) | Validação com dados reais: preço, inadimplência e furto/roubo |
| F3 | **ABLA – Anuário Brasileiro do Setor de Locação de Veículos 2026** (dados de 2025) | Publicação setorial | `database/mercado/benchmarks_mercado.csv` | Tamanho e crescimento do setor, frota de motos em locadoras, idade da frota |
| F4 | **SENATRAN – Frota de Veículos 2026** (setembro/2026) | Dado oficial (Ministério dos Transportes) | `database/mercado/senatran_frota_motos_uf.csv` e `senatran_frota_motos_municipios_sp_pb.csv` | Frota de motos por UF e município: demanda potencial e penetração |
| F5 | **Fenabrave / Abraciclo** (emplacamentos 2025 e projeção 2026) | Dado setorial, citado pela imprensa | `benchmarks_mercado.csv` | Crescimento do mercado e mix de modelos |
| F6 | **SSP-SP** (roubos e furtos de motos 2025, compilados pela FECAP) | Dado oficial de segurança pública | `benchmarks_mercado.csv` | Taxa de furto/roubo de referência |
| F7 | **Preços de mercado** (BluStar/Yamaha, Mottu) e **Tabela FIPE** | Imprensa especializada e FIPE | `benchmarks_mercado.csv` | Preço da locação e valor de reposição da moto |

Cada número de mercado tem fonte, URL, período de referência e a pergunta de negócio a que se relaciona no arquivo `database/mercado/benchmarks_mercado.csv`.

---

## 2. Dados originais e método de obtenção

### F1 – Base hidratada com IA
- **Origem:** gerada com IA generativa (Claude) a partir de um prompt que descrevia uma locadora de motos com filiais em João Pessoa e São Paulo. O prompt pedia modelos, planos (diário, semanal e mensal), formas de pagamento, oficinas e regras de negócio realistas.
- **Dados originais (antes da limpeza):** estão no commit inicial do repositório (`af35416`, pasta `database/`).
  - Volume: 123 motos, 720 clientes, 4.278 locações e 1.377 manutenções, de 06/2023 a 11/2026.
- **Problemas de qualidade encontrados no original:**
  - datas em 4 formatos;
  - valores com "R$" e vírgula decimal;
  - valores negativos e valores multiplicados por 100;
  - sentinelas (`999999`, `N/D`, `000.000.000-00`);
  - categorias com grafias variadas (`ALUGADA`, ` alugada `);
  - linhas e IDs duplicados;
  - chaves estrangeiras órfãs;
  - clientes cadastrados duas vezes;
  - incoerências lógicas, como devolução antes da retirada.
- **Base tratada:** `database/*.csv` (120 motos, 702 clientes, 4.201 locações e 1.361 manutenções). As regras de limpeza estão em `database/README.md`.

### F2 – M2 Locadora de Motos (dados reais)
- **Origem:** documentos da própria empresa, disponibilizados ao grupo:
  - **CRLV-e** das 10 motos (DETRAN-SP): placa, modelo, cor, ano e data de registro;
  - **Contratos de locação** assinados digitalmente (15 contratos, 05/2025 a 10/2026): cliente, moto, data de retirada, plano e valor semanal;
  - **Extratos bancários** exportados em Excel das contas Asaas e Stone (05/2025 a 04/2026): 251 pagamentos recebidos e despesas com oficinas;
  - **Boletins de ocorrência** da Polícia Civil-SP e declarações à seguradora: datas de 6 furtos/roubos.
- **Método:** extração com Python (leitura de PDF e Excel com `pypdf` e `pandas`) para o mesmo layout da base F1, mais um arquivo de pagamentos.
- **Dados originais:** **não são publicados**, porque contêm dados pessoais (nome, CPF, endereço, CNH, telefone). Em atendimento à LGPD, o repositório traz só a versão anonimizada: nome reduzido a "PrimeiroNome XXX"; CPF, nascimento e telefone substituídos por "XXX"; do e-mail, apenas o domínio.

### F3 a F7 – Dados de mercado
| Fonte | Método de obtenção |
|---|---|
| ABLA Anuário 2026 | Consulta ao anuário online ([virapagina.com.br/abla2026](https://virapagina.com.br/abla2026/)) e ao [comunicado oficial da ABLA](https://www.abla.com.br/noticia/anuario-abla-2026-frota-do-setor-de-locacao-ultrapassa-17-milhao-de-automoveis-) |
| SENATRAN | Download das planilhas oficiais "Frota por UF e Tipo de Veículo" e "Frota por Município e Tipo" de setembro/2026 em [gov.br/transportes](https://www.gov.br/transportes/pt-br/assuntos/transito/conteudo-Senatran/frota-de-veiculos-2026). Foram mantidas as colunas de motocicleta e motoneta, de todas as UFs e dos 868 municípios de SP e PB |
| Fenabrave / Abraciclo | Números publicados na imprensa citando as entidades (links no CSV) |
| SSP-SP | Estudo da FECAP com dados da SSP-SP ([link](https://legismap.com.br/conteudos/artigos-e-noticias/roubos-e-furtos-de-motos-caem-11-6-no-estado-de-sp-revela-pesquisa-da-fecap)) |
| Preços e FIPE | Reportagens de 08/2026 e Tabela FIPE de 10/2026 (links no CSV) |

---

## 3. Perguntas de negócio (revisadas)

As perguntas 1 a 5 foram **mantidas** e agora usam o mercado como régua de comparação. Foi **adicionada a pergunta 6**, que responde ao pedido de relacionar o dashboard com o mercado atual.

1. Quais motos, modelos e filiais geram a maior e a menor rentabilidade (receita líquida e retorno sobre o valor de aquisição)? *Agora também comparado ao valor FIPE de reposição.*
2. Qual é a taxa de ocupação da frota e onde está a ociosidade (por moto, modelo, filial e período)?
3. Quanto a manutenção custa e quanto de receita se perde com motos paradas? Como isso se relaciona com idade, quilometragem e modelo? *A idade da frota é comparada com a do setor (ABLA).*
4. Qual o peso de cancelamentos, atrasos na devolução, **inadimplência e furto/roubo** na receita, e em quais planos, filiais e perfis de cliente se concentram? *O furto/roubo foi incluído porque é a maior perda nos dados reais da M2.*
5. Quais perfis de cliente são mais rentáveis e mais fiéis, e como a demanda varia ao longo do ano?
6. **(nova)** Como a locadora se posiciona frente ao mercado atual em preço, crescimento, mix de modelos, alcance geográfico e risco?

---

## 4. Indicadores (revisados)

Legenda da coluna Base: **S** = base hidratada (F1), **M2** = dados reais (F2), **Mer** = mercado (F3 a F7). Itens marcados com **(novo)** foram adicionados nesta revisão.

### Pergunta 1 – Rentabilidade
| Indicador | Fórmula | Base | Comparação com o mercado |
|---|---|---|---|
| Receita total | Soma de `valor_total` das locações não canceladas | S, M2 | Faturamento do setor: R$ 61,7 bi em 2025, +16,6% (ABLA) |
| Ticket médio por locação | Receita total ÷ locações não canceladas | S, M2 | — |
| Receita líquida por moto | Receita da moto − custo de manutenção da moto | S | — |
| ROI da moto | Receita líquida ÷ valor de aquisição | S | — |
| **(novo)** Payback da moto | Valor de aquisição ÷ receita líquida mensal média da moto | S, M2 | Valor de reposição: FIPE da CG 160 Fan 2025 = R$ 19.354 (10/2026) |

### Pergunta 2 – Ocupação
| Indicador | Fórmula | Base | Comparação com o mercado |
|---|---|---|---|
| Taxa de ocupação | Dias alugados ÷ (dias do período × nº de motos) | S, M2 | O ABLA 2026 não publica taxa de ocupação; a comparação fica interna (filial × filial) |
| Taxa de ociosidade | 1 − taxa de ocupação, por moto, modelo e filial | S, M2 | — |

### Pergunta 3 – Manutenção
| Indicador | Fórmula | Base | Comparação com o mercado |
|---|---|---|---|
| Custo de manutenção por moto | Soma de `custo` por moto | S | — |
| Custo de manutenção por km | Custo total ÷ km rodados | S | — |
| Dias parados | Soma de `dias_parada` por moto e modelo | S | — |
| Receita perdida por parada | Dias parados × diária média do modelo | S | — |
| **(novo)** Idade média da frota | Média de (data de referência − data de aquisição), em meses | S, M2 | Locadoras: 16,4 meses em 2025 (ABLA, automóveis). Base S: 25,7 meses |

### Pergunta 4 – Risco
| Indicador | Fórmula | Base | Comparação com o mercado |
|---|---|---|---|
| Taxa de cancelamento | Locações canceladas ÷ total de locações | S | — |
| Taxa de atraso na devolução | Devoluções após a data prevista ÷ locações finalizadas | S, M2 | — |
| Taxa de inadimplência | Valor não pago ÷ valor faturado | S, M2 | — |
| **(novo)** Taxa de furto/roubo | Motos furtadas ou roubadas ÷ motos-ano expostas × 1.000 | M2 | Estado de SP: 4,4 por 1.000 motos/ano. Capital: 7,8 (SSP-SP 2025 ÷ SENATRAN) |
| **(novo)** Receita perdida por sinistro | Valor semanal × semanas restantes de contrato + valor FIPE não indenizado | M2 | FIPE (reposição) |

### Pergunta 5 – Clientes
| Indicador | Fórmula | Base | Comparação com o mercado |
|---|---|---|---|
| Recorrência | Clientes com 2 ou mais locações ÷ clientes com locação | S | — |
| Receita por perfil | Receita por UF, faixa etária, categoria de CNH e plano | S | Faixa etária só na base S, porque na M2 a data de nascimento está mascarada |
| Sazonalidade | Locações e receita por mês de retirada | S, M2 | Emplacamentos mensais de motos (Fenabrave) |

### Pergunta 6 – Posicionamento no mercado (nova)
| Indicador | Fórmula | Base | Comparação com o mercado |
|---|---|---|---|
| Preço semanal equivalente | Diária média × 7, por plano | S, M2 | BluStar Factor 150: R$ 343–364/semana; Mottu: a partir de ~R$ 18/dia |
| Penetração de mercado | Clientes da locadora ÷ motos registradas no município × 10.000 | S, M2 | SENATRAN 09/2026, por município |
| Crescimento da frota | Variação anual do nº de motos da locadora | S, M2 | Motos em locadoras: +81,6% (ABLA). Emplacamentos: +17,1% (Fenabrave) |
| Mix de modelos | % da frota por modelo | S, M2 | CG 160 é a moto mais vendida do Brasil (478.430 em 2025) |
| Peso das motos na frota local | Motos ÷ frota total do município/UF | Mer | PB: 45,7%; SP: 21,2%; João Pessoa: 32,6%; São Paulo: 16,9% |

**Filtros do painel:** período, filial, marca e modelo, plano de locação, forma de pagamento, status da moto e **(novo)** base de dados (hidratada × M2 real).

---

## 5. Como o dashboard se relaciona com o mercado

| Página do dashboard | O que mostra | Dados de mercado usados |
|---|---|---|
| Visão geral (P1) | Receita, ticket, ROI e payback por moto/modelo/filial | Cartão com o faturamento e o crescimento do setor (ABLA); valor FIPE como linha de referência |
| Ocupação e manutenção (P2, P3) | Ocupação, ociosidade, custo/km, dias parados, idade da frota | Idade média da frota das locadoras (ABLA) |
| Risco (P4) | Cancelamento, atraso, inadimplência, furto/roubo | Taxa de roubo/furto do estado e da capital de SP (SSP-SP + SENATRAN) |
| Clientes (P5) | Perfis, recorrência, sazonalidade | — |
| **Mercado (P6)** | Mapa de motos por UF, ranking de municípios, preço × concorrentes, crescimento | SENATRAN (mapa e municípios), ABLA, Fenabrave, preços BluStar/Mottu |

**Ligação entre as tabelas no modelo de dados:**
- `clientes.cidade` ↔ `senatran_frota_motos_municipios_sp_pb.municipio` (em maiúsculas e sem acento);
- `motos.filial` ↔ UF (João Pessoa → PB, São Paulo → SP) ↔ `senatran_frota_motos_uf.regiao_uf`.

---

## 6. Primeiras comparações (base × mercado)

| Tema | Base | Mercado | Leitura |
|---|---|---|---|
| **Preço** | S: plano semanal R$ 74,98/dia (≈ R$ 525/semana); mensal ≈ R$ 434/semana. M2: R$ 449/semana (plano de 28 meses que termina com a transferência da moto ao cliente) | BluStar R$ 343–364/semana; Mottu a partir de ~R$ 126/semana | Os preços da base hidratada estão **acima do mercado**. O preço da M2 se justifica por ser um plano de compra parcelada |
| **Furto/roubo** | M2: 6 das 10 motos furtadas/roubadas, ≈ 610 por 1.000 motos/ano; 4 delas nos primeiros 4 meses de uso | Capital SP: 7,8 por 1.000 motos/ano | Risco **dezenas de vezes acima da média**. É a principal perda da operação real, com impacto maior que manutenção ou inadimplência |
| **Demanda potencial** | S: 303 clientes em João Pessoa e 183 em São Paulo | João Pessoa: 169.690 motos (32,6% da frota); São Paulo: 1.715.050 (16,9%) | A penetração em João Pessoa (~18 clientes por 10 mil motos) é muito maior que em São Paulo (~1 por 10 mil): há espaço para crescer em SP |
| **Crescimento** | Frota S: 120 motos; M2: 10 | Motos em locadoras: +81,6% em 2025 (263.330); Mottu: +26,6% em jan–jul/2026 | O setor de locação de motos cresce muito acima do mercado de motos (+17,1%) |
| **Mix de modelos** | S: 38% CG 160 (Titan + Fan); M2: 100% CG 160 | CG 160 lidera as vendas nacionais | O mix está alinhado ao mercado e ao modelo mais visado em furtos |
| **Idade da frota** | S: 25,7 meses | Locadoras (automóveis): 16,4 meses | A frota da base é mais velha que a média do setor: verificar o impacto no custo de manutenção (P3) |

---

## 7. Limitações
- O ABLA 2026 não publica **taxa de ocupação** nem **diária média**, e a idade média da frota refere-se a automóveis. Por isso a ocupação é comparada só internamente.
- As taxas de furto/roubo dividem ocorrências de 2025 (SSP-SP) pela frota de 09/2026 (SENATRAN). São uma aproximação.
- Os preços de concorrentes vêm de reportagens de 08/2026 e podem variar por cidade e plano.
- A base M2 é pequena (10 motos, 15 contratos), e os extratos cobrem só 05/2025 a 04/2026. Ela serve como validação real, não como base estatística principal.
- Faltam na M2 o valor de aquisição, a quilometragem e a moto atendida em cada manutenção, porque essas informações não constam nos documentos.
