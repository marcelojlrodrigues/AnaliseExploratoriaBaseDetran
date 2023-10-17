
#  Situação problema
A empresa Inteligência DevData Tecnologia trabalha com prestação de serviços para o Detran a nível nacional, focando em análises de mercado e estratégias relacionadas ao volume de carros registrados. 

Uma das principais demandas é avaliar a base de veículos registrados para gerar insights valiosos e disponibilizar esses dados para as áreas de marketing e logística da empresa. Pois o Detran deseja criar novas estratégias para os próximos 2 anos, entendendo assim sua base de cadastro de veículos nos últimos anos – de 2003 até 2021.

As tabelas disponibilizadas para análise foram:
denatran_frota_municipio_tipo
detran_frota_br_uf

São tabelas com a frota de veículos por tipo, estado e município, organizada pelo Ministério da Infraestrutura (MI) contendo informações de 2003 até 2021.
Em pesquisa verifiquei que o Detran (Departamento de Trânsito) classifica os veículos em várias categorias de acordo com o tipo e uso. 
Para iniciar a análise fui pesquisar o que significa cada transporte para facilitar no início das análises, abaixo um descritivo de cada tipo de transporte:

 - Automóvel: Veículos de passeio destinados ao transporte de passageiros.
 - Caminhão: Veículos de carga destinados ao transporte de mercadorias.
 - Caminhão-trator: Veículos usados para rebocar semirreboques.
 - Caminhonete: Veículos menores utilizados para transporte de carga ou passageiros.
 - Camioneta: Semelhante a caminhonete, usada para carga ou passageiros.
 - Chassi-plataforma: Um tipo de veículo usado como base para a montagem de outros tipos de carrocerias.
 - Ciclomotor: Veículos de duas ou três rodas com baixa cilindrada, como motonetas.
 - Micro-ônibus: Veículos maiores usados para transporte de passageiros em menor escala.
 - Motocicleta: Veículos de duas rodas motorizados.
 - Motoneta: Veículos de duas rodas com menor cilindrada, semelhantes a scooters.
 - Ônibus: Veículos maiores usados para transporte coletivo de passageiros.
 - Quadriciclo: Veículos de quatro rodas, frequentemente usados para recreação.
 - Reboque: Veículos usados para transporte de carga rebocados por outro veículo.
 - Semirreboque: Reboques sem rodas dianteiras, que são normalmente acoplados a caminhões.
 - Sidecar: Veículo de três rodas acoplado a uma motocicleta.
 - Triciclo: Veículo de três rodas motorizado.
Utilitário: Veículos usados para fins de trabalho ou utilitários, como picapes.


## Problemas

1) Qual é o entendimento do volume de veículos registrados e como foi realizada a análise exploratória?

A query abaixo retorna uma visão geral do volume de veículos registrados ao longo dos anos, informando o total de veículos por ano.

```
SELECT ano, SUM(total) as total_veiculos_registrados
FROM denatran_frota_municipio_tipo
GROUP BY ano
ORDER BY ano;

Abaixo tentei criar uma query que informe o volume total de veículos registrados e a variação percentual em relação ao ano anterior (porém o cálculo percentual está dando erro verificar com o professor).

SELECT
    ano,
    SUM(total) as total_veiculos_registrados,
    (SUM(total) - LAG(SUM(total)) OVER (ORDER BY ano)) / LAG(SUM(total)) OVER (ORDER BY ano) * 100 as variacao_percentual
FROM denatran_frota_municipio_tipo
GROUP BY ano
ORDER BY ano;


2) Como é a distribuição de veículos por município e estado, e quais são suas estatísticas relevantes?

A query abaixo informa a soma total de veículos para cada combinação de estado (sigla_uf) e município (id_municipio):

```sql
    SELECT sigla_uf, id_municipio, SUM(total) as total_veiculos
    FROM denatran_frota_municipio_tipo
    GROUP BY sigla_uf, id_municipio;
