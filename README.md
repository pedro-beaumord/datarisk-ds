# Case técnico de risco de crédito

O projeto estima o risco de atraso inicial no momento da solicitação de crédito e avalia se o score pode ajudar a priorizar propostas para revisão. O [notebook](notebook/case_risco_credito.ipynb) reúne a análise, a validação temporal, a simulação da política e a geração do arquivo de submissão.

## Como reproduzir

Use **Python 3.14.6**, versão do ambiente de referência. Na raiz do repositório, crie um ambiente virtual e instale as dependências:

```bash
python3.14 -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements.txt
```

Coloque os arquivos originais fornecidos no case dentro de `data/`. Mantenha os nomes abaixo.

| Local | Conteúdo |
| --- | --- |
| `notebook/case_risco_credito.ipynb` | Análise, treinamento, validação e exportação |
| `data/` | `base_cadastral.parquet`, `base_submissao.parquet`, `historico_emprestimos.parquet`, `historico_parcelas.parquet` e `dicionario_dados.csv` |
| `requirements.txt` | Dependências com versões fixadas |

Abra o notebook no VS Code, selecione o kernel da `.venv` e execute todas as células em ordem. Para usar o Jupyter no navegador, execute `python -m notebook` na raiz do repositório e abra o mesmo arquivo.

A última célula gera `submissao_case.csv` no diretório de execução do kernel. O arquivo tem **40.000 linhas**, uma por solicitação, na ordem da base original, com as colunas `id_cliente` e `probabilidade_inadimplencia`.

## Abordagem e resultados

O alvo marca contratos aprovados quando há atraso superior a sete dias em pelo menos uma das três primeiras parcelas. A análise exige vencimentos válidos e janela de observação completa para as três parcelas. Em 2024, a definição de sete dias produziu **238 eventos**, frente a **13** no corte de 30 dias. O score estima o risco desse atraso inicial.

Na validação temporal em 2024, a regressão logística com as informações da proposta obteve **ROC-AUC de 0,609** e **Average Precision de 0,0206**, frente a **0,0132** do baseline constante. A versão com histórico teve Average Precision de **0,0189** e Brier ligeiramente menor. A versão com dados da proposta foi escolhida pelo melhor ranking na validação e pela menor dependência de dados na concessão.

## Política de crédito

Na simulação, uma fila com os **5% de maior score** concentrou **11,3%** dos eventos de atraso. O uso inicial proposto é registrar o score e avaliar a priorização **em modo sombra**, enquanto as decisões seguem a política vigente.

O score médio de **2,86%** superestimou a taxa observada de **1,32%** na validação. Como o resultado histórico cobre apenas contratos aprovados com desfecho observável, a calibração e o desempenho da fila em novas safras devem ser avaliados antes de definir cortes de crédito.
