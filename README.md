## Football Network Analysis with Event Data

This project explores **passing and foul networks** in football using **StatsBomb’s event data**. By applying network science, data transformations, and regression modeling, it evaluates how players influence offensive performance—both through passes and by being fouled.

---

## Objective

To understand which players most impact offensive performance by:

- Measuring **centrality** in passing/foul networks.
- Linking network influence to metrics like **xG Chain, xA, Assists**.
- Exploring how **Anti-Football** tactics target key players through fouls.

---

## Dataset

StatsBomb Open Data (2015–2016 La Liga season)

- [StatsBomb GitHub Repo](https://github.com/statsbomb/open-data)
- Focus on event types: **passes, shots, fouls**
- Extracted with **StatsBombPy** and structured using **DuckDB + Pandas**

---

## 🔧 Methods & Metrics

### Data Preprocessing
- Min-max scaling, log and Box-Cox transforms
- Positional standardization for players
- Merging match-level context with event data

### Network Construction
- Built using **NetworkX** and visualized with **Gravis**
- Three types of networks:
  - General **Passing Network**
  - **Goal-Scoring Passing Network** (GSPN)
  - **Foul Network**

### Centrality Measures
- **Degree**: Number of passes/fouls involved
- **Eigenvector**: Player influence in the network
- **Closeness**: Passing speed and availability
- **Betweenness**: Bridging between teammates

### Offensive Metrics
- **xA**: Expected assists
- **xG Chain**: Involvement in goal-scoring plays
- **xG Buildup**: Play buildup excluding the final assist

---

## Analysis Summary

- Strong positive correlation between **eigenvector centrality** and **xG Chain** (+0.7)
- Players with **high foul centrality** often have **high offensive metrics**
- Foul exposure serves as a proxy for player **threat level** and **tactical importance**

---

## Technologies Used

| Category         | Libraries & Tools                         |
|------------------|-------------------------------------------|
| Language         | Python                                    |
| Network Analysis | NetworkX, Gravis                          |
| Data Processing  | Pandas, Numpy, Scikit-learn, Scipy        |
| Visualization    | Matplotlib, Seaborn                       |
| Database         | DuckDB                                    |
| API & ETL        | StatsBombPy                               |

---

## Results

- Passing networks alone can **predict offensive potential**
- Foul networks reveal the **defensive targeting** of playmakers
- **Combining both** provides richer tactical insights

---

## Conclusion

This project confirms that **network centrality in passes** is a strong indicator of offensive contribution, and **foul exposure** can indicate which players opponents perceive as threats. It bridges possession-based analytics with Anti-Football strategies to offer a **dual lens** into modern match dynamics.
