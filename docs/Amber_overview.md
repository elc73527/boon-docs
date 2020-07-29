![Logo](../images/BoonLogic.png)   

# Boon Amber Overview

Boon Amber is a real-time, self-configuring anomaly detection system based on unsupervised machine learning. Amber uses the Boon Nano clustering algorithm to build a rich, high-dimensional model of any data stream which is then employed to detect anomalies as departures from the learned data distribution. Amber assigns an **anomaly index** to *n*-space input vectors (or **patterns**) in real-time based on the extent to which any given pattern resembles, or departs from, inputs seen in the past. Each pattern is a sequence of **features** that encode numerical attributes of the data stream being monitored.

## Examples of Data Streams

* **Single sensor streaming data:**  Consecutive overlapping windows of the most recent *n* samples from a sensor timeseries form a natural collection of *n*-dimensional vectors.

* **Power spectra:** The output of vibrational sensors is often transformed into frequency spectra. In this case, the features represent adjacent frequency bands and the value of each feature is the power from the source signal in that frequency band.

* **Triggered impulsive data:** If *n* consecutive samples from a single sensor are acquired in a single snapshot, they can be compared by waveform shape and clustered by similarity to gain insight about the variety of signals occurring in the data stream. The snapshot is typically triggered by a threshold crossing of the signal.

* **Gateway-level sensor fusion:** An IoT gateway might aggregate single-sensor streams from numerous sensors on the same physical asset. Amber may be used to detect anomalies not only in any of these streams individually, but in the joint pattern vector created by concatenating the readings across sensors at any point in time. This approach detects anomalies in the joint behavior of interacting sensors which may not be detectable given any of the sensor streams in isolation.

## Using the Boon Amber

The Boon Amber detects anomalies in input data by assigning each pattern a scalar **anomaly index** representing the extent to which that pattern is an outlier with respect to data seen previously.

Anomaly index values range from 0 to 1000, with values closer to 0 representing input patterns which are ordinary given the data seen so far on this sensor. Values closer to 1000 represent novel patterns which are anomalous with respect to data seen before. The anomaly index is derived from a clustering of the data stream produced by the Boon Nano, Boon Logic's proprietary clustering algorithm. Patterns assigned to "large" clusters, or clusters to which other patterns are frequently assigned, are considered less anomalous, while patterns assigned to "small" clusters represent anomalies as these patterns occupy infrequently explored regions of data space. A pattern which creates an entirely new cluster is maximally anomalous and receives an anomaly index of 1000. For more detail on the Boon Nano clustering algorithm see [Clustering with the Boon Nano](../docs/Intro_to_Clustering.md).

### Configuration: data dimensions

The Boon Amber takes a number of different configuration parameters. The first two, **featureCount** and **streamingWindowSize**, determine the "height" (number of rows) and "width" (number of features) of the data to be ingested by the model.

* **featureCount** A pattern is comprised of **features** which represent the properties of each column in the vectors to be processed. This feature count parameter should be set to the number of features in input patterns.

* **streamingWindowSize:** This indicates the number of successive samples to use in creating overlapping patterns. For instance, it is typical in single-sensor streaming mode applications to have only one feature and a streaming window size greater than 1. If there is one feature, and the streaming window size is 25, then each pattern clustered by the Nano is comprised of the most recent 25 values from the sensor. In this case, each pattern overlaps 24 samples with its predecessor and each newly arriving sample slides the moving window over by one.

### Configuration: autotuning

The Boon Amber makes use of the Boon Nano algorithm which requires two clustering hyperparameters: the percent variation between clusters, and a min/max range for each input feature. The first hyperparameter, percent variation, determines the *n*-space distance threshold beyond which two samples are considered as belonging in separate clusters. All else held equal, a higher percent variation setting segments data into fewer clusters which are more spread out, while a lower percent variation segments data into a greater number of clusters closer together.

One of the most difficult parameters to configure in unsupervised machine learning is the desired number of clusters needed to produce the best results (as with K-means) or (in the case of the Boon Nano) the desired percent variation to use. This is because one would not generally know *a priori* the underlying proximity structure of the input vectors to be segmented.

To address this, the Boon Nano can automatically tune its percent variation to create a balanced combination of **coherence within clusters** and **separation between clusters**. This process is called **autotuning** and occurs automatically when a sufficient amount of initial data has been collected. In nearly all cases, autotuning produces the best value for the percent variation setting. In addition to setting the percent variation, autotuning sets the min/max parameters for each feature as well.

### Configuration: streaming

In its default configuration, Amber returns one *analytic* value for each *sensor* value sent into it. The typical analytic value is the smoothed anomaly index (SI) (described below), but other outputs can be included as well. The streaming window size selected in the clustering configuration (described above) is a key parameter as it determines the amount of sensor value "history" that will be included in each pattern to be clustered.

