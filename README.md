A análise geoespacial é essencial em diversos setores, como logística, segurança e marketing. Ao trabalhar com coordenadas, é importante traduzir essas informações para um formato que permita a análise eficiente de localizações e distâncias. Neste artigo, vamos explorar ferramentas poderosas para essa tarefa: geohash, H3 index e a fórmula de Haversine. Vamos abordar o que cada um faz, como utilizá-los e quando cada técnica é mais adequada.


1. O Que é Geohash?
Geohash é um método de codificação geográfica que converte coordenadas de latitude e longitude em um código alfanumérico compacto. Esse código representa uma área geográfica em vez de um ponto exato, permitindo que localizações próximas compartilhem prefixos comuns no código, o que facilita a indexação e busca de localizações vizinhas.

Como Funciona?
O geohash divide a superfície terrestre em uma grade de células. Cada célula tem um código alfanumérico, onde as células próximas compartilham prefixos comuns. Esse sistema é eficiente para buscas geoespaciais rápidas e visualizações. Por exemplo, o geohash de São Paulo, Brasil, pode ser algo como 6gy3tw, onde 6gy cobre uma área maior, e 6gy3tw define uma área mais precisa.

Quando Usar Geohash?
Geohash é ideal para sistemas onde é necessário agrupar e indexar rapidamente localizações próximas. É amplamente usado em bases de dados geoespaciais e sistemas de mapas.


2. Entendendo o H3
H3 é um sistema hierárquico de hexágonos desenvolvido pela Uber para representar áreas geográficas com maior precisão e flexibilidade. Ao contrário do geohash, que usa uma grade retangular, o H3 divide a Terra em hexágonos de tamanhos variáveis. Esse formato é útil para análise de clusters, detecção de padrões e cálculos de vizinhança.

Estrutura e Vantagens do H3
Cada hexágono no H3 possui um identificador único e pode ser subdividido em hexágonos menores, o que permite criar áreas detalhadas sem aumentar muito o número de pontos. Isso é ideal para análises onde a forma dos dados importa, como em regiões urbanas densas.

Quando Usar o H3 Index?
O H3 é ideal para representações mais precisas em análises de densidade e clusters, como para segmentação de áreas em cidades, onde a granularidade é importante.



3. A Fórmula de Haversine: Calculando Distâncias entre Pontos Geográficos
A fórmula de Haversine calcula a distância entre dois pontos na superfície da Terra, considerando sua curvatura. É uma ferramenta essencial quando precisamos calcular distâncias diretas entre coordenadas, como para verificar se duas localizações estão próximas.

A fórmula de Haversine é:



Minimizar imagem
Editar imagem
Excluir imagem

fórmula de Haversine
onde:

d é a distância entre os pontos,

R é o raio da Terra (aproximadamente 6371 km),

Δlat e Δlon são as diferenças de latitude e longitude entre os pontos.

Quando Usar a Fórmula de Haversine?
Essa fórmula é ideal para calcular distâncias entre pontos, como para determinar se um cliente está se movimentando de maneira inesperada ou para medir trajetos em análises de mobilidade.



4. Aplicação Prática: Detectando Padrões e Anomalias Geoespaciais
Para ilustrar o uso do H3 index e da fórmula de Haversine, estamos usando um DataFrame simulado com informações geográficas, incluindo colunas de latitude e longitude. Esses dados não são reais; eles representam uma base fictícia onde cada linha corresponde a uma localização geográfica associada a um cliente. O objetivo é criar um modelo de análise geoespacial capaz de identificar padrões em movimentações.

import numpy as np
import h3

# Função para calcular a distância entre dois pontos geográficos usando a fórmula de Haversine
def haversine_distance(lat1, lon1, lat2, lon2):
    R = 6371000  # Raio da Terra em metros
    dlat = np.radians(lat2 - lat1)
    dlon = np.radians(lon2 - lon1)
    a = np.sin(dlat / 2)**2 + np.cos(np.radians(lat1)) * np.cos(np.radians(lat2)) * np.sin(dlon / 2)**2
    c = 2 * np.arctan2(np.sqrt(a), np.sqrt(1 - a))
    return R * c

