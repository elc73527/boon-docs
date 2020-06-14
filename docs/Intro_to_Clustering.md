![Logo](../images/BoonLogic.png)   
# Introduction to Clustering with the Boon Nano

The Boon Nano is a high-speed, high-efficiency, clustering and segmentation algorithm based on unsupervised machine learning. The Nano builds clusters of similar *n*-space vectors (or **patterns**) in real-time based on their similarity. Each pattern has a sequence of **features** that the Nano uses in its measurement of similarity. 

<img src="../images/Segmentation.png" width="800">

## Examples of Patterns

* **Motor Status Tags:** Industrial motors are controlled by servo feedback loops that have multiple features. A pattern that could be used to segment motor performand date might be  
(Output Current, Torque Trim, Position, Velocity, Position Error, Velocity Error)
* **Flight Recorder Data:** An aircraft has multiple sensors that define its operating state at any point in time. For example, a pattern might have these features<br/>
(Altitude, Air Speed, Ground Speed, Lateral Acceleration, Longitudinal Acceleration, Vertical Acceleration, Left Aileron Position, Right Aileron Position)   
* **Power Spectra:** The output of vibrational sensors is often transformed into frequency spectra. In this case, the features represent adjacent frequency bands and the value of each feature is power from the source signal in that frequency band.
* **Triggered Impulsive Data:** If *n* consecutive samples from a single sensor are acquired like a snap shot, they can be compared by shape and clustered by similarity to gain insight about the varieties of signals occurring in the data stream. The snap shot is typically triggered by a threshold crossing of the signal.
* **Single Sensor Streaming Data:**  Consecutive overlapping windows of the most recent *n* samples from a sensor form a natural collection of *n*-dimensional vectors.

## Using the Boon Nano
The Boon Nano is deployed in both a general use plaform called **Expert** and as a streaming sensor analytics application called **Amber**
* **Expert:** The Expert console provides the full functionality of the Boon Nano including all analytics (described below) and is oriented toward batch processing of input data.
* **Amber:** By specializing the Expert console for real-time streaming data, Amber provides an easy-to-use interface for sending streams of successive sensor values as a time series and getting back the same number of analytic values. These analytic values   


## Configuring the Boon Nano

### Batch Mode vs. Streaming Mode
The Boon Nano can be operated in batch mode or streaming mode depending on the application.  
* **Batch Mode:** In batch mode, successive patterns clustered by the Nano are not assumed to have any temporal relationship. For instance, segmentation of tissue types from a DICOM CT scan uses batch mode. Another example is anomaly detection for doing quality control in camera images of standard units on a manufacturing line.
* **Streaming Mode:** In streaming mode, it assumed that successive patterns are temporally related and often they overlap. Examples of streaming mode data would be overlapping, successive windows of single sensor data or successive sensor fusion patterns coming off multiple related sensors in real-time. 

### Clustering Configuration
The clustering parameters of the Nano determine the properties the clusters of the input data will have. 
* **Numeric Type:** The three types available are floating-point (float32), signed integer (int16), unsigned integer (uint16). For nearly all applications, float32 is a good general numeric type to use. However, some applications naturally lend to integer types (for instance, bin counts in a histogram).
* **Feature List:** A pattern is comprised of **features** which represent the properties of each column in the vectors to be clustered. Each feature has a **range** (a min and max value) that represents the expected range of values that feature will take on in the input data. This range need not represent the true range. For example, if the range is set to -5 to 5, then values greater than 5 will be treated as if they were 5 and value less than -5 will be treated as -5. This truncation (or outlier filtering) can be useful in some data sets. In addition, each feature is assigned a **weight** between 0 and 1000 which determines its relative importance in placing of input vectors into clusters. Setting all weights to 1 is a common setting. Setting a weight to 0 means that feature is ignored in clustering as if it were not in the vector at all. Finally, each feature can be assigned an optional **label** (for example, "systolic", "pulse rate", "payload length", "output current"). Labels have no effect on the clustering.  
* **Percent Variation:** The percent variation ranges from 1% to 20% and indicates the amount of similarity to the Nano will require between patterns within the same cluster. Smaller percent variations creates more finely grained clusters and many clusters. A larger choice for percent variation produces coarser clustering with fewer clusters.
* **Streaming Window Size:** This indicates the number of successive samples to use in creating overlapping patterns. For instance, it is typical in single-sensor streaming mode applications to have only feature and a streaming window size greater than 1. If there is one feature, and the streaming window size is 25, then each pattern clustered by the Nano is comprised of the most recent 25 values from the sensor. In this case, each pattern overlaps 24 samples with its predecessor. In batch mode, it is typical to use a streaming window size of 1.

