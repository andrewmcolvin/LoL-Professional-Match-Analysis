 
# League of Legends Pre-Match Analysis

Author: Andrew Colvin

This analysis of League of Legends professional matches was created as a project requirement for UCSD Data Science class DSC80. It aims to explore what can be learned and improved upon for teams in their pre-game strategies. I focus almost exclusively on the data that is attained prior to the actual game starting.

## Introduction

League of Legends is massively popular MOBA style online video game that pits two groups of 5 in a battle to destroy the other team's nexus. Before the chaos ensues, 


Research:

How accurately can a match outcomes be predicted using only pre-match information? That includes any data up until the players load into summoner's rift. 



Following my research question, I decided to remove all columns that contained data collected during the match. Everthing in this list was available from the very start of the game with the exception of `result` which captures the outcome of the match. That column is needed for prediction.

## Data Cleaning and Exploratory Data Analysis
<iframe
  src="team-win-rate-by-draft-order.html"
  width="1300"
  height="600"
  frameborder="0"
></iframe> 


<style>
table {
  border-collapse: collapse;
  width: 100%;
  margin: 1rem 0;
}

th, td {
  border: 1px solid #999;
  padding: 8px 12px;
  text-align: center;
}

th {
  background-color: #f2f2f2;
  font-weight: bold;
}
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th>result</th>
      <th>firstPick</th>
      <th>top_bans</th>
      <th>jng_bans</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>1.0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <td>1</td>
      <td>0.0</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <td>0</td>
      <td>1.0</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <td>1</td>
      <td>0.0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <td>1</td>
      <td>1.0</td>
      <td>1</td>
      <td>0</td>
    </tr>
  </tbody>
</table>


## Assessment of Missingness

## Hypothesis Testing

## Framing a Prediction Problem

## Baseline Model

## Final Model

## Fairness Analysis