# Função para determinar a resolução H3 adequada, com base na distância média de cada cliente
def determine_h3_resolution(avg_distance):
    thresholds = [200, 600, 2000, 6000, 15000]  # Valores de limiar para distâncias
    resolutions = [13, 12, 10, 9, 8, 7]         # Resoluções H3 correspondentes
    return resolutions[np.searchsorted(thresholds, avg_distance)]

# Calculando o primeiro ponto válido para cada cliente
df_hist['first_latitude'] = df_hist.loc[df_hist['loc_valid']].groupby('codigoCliente')['latitude'].transform('first')
df_hist['first_longitude'] = df_hist.loc[df_hist['loc_valid']].groupby('codigoCliente')['longitude'].transform('first')

# Calculando a distância de cada ponto ao primeiro ponto registrado do cliente usando a fórmula de Haversine
df_hist['distance'] = np.where(
    df_hist['loc_valid'],
    df_hist.apply(lambda row: haversine_distance(row['latitude'], row['longitude'], row['first_latitude'], row['first_longitude']), axis=1),
    np.nan
)

# Calculando a distância média para definir a resolução H3 ideal para cada cliente
average_distance_df = df_hist.groupby('codigoCliente')['distance'].mean().reset_index()
average_distance_df['resolution'] = average_distance_df['distance'].apply(determine_h3_resolution)

# Mesclando as resoluções H3 calculadas para cada cliente com a base histórica
df_hist = df_hist.merge(average_distance_df[['codigoCliente', 'resolution']], on='codigoCliente', how='left')

# Aplicando o H3 index (h3_index) com base na resolução calculada para cada ponto de localização
df_hist['h3_index'] = df_hist.apply(
    lambda row: h3.latlng_to_cell(row['latitude'], row['longitude'], int(row['resolution'])), axis=1
)


Aplicação do Resultado na Base de Validação: Simulando um Cenário Real
Agora que calculamos e armazenamos as resoluções H3 e os h3_index para cada cliente na nossa base histórica, vamos aplicar esses resultados em uma base de validação. Esse passo simula um caso real de monitoramento, onde temos um histórico conhecido de localizações de clientes e queremos verificar se novas localizações estão de acordo com esses padrões, identificando possíveis anomalias.

Nos passos seguintes, vamos passar a lista de h3_index históricos para cada cliente e verificar se os pontos da base de validação, que representam novas localizações, têm um match com o histórico conhecido. 

df_valid = df_valid.merge(average_distance_df[['codigoCliente', 'resolution']], on='codigoCliente', how='left')
df_valid['h3_index'] = df_valid.apply(lambda row: h3.latlng_to_cell(row['latitude'], row['longitude'], int(row['resolution'])), axis=1)

# Criando uma lista dos h3_index históricos para cada cliente
historico_h3 = df_hist.groupby('codigoCliente')['h3_index'].apply(list).reset_index()
historico_h3.columns = ['codigoCliente', 'h3_index_historico']

# Unindo a lista histórica com a base de validação e verificando a correspondência (h3_match)
df_valid = df_valid.merge(historico_h3, on='codigoCliente', how='left')
df_valid['h3_match'] = df_valid.apply(lambda row: row['h3_index'] in row['h3_index_historico'], axis=1)


Exemplo Prático: Verificação de Correspondência com o Histórico (h3_match)
Na tabela abaixo, podemos ver o resultado da comparação das localizações na base de validação com o histórico conhecido de um cliente (codigoCliente = 1). A coluna h3_index mostra o índice H3 gerado para a localização atual, enquanto h3_index_historico contém a lista dos índices H3 onde o cliente esteve anteriormente.

h3_match = True: Indica que a localização atual coincide com uma das localizações históricas.

h3_match = False: Indica uma nova localização fora do padrão histórico.
