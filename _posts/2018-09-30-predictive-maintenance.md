---
title: Predictive Maintenance
excerpt: Predict machine failure of Kaufland forklifts.
published: true
date: 2018-09-30
categories: project
tags: intern predictive maintenance data-science
---
I participated in Global Datahon 2018.

[Dataset](https://www.datasciencesociety.net/global-datathon-2018/kaufland-case/)

[Solution](https://www.datasciencesociety.net/similar-but-not-the-same-an-autoregressive-recurrent-neural-network-with-long-short-term-memory-units-for-forecasting-similar-time-series/)


## <strong>1. Business Understanding</strong>
As part of the Kaufland challenge, we are tasked to build a predictive maintenance model to avert machine failure. Internet of Things (IoT) sensors allow us to monitor the machine’s performance in real time. The model can detect when the machine performs anomalously and suggest the need for action on specific parts of the machine that is about to fail. A predictive system that can detect anomalies in the machine’s behaviour just before it breaks down will help minimise maintenance costs. Compared to a preventive maintenance system, a predictive system is able to reduce costs associated with preventive maintenance done even when maintenance is not required. The business success criteria would therefore be to lower maintenance costs relative to a preventive maintenance system.

Predictive maintenance is a strategy to reduce cost of:
<ul>
 	<li>Labour for skilled repairman</li>
 	<li>Preventative part replacement</li>
 	<li>Post damage repair which might be higher than preventive maintenance</li>
 	<li>Overcome the availability of experts to repair machines</li>
</ul>
The predictive system will be a success if it detects anomalies accurately.

Our project plan is summarised in Figure 1. The first step consists of exploratory data analysis and data preparation. Based on the first step, we decide on our modelling approach and then construct a strategy for deployment.

<img class=" wp-image-7460 aligncenter" src="https://www.datasciencesociety.net/wp-content/uploads/2018/09/business-understanding.png" alt="" width="541" height="304" />
<p style="text-align: center"><em>Figure 1: Project plan</em></p>
In this project, we have access to a dataset of sensor data (velocity and acceleration) attached to seven machines over a period of 5 months to two years.

The tools that we consider for this purpose are Python and R, which are popular languages for statistical and data analysis. Given that our team has mixed proficiency in both languages, we conduct our analysis in a mix of both. The models that we consider for this purpose at the preliminary stage are Long-Short-Term Memory (LSTM) models.

The goal is not to determine how much vibration a machine will withstand before failure. The aim is to obtain a trend in vibration characteristics that can warn of trouble. For example a sudden short peak or increase does not necessarily means the machine is going to fail, it might be caused by sensor error. Instead, look for odd occurrence that persists.

&nbsp;
## <strong>2. Data Understanding</strong>
The data is loaded in R and Python. The data we use is provided by Kaufland, a large hypermarket chain based in Germany. We are provided with data on 7 separate automated storage and retrieval systems machines. Each machine has 6 different sensors (Figure 2). Data from these machines are collected for different time ranges:
<ul>
 	<li>2 years for Machine 1 and 7 (RBG1 and 7).</li>
 	<li>5 months for the other machines.</li>
</ul>
RBG 1 and 7 are the pilot study to test the efficiency of the sensors, this explains why they have long range of data. Sensors are embedded to the other machines after the sensors are proved to be working.

<img class=" wp-image-7464 aligncenter" src="https://www.datasciencesociety.net/wp-content/uploads/2018/09/machine.jpg" alt="" width="547" height="411" />
<p style="text-align: center"> <em>Figure 2: Sensors on machine. Image from Data Science Society.
</em></p>
The sensors attached to the machine are as follows:

1. Lifting mechanism:
<ul>
 	<li>Sensor 1: Lifting motor</li>
 	<li>Sensor 2: Lifting gear</li>
</ul>
2. Drive mechanism:
<ul>
 	<li>Sensor 3: Drive motor</li>
 	<li>Sensor 4: Drive gear</li>
 	<li>Sensor 5: Drive wheel</li>
</ul>
3. Sensor 6: Idle (non-powered) wheel

These six sensors measure the effective velocity of vibrations and the maximum acceleration of shocks, collected under controlled conditions thrice a day.

Figure 3 shows a sample of data from the drive gear sensor on Machine 1. All these sensors measure both the effective velocity of vibrations and the maximum acceleration of shocks. Each sensor collects data under controlled conditions three times a day.  Two of the machines have collected data for almost two years, the other five for five months.
<table>
<tbody>
<tr>
<td></td>
<td><b>machine_name</b></td>
<td><b>sensor_type</b></td>
<td><b>date_measurement</b></td>
<td><b>realvalue</b></td>
<td><b>unit</b></td>
<td><b>start_time</b></td>
<td><b>end_time</b></td>
<td><b>day_of_week</b></td>
<td><b>sensor_group</b></td>
</tr>
<tr>
<td><b>0</b></td>
<td>RBG1</td>
<td>drive_gear_V_eff</td>
<td>2/9/2016</td>
<td>0.395</td>
<td>mm/s</td>
<td>26:42.8</td>
<td>26:42.8</td>
<td>Friday</td>
<td>driving_motor</td>
</tr>
<tr>
<td><b>1</b></td>
<td>RBG1</td>
<td>drive_gear_V_eff</td>
<td>2/9/2016</td>
<td>0.577</td>
<td>mm/s</td>
<td>26:45.7</td>
<td>26:45.7</td>
<td>Friday</td>
<td>driving_motor</td>
</tr>
<tr>
<td><b>2</b></td>
<td>RBG1</td>
<td>drive_gear_V_eff</td>
<td>2/9/2016</td>
<td>0.717</td>
<td>mm/s</td>
<td>26:48.5</td>
<td>26:48.5</td>
<td>Friday</td>
<td>driving_motor</td>
</tr>
<tr>
<td><b>3</b></td>
<td>RBG1</td>
<td>drive_gear_V_eff</td>
<td>2/9/2016</td>
<td>0.832</td>
<td>mm/s</td>
<td>26:51.3</td>
<td>26:51.3</td>
<td>Friday</td>
<td>driving_motor</td>
</tr>
<tr>
<td><b>4</b></td>
<td>RBG1</td>
<td>drive_gear_V_eff</td>
<td>2/9/2016</td>
<td>0.941</td>
<td>mm/s</td>
<td>26:54.1</td>
<td>26:54.1</td>
<td>Friday</td>
<td>driving_motor</td>
</tr>
<tr>
<td><b>5</b></td>
<td>RBG1</td>
<td>drive_gear_V_eff</td>
<td>2/9/2016</td>
<td>1.042</td>
<td>mm/s</td>
<td>26:56.9</td>
<td>26:56.9</td>
<td>Friday</td>
<td>driving_motor</td>
</tr>
</tbody>
</table>
<p style="text-align: center"><em>Figure 3: Data from drive gear sensor of Machine 1.</em></p>

## <b>3. Data Exploration</b>
Data exploration is performed with purpose of finding relationships among factors to motivate a modelling approach.

The values of vibrations of sensors for each sensor is plotted using scatter plot (Figure 4 and 5). The values of six sensors are included. Null values are not shown in the figures.

<img class=" wp-image-7485 aligncenter" src="https://www.datasciencesociety.net/wp-content/uploads/2018/09/RBG1_Vibration_Hourly_Points.png" alt="" width="600" height="600" />
<p style="text-align: center"><em><span class="TextRun SCXW50323285" lang="EN-MY" xml:lang="EN-MY"><span class="NormalTextRun SCXW50323285">Figure 4: Values of sensors for Machine 1 (RBG1). This machine shows a significant decrease in mean vibration read after a documented maintenance event on 8 Feb 2018.</span></span></em></p>
&nbsp;

<img class=" wp-image-7473 aligncenter" src="https://www.datasciencesociety.net/wp-content/uploads/2018/09/RGB7.png" alt="" width="605" height="605" />
<p style="text-align: center"><em><span class="TextRun SCXW70652362" lang="EN-MY" xml:lang="EN-MY"><span class="NormalTextRun SCXW70652362">Figure 5: Values of sensors for Machine 7 (RBG7)</span></span></em></p>
<span class="TextRun SCXW56761986" lang="EN-MY" xml:lang="EN-MY"><span class="NormalTextRun SCXW56761986">Correlation matrix is plotted in Figure 6. Matrix is plotted against each sensor. The sensors are grouped together based on location of the sensors. Group 1 consists of sensor 1 and 2; group 2 sensor 3, 4 and 5; group 3 sensor 6. Sensors are highly correlated within groups.</span></span>

<img class="wp-image-7476 aligncenter" src="https://www.datasciencesociety.net/wp-content/uploads/2018/09/correl.png" alt="" width="744" height="302" />
<p style="text-align: center"><em>Figure 6: Correlation matrix of sensors  from Machine 1. Grey to black indicates high correlation score.</em></p>
The time series is plotted for each sensors of each machine. No seasonality/repeating patterns/cycles is found. Spectral analysis on Figure 7 confirms the absence of periodicity within the data. This provides a strong case to abandon ARIMA method. At this stage, we consider not using classical time series approaches in favour of Long-Short-Term Memory (LSTM) unit for recurrent neural networks (RNN).

<img class=" wp-image-7494 aligncenter" src="https://www.datasciencesociety.net/wp-content/uploads/2018/09/spectral.png" alt="" width="519" height="519" />
<p style="text-align: center"><em>Figure 7: Spectral analysis of Sensor 1 from Machine 1.</em></p>
Figure 8 shows the plot according to sensor for 7 machines.  The patterns in the series suggests that spread may be an indication of the health of the machine. This forms the basis of our study.

<img class="wp-image-7712 aligncenter" src="https://www.datasciencesociety.net/wp-content/uploads/2018/09/Screen-Shot-2018-09-30-at-4.13.16-PM.png" alt="" width="780" height="509" />
<p style="text-align: center"><em>Figure 8: Values of Driver Gear Sensor from 7 machines are plotted at the same axis.</em></p>
&nbsp;

Extreme values in the data are filtered using Tukey's method. Figure 9 illustrates the occurrence rate of outliers that are present in the sensor data aggregated by machine and sensor type. Visual inspection indicates that Machine 7 shows a significantly higher occurrence of outliers, followed closely by Machines 2 and 3. Interpreting the occurrence of outliers as an indicator of machine reliability, we hypothesize that machine health for Machines 1, 4, 5, and 6 were superior to that of the remaining machines during the period of data recorded. We thus propose that a controlled experiment be designed to test said hypothesis and identify supplementary factors that may contribute towards overall machine health.

<img class="size-full wp-image-7738 aligncenter" src="https://www.datasciencesociety.net/wp-content/uploads/2018/09/Rplot01.jpeg" alt="" width="800" height="700" />
<p style="text-align: center"><em>Figure 9: Distribution of outliers
</em></p>

## <strong>3. Data Preparation</strong>
Operations below are performed on the data:
<ol>
 	<li>Datetime variables are converted into machine readable variables.</li>
 	<li>Data aggregation is performed to reduce number of rows as well as fill in missing values. Mean is calculated for time interval of 1 minute.</li>
 	<li>Some columns are grouped together for exploratory data analysis purpose.</li>
 	<li>Outlier removal using standard deviation (Tukey’s Hinges). Any departure from standard operating characteristics are removed.</li>
 	<li>Moving variance is calculated with window length of 10 minutes.</li>
</ol>
## 4. Modelling and Evaluation
### A. AR-LSTM approach
#### <strong>i. Modeling</strong>
To detect anomalies in a sensor, we first identify periods for each machine which appear to be well-behaved, fit an AR(10) autoregressive model (10 lags of the variable of interest) after pooling well-behaved series, then forecast the values. If the difference between the forecasted and actual value is ‘high’, the observation will be highlighted as an anomaly.

We begin by modelling the velocity measure, a common parameter for analysing vibration (Scheffer &amp; Girdhar 2004), of the Idle Wheel sensor. This is done by training on data pooled time series data between March 2018 and July 2018, where data is available across all machines. Pooling time series data is not a new idea in the time series literature, although scarcely explored (see Maharaj and Inder 1999). While the functioning of the machine is unknown, the maintenance events (Figure 10) are the next-best option for labelling the data. These models are evaluated by back-casting the series and cross-checking against maintenance events to identify false positives and false negatives.

We consider the use of neural network models for sequence data, particularly recurrent neural network (RNN) models. Traditional RNN units are unable to remember long-term dependencies and susceptible to the vanishing gradient problem, and for this purpose LSTM units may be more suitable.

&nbsp;
<table>
<tbody>
<tr>
<td style="width: 132.083px"><b>machine_name</b></td>
<td style="width: 188.05px"><b>date</b></td>
<td style="width: 416.483px"><b>repair_description</b></td>
</tr>
<tr>
<td style="width: 132.083px">RBG 1</td>
<td style="width: 188.05px">13.11.2017</td>
<td style="width: 416.483px">Correction of skewed movement of the machine on the rail</td>
</tr>
<tr>
<td style="width: 132.083px"></td>
<td style="width: 188.05px">08.02.2018</td>
<td style="width: 416.483px">Change of idle wheel</td>
</tr>
<tr>
<td style="width: 132.083px"></td>
<td style="width: 188.05px">12.02.2018 - 14.02.2018</td>
<td style="width: 416.483px">Grinding of the rail</td>
</tr>
<tr>
<td style="width: 132.083px">RBG 2</td>
<td style="width: 188.05px">02.04.2018 - 08.04.2018</td>
<td style="width: 416.483px">Grinding of the rail</td>
</tr>
<tr>
<td style="width: 132.083px">RBG 3</td>
<td style="width: 188.05px">02.04.2018 - 08.04.2018</td>
<td style="width: 416.483px">Grinding of the rail</td>
</tr>
<tr>
<td style="width: 132.083px"></td>
<td style="width: 188.05px">05.04.2018</td>
<td style="width: 416.483px">Change of bearings of drive motor</td>
</tr>
<tr>
<td style="width: 132.083px">RBG 7</td>
<td style="width: 188.05px">02.04. 2018  - 08.04.2018</td>
<td style="width: 416.483px">Grinding of the rail</td>
</tr>
</tbody>
</table>
<p style="text-align: center"> <em>Figure 10: Repair description for maintenance that has occurred.
</em></p>
As each of these series have different centers and spread, we assume that the number of standard deviations away from the mean is similarly distributed and can be ‘learned’ in the same way (Figure 11).

<img class="wp-image-7514 aligncenter" src="https://www.datasciencesociety.net/wp-content/uploads/2018/09/Screen-Shot-2018-09-30-at-1.13.02-AM.png" alt="" width="535" height="369" />

<img class=" wp-image-7508 aligncenter" src="https://www.datasciencesociety.net/wp-content/uploads/2018/09/Screen-Shot-2018-09-30-at-1.12.22-AM.png" alt="" width="660" height="506" />
<p style="text-align: center"><i>Figure </i><i>11: </i><i></i><i>Distribution of Idle </i><i>Wheel velocity (standardised) </i><i>across all machines </i>and <i>Time series of each of the machines' Idle Wheel velocity (standardised)</i></p>

### <strong>ii. Evaluation</strong>
<strong>Evaluation of forecast model  </strong>

The AR-LSTM model provides forecasts that correspond to the level and spread of data. The spread of the AR-LSTM forecasts (Figure 12) are rather conservative, and the AR-LSTM model performs similarly well out-of-sample.

<img class="wp-image-7510 aligncenter" src="https://www.datasciencesociety.net/wp-content/uploads/2018/09/Screen-Shot-2018-09-30-at-1.12.41-AM.png" alt="" width="604" height="336" />
<p style="text-align: center"><i>Figure </i><i>12:</i><i> In-sample performance</i></p>
&nbsp;

<strong>Evaluation of classification criteria  </strong>

Figure 13 plots the AR-LSTM forecast errors. We set 3 standard deviations above and below the mean as the threshold for anomalous behaviour.  This standard is convenient to interpret as our data is already scaled. This corresponds to approximately 95% of the data, assuming the data is normally distributed. This cut-off rule has a relatively low false positive rate of roughly 18/7000 (0.2%).

&nbsp;

<img class="wp-image-7558 aligncenter" src="https://www.datasciencesociety.net/wp-content/uploads/2018/09/Screen-Shot-2018-09-30-at-1.49.07-AM.png" alt="" width="544" height="360" />
<p style="text-align: center"><i>Figure </i><i>13: </i><i>Absolute value of forecast error (standard deviations)</i></p>

### B. Moving variance approach
### i. Models
#### Motivation
We depart from our initial approach of fitting an autoregressive model using LSTM because w<span class="TextRun SCXW192680903" lang="EN-MY" xml:lang="EN-MY"><span class="NormalTextRun SCXW192680903">e cannot easily evaluate our model as the data is unlabelled. Even though a maintenance record is provided, </span></span><span class="TextRun SCXW192680903" lang="EN-MY" xml:lang="EN-MY"><span class="NormalTextRun SCXW192680903">the functioning of the machine </span></span><span class="TextRun SCXW192680903" lang="EN-MY" xml:lang="EN-MY"><span class="NormalTextRun SCXW192680903">remains an unobserved variable. Specifically, w</span></span>e don't have an idea of machine health, so we can't evaluate AR-LSTM. Since our EDA suggests we can approximate machine health using the variation in the series, we assume variation to be a general indication of machine health and use LSTM to detect anomalous patterns in the past hour of data.

At a high level, the differences between these two approaches are as follows:
<table width="100%">
<tbody>
<tr>
<td width="22%"></td>
<td width="38%"><strong>AR-LSTM</strong></td>
<td width="38%"><strong>Moving variance approach
</strong></td>
</tr>
<tr>
<td width="22%"><strong>Predictors</strong></td>
<td width="38%">Lagged measure (velocity)</td>
<td width="38%">Moving variance of lagged measures</td>
</tr>
<tr>
<td width="22%"><strong>Outlier threshold</strong></td>
<td width="38%">Standard deviation</td>
<td width="38%">Median which is less sensitive to outlier is used</td>
</tr>
</tbody>
</table>
<p style="text-align: center"><em>Figure 15: Major differences in approach </em></p>
Our use of the spread as a measure of machine health is justified as follows. Figure 16 and 17 show the time series of idle wheel velocity and the moving average respectively, pooled across all machines. It suggests that there are distinct intervals in the data with different spread and centers. This implies that spread may be a good way to indicate machine health.

<img class="wp-image-7715" src="https://www.datasciencesociety.net/wp-content/uploads/2018/09/Screen-Shot-2018-09-30-at-4.13.25-PM.png" alt="" width="435" height="311" />
<p style="text-align: center"><em>Figure 16: Minute-aggregated time series of idle wheel velocity (machine 1)</em></p>
<img class="wp-image-7714" src="https://www.datasciencesociety.net/wp-content/uploads/2018/09/Screen-Shot-2018-09-30-at-4.13.29-PM.png" alt="" width="502" height="336" />
<p style="text-align: center"><em>Figure 17: Moving Variance of Idle Wheel velocity (machine 1)</em></p>
<span class="TextRun SCXW82747640" lang="EN-MY" xml:lang="EN-MY"><span class="NormalTextRun SCXW82747640">We begin by modelling the velocity measure, a common parameter for analysing vibration (</span></span><span class="TextRun SCXW82747640" lang="EN-MY" xml:lang="EN-MY"><span class="NormalTextRun SCXW82747640">Scheffer</span></span><span class="TextRun SCXW82747640" lang="EN-MY" xml:lang="EN-MY"><span class="NormalTextRun SCXW82747640"> &amp; </span></span><span class="TextRun SCXW82747640" lang="EN-MY" xml:lang="EN-MY"><span class="SpellingError SCXW82747640">Girdhar</span></span> <span class="TextRun SCXW82747640" lang="EN-MY" xml:lang="EN-MY"><span class="NormalTextRun SCXW82747640">2004</span></span><span class="TextRun SCXW82747640" lang="EN-MY" xml:lang="EN-MY"><span class="NormalTextRun SCXW82747640">), of the Idle Wheel sensor. </span></span><span class="TextRun SCXW82747640" lang="EN-MY" xml:lang="EN-MY"><span class="NormalTextRun SCXW82747640">We generate a series [latex]j[/latex], a pooled time series of velocity for a single sensor j across all machines. Pooling time series data is not a new idea in the time series literature, although scarcely explored (see Maharaj and Inder 1999).  </span></span>

The training set for <code>[latex]v_j[/latex]</code><code></code> is generated as a concatenation of velocity data from machines 2 to 7. Data for machine 1 will be used as our test set. Machine 1 is selected because repairs on the idle wheel were only conducted for machine 1.

<span class="TextRun SCXW192680903" lang="EN-MY" xml:lang="EN-MY"><span class="NormalTextRun SCXW192680903">To use supervised learning methods, w</span></span><span class="TextRun SCXW192680903" lang="EN-MY" xml:lang="EN-MY"><span class="NormalTextRun SCXW192680903">e define an anomaly flag</span></span><span class="TextRun SCXW192680903" lang="EN-MY" xml:lang="EN-MY"><span class="NormalTextRun SCXW192680903">, </span></span><code>[latex]beta_{j,t}[/latex]</code> <span class="TextRun SCXW192680903" lang="EN-MY" xml:lang="EN-MY"><span class="NormalTextRun SCXW192680903">to equal True if an </span></span><span class="TextRun SCXW192680903" lang="EN-MY" xml:lang="EN-MY"><span class="NormalTextRun SCXW192680903">observation lies outside of Tukey’s </span></span><span class="TextRun SCXW192680903" lang="EN-MY" xml:lang="EN-MY"><span class="NormalTextRun SCXW192680903">h</span></span><span class="TextRun SCXW192680903" lang="EN-MY" xml:lang="EN-MY"><span class="NormalTextRun SCXW192680903">inges across <code>[latex]v_j[/latex]</code></span></span><span class="TextRun SCXW192680903" lang="EN-MY" xml:lang="EN-MY"><span class="NormalTextRun SCXW192680903">. The anomaly flag is used to flag abnormal behaviour in the sensors</span></span><span class="TextRun SCXW192680903" lang="EN-MY" xml:lang="EN-MY"><span class="NormalTextRun SCXW192680903">. Predictions from the model would indicate </span></span><span class="TextRun SCXW192680903" lang="EN-MY" xml:lang="EN-MY"><span class="NormalTextRun SCXW192680903">that repairs are necessary.</span></span> <span class="TextRun SCXW192680903" lang="EN-MY" xml:lang="EN-MY"><span class="NormalTextRun SCXW192680903">In addition, we</span></span><span class="TextRun SCXW192680903" lang="EN-MY" xml:lang="EN-MY"><span class="NormalTextRun SCXW192680903"> evaluate the model by </span></span><span class="TextRun SCXW192680903" lang="EN-MY" xml:lang="EN-MY"><span class="NormalTextRun SCXW192680903">back-casting the series</span></span><span class="TextRun SCXW192680903" lang="EN-MY" xml:lang="EN-MY"><span class="NormalTextRun SCXW192680903">. Using </span></span><span class="TextRun SCXW192680903" lang="EN-MY" xml:lang="EN-MY"><span class="NormalTextRun SCXW192680903">the two </span></span><span class="TextRun SCXW192680903" lang="EN-MY" xml:lang="EN-MY"><span class="NormalTextRun SCXW192680903">maintenance events </span></span><span class="TextRun SCXW192680903" lang="EN-MY" xml:lang="EN-MY"><span class="NormalTextRun SCXW192680903">recorded against the idle wheel </span></span><span class="TextRun SCXW192680903" lang="EN-MY" xml:lang="EN-MY"><span class="NormalTextRun SCXW192680903">(Table </span></span><span class="TextRun SCXW192680903" lang="EN-MY" xml:lang="EN-MY"><span class="NormalTextRun SCXW192680903">1</span></span><span class="TextRun SCXW192680903" lang="EN-MY" xml:lang="EN-MY"><span class="NormalTextRun SCXW192680903">)</span></span><span class="TextRun SCXW192680903" lang="EN-MY" xml:lang="EN-MY"><span class="NormalTextRun SCXW192680903">, we can obtain an idea of </span></span><span class="TextRun SCXW192680903" lang="EN-MY" xml:lang="EN-MY"><span class="NormalTextRun SCXW192680903">false positives and false negatives</span></span><span class="TextRun SCXW192680903" lang="EN-MY" xml:lang="EN-MY"><span class="NormalTextRun SCXW192680903"> in the data</span></span><span class="TextRun SCXW192680903" lang="EN-MY" xml:lang="EN-MY"><span class="NormalTextRun SCXW192680903">.</span></span> <span class="TextRun SCXW192680903" lang="EN-MY" xml:lang="EN-MY"><span class="NormalTextRun SCXW192680903">We emphasise again that </span></span><span class="TextRun SCXW192680903" lang="EN-MY" xml:lang="EN-MY"><span class="NormalTextRun SCXW192680903">we cannot obtain the true rate of false positives and false negatives as machine condition remains latent. </span></span><span class="TextRun SCXW192680903" lang="EN-MY" xml:lang="EN-MY"><span class="NormalTextRun SCXW192680903"> </span></span>

Our exploratory analysis suggests that the variation in the series may indicate the healthy functioning of the machine. We may want to use the instantaneous variation of the series; however the acceleration data does not suggest a relationship between shocks and repair events. As a next-best solution, we use the moving variance to approximate the variation in the series at a point in time.

The moving variance for sensor <code>[latex]j[/latex]</code> across 60 periods, which is equivalent to an hour, is defined as
[latex display = "true"]s_{j,t}^2=1/60 sumlimits_{i=t-59}^t(v_{j,i}-bar v_{j,t})^2 [/latex]
where

[latex display = "true"]bar v_{j,t}=1/60 sumlimits_{i=t-59}^t v_{j,i} [/latex]

We fit a classification model to classify [latex]beta_{j,t}[/latex] using [latex]s_{j, t-1}, s_{j, t-2},...,s_{j, t-10}[/latex] as predictors. Given that the predictors are sequence data, we consider the use of recurrent neural network (RNN) models for classifying anomalies. Traditional RNN units are unable to remember long-term dependencies and susceptible to the vanishing gradient problem, and for this purpose LSTM units may be more suitable.

We fit two LSTM models for this purpose, with different train/test set pairs (Figure 18). The model summary is shown in Figure 19.
<table style="border-collapse: collapse;width: 100%;height: 120px" border="1">
<tbody>
<tr style="height: 24px">
<td style="width: 33.3333%;height: 24px"></td>
<td style="width: 33.3333%;height: 24px"><strong>Train</strong></td>
<td style="width: 33.3333%;height: 24px"><strong>Test</strong></td>
</tr>
<tr style="height: 48px">
<td style="width: 33.3333%;height: 48px"><strong>Model 1</strong></td>
<td style="width: 33.3333%;height: 48px">Velocity measure from idle wheel sensors, machines 2 to 7</td>
<td style="width: 33.3333%;height: 48px">Velocity measure from idle wheel sensors, machine 1</td>
</tr>
<tr style="height: 48px">
<td style="width: 33.3333%;height: 48px"><strong>Model 2</strong></td>
<td style="width: 33.3333%;height: 48px">Velocity measure from idle wheel sensors, machines 1 to 7</td>
<td style="width: 33.3333%;height: 48px">Velocity measure from lifting motor and drive wheel, machines 1 to 7</td>
</tr>
</tbody>
</table>
<p style="text-align: center"><em>Figure 18: Train and test set for Models 1 and 2</em></p>

<img class="alignnone size-full wp-image-7824" src="https://www.datasciencesociety.net/wp-content/uploads/2018/09/Capture-7.png" alt="" width="722" height="350" />
<p style="text-align: center"><em>Figure 19: Model summary</em></p>

### ii. Evaluation
#### Model 1
By visually inspecting Figure 20, observations 300-1300 should be classified as anomalies because the spread of the series is abnormally high. Figure x shows the anomaly flag for Machine 1's velocity on the idle wheel sensor as classified by Tukey's method. The anomaly flag correctly highlights these observations as anomalous. The drop-off point at approximately observation 1700 is likely due to a maintenance event. This event is also flagged as anomalous according to Tukey's Method. Model 1's predictions also closely follow the anomaly labels.

<img class="size-full wp-image-7753 aligncenter" src="https://www.datasciencesociety.net/wp-content/uploads/2018/09/Capture-5.png" alt="" width="550" height="442" />
<p style="text-align: center"><em>Figure 20: Velocity series of idle wheel for machine 1</em></p>

#### Model 2
It is reasonable to assume that the distribution of the normal condition of the sensors is similar across sensors as well as machines. To provide some evidence for our hypothesis, a LSTM model is fitted on the idle wheel data across all machines and tested against lifting motor and drive wheel data across all machines. The results of the predictions on the drive wheel sensors are illustrated in Figure 21.

<img class="wp-image-7762 aligncenter" src="https://www.datasciencesociety.net/wp-content/uploads/2018/09/download-3.png" alt="" width="674" height="674" />
<p style="text-align: center"><em>Figure 21: Drive wheel velocity measure for machines 1-7</em></p>
Again, the predicted data closely trails the actual labels in all drive wheel sensors except for Machine 5, where a false negative was detected. However, the series does not appear to have an abnormal distribution in that region. This suggests that the LSTM approach is able to generalise anomalous patterns better than a naive approach using Tukey's method to flag anomalies.

As for the lifting motor (Figure 22), the model detects anomalies in Machine 1, 2, 6 and 7. Both the anomaly labels and LSTM predictions detect anomalies similarly. The abnormalities in Machine 1's idle wheel are reflected in the predictions and anomaly labels for the drive wheel and lifting motor. This suggests that this modelling approach may find it hard to distinguish between different sections of the machine when predicting faults.

<img class="wp-image-7767 aligncenter" src="https://www.datasciencesociety.net/wp-content/uploads/2018/09/download-4.png" alt="" width="643" height="643" />
<p style="text-align: center"><em>Figure 22: Velocity measures for lifting motor for machines 1-7</em></p>
Assuming the anomaly labels to be the 'ground truth', we can calculate false positive and false negative rates. False negatives are lower than false positives in all sensor-machine combinations evaluated except for Machine 5's drive wheel sensor. Even though the LSTM model was not optimsed to minimise false negatives, Model 2 has a low share of false negatives.

&nbsp;
### C. E-Divisive with Medians (EDM)
E-Divisive with Medians (EDM) is a model for breakpoint (changepoint) detection. Breakpoint detection describes the process of detecting change points in time series. We decided to analyse the breakpoints with an unsupervised approach as we noticed that there are visually obvious changes of distribution resulting in distinct segments in the sensor’s time series data, due to different possible reasons (could be anomalies, repairs or maintenances, in some cases it’s missing data). The EDM algorithm (James et al. 2014) is reported to have several advantages over traditional breakout detection techniques. They include (1) robustness against anomalies, (2) able to detect divergence and change in distributions, and (3) faster computation process. In Figure 23, we segment out series into 3 sections by visually examining the data. We expect the EDM algorithm to automate this process.

<img class="alignnone wp-image-7834" src="https://www.datasciencesociety.net/wp-content/uploads/2018/09/Screen_Shot_2018-09-30_at_6.03.37_PM1.png" alt="" width="559" height="361" />
<p style="text-align: center"><em>Figure 23: Visual inspection of series</em></p>

#### Outcome
We tried to detect the breakouts using different metrics, namely mean (Figure 24), distributions (Figure 25), and median (Figure 26). With no labeled data available, we visually evaluated the graphs and observed that median-based method draws a cleaner boundary between different segments in the time series. We concluded that median is the best option to detect breakouts in our case as it is robust against outliers. One downside of this approach is that computing the median is computationally expensive, with a computational cost of <b>[latex]</b>O(n!)[/latex]. Due to time constraint, we did not generate breakout detection reports for all the sensors.

<img class="wp-image-7831 aligncenter" src="https://www.datasciencesociety.net/wp-content/uploads/2018/09/Screen_Shot_2018-09-30_at_5.55.28_PM1-1.png" alt="" width="459" height="457" />
<p style="text-align: center"><em>Figure 24: Mean-based EDM</em></p>
&nbsp;

<img class="wp-image-7832 aligncenter" src="https://www.datasciencesociety.net/wp-content/uploads/2018/09/Screen_Shot_2018-09-30_at_5.53.50_PM1-1.png" alt="" width="530" height="531" />
<p style="text-align: center"><em>Figure 25: Distribution-based method</em></p>
<img class="wp-image-7838 aligncenter" src="https://www.datasciencesociety.net/wp-content/uploads/2018/09/Screen_Shot_2018-09-30_at_6.13.17_PM1.png" alt="" width="544" height="539" />
<p style="text-align: center"><em>Figure 26: Median-based EDM</em></p>
Breakpoint detection can indicate gradual shifts in the time series that may not be visible to the human eye. As a complement to actual anomaly detection, it can be deployed to provide alerts before an abrupt anomalous event occurs. This in theory can be used as an indicator for scheduling preventative maintenance.

&nbsp;
## <strong>6. Deployment</strong>
### A. AR-LSTM
Figure 27 shows the deployment of the AR-LSTM model. Firstly, 11 minutes of data is collected. 10 minutes of data would be sent to LSTM model to make a 'normal' prediction, which is the 11th minute. The prediction is then compared to the data collected from the sensor, at the 11th minute since the start.

The difference between the prediction and the real data is calculated. If the difference is greater than the threshold, anomaly is detected since the data deviates from the predicted 'normal' value. Otherwise it means there is no anomaly, the machine is operating as normal. The loop continues.
<img class=" wp-image-7585 aligncenter" src="https://www.datasciencesociety.net/wp-content/uploads/2018/09/deploy-1.png" alt="" width="776" height="742" />
<p style="text-align: center"><em>Figure 27: AR-LSTM deployment plan
</em></p>

### B. Moving Variance
Figure 28 shows the deployment of the moving variance model. Data is collected for 70 minutes. Data is transformed by computing moving variances for 60 minute windows. Then, predictions are made based on the LSTM model. Based on the prediction, the model suggests whether preventive maintenance is required.
<p style="text-align: center"><em> </em></p>
<img class="alignnone size-full wp-image-7795" src="https://www.datasciencesociety.net/wp-content/uploads/2018/09/Capture-6.png" alt="" width="1059" height="485" />

&nbsp;
<p style="text-align: center"><em>Figure 28: Moving variances-LSTM deployment plan</em></p>

## <strong>Conclusion </strong>
To sum up our analysis, we have achieved two main actionable outcomes (1) a classification model for outlier detection, and (2) an implementation of breakout detection. With that, we suggest that Kaufland can (1) implement the breakout detection method to alert operators that the machines need to be inspected, (2) use the trained classification model to improve performance of outlier and breakout detection, and finally, (3) do outlier detection in conjunction with detailed repair logs to correlate repair parts with anomalous patterns. In addition, we also recommend several steps that can be taken to improve the overall analyses and modeling: collection of temperature data, machine utilization data, machine movement data and maintaining updated detailed repair logs.
## References
<ol>
 	<li><span lang="EN-US">James, N., Kejariwal A.m, and Matteson, D. (2014). <i>Leveraging Cloud Data to Mitigate User Experience from ‘Breaking Bad’.</i></span>Available at: <a title="https://arxiv.org/pdf/1411.7955.pdf" href="https://arxiv.org/pdf/1411.7955.pdf" target="_blank" rel="noreferrer noopener">https://arxiv.org/pdf/1411.7955.pdf</a></li>
 	<li><span class="st"><em>Girdhar</em>,Paresh dan C. <em>Scheffer</em>., <em>2004</em>, Practical Machinery Vibration Analysis and Predictive Maintenance., Elsevier, Burlington. Gates corp., 2015.</span></li>
 	<li>
<p class="title">Forecasting time series from clusters. Maharaj, E. A. &amp; Inder, B. A. <span class="date">1999</span> Melbourne Vic Australia: Monash University Publishing. <span class="numberofpages">27 p.</span></p>
</li>
</ol>
## Code
The code for this submission is available <a href="https://www.datasciencesociety.net/wp-content/uploads/2018/09/notebooks.zip">here</a>.

{% gist cb5bf55414a4fdf1dbacd4f4d4a0b500 %}