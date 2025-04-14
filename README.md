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

## Methods & Metrics

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

---

## Functions

### **network_data_creation**

#### **Function Purpose**
This function processes event-level **pass data from a football match** to build a structured format suitable for **network and sequence analysis**. It adds sequence identifiers, categorizes sequences by outcomes (Goal, Shot, NoShot), tracks time and positional flow, and computes expected goals (xG) features for buildup and shot events.


#### **Parameters**
- `match_pass_data`: DataFrame with pass event data (including timestamps, players, shot results, xG, etc.)
- `timestamp_flag`: If `1`, converts 'timestamp' column to datetime.
- `minute_flag`: If `1`, adjusts second-half timestamps by adding 45 minutes.


#### **Workflow Step-by-Step**

1. **Timestamp Conversion and Sorting**
   - Ensures `timestamp` column is in datetime format.
   - For second-half events (minute ≥ 45), adds 45 minutes to the timestamp (makes full match timeline linear).
   - Sorts events by timestamp chronologically.

2. **Initialization**
   - Sets up lists to collect sequence-level data: sequence IDs, in-sequence index, sequence type, xG, and recipient positions.
   - Tracks player positions dynamically using a dictionary.
   - `current_sequence_id` and `current_in_sequence_id` keep track of sequences and event order inside each sequence.
   - `previous_player_to` is used to detect when a new passing sequence starts.

3. **Main Loop – Building Sequences**
   - Iterates over each pass:
     - Starts a new sequence if the current passer is different from the previous pass recipient.
     - Checks the **last event of the previous sequence** to label it as:
       - `'Goal'` if it ended in a goal.
       - `'Shot'` if it ended in a non-goal shot.
       - `'NoShot'` otherwise.
     - For each pass:
       - Adds its sequence and in-sequence ID.
       - Tracks recipient position using the most recent known position from earlier passes.

4. **Final Row Adjustment**
   - Applies the same goal/shot/noshot logic to the **last row** in the dataset (since the loop ends without processing it like others).

5. **Post-processing**
   - Adds all collected lists as columns to the dataframe.
   - Ensures **sequence type and xG values are repeated** for all events in a given sequence using `.groupby().transform('last')`.
   - Computes `SequenceBuildUpXG`, where all but the last pass in the sequence carry the sequence's xG (buildup effect).
   - Adds `time_past`, which is the number of seconds elapsed within the sequence.

#### **Outputs**
Returns a DataFrame (`df_network`) enriched with:
- **Sequence IDs and in-sequence order**
- **Shot outcome per sequence**
- **xG values (final shot and buildup)**
- **Recipient positions**
- **Time progression within sequences**

---

### **event_data_creation**

#### **Function Purpose**

This function takes raw StatsBomb event data for a given match and **enriches it** with additional contextual information to prepare it for advanced analysis like:
- Possession duration
- Pass filtering and tagging
- Foul pairing (won vs committed)
- Network creation-compatible pass data

#### **Parameters**
- `events_extra_info`: A DataFrame with extra match-specific data (e.g., kickoff time, home/away labels).
- `match_ID`: The match ID used to fetch base event data via the StatsBomb API.

#### **Workflow Step-by-Step**

1. **Load and Merge Data**
  - Retrieves event data using `sb.events(match_id=match_ID)`.
  - Merges with `events_extra_info` using match ID.

2. **Timestamp Handling**
  - Converts 'timestamp' to datetime.
  - Adjusts second-half events by adding 45 minutes (for linear match timeline).

3. **Calculate Possession Time**
  - Extracts timestamp of each possession.
  - Groups by possession and computes **duration** (end time − start time).
  - Merges possession duration back into the main DataFrame.

4. **Infer Most Common Player Position**
  - For each player, finds the **most frequent position** they played using `value_counts().idxmax()`.
  - Joins this information back to the events dataframe.

5. **Expand Pass Locations**
  - Splits 'location' and 'pass_end_location' columns into x/y components.

6. **Filter and Enrich Passes**
  - Keeps only meaningful passes (ignores 'Injury Clearance').
  - Joins shot information to passes via `shot_key_pass_id`.
  - Creates key columns:
    - `pass_success`: If `pass_outcome` is NaN → successful.
    - `f3rd_pass`: Pass into final third.
    - `pass_progression`: Negative x movement (forward passes).
    - `shot_assisted_pass`: Pass led to a shot.
    - `goal_assisted_pass`: Pass led to a goal.
    - `cross`, `box_pass`: Spatial tags for specific pass types.
  - Adds recipient’s most common position.

7. **Network Enrichment**
  - Pass data (`passes_M`) is sent into `network_data_creation()` to assign sequence features.

8. **Foul Committed Handling**
  - Checks which foul-related columns exist in data.
  - Depending on availability of:
    - `foul_committed_advantage`
    - `foul_committed_card`
  - Creates a consistent DataFrame `foul_commited` by filling missing columns with `None`.

9. **Join Foul Won with Foul Committed**
  - For each “Foul Won” event, matches the related “Foul Committed” event via `related_events`.
  - Merges the data, keeping only pairs where committing player exists.

