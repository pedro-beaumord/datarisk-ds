# Case técnico de risco de crédito

O projeto estima a probabilidade de atraso inicial a partir dos dados disponíveis na solicitação e avalia o uso do score para priorizar propostas de crédito. O [notebook](notebook/case-risco-credito.ipynb) contém a construção do alvo, a validação temporal, a calibração, a simulação da política de crédito e a geração da submissão.

## Como reproduzir

O ambiente de referência usa **Python 3.14.6**. Na raiz do repositório, crie um ambiente virtual e instale as dependências:

```bash
python3.14 -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements.txt
```

Coloque os cinco arquivos originais do case em `data/`, na raiz do repositório, mantendo estes nomes:

- `base_cadastral.parquet`
- `base_submissao.parquet`
- `historico_emprestimos.parquet`
- `historico_parcelas.parquet`
- `dicionario_dados.csv`

No VS Code, abra a pasta do repositório, selecione o Python da `.venv` como kernel e execute todas as células em ordem. Para rodar no navegador, execute `python -m notebook` na raiz e abra `notebook/case-risco-credito.ipynb`. O notebook localiza `data/` quando iniciado da raiz ou de `notebook/`. Para outro local, defina a variável de ambiente `CASE_DATA_DIR`.

A última célula grava `submissao_case.csv` no diretório de execução do kernel, com **40.000 linhas** na ordem da base de submissão e as colunas `id_cliente` e `probabilidade_inadimplencia`. A probabilidade corresponde ao alvo de atraso inicial definido abaixo.

## Alvo, modelo e validação

O alvo marca contratos aprovados com atraso superior a sete dias em alguma das três primeiras parcelas, incluindo parcelas sem pagamento registrado após essa janela. A rotulagem exige três vencimentos válidos e uma janela de observação completa.

A regressão logística combina os termos da proposta com três sinais de pagamento: razão entre valores pagos e previstos, atraso recente e ausência de histórico. O cálculo usa uma defasagem de **30 dias** como margem operacional para o registro dos pagamentos. O modelo foi treinado em contratos de 2022 e 2023 com rótulos disponíveis antes de 2024.

| Validação de 2024 | Só proposta | Proposta + pagamentos |
| --- | ---: | ---: |
| ROC-AUC | 0,608 | 0,655 |
| Average Precision | 0,0206 | 0,0312 |
| Eventos capturados na fila de 5% | 11,3% | 15,5% |

A calibração do intercepto foi ajustada com rótulos maduros disponíveis antes de julho de 2024 e avaliada no segundo semestre. O Brier caiu de **0,009860** para **0,009551**. O score médio calibrado ainda ficou acima da taxa observada (**1,38%** frente a **0,96%**). Para a submissão, o intercepto foi atualizado com os rótulos de 2024 maduros antes da primeira solicitação a pontuar.

## Política de crédito

A proposta começa em **modo sombra**, com acompanhamento do ranking e da calibração por safra e produto. Após validação prospectiva, o piloto usaria as faixas de risco para orientar aprovação, ajuste de condições ou revisão manual. Uma regra de recusa exige estimativas próprias de probabilidade de default, margem e perda. O desempenho histórico foi medido em contratos aprovados com desfecho observável.

A entrega reúne notebook, README, dependências e CSV de submissão. As bases originais são necessárias apenas para reproduzir a análise.
