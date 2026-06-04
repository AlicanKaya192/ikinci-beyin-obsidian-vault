---
tarih: 2026-05-28
konu: Feature Engineering
etiket: [feature-engineering, graf, network, graph, networkx, node2vec, sosyal-ağ]
kaynak: NetworkX Dokümantasyon
zorluk: ⭐⭐⭐
---

## 📌 Özet
Graf/network verisinden özellik çıkarmak; sosyal ağ analizi, dolandırıcılık tespiti, öneri sistemleri ve biyoinformatik gibi alanlarda kritiktir. Node-level, edge-level ve graph-level özellikler üretilebilir.

---

## 🧠 Detay

### NetworkX ile Temel Graf İşlemleri

```python
import networkx as nx
import pandas as pd
import numpy as np

# Graf oluştur
G = nx.Graph()

# Kenarlardan graf
edges_df = pd.DataFrame({
    'kaynak': [1, 1, 2, 3, 4],
    'hedef':  [2, 3, 3, 4, 5],
    'agirlik': [0.8, 0.5, 0.9, 0.3, 0.7]
})
G = nx.from_pandas_edgelist(edges_df, 'kaynak', 'hedef',
                             edge_attr='agirlik')

# Düğüm özellikleri ekle
for node, veri in musteri_df.iterrows():
    G.nodes[node]['yas'] = veri['yas']
    G.nodes[node]['gelir'] = veri['gelir']

print(f"Düğüm: {G.number_of_nodes()}, Kenar: {G.number_of_edges()}")
```

### Düğüm (Node) Özellikleri

```python
def node_ozellikleri(G):
    """Her düğüm için ağ merkeziliği ve yapısal özellikler"""
    ozellikler = {}

    # Derece (bağlantı sayısı)
    derece = dict(G.degree())
    agirlikli_derece = dict(G.degree(weight='agirlik'))

    # Merkezilik ölçüleri
    betweenness = nx.betweenness_centrality(G, normalized=True)
    closeness = nx.closeness_centrality(G)
    eigenvector = nx.eigenvector_centrality(G, max_iter=1000)
    pagerank = nx.pagerank(G, alpha=0.85)

    # Clustering katsayısı (yerel kümelenme)
    clustering = nx.clustering(G)

    # DataFrame oluştur
    df_node = pd.DataFrame({
        'node': list(G.nodes()),
        'derece': [derece[n] for n in G.nodes()],
        'agirlikli_derece': [agirlikli_derece[n] for n in G.nodes()],
        'betweenness': [betweenness[n] for n in G.nodes()],
        'closeness': [closeness[n] for n in G.nodes()],
        'eigenvector': [eigenvector[n] for n in G.nodes()],
        'pagerank': [pagerank[n] for n in G.nodes()],
        'clustering': [clustering[n] for n in G.nodes()]
    })

    return df_node

df_features = node_ozellikleri(G)
```

### Komşu Tabanlı Özellikler

```python
def komsuluk_ozellikleri(G, node):
    """Düğümün komşularına ait istatistikler"""
    komsular = list(G.neighbors(node))

    if not komsular:
        return {
            'komsu_sayisi': 0,
            'komsu_ort_derece': 0,
            'komsu_max_pagerank': 0
        }

    pr = nx.pagerank(G)
    komsu_dereceler = [G.degree(k) for k in komsular]

    return {
        'komsu_sayisi': len(komsular),
        'komsu_ort_derece': np.mean(komsu_dereceler),
        'komsu_max_derece': np.max(komsu_dereceler),
        'komsu_std_derece': np.std(komsu_dereceler),
        'komsu_max_pagerank': max(pr[k] for k in komsular),
        'komsu_ort_pagerank': np.mean([pr[k] for k in komsular])
    }

df_komsuluk = pd.DataFrame([
    {'node': n, **komsuluk_ozellikleri(G, n)}
    for n in G.nodes()
])
```

### Kenar (Edge) Özellikleri

