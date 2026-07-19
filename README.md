# Football Moneyball — Which Strikers Is The Market Mispricing?

## Question
Moneyball showed that baseball's transfer market systematically 
mispriced players — paying for reputation rather than actual 
statistical output. Does the same pattern exist in football's 
striker market? Which forwards are being significantly over or 
undervalued relative to what their 2024/25 performance data 
actually justifies?

## Methodology

### Data Sources
- **Performance:** FBRef 2024/25 Big 5 European Leagues player 
  statistics (via Kaggle — hubertsidorowicz)
- **Market values:** Transfermarkt 2025/26 valuations (via Kaggle 
  — davidcariboo/player-scores)

### The One-Year Lag
Performance data from 2024/25 is used to predict market values 
from 2025/26 — not the same season. This is the core Moneyball 
methodology: the market observes last season's performance and 
prices players accordingly the following season. The same lag 
structure was used in the original baseball Moneyball salary 
model (batting stats from year X predicting salary in year X+1).

### Sample
Pure forwards (Pos == 'FW') in the Big 5 leagues with 900+ 
minutes played and aged 21 or above — 167 players in total. 
Players under 21 were excluded because their market values are 
driven by potential rather than current statistical output, which 
this model does not attempt to capture.

### Model
OLS regression with ln(market value) as the dependent variable. 
Log-transforming market value compresses the extreme skew caused 
by superstar valuations and is consistent with the Moneyball 
baseball salary model.

**Predictors (14 variables):**

Structural:
- Age and Age² — captures the non-linear peak/decline value curve
- Big club dummy — 1 if playing for a top 20 European club
- PPM — team points per match (proxy for team quality)

Attacking output:
- Goals per 90
- Non-penalty xG per 90
- Goals minus xG per 90 (finishing efficiency above/below expectation)
- Assists per 90

Chance creation:
- Expected assisted goals per 90 (xAG)
- Shot creating actions per 90

Ball progression:
- Progressive carries per 90

Shooting quality:
- Shots on target percentage
- npxG per shot (average chance quality)

Physical:
- Aerial duels won percentage

### Variable Selection
All additional variables were tested against the base model using 
adjusted R² — only variables that genuinely improved adjusted R² 
were retained. Att Pen per 90 was tested and rejected as it 
reduced adjusted R².

### Model Performance
- R² = 0.687 — performance stats explain 69% of market value 
  variation
- Adj R² = 0.658
- F-statistic: 23.78, p < 0.001

### Merging Strategy
Players were matched between FBRef and Transfermarkt on name + 
club (primary), with a unique-name-only fallback for unmatched 
players. This prevents duplicate name collisions — for example, 
two players named Vitinha at different clubs being matched to 
the wrong Transfermarkt record.

## Key Findings

### Most Overvalued Forwards
Players above the diagonal line — actual value significantly 
exceeds what 2024/25 stats justify:

| Player | Squad | Actual | Predicted |
|---|---|---|---|
| Antoine Semenyo | Bournemouth | €65m | €14m |
| Mohammed Kudus | West Ham | €55m | €13m |
| Jarrod Bowen | West Ham | €35m | €9m |
| Jean-Philippe Mateta | Crystal Palace | €40m | €11m |
| Lautaro Martínez | Inter | €85m | €25m |
| Erling Haaland | Man City | €200m | €58m |

Note: the Premier League cluster (Semenyo, Kudus, Bowen, Mateta) 
likely reflects the Premier League broadcast premium rather than 
genuine overvaluation — see Limitations.

### Most Undervalued Forwards
Players below the diagonal line — actual value significantly 
below what 2024/25 stats justify:

| Player | Squad | Actual | Predicted |
|---|---|---|---|
| Gonçalo Ramos | PSG | €35m | €127m |
| Abdallah Sima | Brest | €4m | €20m |
| Dani Raba | Leganés | €2.4m | €11m |
| Josué Casimir | Le Havre | €3.5m | €14m |
| Randy Nteka | Rayo Vallecano | €1.2m | €7m |

### Model Validation — Gonçalo Ramos
The model's most dramatic finding — Ramos predicted at €127m 
when Transfermarkt valued him at €35m — was partially validated 
when AC Milan signed him in summer 2026 for a reported €74m fee. 
The model correctly identified the direction and scale of the 
undervaluation; the market subsequently corrected. The model was 
not right — but it pointed in the right direction.

## Limitations

**1. Superstar premium**
The model systematically underestimates elite superstars. 
Haaland at €200m vs €58m predicted reflects commercial value, 
global brand, and media profile that no statistical model can 
capture. This is a known limitation of regression-based 
valuation models.

**2. Premier League broadcast premium**
The overvalued Premier League cluster likely reflects the 
Premier League's global broadcast revenues inflating all player 
values beyond what pure statistical output justifies. The model 
uses PPM as a team quality proxy but does not fully capture 
league-level commercial value. These players may not be 
genuinely overvalued — they may be correctly priced for the 
Premier League market specifically.

**3. Reputation lag**
Lautaro Martínez at €85m vs €25m predicted reflects a player 
whose market value still reflects peak-season reputation while 
his 2024/25 statistical output was below par. The model prices 
current performance; the market prices career trajectory.

**4. Rotation player penalty**
Players at elite clubs who rotate accumulate fewer minutes and 
less visibility, suppressing their Transfermarkt value even when 
their per-90 output is elite. The model rewards statistical 
efficiency; the market penalises squad role.

**5. Opposition quality**
The model does not adjust for opposition strength. A forward 
scoring prolifically for a relegated club faces systematically 
weaker defences than one scoring at a Champions League club. 
The undervalued list is dominated by small-club and relegated-
club players partly for this reason.

**6. OLS ceiling**
OLS regression has a natural ceiling for this type of 
prediction. A proper player valuation model would use ML 
techniques — gradient boosting, random forests, or neural 
networks — with feature engineering, cross-validation, and 
regularisation. This project is a Version 1 applying the 
Moneyball methodology directly to football. A more 
sophisticated ML version is planned once the relevant 
techniques are covered in the University of Michigan Sports 
Performance Analytics Specialisation.

**7. Forwards only**
This model covers pure forwards only. Defenders, midfielders, 
and goalkeepers require entirely different predictor sets and 
are not included. A future version will extend to all positions.

## Data
The merged dataset is not included in this repository due to 
file size. To reproduce it, download the following files from 
Kaggle and run the notebook:

- **FBRef 2024/25 player stats:**
  https://www.kaggle.com/datasets/hubertsidorowicz/football-players-stats-2024-2025
  Download: `players_data-2024_2025.csv`

- **Transfermarkt player valuations:**
  https://www.kaggle.com/datasets/davidcariboo/player-scores
  Download: `players.csv` and `player_valuations.csv`

Place all three files in the same directory as the notebook and 
run cells sequentially. The full pipeline from raw data to 
final chart runs in under 2 minutes.

## Tools Used
Python · Pandas · NumPy · Statsmodels · Matplotlib · 
Jupyter Notebook

## Author
Rafiz Rayhan
linkedin.com/in/rafiz-rayhan
github.com/rafizrayhan
