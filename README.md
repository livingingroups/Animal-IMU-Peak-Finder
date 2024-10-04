# Gundog.Peaks

## Local Peak Identification Method

### Description

Gundog.Peaks is an R function developed to identify local peaks (maxima) in a given time series, such as an acceleration channel from motion sensor data. This function allows users to set criteria for detecting peaks, including a local maximum span (`LoM`) and a minimum height threshold (`thresh`) relative to a baseline constant. Gundog.Peaks was modeled after the `find_peaks()` function in the `ggpmisc` package but offers distinct differences in terms of inputs and outputs.

### Required Inputs

- **Timestamp (TS)**: A vector of timestamps in POSIXct format corresponding to the motion sensor data.
- **Data Values (x)**: A vector of data values from which you want to find peaks (e.g., acceleration along an axis).
- **Threshold (`thresh`)**: The minimum height of candidate peaks, expressed as a percentage relative to the tallest peak. The threshold is computed after subtracting a constant baseline value.
- **Local Maximum Span (`LoM`)**: The span for detecting local maxima. Default is set to 5. The value must be odd; otherwise, the function will add 1 to ensure it is odd.
- **Constant**: A baseline constant to subtract from the data before applying the threshold. The default is the median of the input values (`med`), but users can also specify the mean (`mn`), a rolling mean (`roll.mn`), or an arbitrary constant. If using a rolling mean, specify the window length (`w`), with a default value of 100.
- **Marked Events (`ME`)**: Marks which values to consider for peak detection. Only values greater than 0 are used (e.g., periods of motion, marked as 1). Default is 1.
- **Plot (`plot`)**: Set `plot = TRUE` to generate a plot of the peak spectrum over time, including identified peaks, the height threshold, baseline constant, and marked events. Default is `TRUE`.
- **Outlier (`outlier`)**: Default is `FALSE`. If changed, the height threshold is not scaled relative to the maximum value but rather to a specified quantile (e.g., `outlier = 0.99`).
- **Peaks (`peaks`)**: Specifies whether to detect positive or negative peaks. Default is `"positive"`. To detect troughs, set `peaks = "negative"`.

### Outputs

The function outputs a data frame containing:

- **Timestamp**: The timestamp of each detected peak, indicating the exact time at which the peak occurred in the original time series data.
- **Index**: The original row/element position of the detected peaks in the input data vector (`x`). This allows users to reference the position of each peak in the original dataset.
- **Peak Amplitude (`Peak.Amplitude`)**: The value of `x` at each detected peak. This represents the magnitude of the signal at the identified peak, providing insight into the intensity of movement or activity.
- **Peak Period (`Peak.Period`)**: The duration between consecutive peaks, expressed in seconds. This provides information on the frequency of peaks, which can be useful for understanding periodic behaviors or movement patterns.
- **Marked Events (`Marked.events`)**: The value of the `ME` input at each detected peak. This indicates which segments of the data were considered for peak detection, helping users understand which events were involved in the analysis.

### Required Packages [If one of these required packages is not installed, the script will install it before proceeding]. 

- **dplyr**: For data manipulation and processing.
- **data.table**: For efficient handling of large datasets.

### Usage Notes

- **Positive Values Only**: The function only considers values greater than 0 for peak detection. If negative peaks (troughs) are desired, set `peaks = "negative"`.
- **Adjustable Baseline Constant**: The baseline constant (`constant`) can be modified based on the characteristics of the input data. The default (`med`) works well for typical acceleration signals, but users can experiment with other options as needed.
- **Marked Events**: Ensure that the marked events (`ME`) properly distinguish between periods of motion and inactivity to enhance the accuracy of peak detection.

### Example Call

```R
Gundog.Peaks(TS = timestamps, x = acceleration_data, thresh = 0.3, LoM = 7, constant = "roll.mn", w = 50, ME = 1, plot = TRUE, outlier = 0.95, peaks = "positive")
```

### Example Usage with Pseudo Data

Here is an example using pseudo data to illustrate how to use the `Gundog.Peaks` function. We generate a vector of pseudo acceleration values sampled at 20 Hz for approximately 4 minutes and demonstrate how to detect peaks:

```R
# Load required packages
library(dplyr)
library(data.table)

# Generate pseudo acceleration data (20 Hz for 4 minutes)
set.seed(123)
sample_rate <- 20 # 20 Hz
num_minutes <- 4
num_samples <- sample_rate * 60 * num_minutes
timestamps <- seq.POSIXt(from = as.POSIXct("2023-01-01 00:00:00"), by = 1/sample_rate, length.out = num_samples)

# Simulate acceleration data including walking and resting periods
acceleration_data <- rep(0, num_samples)

# Generate walking bouts with realistic acceleration patterns for a penguin
for (i in seq(1, num_samples, by = 2000)) {
  duration <- sample(200:400, 1) # Walking bout duration between 10 to 20 seconds
  if (i + duration < num_samples) {
    walking_pattern <- 0.5 * sin(seq(0, pi * duration / sample_rate, length.out = duration))
    acceleration_data[i:(i + duration - 1)] <- walking_pattern + rnorm(duration, mean = 0, sd = 0.1)
  }
}

# Plot pseudo acceleration data
plot(timestamps, acceleration_data, type = "l", main = "Simulated Acceleration Data (Penguin Walking with Bouts)", xlab = "Time", ylab = "Acceleration")

# Use Gundog.Peaks to find local maxima
#constant = "med"
peaks <- Gundog.Peaks(TS = timestamps, x = acceleration_data, thresh = 50, LoM = 10, constant = "med", ME = 1, plot = TRUE, peaks = "positive")
#constant = "roll.mn"
peaks <- Gundog.Peaks(TS = timestamps, x = acceleration_data, thresh = 50, LoM = 10, constant = "roll.mn", ME = 1, plot = TRUE, peaks = "positive", w = 100)

# Print detected peaks
print(peaks)
```

In this basic example, we create a vector of `acceleration_data` that simulates walking bouts, with acceleration spikes occurring during specific walking bouts, while the majority of the data remains minimal to simulate inactivity or low activity. The data is sampled at 20 Hz, and we generate approximately 4 minutes worth of data. Walking bouts are modelled as sinusoidal segments with some added noise to reflect the irregularity of natural movement. We then use `Gundog.Peaks` to identify local maxima in the data, with a threshold (`thresh`) of 50% relative to the tallest peak and a span (`LoM`) of 10. The resulting peaks are plotted, and a summary of detected peaks is printed.