### Autotuning Configuration
Two clustering parameters, the percent variation and the range for each feature, can be *autotuned*, that is, chosen automatically, by the Boon Nano prescanning representative data. The range for each feature can be chosen either individually or a single range can be chosen to apply to all features. The percent variation is chosen automatically using an "elbow" technique by preclustering the input data and finding the percent variation that balances cluster cohesion and separation. 

* **Autotune Range:** If this is true, the user-supplied range(s) specified in the Nano configuration gets replaced with the autotuned range.   If false, no the user-supplied range(s) is left intact.
* **Autotune Percent Variation:** If this is true, the user-supplied percent variation in the Nano configuration get replaced with the percent variation found through the autotuning. If false, the user-supplied percent variation is left intact.
* **Autotune By Feature:** If this is true and the option to autotune the range is true, then the autotuning will find a range customized to each feature. If false, then autotuning will find a single range that applies to all features.
* **Exclusions:** An array of exclusions may be provided which causes the autotuning to ignore all of those features. For instance, if the array [2, 7] is provided, then autotuning is applied to all features except the 2nd and 7th features.

### Streaming Configuration (Amber Only)
Streaming parameters apply to the Amber deployment of the Boon Nano. If streaming mode is selected in the configuration, then a variety of parameters determine the analytic values returned back for the sensor samples sent into Amber. In its default configuration, Amber returns one *analytic* sample for each *sensor* value sent into it. The most typical analytic value is the smoothed anomaly index (SI) described below, but other outputs can be selected. The streaming window size selected in the clustering configuration (described above) is a key parameter as it determines the amount of sensor value "history" that will be included in each pattern to be clustered.

<table class="table">
  <tr>
    <td><img src="../images/Figure2.png" width="800"></td>
  </tr>
  <tr>
    <td><em>One feature (all samples from the same sensor) and streaming window size of 250. Each input vector is 250 successive samples where we form successive patterns by dropping the oldest sample from the current pattern and appending the next sample from the input stream.</em></td>
  </tr>
</table>

As sensor values are streamed into it, Amber transitions automatically through four phases: starting, autotuning, learning, and monitoring.

<img src="../images/amber_training.png" width="800">

* **Starting:** As the first sensor samples are sent to Amber, there is not sufficient data to return meaningful analytic results. Return a 0 value as the analytic for each sensor value sent. Eventually, sufficient (e.g. 1000) samples have been acquired to move to the autotuning phase.
* **Autotuning:** Using the autotuning configuration (described above) and the sensor samples buffered thus far, tune the Boon Nano hyperparameters (feature ranges and percent variation) to optimal values for this sensor. During autotuning, zeros are returned for as the anomaly index for sensor values sent in.
* **Learning:** Configure the Boon Nano with the parameters found during autotuning and then start training the Nano with the samples that have been buffered thus far. In this phase, the Boon Nano is adding clusters as needed to build a high-dimensional analytic model that describes the sensor input and that conforms to the clustering configuration (described above). There is sufficient data in this phase to return real analytic output for each sensor sample sent in. Part of the streaming configuration is to determine when learning stops, called **graduation requirements**. Graduation occurs automatically based on a number of parameters (described below). In principle, learning can continue indefinitely, but it is usually desirable to automatically turn off learning upon graduation so that no new clusters are created. This solidifies the model so that the future analytics results retain their same meaning over many months of operation.
* **Monitoring:** The final phase of Amber is the monitoring phase where learning has been turned off. Each sensor value sent in becomes the final sample in the streaming window of prior samples. That window is clustered by the Boon Nano in real-time and the result returned as the Amber analytic value for that sample.

![Figure 3](../images/Figure1.png)   
*Figure 1: Calculation of percent variation between two patterns*


### Numeric Type
You must choose one numeric type to apply to all of the features in each input vector. A good default choice for numeric type is float32, which represents 32-bit, IEEE-standard, floating-point values. In some cases, one may know that all features in the input data are always integers ranging from -32,768 and 32,767 (int16) or non-negative integers ranging from 0 to 65,535 (uint16). Using these integer types may may give slightly faster inference times, however, float32 performance is usually similar to the performance for the integer types.

### Pattern Length
The pattern length of each vector to be clustered is the number of features that it has. For example, if each input vector is an FFT across 256 frequency bands then the pattern length would be 256. The inference time needed by the Nano to assign a cluster ID to an input vector is the same regardless of the dimension of the vector. This is a unique property of the Boon Nano relative other unsupervised clustering algorithms.