As sensor values are streamed into it, Amber transitions automatically through four phases: buffering, autotuning, learning, and monitoring.

* **Buffering:** As the first sensor samples are sent to Amber, there is not sufficient data to return meaningful analytic results. A 0 is returned as the anomaly index for each sensor value sent. Eventually, sufficient samples will have been acquired to move to the autotuning phase (e.g. 1000 samples may be sufficient).
* **Autotuning:** Using the autotuning configuration (described above) and the sensor samples buffered thus far, the Boon Nano hyperparameters (percent variation and feature min/maxes) are tuned to optimal values for this sensor. During autotuning, Amber continues to return zeros as the anomaly index corresponding to the incoming sensor values.
* **Learning:** Amber configures the Boon Nano with the parameters found during autotuning and then starts training the Nano with the samples that have been buffered thus far. In this phase, the Boon Nano is adding clusters as needed to build a high-dimensional analytic model that describes the sensor input and that conforms to the clustering configuration (described above). There is sufficient data in this phase to return real analytic outputs for each sensor sample sent in. Part of the streaming configuration determines when learning stops, called **graduation**. Graduation occurs automatically based on a number of graduation requirements (described below). In principle, learning can continue indefinitely, but it is usually desirable to automatically turn off learning upon graduation so that no new clusters are created. This solidifies the model so that the future analytics results retain their same meaning over many months of operation.
* **Monitoring:** The final phase of Amber is the monitoring phase where learning has been turned off. Each sensor value sent in becomes the final sample in the streaming window of prior samples. That window is clustered by the Boon Nano in real-time and the result returned as the Amber analytic value for that sample.

The streaming configuration is defined by the following parameters:

* **samplesToBuffer:** Incoming sensor samples will be buffered until samplesToBuffer have been collected at which time autotuning begins on the buffered sensor data. Autotuning will typically require approximately 200 more sensor values to complete. Once autotuning is complete, then Amber uses the autotuned ranges and percent variation to configure the Nano.
* **learningMaxClusters (graduation requirement):** If during the learning phase, the Nano has a total of learningMaxClusters clusters in its model, then learning is automatically turned off.
* **learningMaxSamples (graduation requirement):** If during the learning phase, the Nano has processed a total of learningMaxSamples sensor samples, then learning is automatically turned off.
* **learningRateNumerator** and **learningRateDenominator (graduation requirements):** As learning progresses, clusters are added. As the model matures, the growth in the cluster count slows as the Boon Nano has created clusters that account for nearly all of the variation in the input data. The ratio learningRateNumerator divided by learningRateDenominator determines a cluster growth "flatness" threshold. If during the most recent learningRateDenominator inferences, there have been fewer than learningRateNumerator new clusters created, then learning is automatically turned off.

## Streaming results

When a single pattern is assigned an anomaly index, this is called an *inference*. Besides the anomaly index, a number of other useful analytic outputs are generated. 

* **Cluster ID (ID)**: The Boon Nano assigns a **cluster ID** to each input vector as they are processed. The first vector is always assigned to a new cluster ID of 1. The next vector, if it is within the defined percent variation of cluster 1, is also assigned to cluster 1. Otherwise it is assigned to a new cluster 2. Continuing this way all vectors are assigned cluster IDs in such a way that each vector in each cluster is within the desired percent variation of that cluster's template. In some circumstances the cluster ID 0 may be assigned to a pattern. This happens, for example, if learning has been turned off or if the maximum cluster count has been reached. It should be noted that cluster IDs are assigned serially so having similar cluster IDs (for instance, 17 and 18) says nothing about the similarity of those clusters. However, PCA can be used to measure relative proximity of clusters to each other.

* **Raw anomaly index (RI)**: The Boon Nano assigns to each pattern a **raw anomaly index**, that indicates how many patterns are in its cluster relative to other clusters. These integer values range from 0 to 1000 where values close to zero signify patterns that are the most common and happen very frequently. Values close to 1000 are very infrequent and are considered more anomalous the closer the values get to 1000. Patterns with cluster ID of 0 have a raw anomaly index of 1000.

* **Smoothed anomaly index (SI):** Building on the raw anomaly index, we create a **smoothed anomaly index** which is an edge-preserving, exponential, smoothing filter applied to the raw anomaly indexes of successive input patterns. These values are also integer values ranging from 0 to 1000 with similar meanings as the raw anomaly index. In cases where successive input patterns do not indicate any temporal or local proximity, this smoothing may not be meaningful.

* **Frequency index (FI):** Similar to the anomaly indexes, the **frequency index** measures the relative number of patterns placed in each cluster. The frequency index measures all cluster sizes relative to the average size cluster. Values equal to 1000 occur about equally often, neither abnormally frequent or infrequent. Values close to 0 are abnormally infrequent, and values significantly above 1000 are abnormally frequent.