```

3) Cálculos estatísticos da distribuição de veículos por estado e município, sendo eles: Total de Veículos, Média de Veículos, Mediana de Veículos, Min Veículos e Máximo de Veículos.

```sql
    SELECT
        sigla_uf,
        id_municipio,
        SUM(total) as total_veiculos,
        AVG(total) as media_veiculos,
        MEDIAN(total) as mediana_veiculos,
        MIN(total) as min_veiculos,
        MAX(total) as max_veiculos
    FROM denatran_frota_municipio_tipo
    GROUP BY sigla_uf, id_municipio
    ORDER BY total_veiculos DESC;
```
3) Como é feita a análise por diferentes tipos de veículos e como são agrupados em classificações distintas, como por exemplo, veículos de carga, veículos de passeio, motocicletas, etc.?

    ```sql
    SELECT
        SUM(automovel) as veiculos_passeio,
        SUM(caminhao + caminhaotrator + caminhonete + camioneta + utilitario) as veiculos_carga,
        SUM(motocicleta + motoneta) as motocicletas,
        SUM(quadriciclo + triciclo) as veiculos_especiais
    FROM denatran_frota_municipio_tipo;
```

4) Agrupa por categoria e informa: média, mediana e total por categoria em ordem crescente.

```sql
    SELECT
        'Veículos de Passeio' as categoria,
        AVG(automovel) as media,
        MEDIAN(automovel) as mediana,
        SUM(automovel) as total
    FROM denatran_frota_municipio_tipo

    UNION

    SELECT
        'Veículos de Carga' as categoria,
        AVG(caminhao + caminhaotrator + caminhonete + camioneta + utilitario) as media,
        MEDIAN(caminhao + caminhaotrator + caminhonete + camioneta + utilitario) as mediana,
        SUM(caminhao + caminhaotrator + caminhonete + camioneta + utilitario) as total
    FROM denatran_frota_municipio_tipo

    UNION

    SELECT
        'Motocicletas' as categoria,
        AVG(motocicleta + motoneta) as media,
        MEDIAN(motocicleta + motoneta) as mediana,
        SUM(motocicleta + motoneta) as total
    FROM denatran_frota_municipio_tipo

    UNION

    SELECT
        'Veículos Especiais' as categoria,
        AVG(quadriciclo + triciclo) as media,
        MEDIAN(quadriciclo + triciclo) as mediana,
        SUM(quadriciclo + triciclo) as total
    FROM denatran_frota_municipio_tipo
    ORDER BY categoria;
```

5) É possível verificar com essa query qual categoria tem a maior média de veículos por município.

Há variações significativas nas categorias ao longo dos anos?

Quais grupos de veículos têm maior representatividade por regiões geográficas e estados? E como é feita a criação de categorias com agrupamento dos diferentes veículos?

Quais categorias de veículos têm maior representatividade em determinadas regiões ou estados?

Agrupamento dos veículos por categorias ( Veículos de Passeio, Veículos de Carga, Motocicletas e Veículos especiais) para cada Estado:

```sql
SELECT
    d.sigla_uf,
    SUM(dt.automovel) as veiculos_passeio,
    SUM(dt.caminhao + dt.caminhaotrator + dt.caminhonete + dt.camioneta + dt.utilitario) as veiculos_carga,
    SUM(dt.motocicleta + dt.motoneta) as motocicletas,
    SUM(dt.quadriciclo + dt.triciclo) as veiculos_especiais
FROM detran_frota_br_uf d
JOIN denatran_frota_municipio_tipo dt ON d.sigla_uf = dt.sigla_uf
GROUP BY d.sigla_uf;


Levantamento estatístico para cada categoria de veículos em cada estado.

SELECT
    d.sigla_uf,
    'Veículos de Passeio' as categoria,
    AVG(dt.automovel) as media,
    MEDIAN(dt.automovel) as mediana,
    SUM(dt.automovel) as total
