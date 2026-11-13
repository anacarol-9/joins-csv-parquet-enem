# Reduce-side vs Map-side Join: CSV e Parquet com os microdados do ENEM

Comparação de duas estratégias de join usadas em processamento distribuído e do impacto do formato de arquivo (CSV e Parquet), usando os microdados do ENEM 2023, com cerca de 3,9 milhões de linhas.

Projeto do MBA em Engenharia de Dados da UFRJ (disciplina de Modelagem MapReduce).

## O que foi feito

Os conceitos de cluster foram **simulados em Python com pandas**, numa única máquina:

- **Reduce-side Join (shuffle):** simulado com `pd.merge`, que precisa alinhar as chaves das duas tabelas, como acontece quando os dados são redistribuídos entre os nós.
- **Map-side Join (broadcast):** simulado carregando a tabela pequena num dicionário em memória e fazendo a busca direta com `.map`, como no Distributed Cache, em que a tabela pequena vai para a memória de cada mapper.

Os dois joins foram medidos com os arquivos em CSV e, depois, com os mesmos dados convertidos para Parquet.

## Dados

| Papel | Base | Tamanho | Fonte |
| --- | --- | --- | --- |
| Tabela fato | Microdados do ENEM 2023 | 3.933.955 linhas | INEP |
| Tabela dimensão | Municípios do Brasil | 5.571 linhas | base pública de municípios |

**Chave do join:** `CO_MUNICIPIO_PROVA` (código do município onde a prova foi aplicada), trazendo da dimensão a informação de capital.

Os dados não estão no repositório por causa do tamanho. Os microdados podem ser baixados no portal do INEP.

## Resultados

| Formato | Reduce-side Join | Map-side Join | Map-side mais rápido |
| --- | --- | --- | --- |
| CSV | 1,31 s | 0,058 s | 22,6 vezes |
| Parquet | 1,28 s | 0,060 s | 21,2 vezes |

Leitura completa do Parquet (todas as colunas): 12,4 s.

## Conclusões

1. **O Map-side Join foi cerca de 20 vezes mais rápido nos dois formatos.** Quando uma das tabelas é pequena, levá-la para a memória evita o custo de alinhar as chaves da tabela grande.
2. **O formato quase não mudou o tempo dos joins.** Isso é esperado: depois que os dados estão em memória, o join não depende mais de como estavam gravados. O ganho do Parquet aparece na leitura, que é a etapa de I/O.
3. **O ganho do armazenamento colunar depende de ler só as colunas necessárias.** Neste experimento o Parquet foi lido com todas as colunas. A próxima melhoria é medir também a leitura do CSV e a leitura do Parquet com apenas as colunas usadas, que é onde o formato colunar se destaca.

## Limitações

- Simulação em pandas numa única máquina, sem cluster. Os tempos mostram a lógica de cada estratégia, não o desempenho real de um ambiente distribuído.
- O `merge` traz todas as colunas e descarta as linhas sem correspondência, enquanto o `.map` traz uma única coluna. Por isso a comparação mostra a ordem de grandeza da diferença, não um valor exato.

## Tecnologias

Python, pandas, Parquet (pyarrow).
