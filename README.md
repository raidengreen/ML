// PHASE 1: PRE-PROCESSING
LOAD dataset FROM "nwis.waterservices.usgs.gov.txt"
REMOVE headers and CLEAN missing values
CONVERT "gage_height" to numbers and "datetime" to time-objects
SORT data by "datetime" in ascending order

// PHASE 2: STATE SPACE RECONSTRUCTION (The "Memory" Step)
FOR i FROM 1 TO 96:
    CREATE new_column "lag_i"
    SHIFT "gage_height" forward by i intervals
    ADD "lag_i" to feature_matrix
END FOR

// PHASE 3: DEFINING THE FUTURE
SET "target_height" = "gage_height" shifted BACKWARD by 96 intervals (24 hours)
REMOVE rows containing empty (NaN) values
DE-FRAGMENT memory by creating a clean copy of the table

// PHASE 4: DATA SPLITTING & SCALING
SPLIT data at 80% mark (Training_Data = first 80%, Testing_Data = last 20%)
INITIALIZE Scaler to range [0, 1]
FIT Scaler on Training_Data
TRANSFORM both Training and Testing data using the Scaler

// PHASE 5: NEURAL NETWORK TRAINING
DEFINE Model (Architecture: 64 neurons -> 32 neurons)
SET Activation Function = 'ReLU'
SET Optimization Algorithm = 'Adam'
TRAIN Model using (Training_Data_Features, Training_Data_Target)
STOP training when the error (Loss) stops decreasing

// PHASE 6: PREDICTION & EVALUATION
GENERATE "predictions" using Testing_Data_Features
INVERSE Scaler on "predictions" to return values to FEET
CALCULATE RMSE (Error) and R-Squared (Accuracy)

// PHASE 7: BENCHMARKING
SET "persistence_guess" = Current Height (Lag_1)
CALCULATE Difference between AI_Error and Persistence_Error
OUTPUT Improvement Percentage
