# Financial Fraud Topologies using Graphs & Neo4j

<p align="center">
  <img src="renderings/deep_fraud_ring_graph.png" alt="Deep Fraud Ring" width="600px" />
  <br>
  <sub>A complex fraud ring showing interconnected fraudulent transactions, cards, and devices.</sub>
</p>


This repository demonstrates how to create a **Financial Fraud Detection** graph in Neo4j using the [IEEE-CIS Fraud Detection dataset](https://www.kaggle.com/c/ieee-fraud-detection). 

By combining transaction details with identity information, the graph helps organizations move beyond simple rule-based systems to detect complex patterns like **fraud rings**, **synthetic identities**, and **layered money laundering** schemes.

This is a classic use case for financial institutions looking to evolve from reactive "individual transaction analysis" to a proactive, network-based fraud management strategy.

<p align="center">
  <img src="renderings/fraud_island_graph.png" alt="Fraud Islands" width="600px" />
  <br>
  <sub>Tightly connected groups of fraudulent activity, known as "Fraud Islands."</sub>
</p>

This example includes two main notebooks:

1. **[Data Ingestion](loader.ipynb):** Loading the IEEE-CIS dataset (Transactions and Identity) into Neo4j, creating nodes for Transactions, Cards, Devices, Emails, and Addresses.
2. **[Specific Use Cases](analysis.ipynb):** Demonstrating key fraud detection queries and algorithms, including:
   - **Exploratory Data Analysis (EDA):** Understanding the distribution of fraud across devices and identifiers.
   - **Pattern Matching:** Identifying suspicious connectivity, such as devices shared by multiple distinct credit cards.
   - **Community Detection:** Using Graph Data Science (GDS) algorithms like Louvain to find "Fraud Islands"—tightly connected groups of fraudulent activity.
   - **Visualizing Fraud:** Rendering the subgraphs of fraud rings for investigation.

### What is Graph-Based Fraud Detection and Why Does It Matter?

Modern financial systems process millions of transactions daily. It is practically impossible for analysts to review every flag manually, and traditional tabular models often miss the "forest for the trees."

**Graph-Based Fraud Detection** is a strategic approach designed to solve the problem of "hidden links." Instead of treating every transaction as an isolated event, it uses real-world context to determine if a transaction is part of a larger fraudulent network. Graph features calculated within Neo4j can also be used as inputs to well established machine learning models in use by financial institutions, improving their accuracy.

It shifts the focus from "Is this transaction weird?" to "Who is this entity connected to?" by considering factors like:

* **Shared Resources:** Are 50 different user accounts logging in from the same Device ID?
* **Indirect Connections:** Did this account send money to an account that shares an email with a known fraudster?
* **Velocity:** How quickly is this community of accounts forming and transacting?

By connecting these dots, graph analysis helps organizations stop chasing false positives and focus their limited resources on the organized crime rings that cause the most damage.

### Data Sources

In this example, we integrate two primary tables from the IEEE-CIS dataset to build a holistic view:

1. **Transaction Data:** Core payment details including transaction amount, product code, and card information (Card Type, Issuer). This forms the backbone of the graph (`(:Transaction)-[:USED_CARD]->(:Card)`).
2. **Identity Data:** Technical signals collected during the transaction, such as **Device ID, IP Address, Screen Resolution, and Operating System**. These properties are crucial for linking seemingly unrelated transactions (`(:Transaction)-[:ON_DEVICE]->(:Device)`).

### The Graph Advantage

<p align="center">
  <img src="renderings/mule_path_graph.png" alt="Mule Fraud Path" width="600px" />
  <br>
  <sub>The path of fraudulent transactions through mule cards and devices.</sub>
</p>

Traditional fraud management relies on flat lists and rules (e.g., "Amount > $10,000"), which often lead to high false positive rates because they treat a legitimate big spender the same as a thief.

By using Neo4j, we can perform **Link Analysis** to answer critical questions:

* **Reachability:** Is this fresh credit card actually linked to a device previously used for fraud?
* **Impact:** If this account is comprised, how many other accounts share its recovery email?
* **Efficiency:** Which "Supernodes" (e.g., a specific IP address) are hubs for 90% of our fraudulent activity?

### The Resulting Schema

With the data loaded, our graph schema looks like the following:

<p align="center">
  <img src="renderings/schema_graph.png" alt="Fraud Graph Schema" width="600px" />
  <br>
  <sub>High-level overview of the Fraud Detection Graph Schema.</sub>
</p>

We include nodes for **Transactions**, **Cards**, **Devices**, **Emails**, and **Addresses**. The relationships capture the action of a transaction utilizing these resources.

#### The Schema

This Neo4j schema models the relationships between financial events (Transactions) and the entities that facilitate them.

Node labels and relationships (as created in `loader.ipynb`):
- **Transaction:** The central event.
- **Card:** Represents the payment instrument (Credit/Debit).
- **Device:** The physical or digital device fingerprint.
- **Email:** Purchaser (`P_emaildomain`) and Recipient (`R_emaildomain`) domains.
- **Address:** Billing locations (Zip code).

Schema outline in Cypher:

```cypher
// Transactions using Cards
(:Transaction)-[:USED_CARD]->(:Card)

// Transactions associated with Devices
(:Transaction)-[:ON_DEVICE]->(:Device)

// Transactions linked to Contact Info
(:Transaction)-[:PURCHASER_EMAIL]->(:Email)
(:Transaction)-[:RECIPIENT_EMAIL]->(:Email)
(:Transaction)-[:BILLED_TO]->(:Address)
```

## Architecture

### Overview

In our solution, the system is built on a **Fraud Knowledge Graph** architecture. Unlike traditional relational databases that struggle with many-to-many relationships (e.g., finding a path from a Transaction to a Device to a different Card), Neo4j allows us to map the complex relationships between payments, identities, and infrastructure in real-time.

### Typical Data Flow

The architecture follows a standard **Extract, Load, Transform (ELT)** pattern tailored for graph structures:

1. **Ingestion (Python):** We load the raw CSV data using `pandas` and the Neo4j Python Driver.
2. **Entity Resolution (Neo4j MERGE):** Data is streamed into Neo4j using `MERGE` logic. This ensures that if a Device ID appears in 1,000 transactions, it is represented as a single node with 1,000 incoming relationships, instantly revealing its importance.
3. **Contextual Enrichment:** Once the base nodes are loaded, we use Graph Data Science (GDS) to calculate metrics like **Community IDs** (identifying rings) and **PageRank** (identifying influential nodes).
4. **Prioritization/Scoring:** The results can be visualized for analysts or fed back into ML models as new features (e.g., "Degree Centrality of associated Device").

### Outward System Integration

The Fraud graph acts as a **central nervous system** for investigation data. Its value is fully realized when insights are pushed back into the tools where analysts work.

#### Investigation Tools
* **Visual Forensics:** Analysts can query Neo4j Bloom or similar tools to visually explore the "blast radius" of a flagged transaction.

#### Real-time Scoring
* **Feature Engineering:** Graph features (e.g., "Number of other cards used on this device in the last 24h") are often strong predictors for ML models (XGBoost/LightGBM) used in real-time authorization.
