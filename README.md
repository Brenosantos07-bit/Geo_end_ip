# Geolocalização de Endereços IPv4 — Comparação de Algoritmos de Busca

Este repositório reúne implementações de busca **linear**, **binária em vetor** e **em árvore binária balanceada**, além de um experimento específico para a busca binária. O objetivo é estudar como encontrar a faixa à qual um endereço IPv4 pertence e retornar o país associado, quando disponível.

> **Atenção à metodologia:** os programas não executam exatamente o mesmo teste. A busca linear usa faixas **sintéticas**, enquanto a busca binária e a árvore usam uma amostra da **IP2Location LITE DB1**. Portanto, os tempos apresentados por esses programas, sem padronizar dados e consultas, **não constituem uma comparação justa dos três métodos**.

## Linguagem, ambiente e requisitos

- **Linguagem:** Python 3 (no computador usado para o experimento relatado da busca binária: **Python 3.12.8**).
- **Sistema operacional usado naquele experimento:** **Windows 11**.
- **Editor recomendado:** Visual Studio Code, mas não é obrigatório.
- **Dependências:** somente a biblioteca padrão do Python (`csv`, `ipaddress`, `time`, `random`, `statistics`, `platform`, `json`, `bisect`, `tracemalloc`, entre outras). **Não é necessário instalar pacotes com `pip`.**
- Os scripts podem funcionar em outros sistemas operacionais com Python 3; os tempos variam conforme o computador e a execução.

## Arquivos do repositório

| Arquivo | Para que serve |
| --- | --- |
| `base_ip2location_10000_faixas_ipv4.csv` | Amostra de **10.000 intervalos IPv4** da IP2Location LITE DB1, com IP inicial/final, valores inteiros, código e nome do país. |
| `busca_binaria.py` | Carrega e ordena o CSV; verifica sobreposição; procura **dois IPs de exemplo** por busca binária e exibe o país ou `Não encontrado`. **Não mede tempo**. |
| `busca_linear.py` | Gera internamente **10.000 faixas fictícias** de 256 IPs cada, começando em `10.0.0.0`; percorre sequencialmente as faixas em quatro testes e exibe faixa, comparações e tempo em **milissegundos**. **Não lê o CSV**. |
| `arvore_binaria.py` | Carrega o CSV, constrói uma **árvore binária balanceada** e consulta dois IPs; imprime informações do ambiente, altura da árvore, memória medida para a construção dos nós, quantidade de nós visitados e três rodadas de 1.000.000 de consultas. |
| `experimento_topico5_ipv4.py` | Faz benchmark da **busca binária em vetor** usando o CSV; gera 1.000.000 de consultas (500.000 dentro e 500.000 fora dos intervalos selecionados), valida respostas, realiza três rodadas e salva `resultado_topico5_ipv4_local.json`. |
| `resultado_topico5_ipv4_local.json` | **Gerado automaticamente** após executar o experimento; registra sistema operacional, versão do Python, tempos e demais dados da execução. Não é um arquivo de entrada. |
| `README.md` | Este documento com instruções de instalação e execução. |

**Nomes dos arquivos:** se os uploads mantiverem nomes como `busca_binaria(2).py`, `experimento_topico5_ipv4(1).py`, `Árvore binária.py` e `base_ip2location_10000_faixas_ipv4(1).csv`, **renomeie-os** para os nomes da tabela antes de executar. O nome do CSV deve ser **exatamente** `base_ip2location_10000_faixas_ipv4.csv` porque os scripts o procuram com esse nome.

## Como executar no Windows

1. Instale o **Python 3** e, opcionalmente, o **VS Code**.
2. Baixe ou clone o repositório e abra sua pasta no VS Code (**Arquivo → Abrir Pasta**).
3. Verifique se os arquivos `.py` e o `.csv` estão **na mesma pasta**.
4. Abra **Terminal → Novo Terminal**.
5. Confirme a instalação:

   ```powershell
   py --version
   ```

6. Execute os programas, **um de cada vez**:

   ```powershell
   py busca_binaria.py
   py busca_linear.py
   py arvore_binaria.py
   py experimento_topico5_ipv4.py
   ```

Em sistemas em que `py` não está disponível, use `python` ou `python3` no lugar de `py`.

**Importante:** abra o terminal **dentro da pasta do projeto** (aquela que contém o CSV). `busca_binaria.py` e `arvore_binaria.py` usam um caminho relativo para localizar a base.

## Como rodar cada arquivo e o que esperar

### 1. `busca_binaria.py`

```powershell
py busca_binaria.py
```

Saída de exemplo, confirmada com o CSV:

```text
Faixas carregadas: 10000
1.0.16.3 -> Japan
1.0.0.1 -> Não encontrado
```

`Não encontrado` significa que o IP **não pertence às 10.000 faixas selecionadas**, e não que ele não tenha país na base completa da IP2Location.

### 2. `busca_linear.py`

```powershell
py busca_linear.py
```

