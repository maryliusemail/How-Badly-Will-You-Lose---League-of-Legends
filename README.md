# How Badly Will You Lose – League of Legends  
### DSC 80 Final Project – Spring 2025  
By: Ziyan Liu

## Introduction

This project uses professional League of Legends match data from [Oracle’s Elixir](https://oracleselixir.com/match-data/), which contains detailed post-game statistics from major competitive leagues such as the LCS, LEC, LCK, and LPL. The dataset includes **150,588 rows and 163 columns**, with each row representing a single player's performance in a match.

The goal of this project is to predict whether a game was a “stomp” — defined here as a short match (less than 25 minutes) in which one team had a gold lead of at least 1500 at 15 minutes and went on to win, or was behind by that much and lost. Stomps are meaningful because they showcase overwhelming early-game dominance and can reveal patterns in team strategy and execution.

To answer this question, we focus on a subset of early and mid-game features, such as gold, XP, and CS differences at 10 and 15 minutes; kills, assists, and deaths at the same intervals; early objective control (`firstblood`, `firstdragon`, `firstherald`, `firsttower`); and combat impact metrics like `visionscore`, `dpm` (damage per minute), and `damagetakenperminute`. The target column, `is_stomp`, is a binary label derived from game length, result, and gold differential at 15 minutes.