* **Distance index (DI):** The **distance index** measures the distance of each cluster template to the centroid of all of the cluster templates. This overall centroid is used as the reference point for this measurement. The values range from 0 to 1000 indicating that distance with indexes close to 1000 as indicating patterns furthest from the center and values close to 0 are very close. Patterns in a space that are similar distances apart have values that are close to the average distance between all clusters to the centroid.

* **Anomaly detections (AD):** An array of 0's and 1's as **anomaly detection** values. These correspond one-to-one with input samples and are produced by thresholding the smoothed anomaly index (SI). The threshold is determined automatically from the SI values. A value of 0 indicates that the SI has not exceeded the anomaly detection threshold. A value of 1 indicates it has, signaling an anomaly at the corresponding input sample.

* **Anomaly history (AH):** An array of **anomaly history** values. These values are a moving-window sum of the AD value, giving the number of anomaly detections (1's) present in the AD signal over a "recent history" window whose length is the buffer size.

* **Amber metric (AM):** An array of **Amber metric** values. These are floating-point values between 0.0 and 1.0 indicating the extent to which each corresponding AH value shows an unusually high number of anomalies in recent history. The values are derived statistically from a Poisson model, with values close to 0.0 signaling a lower, and values close to 1.0 signaling a higher, frequency of anomalies than usual.

* **Amber warning level (AW):** An array of **Amber warning level** values. This index is produced by thresholding the Amber metric (AM) and takes on the values 0, 1 or 2 representing a discrete "warning level" for an asset based on the frequency of anomalies within recent history. 0 = normal, 1 = asset changing , 2 = asset critical. The default thresholds for the two warning levels are the standard statistical values of 0.95 (outlier, asset changing) and 0.997 (extreme outlier, asset critical).

## Amber status
Whereas the streaming results (above) return analytics for the most recently ingested pattern, **Amber status** provides core analytics about the internal machine learning model that has been constructed since configuration. These results are indexed by cluster ID beginning with cluster 0.

* **clusterSizes:** In learning mode, the values in this list give the number of patterns that have been assigned to each cluster beginning with cluster 0. When learning is turned off, this value does not change.

* **anomalyIndexes:** The values in this list give raw anomaly index (RI) for each cluster in the Nano's current model.  The cluster assigned the most patterns has anomaly index of 0 up to a maximum of 1000 for a cluster that has only been assigned one pattern. Cluster 0 always has anomaly index of 1000.

* **frequencyIndexes:** Similar to the anomaly indexes, each value in this list gives the frequency index associated with the corresponding cluster whose ID beginning with cluster 0. These values are integers that range from 0 and up. While there is no definitive upper bound, each Nano model will have a local upper bound. Values below 1000 indicate clusters whose sizes are smaller than average, where 0 is the most common cluster size. Values above 1000 have been assigned more patterns than average and the further they are above 1000, the larger the cluster is. This statistic is a dual use value where anomalies (very small and very large) can be considered when they have values on either side of 1000.

* **distanceIndexes:** Distance indexes refer to each cluster's spatial relation to the other clusters. Values close to 1000 are very far away from the natural centroid of all of the clusters. Values close to 0 are located near the center of all the clusters. On average, these values don't vary much and develop a natural mean. This is also a dual threshold statistic since the natural mean represents the typical spacing of the clusters and there can be abnormally close clusters and abnormally distant clusters.

* **clusterGrowth:** The cluster growth curve shows the number of inferences between the creation of each new cluster (Figure 2). The list returned by clusterGrowth is the indexed pattern numbers where a new cluster was created which can be used as the x-values of this curve. The y-values can be derived as an ascending sequence of cluster IDs: 0, 1, 2, 3, etc. For instance, if clusterGrowth returns [0 1 5 7 20], the coordinates of the cluster growth plot would be: [0 0], [1 1], [5 2], [7 3] [20 4].

* **PCA:** Clusters in the Boon Nano are naturally mapped into a very high-dimensional space. This makes it difficult to meaningfully visualize the clusters on a two- or three-dimensional plot. The Nano's PCA list is similar to traditional principal component analysis in the sense it can be used to remap a high-dimensional vector into a lower dimensional space that, as far as possible, preserves distances and limits the flattening effects of projection. The PCA coordinates can be used, for example, to assign RGB values to assign a meaningful color to each cluster. Clusters with different but similar colors are from clusters whose assigned patterns are different enough to be in distinct clusters but that are still close to each relative to the other clusters in the model. The zero cluster is always the first value in the list of PCA values and is always represented by [0, 0, 0].

* **numClusters:** This is a single value that is the current number of clusters in the model including cluster 0. This value should equal the length of the lists: PCA, clusterSizes, anomalyIndexes, frequencyIndexes, distanceIndexes, clusterGrowth.

* **totalInferences:** This is the total number of patterns successfully clustered. The total of all the values in clusterSizes should also equal this value.
