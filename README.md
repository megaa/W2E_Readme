# Wrist temperature to core temperature research

This repository is the code and data related to the research that tries to
find the relationship between temperature values measured by IB101 wristband and those by TempPal (TP).

## Experimental data

All data consist of CSV files in the two folders `data/` and `data_refined/`, including the following 22 days:<br/>
<br/>
`2020-12-15 ~ 2020-12-18`<br/>
`2020-12-22 ~ 2020-12-25`<br/>
`2020-12-28 ~ 2020-12-31`<br/>
`2021-01-04 ~ 2021-01-08`<br/>
`2021-01-11`<br/>
`2021-01-14`<br/>
`2021-01-18`<br/>
`2021-01-20 ~ 2021-01-21`<br/>

Differences of `data/` and `data_refined/`:
  * In the latter, noon section (12:00 ~ 14:xx) is removed manually.
  * In the latter, 10 minutes of initial cold start section is removed manually.
  * In the latter, 3 ~ 5 minutes of cold start sections after BLE reconnection are removed manually.

## Time series plotter and statistics

This is a program that plots the time series graph for a specific day. In addition, it also
outputs some basic statistics of the data.

### Usage (original plotting)

`python3 plot.py -d <date>`<br/>
e.g: `python3 plot.py -d 2020-12-17`

### Usage (refined plotting and Q0 Q1 Q2 Q3 Q4 printing)

`python3 plot.py -r -d <date>`<br/>
e.g: `python3 plot.py -r -d 2020-12-17`<br/>

All data are refined as follows.<br/>

Wrist:
  * Filtered by FFT with >50 high frequency removed
  * Adjusted by `NewValue = aw_a x OldValue + aw_b`
  * Noon section removed according to the corresponding TempPal data
  * Averaged per minute

TempPal:
  * CSV files in `data_refined/` folder are used instead of `data/`
  * For ii, lifted by 0.75% (check Q31 cell in `Source_Data` page of `regression/TP_summary.xlsx`)
  * For iii, lifted by 0.79% (check Q32 cell in `Source_Data` page of `regression/TP_summary.xlsx`)
  * Averaged per minute

Note:<br/>
<br/>
`aw_a` = 0.273<br/>
`aw_b` = 2637.144<br/>
<br/>
Below is the procedure to determine them:
1. Modify `aw_a` to 1 and `aw_b` to 0 in `regression/util.py` first
2. Run `plot.py -r` for all 22 days to print the Q0 Q1 Q2 Q3 Q4 statistics for wrist and 2 TPs for each day
3. Collect the statistics, calculate the overall average for wrist and TP, then calculate the adjustment factors (see the C72 cell in `Overall-Stat` page of `regression/TP_summary.xlsx`)

## Similarity section finder

This is a program that finds from a day sections with designated length (e.g: 20 min) within which the wrist and TP
data satisfy one of the following criteria:
  * For the section, after deducting both wrist and TP data's average, MSE (mean squared error) is the smallest (denoted as MSE matching)
  * For the section, original MSE is the smallest (denoted as MSE_orig matching)

The default is to perform MSE matching. To use MSE_orig matching, please change the line `MSE_orig_matching = False` to `True` in `similarity.py`.
Up to 5 best sections for each TP (i/ii/iii) of the day will be found.

### Usage

`python3 similarity.py -w wrist_ID -n TP_username -s sec_len -p step [-f] [-a] [-i2 adj_TP2] [-i3 adj_TP3] -d date`<br/>
e.g: `python3 similarity.py -w 04ED -n megaa -s 20 -p 1 -f -a -i2 0.75 -i3 0.79 -d 2020-12-29`<br/>
<br/>
Note:<br/>
  * Add `-f` to enable FFT filtering for wrist data
  * Add `-a` to enable `NewValue = aw_a x OldValue + aw_b` adjustment for wrist data
  * `date` can be a comma-separated list of dates to run over a list of days such as `2020-12-29,2020-12-30,2020-12-31`

In `regression/TP_summary.xlsx` and `regression/TP_summary_s20.xlsx`, sections found by this program are collected and sorted. The sorted section lists
are then used in the final regression procedure.

## Regression

This is to run the final regression procedure.

### Usage

`python3 reg2021.py -w wrist_ID -n TP_username -i secSpecFile -s sec_len -p step [-f] [-a] [-i2 adj_TP2] [-i3 adj_TP3]`<br/>
e.g: `python3 reg2021.py -w 04ED -n megaa -i regression/secSpec0222.txt -s 20 -p 1 -f -a -i2 0.75 -i3 0.79`<br/>
<br/>
Note:<br/>
  * Add `-f` to enable FFT filtering for wrist data
  * Add `-a` to enable `NewValue = aw_a x OldValue + aw_b` adjustment for wrist data
  * `secSpecFile` is the file that specifies sections to run regression, one section per line in the format `YYYY-MM-DDTHH:MM_[i|ii|iii]`
  * Check `regression/secSpec0208.txt`, `regression/secSpec0208a.txt`, `regression/secSpec0222.txt`, and `regression/secSpec0222a.txt`
