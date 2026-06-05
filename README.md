# tp8-data-lake
## Objectif

Intégrer les architectures lambda et kappa, organiser un data lake en zones (raw, curated, refined) et appliquer des principes de gouvernance des données.

---

## Prérequis

- Python 3.11+

---

## Installation

```bash
cd TPs/TP8_Data_Lake
# Pas de dépendances externes
Exécution
bash
python main.py
Structure
text
TP8_Data_Lake/
├── main.py                 # Orchestration
├── contracts/
│   ├── order.py            # Dataclass Order
│   └── event.py            # Dataclass OrderEvent
├── service/
│   └── order_service.py    # Service synchrone
├── broker/
│   └── mini_broker.py      # Broker partitionné (clé=city)
├── producers/
│   └── event_producer.py   # Producteur événements
├── consumers/
│   ├── consumer_a.py       # Agrégation par statut/ville
│   └── consumer_b.py       # Agrégation par livreur
├── offsets/
│   └── offset_store.py     # Persistance offsets
├── storage/
│   └── partitioned_store.py # Stockage JSONL (city=XXX/)
├── dashboard/
│   └── dashboard.py        # Tableau de bord
├── data/                   # Données produites (city=XXX/)
├── logs/                   # Journaux
└── outputs/                # Résultats
Architecture
text
Client → OrderService (synchrone) → MiniBroker (partition par city)
                                          ↓
                              Consumer A (partitions 0,2)
                              Consumer B (partitions 1,3)
                                          ↓
                              OffsetStore + PartitionedStore
                                          ↓
                                    Dashboard + rapport JSON
Zones du data lake
Zone	Stockage	Description
Raw	data/city=XXX/events.jsonl	Données brutes immutables
Curated	Agrégations en mémoire	Données validées
Refined	rapport_final.json	Résultats analytiques
Résultats
Indicateur	Valeur
Commandes créées	8
Acceptées	6
Rejetées	2
Événements publiés	~30
Fichiers générés :

data/city=Fes/events.jsonl

data/city=Casablanca/events.jsonl

data/city=Rabat/events.jsonl

data/city=Marrakech/events.jsonl

data/city=Tanger/events.jsonl

offsets/offsets_a.json, offsets/offsets_b.json

rapport_final.json

logs/run.log

Dashboard
text
╔────────────────────────────────────────────────────╗
║  TABLEAU DE BORD — 14:30:00 UTC                    ║
╠────────────────────────────────────────────────────╣
║  Commandes créées      : 6                         ║
║  Livrées avec succès   : 4                         ║
║  Livraisons échouées   : 1                         ║
║  Annulées              : 1                         ║
║  Taux de succès        : 66.7%                     ║
╠────────────────────────────────────────────────────╣
║  ACTIVITÉ PAR VILLE                                 ║
║    Casablanca    : 2                               ║
║    Fes           : 2                               ║
║    Rabat         : 1                               ║
╠────────────────────────────────────────────────────╣
║  ACTIVITÉ PAR LIVREUR                               ║
║    CRR-03        : total=2 livré=1 échec=0         ║
║    CRR-07        : total=3 livré=2 échec=1         ║
╠────────────────────────────────────────────────────╣
║  LAG Consumer A : {0: 0, 1: 4, 2: 0, 3: 12}       ║
║  LAG Consumer B : {0: 3, 1: 0, 2: 7, 3: 0}        ║
╚────────────────────────────────────────────────────╝
Difficultés rencontrées
Choix clé de partition : city garantit l'ordre mais peut créer du skew

Architecture lambda vs kappa : compromis entre exactitude batch et rapidité stream

Gouvernance : traçabilité des transformations (lineage)