FROM detran_frota_br_uf d
JOIN denatran_frota_municipio_tipo dt ON d.sigla_uf = dt.sigla_uf
GROUP BY d.sigla_uf

UNION

SELECT
    d.sigla_uf,
    'Veículos de Carga' as categoria,
    AVG(dt.caminhao + dt.caminhaotrator + dt.caminhonete + dt.camioneta + dt.utilitario) as media,
    MEDIAN(dt.caminhao + dt.caminhaotrator + dt.caminhonete + dt.camioneta + dt.utilitario) as mediana,
    SUM(dt.caminhao + dt.caminhaotrator + dt.caminhonete + dt.camioneta + dt.utilitario) as total
FROM detran_frota_br_uf d
JOIN denatran_frota_municipio_tipo dt ON d.sigla_uf = dt.sigla_uf
GROUP BY d.sigla_uf

UNION

SELECT
    d.sigla_uf,
    'Motociclestas' as categoria,
    AVG(dt.motocicleta + dt.motoneta) as media,
    MEDIAN(dt.motocicleta + dt.motoneta) as mediana,
    SUM(dt.motocicleta + dt.motoneta) as total
FROM detran_frota_br_uf d
JOIN denatran_frota_municipio_tipo dt ON d.sigla_uf = dt.sigla_uf
GROUP BY d.sigla_uf

UNION

SELECT
    d.sigla_uf,
    'Veiculos Especiais' as categoria,
    AVG(dt. quadriciclo + dt.triciclo) as media,
    MEDIAN(dt. quadriciclo + dt.triciclo) as mediana,
    SUM(dt. quadriciclo + dt.triciclo) as total
FROM detran_frota_br_uf d
JOIN denatran_frota_municipio_tipo dt ON d.sigla_uf = dt.sigla_uf
GROUP BY d.sigla_uf;
```


6) Quais são as regiões geográficas e estados com maior concentração de veículos? Quais possuem menos? E quais particularidades podem ser percebidas nessas regiões? 

Agrupamento dos Veículos por Regiões Geográficas e Estados:

```sql
SELECT
    d.sigla_uf,
    SUM(dt.total) as total_veiculos
FROM detran_frota_br_uf d
JOIN denatran_frota_municipio_tipo dt ON d.sigla_uf = dt.sigla_uf
GROUP BY d.sigla_uf;
```


7) Contagem de Registros por Ano:
Entender como os dados estão distribuídos ao longo dos anos. Essa query vai trazer uma contagem (geral) de veículos por ano.

````sql
SELECT ano, COUNT(*) as quantidade
FROM denatran_frota_municipio_tipo
GROUP BY ano
ORDER BY ano;

Distribuição de Veículos por Estado:
Verificar a distribuição de veículos por estado, através da tabela "detran_frota_br_uf".

SELECT ano, sigla_uf, SUM(total) as total
FROM detran_frota_br_uf
GROUP BY ano, sigla_uf
ORDER BY ano, sigla_uf;

Distribuição de Veículos por Tipo:
Análise da distribuição de veículos por tipo.
SELECT ano, SUM(automovel) as automoveis, SUM(caminhao) as caminhoes
FROM denatran_frota_municipio_tipo
GROUP BY ano
ORDER BY ano;
```

Agrupamento dos veículos em grupos menores (categorias mais específicas) para uma análise mais detalhada. 
Veículos de Passeio e Veículos de Carga:
Analisar a distribuição de carros de passeio em comparação com caminhões e outros veículos de carga:
Veículos de Passeio;
Veículos de Carga.

Veículos de Duas Rodas e Veículos de Quatro Rodas:
Separação dos veículos em duas grandes categorias para entender a distribuição comparando um com o outro:
Veículos de Duas Rodas
Veículos de Quatro Rodas

Veículos por Uso:
Agrupamento dos veículos com base em seu uso:
Veículos de Uso Comercial - caminhões, caminhonete, micro-ônibus; e 
Veículos de Uso Pessoal - automóveis, motocicletas.