```python
def kenar_ozellikleri(G):
    """Kenarlar için yapısal benzerlik ölçüleri"""
    kenarlar = []

    # Ortak komşu sayısı
    for u, v in G.edges():
        ortak_komsular = len(list(nx.common_neighbors(G, u, v)))
        jaccard = list(nx.jaccard_coefficient(G, [(u, v)]))[0][2]
        adamic_adar = list(nx.adamic_adar_index(G, [(u, v)]))[0][2]
        pref_attach = list(nx.preferential_attachment(G, [(u, v)]))[0][2]

        kenarlar.append({
            'kaynak': u,
            'hedef': v,
            'ortak_komsular': ortak_komsular,
            'jaccard': jaccard,
            'adamic_adar': adamic_adar,
            'pref_attachment': pref_attach
        })

    return pd.DataFrame(kenarlar)

df_kenar = kenar_ozellikleri(G)
```

### Node2Vec — Graf Embedding

```python
# pip install node2vec
from node2vec import Node2Vec

# Rastgele yürüyüş tabanlı embedding
node2vec = Node2Vec(
    G,
    dimensions=64,       # Embedding boyutu
    walk_length=30,      # Her yürüyüş uzunluğu
    num_walks=200,       # Düğüm başına yürüyüş sayısı
    p=1,                 # Return parametresi (BFS/DFS dengesi)
    q=1,                 # In-out parametresi
    workers=4
)

# Modeli eğit
model = node2vec.fit(window=10, min_count=1, batch_words=4)

# Düğüm vektörleri
embedding_dict = {node: model.wv[str(node)] for node in G.nodes()}
df_embed = pd.DataFrame.from_dict(embedding_dict, orient='index',
                                   columns=[f'n2v_{i}' for i in range(64)])
```

### Graf Seviyesi Özellikler

```python
def graf_ozellikleri(G):
    """Tüm grafa ait global istatistikler"""
    return {
        'n_node': G.number_of_nodes(),
        'n_edge': G.number_of_edges(),
        'yogunluk': nx.density(G),
        'ort_derece': np.mean([d for _, d in G.degree()]),
        'ort_clustering': nx.average_clustering(G),
        'bagli_bilesен': nx.number_connected_components(G),
        'cap': nx.diameter(G) if nx.is_connected(G) else -1,
        'ort_yol': nx.average_shortest_path_length(G) if nx.is_connected(G) else -1,
        'asortativite': nx.degree_assortativity_coefficient(G)
    }
```

### Pratik Kullanım: Dolandırıcılık Tespiti

```python
# İşlem ağı oluştur
G_islem = nx.DiGraph()
for _, row in islem_df.iterrows():
    G_islem.add_edge(row['gonderen'], row['alici'], tutar=row['tutar'])

# Dolandırıcılık sinyali olabilecek özellikler
pr = nx.pagerank(G_islem)
in_degree = dict(G_islem.in_degree())
out_degree = dict(G_islem.out_degree())

musteri_df['pagerank'] = musteri_df['id'].map(pr).fillna(0)
musteri_df['giris_derecesi'] = musteri_df['id'].map(in_degree).fillna(0)
musteri_df['cikis_derecesi'] = musteri_df['id'].map(out_degree).fillna(0)
musteri_df['derece_orani'] = (musteri_df['giris_derecesi'] /
                               (musteri_df['cikis_derecesi'] + 1))
```

### Hangi Özellik Ne Zaman?

| Özellik | Anlam | Kullanım |
|---------|-------|----------|
| Derece | Bağlantı sayısı | Popülerlik |
| PageRank | Önemli bağlantı kalitesi | Etki gücü |
| Betweenness | Köprü noktası | Bilgi akışı |
| Clustering | Yerel kümelenme | Topluluk üyeliği |
| Node2Vec | Yapısal benzerlik | Benzer düğüm |

---

## 💡 Bağlantılar
- [[FE - Giriş ve Genel Bakış]]
- [[FE - Özellik Türetme]]
- [[ML - K-Means Kümeleme]]

## ❓ Sorular / Anlamadıklarım
- Node2Vec'in p ve q parametreleri nasıl ayarlanır?
- Büyük graflarda (milyonlarca düğüm) hangi araçlar kullanılır?

## 🔗 Kaynaklar
- https://networkx.org/documentation/stable/
- https://node2vec.readthedocs.io/