### Range (Minimum and Maximum)
For each feature in a pattern, the value to choose for each Minimum Value and Maximum Value is fairly straightforward. Generally, you choose a Min and Max that encompass the range of interest across that feature. For example, if one feature in the pattern is from a transducer with values ranging between +/- 10 volts, then one might choose a minimum of -10 and maximum of 10 for that feature.

### Percent Variation
In Figure 1, the percent variation between the two vectors is 6.5%, which is the ratio of the area between the vectors to the total area = Pattern Length (60) x Range (2.0 - 0.0). If the Nano is configured for a Percent Variation of 6% or less then the two patterns in Figure 1 would be assigned to different clusters. If the Nano Percent Variation is set to 7% or more then these two patterns would be assigned to the same cluster. Selecting smaller values for Percent Variation creates "tighter" clusters and consequently more clusters. Larger values for Percent Variation will segment the input data into fewer, coarser clusters.

### Weight
In some situation, one may want one feature in a pattern to be weighted more heavily than other features in determining the cluster to which it is assigned. For example, assigning a weight of 1 to all features gives them equal weight in the clustering. Changing one of those weights to 4 gives that feature four times the weight in the clustering assignment in comparison to features weighted as 1. Assigning a weight of 0 to a feature means it has no effect on the clustering. 

### Streaming Window Size
In streaming window mode, the Nano works by using successive, overlapping subsignals as a stream of incoming samples. Samples in this case are typically time series data. The Pattern Length is specified by the user as a "detection window size" and the minimum Value and Maximum Value for the clustering are also chosen (Figure 2) to represent the range of interest in the signal. The streaming window size is the number of input samples that the window is moved to the right (in time) in forming the next input signal.

![Figure 2](../images/Figure2.png)   
*Figure 2: One feature (all samples from the same sensor) and streaming window size of 250. Each input vector is 250 successive samples where we form successive patterns by dropping the oldest sample from current pattern and appending the next sample from the input stream.*

### Accuracy
In some rare cases, you may choose to reduce the clustering accuracy in order to reduce inference time. This may produce rare suboptimal assignments of input vectors to clusters, although the results will generally still be good. For example, streaming IoT applications where "lossy" clustering results are acceptable may benefit from adjusting this parameter. Accuracy is a value ranging from 0.75 to 0.99. The default value of 0.99 produces the highest accuracy results while still ensuring very good performance. 

## Autotuning

### Autotuning Range
Once the Pattern Length and numeric type of a block of data are selected, the Boon Nano has the ability to automatically select appropriate minimum and maximum values for each range by sampling the patterns found in that block of data. When autotuning range you must select one of two options
* Autotune by feature: In this case, sampling is done separately for each feature. This may be important, for example, if values in the input vector come from different types of sensor measurements.
* Autotune overall: In this case sampling is done uniformly across all samples in the input data. This produces one minimum value and one maximum value that will be used for all features.
Autotuning basically works through outlier rejection. The minimum is the 0.5th percentile and the maximum is the 99.5th percentile of the sampled data. If those percentiles are the same, then the actual sampled minimum and maximum values are used.

### Autotuning Percent Variation
One of the most difficult parameters to configure in unsupervised machine learning is the desired number of clusters needed to produce the best results (as with K-means) or (in the case of the Boon Nano) the desired Percent Variation to use. This is because one would not generally know *a priori* the underlying proximity structure of the input vectors to be segmented.

To address this, once the Pattern Length and Numeric Type of a block of data are selected, the Boon Nano can automatically tune its Percent Variation to create an balanced combination of **coherence within clusters** and **separation between clusters**. In nearly all cases, autotuning produces the best value for the Percent Variation parameter. However, if more granularity is desired you can lower the Percent Variation manually after autotuning. Similarly, if the autotuned Percent Variation is creating too much granularity (and too many clusters) then you can choose to manually increase the Percent Variation above the autotuned value. 
 
## Clustering Results

### Cluster ID (ID)
The Boon Nano assigns a **Cluster ID** to each input vector as they are processed. The first vector is always assigned to a new cluster ID of 1. The next vector, if it is within the defined Percent Variation of cluster 1, is also assigned to cluster 1. Otherwise it is assigned to a new cluster 2. Continuing this way all vectors are assigned cluster IDs in such a way that each vector in each cluster is within the desired Percent Variation of that cluster's centroid. In some circumstances the cluster ID 0 may be assigned to a vector. This happens if learning has been turned off (an option within the Nano) or if the maximum cluster count has been reached. It should be noted that cluster IDs are assigned serially so having similar cluster IDs (for instance, 17 and 18) says nothing about the similarity of those clusters. However, PCA (see [Guide: Nano Status](../Guides/Guide_Nano_Status.md)) can be used to measure relative proximity of clusters to each other.

