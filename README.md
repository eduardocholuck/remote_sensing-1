# Regionalização da precipitação e do NDVI para a Caatinga por meio de Clusterização

> **Resumo:** Este repositório implementa um pipeline reprodutível para identificar **regiões homogêneas** de **precipitação (utilizando o CHIRPS V3)** e **NDVI (MOD13Q1.061)** na **Caatinga** via **clusterização não supervisionada**. O número de clusters é escolhido por **Elbow** e **Silhouette**. As saídas incluem mapas categóricos, climatologias mensais por cluster.

---

## 1) Modelo em uma frase
Clusterização espacial (*Agglomerative/Ward*) sobre **features climatológicas** de chuva e NDVI para mapear regiões homogêneas.

---

## 2) Proprietário & Manutenção
- **Autor:** Luiz Eduardo Nunes Cho‑Luck
- **Contato:** choluck.eduardo@gmail.com  
- **Instituições:** PPgCC/UFRN; ClimVar
---

## 3) Finalidade (Intended Use)
- Delimitar **regiões homogêneas** de precipitação e NDVI no domínio da **Caatinga**.  
- Descrever **climatologia mensal** por cluster (painéis no estilo Oliveira, 2014).  
- Quantificar a **concordância espacial** entre clusters de chuva e NDVI.

**Fora de escopo:** previsão operacional/nowcasting; inferência causal; decisões de risco sem validação local; tendências (mantidas como **opcionais** no repo).

---

## 4) Dados utilizados
- **CHIRPS (precipitação)** — mensal, 1981–2023 (~0,05°, WGS84).

  **Entrada:** `dataset/chirps_data/` (GeoTIFF mensais).  

- **MOD13Q1.061 (NDVI / Terra‑MODIS)** — 16‑dias, 250 m (SIN).  
  **Mensal:** média das **duas** composições de 16‑dias do mês; escala int16 ÷ **10 000** → NDVI em [−1, 1].  
  **Consolidação:** NetCDF em `dataset/netcdf_data/` (ex.: `ndvi_mensal.nc`). Os GeoTIFFs brutos podem ficar fora de versão.  

- **Limites/máscara:** `dataset/shape/caatinga_estados.shp`.  

---

## 5) Estrutura real do repositório
```
dataset/
  chirps_data/          # entrada CHIRPS (.tif)
  netcdf_data/          # saídas intermediárias (chuva/NDVI/features/clusters)
  shape/                # shapefiles (caatinga_estados.shp)
  sst_global/           # legado (não utilizado)
env/
  environment.yml
outputs/
  climatologia/         
  maps/                 
trash/                  # arquivos não utilizados
01_preprocessamento_dados.ipynb
02_quantis_classificacao.ipynb
03_freq_intensidade.ipynb
04_tendencia_mann_kendall.ipynb   # opcional (não discutido)
05_cluster_regioes.ipynb
06_clusters_alternativos.ipynb
07_ndvi.ipynb
README.md
```

---

## 6) Pré‑processamento (ligação com os notebooks)
- **01_preprocessamento_dados.ipynb**  
  CHIRPS: empilhamento mensal → NetCDF (`time, lat, lon`), ordenação `lat/lon`, máscara da Caatinga.  
  NDVI: média 2 composições/mês, conversão de escala (÷10 000), QA opcional; reprojeção para WGS84/grade CHIRPS **ou** processamento em grade nativa (250 m).

- **02_quantis_classificacao.ipynb**  
  Definição de quantis (ex.: **P95/P99**) e limiares.

- **03_freq_intensidade.ipynb**  
  Cálculo de **frequência** e **intensidade** por período (anual/sazonal) e mapas de significância (opcional).

**Climatologias**  
- Chuva: mensal (média), **anual** (soma por ano → média entre anos), sazonal (DJF/MAM/JJA/SON; **média**).  
- NDVI: mensal e sazonal (média).

---

## 7) Algoritmos & Features (clusterização)
- **05_cluster_regioes.ipynb**  
  
  - **Algoritmo:** `AgglomerativeClustering(linkage="ward", metric="euclidean")`.  
  
  - **k:** escolhido por **Elbow (inércia)** e **Silhouette** (k∈[3,8]).  
  
  - **Features por pixel:** `longterm_mean`, `season_DJF`, `season_MAM`, `season_JJA`, `season_SON` (médias sazonais).  
  
  - **Padronização:** `StandardScaler` (máscara aplicada antes do ajuste).

- **06_clusters_alternativos.ipynb** (comparativo)  
  - **DBSCAN / HDBSCAN** sobre dados padronizados (sem k fixo; possibilidade de classe “ruído”). Discutir parâmetros (`eps`, `min_samples`).

> **Tendências (opcional):** **04_tendencia_mann_kendall.ipynb** computa MK/Sen; não entram na discussão final, podendo ser usadas apenas como *features* se necessário.

- **07_ndvi.ipynb**  
  Processamento NDVI (mensal), *features*, clusterização (ex.: k=6) e comparação com clusters de chuva.

---

## 8) Métricas & Validação
- **Qualidade do particionamento (Ward):** curvas de **Elbow** e **Silhouette**.  

---

## 9) Saídas

- **NetCDF:** `*_mensal.nc`, `*_features.nc`, `*_clusters.nc`.  

- **Figuras:** mapas categóricos (fundo branco, ticks inteiros), painéis de climatologia mensal por cluster (precip: 0–450 mm; NDVI: 0–1), Elbow/Silhouette, mapas de concordância.  

- **Tabelas:** estatísticas por cluster (área, anual/NDVI médio, mês de pico, amplitude).

---

## 10) Limitações

- Reamostragem NDVI 250 m → ~5 km reduz detalhe; análises nativas preservam fronteiras finas.  

- NDVI pode **saturar**; CHIRPS pode ter vieses orográficos/costeiros.  

- Definição de **k** é heurística (Elbow/Silhouette) e depende do objetivo.  

- Associação espacial ≠ causalidade.

---

## 11) Quickstart
```bash
# 1) ambiente
conda env create -f env/environment.yml
conda activate caatinga-clusters

# 2) dados
# dataset/chirps_data/  dataset/netcdf_data/  dataset/shape/
# (NDVI GeoTIFFs brutos podem ficar fora de versão; consolidar em netcdf_data)

# 3) rodar notebooks

# 01_preprocessamento_dados.ipynb
# 02_quantis_classificacao.ipynb
# 03_freq_intensidade.ipynb
# 05_cluster_regioes.ipynb
# 06_clusters_alternativos.ipynb (opcional)
# 07_ndvi.ipynb
# 04_tendencia_mann_kendall.ipynb (opcional)
```

---

## 12) Referências
- Funk, C. et al. (2015) — CHIRPS.  
- Didan, K. (2021) — MOD13Q1.061 (MODIS Vegetation Indices).  
- Oliveira, P. T. de. (2014) — *Estudo estatístico sobre eventos de precipitação intensa no NEB* (Tese, UFRN).

> Cite este repositório e as fontes de dados originais ao reutilizar o código.