O programa testa quatro situações: primeira faixa, uma faixa no meio, o IP rotulado como “Última faixa” e um IP fora da tabela. Mostra a quantidade de comparações e o tempo **individual** de cada consulta.

Valores de referência para o resultado lógico, conforme o código:

| Teste | Resultado | Comparações |
| --- | --- | ---: |
| Primeira faixa (`10.0.0.10`) | `faixa_1` | 1 |
| Meio da tabela (`10.19.136.10`) | `faixa_5001` | 5.001 |
| “Última faixa” (`10.39.255.10`) | `None` | 10.000 |
| Fora da tabela (`172.16.0.1`) | `None` | 10.000 |

**Observação:** o endereço usado no teste chamado “Última faixa” está, na verdade, **fora** das 10 mil faixas criadas pelo programa. O nome do teste não representa uma correspondência encontrada. Em uma execução de verificação com Python 3.13.5 em Linux, os tempos dos quatro testes foram, respectivamente, **0,0101 ms**, **0,2282 ms**, **0,4502 ms** e **0,5186 ms**. São apenas exemplos; os tempos em milissegundos variam a cada execução.

### 3. `arvore_binaria.py`

```powershell
py arvore_binaria.py
```

A implementação constrói a árvore balanceada a partir da amostra IP2Location. Em uma execução de verificação, foram observados:

```text
Faixas carregadas na árvore: 10000
Altura da árvore: 14 níveis
1.0.16.3 -> Japan (nós visitados: 14)
1.0.0.1 -> Não encontrado (nós visitados: 14)
Máximo de nós visitados no benchmark: 14
Total de erros em 3.000.000 de buscas: 0
```

O programa também imprime a versão do Python, o sistema operacional, CPU, RAM, memória da árvore, tempo de cada uma das três rodadas, mediana e média. **Exemplo de medição realizada em Python 3.13.5/Linux (não no notebook Windows do grupo):** 1,151036 s, 1,202514 s e 1,108892 s; mediana de 1,151036 s e média de 1,154147 s. A memória dos nós da árvore, nessa execução, foi de 1,07 MB. Esses valores devem ser medidos novamente **no terminal de quem executar**. Cada rodada processa **1.000.000 de consultas**, totalizando **3.000.000 de buscas nas três rodadas**; o arquivo usa a semente aleatória `42`.

### 4. `experimento_topico5_ipv4.py`

```powershell
py experimento_topico5_ipv4.py
```

O script gera 1.000.000 de consultas com semente aleatória `2026`, executa uma validação antes das medições e mede **somente o laço das buscas**, não a leitura do CSV nem a preparação/validação dos dados. São três repetições, com 500.000 consultas a faixas presentes e 500.000 a endereços ausentes da amostra.

Resultados **registrados no notebook Windows 11 com Python 3.12.8** durante o desenvolvimento:

| Execução | Tempo |
| --- | ---: |
| 1 | 14,895891 s |
| 2 | 13,488810 s |
| 3 | 14,193141 s |
| **Mediana** | **14,193141 s** |
| **Média** | **14,192614 s** |

- Consultas em faixas selecionadas: **500.000**.
- Consultas fora das faixas selecionadas: **500.000**.
- Divergências encontradas pelo validador: **0**.

Esses são **resultados de uma execução específica**, não valores fixos do algoritmo. Ao rodar no seu computador, registre os novos números. O arquivo `resultado_topico5_ipv4_local.json` será **criado ou substituído** com as medições atuais.

## Observações sobre a comparação experimental

- A busca binária em vetor e a árvore balanceada usam o CSV da IP2Location, mas **geram consultas com critérios/sementes diferentes** (`2026` e `42`, respectivamente).
- A busca linear usa outra base, **sintética**, e apenas quatro consultas de demonstração.
- Os programas medem escopos diferentes: por exemplo, a busca linear cronometra conversão + busca individual, enquanto o benchmark binário cronometra um lote de buscas sobre IPs já representados como inteiros.
- Para uma **comparação controlada**, será necessário aplicar os três algoritmos à **mesma base e exatamente ao mesmo conjunto de consultas**, usando um protocolo consistente de medição.
- A ausência de divergências na verificação significa **coerência com os intervalos do arquivo**, não precisão geográfica absoluta da base IP2Location.

## Fonte dos dados e transparência

A amostra CSV foi obtida a partir da **IP2Location LITE DB1 (IP-Country)**. Consulte os termos de uso e a atribuição/licença da fonte antes de republicar o CSV:

https://lite.ip2location.com/pt/database/db1-ip-country

**Declaração de uso de IA:** o ChatGPT (OpenAI) foi utilizado como apoio na organização da documentação, explicação do código e preparação de partes do experimento. Os resultados devem ser atribuídos ao ambiente em que forem efetivamente executados. A documentação e os programas devem ser apresentados ao professor de forma compatível com as regras de autoria e uso de ferramentas estabelecidas na disciplina.
