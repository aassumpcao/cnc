# cnc

Scraper do Cadastro Nacional de Condenações Cíveis por Ato de Improbidade Administrativa e Inelegibilidade - CNJ.

## Instalação

```
if (!require(devtools)) install.packages(devtools)
devtools::install_github('abjur/cnc')
```

Também é necessário instalar o PhantomJS para o funcionamento de algumas funções. Ver ?GET_pjs para detalhes.

## Utilização

### Baixar páginas

- `cnc_pags` para baixar as páginas de resultados do CNC.
- `cnc_pessoas` para baixar as páginas de condenados do CNC.
- `cnc_processos` para baixar as páginas de processos do CNC.
- `cnc_infos_pessoas` para baixar as páginas de informações de pessoas do CNC.

### Parsers

- `parse_pags` transformas os arquivos baixados por `cnc_pags` em `data_frame`.
- `parse_pessoas` transformas os arquivos baixados por `cnc_pessoas` em `data_frame`.
- `parse_processos` transformas os arquivos baixados por `cnc_processos` em `data_frame`.
- `parse_infos_pessoas` transformas os arquivos baixados por `cnc_infos_pessoas` em `data_frame`.

### Workflow

Para obter o conjunto de dados completo, rodar nessa ordem:

```
library(magrittr)
library(cnc)

cnc_pags(path = 'data-raw/pags', pags = 1:2399)
d_pags <- 'data-raw/pags' %>% 
  dir(full.names = TRUE) %>% 
  parse_pags(arqs)

# baixa pessoas e processos
cnc_pessoas(d_pags, path = "data-raw/pessoas")
cnc_processos(d_pags, path = "data-raw/processos")

# parse pessoas e processos
d_pessoas <- "data-raw/pessoas" %>% 
  dir(full.names = TRUE) %>% 
  parse_pessoas(arqs)
d_processos <- "data-raw/processos" %>% 
  dir(full.names = TRUE) %>% 
  parse_pessoas(arqs)

# baixa infos das pessoas
cnc_pessoas_infos(d_pessoas, path = 'data-raw/pessoas_infos')
# parse infos das pessoas
d_pessoas_infos <- "data-raw/pessoas_infos" %>% 
  dir(full.names = TRUE) %>% 
  parse_pessoas_infos(arqs)
```

## Atualização 14/10/2016

Após um estudo mais detalhado da base, conseguimos temos também os códigos para deixar a base tidy! Com a nova função `tidy_cnc` obtemos um bd com 57 colunas já trabalhadas. A base resultante está praticamente em formato analítico, carecendo somente de análises de inconsistências mais detalhadas.

```r 
cnc_tidy <- tidy_cnc(cnc_condenacoes, cnc_pags, cnc_processos, cnc_pessoa_infos)
```

A base de dados possui essas colunas:

1. Condenações

- Metadados e identificadores:
    - `arq_pag`: arquivo que contém a página de pesquisa de onde foi obtido o link da condenação.
    - `id_pag`: id da condenação (1 a 15 condenações), que repete por página.
    - `arq`: arquivo que contém a página HTML com os dados da condenação.
    - `id_condenacao`: id único da condenação.
    - `id_processo`: id único do processo.
    - `id_pessoa`: id único da pessoa.
- Informações básicas:
    - `tipo_pena`: Trânsito em Julgado ou Órgão colegiado.
    - `dt_pena`: Data da pena.
    - `cod_assunto_[1:5]`: códigos dos assuntos (entre 1 e 5 assuntos) da condenação.
    - `nm_assunto_[1:5]`: nomes dos assuntos (entre 1 e 5 assuntos) da condenação.
- Inelegibilidade:
    - `teve_inelegivel`: sim ou vazio.
- Perda de Emprego/Cargo/Função Pública:
    - `teve_perda_cargo`: sim ou vazio.
- Pagamento de multa:
    - `teve_multa`: sim ou vazio.
    - `vl_multa`: valor da multa em reais.
- Ressarcimento integral do dano:
    - `teve_ressarcimento`: sim ou vazio.
    - `vl_ressarcimento`: valor da multa em reais.
- Perda de bens ou valores acrescidos ilicitamente ao patrimônio:
    - `teve_perda_bens`: sim ou vazio.
    - `vl_perda_bens`: valor da multa em reais.
- Pena privativa de liberdade:
    - `teve_pena`: sim ou vazio.
    - `duracao_pena`: duração em dias.
    - `de_pena`: data de início.
    - `ate_pena`: data do fim (pode ser no futuro).
- Suspensão dos Direitos Políticos:
    - `teve_suspensao`: sim ou vazio.
    - `duracao_suspensao`: duração em dias.
    - `de_suspensao`: data de início.
    - `ate_suspensao`: data do fim (pode ser no futuro).
- Proibição de Contratar com o Poder Público ou receber incentivos fiscais ou creditícios, direta ou indiretamente, ainda que por intermédio de pessoa jurídica da qual seja sócio majoritário:
    - `teve_proibicao`: sim ou vazio.
    - `duracao_proibicao`: duração em dias.
    - `de_proibicao`: data de início.
    - `ate_proibicao`: data do fim (pode ser no futuro).

2. Processos

- `arq_processo`: nome do arquivo (contém o id que aparece no link da base `cnc_pags`).
- `id_processo`: código identificador do processo.
- `dt_cadastro`: data de cadastro do processo no sistema.
- `n_processo`: número identificador do processo.
- `esfera_processo`: estadual, federal, militar ou superior.
- `tribunal`: nome do tribunal.
- `instancia`: primeiro grau, segundo grau, militar ou superior.
- `comarca_secao`: nome da comarca ou seção (aplicável somente ao primeiro grau).
- `vara_camara`: nome da vara (primeiro grau) ou câmara/seção de julgamento (segundo grau ou militar).
- `dt_propositura`: data de propositura da ação.

3. Pessoas

- Identificadores:
    - `arq_pessoa`: nome do arquivo que contém as informações.
    - `id_pessoa`: id da pessoa (para juntar com a base de condenações).
- Informações básicas:
    - `tipo_pessoa`: F = física e J = jurídica.
    - `nm_pessoa`: Nome da pessoa.
    - `sexo`: F = feminino e M = masculino.
    - `publico`: S = funcionário público; N = não é funcionário público.
- Informações de funcionários públicos:
    - `esfera`: F = Federal, D = Distrital, E = Estadual, M = Municipal.
    - `orgao`: órgão que a pessoa trabalha (prefeitura, tribunal etc).
    - `cargo`: cargo que a pessoa exerce (prefeito, servidor etc).
    - `uf`:.
    - `cod`: código interno da pessoa (provavelmente não será utilizado).

Para facilitar as análises, construímos uma base unificada, contendo todas as informações de condenações, pessoas e processos. Nessa base, informações sobre pessoas e processos aparecem duplicadas quando fazem parte de mais de uma condenação. A base possui 35.977 linhas (a mesma quantidade da base de condenações) e 57 colunas.

## TODO

- Análises de inconsistências detalhada.
- Aprimorar estudos sobre valores e perfil de condenados.
- Atualizar dados em jan/2016.