### Raw Anomaly Index (RI)
The Boon Nano assigns to each pattern a **Raw Anomaly Index**, that indicates how many patterns are in its cluster relative to other clusters. These integer values range from 0 to 1000 where values close to zero signify patterns that are the most common and happen very frequently. Values close to 1000 are very infrequent and are considered more anomalous the closer the values get to 1000.

When learning is turned off, patterns with cluster IDs of 0 have a raw anomaly index of 1000.

### Smoothed Anomaly Index (SI)
Building on the raw anomaly index, we create a **Smoothed Anomaly Index** which is an edge-preserving, exponential, smoothing filter applied to the raw anomaly indexes of successive input patterns. These values are also integer values ranging from 0 to 1000 with similar meanings as the raw anomaly index. In cases where successive input patterns do not indicate any temporal or local proximity, this smoothing may not be meaningful.

### Frequency Index (FI)
Similar to the anomaly indexes, the **Frequency Index** measures the relative number of patterns placed in each cluster. The frequency index measures all cluster sizes relative to the average size cluster. Values equal to 1000 occur about equally often, neither abnormally frequent or infrequent. Values close to 0 are abnormally infrequent, and values  significantly above 1000 are abnormally frequent.


### Distance Index (DI)
The **Distance Index** measures the distance of each cluster centroid to the centroid of all of the cluster centroids. This overall centroid is used as the reference point for this measurement. The values range from 0 to 1000 indicating that distance with indexes close to 1000 as indicating patterns furthest from the center and values close to 0 are very close. Patterns in a space that are similar distances apart have values that are close to the average distance between all clusters to the centroid. 
 
## Example
We now present a very simple example to illustrate some of these ideas. A set of 48 patterns is shown in Figure 3. A quick look across these indicates that there are at least two different clusters here. Each pattern has 16 features so we configure the Nano for
* Numeric Type of float32
* Pattern Length of 16

![Figure 3](../images/Figure3.png)   
*Figure 3: 48 patterns to be clustered.*


We could select the mininum and maximum by visual inspection, but it is not possible to determine the correct Percent Variation this way. So we instead load the patterns into the Nano and tell the Nano to Autotune those parameters. The results comes back with 
* Min = -4.39421
* Max = 4.34271
* Percent Variation = 0.073

We configure the Nano with these parameters and then run the patterns through the Nano, requesting as a result the "ID" assignd to each input pattern. We receive back the following list: "ID" -> {1, 1, 2, 1, 1, 2, 1, 2, 1, 1, 1, 1, 2, 2, 2, 1, 1, 1, 1, 1, 1, 1, 1, 3, 1, 1, 2, 2, 2, 1, 2, 1, 1, 2, 1, 1, 1, 1, 1, 1, 3, 3, 2, 2, 2, 2, 1, 1}  
 
 Comparing this to the sequence in Figure 3, we see that this is a reasonable clustering assignment. Further, we see that there is a third cluster that may have been missed by our intuitive clustring. This cluster had just three patterns assigned to it. Figure 4 shows the waveforms from Figure 3 plotted on the same axes and colored according according to their assigned cluster IDs. 
 
![Figure 4](../images/Figure4.png)   
*Figure 4: 48 patterns colored according to their assigned clusters*

The Raw Anomaly Index for each of the three clusters are as follows:
* Cluster 1 Raw Anomaly Index: 0
* Cluster 2 Raw Anomaly Index: 170
* Cluster 3 Raw Anomaly Index: 563

This indicates Cluster 1 had the most patterns assigneed to it. Cluster 2 was also common, and Cluster 3 was significantly less common. It is worth noting that a Raw Anomaly Index of 563 would not be signficant in practice to indicate an anomaly in the machine learning model. Typically, useful anomaly indexes must be in the range of 900 to 1000 to indicate a pattern that is far outside the norm of what has been learned.

**Important Simplifications:** This is an artificially small and simple example to illustrate the meaning of some of the basic principles of using the Boon Nano. In particular, 
* The Boon Nano would typically be used to cluster 100s of millions or billions of patterns.
* The number of clusters created from "real" data typically runs into the hundreds or even thousands of clusters. 
* The speed of the Boon Nano for such a small data set is not noticeable over other clustering techniques such as K-means. However, when the input set contains 100s of millions of input vectors or when the clustering engine must run at sustained rates of 100s of thousands of inferences per second (as with video or streaming sensor data), the Boon Nano's microsecond inference speed makes it the only feasible technology for these kinds of solutions.
