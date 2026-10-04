# System Architecture Overview

## Data Layer (Storage + Ingestion)
* Database: (SQL)
  * Stores structured data about players, injuries, and matches
    * Schema (simplified):
      * ```
          Players(player_id, name, dob, position, height, weight)
          Injuries(injury_id, player_id, type, severity, days_out, age_at_injury, minutes_before, minutes_total)
          Matches(match_id, player_id, minutes_played, date, competition)
        ```
* Data Ingestion:
  * From external sources, mainly Transfermarkt Injury History page for each player
  * ETL piplines clean and normalize the data
  
## Model Layer (Machine Learning Pipline)
* Feature Engineering:
  * Convert injury type &rarr; categorical encoding
  * Normalize days out, age, and minutes played
  * Derive features (injury frequency, recovery ratio)
* Model Training:
  * Train ML model (Random Forest, XGBoost, Survival Analysis)
  * Stored in Model Registry (MLflow, S3, or DB)
* Prediction API:
  * Input: player_id
  * Output: risk score (e.g., "Probability of injury in next 3 months: 33%")

## Application Backend
* Framework: Flask or FastAPI
* Responsibilities:
  * Serve REST API endpoints:
    * `POST /predict` &rarr; returns health prediction for a player
    * `GET /player/{id}` &rarr; fetch player profile + injury history
    * `POST /player/{id}/injury` &rarr; add injury record
  * Call ML model service for predictions
  * Manage user authentication 
* Integration with Database:
  * ORM (SQLAlchemy)

## Frontend Layer
### Options:
* Web app (React or Vue) for production
* Streamlit/Dash for quick prototyping and visualizations

### Features:
* Player Profile Dashboard:
  * Age, position, injury history timeline
  * Minutes played chart
* Health Prediction:
  * Risk score visualization (e.g., guage chart or risk heatmap)
  * Next expected downtime estimate
* What-if Analysis:
  * Simulate adding an injury and see how risk changes

## Deployment Layer
* Containerization:
  * Dockerize backend + ML model
* Cloud Hosting:
  * AWS/GCP/Azure or simple Heroku deployment
* Monitoring:
  * Track API usage and latency
  * Model drift monitoring (are predictions degrading?)

## Data Flow
1. Data Ingestion:
   * Load injury + match data into DB
2. Model Training:
   * Batch jobs (offline) update ML model weekly/monthly
3. Model Serving:
   * Prediction API loads tranined model into memory 
4. User Interaction:
   * Frontend requests prediction &rarr; Backend &rarr; Model &rarr; Result shown

### High-Level Diagram
```
[ Data Sources ]  --->  [ ETL / Data Pipline ]  --->  [ Database ]
                                    |
                                    V
                            [ Model Training ]
                                    |
                                    V
                          [ Prediction Service ]
                                    |
            ------------------------------------------------
            |                                              |
    [ Backend API ]                                [ Model Registry ]
            |
            V
     [ Frontend UI ]                             
```


## Tech Stack Architecture
* Neon &rarr; free hosted Postgres 
* FastAPI &rarr; backend REST API seving predictions and player data
* scikit-learn &rarr; ML model training + inference (bundled in FastAPI)
* React &rarr; Frontend UI for input + visualization
* Hosting:
  * Neon (DB)
    * Postgres 500MB DB with autoscaling up to 2 CU
  * Render (Backend)
    * 750 monthly compute hours
  * Vercel (Frontend)
    