10. **Final Event Formatting**
  - Removes original “Foul Won” and “Foul Committed” events from main data.
  - Sets indices for final dataframes.
  - Concatenates all (base events + enriched passes + fouls) into `events_df`.


#### **Output**
- Returns a **clean, enriched event DataFrame** that:
  - Has all passes tagged for analysis
  - Connects shots to key passes
  - Connects fouls to their counterparts
  - Assigns sequence information for buildup play
  - Provides time-adjusted and position-tagged events


---

### **network_creation_pass**

#### **Function Purpose**

This function constructs a **directed passing network** from successful passes in a match. It uses this network to compute **centrality metrics** (which help describe how influential or connected each player is) and returns:
- A NetworkX graph (`DiGraph`)
- The original pass data (`result_df`) enriched with centrality values


#### **Workflow Step-by-Step**

1. **Filter Relevant Data**
  - Makes a copy of `match_pass_data` to avoid editing the original.
  - Filters only:
    - **Successful passes** (`pass_succes == True`)
    - With a **non-null pass recipient**

  This ensures that only meaningful, trackable connections are used for building the network.

2. **Initialize Directed Graph**
  - `G = nx.DiGraph()`: A directed graph where an edge from Player A to Player B represents a pass **from A to B**.

3. **Build Passing Network**
  - Iterates over each row in the filtered DataFrame.
  - For each pass:
    - If an edge already exists between the passer and receiver → increments its weight.
    - Else, creates the edge with a weight of 1.
    
4. **Compute Centrality Metrics**
  These metrics help quantify **each player's role** within the passing network.

  - **Degree Centrality**: Measures how many connections (in/out) a player has relative to all others.
  - **Eigenvector Centrality**: Measures a player's influence by accounting for how connected their neighbors are.
  - **Closeness Centrality**: Measures how close a player is to all others in terms of path distance.
  - **Betweenness Centrality**: Measures how often a player acts as a bridge in the shortest paths between others.

  These metrics give different views on how **central, influential, or connecting** a player is in the flow of passes.

  - If `eigenvector_centrality` fails to converge due to a poorly connected graph, an exception is handled with a printed message.

5. **Attach Centrality to Pass Data**
  Each of the four centrality dictionaries is turned into a DataFrame:
  - Each DataFrame maps `player` to their centrality score.

  They are then **left-joined** onto the filtered pass data (`df`) to produce an enriched DataFrame (`result_df`), which contains:
  - All original pass info
  - Added columns: `degree_c`, `eigen_c`, `closeness_c`, `betweenness_c`

  This allows analyzing pass-level events with **network insights** attached to each player.


### **Output**
- `G`: The constructed directed passing network
- `result_df`: The original pass data enriched with centrality scores for the passer


---

### **network_creation_foul**

#### **Function Purpose**

This function constructs a **directed foul network** from player foul events in a match. It represents **who committed a foul on whom**, then calculates several **centrality metrics** to quantify each player's role in this foul interaction network. It returns:
- A NetworkX directed graph (`DiGraph`)
- The original foul data (`result_df`) enriched with centrality values

---

#### **Workflow Step-by-Step**

1. **Filter Relevant Data**
  - Makes a copy of `match_foul_data` to preserve the original input.
  - Keeps only rows where `player_foul_commitedJ` (the player fouled) is **not null**.

  This ensures only valid foul relationships are considered for building the network.

2. **Initialize Directed Graph**
  - `G = nx.DiGraph()` creates a directed graph where an edge from Player A to Player B means **Player A committed a foul on Player B**.

3. **Build Foul Network**
  - Iterates over each foul event.
  - For each foul:
    - If the edge from fouler to fouled already exists → increments its weight.
    - Otherwise, adds a new edge with weight 1.

  This models **repeated fouls** between the same pair as stronger connections.

4. **Compute Centrality Metrics**
  These network metrics help identify players who are:
  - Frequently involved in fouls,
  - Central in the flow of foul actions,
  - Key connectors or potential disruptors.

  - **Degree Centrality**: How many connections a player has (either as fouler or fouled).
  - **Closeness Centrality**: How close a player is to all others in terms of foul paths.
  - **Betweenness Centrality**: How often a player lies on the shortest foul paths between others.
  - **Eigenvector Centrality**: Reflects influence, considering both direct and indirect foul links.

  If eigenvector centrality fails to converge (which can happen if the graph is too fragmented or sparse), it's handled safely by assigning `None` to all players and printing a warning.

5. **Attach Centrality to Foul Data**
  - Centrality results are converted into DataFrames: each maps `player` to their score.
  - These are **left-joined** with the original foul data on the `player` column.
  
  The result (`result_df`) is the enriched foul dataset, now containing columns:
  - `degree_c`, `eigen_c`, `closeness_c`, `betweenness_c`

  This enables further foul analysis with added structural network information for each player.

---

### **Output**
- `G`: The directed foul interaction network graph
- `result_df`: The original foul event data enriched with centrality scores for each fouling player

